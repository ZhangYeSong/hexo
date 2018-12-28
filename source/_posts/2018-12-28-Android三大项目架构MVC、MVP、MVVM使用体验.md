---
title: Android三大项目架构MVC、MVP、MVVM使用体验
date: 2018-12-28 15:46:39
categories:
 - Android
tags:
 - Android
---
先上个链接吧，[谷歌官方github的Android架构demo](https://github.com/googlesamples/android-architecture)，里面有各种各样的MVP、MVVM架构，可以clone下来自己新建项目的时候作参考。clone后需要切换branch才能看到其他架构的代码。
今年MVC、MVP、MVVM都有写，其中MVVM以前没接触过，现在终于可以谈谈这三种架构的使用感受了。由于今年写的Android项目都是商业的，没写个人项目，代码贴不了（可以说纯干聊，无干货- -）。

### 子虚乌有的MVC（Model-View-Controller）
为什么说Android MVC架构子虚乌有？MVC分层概念是在Java后台Spring框架里拿过来的。人家的Controller和View层代码确实是分离的。Android里面哪里分了，对此我斗胆做个猜测：在发明Android-MVP架构的时候，大家不知道怎么称呼原始的Android架构，于是强行从Spring那里拿了个MVC的名字把它和MVP区分开来。

Controller层的本意是处理事件的分发和响应，把Activity看作Controller层也不是不可以，但是既然View和Controller并没有分层，没必要牵强附会到MVC上去。

所以所谓的MVC在Android里就是没有分层架构的意思。没有分层的架构是不是就一定不好呢？当然不是，别说小型应用，就是谷歌自己写的应用也有好几千行代码一个Activity好几千行代码撸到底的呢（想找个例子来着，发现最新版本的aosp里Activity和Fragment代码都精简了不少）。

在代码领域很多事情不是非此即彼，我见过不少商业项目有着各种架构、设计模式还有封装好的基类，最后Presenter里就几行代码甚至一行代码都没有。因此对于小项目，Activity里的代码不重的话，没必要各种分层。对于可能会写成大项目的项目，也可以先不分层，后续该重构的时候再重构。真的要一个Activity几千行的话肯定是不建议的，阅读性太差，定位、修改起来也十分困难。

### 商业项目广泛使用的MVP（Model-View-Presenter）
MVP架构在Android领域可谓十分成熟，我见过的商业项目很多都喜欢用MVP架构。好处自然不用多说，把与UI无关的逻辑代码全部抽出来放到Presenter层里，用接口明确定义好View层和Presenter层的功能。这样的话架构层次清晰，阅读和修改起来要容易得多。

不过使用MVP架构的项目往往会有过度封装的倾向。基本上都会封装有BaseActivity、BaseFragment、BasePresenter，BaseView、BaseModel等等。封装可以让我们更多地复用代码，少写重复代码。但是封装是不是百利而无一害呢？当然不是，封装有时候会牺牲代码的灵活性。比如在基类里定义了太多类的行为，子类想要修改就只能复写。但是可能一个地方例外，所以地方都变得不一样，封装的太死的话很多地方都要重写。因此基类的东西应该只有确定所有子类都一致的东西，不要试图把所有的东西都塞到基类里。我当时所在的一个项目用的[mvp‑dagger](https://github.com/googlesamples/android-architecture/tree/todo-mvp-dagger/)架构，新建一个Fragment大概要添加/修改七八个类，不得不说真的挺费劲的。

毫无疑问，MVP给Android带来了代码分层的概念，确实是一个很好的实践。但是在使用MVP之前先考虑一下自己能不能接受一个页面至少要写四五个类- -

### 最“流行”的MVVM（Model-View-ViewModel）
为啥流行加了双引号呢，因为在Android领域还是MVP最流行，而在Javascript前端领域，Angular、Vue、React都是MVVM架构。所以MVVM是真的火到爆炸。至于为什么Android MVVM还不是很流行呢（起码我身边很多同行都没用过，我也是今年才真正用过）？之前和微信群里的一个网友讨论过，我觉得他说的很有道理：前几年Android开发火热的时候MVP架构应运而生，人人面试都要谈MVP。而今更新一代的MVVM架构出来后，Android开发市场早已冷却，大家没有动力去切换新的设计模式，毕竟哪怕啥架构模式也没有不也一样能开发大型项目么？

MVP和MVVM有啥区别呢？我也看过很多文章，不过talk is cheap，看再多还是要真正去实践才能感受MVVM架构的好用之处，也明白了为什么三大JS前端框架都用这种架构。

MVP和MVVM区别就是P和VM区别。Presenter层是把UI无关的逻辑自己处理掉，最后把处理完的数据传输给View层展示；ViewModel是把Model和View层关联起来的一个中间层，让View层可以响应式地展示数据。在我看来，VM的核心思想就是UI回调。View层通过ViewModel可以直接动态地展示数据，我们不需要在数据改变时去回调UI，这样可以省去非常非常多的UI代码，怪不得前端都喜欢MVVM。

Android MVVM的实现方式主要有两种，一种是给数据加监听，比如使用[LiveData](https://developer.android.com/topic/libraries/architecture/livedata)，在数据变化时执行监听的回调，这样就不需要每次改变数据后再手动地修改UI。第二种就是[DataBinding]{https://developer.android.com/topic/libraries/data-binding/}框架了。在layout xml里面定义Observable数据，然后直接把数据塞给控件。通常我们会把要展示的最终数据封装到ViewModel层代码里。而且在xml里也能对数据进行简单的操作，具体数据操作大家在[这里](https://developer.android.com/topic/libraries/data-binding/expressions)查到。

MVP并没有真正减少代码，只是把代码做了分层。而MVVM不但让代码分层的更加干净，还减少了大量的UI回调代码。因此用过MVVM之后我就爱上了它，相信你也一样！

### 关于Model层
从写MVP代码开始我就在想，Model层到底是个啥？很多人把Model层翻译为数据层，可是Model是模型的意思啊。Model是不是只包括Model类，还是包含数据增删改查所有代码，我一直在想这个问题。而且我问过不少人看过不少文章，各种看法都有。

在上面的googlesamples，很多架构的M层是包含数据的获取的，并且会封装到XXXRepository类里。这样做的好处就是保证数据来源的唯一性(前端数据层框架Redux三大原则第一条就是Single source of truth)。而在SpringMVC里面，pojo数据的增删改查被封装到dao层，业务处理会封装到service层。因此大型架构的数据的操作基本上都会进行封装，这点没有疑问。至于这些repository、dao、service层属于Model层还是Presenter层还是另外一个层面，可以看看[Wiki上关于MVC的定义](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)：

**The *model* is the central component of the pattern. It is the application's dynamic data structure, independent of the user interface.It directly manages the data, logic and rules of the application.**

从这段话可以看出来dao层，repository层确实属于Model层，至于更进一步的业务处理，则不属于Model层（比如在MVP里就是P层）。

最后欢迎大家多多交流，不吝指点。
