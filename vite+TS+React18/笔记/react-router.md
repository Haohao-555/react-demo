## 一：前期准备

页面 Home

```tsx
import React from 'react'
const Home = () : React.FC => {
    return (
        <div>Home 页面</div>
    )
}
export default Home
```

页面 About

```tsx
import React from 'react'
const About = () : React.FC => {
    return (
        <div>About 页面</div>
    )
}
export default About
```

页面 detail

```tsx
import React from 'react'
const detail = () : React.FC => {
    return (
        <div>detail 页面</div>
    )
}
export default detail
```

<br/>

## 二：v5 路由配置

### 1、基本使用

* `src/router/App.tsx`

  > * `exact`：使用精准匹配
  > * `Switch`：每次只会渲染一条路由
  > * `Redirect`：重定向

  ```tsx
  import { Router, Switch, BrowserRouter, Redirect } from 'react-router-dom';
  import Home from '@/views/Home/index'
  import About from '@/views/About/index'
  function App() {
      // 路由优先级，最上面优先级越高
      return (
          <div className="app">
              <BrowserRouter>
                   <Switch> 
                       <Route exact path="/" component={<Redirect to="/home" />} />
                       <Route exact path="/home" component={Home}  />
                       <Route path="/about" component={About} />
                       <Route path="/detail/:id" component={Detail}>
                       <Route render={() => <h1>404</h1>} />
                  </Switch>
              </BrowserRouter>
          </div>
      )
  }
  ```

### 2、获取路由信息

```tsx
// localhost:3000/detail/12
import React from 'react'
import { RouteComponentProps } from 'react-router-dom'
interface MathParams {
    id: string
}
export const Detail : React.FC<RouteComponentProps<MathParams>> = (props) => {
    console.log(props.history);
    console.log(props.location);
    console.log(props.match);
    
    // 获取url参数
    console.log(props.match.params.id);
    return <h1>详情页面</h1>
}
```

> 同理：类组件也是**通过 props 来传递路由信息**

### 3、跨组件路由信息传递

> 什么是跨组件路由信息传递，比如 `home 页面` 可以获取到路由信息，但 `header 组件`，应该如何获取到路由信息

```tsx
import React from 'react'
import { Header } from '@/component/Header/index.tsx'
export class Home extends React.Component {
    render() {
        return (
          <div className="home">
                <Header>头部</Header>
          </div>
        )
    }
}
```

```tsx
import React from 'react';
export const Header : React.FC = () => {
    return (
       <div className="header"></div>
    )
}
```

#### 3.1、HOC 高阶组件

```tsx
import React from 'react'
import { withRouter } from 'react-router-dom'
const HeaderComponent : React.FC = ({ history, location, match }) => {
    
    console.log(history);
    console.log(location);
    console.log(match);
    
    return (
       <div className="header">
            <span className="home" onClick={() => {history.push("/home")}}>主页</span> 
       </div>
    )
}
export const Header = withRouter(HeaderComponent);
```

> **headerComponent 可以是函数式组件、类组件**

#### 3.2、hook 钩子

```tsx
import React from 'react'
import { useHistory, useLocation, useParams, useRouteMatch } from 'react-router-dom'
export const header : React.FC = () => {
     // 所有路由信息
     console.log(useHistory());
     // 当前路由信息
     console.log(useLocation());
     // 当前路由的参数 ?a=1&&b=2 
     console.log(useParams());
     // 当前路由匹配的参数 localhost:3000/:id
     console.log(useRouteMatch());
     return (
       <div className="header">
            <span className="home" onClick={() => {useHistory().push("/home")}}>主页</span> 
       </div>
    )
}
```

> **header 只能是 函数式组件**

### 4、路由信息有哪些？

* 对于页面而言可以获取 
  * **history**
  * **location**
  * **match**
* 对于跨组件可以获取
  * 通过 HOC 高阶组件 处理后 **history**、 **location**、 **match**
  * 通过 hook 函数 **useHistory()**、**useLocation()**、**useParams()**、**useRouteMatch()**

<br/>

## 三：v6路由配置方式一

### 1、基本使用

* `src/router/index.tsx`

  > * `BrowserRouter`： History 模式
  > *  `HashRouter`：Hash 模式
  > * `Navigate`：路由重定向

  ```tsx
  import App from '@/App'
  import Home from '@/views/Home/index'
  import About from '@/views/About/index'
  
  import { BrowserRouter, HashRouter, Routes, Route, Navigate } from 'react-router-dom'
  
  const baseRouter = () => (
    <BrowserRouter>
        <Routes>
            <Routes path="/" element={<App/>}>
                <Route path="/" element={<Navigate to="/home"/>}></Route> 
                <Route path="/home" element={<Home/>}></Route>  
                <Route path="/about" element={<About/>}></Route>  
            </Routes>   
        </Routes>
    </BrowserRouter>
  )
  export default baseRouter
  ```

* `src/index.tsx`

  ```tsx
  import React from 'react'
  import ReactDOM from 'react-dom/client'
  import Router from '@/router/index'
  ReactDOM.createRoot(document.getElementById('root') as HTMLElement).render(
    <React.StrictMode>
       <Router /> 
    </React.StrictMode>
  )
  ```

* `src/App.tsx`

  > * `Outlet`：类似于 `<router-view />`
  > * `Link`：点击跳转

  ```tsx
  import { Outlet, Link } from 'react-router-dom'
  function App() {
    return (
      <div className="App">
         <Link to="/home">Home</Link>
         <Link to="/about">About</Link>
         <Outlet></Outlet>
      </div>
    )
  }
  
  export default App
  ```

  ```html
  <Link to={`/detail/${id}`}>detail</Link>
  ```
  
  <br/>

##  四：v6路由配置方式二

### 1、基本使用

  * `src/router/index.tsx`
  
    > * `lazy`：组件懒加载
    > * `Navigate`：重定向
    >
    > 组件懒加载需要被嵌套在`React.Suspense` 中
  
    ```tsx
    import React, { lazy } from 'react'
    import { Navigate } from  'react-router-dom'
    const Home = lazy(() => import('@/views/Home/index'))
    const About = lazy(() => import('@/views/About/index'))
    
    const withLoadingComponent = (component: JSX.Element) => (
      <React.Suspense fallback={<div>Loading...</div>}>
        {component}
      </React.Suspense>
    )
    
    const routes = [
       {
        path: '/',
        element: <Navigate to="/home" />
       },
       {
        path: '/home',
        element: withLoadingComponent(<Home />)
       },
       {
        path: '/about',
        element: withLoadingComponent(<About />)
       }
    ]
    export default routes
  ```
  
  * `src/index.tsx`
  
    > * `BrowserRouter`： History 模式
    > *  `HashRouter`：Hash 模式
  
    ```tsx
    import React from 'react'
    import ReactDOM from 'react-dom/client'
    import App from './App'
    import { BrowserRouter, HashRouter } from 'react-router-dom'
    
    ReactDOM.createRoot(document.getElementById('root') as HTMLElement).render(
      <React.StrictMode>
        <BrowserRouter>
           <App />
        </BrowserRouter>
      </React.StrictMode>
    )
    ```

  * `src/App.tsx`
  
    ```tsx
    import router from '@/router/index-new'
    import { useRoutes, Link } from 'react-router-dom'
    function App() {
      const outlet = useRoutes(router)
      return (
        <div className="App">
           <Link to="/home">Home</Link>
           <Link to="/about">About</Link>
           {outlet}
        </div>
      )
    }
    
    export default App
    ```


## 五、v6路由跳转及路由信息获取

```tsx
import { useParams, useLocation, useNavigate } from 'react-router-dom'
function App() {
    
    const params = useParams();
    // 当前路由状态
    const location = useLocation();
    const navigate = useNavigate();
    
    return (
        <div className="App">
            <button onClick={() => {navigate("/home")}}>Home</button>
            <button onClick={() => {navigate("/about")}}>About</button>
        </div>
    )
}
```

## 六：v5 与 v6 对比

* v6 全面倒向了函数式组件，废除了`useHistory()`，取而代之的是`useNavigate()`
* v5 将路由信息通过 props 的注入在页面中获取到路由信息
* v5 组件获取路由信息
  * 类组件或函数式组件：HOC高阶组件（withRouter）通过props来获取`history`，`location`， `match`
  * 函数式组件：hook函数`useHistory`， `useLocation`，`useParams`， `useRouteMatch`

> 钩子函数是没法在类式组件中使用的，故在v6中，如果页面或组件是类式的，无法通过钩子函数来获取路由信息及路由跳转方法

## 七：解决在 v6 中不支持类组件

> 在v6中，是没有 withRouter 函数。但可以利用该思想，将类式组件封装成高阶组件

* `src/helpers/withRoters.tsx`

  ```tsx
  import { useParams, useLocation, useNavigate } from 'react-router-dom'
  const withRouter = (component) => {
      const Wrapper = (props) => {
          return <component {...props} navigate={useNavigate()} />
      }
      // Wrapper 就是一个函数组件
      export Wrapper;
  }
  ```

* `src/view/Home/index.tsx`

  ```tsx
  import React from 'react'
  import { withRouter } from '@/helpers/withRouter.tsx'
  class HomeComponent extends React.Component {
      this.props.navigate(); // 获取路由信息
      render() {
          return (
            <div className="home"></div>
          )
      }
  }
  export const Home = withRouter(HomeComponent)
  ```

  