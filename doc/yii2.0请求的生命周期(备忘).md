## yii2.0请求的生命周期(备忘)

```
1,用户向入口脚本 web/index.php 发起请求。
2,入口脚本加载应用配置 并创建一个应用实例去处理请求。
3,应用通过请求组件 解析请求的路由。
4,应用创建一个控制器实例去处理请求。
5,控制器创建一个动作实例并针对操作执行过滤器。
6,如果任何一个过滤器返回失败，则动作取消。
7,如果所有过滤器都通过，动作将被执行。
8,动作会加载一个数据模型，或许是来自数据库。
9,动作会渲染一个视图，把数据模型提供给它。
10,渲染结果返回给响应组件。
11,响应组件发送渲染结果给用户浏览器。
```

![20170309120603360.png][1]


  [1]: https://www.skylway.com/usr/uploads/2019/09/1859649879.png