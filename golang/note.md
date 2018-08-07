## Note

* range会复制目标数据，受直接影响的是数组，可改用数组指针或切片类型
* 字典切片都是引用类型，拷贝开销低
* 编译器会进行逃逸分析将栈上变量分配在堆上
* go build -gcflags "-l -m" 禁用函数内联(-l)，输出优化信息(-m)
* []byte和string的头部结构相同可以通unsafe.Pointer来互相转换
* golang在编译器层面会对[]byte转换为string进行map查询的时候进行优化，避免string拷贝
* golang在编译器层面会对string转换[]byte，进行for range迭代时，直接取字节赋值给局部变量
* string.Join 一次性分配内存等同于bytes.Buffer先Grow事先准备足够的内存
* slice类型不支持比较操作，仅能判断是否为nil，
* slice的len、cap、底层数组的长度，当超过cap的时候会按照cap进行2倍的扩容(并非总是2倍，对于较大的slice则是1/4)
* 不能对nil字典进行写操作，但是却能读
* 字典对象本身就是指针包装的，传参的时候无须再次取地址
* 对于海量小对象应该直接用值拷贝，而非指针，这有助于减少需要扫描的对象数量，大幅缩短垃圾回收时间
* 空结构都指向runtime.zerobase
* 不能将基础类型和其对应的指针类型同时匿名嵌入到结构中(本质上两者的隐式名字相同)
* 如何选择方法的receiver的类型
    1. 要修改实例的状态用`*T`
    2. 无须修改状态的小对象或固定值，建议用`T`
    3. 大对象建议用`*T`，以减少复制成本
    4. 引用类型、字符串、函数等指针包装对象，直接用T
    5. 若包含了`Mutex`等同步字段，用`*T`，避免因复制构造成锁操作而无效
    6. 其他无法确定的情况都用`*T`
* 方法集，通过reflect.TypeOf拿到反射的对象，然后通过NumMethod可以得到方法集的数量，通过Method拿到方法对象，通过方法对象的
  Name方法拿到方法名，通过Type方法拿到方法的声明
    1. 类型T的方法集包含了所有receiver T的方法
    2. 类型`*T`方法集包含了所有`receiver T + *T`方法
    3. 匿名嵌入S，T方法集包含了所有receiver S的方法
    4. 匿名嵌入`*S`，T方法集包含了所有`receiver S + *S`方法
    5. 匿名嵌入`*S`或S,`*T`方法集包含了所有`receiver S + *S`的方法
* Method Expression 和Method Value
* 接口通常以er作为后缀名
* 超集接口可以隐式转换为子集
* 将对象赋值给接口的变量时，会复制该对象，解决办法就是将对象指针赋值给接口。
* 只有将接口变量内部的两个指针(itab，data)都为nil时，接口才等于nil
* 默认协程的堆栈大小时2KB，可以扩大。
* rumtime.GOMAXPROCS，runtime.NumCPU
* 一次性的事件使用close来触发，连续或多样性事件，可传递不同数据标志来实现，还可以使用sync.Cond来实现单播火广播事件
* 向已关闭的通道发送数据会引发panic，从已关闭的通道接收数据，返回已缓冲数据或零值，无论收发，nil通道都会阻塞，重复关闭通道会引发panic。
* 通常使用类型转换来获取单向通道，并分别赋予操作双方，不能在单向通道上做逆向操作，同样close不能作用于接收端，无法将单向通道重新转换回去。
* 无论是否执行reciver，所有延迟调用都会被执行
* 连续调用panic，仅最后一个会被recover捕获
* `runtime.GC()` 主动垃圾回收、`GODEBUG="gctrace=1,schedtrace=1000,scheddetail=1"`
* `sync.Mutex`不能复制会导致失效，所以当包含了`sync.Mutex`的匿名字段的时候，需要将其实现为指针型的receiver，或者嵌入`*sync.Mutex`
* 包内每一个源码文件都可以定义一到多个初始化函数，但是编译器不保证执行次序，只能保证所有的初始化函数都是单一线程执行的，且仅执行一次
* 所有保存在internal目录下的包(包括自身)仅能被其父目录下的包(含所有层次的子目录)访问
* `io.MultiWriter`可以接收多个writer(只要实现了`io.writer`即可)，通过`io.MultiWriter`可以将数据写入到多个writer中
* 执行一个shell命令(exec.Command)并拿到结果
  1. `cmd.CombinedOutput`
  2. `cmd.Stdout` 和 `cmd.Stderr`
  3. `cmd.StdoutPipe` 和 `cmd.StderrPipe`
  4. `cmd.Env`设置程序执行的环境变量