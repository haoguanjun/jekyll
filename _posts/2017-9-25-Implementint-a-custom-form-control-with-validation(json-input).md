# 实现自定义的表单控件

本文介绍如何实现带有验证，且与 ngModel 和 Reacative Forms 兼容的表单控件。

作为示例，我们将创建一个自定义的 textarea 组件，其验证输入用户的输入是否是有效的 Json 串。
通过验证之后的结果通过绑定显示在表单之外，它仅在用户输入有效的情况下更新。

实现步骤
* 创建组件
* 实现控件值访问器接口
* 注册提供器
* 实现自定义验证
* 使用控件

## 创建控件

### 设置控件

创建一个带有一些功能的空间，如下所示：

```javascript
import { Component } from '@angular/core';
@Component({
    selector: 'json-input',
    template:
        `
        <textarea
          [value]="jsonString" 
          (change)="onChange($event)" 
          (keyup)="onChange($event)">
        
        </textarea>
        `     
})
export class JsonInputComponent {
    
    private jsonString: string;
    private parseError: boolean;
    private data: any;
    // change events from the textarea
    private onChange(event) {
      
        // get value from text area
        let newValue = event.target.value;
        try {
            // parse it to json
            this.data = JSON.parse(newValue);
            this.parseError = false;
        } catch (ex) {
            // set parse error if it fails
            this.parseError = true;
        }
    }
}
```

我们监听两个事件： change 和 keyup，以更新模型和进行适当的验证。
为了使其可用，我们必须实现接口 *ControlValueAccessor*。

### 实现控件值访问器

接口 [ControlValueAccessor](https://angular.io/docs/ts/latest/api/forms/index/ControlValueAccessor-interface.html) 提供将值写入到浏览器原生 DOM 的接口。

表单控件可以选择实现控件值访问器，启用读取值和监听值的变化。该接口用于 NgModel 和 FormControlName 指令。Angular 使用称为 DefaultValueAccessories  标准的处理









See also:
* [Angular - Implementing a custom form control with validation](https://medium.com/@tarik.nzl/angular-2-custom-form-control-with-validation-json-input-2b4cf9bc2d73){:_target="_blank"}
