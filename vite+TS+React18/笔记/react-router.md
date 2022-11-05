页面 Home

```tsx
const Home = () => {
    return (
        <div>Home 页面</div>
    )
}
export default Home
```

页面 About

```tsx
const About = () => {
    return (
        <div>About 页面</div>
    )
}
export default About
```

<br/>

### 旧版本路由配置

* `src/router/index.tsx`

  > * `BrowserRouter`： History 模式
  > *  `HashRouter`：Hash 模式
  > * `Navigate`：路由重定向

  ```tsx
  import App from '@/App'
  import Home from '@/views/Home/index'
  import About from '@/views/About/index'
  
  import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom'
  
  const baseRouter = () => (
    <BrowserRouter>
        <Routes>
            <Route path="/" element={<App/>}>
                <Route path="/" element={<Navigate to="/home"/>}></Route> 
                <Route path="/home" element={<Home/>}></Route>  
                <Route path="/about" element={<About/>}></Route>  
            </Route>   
        </Routes>
    </BrowserRouter>
  )
  export default baseRouter
  ```

* `src/main.tsx`

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

  <br/>

  

###  新版本路由配置

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

  * `src/main.tsx`
  
    > * `BrowserRouter`： History 模式
    > *  `HashRouter`：Hash 模式
  
    ```tsx
    import React from 'react'
    import ReactDOM from 'react-dom/client'
    import App from './App'
    import { BrowserRouter } from 'react-router-dom'
    
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
  
    