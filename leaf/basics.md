# Basics

欢迎来到Leaf，Leaf的目标是用简单的模板语言生成页面。现在有很多不错的模板语言，用哪个一适合你呢，来试试Leaf吧。Leaf的目标是：

* 严格的执行标准
* 一致性
* 优先解析
* 可扩展性
* 异步和响应式

# 渲染模板

如果你已经安装了Leaf，你需要在工程目录下创建 “Resources” 文件夹。在Resources文件夹内再创建一个名为 “Views” 的文件夹。 Resources/Views 文件夹是默认的Leaf模板路径，当然如果你想使用其它路径也可以进行修改

首先 导入Leaf到 routes.swift 文件

~~~swift
import Leaf
~~~

通过路由渲染最基本的Leaf模板

~~~swift

router.get { req -> Future<View> in
    let leaf = try req.make(LeafRenderer.self)
    let context = [String: String]()
    return try leaf.render("home", context)
}

~~~

上面代码会从 Resources/Views 文件夹下加载 home.leaf 文件。 **context** 字典允许你提供一些需要渲染到模板中的自定义数据，但是你最好使用遵守了Codable协议的结构体来代替字典，来保证类型安全。例如：

~~~swift
struct HomePage: Codable {
    var title: String
    var content: String
}
~~~

# Async 异步

Leaf引擎是完全响应式的，支持 steams 和 futures.

Future 可以直接在模板上下文中使用，Streams 需要使用EncodableStream进行编码后使用

~~~swift
struct Profile: Codable {
    var friends: EncodableStream
    var currentUser: Future<User>
}
~~~

**currentUser** 变量在Leaf运行过程中会作为 **User** 类型。如果在渲染过程中没有使用User，那么Leaf将不会读取User.

**EncodableStream** 运行过程中会作为 LeafData类型的数组，占用少量内存，高性能。建议在查询大型数据库时使用

~~~swift
Your name is #(currentUser.name).

#for(friend in friends) {
    #(friend.name) is a friend of you.
}
~~~

# 模板语法

### Structure

Leaf tags 由四个元素构成

* Token : **#** 代表 Token
* Name : A **String** 是识别标识
* Parameter List: **( )** 代表可以接受0或者多个参数
* Body(option): **{ }** 和 Parameter 之间必须用一个空格分割

有很多种方法可以组合使用这四种元素来创建标签，下面例子展示了Leaf怎么创建标签节点

~~~swift
#()
#(variable)
#embed("template")
#set("title") { Welcome to Vapor }
#count(friends)
#for(friend in friends) { <li>#(friend.name)</li> }
~~~

### 使用 context

在我们之前的示例中，我们使用空的 **[String:String]** 字典传递给 context。如果需要传递自定义数据到Leaf可以像下面这样做

~~~swift
let context = ["title": "Welcome", "message": "Vapor and Leaf work hand in hand"]
return try leaf.make("home", context)
~~~

这样 title 和 message会被传递到Leaf中， 在Leaf中我们可以这样使用

~~~html
<h1>#(title)</h1>
<p>#(message)</p>
~~~

### Checking conditions 条件判断

Leaf 允许通过使用 **#if** 标签进行范围，条件判断，如果你需要检查上下文环境中变量是否为真，可以像下面这样做

~~~html
#if(title) {
    The title is #(title)
} else {
    No title was provided.
}
~~~

当然也可以进行比较判断

~~~html
#if(title == "Welcome") {
    This is a friendly web page.
} else {
    No strangers allowed!
}
~~~

如果你想使用其他标签作为判断条件的一部分，需要省略内部标签的 **#**,例如：

~~~html
#if(lowercase(title) == "welcome") {
    This is a friendly web page.
} else {
    No strangers allowed!
}
~~~

### 循环

如果你提供了 item 集合，使用 **#for** 标签，Leaf 可以帮助获取每一个 item.

~~~swift
let context = ["team": ["Malcolm", "Kaylee", "Jayne"]]
~~~

在 leaf 中我们可以这样写 

~~~html
#for(name in team) {
    <p>#(name) is in the team.</p>
}
~~~

Leaf提供了一些属性来帮助从for循环中获取更多的信息

* **loop.isFirst** 属性为 true ，那么当前循环就是第一次迭代
* **loop.isLast** 属性为 true ,那么就是最后一次迭代
* **loop.index** 设置当前迭代的数量，从0开始

### Embedding templates

Leaf的 **#embed** 标签可以复制模板内容到其他模板中，在使用这个标签的时候需要省略模板的文件后缀 .leaf

嵌入模板在一些内容复制上是非常有用的，例如嵌入页脚或者广告代码

~~~swift
#embed("footer")
~~~

这个标签对于嵌入其他模板顶部也非常有用，例如，你有一个包含了所有代码请求网站布局（html结构，CSS和JavaScript）的master.leaf 文件。

使用这种方法你可以构建一个唯一子模板来嵌入到父模板中。

例如可以创建一个 child.leaf

~~~html
#set("body") {
<p>Welcome to Vapor!</p>
}

#embed("master")
~~~

上面创建一个 body 的上下文，但是不直接显示他，相反，我们把它嵌入到 master.leaf 中。它同样可以从Swift代码中获取变量.

~~~html
//master.leaf
<html>
<head><title>#(title)</title></head>
<body>#get(body)</body>
</html>
~~~

当上下文字典为 **["title":"Hi there!"]**, child.leaf 会像下面这样被渲染

~~~html
<html>
<head><title>Hi there!</title></head>
<body><p>Welcome to Vapor!</p></body>
</html>
~~~

### 其他标签

#### #capitalize

**#capitalize** 标签可以把任何字符串的首字母转成大写 例如 “taylor” 会被转换成 “Taylor”

~~~html
#capitalize(name)
~~~

#### #contains

**#contains** 接收两个参数，第一个参数为数组，第二参数为值，如果数组中包含值则返回true。例如

~~~html
#if(contains(team, "Jayne")) {
    You're all set!
} else {
    You need someone to do PR.
}
~~~

#### #count

**#count** 标签返回数组中元素个数

~~~html
Your search matched #count(matches) pages.
~~~

#### #lowercase

**#lowercase**会把字符串转成小写,例如“Taylor” 转换 “taylor”.

~~~html
#lowercase(name)
~~~

#### #uppercase

**#uppercase**字符串转大写，例如“Taylor” 转成 “TAYLOR”.

~~~html
#uppercase(name)
~~~