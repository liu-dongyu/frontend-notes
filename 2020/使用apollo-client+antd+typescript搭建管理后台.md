## 前言

基于 apollo-client + antd + typescript 快速搭建管理后台，具备开发体验舒适、开发效率高、可维护性高等优点。本文用于记录基础，方便日后回顾

#### [apollo-client](https://www.apollographql.com/docs/react/)

简单说，apollo-client 是用于对接 graphql 端的框架。个人认为最大的杀手锏是 apollo 会缓存所有`query`获取到的数据并提供状态管理能力，当`mutation`修改了某些数据时，只要正确返回该数据所在的`type`，所有使用了相关`query`的页面都会自动变更，不需要人为进行状态管理。

#### [ant](https://ant.design/index-cn)

antd 提供了一套相对完整的 react 组件库，主要用于制作管理后台。兼容主流浏览器以及 IE10+(需要 polyfill)

#### [TypeScript](https://www.typescriptlang.org/)

TypeScript 是 JavaScript 的超集，增加了类型检查以及便捷的语法糖（例如可选链式调用，`a?.b?.c`）。个人认为最大的杀手锏在于类型检查，增加了代码的可维护性和减少手误等导致的 bug。配合 graphql 时开发体验更是舒适，根据 schema 生成类型说明文件，对接接口不再需要查看文档，开发过程中编辑器就会自动补全相应字段，省心省力。

## 正文

### 一: 通过`create-react-app`创建初始项目

#### 1.1 如曾经全局安装过 create-react-app，先运行以下命令

```bash
npm uninstall -g create-react-app
which create-react-app # eg: /usr/local/bin/create-react-app
rm -rf /usr/local/bin/create-react-app
```

#### 1.2 创建名为 my-admin 的项目

```bash
npx create-react-app my-admin --template typescript
cd my-admin
npm install
```

> 执行完毕后，运行`npm run start`，打开 localhost:3000 看到`react`标志则初始化成功

#### 1.3 配置 tslint

```bash
npm install tslint tslint-config-prettier tslint-eslint-rules tslint-loader tslint-react tslint-react-hooks --save-dev
```

<font size="3">在根目录创建 tslint.json，属于以下配置</font>

```json
{
  "defaultSeverity": "error",
  "extends": [
    "tslint:recommended",
    "tslint-react",
    "tslint-eslint-rules",
    "tslint-config-prettier",
    "tslint-react-hooks"
  ],
  "rules": {
    // 允许使用todo注视
    "no-suspicious-comment": false,
    // 允许使用console
    "no-console": false,
    // import不需要按照字母从小到大排序
    "ordered-imports": false,
    // obj属性不需要按字母从小到大排序
    "object-literal-sort-keys": false,
    // class不需要显示定义public等
    "member-access": false,
    // 允许使用<reference path=>
    "no-reference": false,
    // 不满足tslint-react-hooks规则的标记为错误
    "react-hooks-nesting": "error"
  }
}
```

<font size="3">打开 package.json，将 scripts 命令相关改为如下</font>

```json
{
  "scripts": {
    "lint": "tslint --project \"tsconfig.json\""
  }
}
```

#### 1.4 兼容 IE11

> 详情查看 [supported-browsers](https://create-react-app.dev/docs/supported-browsers-features/#supported-browsers)

```bash
npm install react-app-polyfill --save
```

```ts
// 打开src/index.tsx，增加如下代码

import "react-app-polyfill/ie11";
import "react-app-polyfill/stable";
```

#### 1.5 配置 Alias

> 希望能通过 @pages/xxx.tsx 来访问 src/pages/xxx.tsx 文件

<font size="3">打开 config-overrides.js ，增加如下代码</font>

```ts
const { override, addWebpackAlias } = require("customize-cra");
const path = require("path");

module.exports = override(
  addWebpackAlias({
    ["@pages"]: path.resolve(__dirname, "src/pages")
  })
);
```

<font size="3">根目录新建 tsconfig.paths.json</font>

```json
{
  "compilerOptions": {
    "paths": {
      "@pages/*": ["./src/pages/*"]
    }
  }
}
```

<font size="3">打开 tsconfig.json，增加如下代码</font>

```json
{
  "extends": "./tsconfig.paths.json",
  "compilerOptions": {
    "baseUrl": "."
  }
}
```

---

### 二：添加 antd

```bash
npm install antd dayjs --save
npm install customize-cra react-app-rewired babel-plugin-import --save-dev
```

<font size="3">打开 package.json，将 scripts 命令相关改为如下</font>

```json
{
  "scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build"
  }
}
```

<font size="3">在根目录添加 config-overrides.js，配置如下</font>

```ts
const {
  override,
  fixBabelImports,
  addWebpackPlugin
} = require("customize-cra");

module.exports = override(
  /**
   * 这个东西用来干嘛的 ？ 
   *
   * 该插件用于将
   * import { Button } from 'antd';
   * 转化为
   * var _button = require('antd/lib/button');
   * require('antd/lib/button/style/css');
   *
   * 所以不用也可以，但需要每次都自行指定组件的路径和引入相应样式
   */
  fixBabelImports("import", {
    libraryName: "antd",
    libraryDirectory: "es",
    style: "css"
  })
);
```

<font size="3">打开 src/index.tsx，修改为如下代码</font>

```ts
import React from "react";
import ReactDOM from "react-dom";
import zh_CN from "antd/lib/locale-provider/zh_CN";
import { Alert, ConfigProvider } from "antd";

ReactDOM.render(
  <ConfigProvider locale={zh_CN}>
    <Alert message="Hello word" />
  </ConfigProvider>,

  document.getElementById("root")
);
```

---

### 三：路由

#### 3.1 基础配置

```bash
npm install react-router-dom --save
npm install @types/react-router-dom --save-dev
```

<font size="3">创建 src/pages/News.tsx</font>

```ts
import React from "react";

const News: React.FC = () => {
  return <div>news</div>;
};

export default News;
```

<font size="3">创建 src/Route.tsx</font>

```ts
import React, { Suspense, lazy } from "react";
import { Route, Switch } from "react-router-dom";
import { Skeleton } from "antd";

const News = lazy(() => import("@pages/News.tsx"));

const MyRoute = () => {
  return (
    <BrowserRouter>
      <Suspense fallback={<Skeleton active={true} />}>
        <Switch>
          <Route path="/news" component={News} />
        </Switch>
      </Suspense>
    </BrowserRouter>
  );
};

export default MyRoute;
```

<font size="3">打开 src/index.tsx，修改如下代码</font>

```ts
import MyRoute from "./Route";

ReactDOM.render(
  <ConfigProvider locale={zh_CN}>
    <Alert message="Hello word" />
    <MyRoute />
  </ConfigProvider>,

  document.getElementById("root")
);
```

#### 3.2 设置私人路由

```ts
import { Route, Redirect, RouteProps } from "react-router-dom";
import { Result } from "antd";

const AccessDenied = () => (
  <Result
    status="403"
    title="403"
    subTitle="您没有权限访问该页面"
    extra={null}
  />
);

// eg: <PrivateRoute path={'/news'} component={News} />
const PrivateRoute = ({ ...rest }: RouteProps) => {
  /**
   * eg: loading中间状
   *
   * if (loading) return <Skeleton active={true} />
   */

  /**
   * eg: 判断登录
   *
   * if (!isLogin) return <Redirect to={{ pathname: '/login' }} />
   */

  /**
   * eg: 判断权限
   *
   * if (noPermission) return <Route {...rest} component={AccessDenied} />
   */

  return <Route {...rest} />;
};
```

---

### 四：配置 apollo

> 需要 graphql 服务器，如果没有，请参考[apollo-server](https://www.apollographql.com/docs/apollo-server/getting-started/)，以下均以 localhost:4000/graphql 做例子

```bash
npm install apollo --save-dev
npm install apollo-boost @apollo/react-hooks graphql-tag --save
```

#### 4.1 下载 schema

```json
// package.json，添加schema命令

{
  "scripts": {
    "schema": "apollo client:download-schema schema.graphql --endpoint http://localhost:4000/graphql"
  }
}
```

```bash
npm run schema
```

> 执行成功后，会自动在根目录添加 schema.graphql 文件，可用于开发时参考

#### 4.2 配置 client

<font size="3">新建 src/graphql/client.ts</font>

```ts
import ApolloClient from "apollo-boost";

const client = new ApolloClient({
  uri: "http://localhost:4000/graphql",
  // 添加自定义header，例如authentication token
  request: operation => {
    // 可以从localstorage等地方获取，每次请求时都会附带，可为异步方法(await)
    const token = "xxxx";

    operation.setContext({
      headers: {
        authorization: token
      }
    });
  },
  // 统一处理错误返回
  onError: ({ graphQLErrors, networkError }) => {
    // networkError: 网络错误
    // graphQLErrors: 服务端抛出的错误
    // 状态码: networkError && 'statusCode' in networkError && networkError.statusCode;
  }
});
```

<font size="3">src/index.tsx, 添加如下代码</font>

```ts
import apolloClient from "@graphql/client"; // 需要先配置alias
import { ApolloProvider } from "@apollo/react-hooks";

ReactDOM.render(
  <ApolloProvider client={apolloClient}>xxxx</ApolloProvider>,
  document.getElementById("root")
);
```

#### 4.3 配置自动生成 ts 类型描述文件

<font size="3">打开 package.json，添加新的 scripts 命令</font>

```json
{
  "scripts": {
    // 识别由graphql-tag方法提供的gql方法，根据内容自动在src/__generated__文件夹内生成描述文件，--watch表示开启监听
    "codegen": "apollo client:codegen src/__generated__ --target typescript --endpoint http://localhost:4000/graphql --outputFlat --watch"
  }
}
```

#### 4.4 本地的全局状态管理 local state management

> 假设要将登录人员的 token 保存为全局状态

<font size="3">创建 src/graphql/typeDefs.ts，用于定义 schema</font>

```ts
import gql from "graphql-tag";

// 客户端schema
const typeDefs = gql`
  # 当前登录的用户
  type LoggedUser {
    id: Float
    token: String!
  }

  # 设置登录用户需要的参数
  input LoggedUserInput {
    id: Float
    token: String!
  }

  extend type Query {
    # 获取已登录的用户
    loggedUser: LoggedUser
  }

  extend type Mutation {
    # 保存用户信息
    saveLoggedUser(input: LoggedUserInput!): LoggedUser!
  }
`;

export default typeDefs;
```

<font size="3">创建 src/graphql/resolvers.ts，用于定义 resolvers</font>

```ts
import { Resolvers } from "apollo-boost";

const resolvers: Resolvers = {
  Mutation: {
    saveLoggedUser: (_, variables, { cache }) => {
      const loggedUser = { ...variables.input, __typename: "LoggedUser" };
      cache.writeData({ data: { loggedUser } });

      return loggedUser;
    }
  }
};

export default resolvers;
```

<font size="3">打开 src/graphql/client.ts，添加 resolvers 和 typeDefs</font>

```ts
import typeDefs from "./typeDefs";
import resolvers from "./resolvers";

const client = new ApolloClient({
  // xxx
  resolvers,
  typeDefs
});
```

<font size="3">创建 src/graphql/gql.ts，用于定义 grqphql 语句</font>

```ts
import gql from "graphql-tag";

// 获取本地的登录人员信息
export const LOGGED_USER = gql`
  query LoggedUser {
    loggedUser @client {
      id
      token
    }
  }
`;

// 将登录信息保存到全局
export const SAVE_LOGGED_USER = gql`
  mutation SaveLoggedUser($input: LoggedUserInput!) {
    saveLoggedUser(input: $input) @client {
      id
      token
    }
  }
`;
```

> 保存 gql.ts 后，执行`npm run codegen`，会看到相关类型描述文件已生成在 src/**generated**文件夹中

<font size="3">获取全局登录人员信息</font>

```ts
import { LoggedUser } from "@generated/LoggedUser";
import { useQuery } from "@apollo/react-hooks";
import { LOGGED_USER } from "@graphql/gql";

// LoggedUser是data的类型描述，由npm run codegen生成
const { data, loading } = useQuery<LoggedUser>(LOGGED_USER);
```

<font size="3">设置全局登录人员信息</font>

```ts
import {
  SaveLoggedUser,
  SaveLoggedUserVariables
} from "@generated/SaveLoggedUser";
import { useMutation } from "@apollo/react-hooks";
import { SAVE_LOGGED_USER } from "@graphql/gql";

// SaveLoggedUser是saveLoggedUser返回结果的类型描述，由npm run codegen生成
// SaveLoggedUserVariables是saveLoggedUser所需要的参数的类型描述，由npm run codegen生成
const [saveLoggedUser] = useMutation<SaveLoggedUser, SaveLoggedUserVariables>(
  SAVE_LOGGED_USER
);

await saveLoggedUser({
  variables: {
    input: {
      id: 1,
      token: "123"
    }
  }
});
```

> 请求服务器端数据时，操作和请求全局状态类似，都是先添加定义 grqphql 语句，后使用`useQuery`或`useMutation`

---

### 五：问题集

#### 5.1 antd 中，如何 s 使用 Dayjs 替换 Momentjs

```bash
npm install dayjs --save
npm install antd-dayjs-webpack-plugin --save-dev
```

<font size="3">打开 config-overrides.js，增加如下配置</font>

```ts
const { override, addWebpackPlugin } = require("customize-cra");
const AntdDayjsWebpackPlugin = require("antd-dayjs-webpack-plugin");

module.exports = override(
  /**
   * 这个东西用来干嘛的？
   *
   * 使用dayjs替换antd默认使用的Moment.js
   */
  addWebpackPlugin(new AntdDayjsWebpackPlugin())
);
```

<font size="3">打开 src/index.tsx，增加如下代码</font>

```ts
import "dayjs/locale/zh-cn";
import dayjs from "dayjs";
import customParseFormat from "dayjs/plugin/customParseFormat";
import weekOfYear from "dayjs/plugin/weekOfYear";
import isMoment from "dayjs/plugin/isMoment";
import badMutable from "dayjs/plugin/badMutable";
import localeData from "dayjs/plugin/localeData";
import advancedFormat from "dayjs/plugin/advancedFormat";
import weekYear from "dayjs/plugin/weekYear";
import isSameOrBefore from "dayjs/plugin/isSameOrBefore";
import isSameOrAfter from "dayjs/plugin/isSameOrAfter";

dayjs.extend(isSameOrBefore);
dayjs.extend(isSameOrAfter);
dayjs.extend(advancedFormat);
dayjs.extend(customParseFormat);
dayjs.extend(weekYear);
dayjs.extend(weekOfYear);
dayjs.extend(isMoment);
dayjs.extend(localeData);
dayjs.extend(badMutable);
dayjs.locale("zh-cn");
```

#### 5.2 react-route 如何添加 nprogress

> [nprogress](https://github.com/rstacruz/nprogress) 用于切换页面时，在浏览器顶部显示过渡动画

```bash
npm install nprogress --save
npm install @types/nprogress --save-dev
```

<font size="3">打开 src/Route.tsx，添加如下代码</font>

```ts
import "nprogress/nprogress.css";
import nprogress from "nprogress";

const MyRoute = () => {
  nprogress.start();

  useEffect(() => {
    nprogress.done();
  });

  // xxx
};
```

#### 5.3 如何手动触发 useQuery

官方提供了`useLazyQuery`，但存在[多次调用问题](https://github.com/apollographql/react-apollo/issues/3505)。暂时可通过以下方式模拟

```ts
const [skipQuery, setSkipQuery] = useState(true);

const { data, loading } = useQuery(SOME_QUERY, {
  skip: skipQuery
});

// 当想要触发query时
skipQuery(false);
```

#### 5.4 如何对 antd Form 组件添加 Typescript 类型支持

```ts
import React from "react";
import { FormComponentProps } from "antd/lib/form";

interface IDemo extends FormComponentProps {
  text: string;
}

const Demo: React.FC<IDemo> = ({ form, text }) => {
  // xxxx
};

export default Form.create<IDemo>({})(Demo);
```

#### 5.5 [Warning: Can't perform a React state update on an unmounted component]

<font size="3">某些时候，需要在异步操作后修改组件状态，如果在此之前组件被 unmounted 了，那修改相关状态就会报错</font>

<font size="3">可通过自定义 hook 判断组件是否 mounted</font>

```ts
import { useRef, useEffect } from "react";

const useIsMounted = () => {
  const isMounted = useRef(false);
  useEffect(() => {
    isMounted.current = true;

    return () => {
      isMounted.current = false;
    };
  }, []);

  return isMounted;
};

export default useIsMounted;
```

```ts
const isMounted = useIsMounted();

if (isMounted) {
  // 组件还活着，放心修改状态
}
```

#### 5.6 如何记录上一个的 render 状态值

```ts
import { useRef, useEffect } from "react";

export default function usePrevious<T>(
  value?: T
): [T | undefined, (argValue: T) => void] {
  const ref = useRef<T>();
  const setCurrent = (argValue: T) => (ref.current = argValue);

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return [ref.current, setCurrent];
}
```

```ts
const [preState, setPreState] = usePrevious();
```
