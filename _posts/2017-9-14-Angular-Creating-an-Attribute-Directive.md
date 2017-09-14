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






