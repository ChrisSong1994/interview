### axios 问题

- 1.axios 怎样设置全局参数？

- 2.axios 是怎样对 ajax 进行封装的？

- 3.axios 的中间件机制？

首先看下源码

```js
var chain = [dispatchRequest, undefined];
var promise = Promise.resolve(config);

this.interceptors.request.forEach(function unshiftRequestInterceptors(
  interceptor
) {
  chain.unshift(interceptor.fulfilled, interceptor.rejected);
});

this.interceptors.response.forEach(function pushResponseInterceptors(
  interceptor
) {
  chain.push(interceptor.fulfilled, interceptor.rejected);
});

while (chain.length) {
  promise = promise.then(chain.shift(), chain.shift());
}

return promise;
```

这块代码是在 `Axios.prototype.request` 内,这个方式是所有 get,put 等请求的方法，在里面维护了一个 chain 数组来接受注册的拦截器，其中我们可以看到，拦截器成对的被当成 promise 的 resolve 和 reject 引用，能够用 Axios.then() 方式调用，并且执行上面所有的.then 方法。

那么 axios 是怎样分开同是在 chain 调用栈上的请求拦截器和相应拦截器的呢？axios 巧妙的在请求和响应拦截器执行的中间位置放置了一个`dispatchRequest` 函数，其会根据前面拦截器返回的 config 去返回一个 `adapter` 执行函数

```js
return adapter(config).then(
  function onAdapterResolution(response) {
    throwIfCancellationRequested(config);

    // Transform response data
    response.data = transformData(
      response.data,
      response.headers,
      config.transformResponse
    );

    return response;
  },
  function onAdapterRejection(reason) {
    if (!isCancel(reason)) {
      throwIfCancellationRequested(config);

      // Transform response data
      if (reason && reason.response) {
        reason.response.data = transformData(
          reason.response.data,
          reason.response.headers,
          config.transformResponse
        );
      }
    }

    return Promise.reject(reason);
  }
);
```

`adapter`是 axios 封装的一个请求函数，分为 node 环境和浏览器环境

```js
function getDefaultAdapter() {
  var adapter;
  if (typeof XMLHttpRequest !== "undefined") {
    // For browsers use XHR adapter
    adapter = require("./adapters/xhr");
  } else if (typeof process !== "undefined") {
    // For node use HTTP adapter
    adapter = require("./adapters/http");
  }
  return adapter;
}
```

实际上 adapter 执行返回的也是一个 promise

```js
// http
module.exports = function httpAdapter(config) {
  return new Promise(function dispatchHttpRequest(resolve, reject) {
    //   ...
  }})


// xhr
module.exports = function xhrAdapter(config) {
  return new Promise(function dispatchXhrRequest(resolve, reject) {
    //   ...
  }})

```

总结：axios 的中间件机制和 redux 不同，axios 主要是围绕 promise 是实现的一个从配置到请求到响应的链式调用的过程
