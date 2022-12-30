## 一、概览

### 1、评估算法优劣的核心指标

（1）时间复杂度（流程决定）

（2）额外空间复杂度（流程决定）

（3）常数项时间：O(1)（细节决定）

常数时间的操作：如果一个操作的执行时间不以具体样本量为转移，每次执行时间都是固定的。如：算术运算（+、-、*、/、%）、位运算（|、&、^、>>、<<）、赋值、比较、自增减、数组寻址等。

### 2、如何确定算法流程的总操作数量与样本数量之间的表达式关系？

- 想象该算法流程所处理的数据状况，要按照最差的情况来，用 O(N) 表示；
- 把整个流程彻底拆分为一个个基本动作，保证每个动作都是常数时间的操作；
- 如果数据量为 N，看看基本动作的数量和 N 是什么关系。

### 3、对数器

1.有一个你想要测的方法a；
2.实现一个绝对正确但是复杂度不好的方法b；
3.实现一个随机样本产生器；
4.实现对比算法a和b的方法；
5.把方法a和方法b比对多次来验证方法a是否正确；
6.如果有一个样本使得比对出错，打印样本分析是哪个方法出错；
7.当样本数量很多时比对测试依然正确，可以确定方法a已经正确。

### 4、位运算

```go
/**
 * 位运算
 *  & 按位与
 *  | 按位或
 *  ^ 按位异或：a ^ b
 *  ^ 按位取反：^x
 *  >> 右移：M >> N = M / 2^N
 *  << 左移：M << N = M * 2^N
 */
```

> 异或运算的性质：

- 0 ^ N == N
- N ^ N == 0
- 异或运算满足交换律和结合律：a ^ b ^ c == a ^ c ^ b

#### （1）不用额外变量交换两个数

```go
// 不用额外的变量交换两个变量
func swap(x, y int) {
    fmt.Printf("Before: x = %v, y = %v \n", x, y)
    x = x ^ y // x = x ^ y
    y = x ^ y // y = (x ^ y) ^ y = x
    x = x ^ y // x = (x ^ y) ^ x = y
    fmt.Printf("After: x = %v, y = %v \n", x, y)

    /**
     * 类似：
     * a = a + b
     * b = a - b // b = (a+b)-b = a
     * a = a - b // a = (a+b)-a = b
     */
}
```

#### （2）数组中仅有一个数是奇数个，其余数都是偶数个，找出奇数个的那个数

```go
// 数组中仅有一个数是奇数个，其余数都是偶数个，找出奇数个的那个数
func onlyOneOdd() {
    arr := []int{1, 2, 3, 4, 3, 1, 2, 3, 4} // 3 出现了 3次

    var eor int
    for _, v := range arr {
        eor ^= v
    }

    // eor = 1 ^ 1 ^ 2 ^ 2 ^ 3 ^ 3 ^ 3 ^ 4 ^ 4 = 3
    fmt.Println(eor)
}
```

#### （3）保留 int 型二进制数中最右侧的 1

```go
// 保留 int 型二进制数中最右侧的 1
// 如：11111010 => 00000010
func retainRightmost1() {
    x := 250
    // x = 11111010
    fmt.Printf("x = %.8b\n", x)

    y := x & (^x + 1)
    // 取反       ^x = 00000101
    // +1    ^x + 1 = 00000110
    // x & (^x + 1) = 00000010

    // y = 00000010
    fmt.Printf("y = %.8b\n", y)
}
```

#### （4）数组中有两个数是奇数个，其余数都是偶数个，找出这两个数

```go
// 数组中有两个数是奇数个，其余数都是偶数个，找出这两个数
func twoOdd() {
    arr := []int{1, 1, 4, 4, 5, 9, 9, 9}

    // 假设那两个数分别为：a、b，由题意：a ≠ b
    // 将所有元素异或结果：eor = a ^ b ≠ 0
    var eor int
    for _, v := range arr {
        eor ^= v
    }
    // eor = 00001100
    fmt.Printf("eor = %.8b \n", eor)

    // 继续求 a、b 分别是什么
    // 因为 eor ≠ 0，所以其上至少有一个位置为 1
    // 根据异或性质 eor 结果为 1 的位置，a 和 b 的同位置一定 0 ^ 1
    // 保留 eor 上最右侧的 1
    eorRightmost1 := eor & (^eor + 1)
    // eorRightmost1 = 00000100
    fmt.Printf("eorRightmost1 = %.8b \n", eorRightmost1)

    // 将数组中的元素分为：与 eorRightmost1 同位置是 1 的元素和同位置是 0 的元素
    // 对与 eorRightmost1 同位置是 1 的元素集合异或
    // 其中偶数个的那些数不影响异或结果，即结果为 a 或 b
    with0Eor := 0
    for _, v := range arr {
        // 与 eorRightmost1 同位置是 0
        if v&eorRightmost1 == 0 {
            with0Eor ^= v
        }
    }

    // with0Eor 的结果为 a 或 b
    // 则另一个结果：another = with0Eor ^ eor = with1Eor ^ a ^ b
    // 当 with0Eor = a 时，another = b
    // 当 with0Eor = b 时，another = a

    fmt.Printf("这两个数分别是：%v, %v \n", with0Eor, eor^with0Eor)
}
```

#### （5）二进制数中 1 的数量

```go
// 二进制数中 1 的数量
func binary1Num() {
    x := 250
    count := 0
    // x = 11111010
    fmt.Printf("x = %.8b \n", x)

    for x != 0 {
        count++

        // 抹掉最右侧的 1
        // r1   = 00000010
        // x    = 11111010
        // x^r1 = 11111000
        r1 := x & (^x + 1)
        x ^= r1
    }

    // 1 的数量为：6
    fmt.Printf("1 的数量为：%v \n", count)
}
```



## 二、基础数据结构

### 1、单向链表

#### （1）反转链表

#### （2）把给定值都删除

```go
// 单向链表 - 节点
type SingleNode struct {
    Value int
    Next  *SingleNode
}

// 反转单链表
func reverseSingleList(head *SingleNode) *SingleNode {
    // 当前节点的前一个、后一个节点
    var prev, next *SingleNode

    for head != nil {
        // 记录下当前节点的 next
        next = head.Next
        // 指向前一个
        head.Next = prev
        // 准备进行下一轮
        prev = head
        head = next
    }

    // 迭代结束，此时 head、next 已全部指向 nil
    // prev 指向头
    return prev
}

// 把给定值都删除 - 单链表
func deleteGiveValueForSingleList(head *SingleNode, k int) *SingleNode {
    // 先确定新 head 的位置
    // 若前几个元素都是需要删除的，则跳到第一个不是 k 的位置
    for head != nil {
        if head.Value != k {
            break
        }

        head = head.Next
    }

    // prev 记录前一个非 k 节点
    prev := head
    // 上一轮中已确定 head 不是 k，所以直接从 head 的下一个开始判断
    cur := head

    for cur != nil {
        // 命中
        if cur.Value == k {
            // 跳过当前节点
            prev.Next = cur.Next
        } else {
            // 已命中时，当前节点即将删除，不能作为 prev，故不用更新
            // 未命中时，才将 prev 更新为当前节点，准备进行下一轮
            prev = cur
        }

        // 后移，进入下一轮
        cur = cur.Next
    }

    return head
}
```



### 2、双向链表

```go
// 双向链表 - 节点
type DoubleNode struct {
    Value int
    Prev  *DoubleNode
    Next  *DoubleNode
}

// 双向链表 - 实现
type List struct {
    Head *DoubleNode
    Tail *DoubleNode
}

// 头插
func (this *List) FrontPush(v int) {
    node := &DoubleNode{Value: v}

    if this.Head == nil {
        this.Head = node
        this.Tail = node
    } else {
        // 向链表头插入，处理前、后指针
        this.Head.Prev = node
        node.Next = this.Head
        // 头指针前移
        this.Head = node
    }
}

// 尾插
func (this *List) BackPush(v int) {
    node := &DoubleNode{Value: v}

    if this.Tail == nil {
        this.Head = node
        this.Tail = node
    } else {
        // 向链表尾插入，处理前、后指针
        this.Tail.Next = node
        node.Prev = this.Tail
        // 尾指针后移
        this.Tail = node
    }
}

// 头出
func (this *List) FrontPop() *DoubleNode {
    if this.Head == nil {
        return nil
    }

    // 保存头结点
    node := this.Head
    if this.Head.Next == nil {
        // 该节点为最后一个节点
        // 将 Head、Tail 置 nil
        this.Head = nil
        this.Tail = nil
    } else {
        // 头结点后移
        this.Head = this.Head.Next
        // 新头结点的 Prev 置 nil，原头结点丢失
        this.Head.Prev = nil
    }

    return node
}

// 尾出
func (this *List) BackPop() *DoubleNode {
    if this.Tail == nil {
        return nil
    }

    // 保存尾节点
    node := this.Tail
    if this.Tail.Prev == nil {
        // 该节点为最后一个节点
        // 将 Head、Tail 置 nil
        this.Head = nil
        this.Tail = nil
    } else {
        // 尾结点前移
        this.Tail = this.Tail.Prev
        // 新尾结点的 Next 置 nil，原尾结点丢失
        this.Tail.Next = nil
    }

    return node
}

// 反转双链表
func reverseDoubleList(head *DoubleNode) *DoubleNode {
    var prev, next *DoubleNode

    for head != nil {
        // 保留下一个节点
        next = head.Next
        // 反转前、后指针
        head.Prev = next
        head.Next = prev
        // 准备下一轮
        prev = head
        head = next
    }

    return prev
}

// 把给定值都删除 - 双链表
func deleteGiveValueForDoubleList(head *DoubleNode, k int) *DoubleNode {
    return nil
}
```



###  3、栈 & 队列

#### （1）用双向链表实现栈

```go
// 1、用双向链表实现栈
type StackWithList struct {
    stack List
}

// 入栈
func (this *StackWithList) Push(v int) {
    this.stack.FrontPush(v)
}

// 出栈
func (this *StackWithList) Pop() int {
    if this.Empty() {
        panic("Stack Empty...")
    }

    return this.stack.FrontPop().Value
}

// 判断是否为空
func (this *StackWithList) Empty() bool {
    if this.stack.Head == nil || this.stack.Tail == nil {
        return true
    }

    return false
}
```

#### （2）用数组实现栈

```go
// 2、用数组实现栈
type StackWithArray struct {
    stack []int
}

// 入栈
func (this *StackWithArray) Push(v int) {
    this.stack = append(this.stack, v)
}

// 出栈
func (this *StackWithArray) Pop() int {
    if this.Empty() {
        panic("Stack Empty...")
    }

    v := this.stack[len(this.stack)-1]
    this.stack = this.stack[:len(this.stack)-1]

    return v
}

// 判断是否为空
func (this *StackWithArray) Empty() bool {
    if len(this.stack) == 0 {
        return true
    }

    return false
}
```

#### （3）用双向链表实现队列

```go
// 3、用双向链表实现队列
type QueueWithList struct {
    queue List
}

// 入队
func (this *QueueWithList) Enqueue(v int) {
    this.queue.FrontPush(v)
}

// 出队
func (this *QueueWithList) Dequeue() int {
    if this.Empty() {
        panic("Queue Empty...")
    }

    return this.queue.BackPop().Value
}

// 判断是否为空
func (this *QueueWithList) Empty() bool {
    if this.queue.Head == nil || this.queue.Tail == nil {
        return true
    }

    return false
}
```

#### （4）用数组实现队列

```go
// 4、用数组实现队列
type QueueWithArray struct {
    queue   []int // 环形数组
    qSize   uint  // 队列大小
    bufSize uint  // 缓冲区大小
    enIdx   uint  // 入队位置，初始为 0
    deIdx   uint  // 出队位置，初始为 0
}

// 新建队列
func MakeQueue(size uint) QueueWithArray {
    return QueueWithArray{
        queue:   make([]int, size),
        qSize:   0,
        bufSize: size,
        enIdx:   0,
        deIdx:   0,
    }
}

// 入队
func (this *QueueWithArray) Enqueue(v int) {
    if this.qSize == this.bufSize {
        panic("Queue Full...")
    }

    // 存 入队位置
    this.queue[this.enIdx] = v
    // 加库存，下一个入队位置
    this.qSize++
    this.enIdx++
    // 环形数组实现
    if this.enIdx == this.bufSize {
        this.enIdx = 0
    }
}

// 出队
func (this *QueueWithArray) Dequeue() int {
    if this.Empty() {
        panic("Queue Empty...")
    }

    // 取 出队位置
    v := this.queue[this.deIdx]
    // 减库存，下一个出队位置
    this.qSize--
    this.deIdx++
    // 环形数组实现
    if this.deIdx == this.bufSize {
        this.deIdx = 0
    }

    return v
}

// 判断是否为空
func (this *QueueWithArray) Empty() bool {
    if this.qSize == 0 {
        return true
    }

    return false
}
```

#### （5）用两个队列实现栈

```go
// 5、用两个队列实现栈
type StackWithQueue struct {
    q1 QueueWithList // 主队
    q2 QueueWithList // 辅队
}

// 入栈
func (this *StackWithQueue) Push(v int) {
    // 入栈时，将数据写入非空的队列
    if !this.q1.Empty() {
        this.q1.Enqueue(v)
    } else {
        this.q2.Enqueue(v)
    }
}

// 出栈
func (this *StackWithQueue) Pop() int {
    if this.Empty() {
        panic("Stack Empty...")
    }

    // 出栈时，将非空队列的前 N-1 个元素导入到另一个队列
    // 弹出最后一个元素
    var last int
    if !this.q1.Empty() {
        for !this.q1.Empty() {
            // 先出队
            v := this.q1.Dequeue()
            // 如果出队后，队列为空，说明这是最后一个元素
            if this.q1.Empty() {
                last = v
                break
            }

            // 否则，丢到另一个队列
            this.q2.Enqueue(v)
        }
    } else {
        for !this.q2.Empty() {
            // 先出队
            v := this.q2.Dequeue()
            // 如果出队后，队列为空，说明这是最后一个元素
            if this.q2.Empty() {
                last = v
                break
            }

            // 否则，丢到另一个队列
            this.q1.Enqueue(v)
        }
    }

    return last
}

// 判断是否为空
func (this *StackWithQueue) Empty() bool {
    if this.q1.Empty() && this.q2.Empty() {
        return true
    }

    return false
}
```

#### （6）用两个栈实现队列

```go
// 6、用两个栈实现队列
type QueueWithStack struct {
    ins  StackWithArray // 入栈
    outs StackWithArray // 出栈
}

// 入队
func (this *QueueWithStack) Enqueue(v int) {
    // 入队只往 in 栈中插
    this.ins.Push(v)
}

// 出队
func (this *QueueWithStack) Dequeue() int {
    if this.Empty() {
        panic("Queue Empty...")
    }

    // 出栈时，先把 out 栈中的元素拿空
    // 再把 in 栈中的数据全部导入到 out 栈
    if this.outs.Empty() {
        for !this.ins.Empty() {
            this.outs.Push(this.ins.Pop())
        }
    }

    // 从 out 栈中弹出
    return this.outs.Pop()
}

// 判断是否为空
func (this *QueueWithStack) Empty() bool {
    if this.ins.Empty() && this.outs.Empty() {
        return true
    }

    return false
}
```



## 三、递归

递归思想：

递归的时间复杂度算法：

