# PlantUml使用



## 活动图

### 简单活动图

活动标签以冒号开始，以分号结束，活动默认按照定义的顺序进行自动连接。

> @startuml
>
> :Hello worl;
>
> :test
>
> @enduml

### 开始/停止/结束

可以是 `start` 和 `stop` / `end` 关键字来表示一个图的开始和结束

> @startuml
>
> start
>
> :Hello worl;
>
> :test
>
> end
>
> @enduml

### ### 条件判断

使用关键字 `if` ,  `then` 和 `else` 设置分支测试。标注文字则放在括号中



https://plantuml.com/zh/activity-diagram-beta

## 参考链接

[官方链接](https://plantuml.com/zh/)