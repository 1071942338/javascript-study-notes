
## 前端路由
- 路由这个概念最早出现在后端，通过⽤户请求的url导航到具体的html⻚⾯。
- 现在的前端路由不同 于传统路由，它不需要服务器解析，⽽是可以通过hash函数或者history API来实现。
- 在前端开发中，可以使⽤路由设置访问路径，并根据路径与组件的映射关系切换组件的显示
- 这整个过程都是在同 ⼀个⻚⾯中实现的，不涉及⻚⾯间的跳转，这也就是常说的单⻚应⽤（spa）。

![image](https://note.youdao.com/favicon.ico)
## 1、查看源码姿势

### 1.1 代码仓库

https://github.com/ReactTraining/react-router

### 2.2 包说明
- react-router  公共基础包
- react-router-dom 在浏览器中使⽤，依赖react-router
- react-router-native 在react-native中使用，依赖react-router

### 2.3 源码位置

- react-router 源码位置 `react-router/packages/react-router/modules/`
- react-router-dom 源码位置 `react-router/packages/react-router-dom/modules/`

### 2.4 文件结构
- `Hooks`~~暂不关注~~
- `<Link>`
    - `<NavLink>`
- `<Prompt>` ~~暂不关注~~
- `<Redirect>`
- `<Route>`
- `<Router>`
    - `<HashRouter>`
    - `<BrowserRouter>`
    - `<MemoryRouter>` ~~暂不关注~~
    - `<StaticRouter>` ~~暂不关注~~
- `<Switch>`
- generatePath ~~暂不关注~~
- history
- location ~~暂不关注~~
- match ~~暂不关注~~
- matchPath ~~暂不关注~~
- withRouter

## 3、React-Router源码分析

### 3.1 四种 Router 源码对比

```javascript
<!--HashRouter-->
//便于梳理代码结构，已删除部分代码
import React from "react";
import { Router } from "react-router";
import { createHashHistory as createHistory } from "history";
/**
 * The public API for a <Router> that uses window.location.hash.
 */
class HashRouter extends React.Component {
  history = createHistory(this.props);
  render() {
    return <Router history={this.history} children={this.props.children} />;
  }
}
export default HashRouter;

```

```javascript
<!--BrowserRouter-->
//便于梳理代码结构，已删除部分代码
import React from "react";
import { Router } from "react-router";
import { createBrowserHistory as createHistory } from "history";
/**
 * The public API for a <Router> that uses HTML5 history.
 */
class BrowserRouter extends React.Component {
  history = createHistory(this.props);
  render() {
    return <Router history={this.history} children={this.props.children} />;
  }
}
export default BrowserRouter;

```

```javascript
<!--MemoryRouter-->
//便于梳理代码结构，已删除部分代码
import React from "react";
import { createMemoryHistory as createHistory } from "history";
import Router from "./Router.js";
/**
 * The public API for a <Router> that stores location in memory.
 */
class MemoryRouter extends React.Component {
  history = createHistory(this.props);

  render() {
    return <Router history={this.history} children={this.props.children} />;
  }
}
export default MemoryRouter;

```
```javascript
<!--StaticRouter-->
//便于梳理代码结构，已删除部分代码
import React from "react";
import { createLocation, createPath } from "history";
import Router from "./Router.js";
/**
 * The public top-level API for a "static" <Router>, so-called because it
 * can't actually change the current location. Instead, it just records
 * location changes in a context object. Useful mainly in testing and
 * server-rendering scenarios.
 */
class StaticRouter extends React.Component {
  render() {
    const { basename = "", context = {}, location = "/", ...rest } = this.props;
    const history = {
      createHref: path => addLeadingSlash(basename + createURL(path)),
      action: "POP",
      location: stripBasename(basename, createLocation(location)),
      push: this.handlePush,
      replace: this.handleReplace,
      go: staticHandler("go"),
      goBack: staticHandler("goBack"),
      goForward: staticHandler("goForward"),
      listen: this.handleListen,
      block: this.handleBlock
    };
    return <Router {...rest} history={history} staticContext={context} />;
  }
}
export default StaticRouter;

```
都是基于<Router>组件实现，区别就是传递不同的参数。重点关注 <HashRouter> 和 <BrowserRouter>

### 3.2 <Router> 源码分析


```javascript
<!--Router-->
//便于梳理代码结构，已删除部分代码
class Router extends React.Component {
  render() {
    return (
      <RouterContext.Provider
        value={{
          history: this.props.history,
          location: this.state.location,
          match: Router.computeRootMatch(this.state.location.pathname),
          //只有StaticRouter在使用，暂不考虑
          staticContext: this.props.staticContext
        }}
      >
        {/*HistoryContext在hooks中使用，暂不考虑*/}
        <HistoryContext.Provider
          children={this.props.children || null}
          value={this.props.history}
        />
      </RouterContext.Provider>
    );
  }
}
```
通过源码可以看出Router的核心功能就是提供以下数据
- history，父组件传入，由history库生成
- location，组件内计算生成
- match，组件内静态方法计算生成

### 3.3 <Route> 源码分析


```javascript
<!--Route-->
//便于梳理代码结构，已删除部分代码
class Route extends React.Component {
  render() {
    return (
      <RouterContext.Consumer>
        {context => {
          return (
            <RouterContext.Provider value={props}>
              {props.match
                ? children
                  ? typeof children === "function"
                    ? __DEV__
                      ? evalChildrenDev(children, props, this.props.path)
                      : children(props)
                    : children
                  : component
                  ? React.createElement(component, props)
                  : render
                  ? render(props)
                  : null
                : typeof children === "function"
                ? __DEV__
                  ? evalChildrenDev(children, props, this.props.path)
                  : children(props)
                : null}
            </RouterContext.Provider>
          );
        }}
      </RouterContext.Consumer>
    );
  }
}
```
由于三项表达式嵌套不便于阅读，代码可以转换成

```javascript
/*
  * 1、检测是否 match
  * 1.1 不匹配，children 为函数则返回 children(props)，否则返回 null
  * 1.2 匹配，进行第2步
  *
  * 2、检查 children
  * 2.1 存在， children 为函数则返回 children(props)，否则返回 children
  * 2.2 不存在，进行第3步
  *
  * 3、检查component
  * 3.1 存在，返回React.createElement(component, props)
  * 3.2 不在存，进行第4步
  *
  * 4、检查render
  * 4.1 存在，返回 render(props)
  * 4.2 不存在，返回 null
  * */
getComponent(props, children, component, render) {
  if (props.match) {
    if (children) {
      if (typeof children === "function") {
        return children(props);
      } else {
        return children;
      }
    } else {
      if (component) {
        return React.createElement(component, props);
      } else {
        if (render) {
          return render(props);
        } else {
          return null;
        }
      }
    }
  } else {
    if (typeof children === "function") {
      return children(props);
    } else {
      return null;
    }
  }
}
```

通过源码可以看出Reoute的核心功能是渲染组件，并有以下特点

- 三者优先级 children > component > render
- children 为函数可以做到匹配与否都显示
- children 为组件只能在匹配显示
- component 只能为组件
- render 只能为函数

### 3.4 <Redirect> 源码分析

```javascript
<!--Switch-->
//便于梳理代码结构，已删除部分代码
function Redirect({ computedMatch, to, push = false }) {
  const method = push ? history.push : history.replace;
  return (
    <RouterContext.Consumer>
      {context => {
        return (
          <Lifecycle
            onMount={() => {
              method(location);
            }}
            onUpdate={(self, prevProps) => {
              method(location);
            }}
            to={to}
          />
        );
      }}
    </RouterContext.Consumer>
  );
}
```
通过源码可以看出 Redirect 的核心功能是跳转到 to 指向的页面。

### 3.5 <Switch> 源码分析
```javascript
<!--Switch-->
//便于梳理代码结构，已删除部分代码
class Switch extends React.Component {
  render() {
    return (
      <RouterContext.Consumer>
        {context => {
          const location = this.props.location || context.location;
          let element, match;
          // 匹配第一个Route
          React.Children.forEach(this.props.children, child => {
            if (match == null && React.isValidElement(child)) {
              element = child;
              const path = child.props.path || child.props.from;
              match = path
                ? matchPath(location.pathname, { ...child.props, path })
                : context.match;
            }
          });
          //匹配则返回React生成的元素，否则返回null
          return match
            ? React.cloneElement(element, { location, computedMatch: match })
            : null;
        }}
      </RouterContext.Consumer>
    );
  }
}
```
通过源码可以看出 Switch 的核心功能是渲染匹配到第一个 Route 组件。

### 3.6 withRouter 源码分析

```javascript
<!--withRouter-->
//便于梳理代码结构，已删除部分代码
function withRouter(Component) {
  const C = props => {
    const { wrappedComponentRef, ...remainingProps } = props;
    return (
      //通过 Context.Consumer 传递 Router 属性给 Component，达到增强目的
      <RouterContext.Consumer>
        {context => {
          return (
            <Component
              {...remainingProps}
              {...context}
              ref={wrappedComponentRef}
            />
          );
        }}
      </RouterContext.Consumer>
    );
  };
  //静态属性拷贝
  return hoistStatics(C, Component);
}
```
通过源码可以看出 withRouter 的核心功能是 传递 Router 属性给 Component。

### 3.7 <Link> 源码分析

```javascript
<!--Link-->
//便于梳理代码结构，已删除部分代码
const Link = forwardRef(
  (
    {
      component = LinkAnchor,
      replace,
      to,
      innerRef, // TODO: deprecate
      ...rest
    },
    forwardedRef
  ) => {
    return (
      <RouterContext.Consumer>
        {context => {
          const { history } = context;

          //把 to 属性转换成 href 属性
          const location = normalizeToLocation(
            resolveToLocation(to, context.location),
            context.location
          );
          const href = location ? history.createHref(location) : "";
          
          //组装 props 数据
          const props = {
            ...rest,
            href,
            navigate() {
              const location = resolveToLocation(to, context.location);
              const method = replace ? history.replace : history.push;

              method(location);
            }
          };
          // 设置forwardedRef
          props.ref = forwardedRef
          //创建并返回组件
          return React.createElement(component, props);
        }}
      </RouterContext.Consumer>
    );
  }
);
```

```javascript
<!--LinkAnchor-->
//便于梳理代码结构，已删除部分代码
const LinkAnchor = forwardRef(
  (
    {
      innerRef, // TODO: deprecate
      navigate,
      onClick,
      ...rest
    },
    forwardedRef
  ) => {
    const { target } = rest;

    //组装 props 数据
    let props = {
      ...rest,
      onClick: event => {
        //处理点击事件
        try {
          if (onClick) onClick(event);
        } catch (ex) {
          event.preventDefault();
          throw ex;
        }
        //执行路由跳转事件
        if (
          !event.defaultPrevented && // onClick prevented default
          event.button === 0 && // ignore everything but left clicks
          (!target || target === "_self") && // let browser handle "target=_blank" etc.
          !isModifiedEvent(event) // ignore clicks with modifier keys
        ) {
          event.preventDefault();
          navigate();
        }
      }
    };

    // 设置forwardedRef
    props.ref = forwardedRef;

    /* eslint-disable-next-line jsx-a11y/anchor-has-content */
    return <a {...props} />;
  }
);
```
通过源码可以看出 Link 的核心功能如下
- 通过 component 自定义 <Link>,否则默认使用 <LinkAnchor>
- <Link> 的主要功能就是把 to 属性转换成 href 属性
- <LinkAnchor> 主要功能是屏蔽 <a> 默认点击事件，使用 history 进行路由跳转

### 3.8 <NavLink> 源码分析
```javascript
<!--NavLink-->
//便于梳理代码结构，已删除部分代码
const NavLink = forwardRef(
  (
    {
      "aria-current": ariaCurrent = "page",
      activeClassName = "active",
      activeStyle,
      className: classNameProp,
      exact,
      isActive: isActiveProp,
      location: locationProp,
      sensitive,
      strict,
      style: styleProp,
      to,
      innerRef, // TODO: deprecate
      ...rest
    },
    forwardedRef
  ) => {
    return (
      <RouterContext.Consumer>
        {context => {
          //根据 isActive 属性判断是否 Active
          const isActive = !!(isActiveProp
            ? isActiveProp(match, currentLocation)
            : match);
          //根据 isActive 属性设置 className
          const className = isActive
            ? joinClassnames(classNameProp, activeClassName)
            : classNameProp;
          //根据 isActive 属性设置内联样式 style
          const style = isActive ? { ...styleProp, ...activeStyle } : styleProp;
          //组织 props 数据
          const props = {
            "aria-current": (isActive && ariaCurrent) || null,
            className,
            style,
            to: toLocation,
            ...rest
          };
          // 设置forwardedRef
          props.ref = forwardedRef;
          return <Link {...props} />;
        }}
      </RouterContext.Consumer>
    );
  }
);
```

通过源码可以看出 <NavLink> 的核心功能如下
- 在 <Link> 基础上允许自定义默认和激活状态样式
- isActive 优先级高于默认的路由匹配


## 4、history 源码分析

### 4.1 createBrowserHistory
```typescript
<!--createBrowserHistory-->
//便于梳理代码结构，已删除部分代码
export function createBrowserHistory( options: BrowserHistoryOptions = {} ): BrowserHistory {
  //获取 history
  let { window = document.defaultView! } = options;
  let globalHistory = window.history;


  window.addEventListener(PopStateEventType, handlePop);

  let action = Action.Pop;
  let [index, location] = getIndexAndLocation();
  let listeners = createEvents<Listener>();

  //pathname + search + hash
  function createHref(to: To) {
    return typeof to === 'string' ? to : createPath(to);
  }

  //用来处理事件订阅
  function applyTx(nextAction: Action) {
    action = nextAction;
    [index, location] = getIndexAndLocation();
    listeners.call({ action, location });
  }

  //基于 pushState 方法实现
  function push(to: To, state?: State) {
    let nextAction = Action.Push;
    let nextLocation = getNextLocation(to, state);
    function retry() {
      push(to, state);
    }
    if (allowTx(nextAction, nextLocation, retry)) {
      let [historyState, url] = getHistoryStateAndUrl(nextLocation, index + 1);

      // TODO: Support forced reloading
      // try...catch because iOS limits us to 100 pushState calls :/
      try {
        globalHistory.pushState(historyState, '', url);
      } catch (error) {
        // They are going to lose state here, but there is no real
        // way to warn them about it since the page will refresh...
        window.location.assign(url);
      }
      //用来处理事件订阅
      applyTx(nextAction);
    }
  }

  //基于replaceState方法实现
  function replace(to: To, state?: State) {
    let nextAction = Action.Replace;
    let nextLocation = getNextLocation(to, state);
    function retry() {
      replace(to, state);
    }

    if (allowTx(nextAction, nextLocation, retry)) {
      let [historyState, url] = getHistoryStateAndUrl(nextLocation, index);

      // TODO: Support forced reloading
      globalHistory.replaceState(historyState, '', url);

      //用来处理事件订阅
      applyTx(nextAction);
    }
  }

    //基于go方法实现
  function go(delta: number) {
    globalHistory.go(delta);
  }

  let history: BrowserHistory = {
    get action() {
      return action;
    },
    get location() {
      return location;
    },
    createHref,
    push,
    replace,
    go,
    back() {
      go(-1);
    },
    forward() {
      go(1);
    },
    listen(listener) {
      return listeners.push(listener);
    },
    block(blocker) {...}
  };

  return history;
}
```
- back、forward 都是基于 go 方法实现
- push 基于 pushState 方法实现
- replace 基于 replaceState 方法实现
- 浏览器相关（url输入、前进、后退）事件通过监听 popstate 事件处理，然后通过事件订阅的形式传递给Route组件

### 4.2

- createHashHistory 同 createBrowserHistory
- createMemoryHistory 同 createBrowserHistory


> 参考链接
- [History.pushState](https://developer.mozilla.org/zh-CN/docs/Web/API/History/pushState)
- [History.replaceState](https://developer.mozilla.org/zh-CN/docs/Web/API/History/replaceState)
- [popstate_event](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/popstate_event)
- [hashchange_event](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/hashchange_event)