title: "Eclipse配置"
date: 2016-01-14 16:37:05
tags: "eclipse"
categories: "其他"
---

### 取消项目JavaScript验证

> [Eclipse 去掉JavaScript Validator](http://www.cnblogs.com/wucg/archive/2012/08/06/2625458.html)

选中一个项目,右键选择**Properties**,弹出的面板中点击**Builders**,然后将**JavaScript Validator**选项去掉，保存

----

### 修改快捷键

> [修改Eclipse快捷键的方法](http://jythoner.iteye.com/blog/313355)

**Windows->Preferences->General->Keys**,，如果有快捷键冲突可以先取消绑定**Remove Binding**，然后点击**Binding**输入框，按入快捷键会自动录入；如果需要在特定条件下触发可以选择**When**条件。

----

### 折叠代码

> [eclipse中函数不能折叠](http://zhidao.baidu.com/link?url=kRvwFeOSiV-7tTaLm8gr-gUQOR1V9n8hEAOkxpX86Q8IlJi05zpe22M7zgvOaQQMvEAD8blccm2fe-ZnEKPK1q)
> [Eclipse中如何折叠代码](http://zhidao.baidu.com/link?url=MlMxLWQ8Xv__5iYR7jmJ3JYdtP8fbNrtuA9E2efcKGGS-EkMQ8Nk_WdYJBzBYncShq24A7wPrJ8SUooO_mSxqK)

在代码页行号左边的空白点击鼠标右键，弹出的菜单选择**Folding**,然后点击第一项**Enable Folding**即可。或者**Windows->Preferences->Java->Editor->Folding**,勾上**Enable Folding**。接下来可以使用快捷键来控制代码折叠了:
`Ctrl+/(小键盘)`：折叠/展开代码
`Ctrl+Shift+/(小键盘)`：折叠当前类代码
`Ctrl+Shift+*(小键盘)`：展开当前类代码