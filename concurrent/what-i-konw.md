# 了解容器和镜像

A thread state. A thread can be in one of the following states:
NEW
A thread that has not yet started is in this state.
RUNNABLE
A thread executing in the Java virtual machine is in this state.
BLOCKED
A thread that is blocked waiting for a monitor lock is in this state.
WAITING
A thread that is waiting indefinitely for another thread to perform a particular action is in this state.
TIMED_WAITING
A thread that is waiting for another thread to perform an action for up to a specified waiting time is in this state.
TERMINATED
A thread that has exited is in this state.
A thread can be in only one state at a given point in time. These states are virtual machine states which do not reflect any operating system thread states.

## 线程引入的开销

与单线程相比

* 调度开销
* 同步开销
