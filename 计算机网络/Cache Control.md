# Cache Control

https://www.cnblogs.com/skynet/archive/2012/11/28/2792503.html

## 名词

Shared cache：存在于源服务器和客户端之间的缓存（例如代理，CDN），它存储单个响应并且和多个用户重用-因为开发人员应该避免将个性化内容缓存在共享缓存中。

Private cache：客户端中存在的缓存，也成为本地缓存或者浏览器缓存，服务于单个用户。

## 响应指令

| 响应指令                                                    | 含义                                                         |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| Cache-Control：max-age=604800                               | 指示响应在生成响应后*N*秒之前保持新鲜，可用于后续请求<br />注意，不是收到响应以来经过的时间，而是自从在源服务器上生成响应以来经过的时间。 |
| Cache-Control: s-maxage=604800                              | 特定于共享缓存，当max-age存在时它被忽略                      |
| Cache-Control: no-cache                                     | 指示响应可以存储在缓存中，但**响应必须在每次重用之前与源服务器进行验证**，即使缓存与源服务器断开连接也是如此。<br />请注意，这`no-cache`并不意味着“不缓存”。`no-cache`允许缓存存储响应，但要求它们在重用之前重新验证它。如果您想要的“不缓存”实际上是“不存储”，那么`no-store`就是要使用的指令。 |
| Cache-Control: max-age=604800, must-revalidate              | 示响应可以存储在缓存中，并且可以在[新鲜](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching#fresh_and_stale_based_on_age)时重复使用。**如果响应变得[陈旧](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching#fresh_and_stale_based_on_age)，则必须在重新使用之前使用源服务器对其进行验证。**<br />通常，`must-revalidate`与`max-age` 一起使用。<br />HTTP 允许缓存在与源服务器断开连接时重用[过时的响应。](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching#fresh_and_stale_based_on_age)`must-revalidate`是一种防止这种情况发生的方法 - 存储的响应通过原始服务器重新验证或生成 504（网关超时）响应。 |
| no-store                                                    | 不缓存，不存储                                               |
| private                                                     | 指示响应只能存储在私有缓存中（例如浏览器中的本地缓存）。     |
| public                                                      | response指令`public`指示响应可以存储在共享缓存中。对带有头字段的请求的响应`Authorization`不得存储在共享缓存中；但是，该`public`指令将使此类响应存储在共享缓存中。 |
| immutable（不会改变的）                                     | response指令`immutable`指示响应在新鲜时不会[更新](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching#fresh_and_stale_based_on_age)。<br />Cache-Control: public, max-age=604800, immutable |
| Cache-Control: max-age=604800, stale-while-revalidate=86400 | response指令`stale-while-revalidate`指示缓存可以重用过时的响应，同时将其重新验证到缓存。<br />在上面的例子中，响应是[新鲜](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching#fresh_and_stale_based_on_age)的 7 天 (604800s)。7 天后它变得[陈旧](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching#fresh_and_stale_based_on_age)，但允许缓存将它重新用于第二天（86400 秒）发出的任何请求，前提是它们在后台重新验证响应。 |
| stale-if-error                                              | response指令`stale-if-error`指示缓存可以在上游服务器生成错误时或在本地生成错误时重用[陈旧的响应。](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching#fresh_and_stale_based_on_age)此处，错误被视为状态代码为 500、502、503 或 504 的任何响应 |
|                                                             |                                                              |
|                                                             |                                                              |

## Last-Modified/If-Modified-Since

Last-Modified/If-Modified-Since要配合Cache-Control使用。

l Last-Modified：标示这个响应资源的最后修改时间。web服务器在响应请求时，告诉浏览器资源的最后修改时间。

l If-Modified-Since：当资源过期时（使用Cache-Control标识的max-age），发现资源具有Last-Modified声明，则再次向web服务器请求时带上头 **If-Modified-Since**，表示请求时间。**web****服务器收到请求后发现有头If-Modified-Since** **则与被请求资源的最后修改时间进行比对**。若最后修改时间较新，说明资源又被改动过，则响应整片资源内容（写在响应消息包体内），HTTP 200；若最后修改时间较旧，说明资源无新修改，则响应HTTP 304 (无需包体，节省浏览)，告知浏览器继续使用所保存的cache。

## Etag/If-None-Match

Etag/If-None-Match也要配合Cache-Control使用。

l Etag：web服务器响应请求时，告诉浏览器当前资源在服务器的唯一标识（生成规则由服务器觉得）。*Apache**中，ETag**的值，默认是对文件的索引节（INode**），大小（Size**）和最后修改时间（MTime**）进行Hash**后得到的*。

l If-None-Match：当资源过期时（使用Cache-Control标识的max-age），发现资源具有Etage声明，则再次向web服务器请求时带上头If-None-Match **（Etag****的值）**。**web****服务器收到请求后发现有头If-None-Match** **则与被请求资源的相应校验串进行比对，决定返回200****或304**。



# 既生Last-Modified何生Etag？

你可能会觉得使用Last-Modified已经足以让浏览器知道本地的缓存副本是否足够新，为什么还需要Etag（实体标识）呢？HTTP1.1中Etag的出现主要是为了解决几个Last-Modified比较难解决的问题：

l Last-Modified标注的最后修改只能精确到秒级，如果某些文件在1秒钟以内，被修改多次的话，它将不能准确标注文件的修改时间

l 如果某些文件会被定期生成，当有时内容并没有任何变化，但Last-Modified却改变了，导致文件没法使用缓存

l 有可能存在服务器没有准确获取文件修改时间，或者与代理服务器时间不一致等情形

Etag是服务器自动生成或者由开发者生成的对应资源在服务器端的唯一标识符，能够更加准确的控制缓存。**Last-Modified****与ETag****是可以一起使用的，服务器会优先验证ETag****，一致的情况下，才会继续比对Last-Modified****，最后才决定是否返回304**。
