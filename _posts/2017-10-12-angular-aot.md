Angular AOT

在 [angular-seed](https://github.com/mgechev/angular-seed) 中加入了 Ahead-of-Time(AoT) 之后，我收到了许多关于新特性的问题。为了回答这些问题，我们从下列问题开始：
* 为什么在 angular 中需要编译
* 什么需要编译？
* 如何被编译
* 何时编译？Just-in-Time(JiT) vs Ahead-of-Time(AoT)
* 从 AoT vs 得到什么
* AoT 如何编译？
* 使用 AoT vs Jit 我们失去什么？

## 为什么在 Angular 中 需要编译？
简单的回答就是：我们需要编译使 Angular 应用更为高效。对于高效，我通常指性能改进，但也与消耗的带宽有关。
Angular JS1.x 对渲染和变更检测都使用动态方式。例如，Angular JS1.x 编译器相当通用。它通过一系列动态计算来支持任意模板。尽管在通常情况下很有用，JavaScript VM 由于其本质上的动态而需要与底层的优化计算做斗争。因为 VM 对提供脏检查逻辑上下文的对象状况并不知情，它的内联缓存会导致执行速度降低的很多遗漏。

对 Angular 来说，采用了不同的方式来处理。不是对不同的组件采用相同的逻辑来执行渲染和变更检测，框架在运行时、或者构建时生成 VM 友好的代码
。这使得 JavaScript VM 可以通过属性访问缓存，更快地执行变更检测和渲染逻辑。

例如，我们看一下下列代码：

```javascript
// ...
Scope.prototype.$digest = function () {
  'use strict';
  var dirty, watcher, current, i;
  do {
    dirty = false;
    for (i = 0; i < this.$$watchers.length; i += 1) {
      watcher = this.$$watchers[i];
      current = this.$eval(watcher.exp);
      if (!Utils.equals(watcher.last, current)) {
        watcher.last = Utils.clone(current);
        dirty = true;
        watcher.fn(current);
      }
    }
  } while (dirty);
  for (i = 0; i < this.$$children.length; i += 1) {
    this.$$children[i].$digest();
  }
};
// ...
```

这段代码来自我的[轻量级 AngularJS 1.x 实现](https://github.com/mgechev/light-angularjs/blob/master/lib/Scope.js#L61-L79)。在整个 scope 树上执行深度优先的搜索，在我们的绑定中查找变更。这种方式用于任何指令。但是，它显然慢于特定的指令代码：

```javascript
// ...
var currVal_6 = this.context.newName;
if (import4.checkBinding(throwOnChange, this._expr_6, currVal_6)) {
    this._NgModel_5_5.model = currVal_6;
    if ((changes === null)) {
        (changes = {});
    }
    changes['model'] = new import7.SimpleChange(this._expr_6, currVal_6);
    this._expr_6 = currVal_6;
}
this.detectContentChildrenChanges(throwOnChange);
// ...
```

该段来自 [angular-seed project](https://github.com/mgechev/angular-seed) 生成的 *detectChangesInternal* 方法。这里对不同的绑定使用直接的属性访问，使用更为高效的方式与新值进行比较。一旦发现值发生了变化，只更新受到影响的 DOM 元素。

对于“为什么我们需要编译” 这个问题，与“需要编译什么？” 一样，我们需要将组件的模板编译成 JavaScript 类。这些类含有检测绑定变更的方法和渲染用户界面的方法。这样我们不需要将底层平台耦合起来。从另一个角度来说，对于同样的 AoT 编译的组件，我们可以有不同的渲染实现，而不需要修改任何代码。例如，上层组件可以在 NativeScript 中渲染，一旦渲染器理解传递的参数。

## Just in Time(JiT) vs Ahead of Time(AoT)

这里回答“何时编译介入？” 问题。

非常棒的是 Angular 编译器既可以在运行时（比如用户浏览器中），又可以在构建时（在构建流程中）。这是由于 Angular 的可移植性，我们可以任何平台运行 JavaScript VM , 所以为什么我们不让 Angular 编译器可以同时运行在浏览器中和 node 环境下。

### Just in Time 编译事件流

我们先看一下典型的非 AoT 开发流程
* 使用 TypeScript 开发 Angular 应用
* 使用 tsc 编译应用
* 打包
* 紧缩
* 发布

一旦我们部署应用之后，用户使用浏览器打开它，用户将通过以下步骤（没有强制的 CSP）
* 下载所有的 JavaScript 资源
* Angular 启动
* Angular 进行 JiT 编译流程，为每个组件生成 JavaScript 代码
* 应用开始渲染

### Ahead of Time 编译流程

作为对比，AoT 将通过以下步骤：
* 使用 TypeScript 开发应用
* 使用 ngc 编译应用
   * 使用 Angular 编译器编译模板，生成 TypeScript
   * 将 TypeScript 编译为 JavaScript
* 打包
* 紧缩
* 发布

虽然上面看起来有点复杂，但是用户仅仅需要下列步骤：
* 下载所有资源
* Angular 启动
* 应用渲染

你会看到第三步被丢弃了，这意味着更快更好的用户体验，像 [angular-seed](https://github.com/mgechev/angular-seed) 和 [angular-cli](https://github.com/angular/angular-cli) 将自动处理这些流程。

在 Angular 中，JiT 与 AoT 之间的主要区别在于：
* 编译介入的时间
* JiT 生成 JavaScript 代码，但是，AoT 通常生成 Typescript 代码。

## 深入 AoT 编译

这里我们回答三个问题：
* AoT 编译器生成什么？
* 产品组件的上下文是什么？
* 如何开发即 AoT 友好又封装良好的代码？

我们将快速浏览编译过程，因为我们不会一行行说明 @angular/compiler 中的代码。如果您对语法分析，代码生成感兴趣，您可以看看  [The Angular 2 Compiler](https://www.youtube.com/watch?v=kW9cJsvcsGo) by Tobias Bosch 或 [this slide deck](https://speakerdeck.com/mgechev/angular-toolset-support?slide=69).

Angular 模板编译器接收作为输入的组件和其上下文（我们可以认为上下文是组件树种的一个位置），并生成如下文件：
* *。ngfactory.ts - 下面我们会看这个文件
* *.css.shim.ts - 基于组件的封装模式涉及的 CSS 文件
* *.metadata.json - 当前组件的元数据（或者 NgModule）装饰器。这是一个 JSON 格式的描述的对象，将传递给 @Component 或者 @NgModule 装饰器。

* 是文件名的占位符，对于 hero.component.ts 来说，编译器将生成 hero.component.ngfactory.ts, hero.component.css.shim.ts, hero.component.metadata.json。
*.css.shim.ts 对于我们讨论的内容来说没有太大关系，我们就不看了。如果您想知道 *.metadata.json，您可以看看 [AoT and third-part modules].

### *.ngfactory.ts 内幕

文件 *.ngfactory.ts 包含如下定义
* _View_{COMPONENT}_Host{COUNTER} - 我们成为 *内部的宿主组件*.
* _View_{COMPONENT}{COUNTER} - 我们称为 *内部组件*.

以及两个函数:
* viewFactory_{COMPONENT}_Host{COUNTER}
* viewFactory_{COMPONENT}{COUNTER}

上述的 {COMPONENT} 是组件名称，而 {COUNTER} 则是一个无符号整数。

这些类扩展了 AppView 并实现如下方法：
* createInternal - 渲染组件.
* destroyInternal - 执行清理 (删除事件监听等等).
* detectChangesInternal - 使用内联缓存的优化方法检测变更.

上面的工厂仅仅用来实例化生成的 AppView 。
如前所述，detectChangesInternal 方法包含了 VM 友好的代码。我们看一看编译版本的模板。

```html
<div>{{newName}}</div>
<input type="text" [(ngModel)]="newName">
```

生成的 detectChangesInternal 应该类似如下：

```javascript
// ...
var currVal_6 = this.context.newName;
if (import4.checkBinding(throwOnChange, this._expr_6, currVal_6)) {
    this._NgModel_5_5.model = currVal_6;
    if ((changes === null)) {
        (changes = {});
    }
    changes['model'] = new import7.SimpleChange(this._expr_6, currVal_6);
    this._expr_6 = currVal_6;
}
this.detectContentChildrenChanges(throwOnChange);
// ...
```

我们假设 currVal_6 的值为 3， this_expr_6 的值为 1， 现在方法的执行：
调用 import4.checkBinding(1, 3), 在产品模式，checkBinding 执行如下检查：

```typescript
1 === 3 || typeof 1 === 'number' && typeof 3 === 'number' && isNaN(1) && isNaN(3);
```

上述表达式将返回 false，所以将会保存变更，并更新 NgModel 实例的 model 值。在 detectContentChildrenChanges 之后，将被调用，它将调用所有子内容的 detectChangesInternal 方法。一旦 NgModel 指令发现外部更新了 model 值，它将通过渲染器更新相关元素。

没有什么特别和复杂的地方。

### context 属性

您可能注意到在内部组件中，我们通过属性 this.context 访问属性。
内部组件的 context 属性是我们定义的组件本身的实例。对于如下示例。

```javascript
@Component({
  selector: 'hero-app',
  template: '<h1>{{ hero.name }}</h1>'
})
class HeroComponent {
  hero: Hero;
}
```

this.context 将等于 new HeroComponent()。这意味着在 detectChangesInternal 中，我们不得不访问 this.context.hero.name。这会带来问题，如果我们希望将 TyeScript 作为 AoT 的编译结果，我们必须确认我们只能访问组件中模板使用的公共字段。为什么这样？如前所述，编译器既可以生成 Typescript 又可以生成 JavaScript，因为 TypeSCript 有访问控制符，强制只能访问继承链中的公共属性，在内部组件中，我们不能访问私有的上下文对象属性，所以：

```javascript
@Component({
  selector: 'hero-app',
  template: '<h1>{{ hero.name }}</h1>'
})
class HeroComponent {
  private hero: Hero;
}
```
和

```javascript
class Hero {
  private name: string;
}

@Component({
  selector: 'hero-app',
  template: '<h1>{{ hero.name }}</h1>'
})
class HeroComponent {
  hero: Hero;
}
```

都将在生成 *.ngfactory.ts 的时候抛出异常。第一个例子中，内部组件不能访问 hero 属性，因为它是私有属性。第二个例子中，内部组件不能访问 name ，因为它在 Hero 中是私有的。

### AoT 和封装

好吧，我们仅仅能在模板中绑定公共属性和公共方法。对组件的封装呢？开始不是问题，但是想想一下如下场景：

```javascript
// component.ts
@Component({
  selector: 'third-party',
  template: `
    {{ _initials }}
  `
})
class ThirdPartyComponent {
  private _initials: string;
  private _name: string;

  @Input()
  set name(name: string) {
    if (name) {
      this._initials = name.split(' ').map(n => n[0]).join('. ') + '.';
      this._name = name;
    }
  }
}
```

组件有一个单输入的 name 属性。在 name 属性的设置器内，它计算 _initials 的值。我们可以这样使用组件。

```
@Component({
  template: '<third-party [name]="name"></third-party>'
  // ...
})
// ...
```

在 JiT 模式，Angular 生成 JavaScript 代码，这可以很好的工作。每次 name 值变化且值为 true ，都会更新 _initials 的值。但是，组件 ThirdPartyComponent 并不是 AoT 友好的。您需要做如下修改

```javascript
// component.ts
@Component({
  selector: 'third-party',
  template: `
    {{ initials }}
  `
})
class ThirdPartyComponent {
  initials: string;
  private _name: string;

  @Input()
  set name(name: string) {...}
}
```
这样的话，就可能被用户如下使用：

```javascript
import {ThirdPartyComponent} from 'third-party-lib';

@Component({
  template: '<third-party [name]="name"></third-party>'
  // ...
})
class Consumer {
  @ViewChild(ThirdPartyComponent) cmp: ThirdPartyComponent;
  name = 'Foo Bar';

  ngAfterViewInit() {
    this.cmp.initials = 'M. D.';
  }
}
```

这会导致 ThirdPartyComponent 组件状态的不一致。属性 name 的值为 "Foo Bar"，但是， initials 将会是 "M. D."，替换了 "F. B."。

解决这个问题的根源在于 [Angular 的代码](https://github.com/angular/angular/blob/14ee75924b6ae770115f7f260d720efa8bfb576a/modules/%40angular/common/testing/mock_location_strategy.ts#L26)。在这个例子中，我们希望使我们的代码 AoT 友好，同时还要封装性，我们可以使用 TypeScript annotaion /** @internal */

```javascript
// component.ts
@Component({
  selector: 'third-party',
  template: `
    {{ initials }}
  `
})
class ThirdPartyComponent {
  /** @internal */
  initials: string;
  private _name: string;

  @Input()
  set name(name: string) {...}
}
```

initials 属性将是公共的，但是如果我们使用 tsc 编译这个第三方库，并且设置了 --stripInternal  和 --declarations 标志，initials 属性将从 ThirdPartyComponent 类型定义文件中忽略掉。这样我们就能够在库内部访问，但是限制使用库的用户不能访问。

### 总结 ngfactory.ts

快速总结一下，假设我们有 HeroComponent 组件，对这个组件，Angular 编译器将会生成两个类：
* _View_HeroComponent_Host1 - 内部宿主组件.
* _View_HeroComponent1 - 内部组件.

_View_HeroComponent1 将负责渲染 HeroComponent 的模板，并且，执行变更检测。当执行变更检测的时候，_View_HeroComponent1 将比较当前的 this.context.hero.name 值与以前存储的值。如果值变化了，<h1/> 元素将被更新。这意味着我们需要确认 this.context.hero 和 hero.name 都是公共的。从另一方面来说，_View_HeroComponent_Host1 将负责渲染 <hero-app></hero-app> (宿主元素)，_View_HeroComponent1 负责本身。

## AoT vs JiT - 开发体验

这里我们讨论使用 AoT 和 JiT 的开发体验。

可能最大的区别在于 JiT 总是编译为 JavaScript，所以，内部组件和内部宿主组件都定义成了公共的。这意味着我们的组件的成员都是公共的。所以不会有任何的编译时错误。

在 JiT 模式下，一旦应用启动，我们就已经拥有了根注入器，根组件的所有指令可用了。（他们在 BrowserModule 中，我们在根模块导入所有的模块）。元数据会传递给编译器编译根组件的流程。一旦编译器生成了 JiT 代码，其中就包含了用于生成子组件的所有元数据。它可以生成所有需要的代码，因为它已经不仅知道了在组件树的这个级别上哪些 provider 可用，还有那些指令可见。

这使得编译器知道当它访问模板中元素的时候应该做什么。例如，元素 <bar-baz></bar-baz> 可以基于是否有选择器 bar-baz 存在表示为不同的形式。

这带来一个问题，在构建时，我们如何知道那个指令在组件树的所有级别可访问呢？感谢 angular 的优秀设计，我们可以执行一个静态代码分析来找出来。收集器遍历组件树来抽取每个组件的和 NgModule 的元数据。其中引入了一些我们这里不再讨论的技术。

### AoT 与第三方模块

好的，所以编译器需要元数据来编译模板。假设我们项目使用了第三方的组件库。如果其使用纯 JavaScript 发布，Angular AoT 如何知道组件的元数据呢？它不知道，为了能够编译 AoT 应用，需要引用外部的一个 Angular 库，库需要发布编译器生成的 *.metadata.json。

进一步了解 Angular 编译器，您可以看一下后面的链接，对于创建 AoT Ready 的组件库，您可以看看 [angular/mobile-toolkit](https://github.com/angular/mobile-toolkit/blob/master/app-shell/gulpfile.ts#L52-L54).


## 从 AoT 中我们得到什么？

您可能猜到了，我们通过 AoT 获得性能改进。AoT 的初始渲染性能远快于 JiT 的，因为 JavaScript VM 只需要很少的计算了。我们仅需要在开发阶段一次性编译模板，然后用户得到编译之后的模板。

下图展示了 JiT 的初始渲染时间：

![](http://blog.mgechev.com/images/aot-angular/jit.png)

下图是使用 AoT 的渲染时间：

![](http://blog.mgechev.com/images/aot-angular/aot.png)

另外一个惊奇的是 Angular 编译器不仅可以生成 JavaScript ，还允许在模板中执行类型检查！

因为应用的模板是纯的 JavaScript/TypeScript，我们可以确切知道做什么和在哪里使用。这使得我们可以执行有效的摇树，并在应用的产品代码中删除没有用的的指令和模块。最终我们不需要在产品包中包含 @angular/compiler 模块，因为不需要执行运行时编译了。

注意，对于大型和中型的 AoT 编译后的应用包，通常会比使用 JiT 编译的包变得更大。这是因为由 ngc 生成的 VM 友好代码更为详细地比较类 HTML 模板，包括脏检查逻辑。这种情况下，您可以通过 Angular 路由本地支持的延迟加载来减少应用的尺寸。

在有些情况下，JiT 编译完全不能处理。因为 JiT 使用 eval 在浏览器中执行生成和代码求值。[CSP](https://developer.chrome.com/extensions/contentSecurityPolicy) 和某些特定环境不允许我们对动态代码进行求值。

最后，但不是结束，节能！用户设备执行的更少，因为其接收已经编译完成的代码。这减少了电池的消耗。



See also:
* [Ahead-ofTime Complilation in Angular](http://blog.mgechev.com/2016/08/14/ahead-of-time-compilation-angular-offline-precompilation/)
* [Inline Caches](http://mrale.ph/blog/2012/06/03/explaining-js-vms-in-js-inline-caches.html)
* [2.5X Angular Closure 编译器带来更小的 Angular 应用](http://blog.mgechev.com/2016/07/21/even-smaller-angular2-applications-closure-tree-shaking/)
* [Angular Source Code](https://github.com/angular/angular)
