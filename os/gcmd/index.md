# gcmd

`gcmd`模块提供了对命令行参数、选项的读取功能，以及对应参数的回调函数绑定功能。

使用方式：
```go
import "gitee.com/johng/gf/g/os/gcmd"
```

## 参数/选项读取

假如执行命令如下：
```
./binary start daemon --timeout=60 --logpath=/home/www/log
```
1. 获取执行参数start、daemon
```go
// start
gcmd.Value.Get(0)
// daemon
gcmd.Value.Get(1)
```

参数获取是从索引0开始，类似数组参数索引形式进行获取。

1. 获取执行选项timeout、logpath
```go
// timeout
gcmd.Option.Get("timeout")
// logpath
gcmd.Option.Get("logpath")
```

执行选项通过给定键名获取，并且执行选项的格式可以是```--键名=键值```(两个"-"符号)，也可以是```-键名=键值```(一个"-"符号)。


## 回调函数绑定

对于命令行的可执行程序来讲，需要根据执行参选定位到对应的入口函数进行处理，gcmd提供了以下方法来实现：
```go
func AutoRun() error
func BindHandle(cmd string, f func()) error
func RunHandle(cmd string) error
```

1. BindHandler绑定执行参数对应的回调函数，执行参数索引为0，回调函数参数为空。
2. RunHandler运行指定执行参数的回调函数；
3. AutoRun根据执行参数[0]自动运行对应注册的回调函数； 










