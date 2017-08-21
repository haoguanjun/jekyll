---
layout: default
title: jekyll-要点
---

### layout
layout 保存在 _layouts 文件夹中，文件的名称即为布局的名称。
比如保存在 _layout 文件夹中的 default.html 即为定义的, 名为 default 的布局文件。
在其它任何页面中可以使用此名称来使用该布局。
例如，在 /index.html 中。

    ---
    layout: default
    title: Index
    ---

### 引用网站中的其它文件
如果我们将自定义的样式表文件 main.css 保存在 /css/main.css 中，在布局文件，或者其它文件中引用时，可以使用变量来引用。
{{site.baseurl}} 表示网站的根目录，例如：

    <link href="{{site.baseurl}}/css/main.css" rel="stylesheet">

### include 包含其它文件
在 layout 中，可以通过 include 将大篇内容拆分为多个小的部分，然后通过 include 组合起来。
每个小片保存在 \_includes 文件夹中，然后通过 {% include 分片名称 %} 来潜入。
例如：

    {% include header.html %}
        <main class="col-12 col-md-9 col-xl-8 py-md-3 pl-md-5 bd-content" role="main">
            {{ content }}
        </main>
    {% include footer.html %}
    
 其中，header.html 和 footer.html 都保存在 /\_includes 文件夹中。


 
