# Rx.js 版本演变
Rx.js 有两个主要版本：v4 和 v5，操作符有了许多变化。
[Migrating from RxJS 4 to 5](https://github.com/ReactiveX/rxjs/blob/master/MIGRATION.md){:target="_blank"} 说明了详细的变化内容。5.0 的新版本有三个基本目标：
* 更好的性能
* 更好的调试支持
* 兼容 [ES7 Observable Spec](https://github.com/zenparsing/es-observable){:target="_blank"}


例如，Observer 接口发生了变化：
* observer.onNext(value) -> observer.next(value)
* observer.onError(err) -> observer.error(err)
* observer.onCompleted() -> observer.complete()
所以，原来的 subject.onNext("hi") 现在变成了 subject.next("hi").
而原来的 dispose() 则成为了 unsubscribe

### See also
* [RxJS 5 in GitHub](https://github.com/ReactiveX/rxjs){:target="_blank"}
* [RxJS 4 不推荐](https://github.com/Reactive-Extensions/RxJS){:target="_blank"}
* [RxJS WebSite](http://reactivex.io/rxjs/){:target="_blank"}
* [RxJS Manual](http://reactivex.io/rxjs/manual/index.html){:target="_blank"}
* [RxJS 规范](http://reactivex.io/documentation/contract.html){:target="_blank"}
* [ReactiveX/RxJava 文档中文版](https://www.gitbook.com/book/mcxiaoke/rxdocs/details){:target="_blank"}
* [RxViz - Observable 可视化工具](https://rxviz.com/){:target="_blank"}
* [RxVision - Observable 可视化工具](http://jaredly.github.io/rxvision/){:target="_blank"}
* [RxMarbles - Observable 可视化工具](http://rxmarbles.com/){:target="_blank"}
* [The introduction to Reactive Programming you've been missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754){:target="_blank"}




