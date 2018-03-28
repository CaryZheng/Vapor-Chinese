# Parameters

Parameters 可以通过字符串初始化

Parameters 作为路由的一部分，你可以从被调用的 route中的 request中获取 Parameters

# 创建自定义 parameters

创建自定义 parameter类型只需要遵守Parameter协议，并实现转换方法 make 和 unique slug 标识

下面这个例子里，我们通过 parameter 生成一个 user 类

我们提醒您，使用过程需要修改唯一标识符

~~~
class User : Parameter {
  var username: String

  // 唯一标识（需要修改）
  static var uniqueSlug = "my-app:user"

  // 通过parameter 生成 User
  static func make(for parameter: String, in request: Request) throws -> User {
    return User(named: parameter)
  }

  init(named username: String) {
    self.username = username
  }
}
~~~

# 使用自定义 parameter

遵守了 Parameter 协议的类型会拥有一个名为parameter的静态属性，它可以放在路由中使用

~~~
router.on(.get, to: "users", User.parameter, "profile") { request in
  let user = try request.parameters.next(User.self)

  // Return the user's Profile sync or async (depending on the router)
}
~~~