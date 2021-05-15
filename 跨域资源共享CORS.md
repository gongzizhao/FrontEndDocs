### 什么是跨域？

跨域是指跨域名的访问。如果**域名和端口都相同，但是请求路径不同**，不属于跨域，如：

```
www.jd.com/item
www.jd.com/goods
```

当一个资源请求一个其它域名的资源时会发起一个跨域HTTP请求(cross-origin HTTP request)。比如说，域名A (http://domaina.example) 的某 Web 应用通过<img>标签引入了域名B (http://domainb.foo) 的某图片资源 (http://domainb.foo/image.jpg) ，域名A的 Web 应用就会导致浏览器发起一个跨域 HTTP 请求。在当今的 Web 开发中，使用跨域 HTTP 请求加载各类资源（包括CSS、图片、JavaScript 脚本以及其它类资源），已经成为了一种普遍且流行的方式。

### 为什么有跨域问题？

跨域不一定会有跨域问题。

因为跨域问题是浏览器对于ajax请求的一种安全限制：**一个页面发起的ajax请求，只能是于当前页同域名的路径**，这能有效的阻止跨站攻击。

因此：**跨域问题是针对ajax的一种限制**。

### 跨域资源共享Cross-Origin Resource Sharing (CORS)

通过客户端+服务端协作声明的方式来确保请求安全的。服务端会在HTTP请求头中增加一系列HTTP请求参数(例如Access-Control-Allow-Origin等)，来限制哪些域的请求和哪些请求类型可以接受，而客户端在发起请求时必须声明自己的源(Orgin)，否则服务器将不予处理，如果客户端不作声明，请求甚至会被浏览器直接拦截都到不了服务端。服务端收到HTTP请求后会进行域的比较，只有同域的请求才会处理。

#### 1.什么是CORS

CORS是一个W3C标准，全称是"跨域资源共享"（Cross-origin resource sharing）。

它允许浏览器向跨源服务器，发出[`XMLHttpRequest`](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2012%2F09%2Fxmlhttprequest_level_2.html)请求，从而克服了AJAX只能[同源](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2016%2F04%2Fsame-origin-policy.html)使用的限制。

CORS需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。

- 浏览器端：

  目前，所有浏览器都支持该功能（IE10以下不行）。整个CORS通信过程，都是浏览器自动完成，不需要用户参与。

- 服务端：

  CORS通信与AJAX没有任何差别，因此你不需要改变以前的业务逻辑。只不过，浏览器会在请求中携带一些头信息，我们需要以此判断是否运行其跨域，然后在响应头中加入一些信息即可。这一般通过过滤器完成即可。

#### 2.一个使用CORS实现跨域请求的示例：

客户端：

```
function getHello() {
    var xhr = new XMLHttpRequest();
    xhr.open("post", "http://b.example.com/Test.ashx", true);
    xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");　　　
    // 声明请求源
    xhr.setRequestHeader("Origin", "http://a.example.com");
    xhr.onreadystatechange = function () {
        if (xhr.readyState == 4 && xhr.status == 200) {
            var responseText = xhr.responseText;
            console.info(responseText);
        }
    }
    xhr.send();
}
```

服务端：

```
public class Test : IHttpHandler
   { 
       public void ProcessRequest(HttpContext context)
       {
           context.Response.ContentType = "text/plain";
           // 声明接受所有域的请求
           context.Response.AddHeader("Access-Control-Allow-Origin", "*");
           context.Response.Write("Hello World");
       }
 
       public bool IsReusable
       {
           get
           {
               return false;
           }
       }
   }
```

#### 2.基本流程

1. 浏览器将请求分为两类，一类是简单请求，一类是非简单请求，满足下列条件的就是简单请求，否则即使非简单请求：

```
条件：
    1、请求方式：HEAD、GET、POST
    2、请求头信息：
        Accept
        Accept-Language
        Content-Language
        Last-Event-ID
        Content-Type 对应的值是以下三个中的任意一个
                                application/x-www-form-urlencoded
                                multipart/form-data
                                text/plain
 
注意：同时满足以上两个条件时，则是简单请求，否则为复杂请求
```

2. 简单请求和复杂请求的区别：

简单请求：一次请求

非简单请求：两次请求，在发送数据之前会先发第一次请求做预检，只有预检通过后再发一次请求作为数据传输。

3. 关于预检：

```
- 请求方式：OPTIONS
- “预检”其实做检查，检查如果通过则允许传输数据，检查不通过则不再发送真正想要发送的消息
- 如何“预检”
     => 如果复杂请求是PUT等请求，则服务端需要设置允许某请求，否则“预检”不通过
        Access-Control-Request-Method
     => 如果复杂请求设置了请求头，则服务端需要设置允许某请求头，否则“预检”不通过
        Access-Control-Request-Headers
```

4. CORS优缺点

　　CORS的优点：可以发任意请求

　　CORS的缺点：上是复杂请求的时候得先做一个预检，再发真实的请求，发了两次请求会有性能上的损耗。
