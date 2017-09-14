---
layout: post
---
# 创建属性指令
让我们创建一个简单的按钮，支持将用户导航到其它页面。

```typescript 
@Component({
   selector: 'app-visit-rangle',
   template: `
   <button
      type="button"
      (click)="visitRangle()">
      Visit Rangle
   </button>
`
})
export class VisitRangleComponent {
   visitRangle() {
      location.href = 'https://rangle.io';
   }
}
```

我们是有礼貌的，所以不是直接将用户导航到新页面，我们将先询问用户进行确认。

首先创建一个属性指令，并将其链接到按钮上。

```typescript
@Directive({
   selector: `[appConfirm]`
})
export class ConfirmDirective {
   @HostListener('click', ['$event'])
   confirmFirst(event: Event) {
      return window.confirm('Are you sure you want to do this?');
   }
}
```

指令使用定义在类上的 `@Directive` 装饰器，以及定义一个选择器来创建。对于指令，选择器的名字必须是驼峰格式，并使用中括号 [] 包围来指定其使用属性绑定。我们还使用 `@HostListener` 装饰器来监听其连接的组件或者元素的事件。在本例中，我们监听 `click` 事件，并使用 $event 关键字来传递事件的详细信息。然后，我们将这个指令连接到先前创建的按钮上。

```typescript
template: `
   <button
      type="button"
      (click)="visitRangle()"
      appConfirm>
         Visit Rangle
   </button>
`
```

注意，按钮现在并不如我们所预期的工作。这是因为在我们监听按钮的点击事件来显示确认对话框的时候，组件的点击处理在指令的点击处理之前执行，它们之间没有协调。我们需要重写指令以便与组件的点击处理器协作。

```typescript
@Directive({
   selector: `[appConfirm]`
})
export class ConfirmDirective {
   @Input() appConfirm = () => {};
   @HostListener('click', ['$event'])
   confirmFirst() {
      const confirmed = window.confirm('Are you sure you want to do this?');
      if(confirmed) {
         this.appConfirm();
      }
   }
}
```

现在，我们希望指定在确认之后的行为，如我们在组件之上的输入绑定。我们使用指令名来完成这个绑定，组件变成这样：

```html
<button
   type="button"
   [appConfirm]="visitRangle">
   Visit Rangle
</button>
```

现在按钮如我们预期工作。其实我们还可能希望定制确认提示信息。为此我们做另一个绑定。

```typescript
@Directive({
   selector: `[appConfirm]`
})
export class ConfirmDirective {
   @Input() appConfirm = () => {};
   @Input() confirmMessage = 'Are you sure you want to do this?';
   @HostListener('click', ['$event'])
   confirmFirst() {
      const confirmed = window.confirm(this.confirmMessage);
      if(confirmed) {
         this.appConfirm();
      }
   }
}
```

现在指令有了一个新的输入属性，表示确认提示框中的信息，我们将其传递给 `window.confirm` 调用。为使用这个新输入属性，我们为按钮添加另一个绑定。

```html
<button
   type="button"
   [appConfirm]="visitRangle"
   confirmMessage="Click ok to visit Rangle.io!">
   Visit Rangle
</button>
```

现在，我们拥有了一个在导航到新页面之前的可定制确认信息的按钮。

## 监听元素宿主

监听宿主，也就是指令连接到的 DOM 元素，是指令扩展组件或者元素行为的基本方式。刚刚，我们看到一个常见用例。

```typescript
@Directive({
   selector: '[appMyDirective]'
})
class MyDirective {
   @HostListener('click', ['$event'])
   onClick() {}
}
```

我们也可以响应外部的事件，比如来自 `window` 或者 `document`, 通过为监听器添加目标来实现。

```typescript
@Directive({
   selector: `[appHighlight]`
})
export class HighlightDirective {
   constructor(private el: ElementRef, private renderer: Renderer) { }
   @HostListener('document:click', ['$event'])
   handleClick(event: Event) {
      if (this.el.nativeElement.contains(event.target)) {
         this.highlight('yellow');
      } else {
         this.highlight(null);
      }
   }
   highlight(color) {
      this.renderer.setElementStyle(this.el.nativeElement, 'backgroundColor', color);
   }
}
```

    尽管不常用，如果我们希望在组件的宿主元素上注册监听器，还是可以使用 `@HostListener`

## 宿主元素

宿主元素的概念应用于指令和组件两者。

对于指令，此概念相当直接了当。使用指令的模板元素就是宿主元素。如果我们如下实现 `HighlightDirective` 指令。

```html
<div>
   <p appHighlight>
      <span>Text to be highlighted</span>
   </p>
</div>
```

标记 `<p>` 将是宿主元素。如果我们使用自定义的 `TextBoxComponent` 作为宿主，代码将如下所示：

```html
<div>
   <app-my-text-box appHighlight>
      <span>Text to be highlighted</span>
   </app-my-text-box>
</div>
```

对于组件，宿主元素是您通过 `selector` 在组件配置中定义的标记。对于上边示例中的 `TextBoxComponent` 组件，组件类的宿主元素就是 `<app-my-text-box>`。

## 通过指令设置属性

我们可以通过属性指令使用 `@HostBinding` 装饰器来改变宿主元素的属性。

`@HostBinding` 装饰器允许我们可编程地设置指令宿主元素的属性值。工作方式类似模板中定义的属性绑定，除了它专门针对宿主元素。绑定在每个变更检测周期中被检查，如果期望的话，它可以动态改变。

例如，我们希望创建一个指令，可以在按钮被按下时动态添加样式 class。这可能类似如下示例：

```typescript
import { Directive, HostBinding, HostListener } from '@angular/core';

@Directive({
   selector: '[appButtonPress]'
})
export class ButtonPressDirective {
   @HostBinding('attr.role') role = 'button';
   @HostBinding('class.pressed') isPressed: boolean;
   @HostListener('mousedown') hasPressed() {
      this.isPressed = true;
   }
   @HostListener('mouseup') hasReleased() {
      this.isPressed = false;
   }
}
```

需要注意到，两个示例中的 `@HostBinding` 我们都为希望影响的属性传递了一个字符串值。如果我们没有为装饰器提供字符串，类的成员名称将被使用。

在第一个 `@HostBinding` 中，我们静态设置 `button` 的 role 属性。对于第二个示例，当 `isPressed` 为真时，`pressed` 将被应用。

    提示：虽然不常用到，在需要的时候，`@HostBinding` 可以用于组件。






