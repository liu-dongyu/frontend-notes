## `styled-components`有可能无法覆盖`antd`的默认样式

```js
// StyledAlert.jsx
import styled from "styled-components";
import Alert from "antd/lib/alert/index";
import "antd/lib/alert/style/css";

const StyledAlert = styled(Alert)`
  background-color: red;
`;

export default StyledAlert;
```

```js
// StyledOther.jsx
import styled from "styled-components";

const StyledOther = styled.div`
  color: red;
`;

export default StyledOther;
```

```js
// page
import StyledDemo from "StyledOther";
import StyledAlert from "StyledAlert";

const Page = () => {
  return (
    <div>
      <StyledOther>我是红色的</StyledOther>
      <StyledAlert message="我依旧是默认的蓝色" />
    </div>
  );
};

export default Page;
```

**问题：**

开发过程中，会发现`StyledAlert`书写的样式无法覆盖`Alert`的样式

**原因：**

`webpack`会根据`import`的顺序将样式代码生成到`<style>`中，由于`StyledOther`引入更早，所以生成的`<style>`会在`StyledAlert`的前面，
且因为所有当前页面相关的`styled-components`样式都会插入到首次使用 `styled-components`生成的`<styled>`中，所以`StyledAlert`样式的优先级不如`Alert`

**解决方案：**

1. 调整`import`引入顺序，确保`antd`组件在`styled-components`组件前面，例如先引入

```js
import StyledAlert from "StyledAlert";
import StyledDemo from "StyledOther";
```

2. 通过`&&`，提高`styled-components`组件的样式权重，例如

```js
const StyledAlert = styled(Alert)`
  && {
    background-color: red;
  }
`;
```
