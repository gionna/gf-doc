# gflock

文件锁常用于多进程的互斥锁操作，类似于单进程中的```sync.Mutex```机制，当一个进程对指定文件锁进程```Lock```之后，其他进程将会被阻塞等待，直到该文件锁被同一个进程```Unlock```(或者对应进程退出)。同时，gflock也支持类似于```sync.RWMutex```读写文件锁特性，以及```TryLock/TryRLock```特性。

使用方式：
```go
import "gitee.com/johng/gf/g/os/gflock"
```

方法列表：[godoc.org/github.com/johng-cn/gf/g/os/gflock](https://godoc.org/github.com/johng-cn/gf/g/os/gflock)
```go
type Locker
    func New(file string) *Locker
    func (l *Locker) IsLocked() bool
    func (l *Locker) Lock()
    func (l *Locker) Path() string
    func (l *Locker) RLock()
    func (l *Locker) RUnlock()
    func (l *Locker) TryLock() bool
    func (l *Locker) TryRLock() bool
    func (l *Locker) UnLock()
```
方法比较实用也很简单，我们这里来展示一个```Lock/Unlock```的示例。

```go
package main

import (
    "time"
    "gitee.com/johng/gf/g/os/glog"
    "gitee.com/johng/gf/g/os/gproc"
    "gitee.com/johng/gf/g/os/gflock"
)

func main() {
    l := gflock.New("demo.lock")
    l.Lock()
    glog.Printfln("locked by pid: %d", gproc.Pid())
    time.Sleep(3*time.Second)
    l.UnLock()
    glog.Printfln("unlocked by pid: %d", gproc.Pid())
}
```
我们将这个程序运行两次，会生成两个进程，进程ID分别为```25694```及```25737```(通过```ps```命令或者进程管理器查看)。两个进程都会操作同一个文件锁```demo.lock```文件，但是只有第一个进程```25694```得到了该文件锁操作权限，第二个进程```25737```只有等待该文件锁释放之后才能进一步操作，因此被阻塞。程序中的```Sleep```操作正是为了演示这一阻塞结果而特意设定的。当第一个进程```25694```释放文件锁之后，第二个进程```25737```将会立即开始执行。最终的执行结果如下（注意打印时间的差异）：

进程1（```25694```）：
```shell
2018-05-18 13:51:31.191 locked by pid: 25694
2018-05-18 13:51:34.191 unlocked by pid: 25694
```

进程2（```25737```）：
```shell
2018-05-18 13:51:34.191 locked by pid: 25737
2018-05-18 13:51:37.191 unlocked by pid: 25737
```