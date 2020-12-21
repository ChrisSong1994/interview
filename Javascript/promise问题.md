### promise 问题

- 1.promise 内部报错在 try catch 里面能捕获?
  try catch 只能朴获同步操作例如 async/await 不能捕获在 promise 中的异步操作
