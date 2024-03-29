# 访问者模式
访问者模式建议将新行为放入一个名为**访问者** 的独立类中，而不是试图将其整合到已有类中。**访问者类可以定义一组 （而不是一个） 方法， 且每个方法可接收不同类型的参数，需要执行操作的原始对象将作为参数被传递给访问者中的方法** ，让方法能访问对象所包含的一切必要数据。

它使用了一种名为双分派的技巧，避免了大量的`instance of` 判断。如果我们抽取出所有访问者的*通用接口*， 所有已有的节点都能与我们在程序中**引入** 的任何访问者交互。 如果需要引入与节点相关的某个行为， 你只需要实现一个新的访问者类即可。

![](http://img.070077.xyz/20221217034708.png)

---
参考：
[常用设计模式有哪些？ (refactoringguru.cn)](https://refactoringguru.cn/design-patterns)

