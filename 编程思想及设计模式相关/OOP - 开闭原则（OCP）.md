>本文是对王争大佬《设计模式之美》的学习总结

# 开闭原则 Open Closed Principle (OCP)

&emsp;&emsp;开闭原则的英文全称是**Open Closed Principle**，简写为**OCP**。他的英文描述是：**software entities (module, classes,functions,etc.) should be open for extension, but closed for modification**。翻译为中文就是：软件实体（模块、类、方法等）应该“对扩展开放、对修改关闭”。

&emsp;&emsp;这个描述比较简略，如果我们详细表述一下，那就是，添加一个新的功能应该是，在已有代码基础上扩展代码（新增模块、类、方法等），而非修改已有代码（修改模块、类、方法等）。有两点需要注意，第一点是，开闭原则并不是说完全杜绝修改，而是以最小的修改代码的代价来完成新功能的开发。第二点是，同样的代码改动，在粗代码粒度下，可能被认定为“修改”；在细代码粒度下，可能又被认定为“扩展”。

# 修改代码就意味着违背开闭原则吗？

&emsp;&emsp;添加一个新功能，不可能任何模块、类、方法的代码都不“修改”，这个是做不到的。类需要创建、组装、并且做一些初始化操作，才能构建成可运行的程序，这部分代码的修改是在所难免的。我们要做的是尽量让修改更集中、更少、更上层，尽量让最核心、最复杂的那部分逻辑代码满足开闭原则。

# 如何做到“对扩展开放、修改关闭”？

&emsp;&emsp;实际上，开闭原则讲的就是代码的扩展性问题，是判断一段代码是否易扩展的“金标准”。如果某一段代码在应对未来需求变化的时候，能够做到“对扩展开放、修改关闭”，那就说明这段代码的扩展性比较好。所以，问如何才能做到“对扩展开放、修改关闭”，也就是粗略地等同于在问，如何才能写出扩展性好的代码。

&emsp;&emsp;为了尽量写出扩展性好的代码，我们要时刻具备扩展意识、抽象意识、封装意识。这些“潜意识”可能比任何开发技巧都重要。在写代码的时候，我们要多花点时间往前多思考一下，这段代码未来可能有哪些需求变更、如何设计代码结构，事先留好扩展点，以便在未来需求变更的时候，不需要改动代码整体结构、做到最小代码改动的情况下，新的代码能够灵活地插入到扩展点上，做到“对扩展开放、修改关闭”。

&emsp;&emsp;还有，在识别出代码可变部分和不可变部分之后，我们要将可变部分封装起来，隔离变化，提供抽象化的不可变接口，给上层系统使用。当具体的实现发生变化的时候，我们只需要基于相同的抽象接口，扩展一个新的实现，替换掉老的实现即可，上游系统的代码几乎不需要修改。

&emsp;&emsp;在众多的设计原则、思想、模式中，最常用来提高代码扩展性的方法有：多态、依赖注入、基于接口而非实现编程，以及大部分的设计模式（比如，装饰、策略、模板、职责链、状态等）。

&emsp;&emsp;举例：我们代码中通过 `Kafka` 来发送异步消息。对于这样一个功能的开发，我们要学会将其抽象成一组跟具体消息队列（`Kafka`）无关的异步消息接口。所有上层系统都依赖这组抽象的接口编程，并且通过依赖注入的方式来调用。当我们要替换新的消息队列的时候，比如将 `Kafka` 替换成 `RockeMQ`，可以很方便地拔掉老的消息队列实现，插入新的消息队列实现。具体代码如下所示：

```java
// 这一部分体现了抽象意识
public interface MessageQueue { //... }
public class KafkaMessageQueue implements MessageQueue { //... }
public class RocketMessageQueue implements MessageQueue { //... }
  
public interface MessageFormatter { //... }
public class JsonMessageFormatter implements MessageFormatter { //... }
public class ProtoBufMessageFormatter implements MessageFormatter { //... }
  
public class Demo {
  private MessageQueue msgQueue; // 基于接口而为实现编程
  public Demo(MessageQueue msgQueue) { // 依赖注入
    this.msgQueue = msgQueue;
  }
  
  // msgFormatter: 多态、依赖注入
  public void sendNotification(Notification notification, MessageFormatter msg){
    // ...
  }
}
```