# Golang之Map


## Map

一种成对出现的键值数据结构，一个map中有多个键值对

key必须是可哈希的数据类型

map是线程不安全的

### Map实现方式

map是基于哈希表存储的

- 1.根据hash算法计算出键的哈希值
- 2.根据哈希值和哈希表的数量进行取模运算
- 3.计算出来的值就是这个key放入到哈希表的位置

哈希计算一般会存在哈希碰撞问题，不同的key能能计算出的值是一样的

如果出现了哈希碰撞，则会在这个位置以链表的形式存储落到该位置上的key

### 初始化

通过make来初始化分配内存

每个key只能出现一次，如果同一个key出现两次，后面key的val会覆盖之前key的val

```go
m = make(map[int]string, 10)
```

- 1.当make的时候，会创建一个hmap结构体对象
- 2.生成一个哈希因子`hash0`并赋值到hmap对象中，用于后续为key创建哈希值
- 3.根据指定的map长度计算出B的长度

- 4.然后根据B去创建桶（bmap）并存放在buckets数组中
  - 当B<4时，根据B创建桶的个数的规则为：2^B（标准桶）
  - 当B>=4时，根据B创建桶的个数的规则为2^B+2^B-4（标准桶+溢出桶）
- 每个bmap可以存放8个键值对，当空间不够时，则需要使用溢出桶，并将当前bmap中的overflow指向溢出桶的位置

### 写入数据

```go
m[1] = "test"
```

- 1.使用哈希因子对键进行哈希运算，得到一个结果
- 2.获取改结果的后B位，并根据后B为的值来决定将此键值对存放到那个桶中（bmap）

```
将哈希值和桶掩码进行&运算，最终得到哈希值的后B位的值。假设当B位1时，其结果为0
哈希值：0110111000111111111101110111010
桶掩码：0000000000000000000000000000001
结果	:00000000000000000000000000000000 = 0

找桶的原则实际上是根据后B位的位运算计算出索引位置，然后再根据buckets数组中根据索引找到目标桶（bmap）
```

- 也可以在声明时直接赋值，该方式不用make

```go
var m map[string]string{
    "test1":"a",
    "test2":"b",
    "test3":"c"
}
```

### 修改数据

```go
m["test1"] = 1
```

- 上面提到过，每个key只能出现一次，如果修改的key在map中已经存在，上面代码则会修改之前key的val
- 如果没有，则会新增
- 修改之前可以先查询下

### 删除数据

```go
delete(mapName,"key")
```

- key存在时则执行删除操作
- key不存在时，也不会报错

- 删除所有的key
  - 1.可以遍历删除
  - 2.可以再次对该map进行make操作，旧的key会被GC回收
  - 没有内置的清空所有key的方法

### 遍历数据

- 可以使用for range方法和for循环来遍历

```go
//for range方法示例
m := map[string]string{
    "test1":"1",
    "test2":"2",
}

for k, v := range m {
    fmt.Println(k, v)
}

test1 1
test2 2
```

- 遍历嵌套map时可以在for range里面再套一层for range

## sync.Map

golang中的map是并发不安全的，从go1.9开始，加入了sync.Map，用来解决map并发安全的问题

### map实现并发安全

- map需要加锁实现并发安全

```go
var m = make(map[string]int)
var rwlock sync.RWMutex

func get(key string) int {
	rwlock.RLock()
	s := m[key]
	rwlock.RUnlock()
	return s
}

func set(key string, value int) {
	rwlock.RLock()
	m[key] = value
	rwlock.RUnlock()
}

func main() {
	wg := sync.WaitGroup{}
	for i := 0; i < 21; i++ {
		wg.Add(1)
		go func(n int) {
			key := strconv.Itoa(n)
			set(key, n)
			fmt.Printf("key=%v,v=%v\n", key, get(key))
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```

- 但是如果map中数据量较大时，加锁则会影响性能

### sync.Map

```go
type Map struct {
	mu Mutex

	// read contains the portion of the map's contents that are safe for
	// concurrent access (with or without mu held).
	//
	// The read field itself is always safe to load, but must only be stored with
	// mu held.
	//
	// Entries stored in read may be updated concurrently without mu, but updating
	// a previously-expunged entry requires that the entry be copied to the dirty
	// map and unexpunged with mu held.
	read atomic.Value // readOnly

	// dirty contains the portion of the map's contents that require mu to be
	// held. To ensure that the dirty map can be promoted to the read map quickly,
	// it also includes all of the non-expunged entries in the read map.
	//
	// Expunged entries are not stored in the dirty map. An expunged entry in the
	// clean map must be unexpunged and added to the dirty map before a new value
	// can be stored to it.
	//
	// If the dirty map is nil, the next write to the map will initialize it by
	// making a shallow copy of the clean map, omitting stale entries.
	dirty map[interface{}]*entry

	// misses counts the number of loads since the read map was last updated that
	// needed to lock mu to determine whether the key was present.
	//
	// Once enough misses have occurred to cover the cost of copying the dirty
	// map, the dirty map will be promoted to the read map (in the unamended
	// state) and the next store to the map will make a new dirty copy.
	misses int
}
```

sync.Map通过read和dirty两个字段把读写分离，读的数据存到read上，写入的数据存到dirty字段

读的时候先查询read，查不到再查dirty

写的话只写入到dirty中

读read不加锁，读写dirty都加锁

通过misses来记录查询dirty的次数，超过指定的次数则将dirty中的数据同步到read中

- 通过上面可以了解到，sync.Map通过读写分离的方式解决并发安全问题，适用于读取比写入频率高的场景

### sync.Map中的方法

#### Store

- 用来新增数据

```go
func (m *Map) Store(key, value interface{}) {
```

```go
var m sync.Map
m.Store("test","a")
```

#### Load

- 返回指定key的val，如果key没有查到，val则被置为nil，ok会被标记为false

```
func (m *Map) Load(key interface{}) (value interface{}, ok bool) {
```

```go
var m sync.Map
m.Store("test","a")
valA,ok := m.Load("test")
if !ok {
    log.Println("key does not exist")
    return
}else {
    fmt.Println(valA) //a
}
```

- 查找不存在的key

```go
// 不存在的key
valB, ok := m.Load("test1")
if !ok {
	log.Println("key does not exist") //key does not exist
	return
}else {
	fmt.Println(valB)
}
```

#### Delete

- 删除指定的key

```go
func (m *Map) Delete(key interface{}) {
```

```go
var m sync.Map
m.Store("test","a")
m.Delete("test")
m.Delete("pass") //指定的key不存在时也不会报错
```

#### Range

- 用来遍历元素

```go
func (m *Map) Range(f func(key, value interface{}) bool) {
```

```go
var m sync.Map
m.Store("test1","a")
m.Store("test2","b")
m.Store("test3","c")

m.Range(func(key, value interface{}) bool {
    fmt.Printf("key:%v,val:%v\n",key,value)
    return true
})
//key:test1,val:a
//key:test2,val:b
//key:test3,val:c
```

#### LoadOrStore

- 传入一个key和val，返回val和bool
- 如果传入的key存在，则返回key的val和true，如果不存在，则存储传入的key和val，并返回val和false

```go
func (m *Map) LoadOrStore(key, value interface{}) (actual interface{}, loaded bool) {
```

```go
var m sync.Map
m.Store("test1", "a")

store, ok := m.LoadOrStore("test4", "d")
fmt.Println(store, ok) // d false

tStore, ok := m.LoadOrStore("test1", "1")
fmt.Println(tStore, ok) // a true
```



## 总结

map+sync.RWMutex使用于数据量比较小的场景

sync.Map适用于读比写频率高的场景

学习的还不够深入，继续加油



## 学习资料

Go SDK 1.14.4 sync.Map

https://qcrao91.gitbook.io/go/map

https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-hashmap/

https://blog.csdn.net/jiankunking/article/details/78808978

https://blog.csdn.net/u010230794/article/details/82143179


