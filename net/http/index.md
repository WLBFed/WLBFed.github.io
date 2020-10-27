# http

15.cookie、localStorage、sessionStorage 区别和使用场景
【https://juejin.im/entry/6844903587764502536】
localStorage、sessionStorage
相同点 1、存储大小均为 5M 左右
2、都有同源策略限制
3、仅在客户端中保存，不参与和服务器的通信
区别

- localStorage: 存储的数据是永久性的，除非用户人为删除否则会一直存在。
- sessionStorage: 与存储数据的脚本所在的标签页的有效期是相同的。一旦窗口或者标签页被关闭，那么所有通过 sessionStorage 存储的数据也会被删除。
  Cookie 大小限制为 4KB 左右，主要用途是保存登录信息和标记用户(比如购物车)等，不过随着 localStorage 的出现，现在购物车的工作 Cookie 承担的较少了。

  16.跨域相关问题，怎么解决？几种方式？CORS 是跨站资源共享，因为浏览器的同源策略，限制了从一个域到另一个域进行资源的交互。
  同源：同域名、同协议、通端口

JSONP 和 CORS（一般服务器端设置 ACAO Access-Control-Allow-Origin）
都能解决 Ajax 直接请求普通文件存在的跨域无权限访问的问题。
JSONP 只支持 get，CORS 支持所有类型。
JSONP 主要是全浏览器支持，CORS 有一小部分老的浏览器不支持。
