Ajax步骤:

* 创建Ajax对象  `XMLHttpRequest`
* 为Ajax对象绑定监听
* 绑定提交地址
* 发送请求
* 接受响应数据

```js
//创建对象
var xmlhttp = new XMLHttpRequest()
//绑定监听
xmlhttp.onreadystatechange = function(){
    //接受响应
    if (xmlhttp.readyState==4 && xmlhttp.status==200){
        var res = xmlhttp.responseText;
    }
}
//绑定地址
//method：请求的类型；GET 或 POST
//url：文件在服务器上的位置
//async：true（异步）或 false（同步）
xmlhttp.open('GET',url,true)

//发送请求 send(string)
//string：仅用于 POST 请求
xmlhttp.send()
```

POST 请求，需要在发送之前,使用 setRequestHeader() 来添加 HTTP 头 

```
xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");
```



 XMLHttpRequest 对象的三个重要的属性：

| 属性               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| onreadystatechange | 存储函数（或函数名），每当 readyState 属性改变时，就会调用该函数。 |
| readyState         | 存有 XMLHttpRequest 的状态。从 0 到 4 发生变化 . `0: 请求未初始化`  `1: 服务器连接已建立` `2: 请求已接收` `3: 请求处理中` `4: 请求已完成，且响应已就绪` |
| status             | 200: "OK" 404: 未找到页面                                    |



jquery get post请求中文乱码

get:中文编解码 new String(parameter.getBytes("iso8859-1"),"UTF-8")

post:jquery已经帮忙编解码,可以不用request.setCharacterEncoding("UTF-8")

浏览器请求中文乱码:

get:中文编解码 new String(parameter.getBytes("iso8859-1"),"UTF-8")

post:request.setCharacterEncoding("UTF-8")

或者

 new String(parameter.getBytes("iso8859-1"),"UTF-8")也可以



get post 区别

get:请求参数在地址后面,相对不安全,有大小限制

post:请求参数在请求体中,相对安全,没有大小限制