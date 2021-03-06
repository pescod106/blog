# GET和POST

## GET
GET通常用于请求服务器发送某个资源。使用GET的请求应该只检索数据，并且不应该对数据产生其他影响。GET请求通过在请求的URL部分指定参数才从Web服务器检索数据。这是用于静态文档检索的主要方法。

### 何时使用HTTP GET请求
GET方法不安全，因此不适合传输机密数据，但GET方法对于从Web服务器检索静态内容非常有用。下面是一些使用GET方法的例子：

1. 没有重复请求的副作用。GET是幂等的(每次执行的结果都相同)
2. 没有传递任何敏感和机密数据。而是只传递一些配置数据或会话ID。
3. 希望HTTP GET请求指向的URL可以书签
4. 需要发送到服务器的数据量不大，并且可以安全的容纳在所有浏览器支持的URL的最大长度内。

## POST
POST方法起初是用来向服务器输入数据的。实际上，通常会用它来支持HTML的表单。表单填好的数据通常会被送给服务器，然后由服务器将其发送到他要去的地方。

POST将数据作为消息正文的一部分发送。POST方法是安全的，因为数据在URL字符串中不可见，并且可以使用HTTPS进行安全加密以提高安全性。发送服务器的所有敏感数据和机密信息都必须通过POST请求并通过HTTPS(带ssl的HTTP)进行。POST方法也用于向服务器提交数据，任何可以改变应用程序状态的信息都是一些应该考虑在HTTP中使用POST方法的。下面是一些使用POST方法的示例：

1. 发送的GET数据不适合URL，则使用POST
2. 将敏感和机密信息传递给服务器
3. 提交的数据会改变应用程序的状态
4. 不想在URL中显示查询参数

### 何时使用HTTP POST请求

## 区别

1. GET方法在URL中传递参数，而POST方法在请求体中传递请求参数
2. GET请求只能传递数量有限的数据，而POST方法可以将大量数据传递给服务器
3. 与POST请求不同，GET请求可以添加书签和缓存
4. GET主要用于查看目的，而POST主要用户更新目的
5. GET是默认的HTTP方法，使用POST方法需要特殊指定
6. GET受限于浏览器和Web服务器支持的最大URL长度，而POST没有这种限制

### 通过表格展示区别
||GET|POST|
|:---:|:---:|:---:|
|请求参数传递|在URL字符串中|在请求体中|
|可通过的数据量|只能传递有限的数据量|可以传递大量的数据|
|书签/缓存|get有助于书签和缓存存储|不支持|
|后退按钮/刷新|无害|数据会被重新提交|
|对数据类型的限制|只允许ASCII字符|没有限制，也允许二进制数据|
|安全性|与POST相比，GET的安全性较差，因为所发送的数据时URL的一部分|POST比GET更安全，因为参数不会被保存在浏览器或WEB服务器日志中|
|可见性|数据在URL中对所有人都是可见的|数据不会显示在URL中|
|目的|用于查看展示数据|用于更新目的|
|长度|很短|比通过get方法发送的数据长|
|速度|由于get方法不涉及大量数据，因此速度很快|比get慢|
|默认|get方法时HTML表单提交的默认方法|POST方法必须指定，而不是HTML表单提交的默认方法|
