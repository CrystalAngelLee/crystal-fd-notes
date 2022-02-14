# 交互实现

## meta 标签：自动刷新/跳转

**假设要实现一个类似 PPT 自动播放的效果**，可以通过 meta 标签来实现：

```html
<meta http-equiv="Refresh" content="5; URL=page2.html">
```

上面的代码会在 5s 之后自动跳转到同域下的 page2.html 页面。我们要实现 PPT 自动播放的功能，只需要在每个页面的 meta 标签内设置好下一个页面的地址即可