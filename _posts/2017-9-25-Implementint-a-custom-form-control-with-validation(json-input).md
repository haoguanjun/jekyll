# 实现自定义的表单控件

本文介绍如何实现带有验证，且与 ngModel 和 Reacative Forms 兼容的表单控件。

作为示例，我们将创建一个自定义的 textarea 组件，其验证输入用户的输入是否是有效的 Json 串。
通过验证之后的结果通过绑定显示在表单之外，它仅在用户输入有效的情况下更新。

![](https://github.com/haoguanjun/jekyll/raw/master/images/1-implement-custom-form-control.gif)

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

表单控件可以选择实现控件值访问器，启用读取值和监听值的变化。该接口用于 NgModel 和 FormControlName 指令。Angular 使用称为 [DefaultValueAccessories](https://angular.io/docs/ts/latest/api/forms/index/DefaultValueAccessor-directive.html)  的标准处理接口，用于 Input、checkbox 和标准的浏览器元素，我们需要对自定义控件添加自己的实现。

满足接口需要实现以下方法：

```javascript
    /**
     * Write a new value to the element.
     */
    writeValue(obj: any): void;
    /**
     * Set the function to be called 
     * when the control receives a change event.
     */
    registerOnChange(fn: any): void;
    /**
     * Set the function to be called 
     * when the control receives a touch event.
     */
    registerOnTouched(fn: any): void;
```

根据接口的[文档](https://angular.io/docs/ts/latest/api/forms/index/ControlValueAccessor-interface.html)，我们需要使用 writeValue（obj: any) 初始化值，然后使用 registerOnChange(fn: any) 注册 changes。最后一个方法处理 touch 事件，我们不需要它，但是为了满足接口，我们仅使用一个空的代码块。

下面是我们在组件中的实现。
```javascript
import { Component, Input, forwardRef } from '@angular/core';
import { ControlValueAccessor } from '@angular/forms';
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
export class JsonInputComponent implements ControlValueAccessor {
    private jsonString: string;
    private parseError: boolean;
    private data: any;
    
    // the method set in registerOnChange, it is just 
    // a placeholder for a method that takes one parameter, 
    // we use it to emit changes back to the form
    private propagateChange = (_: any) => { };

    // this is the initial value set to the component
    public writeValue(obj: any) {
        if (obj) {
            this.data = obj;
            // this will format it with 4 character spacing
            this.jsonString = 
                    JSON.stringify(this.data, undefined, 4); 
        }
    }
    // registers 'fn' that will be fired when changes are made
    // this is how we emit the changes back to the form
    public registerOnChange(fn: any) {
        this.propagateChange = fn;
    }
    // not used, used for touch input
    public registerOnTouched() { }

    // change events from the textarea
    private onChange(event) {
      
        .....
        // update the form
        this.propagateChange(this.data);
    }
}
```
总结一下修改内容：
1. 确认组件实现了接口 ControlValueAccessor
2. 通过将值转换为字符串，并格式化来初始化 textarea 的值。
3. 注册我们的 changs 函数以触发 changs
4. 更新 onChange 来使用更新后的值调用注册的函数

该接口提供被其它组件调用的契约。所以我们需要作为值访问器注册我们的组件，以便在表单中使用时， Angular 知道它可以调用这些方法。

### 注册提供器

由于我们的组件实现了接口 ControlValueAccessor ，然后，我们需要把它作为一个提供器注册。

如下完成。

```javascript
import { Component, Input, forwardRef } from '@angular/core';
import { 
    ControlValueAccessor, 
    NG_VALUE_ACCESSOR 
} from '@angular/forms';
@Component({
    selector: 'json-input',
    template:
        `
        <textarea
          [value]="jsonString" 
          (change)="onChange($event)" 
          (keyup)="onChange($event)">
        
        </textarea>
        `,
    providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => JsonInputComponent),
      multi: true,
    }      
})
export class JsonInputComponent implements ControlValueAccessor
{
   ...
}
```

我们对类 JsonInputComponent 提供 NG_VALUE_ACCESSOR。我们需要使用 forwardRef() ，因为此刻组件还未定义。最后，我们使用 *multi: true* ，因为 NG_VALUE_ACCESSOR 可以对同样的组件有多个提供器注册。这样到验证的时候更为合理。

实际上我们已经完成了，但还没有任何验证。让我们快速完成。

### 实现自定义验证

现在我们需要实现 Validator 接口，它仅仅需要实现一个方法，这个方法很方便地被称为：validate

```javascript
validate(c: AbstractControl): {
   [key: string]: any;
};
```

该方法接收一个表单控件，然后一个验证项目的键值对。如果表单控件是有效的，它返回 null。

在我们这里，我们如下实现。

```javascript
    // returns null when valid else the validation object 
    // in this case we're checking if the json parsing has 
    // passed or failed from the onChange method
    public validate(c: FormControl) {
        return (!this.parseError) ? null : {
            jsonParseError: {
                valid: false,
            },
        };
    }
```

在这里，我们仅仅检查 parseError 对象，因为我们知道当我们在更新文本的时候，JSON 是否被正确解析。

如同我们注册 ControlValueAccessor 一样，我们需要作为提供器注册这个验证器。

```javascript
import { Component, Input, forwardRef } from '@angular/core';
import { 
    ControlValueAccessor, 
    NG_VALUE_ACCESSOR, 
    NG_VALIDATORS, 
    FormControl, 
    Validator 
} from '@angular/forms';
@Component({
    selector: 'json-input',
    template:
        `
        <textarea
          [value]="jsonString" 
          (change)="onChange($event)" 
          (keyup)="onChange($event)">
        
        </textarea>
        `,
    providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => JsonInputComponent),
      multi: true,
    },
    {
      provide: NG_VALIDATORS,
      useExisting: forwardRef(() => JsonInputComponent),
      multi: true,
    }        
})
export class JsonInputComponent 
    implements ControlValueAccessor, Validator {
    ...
}
```

## Question
* 注册回调的 registerOnChange 方法仅仅能支持注册一个回调方法。
* ControlValueAccessor 用于与 ngForm 配合

See also:
* [Angular - Implementing a custom form control with validation](https://medium.com/@tarik.nzl/angular-2-custom-form-control-with-validation-json-input-2b4cf9bc2d73){:_target="_blank"}
