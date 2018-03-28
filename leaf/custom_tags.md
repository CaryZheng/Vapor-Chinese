# Custom Tags


你可以通过添加一些自定义方法来扩展Leaf的标签。为了演示这个功能，我们一起去重新创建一个 **#uppercase** 标签，这个标签只接受一个参数。

在我们进行自定义标签的时候，下面四件事情是非常重要的

* 你需要调用 **requireParameterCount()** 参数是你希望标签接受参数的个数。如果你没有正确的使用标签，会抛出异常
* 如果你使用body或不使用，你需要调用 **requireBody()** 或者 **requireNoBody()**。如果没有正常使用会抛出异常
* 你可以访问 **parameters** 数组中的每一个元素。它们都是LeafData类型，你可以通过 **.string**, **.dictionary** 等方法转换成具体类型
* 你必须返回一个包含需要渲染的信息的 **Future < LeafData? >**，在下面示例中，我们把大写后的String包装成LeafData，并用Future包装后返回

下面是 **CustomUppercase** 标签的代码

~~~swift
import Async
import Leaf

public final class CustomUppercase: Leaf.LeafTag {
    public init() {}
    public func render(parsed: ParsedTag, context: LeafContext, renderer: LeafRenderer) throws -> Future<LeafData?> {
        // ensure we receive precisely one parameter
        try parsed.requireParameterCount(1)

        // pull out our lone parameter as a string then uppercase it, or use an empty string
        let string = parsed.parameters[0].string?.uppercased() ?? ""

        // send it back wrapped in a LeafData
        return Future(.string(string))
    }
}
~~~

现在我们可以在 **configure.swift**文件中配置自定义标签

~~~swift
services.register { container -> LeafConfig in
    // take a copy of Leaf's default tags
    var tags = defaultTags

    // add our custom tag
    tags["customuppercase"] = CustomUppercase()

    // find the location of our Resources/Views directory
    let directoryConfig = try container.make(DirectoryConfig.self, for: LeafRenderer.self)
    let viewsDirectory = directoryConfig.workDir + "Resources/Views"

    // put all that into a new Leaf configuration and return it
    return LeafConfig(tags: tags, viewsDir: viewsDirectory)
}
~~~

至此，你就可以使用自定义标签 **#customuppercase(some_variable)**了

> 注意：我们并不鼓励使用字母混合数字的方式定义自定义标签，这在Leaf的未来版本中有可能会被禁止使用