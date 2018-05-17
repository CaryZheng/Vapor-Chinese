# 使用Routing

Routing([vapor/routing](https://github.com/vapor/routing))是像HTTP请求路由那样的一个小巧的框架。它允许你注册或者查看一些嵌套或者动态路径的组件。

> Tip：
> 如果你使用Vapor，大多数路由API会被包装一层更方便的方法。你可以通过 [Vapor -> Routing](../)来了解更多信息

本引导将会向你展示如何注册静态和动态路由，并且让你了解如何使用[Parameters](https://api.vapor.codes/routing/latest/Routing/Protocols/Parameter.html)

# 注册

使用Routing的第一步是注册路由，我们一起来看看如何完成一个简单路由，**TrieRouter < Double >** 可以获得数字，通常情况下你可能会存储一些像HTTP responder的信息，但是我们在这个示例中我们只做一些简单的事情

~~~swift
// 创建一个路由来存储浮点型数字
let router = TrieRouter(Double.self)

// 注册一个路由到Router中
router.register(route: Route(path: ["funny", "meaning_of_universe"], output: 42))
router.register(route: Route(path: ["funny", "leet"], output: 1337))
router.register(route: Route(path: ["math", "pi"], output: 3.14))

// 创建一个空参数，用来存放参数
var params = Parameters()

// 测试路由
print(router.route(path: ["fun", "meaning_of_universe"], parameters: &params)) // 42
print(router.route(path: ["foo"], parameters: &params)) // nil
~~~

这里我们使用[register(...)](https://api.vapor.codes/routing/latest/Routing/Classes/TrieRouter.html#/s:7Routing10TrieRouterC8registeryAA5RouteCyxG5route_tF)来注册路由到Router中，然后使用[route(...)](https://api.vapor.codes/routing/latest/Routing/Classes/TrieRouter.html#/s:7Routing10TrieRouterC5routexSgSayqd__G4path_AA10ParametersVz10parameterstAA17RoutableComponentRd__lF)来获取他们。[TrieRouter]()内部使用特里结构（数字查找树）来保证在Router中查找值的效率

# Parameter

让我们来看看注册一些动态路径组件的时候，其中有一部分路径为变量，我们需要收集起来稍后来使用这些变量。你会经常见到下面的情景当你像用户展示一个web页面的时候.

~~~
/users/:user_id
~~~

下面是如何通过 **TrieRouter** 来实现。在这个示例中，我们先忽略路由的 output

~~~swift
// 创建一个路由 /users/:user_id
let user = Route(path: [.constant("users"), .parameter("user_id")], output: ...)

// 创建一个Router并注册我们的路由
let router = TrieRouter(...)
router.register(route: user)

// 创建一个空参数集合
var params = Parameters()

// 路由到路径 /users/42
_ = router.route(path: ["users", "42"], parameters: &params)

// params 中包含我们传入的动态值
print(params) // ["user_id": "42"]
~~~

注意当你使用 **.parameter(...)** 时候传入的字符串，将会作为你从 **Parameters** 中获取value时的key。

# API 文档

查看 [API 文档](https://api.vapor.codes/routing/latest/Routing/index.html) 来获取一些关于参数和方法的深层信息