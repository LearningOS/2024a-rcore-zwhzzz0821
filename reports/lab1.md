# CH3 分时多任务系统
先看看get_time()函数，在维护Task_info的时候我们需要获取到系统当前的时间戳：

该函数就是调用了一下函数`sys_get_time`，该函数的功能是调用系统调用：系统调用号：`SYSCALL_GETTIMEOFDAY`，来获取当前时间，存入到设计好的结构体TimeVal中。

接下来讲一下我的思路，这部分可能会频繁更改：
首先看看任务的一些信息存储在哪里，可以看到TCB中存储了任务的基本信息，包括cx和TaskStatus，我们可以尝试在里面加入我们需要统计的信息：系统调用次数，时间。那么还要考虑一下，什么时候去维护这些信息：

可以看看TaskManager这部分的内容：

我们可以看到，TaskManager存储了所有的TCB，同时包括了当前运行的任务 current_task，那么如果我们加入time信息，是不是可以在切换任务的时候调用gettime来维护time信息，也就是：**获取第一次被调度的时间**。

所以我们考虑，每次trap_handle的时候去TaskManager拿到当前运行任务的TCB，然后就可以维护task_call_time了。

所以我们在TaskManager中增加更新函数（TaskManager是全局变量）：

维护也很简单，直接在trap_handle中调用函数就行：这里cx是trap上下文，x[17]存储系统调用号，x[10-12]存储调用参数

然后，就是维护time，根据文档中的要求，time指的是：系统调用时刻距离任务第一次被调度时刻的时长，任务什么时候会被第一次调度呢，答案也就在TaskManager中。

分别在first run和run next中增加代码：

然后就到了系统调用sys_task_info的实现了，那也就比较简单了，获取tcb，然后获取time的差值即可：

看不懂这里get_time为什么这么做，他对sec取了前四位然后×1000 + usec / 1000，模仿试试吧。然后就过了（


最后一个测试点挂了一下，我们看注释的提示：`// 想想为什么 write 调用是两次`，然后我想到println也会去调用sys_write，然后我又在测试里加了个println来debug，去了就过了。