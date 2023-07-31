事件时间与处理时间

-   时间处理滞后的原因
    -   排队，网络故障，消费者或消费代理的崩溃，重新处理过去的事件



用谁的时钟

-   用户控制的设备上的时钟通常是不可行的
-   记录三个时间戳
    -   根据设备时钟，记录事件发生的时间
    -   根据设备时钟，记录将时间发送到服务器的时间
    -   根据服务器时钟，记录服务器收到时间的时间



窗口类型

-   轮转窗口
    -   长度固定，每个事件都属于一个窗口
    -   A~B时间段的被分组到a窗口
    -   B~C时间段的被分组到b窗口
-   跳跃窗口
    -   长度固定，允许窗口重叠以提供一些平滑过渡
    -   窗口长度5min，跳跃值1min
    -   A~A+5min
    -   A+1min~A+5min+1min
-   滑动窗口
    -   包含在彼此的某个间隔内发生的所有事件
-   会话窗口
    -   没有固定的持续时间
    -   将同一用户在时间上紧密相关的所有事件分组在一起，一旦用户在一段时间内处于非活动状态，窗口结束



Pp. 439-443