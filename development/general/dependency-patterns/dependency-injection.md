# Dependency Injection

## What is it?

简单地说，这是一种使用类型解析器的大型容器来解析类依赖关系的方法。几乎每种现代编程语言都有大量的DI框架，当我们关注C#语言时，也有大量的框架，比如**Ninject**、**AutoFac**、**Unity**（Microsoft）等等。然而，在game dev（Unity）的上下文中，支持的并不多，所以主要的选项是**Zenject**和**StrangeIoC**，尽管对于这个例子，我们将在这里使用**Ninject**，纯粹是因为它更轻量级，并且有很多文档。

> 您可以在他们的网站上找到所有文档和信息：
- [**Ninject**](https://github.com/ninject/ninject/wiki)
- [**AutoFac**](https://autofaccn.readthedocs.io/en/latest/)

> 还可以在这里学习到IOC相关概念[IOC](https://www.tutorialsteacher.com/ioc)

在高层次上，几乎所有依赖注入框架都共享相同的主体，因此尽管我们在这里使用特定的框架，但是即使语法不同，您也可以轻松地应用相同的主体。因此，让我们从一个简单的绑定容器文件开始，这是您经常放置所有设置绑定的地方。

### Lifecycle

通常在DI世界中，对象的生命周期是相同的:

- Binding
- Resolving
- Activation
- Deactivation

因此，首先，您将绑定所有对象，告诉它们如何解决这些问题，然后一旦完成，您通常能够提供在激活（实例化）对象和停用（释放）对象时运行的任何自定义代码方面的附加逻辑。并非所有的DI框架都将这些步骤称为相同的步骤，即Bind可能被称为Register，Resolve可能被称为Get等，但通常语法上的差异意义不大，仍然是相同的事情。

激活和去激活并不是非常重要，因为一般来说你在这方面不会做太多，但值得知道的是这个概念的存在，你也可以在此基础上做一些类似**AOP**的事情，我们将在后面讨论。

### Binding Setup

所以从Ninject开始，它有一个MonoInstaller的概念，它包含所有绑定设置。

```csharp
using Ninject;

public class MyInstaller : NinjectModule
{
    public override void Load()
    {
        Bind<ISomething>().To<SomeImplementation>();
        Bind<ISomethingElse>().To<SomethingElse>();
    }
}
```

现在您可以看到，我们正在将类型 `ISomething` 绑定到类 `SomeImplementation`，您还可以使用 `typeof(T)` 作为参数，而不是上面使用的泛型方法。我只是在编一些场景，但希望您能将接口和实现它的类可视化，以防下面不是它的样子：

```csharp
public interface ISomething {}
public class Something : ISomething {}
```

所以这是一个常见的绑定场景，您使用一个接口并将其绑定到一个类（在本文中通常称为具体类），这意味着如果我有这样一个类：

```csharp
public class SomeClassWithDependency
{
	private ISomething _something;

	public SomeClassWithDependency(ISomething something)
	{
		_something = something;
	}
}
```

依赖性框架知道如何为您解析 `ISomething` 类，正如我们在看到 `ISomething` 时告诉它的那样，只要您传递给它一个 `ISomething` 。如前所述，几乎所有DI框架都有这个概念，通常称为**Bind**或**Register**。

#### Object Scoping (Transient, Singleton etc)
现在我们已经讨论了绑定的方式，让我们看看如何提高对象的绑定寿命。所以上面的例子将被称为Transient，这意味着它基本上会为每个解析的 `ISomething` 创建一个新的 `Something` 实例。例如，如果我们有3个类依赖于 `ISomething` ，就会创建3个 `Something` 实例。这可能是好的，但是在某些情况下，您可能希望只有一个给定类的实例。

```csharp
Bind<ISomething>().To<Something>().InSingletonScope();
```

因此，对于每个依赖项，现在只提供一个 `Something` 实例。这里允许我们有一个对象，它的行为就像一个单体，但没有任何缺点。如前几章所述，当您正确使用DI时，很少需要显式地使用单例类和静态类，这使得类的耦合性大大降低，并且更易于测试，因为这将成为一个配置问题。

#### Binding to Instances/Self
另一个相关的绑定场景是一个实例，这个实例并不经常使用，但是在某些场景中，您需要进行一些复杂的设置来创建一个实例，这看起来像：

```csharp
var something = new Something(); // or some complex setup
Bind<ISomething>().ToConstant(something);
```

这将传递您创建的实例，而不是让DI框架处理创建。几乎所有的DI框架都有上述概念。

还可以将类绑定到其自身，这主要用于**具体类**。

```csharp
Bind<ConcreteClass>().ToSelf(); // Just use itself and sort the dependencies
```
> 有更多的绑定场景，一些是特定于游戏开发人员的世界（比如用**Zenject**绑定到**预置**、**方法**、**游戏对象**），还有一些是特定于web开发人员的世界，最终你可以在他们的站点上为每个DI框架读到更多关于这个的内容。

### Skill Transference
因此，尽管这些都是针对特定对象 **Ninject** 的，但这里学到的知识可以应用于其他框架和平台。例如，这里的示例是如何在中执行常见操作：

#### Using Zenject (Unity 3d)

```csharp
using Zenject;

public class MyInstaller : MonoInstaller
{
    public override void InstallBindings()
    {
        Container.Bind<ISomething>().ToTransient<SomeImplementation>();
        Container.Bind<ISomethingElse>().ToTransient<SomethingElse>();
        Container.Bind<ISomethingMore>().ToInstance(new SomethingMore())
    }
}
```

#### Using AutoFac

```csharp
using AutoFac;

public class MyModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        builder.RegisterType<SomeImplementation>().As<ISomething>(); // Notice how its other way around
        builder.RegisterType<SomethingElse>().As<ISomethingElse>().InstancePerDependency();
        builder.Register(c => new SomethingMore()).As<ISomethingMore>();
    }
}
```

正如你所看到的，虽然语法略有不同，但它仍然在做同样的事情。

## Resolving Bindings

正如前面提到的，在几乎所有的情况下，您**总是希望使用构造函数注入**，这是为您自动完成的，假设您遵守了**ioc**。但是，您可能需要处理其他场景，例如属性注入或游戏开发人员世界中的scene/gameobject注入（这些将在游戏开发人员特定部分中进行更多讨论）。

> 您几乎总是希望使用构造函数注入，因为这意味着您的对象不知道DI框架，也就是说，如果您希望使用属性注入，您通常必须在属性上添加一个属性，这个属性意味着您必须添加 `using SomeDIFramework`，这使得该模型和使用它的所有内容都依赖于特定的DI框架。理想情况下，你不想落入这个陷阱，因为它就像一个病毒传播。

一旦设置了应该如何解析依赖项，现在就需要从容器中获取所需类型的实例（用于存储所有绑定的名称）。这可以根据您的场景以不同的方式处理，例如在ASP MVC（web世界）中，您最终将加载一个引导程序来为您进行解析，因此您不需要手动解析任何内容，但是让我们假设我们确实需要手动解析内容。

### Manually Resolving Instances

让我们快进一分钟说“这就是解析对象的方式”：

```csharp
var somethingImplementation = container.Get<ISomething>();
```

既然我们已经排除了这个障碍，让我们回顾一下，看看从开始到结束发生了什么，更深入一点。

> 这并不是100%发生的事情，因为实际的反射有时会提前发生，有时会在解析时发生，而且在某些情况下，它可能会有其他元数据来描述它应该如何处理绑定，但是对于所有意图和目的来说，这都足够准确，可以让您了解引擎盖下发生的事情。

```csharp
// Our class in some file
public class SomeClassWithDependency : ISomeClassWithDependency
{
	private ISomething _something;

    // We have a dependency in ISomething
	public SomeClassWithDependency(ISomething something)
	{
		_something = something;
	}
}

// In our module we start binding
Bind<ISomething>().To<Something>();
// 1. Get type ISomething
// 2. Track that it has an implementation for type Something

Bind<ISomeClassWithDependency>().To<SomeClassWithDependency>();
// 3. Get type ISomeClassWithDependency
// 4. Track that it has an implementation for type SomeClassWithDependency

// This lives in some file where you need an instance of ISomeClassWithDependency
var someInstanceWithDependenciesMet = container.Get<ISomeClassWithDependency>();
// 5. Get type ISomeClassWithDependency from the binding information on the container
// 6. Get the default implementation type bound for ISomeClassWithDependency
// 7. Get the constructor/s for SomeClassWithDependency
// 8. Check what dependencies the constructors require (in this case ISomething)
// 9. Go back to step 5 for each dependency (this is looping through creating the dependency tree)
// 10. Once all dependencies are met instantiate and return implementation for ISomeClassWithDependency
```

现在这看起来像是很多步骤，但实际上很简单，也没有什么神奇的事情发生，它只是分析依赖树，为你需要什么，并提前寻找所有的依赖关系，并返回给你一个对象与一切建成。

在大多数真实场景中，您可能拥有非常大的树，因为您越是坚持良好的设计实践（即组合而不是继承、ioc等），最终您将拥有许多较小的对象，这些对象将在需要时自动为您生成。

正如我们刚才提到的，这里还需要考虑scopes/lifetimes的概念，因此，如果假设我们已经完成了 `Bind<ISomething>().To<Something>().InSingletonScope()` ，容器只会实例化一次 `ISomething` 的实现，然后每次需要它时（无论是直接的还是作为另一个类中的依赖项），它都会只需返回现有的实现。这基本上允许您拥有单例样式的实例，而无需将其编码为单例（有关为什么单例不确定的主题，请参阅反模式）。

### Auto Resolving

在大多数真实世界的用例中，您不需要手动解析任何内容，因为webapp世界中的大多数大型框架都有引导程序库，它们将自动为您解析类，例如在[ASP MVC中，您可以引导Ninject](https://github.com/ninject/Ninject.Web.Mvc)它将自动让所有绑定的 `Controller` 类在MVC中注册，因此您只需要处理绑定方面。和**Zenject**一样，你只要给它一个project/scene上下文和你的安装程序，然后关闭它，自动为你解析所有的部分。