---
title: React路由守卫和全局loading控制
tags: react
abbrlink: 25787
date: 2019-11-26 12:13:25
---

在前后端分离的项目中，我们通常会遇到实现前端路由权限的需求以及全局 loading 效果的需求，在 Vue 项目中，我们可以通过路由守卫 beforEach、afterEach 这个两个钩子函数来实现进入一个路由时的全局 loading 效果。而 vue-router 也提供了灵活的路由配置项允许我们赋予路由更多的信息，包括权限等等。反观 react-router 并没有直接提供给这样的组件。虽然说 vue-router 本身就提供了灵活的配置，但是 React 高阶组件也赋予了我们大展身手的机会。

<!-- more -->

## 封装路由组件

```tsx
const App: React.FC = () => (
  <Provider store={store}>
    <div className="App">
      <Switch>
        <AuthRoute config={RouteConfig} />
      </Switch>
    </div>
  </Provider>
);

export default withRouter(App);
```

在最外部我们不使用 react-router 提供的 Route 的组件，而是使用我们自己封装的路由组件，这个组件接受一个 config 参数，传入路由配置，这样我们也可以像 vue 中那样编写路由配置文件了。

## 路由配置文件

定义单个路由配置的类型

```tsx
export interface RouteItem {
  path: string;
  component?: FC;
  auth?: boolean;
}
```

最后 export 出的路由配置信息，就是由 RouteItem 组成的数组。path 代表路由路径，component 表示对应的组件，auth 表示是否需要鉴权，如果有多种角色的话，那么将 auth 设置成角色名称，后面增加一下判断方式便可。

## 全局 loading 的 Redux 设计

既然要实现全局的 loading，那么使用 redux 最合适不过了。
这里就直接贴代码了，redux 的知识就不细说了。
由于使用了 combineReducers，所有我们把 loading 的状态放在了 app 这个 reducer 中。
**actionTypes.ts**

```ts
const SET_LOADING = "SET_LOADING";

export default {
  /**
   * 设置页面的loading状态
   */
  SET_LOADING
};
```

**app.action.ts**

```ts
import actionTypes from "./actionTypes";

export const setLoading = (newStatus: boolean) => ({
  type: actionTypes.SET_LOADING,
  data: newStatus
});
```

**app.reducer.ts**

```ts
import actionTypes from "./actionTypes";

export interface AppState {
  loading: boolean;
}

const defaultState: AppState = {
  loading: false
};

export default (state = defaultState, action: any) => {
  switch (action.type) {
    case actionTypes.SET_LOADING:
      return { ...state, loading: action.data };
    default:
      return state;
  }
};
```

## 实现 AuthRoute 组件

由于 AuthRoute 组件放在了 Switch 组件内部，React Router 还自动为 AuthRoute 注入了 location 属性，当地址栏的路由发生变化时，就会触发 location 属性对象上的 pathname 属性发生变化，我们根据这个变化，再去匹配先前写好的路由配置获得相应的组件重新渲染就可以了。

### 实现全局 loading

我们只需要在 Route 组件的外部包裹一层 Spin 组件就可以了，spin 组件的 loading 状态就是 redux 中的 loading，如果需要根据网络请求来决定 loading 时间，只需要在相应的组件里设置 loading 的值就可以了，为了方便看效果，我这里就直接用定时器了。

### 代码

```tsx
const AuthRoute: React.FC<any> = props => {
  const dispatch = useDispatch();
  const loading: boolean = useSelector((state: Store) => state.app.loading);

  const { pathname } = props.location;
  const isLogin = localStorage.getItem("user_token");
  let timer = 0;

  useEffect(() => {
    window.scrollTo(0, 0);
    dispatch(setLoading(true));
    clearTimeout(timer);
    timer = window.setTimeout(() => {
      dispatch(setLoading(false));
    }, 1000);
  }, [pathname]);

  const targetRouterConfig: RouteItem = props.config.find(
    (v: RouteItem) => v.path === pathname
  );

  if (targetRouterConfig && !targetRouterConfig.auth && !isLogin) {
    const { component } = targetRouterConfig;
    return <Route exact path={pathname} component={component} />;
  }
  if (isLogin) {
    // 如果是登陆状态，想要跳转到登陆，重定向到主页
    if (pathname === "/login") {
      return <Redirect to="/" />;
    }
    // 如果路由合法，就跳转到相应的路由
    if (targetRouterConfig) {
      return (
        <Spin
          tip="Loading"
          size="large"
          spinning={loading}
          // indicator={<Icon type="loading" style={{ fontSize: 24 }} spin />}
          style={{ maxHeight: "none" }}
        >
          <Route path={pathname} component={targetRouterConfig.component} />
        </Spin>
      );
    }
    // 如果路由不合法，重定向到 404 页面
    return <Redirect to="/404" />;
  }
  // 非登陆状态下，当路由合法时且需要权限校验时，跳转到登陆页面，要求登陆
  if (targetRouterConfig && targetRouterConfig.auth) {
    return <Redirect to="/login" />;
  }
  // 非登陆状态下，路由不合法时，重定向至 404
  return <Redirect to="/404" />;
};

export default AuthRoute;
```
