发起一个ajax请求时，request header 里面有三个属性会涉及请求源信息。前端可能用不到这些值，但是，后台业务系统会比较关心它们，场景可能有：
- 处理跨域请求时，必须判断来源请求方是否合法；
- 后台做重定向时，需要原地址信息；

作为前端，了解三者的区别和使用场景，还是有很意义的。

[先看图：](https://upload-images.jianshu.io/upload_images/25750-05d5812b7dc7f995.png?imageMogr2/auto-orient/strip|imageView2/2/w/1054/format/webp)

#### 1. Host
- 描述请求将被发送的==目的地==，包括，且仅仅包括==域名和端口号==。
- 在任何类型请求中，request都会包含此header信息。

#### 2. Origin
- 用来说明请求==从哪里发起==的，包括，且仅仅包括==协议和域名==。
- 这个参数一般只存在于 ==CORS== 跨域请求中，可以看到response有对应的header：Access-Control-Allow-Origin。

#### 3. Referer
- 告知服务器==请求的原始资源的URI==，其用于所有类型的请求，
- 并且包括：协议+域名+查询参数（注意，不包含锚点信息）。
- 因为原始的URI中的查询参数可能包含ID或密码等敏感信息，如果写入referer，则可能导致信息泄露。




