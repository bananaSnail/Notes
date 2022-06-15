### ajax
- jquery ajax
```js
$.ajax({
   type: 'POST',
   url: url,
   data: data,
   dataType: dataType,
   success: function () {},
   error: function () {}
});
```
- 优缺点：
    - 本身是针对MVC的编程,不符合现在前端MVVM的浪潮。
    - 基于原生的XHR开发，XHR本身的架构不清晰，已经有了fetch的替代方案。
    - JQuery整个项目太大，单纯使用ajax却要引入整个JQuery非常的不合理（采取个性化打包的方案又不能享受CDN服务）。

### axios
- axios 基于promise用于浏览器和node.js的http客户端
- 优缺点：
    - 从 node.js 创建 http 请求。
    - 支持 Promise API。
    - 提供了一些并发请求的接口（重要，方便了很多的操作）。
    - 在浏览器中创建 XMLHttpRequests。
    - 在 node.js 则创建 http 请求。（自动性强）
    - 支持 Promise API。
    - 支持拦截请求和响应：通过 axios.interceptors.request 和 axios.interceptors.response 对象提供的 use 方法，就可以分别设置请求拦截器和响应拦截器。interceptors是 InterceptorManager 对象
    - 转换请求和响应数据。
    - 取消请求。
        - 用XMLHttpRequests.abort()取消请求
        ```js
        if (config.cancelToken) {
            config.cancelToken.promise.then(function onCanceled(cancel) {
                if (!request) {
                return;
                }

                request.abort();
                reject(cancel);
                // Clean up request
                request = null;
            });
        }

        ```
    - 自动转换 JSON 数据。
    - 客户端支持防止CSRF。
    - 客户端支持防御 XSRF。

```js
// 添加请求拦截器
const myRequestInterceptor = axios.interceptors.request.use(config => {
    // 在发送http请求之前对config做点什么
    return config; // 必须返回config否则后续http请求无法获取到config
}, error => {
    // 错误处理代码写在这里
    
    // 但是，为什么返回Promise.reject(error)
    // 这和promise的机制有关
    // 如果直接抛出error就相当于抛出一个对象，
    // 这就会运行下一级promise的fulfilled方法。并且参数是error，这个fulfilled参数应该是config才对
    // 或者运行到dispatchRequest，参数error，但dispatchRequest参数也要是config才对
    return Promise.reject(error);
});

// 添加响应拦截器
axios.interceptors.response.use(response => {
  // 对响应数据处理代码写这里
  return response; // 有且必须有一个response对象被返回
}, error => {
  // 对响应错误处理代码写这里
  
  // 同理请求拦截器，这需要返回Promise.reject(error);
  return Promise.reject(error);
});

// 移除某次拦截器
axios.interceptors.request.eject(myRequestInterceptor);

```
- 数据修改器和拦截器的区别：拦截器侧重整体配置的修改。而数据修改器，侧重于对操作的数据进行修改。数据修改器和应用程序的耦合度更高

### fetch

### axios和fetch区别，axios为什么可以取消请求