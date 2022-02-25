---
title: "Go Question"
date: 2022-02-25T14:12:22+08:00
draft: false
description: "Go常见面试题"
tags:
- Go
- 面试
---

### Go Questions

#### 1.nil切片和空切片、零切片

nil切片指向的地址为0，而所有创建的空切片的内存地址是存在的，并且是一个固定值。

零切片就是底层数组内部数据全是`零变量`。说白了就是使用`make`初始化之后的切片。

```go
func compareSlice() {
	var s1 []int
	s2 := make([]int,0)
	s4 := make([]int,0)

	fmt.Printf("s1 pointer:%+v, s2 pointer:%+v, s4 pointer:%+v, \n", *(*reflect.SliceHeader)(unsafe.Pointer(&s1)),*(*reflect.SliceHeader)(unsafe.Pointer(&s2)),*(*reflect.SliceHeader)(unsafe.Pointer(&s4)))
	fmt.Printf("%v\n", (*(*reflect.SliceHeader)(unsafe.Pointer(&s1))).Data==(*(*reflect.SliceHeader)(unsafe.Pointer(&s2))).Data)
	fmt.Printf("%v\n", (*(*reflect.SliceHeader)(unsafe.Pointer(&s2))).Data==(*(*reflect.SliceHeader)(unsafe.Pointer(&s4))).Data)
}
// 结果
/**
s1 pointer:{Data:0 Len:0 Cap:0}, s2 pointer:{Data:824633999016 Len:0 Cap:0}, s4 pointer:{Data:824633999016 Len:0 Cap:0}, 
false
true
*/
```

#### 2.字符串转换为byte数组，会发生内存拷贝吗？

严格来说，只要进行了类型的强制转换都会发生内存拷贝。所以说字符串转换为byte数组会发生内存拷贝。go的字符串也为不可变对象，在内存中的实现方式是一个只读的字节数组。字符串要想修改只能先转换为可写的数组，然后在转换为字符串。其数据结构如下：

```go
type StringHeader struct {
	Data uintptr
	Len  int
}
```

用代码展示，可以从结果上看出，不论是从字符串转换为byte数组还是从byte数组转换为字符串，均发生内存拷贝了。

```go
func StringAndByte() {
	s := "hello"
	tempByte := []byte(s)
	s2 := string(tempByte)
	fmt.Printf("s Pointer:%+v\n", *(*reflect.StringHeader)(unsafe.Pointer(&s)))
	fmt.Printf("tempByte Pointer:%+v\n", *(*reflect.SliceHeader)(unsafe.Pointer(&tempByte)))
	fmt.Printf("s2 Pointer:%+v\n", *(*reflect.StringHeader)(unsafe.Pointer(&s2)))
}
// 结果
/**
s Pointer:{Data:12543120 Len:5}
tempByte Pointer:{Data:824633999064 Len:5 Cap:32}
s2 Pointer:{Data:824633999032 Len:5}
*/
```

**不过也有方法可以不用进行内存拷贝实现转换**，实际上，字符串和byte数组的底层结构之间只是少了Cap字段，所以我们可以将`StringHeader` 的地址强转成 `SliceHeader` 就可以了。

```go
a :="aaa"
ssh := *(*reflect.StringHeader)(unsafe.Pointer(&a))
b := *(*[]byte)(unsafe.Pointer(&ssh))  
```

#### 3.翻转含有中文、数字、英文字母的字符串

因为中文、英文、数字所占用的字节数是不相同的，所以我们不可以使用转换为byte数组来进行反转在转换，我们这个情况需要将字符串转换为`[]rune`，因为其表示的范围更大，`rune==int32`而`byte==uint8`。

```go
func ReverseComplexString()  {
	reverse := func (s []rune) []rune {
		for i, j := 0, len(s)-1; i < j; i, j = i+1, j-1 {
		s[i], s[j] = s[j], s[i]
	}
		return s
	}
	src := "你好abc啊哈哈"
	dst := reverse([]rune(src))
	fmt.Printf("%v\n", string(dst))
}
```

#### 4. 拷贝大切片一定比小切片的代价大吗？

并不是，所有切片的大小相同；**三个字段**（一个 uintptr，两个int）。切片中的第一个字是指向切片底层数组的指针，这是切片的存储空间，第二个字段是切片的长度，第三个字段是容量。将一个 slice 变量分配给另一个变量只会复制三个机器字。所以 **拷贝大切片跟小切片的代价应该是一样的**。

*拷贝就相当于是变换指针的指向，而不是将内存数据从一个地址拷贝到另一处地址，所以这个只更换指针的指向也造成了更改其中一个切片的数据，另一个切片显示的数据也会改变即为**浅拷贝***

#### 5. map不初始化使用会怎么样，slice呢？

map不初始化为nil，向里面添加值会直接报错。`panic: assignment to entry in nil map`。但是可以进行取值，不过返回的是对应类型的零值。并且初始化和不初始化的map。长度均为0。但是`slice`是可以声明之后就可以使用（**不是真正意义上的使用**）的，可以使用`append`向里面添加元素(这种方式其实是返回了一个新的切片)，但是不能使用直接用索引赋值的方法添加元素，不过`slice`声明不初始化的话其指向的底层数组地址为`0`，第一次添加元素之后会给出一个内存地址。

#### 6. map承载多大，大了之后怎么扩容？

```go
// Maximum number of key/elem pairs a bucket can hold.
bucketCntBits = 3
bucketCnt     = 1 << bucketCntBits
// 每个桶大小为 8
// Maximum average load of a bucket that triggers growth is 6.5.
// Represent as loadFactorNum/loadFactorDen, to allow integer math.
loadFactorNum = 13
loadFactorDen = 2
// 溢出因子大小为 6.5
```

哈希表 `runtime.hmap` 的桶是 `runtime.bmap`。每一个 `runtime.bmap` 都能存储**8**个键值对，当哈希表中存储的数据过多，单个桶已经装满时就会使用 `extra.nextOverflow` 中桶存储溢出的数据。 而发生扩容的条件是：

1. 触发 `load factor` 的最大值，负载因子已达到当前界限。负载因子越大，证明空间效率越高，同时发生冲突的概率也越大。
2. 溢出桶 `overflow buckets` 过多。即溢出桶和全部正常桶数量的比值。比值过大就证明溢出桶过多。

而map的扩容也是分为两种情况进行扩容的，如果是负载因子达到最大值，则是直接动态扩容当前大小两倍作为新容量的大小。而如果是溢出桶过多，则是不改变大小的扩容。而扩容并不是一步到位，而是先申请扩容空间，但是不会进行初始化，而是等到有新的访问落到某个桶中，才会对这个桶进行扩容，也就是将`oldbucket`迁移到`bucket`。

#### 7. map的iterator是否安全？能不能一边delete一边遍历？

map的iterator是不安全的，我们需要手动对其进行并发约束来使其到达在并发中的数据安全。一般是使用`sync.RWMutex`或者是使用`channel chan`。

```go
var myMap map[int]string

func myGoRoutine(id int, numKeys int, wg *sync.WaitGroup) {
	defer wg.Done()

	for key, _ := range myMap {
		myMap[key] = strconv.Itoa(id)
	}

	for key, value := range myMap {
		fmt.Printf("Goroutine #%d -> Key: %d, Value: %s\n", id, key, value)
	}

}
func UnSafeMap() {
	// Initially set some values
	myMap = make(map[int]string)
	myMap[0] = "test"
	myMap[2] = "sample"
	myMap[1] = "GoLang is Fun!"

	// Get the number of keys
	numKeys := len(myMap)

	var wg sync.WaitGroup

	for i := 0; i < 3; i++ {
		wg.Add(1)
		go myGoRoutine(i, numKeys, &wg)
	}

	// Blocking wait
	wg.Wait()

	// Iterate over all keys
	for key, value := range myMap {
		fmt.Printf("Key: %d, Value: %s\n", key, value)
	}
}
/*
结果
Goroutine #2 -> Key: 0, Value: 2
Goroutine #2 -> Key: 2, Value: 2
Goroutine #2 -> Key: 1, Value: 2
Goroutine #1 -> Key: 0, Value: 1
Goroutine #1 -> Key: 2, Value: 1
Goroutine #1 -> Key: 1, Value: 1
Goroutine #0 -> Key: 0, Value: 0
Goroutine #0 -> Key: 2, Value: 1
Goroutine #0 -> Key: 1, Value: 1
Key: 0, Value: 1
Key: 2, Value: 1
Key: 1, Value: 1
*/
```

同时go中的map是可以一边进行操作然后一边进行遍历的。但是虽然不会出错，但是当前遍历不会受到影响。不像java他们的迭代器具有Fail-Fast性质。

#### 8. 怎么判断一个数组是否有序

第一种方法是实现`sort.Interface`的接口类型，然后直接调用`sort.IsSorted()`方法即可。或者直接使用`sort.SliceIsSorted`函数。

```go
func jungeSorted() {
	arr := []int{5,4,3,2,1}
	fmt.Println(sort.SliceIsSorted(arr, func(i, j int) bool {
		return arr[i] > arr[j]
	}))
}
```

#### 9. array和slice的区别

数组`array`是值类型的，其作为参数传递给函数，就是将数组拷贝一份，而切片`slice`是一个引用类型，是一个动态指向数组切片的指针，不定长。声明数组的时候，方括号内写明了数组长度或者使用`...`进行代替，而声名切片的时候方括号内部为空。作为函数参数时候，切片传递的是指针，所以在函数内部改动，外部切片也会发生相应的变化。

#### 10. json包变量不加tag会怎么样？

- 如果变量`首字母小写`，则为`private`。无论如何`不能转`，因为取不到`反射信息`。

- 如果变量`首字母大写`，则为`public`。

- - `不加tag`，可以正常转为`json`里的字段，`json`内字段名跟结构体内字段`原名一致`。
  - `加了tag`，从`struct`转`json`的时候，`json`的字段名就是`tag`里的字段名，原字段名已经没用。

`tag`信息是可以通过`reflect`获取的。即：

```go
type J struct {    a string //小写无tag    b string `json:"B"` //小写+tag    C string //大写无tag    D string `json:"DD" otherTag:"good"` //大写+tag}func printTag(stru interface{}) {    t := reflect.TypeOf(stru).Elem()    for i := 0; i < t.NumField(); i++ {        fmt.Printf("结构体内第%v个字段 %v 对应的json tag是 %v , 还有otherTag？ = %v \n", i+1, t.Field(i).Name, t.Field(i).Tag.Get("json"), t.Field(i).Tag.Get("otherTag")) }}func main() {    j := J{      a: "1",      b: "2",      C: "3",      D: "4",    }    printTag(&j)}
```



- `printTag`方法传入的是`j`的指针。
- `reflect.TypeOf(stru).Elem()`获取指针指向的值对应的结构体内容。
- `NumField()`可以获得该结构体的含有几个字段。
- 遍历结构体内的字段，通过`t.Field(i).Tag.Get("json")`可以获取到`tag`为`json`的字段。
- 如果结构体的字段有`多个tag`，比如叫`otherTag`,同样可以通过`t.Field(i).Tag.Get("otherTag")`获得

#### 11. 深拷贝和浅拷贝

​	一般简单的拷贝，即指针指向转换，即虽然是两个切片，但是他们指向同一个底层数组，此为浅拷贝，即通过其中一个切片修改数据，另一个切片的内容也会发生变化。而深拷贝，则是进行了内存拷贝，底层数组的拷贝，两个切片指向的地址是不相同的。同时需要注意引用切片，其中的引用如果不进行递归深拷贝，则还是会出现问题。

#### 12. make和new的区别

​	new可以用于任何类型的分配空间，指定内存，并返回该类型的指针。同时 new 函数会把分配的内存置为零，也就是类型的零值。

```go
// The new built-in function allocates memory. The first argument is a type,// not a value, and the value returned is a pointer to a newly// allocated zero value of that type.func new(Type) *Type
```

`make`只能用于`slice, map, channel`的初始化。这三个刚好为引用类型，并且**Unlike new, make's return type is the same as the type of its
argument, not a pointer to it.**

```go
// The make built-in function allocates and initializes an object of type// slice, map, or chan (only). Like new, the first argument is a type, not a// value. Unlike new, make's return type is the same as the type of its// argument, not a pointer to it. The specification of the result depends on// the type://	Slice: The size specifies the length. The capacity of the slice is//	equal to its length. A second integer argument may be provided to//	specify a different capacity; it must be no smaller than the//	length. For example, make([]int, 0, 10) allocates an underlying array//	of size 10 and returns a slice of length 0 and capacity 10 that is//	backed by this underlying array.//	Map: An empty map is allocated with enough space to hold the//	specified number of elements. The size may be omitted, in which case//	a small starting size is allocated.//	Channel: The channel's buffer is initialized with the specified//	buffer capacity. If zero, or the size is omitted, the channel is//	unbuffered.func make(t Type, size ...IntegerType) Type
```

#### 13.  slice,map,channel创建的时候的几个参数什么含义？

​	由于切片的底层即`SliceHeader`的结构如下所示。而创建切片的使用`make(type, len, cap)`，`cap`通常可以省略，省略情况下`cap=len`。因为两者之间的关系为`cap>=len>=0`。其中`cap`代表容量，`len`代表当前切片的长度。

```go
type SliceHeader struct {	Data uintptr	Len  int	Cap  int}
```

而我们创建`map`通常使用`make(map[Type]Type,size)`，`size`表示map的存储能力，可以省略。使用`make(chan Type, size)`创建channel而`size`表示的具有通道的缓冲区大小，如果不设置，则表示该通道不具有缓冲区，默认`size=0`。

#### 14 slice扩容

源代码如下：可以看出当当前的容量小于扩容之后的容量的长度的时候，并且当前的长度小于1024，则扩容为当前的两倍，否则扩容四分之一大小。

```go
func grow(s Value, extra int) (Value, int, int) {	i0 := s.Len()	i1 := i0 + extra	if i1 < i0 {		panic("reflect.Append: slice overflow")	}	m := s.Cap()	if i1 <= m {		return s.Slice(0, i1), i0, i1	}	if m == 0 {		m = extra	} else {		for m < i1 {			if i0 < 1024 {				m += m			} else {				m += m / 4			}		}	}	t := MakeSlice(s.Type(), i1, m)	Copy(t, s)	return t, i0, i1}
```

#### 15. 线程安全的map怎么实现

go里面的map并不是并发安全的，实现其安全主要有三种方法；

- 采用`sync.RWMutex`或者是在协程环境下使用`chan`

  ```go
  type RWMap struct { // 一个读写锁保护的线程安全的map    sync.RWMutex 	// 读写锁保护下面的map字段    m map[int]int}
  ```

- 简单的采用`sync.RWMutex`，虽然功能上能够满足，但是在性能上，由于是对整个哈希表进行加锁，所以会导致性能下降。我们可以学习`java`对哈希表加锁的处理方式，使用多段锁，降低锁的粒度，go中比较知名的分片map实现是`orcaman/concurrent-map`,其对将整个map分为n快，每个块读写操作互相不干扰。实现原理也类似，就是在一个切片中存储带有读写锁的map，然后通过计算key在哪一个分片上来进行哈希表的读写。

  ```go
  var SHARD_COUNT = 32// 分成SHARD_COUNT个分片的maptype ConcurrentMap []*ConcurrentMapShared// 通过RWMutex保护的线程安全的分片，包含一个maptype ConcurrentMapShared struct {    items        map[string]interface{}    sync.RWMutex // Read Write mutex, guards access to internal map.}// 创建并发mapfunc New() ConcurrentMap {    m := make(ConcurrentMap, SHARD_COUNT)    for i := 0; i < SHARD_COUNT; i++ {        m[i] = &ConcurrentMapShared{items: make(map[string]interface{})}    }    return m}// 根据key计算分片索引func (m ConcurrentMap) GetShard(key string) *ConcurrentMapShared {    return m[uint(fnv32(key))%uint(SHARD_COUNT)]}
  ```

- 内置的`sync,map`是一个并发安全的map，但是用的比较少，主要是用于一写多读或者是各个协程操作的key集合没有交集或者是交集很少，能够显著提升性能。

#### 16. struct是否可以进行比较?

在go中数据类型可以比较与不可以比较的数据类型如下所示：

- 可比较：*Integer*，*Floating-point*，*String*，*Boolean*，*Complex(复数型)*，*Pointer*，*Channel*，*Interface*，*Array*
- 不可比较：*Slice*，*Map*，*Function*

`struct`是否可以比较以及比较之后的结果，主要是看其内部字段的类型，如果其内部字段类型均为可以比较的数据类型，则该`struct`是可以比较的，如果含有不可比较的数据类型，则`struct`也不可比较。并且结构体是否相等，也要看其内部的数据值。当然我们可以通过`reflect.DeepEqual`来实现包含不可直接比较的数据类型的结构体实例的比较。`reflect.DeepEqual`就是比较所有的值，即两个值深度一致。

#### 17. map如何实现顺序读取

​	`map`一般的读写的顺序是不固定，想要实现顺序读写，需要先将`key`取出，然后再通过`key`取出`value`。而一般对`map`进行排序输出，也是通过这种方式，不过是需要对`key`进行排序。

#### 18.  go中实现set

​	效仿`java`中`hashset`的实现，由于`map`中不会存在相同的`key`值，所以我们可以通过`map`实现。当然由于`map`中的`key`必须是要由可以比较的数据类型构成，所以例如切片、哈希表、函数是不可以的。而结构体内部也必须不含有不可比较类型。数据类型如下，使用`struct{}`作为`value`的原因是因为其占用的内存大小为0。

```go
type Set map[interface{}]struct{}
```

#### 19.  golang中是否可以进行指针运算？

​	golang中有普通指针，`unsafe.Poniter`，以及`uintptr`。其中只有`uintptr`可以进行指针运算，但是go的垃圾回收机制不会将`uintptr`看作指针，`uintptr`无法持有对象，并且会被回收。但是我们可以将普通指针通过`unsafe.Poniter`转换为`uintptr`，进行运算操作之后，在转换为普通指针。所以golang严格意义上不能进行指针运算，但是可以通过转换间接完成指针运算。

#### 20.  for select时候，如果通道已经关闭会发生什么情况，如果select中只有一个case呢？

我们将其分为几种情况进行讨论：

1. 第一种情况就是for循环里面被关闭的通道。从结果上可以看出，通道关闭之后，还是会进入读取通道信息的case。这是因为通道关闭之后，只是不能再向里面写入数据，但是可以从通道中读取数据。我们可以通过在确认通道已经关闭，并且已经没有数据读出的时候，将通道置为nil，就不会再读取已经关闭的通道了。

   ```go
   const fmat = "2006-01-02 15:04:05"func closeChannelInFor() {	c := make(chan int)	go func() {		time.Sleep(1*time.Second)		c <- 10		close(c)	}()	for {		select {		case x, ok := <- c:			fmt.Printf("%v, 通道读取到：x=%v, ok=%v\n", time.Now().Format(fmat), x, ok)            //if !ok {            //    c = nil            //}			time.Sleep(500*time.Millisecond)		default:			fmt.Printf("%v, 通道么有读取到数据进入defult \n", time.Now().Format(fmat))			time.Sleep(500*time.Millisecond)		}	}}/*结果2022-02-22 13:02:51, 通道么有读取到数据进入defult 2022-02-22 13:02:52, 通道么有读取到数据进入defult 2022-02-22 13:02:52, 通道读取到：x=10, ok=true2022-02-22 13:02:53, 通道读取到：x=0, ok=false2022-02-22 13:02:54, 通道读取到：x=0, ok=false2022-02-22 13:02:54, 通道读取到：x=0, ok=false2022-02-22 13:02:55, 通道读取到：x=0, ok=false2022-02-22 13:02:55, 通道读取到：x=0, ok=false2022-02-22 13:02:56, 通道读取到：x=0, ok=false*/
   ```

2. 而如果只有一个case，则还是会进入该case，但是如果将其置为nil，则会造成协程死锁。

   ```go
   func closeChannelInForOneCase() {	c := make(chan int)	go func() {		time.Sleep(1*time.Second)		c <- 10		close(c)	}()	for {		select {		case x, ok := <- c:			fmt.Printf("%v, 通道读取到：x=%v, ok=%v\n", time.Now().Format(fmat), x, ok)			if !ok {				c = nil			}			time.Sleep(500*time.Millisecond)		}	}}/*结果:2022-02-22 13:14:49, 通道读取到：x=10, ok=true2022-02-22 13:14:50, 通道读取到：x=0, ok=falsefatal error: all goroutines are asleep - deadlock!*/
   ```

所以说`select`中如果有某个通道有值可以读的时候，就会执行该`case`，但是如果没有`default`，则有可能造成阻塞，直到有通道可以运行。

#### 21. defer 的使用

`defer`关键字就是实现在作用域结束之后执行函数的关键字，主要作用就是在当前函数或者是方法返回之前调用一些用于收尾的函数，例如关闭文件、关闭数据库连接以及解锁资源。

##### 多个defer语句的执行顺序

`defer`语句的执行顺序和在代码中的位置相关，在函数执行语句返回之前，按照先进后出的方式执行所有的`defer`语句。但是如果存在`panic`语句是例外。`panic`会导致程序崩溃，但是不会影响`defer`语句的运行。

##### defer的值

`defer`修饰的语句中的值，在该语句出现的时候确定，后续的执行并不影响结束的时候语句中的值（非引用类型或者是指针）。当 `defer` 调用时其实会对函数中引用的外部参数进行拷贝。但是如果拷贝的指针类型，则还是会出现变化。

#### 22. select的使用

`select`能够让协程同时等待多个通道可读或者可写，在多个文件或者是通道状态改变之前，`select`会一致阻塞当前的协程。`select`可以在通道上进行非阻塞的手法操作，并且当多个通道都可以进行操作的时候，将会进行随机选择一个`case`进行执行。

##### 典型应用

1. 超时判断，即一个`case`作为接收消息，另一个`case`为一个`time.After(...)`。则当第一个`case`在一定时间内阻塞，则将会执行另外一个`case`，判断超时，做出相应的处理。
2. 判断通道是否阻塞
3. 用于多个协程在某个协程达到退出条件的时候，退出其他所有的协程。

#### 23. 如何从panic中恢复？

​	在了解如何从`panic`中恢复之前，我们先了解`panic`的机制。`panic`会改变程序的控制流，调用`panic`之后会立刻停止执行当前函数的剩余代码，并在当前协程中递归执行调用方的`defer`。而`recover`可以中值`panic`造成的程序崩溃，她是一个只能在`defer`中发挥作用的函数，在其他作用域是不会发挥作用的。

- `panci`只会触发当前协程的`defer`。
- `recover`只有在`defer`中调用才会有效。
- `panic`允许在`defer`中嵌套多次使用。

所以说我们从`panic`中恢复的话，需要将`recover`语句放置在`defer`关键词之后。示例如下：

```go
func badCall() {    panic("bad end")}func test() {    defer func() {        if e := recover(); e != nil {            fmt.Printf("Panicing %s\r\n", e)        }    }()    badCall()    fmt.Printf("After bad call\r\n") // <-- wordt niet bereikt}func main() {    fmt.Printf("Calling test\r\n")    test()    fmt.Printf("Test completed\r\n")}/*结果Calling testPanicing bad endTest completed*/
```

#### 24. 如何避免内存逃逸

​	内存逃逸就是值得局部变量（存在栈上）没有在栈上进行回归，而是进入到堆中，在堆中被回收，就叫做内存逃逸。所以避免内存逃逸就是避免局部变量进入到堆中。可以通过命令行`go build -gcflags=-m`查看内存逃逸，内存逃逸发生的原因有以下几种：

1. 向`chan`发送指针数据。由于在编译的时候，不知道该数据会被哪个goroutine接收，所以不知道这个局部变量什么时候才能释放，所以只能放到堆中，在堆中等待被回收。
2. 局部变量在函数调用结束后还被其他地方使用，比如函数返回局部变量指针或闭包中引用包外的值。因为变量的生命周期可能会超过函数周期，因此只能放入堆中。
3. 在 slice 或 map 中存储指针。比如 []*string，其后面的数组可能是在栈上分配的，但其引用的值还是在堆上。
4. 切片扩容后长度太大，导致栈空间不足，逃逸到堆上。初始化的时候是在栈上进行分配，运行时数据扩充则要在对上进行分配，但是初始化的时候不知道容量大小，则会直接在堆上进行分配。
5. 在 interface 类型上调用方法。 在 interface 类型上调用方法时会把interface变量使用堆分配， 因为方法的真正实现只能在运行时知道。

针对发生内存逃逸的原因我们可以通过以下方式来避免内存逃逸：

1. 对于小型的数据，使用传值而不是传指针，避免内存逃逸。
2. 避免使用长度不固定的slice切片，在编译期无法确定切片长度，只能将切片使用堆分配。
3. interface调用方法会发生内存逃逸，在热点代码片段，谨慎使用。

#### 25. Goroutine 泄露

goroutine泄露的原因主要集中在以下几个方面：总结来看，只要发生了阻塞，就会产生Gououtine泄露。

1. goroutine中正在进行channel的读写操作，但是因为代码逻辑问题，导致一直被阻塞。
2. goroutine内的业务逻辑进入死循环，资源一直无法被释放。
3. goroutine内的业务逻辑进入长时间的等待，并且有不断新增的goroutine进入等待。

造成阻塞的原因大多可以分为：

1. 向通道中发送数据，但是没有从通道中取出数据。或者是想从通道中取出数据，但是没有向通道中发送数据。
2. 通道没有进行初始化，使用了nil通道
3. 没有阻塞，但是一个操作等待时间比较长（例如在获取网页内容的时候，由于网速问题，并且没有设置超时，无法进行复用），就会导致`goroutine`的数量越来越多。
4. 锁使用不当，加锁忘记释放锁，或者是同步锁使用

##### 排查方法

1. 可以使用`runtime.NumGoroutine`来获取Goroutine的运行数量，然后前后进行比较，就可以知道是否泄露了。

2. 在业务运行场景中，一般可以直接使用PProf。

   ```go
   import (    "net/http"     _ "net/http/pprof")http.ListenAndServe("localhost:6060", nil))
   ```

#### 26. 内存泄露问题

​	首先要说明的内存泄漏和内存逃逸是不同的概念，内存泄露是一部分内存无法得到回收。而内存逃逸只是局部变量从栈跑到堆中，但是还是会被回收，如果在这个阶段不被回收才是内存泄露。造成内存泄露的原因如下：

1. 获取长字符串的一段导致长字符串未释放。
2. 获取长切片的一段导致长切片没有释放。
3. 在长切片中新建切片导致泄露。
4. goroutine泄露，这个一般是由于goroutine阻塞引起的。
5. `time.Ticker`没有关闭导致泄露。
6. `Finalizer`导致泄露。
7. `Deferring Function Call`导致泄露。

#### 27. sync.Pool的适用场景

​	`sync.Pool`是一个单独保存和检索的临时对象。其目的是为了缓存已经分配到那时没有使用的元素以便于后续重用。其一大特点就是可以减轻垃圾收集器的压力，而且他是并发安全的。所以其可以很容易的构成高效、并发安全的空闲列表。一个很好的例子就是在`fmt.Printf`中，管理了一个动态大小用于存储输出的缓冲区。下面是官方源码注释：

```go
A Pool is a set of temporary objects that may be individually saved andretrieved.Any item stored in the Pool may be removed automatically at any time withoutnotification. If the Pool holds the only reference when this happens, theitem might be deallocated.A Pool is safe for use by multiple goroutines simultaneously.Pool's purpose is to cache allocated but unused items for later reuse,relieving pressure on the garbage collector. That is, it makes it easy tobuild efficient, thread-safe free lists. However, it is not suitable for allfree lists.An appropriate use of a Pool is to manage a group of temporary itemssilently shared among and potentially reused by concurrent independentclients of a package. Pool provides a way to amortize allocation overheadacross many clients.An example of good use of a Pool is in the fmt package, which maintains adynamically-sized store of temporary output buffers. The store scales underload (when many goroutines are actively printing) and shrinks whenquiescent.On the other hand, a free list maintained as part of a short-lived object isnot a suitable use for a Pool, since the overhead does not amortize well inthat scenario. It is more efficient to have such objects implement their ownfree list.A Pool must not be copied after first use.
```

#### 28. 对已经关闭的`chan`进行读写，会发生什么？如果是未初始化的`chan`呢？

对于已经关闭的`chan`需要分情况，情况如下：

1. 读已经关闭的通道会一直读出信息，但是信息是什么会根据通道中是否还有数据来决定的。
   - 第一种情况如果是关闭的通道还有数据，则会将数据读出，返回值第一个为取出的数据，第二个为标志是否读取成功的标志为`true`。
   - 第二种情况关闭的通道中已经没有数据了，此时读出的数据为通道中存储数据类型的零值，第二个标志位为`false`。
2. 向已经关闭的通道写数据会导致`panic`。

如果是未初始化的`chan`，从`chan`中读取数据会导致一直阻塞，同时向`chan`中写入数据也会导致阻塞。

#### 29. `sync.map`的优缺点和使用场景

由于go中的map不是并发安全的，go提供了`sync.map`用于并发使用。 从官方源码注释上我们了解到，`sync.map`主要更适合以下两个使用场景（对其专门做了优化），在这两种情况下使用`sync.map`比使用普通`map`加上`MUtex、RWMutex`性能要好很多。其工作原理就是在写的时候直接邪写入到`dirty map`，读取的时候先读`read map`如果`read map`中没有再去读`dirty map`。就是读写分离，空间换取时间。

1. 对于map中元素多读少写的情况。
2. 多个goroutine读取和写入是`key`没有相关的元素的情况。即插入元素或者读取元素分散性强。

所以其优缺点也很明显。

优点：通过读写分离，降低锁时间来提高性能

缺点：不适用于大量写的场景(大量写的场景可以使用普通map+锁的方式)，这样会导致read **map**读不到数据而进一步加锁读取，同时dirty **map**也会一直晋升为read **map**，整体性能较差。

#### 30.  如何让主协程等待子协程完成之后再继续执行?

这一点涉及到了并发同步问题，一般简单有两种方式：

1. 使用通道传递信号量，其实就是变相模仿`sync.WaitGroup`。即每个goroutine完成之后向通道中写入消息，而主goroutine则会通过`for range`来判断子goroutine是否完成。
2. 使用`sync.WaitGourp`，即开始时间使用`Add()`方法确认有多少个子协程，在子goroutine中完成业务代码后调用`Done()`方法，主goroutine中调用`Wait()`方法等待子goroutine的完成。

#### 31. channel有无缓存有何区别？

​	带缓存的channel和不带缓存的channel最大的区别就在于无缓存的的channel如果发送方或者接收方没有准备好就会被阻塞，所以无缓存的channel一般用于需要同步的场景中。而有缓存的channel只有在缓冲区被写入满了并且没有读取才会阻塞写入。

#### 32. goroutine的并发控制

goroutine只能由自己本身控制在何种情况下退出，外界一般无法强制结束（程序崩溃或者是main函数结束除外）。而针对goroutine的并发控制类型只分为以下三种：

1. 全局共享变量

   全局共享变量是一种最简单的控制并发的方式。一般的实现方式如下：

   - 声明一个全局变量
   - 所有子goroutine共享这个变量，并且不断轮询这个变量检查是否更新
   - 在主goroutine中变更该全局变量
   - 子goroutine检测到变量更新然后执行相应的逻辑。

   其优点就是实现简单，但是缺点就是只能多读一写，如果想要多写，就需要解决全局共享变量的同步问题，例如给他加上锁，但是这样会降低性能，增加实现复杂度，并且不适合在子goroutine间进行通信。而且由于是单向通信，所以只能由主goroutine向子goroutine进行通信，所以主goroutine无法精确等待子goroutine完成之后再退出。

2. channel通信

   channel通信控制基于CSP模型，避免了大量加锁解锁的性能消耗，而且比Actor模型更加灵活。而使用channel进行通信一般用的最多的还有`select、for range、sync.WaitGroup`等等。

3. Context包

   context通常叫做上下文，我们通常可以将一些数据封装在context变量中。最常见的就是再网络编程下，获取到一个请求之后，可能会对这个请求开启新的子goroutine进行后续处理。所以可以将信息封装再context中用于通信和控制。

#### 33.  channel的底层实现

​	首先说明一点，channel本身就是一个指针，指向的是堆中分配的一个`hchan`的结构体。在一般情况下即没有阻塞发生的情况，`sendx`表示在幻想链表中`chan`接收的元素将会存放的索引，而`recvx`表示`chan`将会发送的数据在环形链表中所在的索引。而如果缓存满了，所以这个时候会阻塞当前的goroutine。并将含有当前goroutine的指针和要send的元素放入到`sendq`队列中等待被唤醒，如果是要`recv`被阻塞则相应的放入到`sendq`中。

```go
type hchan struct {	qcount   uint           // total data in the queue	dataqsiz uint           // size of the circular queue	buf      unsafe.Pointer // points to an array of dataqsiz elements	elemsize uint16	closed   uint32	elemtype *_type // element type	sendx    uint   // send index	recvx    uint   // receive index	recvq    waitq  // list of recv waiters	sendq    waitq  // list of send waiters	// lock protects all fields in hchan, as well as several	// fields in sudogs blocked on this channel.	//	// Do not change another G's status while holding this lock	// (in particular, do not ready a G), as this can deadlock	// with stack shrinking.	lock mutex}
```

具体详解查看引用7。值得注意的一个点是因为从`chan`中取数据被阻塞和因为将数据放入到`chan`中被阻塞，两种情况唤醒时处理方式不一样。`chan`中取数据被阻塞，唤醒的时候会直接从发送数据的那个goroutine中将数据复制到当前被唤醒的goroutine中。不会再经过`chan`。减少了内存复制的开销。

#### 34. 读写锁底层实现

读写锁主要是读与读之间不互斥，读写与写写之间是互斥的。要了解底层实现，首先我们要了解读写锁底层结构体以及相关加锁释放锁实现：

```go
type RWMutex struct {	w           Mutex  // held if there are pending writers	writerSem   uint32 // semaphore for writers to wait for completing readers	readerSem   uint32 // semaphore for readers to wait for completing writers	readerCount int32  // number of pending readers	readerWait  int32  // number of departing readers}
```

整个读写锁并发控制过程如下所示：

1. 如果没有写操作进入，则每个读操作都会使得readerCount加1，完成后readerCount减1.整个过程是不会阻塞的，因为读与读之间不互斥。
2. 当由写操作进入的时候，首先会进行互斥锁阻塞其他写操作，并将readerCount修改为很小的值，从而阻塞新来的读操作。
3. 如果写操作进入的额时候还有没有完成的读操作，则会记录这些写操作的数量，等待他们全部完成的时候，再将写操作唤醒。注意这个时候已经不会由读操作在进入。
4. 写操作完成之后需要将readerCount置为原来的值，保证新的读操作不会被阻塞，然后唤醒之前等待的读操作，再将互斥锁释放。使得后续写操作不会被阻塞。

#### 35. golang中的CSP思想

传统的CSP模型是用于描述两个独立的并发实体通过共享的管道进行通信的并发模型，不关注发送消息的实体而关注与发送消息时使用的channel。golang借用了process和channel的概念，但是并没有完全实现CSP模型的所有理论。process再go语言上表现就是goroutine 是实际并发执行的实体，每个实体之间是通过channel通讯来实现数据共享。

#### 36. uintptr和unsafe.Pointer的区别？

- unsafe.Pointer只是单纯的通用指针类型，用于转换不同类型指针，它不可以参与指针运算；
- 而uintptr是用于指针运算的，GC 不把 uintptr 当指针，也就是说 uintptr 无法持有对象， uintptr 类型的目标会被回收；
- unsafe.Pointer 可以和 普通指针 进行相互转换；
- unsafe.Pointer 可以和 uintptr 进行相互转换。                 

#### 37. golang垃圾回收

​	垃圾回收的概念主要是为了回收堆上的内存空间，对于那些没有任何变量引用的对象进行回收，垃圾回收机制做的最好的还是要看`java`的垃圾回收机制。垃圾回收总体上分为两个步骤，第一个步骤是判断哪些内存空间是需要被回收的，第二步是选择合适的回收算法来进行回收。`c,c++,Rust`等都是再栈上创建的变量作用域结束后自动回收，但是通过`malloc`在堆上申请的需要使用`free`手动释放内存。而`python,java,go`则是自动进行垃圾回收，垃圾回收器会周期性释放已经没有引用的对象所占用的内存空间。

​	垃圾回收器的目标：

1. 防止内存泄露，最基本的目标就是防止未及时收集而造成内存泄露。
2. 自动回收没有用的内存。
3. 减少内存碎片的产生，重整内存空间，提高内存利用率。

第一步判断哪些对象可以回收的主要有两种方式：

1. 引用计数法，引用计数实现简单，并且回收快速，不需要暂停。但是有一个缺点就是不能解决循环引用的问题。
2. 可达性分析算法。这也是大多数回收算法所用的判断是否可以回收的算法。可达性分析算法需要一个`GC Root`,一般情况下`GC Root`选择对象是全局对象，栈上的对象（函数参数与内部变量。）。但是可达性分析算法需要暂停整个程序，即`Stop the World STW`。因为如果不暂停程序，可能会造成标记的过程中会出现错误，有可能有的对象重新被使用，但是却在之前被标记为回收。

第二步回收算法：

1. 标记清除 造成内存碎片较多
2. 标记整理 整理内存碎片时间较长
3. 标记复制 内存只能用一半

golang的垃圾回收算法则是三色标记法，在不暂停程序的情况下，完成对象的可达性分析。其会将全部对象分为三类:

- 白色：未搜索的对象，在回收周期开始时所有对象都是白色，在回收周期结束时所有的白色都是垃圾对象
- 灰色：正在搜索的对象，但是对象身上还有一个或多个引用没有扫描
- 黑色：已搜索完的对象，所有的引用已经被扫描完

具体搜索过程如下：

- 初始时所有对象都是白色对象
- 从`GC Root`对象出发，扫描所有可达对象并标记为灰色，放入待处理队列
- 从队列取出一个灰色对象并标记为黑色，将其引用对象标记为灰色放入队列
- 重复上一步骤，直到灰色对象队列为空
- 此时所有剩下的白色对象就是垃圾对象

其优点就是不用暂停程序就可以进行回收。但是在程序垃圾对象的产生速度大于垃圾对象的回收速度时，可能导致程序中的垃圾对象越来越多而无法及时收集。

#### 38. 写屏障，混合写屏障

​	这两个主要是为了三色标记法和用户程序并发过程出现的问题而出现的。当三色标记收集过程中满足下面两个条件就可能出现错误回收非垃圾对象的问题。

- 条件1：某一黑色对象引用白色对象
- 条件2：对于某个白色对象，所有和它存在可达关系的灰色对象丢失了访问它的可达路径

常见解决方法就是使用`STW`，但是这个违背了三色标记设计的目的，而另外一种就是读写屏障技术。

使用屏障技术可以使得用户程序和三色标记过程并发执行，我们只需要达成下列任意一种三色不变性：

- 强三色不变性：黑色对象永远不会指向白色对象
- 弱三色不变性：黑色对象指向的白色对象至少包含一条由灰色对象经过白色对象的可达路径

`GC`中使用的内存读写屏障技术指的是编译器会在编译期间生成一段代码，该代码在运行期间用户读取、创建或更新对象指针时会拦截内存读写操作，相当于一个`hook`调用，根据`hook`时机不同可分为不同的屏障技术。由于读屏障`Read barrier`技术需要在读操作中插入代码片段从而影响用户程序性能，所以一般使用写屏障技术来保证三色标记的稳健性。

#### 39. `var _ io.Writer = (*myWriter)(nil)`这样写的目的是为了什么？

主要是为检查是否实现了某个接口。例如题目中的意思就是为了看`myWriter`是否实现了`io.Writer`接口。

#### 40.  GMP模型

​	GMP模型golang的并发调度模型。其中G表示Goroutine，M表示内核线程，P表示调度器。GMP模型的组成就是由全局协程队列，每个P所具有的本地协程队列。当前P本地协程队列中没有G，则会取全局协程队列中取。而如果全局队列中也没有，就会从其他的P的本地协程队列中偷取。

#### 41. 必须要手动内存对齐的情况

手动内存对齐主要是为了平台的移植。例如`struct`中字段顺序不同，内存占用也会不同。主要是因为在编译过程中会使用内存对齐，所以在内存中分布会有高位地位的区别。也就是不同的顺序会造成内存部分内存无法使用。而且加入程序运行在不同对齐方式的平台，那么可能会导致`panic`。

#### 42. go 栈扩容和栈缩容，以及连续栈的缺点

go的栈更新过后，从分段栈转换为连续栈。连续栈的实现方式：当检测到需要耕读哦的栈的时候，分配比原来大一倍的栈，把旧数据拷贝到新栈，释放旧栈。

- 栈扩容会将栈扩充到比前面两倍。
- 栈缩容发生在GC期间，缩容就是用分配一块新的内存来替换原来的，大小也是缩小一倍。

连续栈虽然解决了分段栈的2个问题，但这种实现方式也会带来其他问题：

- 更多的虚拟内存碎片。尤其是你需要更大的栈时，分配一块连续的内存空间会变得更困难
- 指针会被限制放入栈。在go里面不允许二个协程的指针相互指向。这会增加实现的复杂性。

#### 43. golang 闭包

**闭包** 是由函数及其相关引用环境组合而成的实体(即：闭包=函数+引用环境)。从下面的例子，可以看出，`a,b`的值会根据调用闭包函数的次数逐渐更新。

```go
func fib() func() int {	a, b := 0, 1	return func() int {		a, b = b, a+b		return a	}}// 调用如下f00 := fib()fmt.Println(f00(), f00(), f00(), f00(), f00())// 输出结果是：1 1 2 3 5
```

闭包函数主要有两种场景。

1. 闭包里没有引用环境&获取引用全局变量。这种场景下，其实现就是普通的函数，按照普通的函数调用方式执行闭包调用。
2. 闭包里引用局部变量。这种场景下，才是真正的闭包（函数+引用环境），并且以一个struct{FuncAddr, LocalAddr3, LocalAddr2, LocalAddr1}结构存储该闭包，等到调用闭包时，会把该结构地址提前放置一个寄存器，闭包内部通过该寄存器访问引用环境的变量。也就是上述例子的情况。`a,b`被存储。

#### 44.  Goroutine什么时候会被挂起？

goroutine挂起的原因有很多，这个在go源码中由详细的叙述，并且将所有的原因列了出来。如下：

```go
// package runtime\runtime2var waitReasonStrings = [...]string{	waitReasonZero:                  "",	waitReasonGCAssistMarking:       "GC assist marking", // GC辅助标记阶段	waitReasonIOWait:                "IO wait", // IO阻塞等待	waitReasonChanReceiveNilChan:    "chan receive (nil chan)", // 对未初始化的chan进行读操作	waitReasonChanSendNilChan:       "chan send (nil chan)", // 对未初始化的chan进行写操作	waitReasonDumpingHeap:           "dumping heap", // 对 Go Heap 堆 dump 时个的使用场景仅在 runtime.debug 时，也就是常见的 pprof 这一类采集时阻塞。	waitReasonGarbageCollection:     "garbage collection", // 在垃圾回收时，主要场景是 GC 标记终止(GC Mark Termination)阶段时触发。	waitReasonGarbageCollectionScan: "garbage collection scan", // 在垃圾回收扫描时，主要场景是 GC 标记(GC Mark)扫描 Root 阶段时触发。	waitReasonPanicWait:             "panicwait", // 在 main goroutine 发生 panic 时，会触发。	waitReasonSelect:                "select", // 关键字 select	waitReasonSelectNoCases:         "select (no cases)", //在调用关键字 select 时，若一个 case 都没有，会直接触发。	waitReasonGCAssistWait:          "GC assist wait", // GC 辅助标记阶段中的结束行为，会触发。	waitReasonGCSweepWait:           "GC sweep wait", // GC 清扫阶段中的结束行为，会触发。	waitReasonGCScavengeWait:        "GC scavenge wait", //GC scavenge 阶段的结束行为，会触发。GC Scavenge 主要是新空间的垃圾回收，是一种经常运行、快速的 GC，负责从新空间中清理较小的对象。	waitReasonChanReceive:           "chan receive", //在 channel 进行读操作，会触发。	waitReasonChanSend:              "chan send", // 在 channel 进行写操作，会触发。	waitReasonFinalizerWait:         "finalizer wait", //在 finalizer 结束的阶段，会触发。在 Go 程序中，可以通过调用 runtime.SetFinalizer 函数来为一个对象设置一个终结者函数。这个行为对应着结束阶段造成的回收。 	waitReasonForceGCIdle:           "force gc (idle)", // 强制 GC(空闲时间)结束时，会触发。	waitReasonSemacquire:            "semacquire", // 信号量处理结束时，会触发。	waitReasonSleep:                 "sleep", // 经典的 sleep 行为，会触发。 	waitReasonSyncCondWait:          "sync.Cond.Wait", // 结合 sync.Cond 用法能知道，是在调用 sync.Wait 方法时所触发。	waitReasonTimerGoroutineIdle:    "timer goroutine (idle)", // 与 Timer 相关，在没有定时器需要执行任务时，会触发。	waitReasonTraceReaderBlocked:    "trace reader (blocked)", // 与 Trace 相关，ReadTrace会返回二进制跟踪数据，将会阻塞直到数据可用。 	waitReasonWaitForGCCycle:        "wait for GC cycle", // 等待 GC 周期，会休眠造成阻塞。	waitReasonGCWorkerIdle:          "GC worker (idle)", // GC Worker 空闲时，会休眠造成阻塞。	waitReasonPreempted:             "preempted", // 发生循环调用抢占时，会会休眠等待调度。	waitReasonDebugCall:             "debug call", // 调用 GODEBUG 时，会触发。}
```

综上所述主要的场景是：

- 通道(Channel)。
- 垃圾回收(GC)。
- 休眠(Sleep)。
- 锁等待(Lock)。
- 抢占(Preempted)。
- IO 阻塞(IO Wait)
- 其他，例如：panic、finalizer、select 等。

#### 45. DATA Trace是什么？怎么检测以及解决？

Data Trace就是并发过程中的数据竞争问题。通常我们可以使用`-trace`添加到编译命令行来检测data trace情况。

解决这种情况就是解决并发同步问题。所以可以使用`sync.WaitGroup,无缓冲通道，mutex锁`。

### 参考

[1].[Go 并发之三种线程安全的 map - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/356739568)

[2]. [Golang 之 struct能不能比较 - 掘金 (juejin.cn)](https://juejin.cn/post/6881912621616857102)

[3].[Review 《JSON and Go》 - 大白的碎碎念 (bwangel.me)](https://www.bwangel.me/2019/05/21/review-json-and-go/)

[4].https://learnku.com/docs/the-way-to-go/

[5].[简单聊聊内存逃逸 ｜ 剑指 offer - golang - SegmentFault 思否](https://segmentfault.com/a/1190000039843497)

[6].[深入golang之---goroutine并发控制与通信 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/36907022)

[7].[图解Go的channel底层实现 - 菜刚RyuGou的博客 (i6448038.github.io)](https://i6448038.github.io/2019/04/11/go-channel/)

[8].[go 读写锁实现原理解读 - SegmentFault 思否](https://segmentfault.com/a/1190000039712353)

[9].[图示Golang垃圾回收机制 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/297177002)

[10].[Golang 是否有必要内存对齐？ - 云+社区 - 腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1489459)

[11].[【面试高频问题】线程、进程、协程 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/70256971)

