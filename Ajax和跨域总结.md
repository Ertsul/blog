## Ajax（JavaScript实现）
#### 简介
Ajax（Async Javascript And Xml）：异步Js和Xml，意思就是用Js执行异步网络请求。作用主要是在不重新加载**全部网页**的情况下，对**部分页面**进行更新。

#### 创建XMLHttpRequest对象
除了IE5和IE6不支持之外（支持ActiveXObject对象），几乎现在所有的有内建的XMLHttpRequest对象。所以在创建请求对象之前判断下就好了：

````
let xhr = null;
xhr = window.XMLHttpRequest ? new xmlHttpRequest() : new ActiveXObject("Microsoft.XMLHTTP")
// 或者
if(window.XMLHttpRequest){
    xhr = new XMLHttpRequest()
}else{
    xhr = new ActiveXObject('Microsoft.XMLHTTP')
}
````

#### 通过open()和send()向服务器发送请求
- xhr.open(method, url, async)
这个方法的参数分别是：method请求的类型GET或者POST；URL请求的文件在服务器上的地址；async是否异步请求true或者false（一般不设置为false）。
- xhr.send()：发送请求。
- xhr.setRequestHeader(header, value)：如果是POST的话，要设置发送的HTTP头，然后通过send方法发送内容。参数分别是：发送的头部名称，头部的值。一般都是固定的。

````
// 向浏览器发送请求
xhr.open('GET', './server.php', true)
// 添加HTTP头（POST方法）
xhr.setRequestHeader('Content-Text', 'application/x-www-form-urlencoded')   
xhr.send()
````

GET方法不需要参数，POST方法则需要把body参数；如果要想向服务器发送相关的参数，则通过open里面的第二个参数url进行发送，具体实现如下代码：

````
url = "./filePath?paramName=" + paramValue + ...;
````

#### 判断准备状态

````
xhr.onreadystatechange = function () {
    if (xhr.readyState == 4 && xhr.status == 200) {
        ...
    }
}
````
- xhr.onreadystatechange方法中，每当readyState改变时，就会触发xhr.onreadystatechange事件。
- readyState属性：存有XMLHttpRequest的状态信息。0：请求未初始化；1：服务器链接已经建立；2：请求已经接受；3：请求处理中；4：请求已完成，且响应已就绪。一共有5个状态，所以onreadystatechange事件会被触发5次。
- 实际上onreadystateschange是通过回调函数实现的，即：每当readyState的状态改变时，这个回调函数就会被执行。
- status属性：表示响应的结果。

总的代码如下：
````
document.getElementById('btn').onclick = function () {
// 创建请求对象（先判断浏览器类型）
// var xhr = null;
// if (window.XMLHttpRequest) {
//     xhr = new XMLHttpRequest()
// } else {
//     xhr = new ActiveXObject('Microsoft.XMLHTTP')
// }
xhr = (window.XMLHttpRequest) ? (new XMLHttpRequest()) : (new ActiveXObject('Microsoft.XMLHTTP'))
// 判断响应状态并执行相关的操作
xhr.onreadystatechange = function () {
    if (xhr.readyState == 4 && xhr.status == 200) {
        let data = xhr.responseText
        console.log(data);
        document.getElementById('myDiv').innerText = data
    }
}
// 向浏览器发送请求
let url = "./server.txt"
xhr.open('GET', url, true)
// 添加HTTP头（POST方法）
// xhr.setRequestHeader('Content-Text', 'application/x-www-form-urlencoded')
xhr.send(null);
}
````
![异步1.gif](http://upload-images.jianshu.io/upload_images/659084-b5b344a3944ba224.gif?imageMogr2/auto-orient/strip)

但是，由于在**同源策略**的作用下，浏览器只能同源的其他域的资源，先看看什么是同源策略～

## 同源策略/SOP（Same origin policy）
来自百度的解释：
> 同源策略（Same origin policy）是一种约定，它是浏览器最核心也最基本的安全功能（防止CSRF/XSRF攻击：跨站请求伪造）。现在所有支持JavaScript 的浏览器都会使用这个策略。所谓同源是指，域名，协议，端口相同。

简单来说，就是浏览器只能请求同源的其他域的资源，即同源的网页才能相互获取资源，而不能访问其他域的资源；但是有一个例外，同源策略不会阻止动态脚本插入到文档中；其中，**<script>\</script>**是开放策略；当web网页执行一个脚本之前，会检查是否同源，只有同源才会被执行，非同源则会被拒绝访问。
截取网上的图片说明同源：
![image.png](http://upload-images.jianshu.io/upload_images/659084-80411ea5e317ed19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/400)

## 跨域
跨域就是突破同源策略的限制，获取其他源的资源。
非同源主要有三种行为会受到限制：
- Cookie,LocalStorage和IndexDB无法读取。
- DOM无法获取。
- Ajax无法获取。

#### JSONP
JSONP是实现跨域的常用方法，但是JSONP只能实现GET方法。实际上是是利用了浏览器允许跨域引用JavaScript资源（script标签是开放策略）。

下面举一个例子：我们从360浏览器首页的天气查询获取天气查询的异步请求地址；这里我们实现通过本地localhost服务器跨域请求360的天气查询，来说明JSONP怎么实现跨域。
我们请求资源的地址为：
https://cdn.weather.hao.360.cn/sed_api_weather_info.php?code=101280501&v=2&param=weather&app=hao360&_jsonp=__jsonp21__&t=2521600
在浏览器中打开这个地址，我们会得到这样的数据：\_\_jsonp21\_\_({...})。
解释：JSONP返回的通常是以函数的形式返回，前面这个\_\_jsonp21\_\_是函数名，所以我们需要在我们的代码中事先准备好名称为\_\_jsonp21\_\_的函数，我们对跨域请求到的数据就是放在这个函数中的，然后动态加载一个**script**节点，相当于动态读取外域的JavaScript资源，最后就等着接收回调了。

具体代码如下：

````
 // 请求的地址为：https://cdn.weather.hao.360.cn/sed_api_weather_info.php?code=101280501&v=2&param=weather&app=hao360&_jsonp=__jsonp21__&t=2521600

function __jsonp21__(data) {   // 将要处理了的数据放在这个函数中
    console.log(data);  // 得到的是跨域请求的数据
    let myDiv = document.getElementById('myDiv')  
    myDiv.innerHTML = data.area
}
window.onload = function () { 
    let btn = document.getElementById('btn')
    btn.addEventListener('click', function () {  
        let url = 'https://cdn.weather.hao.360.cn/sed_api_weather_info.php?code=101280501&v=2&param=weather&app=hao360&_jsonp=__jsonp21__&t=2521600'
        let scriptTag = document.createElement('script')    // 动态创建script标签 
        scriptTag.setAttribute('src', url)      // 将其他源的地址设置为动态script的src属性
        // console.log(scriptTag);  // 得到的是script标签
        document.getElementsByTagName('body')[0].appendChild(scriptTag)  // 将动态创建的script标签添加到html页面中
    })
}
````
注意：数据处理的函数名要跟跨域返回的函数名一样，这样我们通过JSONP请求数据的时候，服务器才能返回数据。
最终结果如下：
![跨域1.gif](http://upload-images.jianshu.io/upload_images/659084-df752a8458c616ff.gif?imageMogr2/auto-orient/strip)

再句一个例子：
请求地址是http://api.money.126.net/data/feed/0000001,1399001?callback=refreshPrice

````
function refreshPrice(data) { 
console.log(data);
}
window.onload = function () {  
    let scriptTag = document.createElement('script')
    let url = 'http://api.money.126.net/data/feed/0000001,1399001?callback=refreshPrice'
    scriptTag.setAttribute('src', url)
    document.getElementsByTagName('body')[0].appendChild(scriptTag)
}
````

## Ajax（JQuery实现）
在JQuery中，实现Ajax主要是由**$.ajax({...})**方法实现。方法的常用参数有：
- async：是否实现异步加载，一般来说，是true。
- type：GET或者POST。
- url：发送请求的地址。
- timeout：设置请求的超时时间。
- success：请求成功后的回调函数。
- error：请求失败后的回调函数。
- jsonp：在一个JSONP请求中重写回调函数的名字，默认为callback，用来重新命名回调我们跨域请求时候的函数名字。这个值用来替代在 "callback=?" 这种 GET 或 POST 请求中 URL 参数里的 "callback" 部分，比如 {jsonp:'onJsonPLoad'} 会导致将 "onJsonPLoad=?" 传给服务器。
- jsonpCallback：为JSONP请求指定一个回调函数名。 
- dataType：预期服务器返回的数据类型。不指定的话，JQueryhi自动判断。"xml": 返回 XML 文档，可用jQuery处理。

>"html": 返回纯文本 HTML 信息；包含的 script 标签会在插入 dom 时执行；"script": 返回纯文本JavaScript代码。不会自动缓存结果。除非设置了"cache"参数。注意：在远程请求时(不在同一个域下)，所有POST请求都将转为GET请求。（因为将使用DOM的 script标签来加载）；"json": 返回JSON数据；"jsonp": JSONP格式。使用 JSONP 形式调用函数时，如 "myurl?callback=?" jQuery 将自动替换 ? 为正确的函数名，以执行回调函数；"text": 返回纯文本字符串。

用JQuery实现同源的Ajax请求：

````
$(function () {  
    $('button').on('click', function () {  
        $.ajax({
            url: './server.txt',    // 请求地址
            type: 'GET',    // 请求方式
            async: true,    // 是否异步
            success: function (data) {  
                console.log(data);
            }   // 请求成功后执行的函数
        })
    })
})
````

实现结果如下：
![异步2.gif](http://upload-images.jianshu.io/upload_images/659084-b1fcb9ed574a8a24.gif?imageMogr2/auto-orient/strip)

用JQuery实现非同源的JSONP跨域请求：

````
$(function () {  
$('button').on('click', function () {  
    $.ajax({
        url: 'https://cdn.weather.hao.360.cn/sed_api_weather_info.php?code=101280501&v=2&param=weather&app=hao360&_jsonp=__jsonp21__&t=2521600',
        type: 'GET',
        async: true,
        dataType: 'jsonp',
        jsonp: '_jsonp',
        jsonpCallback: '__jsonp21__',
        success: function (data) {  
            console.log(data);
            $('#myDiv').html(data.area)
        },
        error: function (err) {  
            console.log(err);
        }
    })
})
})
````

实现结果如下：
![跨域2.gif](http://upload-images.jianshu.io/upload_images/659084-14e8fc745c21ab00.gif?imageMogr2/auto-orient/strip)
