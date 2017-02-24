### Ajax

​	ajax是使用脚本操纵http和web服务器进行数据交换，并且不会导致页面重载；comet技术与ajax相反，ajax是客户端向服务器请求数据，而comet是服务器推送数据到客户端；

​	实现ajax的方式有很多种，例如：img标签的src属性当设置后，会发送一个http的get请求；iframe标签的src属性，iframe比img更加强大一些，可以接受src加载后返回的数据，并加载到页面中，我们可以通过遍历iframe文档来处理响应内容；script标签的src可以发起http的get请求，并且script是可以跨域通信，当执行脚本时，服务器返回json格式的响应，js能够自动将其解码，这种方式也叫做“jsonp”；另外还有更简单的方式为XMLHttpRequest；

​	comet传输协议需要客户端和服务器之间建立一个保持开启的连接，当服务器发送消息到客户端后，将这个连接关闭，客户端处理该消息之后，客户端马上为后续消息建立一个新的连接；

#### 使用XMLHttpRequest

##### 1、创建xhr对象

var request = new XMLHttpRequest();

在ie7之前不支持上面直接调用构造函数的方式来创建xhr对象，而是需要使用一个ActiveX对象，如下方式模拟：

```javascript
if(window.XMLHttpRequest === undefined){
  window.XMLHttpRequest = function (){
    try{
    	return new ActiveXObject("Msxml2.XMLHTTP.6.0");
    }catch (e1){
    	return new ActiveXObject("Msxml2.XMLHTTP.3.0");
    }catch (e2){
    	throw new Error("not support");
    }
  }
}
```

##### 2、设置请求

请求包括请求的方法、请求的url、请求头、请求的主体

request.open( method , url , async );   //参数分别为请求方法、url、是否异步（默认true）

request.setRequestHeader( "Content-Type" , "text/plain" );  //设置请求头

request.send();   //get请求没有主体，不写或者为null；post请求通常拥有主体 

一些请求头是xhr对象自动添加的，用来防止伪造，例如：cookie、Date、User-agent等；

##### 3、取得响应

http响应由状态码、响应头集合、响应主体组成；

1、status和statusText属性以数字和文本的形式返回http状态码；

2、使用getResPonseHeader()和getAllResponseHeaders()能查询响应头，后者会自动过滤掉cookie头；

3、响应主体可以从responseText属性中得到文本形式响应内容，从responseXML得到Document形式内容；

4、监测请求的状态

xhr对象由readyState属性用来表示请求的状态，它是一个整数：0代表open()尚未调用，1代表open()已调用，2代表接收到头信息，3代表接收到响应主体，4代表响应完成；而readyState属性的每次改变都会触发xhr上的readystatechange事件，所以我们需要定义xhr的readystatechange事件来监听readyState的值；

#### 取消请求

可以通过xhr的abort()方法来终止http请求，xhr2中定义了timeout属性来制定请求自动终止的毫秒数（多数浏览器不支持此属性），我们可以通过setTimeout()方法和abort()方法实现超时自动取消；

#### 跨域http请求

因为同源策略，浏览器不允许脚本查找跨域文档的内容，我们可以通过集中方式实现跨域请求；

1. xhr2设置CORS（Cross-Origin Resource Sharing，跨域资源共享）允许跨域访问网站，ie8通过XDomainRequest对象支持它；通过测试xhr的withCredentials属性可以查看浏览器是否支持cors方式；

2. 使用script标签（jsonp）:使用jsonp方式发送请求时，第三方脚本必须是我们可信任的，攻击者可以通过攻击第三方服务器来接管你的网页，从而运行他的代码，这是很危险的；对于可信任的jsonp，其响应内容是用js函数名和圆括号包裹起来的，例如：handleResponse ( response_data )，当返回后，这个函数就会成为script标签的内容，然后自动去执行；这个回调函数的名字，我们可以通过发送名为jsonp的参数来指定，或者还有一个常见的callback参数；

   ```javascript
   function jsonpCallback(result) {    
     for(var i in result) {  
       console.log(i+":"+result[i]);
     }  
   }  
   var JSONP=document.createElement("script");  
   JSONP.type="text/javascript";  
   JSONP.src="data.js?callback=jsonpCallback";  
   document.getElementsByTagName("head")[0].appendChild(JSONP); 
   ```

   ```javascript
   //data.js
   jsonpCallback({    //只示意返回的数据格式，返回的数据应为callback(data)的形式
   	a:1,
   	b:2,
   	c:3,
   	d:4,
   	e:5
   })
   ```

#### comet技术

js定义了一个EventSource对象，我们可以传递给其构造函数一个URL，然后在返回的实例上定义监听消息事件：

```javascript
var ticker = new EventSource( server_api );
ticker.onmessage = function(e){
  var type = e.type;
  var data = e.data;
}

```

其他方式可以使用web socket和xhr长连接的方式