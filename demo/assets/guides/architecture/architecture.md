# 整体架构

看到这里，想必你对Sein的运作已经有一些模糊的概念了，在这一节，我们将从架构的角度，来从顶层理解引擎的整体设计，以便于后续的章节。

如果让你去构建一个游戏世界，你会如何去抽象？这个问题在不同的引擎中有不同的答案。比如Unity中就是一个个的场景，配合在其中可挂载组件的GameObject来填充这个场景，构成游戏体验。又比如Cocos，则是一个个的Node，所有的操作都直接集成在Node中。当然，无论是用GameObject-Component模式还是Node模式都可以构建出优秀的引擎，但每一种选择都伴随着取舍。比如Unity的模式，其非常灵活，但这种灵活却可能带来缺乏约束的问题，这会导致新手很可能写出不可维护、不可用的代码，当然在高手手里这可能是一把利器，但现实并没有这么多高手，所以使用Unity的团队往往会有资深的游戏架构师来做架构，而这一点对于Web开发团队是很难做到的。  

相对而言，UE4的Actor-Component模式以及其对世界、关卡、控制器、状态等等的拆分和约束，都确保了一个最佳实践的下限，这相当于帮助新手做好了一定程度上的规范，而对于高手，也可以更深层次的去进行改造和定制，这也是我选择借鉴UE4来做Sein的基础架构的原因。所以使用过UE4的开发者可能在使用Sein的时候感觉十分熟悉，这是正常的。而与此同时，我也在这种架构基础上做出了一些优化、调整和裁剪，使其保留UE4优秀架构的同时，更加符合Web端的行为。  

当然，相对于完整的UE4，Sein的能力肯定是远不及的，毕竟Sein只是一个压缩后140K的js库，但麻雀虽小，五脏俱全，良好的扩展性也为Sein提供了无限的可能。通过模块化的设计理念，我将Sein的内核部分抽离了出来，其大多部分外围功能都是为可拆卸、可替换的。也就是说我们未来可以使用WebGPU替换掉WebGL而不需要修改任何原来的业务代码来达到无缝迁移，这也是一个可用的游戏引擎应当做到的。

## 容器

在整体架构中，首先要说明的就是容器。容器是什么概念？在现实生活中，宇宙是一个容器，它容纳着所有的物质，地球也是一个容器，它容纳着草木虫鱼，家庭也是一个容器，它容纳着你和你的家人、以及你们之间的酸甜苦辣。毋庸置疑，我们存在于容器中，容器存在于更大的容器中，而我们自身也是其它存在的容器。这么思考下来，是不是有一种万物皆**Node**的感觉？但这里我们需要先缓一下自己的思考。如果将一切都视为容器，那么就缺少一个“界”的约束，你有你的想法，我有我的想法，这在工程上其实并不是什么好事，我们需要一个统一的看法，也就是将容器进行分类，Sein为大家提供了这样的一种分类——它或许不是在所有状况都是最优的，但绝对是可以应对绝大多数状况的：  

>Engine -> Game -> World -> Level

1. Engine: 整个游戏引擎的顶层，可以理解为一个单纯的容器，容纳着若干个`Game`以及基准时钟`Ticker`。
2. Game: 单个游戏逻辑的顶层，对世界`World`和`Level`进行管理，同时自身也可以容纳一些系统层面的`Actor`，也承担这资源、全局事件、全局状态的管理等功能。
3. World: 游戏中的一个个“世界”，用于间接容纳实质上的3D场景内的物体`Actor`（`SceneActor`），同时也管理着自身的逻辑脚本、状态以及下层的`Level`。
4. Level: 世界中一个个“关卡”，用于直接容纳3D场景内的`SceneActor`，同时也管理着自身的逻辑脚本、状态以及下层的`Level`。

在这种架构下，一个`Engine`可以拥有多个`Game`（拥有不同的Canvas），`Game`拥有多个`World`，`World`拥有多个`Level`，`Level`中又实际承载着3D实例。借由其，我们可以合理分配游戏逻辑，来达到设计上的秩序和复杂度的分割。

## 基石

有了容器，我们便需要一些基石去填充它，让游戏世界生动起来。这些基石，在Sein中被称为`Actor`，你可以认为Actor是游戏中万物的本质，它可以是一个具体的3D物体（`SceneActor`），也可以是一个状态的集合（`StateActor`），甚至可以是一种规则的体现（`InfoActor`）。对于一个游戏，Actor承载了几乎所有的展示、逻辑和控制，但原则上来讲，Sein原生提供的Actor主要分为两种：

1. SceneActor: `World`层面的Actor，也是唯一一种可以被放入世界`World`或者说关卡`Level`中的Actor，其特点是拥有3D变换`Transform`。
2. InfoActor: 在`Game`层面的Actor，起着“书记官”的功能，主要用于描述规则。

而继续细分下去，又有以下种种分类等等：

1. ControllerActor: 承载控制逻辑的Actor，用于对世界中的Actor进行控制，一般用其实现AI或者玩家逻辑，其一般存储在`Game`下。
2. StateActor: 用于记录状态的Actor，其一般存储在`Game`下。
3. ScriptActor: 承载世界或者关卡的具体玩法逻辑的Actor。
4. SystemActor: 承载着系统功能的Actor，比如渲染系统、物理系统等等。
5. ......

>当然你现在不必记住这些，后面都有更加详细的介绍。  

你也可以自己派生出需要的Actor类型。有了这些Actor，我们就拥有了构建世界的基石，然而和最初说到的一样，本质上这个Actor也是一种容器。很多时候我们不能指望一个Actor就承担了所有的功能，也不能指望所有Actor都能完美正交，它们的逻辑大概率会有重合，既然有“重合”和“分隔”，那么“提取”和“复用”也就被自然而然得被提出来了。

## 组件

用于提供“复用”和“分隔”功能的就是组件。组件这个概念并不少见，对于前端同学而言应该更加熟悉，无论是React、Vue还是Angular，都是讲基本功能拆成了一个个内聚的组件，组件提供了良好的复用性以及易用性，通过嵌套组件便可以构成应用大厦。而在Sein中，组件的理念也基本一致——提取和分隔出Actor共用的功能，使得一种逻辑可以被应用到各种不同的Actor上。  

这也就是Sein的**Actor-Component**模式的理念，`Actor`本身原则上只是一种带有特殊功能的容器，而其大部分功能逻辑是由`Component`实现的。当然这里要区分“功能逻辑”和“业务逻辑”，功能逻辑指的是与业务无关的逻辑，这种逻辑无论在什么项目理论上都是可以复用的，比如摄像机控制器、粒子组件；而业务逻辑，则适合项目耦合严重的逻辑，这部分逻辑建议写在逻辑脚本或者Actor容器内。  

从这个层面来看，Actor可以看做是Component的一个包装，所以所有Actor都拥有一个子级的根级组件`RootComponent`，它承载着Actor的核心逻辑。对于一些Actor，其本身并无特别逻辑，比如`StaticMeshActor`的根组件就是一个`StaticMeshComponent`，而StaticMeshActor只是一个单纯的容器；而对于一些Actor却有自己的一些逻辑，比如`SceneActor`就在`SceneComponent`外还有一些其他逻辑，所以你要想使得一个Actor可以被放入世界，就必须让继承自`SceneActor`。

当然，对于`InfoActor`，一般来讲其本身就承载了完成的逻辑。

## 内核与多态

容器、基石和组件都是整个GamePlay层次的架构，而对于引擎本身而言，则还有更加底层的设计：

1. SObject: Sein里除了渲染层之外绝大部分类的基类，提供了基本的UUID自动计算、序列化与反序列化等能力。
2. SName: Sein中绝大部分和“名字”、“标签”有关的变量的类，其本质上提供了一个字符串池，用于快速对比。
3. Ticker: 时钟。
4. Timer: 定时器。
5. Decorator和MetaTypes: 装饰器和元数据。装饰器目前提供了`SClass`和`SMaterial`，分别用于给继承自`SObject`和`RawShaderMaterial`的类标注类型，并将它们添加到元数据中来方便反射。
6. Tween: Tween动画单例。
7. Constants: 存储着GL相关的静态变量。
8. Math: 提供了一些基本的数学类。

## 玩法逻辑

有了这些基础，我们又该将玩法逻辑编写在哪里呢？不同引擎同样有不同的答案，比如Unity，就是万物皆`Component`，所有逻辑都在组件内，这样固然灵活，但问题上面也说过了。在Sein中，你同样有很多方式来编写逻辑，但推荐的最佳实践是——利用`GameModeActor`和`LevelScriptActor`。  

这两个类前面已经提过很多遍了，在模板工程中也对它们做了基本介绍，后面也会有更详细的介绍，这里就大概提点一下它们的功能——承载游戏业务逻辑。在Sein的整个游戏编程体验中，能承载业务逻辑的部分很多，但实际上承载它们的却只有几个部分：`Game`、`World`、`Level`以及后面的`Player`。一般而言，我们不会直接去修改`Game`的逻辑，最多对其进行简单的修改，而在直觉上，我们也会发现游戏的绝大部分逻辑都是和世界、关卡、玩家相关的。一个世界的所有关卡会共用同一个玩法逻辑，而每个关卡又有各自的展示逻辑，比如**超级马里奥 奥德赛**。  

Sein将世界的玩法逻辑抽象出来，称之为**GameModeActor**，将每个关卡的展示逻辑抽象出来，称之为**LevelScriptActor**，它们实际上是世界和关卡在逻辑层面的代理。作为Actor的派生，它们在Actor原先的能力上增添了不少能力（比如一些生命周期）而同时也保留了Actor的容器能力（可以添加组件），利用这一特性，你可以将一段复杂的业务逻辑拆到很多组件去实现。同时每个脚本都可以指定自己的`StateActorClass`，它可以为世界或者关卡添加一个当前作用于的全局状态，方便用于记录和处理一些数据。  

在实际使用中，你向游戏中添加一个世界，实际上是为世界指定一个玩法逻辑脚本，添加一个关卡，则是为关卡指定一个展示逻辑脚本。

但当然，游戏逻辑是万变的，所以在实际状况中我们也常常会自定义`SceneActor`和`InfoActor`来承载具体的逻辑。甚至这种策略在配合**Unity做编辑器**的时候我们经常会用到。

## 玩家和AI

在世界和关卡之外，另一个承担业务逻辑的就是玩家了。现在很多手游都有重度的玩家系统，它们甚至超越了游戏玩法逻辑本身，Sein提供了一整套玩家系统机制来帮助开发者解决这个问题。  

Sein的玩家系统核心是**M（状态模型）**、**V（控制对象）**和**C（控制器）**，它们的具体实现分别是`PlayerStateActor`类、`SceneActor`类和`Player`/`PlayerControllerActor`类，三者相辅相成却又互相独立，可以随意拆卸组合。`PlayerStateActor`是玩家的状态，用于管理玩家的模型；`Player`是游戏层面的的玩家的实体，一般可以用于处理玩家输入等逻辑，它一般在世界层面进行创建和释放，`PlayerControllerActor`则是玩家在世界中的一个代理，用于实际操纵世界内的3D对象；而`SceneActor`则是具体操纵的对象，根据实际的逻辑而定。  

AI系统和玩家系统也是很相似，只不过去掉了`Player`，因为AI的生命周期理论上不会超出当前场景，最多也就是状态会持久化，所以其系统由`PlayerStateActor`和`AIControllerActor`和`SceneActor`构成。这样一来，玩家系统和AI系统的各个部分都是可以完美隔离的，设计合理可以轻易实现玩家和AI的切换等操作。  

而在AI逻辑层面，Sein原生集成了FSM来进行逻辑编程，当然开发者也可以自己编写HFSM或者行为树组件，来达到更好的效果，这要根据场景进行选择。

## 资源

资源管理系统，理论上是游戏引擎中最复杂的一部分，但对于一个Web端的游戏引擎，其本身就不可能特别复杂，所以Sein的资源管理也相对简单一些，然而虽然相对简单，却也足够。资源管理是由资源管理器类`ResourceManager`以及资源加载器`ResourceLoader`的派生类们两部分构成的，而加载器本身采取了“注册”的机制，使得你可以随时添加新的或者替换已有的加载器，十分灵活方便扩展。  

资源管理器提供了标准的接口，用于**加载**、**创建**、**释放**、**实例化**资源，结合脚本的生命周期，可以达到在Web端比较友好的资源管理体验。

## 事件

事件系统是游戏引擎非常重要的组成部分，在Sein中深度集成了一个基于`Observable`的事件系统，其核心是事件管理器类`EventManager`以及可选的事件触发器`EventTrigger`的派生类。和资源管理器同样，事件管理器也提供了注册的方式来管理事件，你可以在注册时指定一个事件触发器，将事件的触发完全托管给触发器，比如**HID**的各个实现，也可以直接利用手动触发。不过与资源管理器不同，若你直接向一个事件管理器添加未注册的事件，事件将被自动注册。与此同时，事件管理器还提供了可选的**帧同步**机制，它将保证事件在每一帧前触发，避免多余的开销。

时间管理器提供了标准的接口，用于**注册**、**添加**、**移除**、**触发**事件，每个Component都会默认挂载一个事件管理器，`Game`全局也有一个全局的事件管理器，对于不同的特例化Component，可能有一些默认事件，你可以通过泛型来帮助类型推断。

## 异常

对于异常的处理，一直是各大游戏引擎、或者说是所有框架的痛点。一不留神就会出现`try-catch`满天飞的状况，为了避免这种情况，Sein学习了React的[Error boundaries](https://reactjs.org/docs/error-boundaries.html)，对容器以及组件和其他大部分类都提供了一个生命周期`onError: (error, details) => boolean | void`，并结合`throwException`函数来实现一个层层传递的错误链，这使得异常可以追踪并且集中处理，比如一个组件抛出了异常，便会在开发模式下控制台打印：  

>Stack: Component(customComponent) -> SceneActor(customActor) -> Level(persistent) -> World(main) -> Game(intro-game) -> Engine(SeinEngine)

这给出了整个异常的链条，你可以在其中任意一环的`onError`中对其异常进行访问并返回`true`来拦截异常的传递。  

注意原生的`throw new Error()`的方式只能在**生命周期**的**同步方法**中才能正确进入标准异常流程，如果想捕获异步异常，请使用`Sein.throwException`方法。

## 物理

物理引擎也是游戏引擎相当重要的一部分，其一般由**碰撞**和**模拟**构成。Sein通过组件的方式集成了一整套物理引擎，这里要感谢BabylonJS，它良好的设计为Sein提供了很好的借鉴，Sein提供了一个标准的物理世界接口，理论上任何第三方引擎都可以通过实现这个接口来接入Sein。Sein默认提供了[CANNONJS](https://github.com/dtysky/cannon.js)（注意是魔改版！）的适配，你可以直接利用`world.enablePhysic`来启用物理引擎。加之`RigidBodyComponent`和各类`ColliderComponent`的派生，便可以十分简单地使用物理功能了。

Sein提供了许多方式来为物理效果做优化，并且对于模型也有glTF扩展来直接追加物理功能。

## 渲染

渲染可谓是游戏引擎最重要和复杂的部分之一，由于一些缘故，Sein的渲染引擎是同团队开发的**Hilo3D**，并在其上做了一些改造使其更加符合Sein的设计（比如`ShaderMaterial`）。当然由于Hilo3D自身是比较纯粹的渲染引擎，所以改动也不多。  

在渲染层，Sein暴露了许多必要特性来使得开发者可以充分利用其能力，比如各类`Geometry`、`Material`、`Texture`、`Mesh`以及`Camera`、`Light`等，对于其中大部分和逻辑息息相关的内容，比如`Mesh`、`Camera`，Sein全部进行了组件化封装。所以一般情况下开发者是不需要直接接触到渲染层的，就算接触到也是Sein这一层，Hilo这一层基本不会接触。

## 动画

动画系统和渲染层结合紧密，也有很多分类。其核心是一个组件`AnimatorComponent`，用于管理一个Actor下的所有动画，这些动画都派生自`Animation`基类。你可以通过继承Animation来实现自己的基准动画并在AnimatorComponent中注册它，同时也可以通过组件内置的FSM来进行动画的状态编程。  

目前Sein内置了`ModelAnimation`、`TweenAnimation`等关键动画，可以解决绝大部分的Web游戏开发场景。详细介绍请见后续章节。

## 音频

音频系统对于游戏也很重要，Sein利用其扩展架构，可以通过第三方组件将音频系统引入，详见[空间音频系统](../../extension/web-extensions/audio)。
