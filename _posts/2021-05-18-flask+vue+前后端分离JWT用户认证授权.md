---
layout: post
title: 前后端分离JWT用户认证授权及集成CAS单点登录
date: 2021-05-18
Author: Los
tags: [flask,vue,jwt]
comments: false
---

传统Flask通过Flask-Login的login_user()解决登录问题，通过session进行处理，不适合**前后端分离**系统，所以使用JWT进行用户认证

**Session-cookie:**

Session是对于服务端来说的，客户端是没有Session一说的。Session是服务器在和客户端建立连接时添加客户端连接标志，最终会在服务器软件(Apache、Tomcat、JBoss)转化为一个临时Cookie(SessionId)发送给给客户端，当客户端请求时服务器会检查是否携带了这个SessionId(临时Cookie)，如果没有则会要求重新登录。

***问题：***

1. 如果我们的页面出现了 **XSS 漏洞**，由于 cookie 可以被 JavaScript 读取，XSS 漏洞会导致用户SessionId泄露，而作为后端识别用户的标识，cookie 的泄露意味着用户信息不再安全。在设置 cookie 的时候，其实你还可以设置 httpOnly 以及 secure 项。设置 httpOnly 后 cookie 将不能被 JS 读取，浏览器会自动的把它加在请求的 header 当中，设置 secure 的话，cookie 就只允许通过 HTTPS 传输。
2. httpOnly 选项使得 JS 不能读取到 cookie，那么 XSS 注入的问题也基本不用担心了。但设置 httpOnly 就带来了另一个问题，就是很容易的被 **XSRF或CSRF(跨站请求伪造)**。当你浏览器开着这个页面的时候，另一个页面可以很容易的跨站请求这个页面的内容。因为 cookie 默认被发了出去。
3. 由于后端保存了所有用户的Session，后端每次都需要根据SessionId查出用户Session进行匹配，加大了**服务器端的压力**。

**JWT:**

JWT 是一个开放标准(RFC 7519)，它定义了一种用于简洁，自包含的用于通信双方之间以 JSON 对象的形式安全传递信息的方法。JWT 可以使用 HMAC 算法或者是 RSA 的公钥密钥对进行签名。它具备两个特点：

1. 简洁(Compact)

   可以通过URL, 参数或者在 HTTP header 发送，因为数据量小，传输速度快

2. 自包含(Self-contained)

   负载中包含了所有用户所需要的信息，避免了多次查询数据库

   **JWT一共由三部分组成，header（头部）、payload（负载）、signature（签名）**通过‘.’进行拼接

   

  ![2169646-20210118002911489-1547541084.jpg](https://i.loli.net/2021/05/18/auFwXsORvVoCb1f.jpg)

   **header(头部)** *转Base64*

   - `alg` 加密算法
   - `typ` 类型

   **payload(负载)** 自定义信息内容, 不建议存储敏感信息(如密码) *转Base64*

   - `iss` 签发者
   - `sub` 面向的用户
   - `aud` 接收jwt的一方
   - `exp` 过期时间(必须大于签发时间jat)
   - `nbf` 定义在什么时间之前，该jwt都是不可用的
   - `jat` 签发时间
   - `jti` 唯一身份标识，主要用来作为一次性token,从而回避重放攻击

   **signature(签名)** *一共三部分。转base64的header和转base64的payload拼接之后，然后使用header中声明的加密方式和secret加盐的方式加密字符串*

   - `转Base64的header`

   - `转Base64的payload`

   - `secret(私钥)`

     最后一步签名的过程，实际上是对头部以及负载内容进行签名，防止内容被窜改。如果有人对头部以及负载的内容解码之后进行修改，再进行编码，最后加上之前的签名组合形成新的JWT的话，那么服务器端会判断出新的头部和负载形成的签名和JWT附带上的签名是不一样的。如果要对新的头部和负载进行签名，在不知道服务器加密时用的密钥的话，得出来的签名也是不一样的。

- **差异比较**

Session方式存储用户id的最大弊病在于Session是存储在服务器端的，所以需要占用大量服务器内存，对于较大型应用而言可能还要保存许多的状态。一般而言，大型应用还需要借助一些KV数据库和一系列缓存机制来实现Session的存储。

JWT方式将用户状态分散到了客户端中，可以明显减轻服务端的内存压力。除了用户id之外，还可以存储其他的和用户相关的信息，例如该用户是否是管理员、用户所在的分组等。虽说JWT方式让服务器有一些计算压力（例如加密、编码和解码），但是这些压力相比磁盘存储而言可能就不算什么了。具体是否采用，需要在不同场景下用数据说话

- **单点登录**

Session方式来存储用户id，一开始用户的`Session`只会存储在一台服务器上。对于有多个子域名的站点，每个子域名至少会对应一台不同的服务器，例如：`www.taobao.com`，`nv.taobao.com`，`nz.taobao.com`，`login.taobao.com`。所以如果要实现在`login.taobao.com`登录后，在其他的子域名下依然可以取到`Session`，这要求我们在多台服务器上同步`Session`。使用`JWT`的方式则没有这个问题的存在，因为用户的状态已经被传送到了客户端。



**具体实现：Vue前端**

**Vue-router**

成功登录时，将后端返回的`jwt`存入`sessionStorage`

使用`Vue-router`在前端每次界面切换前都判断`jwt`，不符合要求则跳转至`login`登录界面

```js
router.beforeEach((to, from, next)=>{
  const accessToken = window.sessionStorage.getItem('accessToken')
  
  if(accessToken) 
  {
      // 重新登录后，转到之前的页面
      if(Object.keys(from.query).length !== 0)
      {
        let redirect = from.query.redirect
        if(to.path === redirect) // 解决无限循环问题
        {
          next()
        }
        else
        {
          next({path:redirect}) // 重新登录后，转到之前的页面
        }
      }
  }

  if(accessToken && to.path !== '/login')
  {
    // 有token 但不是去 login页面
    next()
  }
  else if(accessToken && to.path === '/login')
  {
    //用户已经登陆，不让访问Login登录界面
    next({path: from.fullPath})
  }
  else if(!accessToken && to.path !== '/login')
  {
    // 未登录
    next('/login')
  }
  else
  {
    next()
  }
})
```



**2.axios**

axios 全局配置拦截器

`request拦截器`每次向后端请求携带header头`Authorization`信息

```js
// http request 拦截器
axios.interceptors.request.use(
  config =>{
    if(sessionStorage.getItem("accessToken"))
    {
        config.headers.Authorization = sessionStorage.getItem("accessToken")
    }
    return config;
  },
  err => {
    return Promise.reject(err);
  }
)
```

`response拦截器`若接收到`403`错误，则是未登录，无权访问，则清除`sessionStorage`信息并跳转至`login`登录界面

```js
// http response 拦截器
axios.interceptors.response.use(
  response => {
    return response;
  },
  error => {
    if(error.response){
      console.log('axios:' + error.response.status);
      switch(error.response.status){
        case 403:
          // 返回403 清除token信息并跳转到登录页面
          sessionStorage.clear() 
          router.replace({
            path: '/login',
            query: {redirect: router.currentRoute.fullPath}   // 重新登录后，返回之前的页面
          })
          Message({showClose:true, message:'未登录，返回登陆界面', type:'error', duration:3000})  
   
      }
    }
    return Promise.reject(error);   // 返回接口的错误信息
  }
)
```

![_flask_vue_sso_jwt.png](https://i.loli.net/2021/05/18/jzNX5Y6pingW9Ro.png)

[参考](https://www.cnblogs.com/leon7/p/14287566.html)

