### cookie和localSrorage、session、indexDB 的区别



#### 区别

| 特性    | cookie          | localStorage | sessionStorage | indexDB                    |
| ----- | --------------- | ------------ | -------------- | -------------------------- |
| 生命周期  | 由服务器生成，可以设置过期时间 | 除非被清理，否则一直存在 | 页面关闭即清理        | 除非被清理，否则一直存在               |
| 存储大小  | 4K              | 5M           | 5M             | 无限                         |
| 服务端通信 | 每次都携带在header中   | 不参与          | 不参与            | 不参与                        |
| 适用    | 鉴权，不建议用于存储数据    | 变动少的数据存储     | 变动大的数据存储       | 存储大量的结构化数据，如文件/二进制大型对象blob |

#### cookie的安全性

- `value`用于保存登录态，应加密

- `http-only`不能通过js访问Cookie，减少XSS攻击

- `secure`只能在协议为HTTPS的请求中携带

- `same-site`规定浏览器不能在跨域请求中携带Cookie，减少CSRF攻击

#### indexDB的特性

- 使用索引实现对数据的高性能搜索，是基于 JavaScript 的面向对象数据库

- 操作是异步执行的

-  常用库`Dexie.js`


