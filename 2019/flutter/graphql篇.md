## 前言

`graphql`是一种用于 API 的查询语言，使用得当能带来成产力的提高，本文记录如何相对舒服的在`flutter`中使用`graphql`

## 开始

### 构建基于`schema`的 dart 类型描述

> [graphql-to-dart](https://github.com/micimize/graphql-to-dart)：通过 node 将 schema 转换为 dart 描述

1. 在 package.json 中添加如下代码，然后执行`npm install`。（如没有 package.json，先运行`npm init`）

```json
 // package.json

 "scripts": {
    "gql-gen": "gql-gen",
    "format": "flutter format lib/types.dart",
    "source-gen": "flutter packages pub run build_runner build",
    "build": "yarn gql-gen && yarn source-gen && yarn format"
  },
  "dependencies": {
    "graphql": "^14.5.4",
    "graphql-code-generator": "^0.18.2",
    "graphql-to-dart": "^0.3.6"
  }
```

2. 根目录创建 codegen.yml 文件

```yml
# codegen.yml

schema: "http://127.0.0.1:4000/graphql" # graphql请求地址，也可以指定本地文件
overwrite: true
generates:
  ./lib/graphql/types.dart:
    plugins:
      - graphql-to-dart
config:
  requiredFields: false # 如果为true，带有!的属性必须在对应query或mutation中加入查询，否则反序列化时会出错
  parts:
    - "types.g.dart"
```

5. 在 pubspec.yml 中添加对应 flutter 依赖，然后运行`flutter pub get`进行安装

```yml
# pubspec.yml

dev_dependencies:
  # ...
  build_runner: ^1.0.0
  json_serializable: ^3.2.0

dependencies:
  # ...
  json_annotation: ^3.0.0
```

6. 运行`npm run build`，若执行成功就会在 lib 文件夹中看到 type.dart 和 type.d.dart

### 配置 graphql client

> [flutter_graphql](https://github.com/zino-app/graphql-flutter)

在 pubspec.yml 中添加相关依赖，并执行`flutter pub get`

```yml
# pubspec.yml

dependencies:
  # ...
  graphql_flutter: ^2.1.0
```

在 main.dart 中配置 graphql 客户端

```dart
import 'package:flutter/material.dart';
import 'package:graphql_flutter/graphql_flutter.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final HttpLink httpLink = HttpLink(
      // graphql请求地址
      uri: 'http://127.0.0.1:4000/graphql',
    );

    final AuthLink authLink = AuthLink(
      // 每次请求时都会触发的方法，可以从持久化中(例如SharedPreferences)或其他方式获取token
      getToken: () {
        return '';
      },
    );

    final Link link = authLink.concat(httpLink);

    final ValueNotifier<GraphQLClient> client = ValueNotifier(
      GraphQLClient(
        link: link,
        cache: NormalizedInMemoryCache(
          // 以id和__typename为了key缓存结果
          dataIdFromObject: typenameDataIdFromObject,
        ),
      ),
    );

    Widget rlt = MaterialApp(
      // ...
    );

    // offline Cache
    rlt = CacheProvider(
      child: rlt,
    );

    return GraphQLProvider(
      child: rlt,
      client: client,
    );
  }
}
```

## 使用方式，以 query 为例

#### 假设 schema 定义如下

```gql
input BaseType {
  aa: String
}

type Me {
  is_success: Boolean!
  msg: String
  result: User!
}

type User {
  id: Float
  nickname: String
}

type Query {
  me(input: BaseType!): Me!
}
```

#### 定义 query 语句查询 me

```dart
final String meQuery = r'''
  query Me($input: BaseType!) {
    me(input: $input) {
      is_success,
      msg,
      result {
         id
         nickname
      }
    }
  }
'''
```

#### query code fragment

```dart
// schema对应的dart类型描述
import 'type.dart';
// Query定义和type.dart命名冲突，如果要使用该widgets，可以import '' as gql，然后gql.Query
import 'package:graphql_flutter/graphql_flutter.dart' hide Query;

//...

// 从context中获取graphql client
final GraphQLClient client = GraphQLProvider.of(context).value;

// 定义请求语句和对应参数
// 其中QueryMeArgs和BaseType由type.dart提供
final QueryOptions options = QueryOptions(
  document: meQuery,
  variables: QueryMeArgs(
    input: BaseType(
      // xxx: xxx
    ),
  ).toJson(),
);

// 发起请求
final QueryResult result = await client.query(options);

// 请求异常
if (result.hasErrors) {
  // result.errors
}

// 反序列化获取到的结果
// 其中Me，Query.deserializeFromJson由type.dart提供
final Me me = Query.deserializeFromJson(result.data).me;

// 获取想要的user
// 其中User由type.dart提供
final User user = me.result;

// user.id
// user.nickname
```

**注意!!**

套一层 is_suceess 是不好的做法，一来书写多一层嵌套，二来当 is_success 为 false 时，结果也会被缓存，因为在 client 看来，这就是正确的返回。  
更恰当的方式是走 graphql 本身的 error handle。

## 其他

### 善用 graphql fragment

为了避免各处都一坨坨 graphql 语句，常用的返回可以通过 fragment 定义，例如

```

const gqlMeFidlds = r'''
fragment meFields on User {
  __typename
  id
  gender
  status
  created_at
  updated_at
  nickname
  image_count
  // ...
}
'''

```

```dart
final String meQuery = r'''
  query Me($input: BaseType!) {
    me(input: $input) {
      is_success,
      msg,
      result {
         ...meFields
      }
    }
  }
''' + gqlMeFidlds;
```

### graphql_flutter 对 error 和 data 的处理存在 Bug，可见 [issues](https://github.com/zino-app/graphql-flutter/issues/405)

从分析源码得知，`query`默认的`ErrorPolicy`为`none`，

```dart
// query_options.dart第70行

class QueryOptions extends BaseOptions {
  QueryOptions({
    // ...
    ErrorPolicy errorPolicy = ErrorPolicy.none,
    // ...
  }) : super(
          // ...
        );
}
```

即当出现 error 时，只处理 error 并返回，忽略 data。

```dart
// query_manager.dart第332行

switch (options.errorPolicy) {
  case ErrorPolicy.all:
    // handle both errors and data
    errors = _errorsFromResult(fetchResult);
    data = fetchResult.data;
    break;

  case ErrorPolicy.ignore:
    // ignore errors
    data = fetchResult.data;
    break;

  case ErrorPolicy.none:
  default:
    // TODO not actually sure if apollo even casts graphql errors in `none` mode,
    // it's also kind of legacy
    errors = _errorsFromResult(fetchResult);
    break;
}
```

但缓存的动作在处理 error 以前，且和`ErrorPolicy`无关联。

```dart
// query_manager.dart第115行

if (fetchResult.data != null && options.fetchPolicy != FetchPolicy.noCache) {
  cache.write(
    operation.toKey(),
    fetchResult.data,
  );
}
```

导致当存在 error 时，如服务端一并返回 data，client 会 cache 该 data，当请求参数不变更时，会一直返回该 data，不再请求服务端

**暂时的处理方式**

在`QueryOptions`中，设置`fetchPolicy`为`FetchPolicy.noCache`
