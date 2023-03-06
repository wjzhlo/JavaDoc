## Cookie、Session、Token 对比

### Cookie

#### 概念

`cookie 是保存在客户端的数据，由服务器生成，发送给浏览器，浏览器将 cookie 以键值对方式存储。之后每一次发给同一服务器的请求都会带上 cookie。cookie 中一般存储着用户的标识、部分参数等不是很敏感的数据`



#### 作用

- 存储不是很敏感的数据
- 自动登录，客户端登录状态的保持



#### 生命周期

 cookie 随着浏览器的关闭而销毁，同时也可以给 cookie 设置过期时间，若 cookie=0或为负值，则关闭会话窗口即销毁



#### 特点

- 由于 cookie 是存储在浏览器中，根据浏览器的不同，对 cookie 的大小和数量都做了限制(50个，4095byte)
- cookie 可以跨域名访问，需要设置 domain
- 安全性低，客户端应用可以获取到本地的 cookie 并解析，因此不建议将敏感数据存储在 cookie 中



#### 若 Cookie 被禁用

服务端获取 Session 依赖于 Cookie 中携带的 JSESSIONID，若被禁用则服务器无法使用对应 session 了。

`可以使用重定向的方式追踪到 session，即获取到请求后，将请求重定向到需要跳转的页面去`



### Session

#### 概念

session 是服务器对客户端连接的唯一标识，由服务器生成，存储在服务器上。在客户端与服务器连接断开后会被销毁。Session 的安全性比 Cookie 更高，可以用于保存用户的重要数据



#### 特点

- 服务端可以通过 Http 请求内的 Session，找到这个唯一的客户端连接
- Session 的安全性比 Cookie 更高
- Session 无法跨域
- 由于 Session 是保存在服务器中的，若连接过多会占用服务器资源



### Token

token 即令牌，是用户验证身份的方式。在实际项目中一般使用自定义 token，并将 token 存储在 header 中，在每次 http 请求的时候都发送给服务器。

服务器会存储 Token ，可以避免重放攻击、签名验证以及参数传递。



#### token 刷新方案

如使用 jwt ，结合 Redis 可以实现两套方案

##### 双 token 方案

`双 token 即每次客户端登录都生成两个 token，一个用于验证请求，另一个用于刷新 token。同时 token 的刷新依赖于客户端主动调用刷新接口。其中需要使用 jwt 生成 token，并设置过期时间`

- 在客户端登录时，服务端使用 jwt 生成 access_token(请求验证) 和 frush_token(刷新token)，并在 redis 中记录用户id和对应的token，分别为：[uid，access_token]，[uid，frush_token]，并将 access_token 和 frush_token 在响应中返回给客户端
- 客户端在登录后获取到自己的 access_token 和 frush_token，并开启线程，在 access_token 即将过期时调用特定接口发送 frush_token，来刷新自己的 token
- 服务端刷新 token 的接口中，在收到客户端传来的 frush_token 后，应该去 redis 中获取这个用户的 id，并重新生成这两个 token，将其写入 Redis 中 [uid，access_token] 和 [uid，frush_token]，将新生成的 access_token 和 frush_token 返回给客户端
- 客户端之后的请求使用新的 access_token 请求



##### Redis 中心化方案

`Redis 中心化的方案即利用 Redis 可以给 key 设置过期时间的特点，将 [uid, token] 存储在 Redis 中，并设置过期时间。每次请求验证时，可以从 redis 中取出对应的 [uid, token] 对进行验证。同时可以获取到这个键值对的过期时间，若即将过期(如仅剩30秒)，则重置设置键值对的过期时间。`

Redis 中心化的方案对 token 格式的没有特定要求，可以自己生成，但必须是唯一的

- 客户端登录时，服务端为其生成唯一的 token，并在 redis 中记录对应关系 [token, uid]，并设置过期时间
- 每次拦截到客户端请求内容时，从 redis 中获取这个 token 对应的键值对，若获取到了则有效，若获取不到则是无效或过期的
- 若获取到了键值对，查看键值对的过期时间，若即将过期，则重新设置这个键值对的过期时间，以此实现类似刷新 token 的方式
- 为什么不每次都刷新？ => 避免频繁的写操作，降低 redis 性能



##### Token 永不过期方案

`Token 永不过期即将生成的 [uid，token] 永久存储在 Redis 中，只有每次用户登录时会刷新 token，以及用户调用登出接口时销毁对应的键值对，若用户异常退出，没有调用登出接口，则这条数据永远存储在 Redis 中了`

为了避免 Redis 占用内存过高，影响其他服务使用，需要设定 Redis 的可使用最大内存。同时也要修改 Redis 的淘汰策略为 `allkeys-lru`

- 客户端登录时，服务端为其生成唯一的 token，并在 redis 中记录对应的关系 [token, uid]，但不设置过期时间
- 每次拦截到客户顿请求内容时，都去 Redis 中验证这个 token
- 当客户端调用登出接口时，从 Redis 中移除这个键值对

