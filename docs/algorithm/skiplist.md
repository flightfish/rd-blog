# 跳表

- 链表加多级索引的结构，就是跳表
- 跳表中查询任意数据的时间复杂度就是 O(logn)
- 跳表是通过随机函数来维护平衡性

跳表使用空间换时间的设计思路，通过构建多级索引来提高查询的效率，实现了基于链表的“二分查找”。跳表是一种动态数据结构，支持快速地插入、删除、查找操作，
时间复杂度都是 O(logn)。跳表的空间复杂度是 O(n)。不过，跳表的实现非常灵活，可以通过改变索引构建策略，有效平衡执行效率和内存消耗。虽然跳表的代码实现并不简单，
但是作为一种动态数据结构，比起红黑树来说，实现要简单多了。所以很多时候，我们为了代码的简单、易读，比起红黑树，我们更倾向用跳表。

>Redis 中的有序集合是通过跳表来实现的和散列表。
>Redis 之所以用跳表来实现有序集合，还有其他原因，比如，跳表更容易代码实现。虽然跳表的实现也不简单，但比起红黑树来说还是好懂、好写多了，
>而简单就意味着可读性好，不容易出错。还有，跳表更加灵活，它可以通过改变索引构建策略，有效平衡执行效率和内存消耗。
```go

const (
	//最高层数
	MaxLevel  = 16
	SkipListP = 0.5
)
//跳表节点结构体
type skipListNode struct {
	//跳表保存的值
	v interface{}
	//用于排序的分值
	score int
	//层高
	level int
	//每层前进指针
	forwards []*skipListNode
}
//新建跳表节点
func newSkipListNode(v interface{}, score, level int) *skipListNode {
	return &skipListNode{v: v, score: score, forwards: make([]*skipListNode, level, level), level: level}
}
//跳表结构体
type SkipList struct {
	//跳表头结点
	head *skipListNode
	//跳表当前层数
	level int
	//跳表长度
	length int
}

//实例化跳表对象
func NewSkipList() *SkipList {
	//头结点，便于操作
	head := newSkipListNode(0, math.MinInt32, MaxLevel)
	return &SkipList{head, 1, 0}
}
//获取跳表长度
func (sl *SkipList) Length() int {
	return sl.length
}
//获取跳表层级
func (sl *SkipList) Level() int {
	return sl.level
}
// 理论来讲，一级索引中元素个数应该占原始数据的 50%，二级索引中元素个数占 25%，三级索引12.5% ，一直到最顶层。
// 因为这里每一层的晋升概率是 50%。对于每一个新插入的节点，都需要调用 randomLevel 生成一个合理的层数。
// 该 randomLevel 方法会随机生成 1~MAX_LEVEL 之间的数，且 ：
//        50%的概率返回 1
//        25%的概率返回 2
//      12.5%的概率返回 3 ...
func (sl *SkipList) randomLevel() int {
	level := 1
	for rand.Float32() < SkipListP && level < MaxLevel {
		level += 1
	}
	return level
}

//插入节点到跳表中
func (sl *SkipList) Insert(v interface{}, score int)  {
	level:=sl.randomLevel()
	//创建一个新的跳表节点
	newNode := newSkipListNode(v, score, level)

	//查找插入位置
	cur := sl.head
	//记录每层的路径
	update := make([]*skipListNode,level)
	i := level - 1
	for ; i >= 0; i-- {
		for nil != cur.forwards[i] {
			if cur.forwards[i].score > score {
				update[i] = cur
				break
			}
			cur = cur.forwards[i]
		}
		if nil == cur.forwards[i] {
			update[i] = cur
		}
	}

	//原有节点连接
	for i := 0; i <= level-1; i++ {
		next := update[i].forwards[i]
		update[i].forwards[i] = newNode
		newNode.forwards[i] = next
	}

	//如果当前节点的层数大于之前跳表的层数
	//更新当前跳表层数
	if level > sl.level {
		sl.level = level
	}

	//更新跳表长度
	sl.length++
}

//查找
func (sl *SkipList) Find(v interface{}, score int) *skipListNode {
	if nil == v || sl.length == 0 {
		return nil
	}

	cur := sl.head
	for i := sl.level - 1; i >= 0; i-- {
		for nil != cur.forwards[i] {
			if cur.forwards[i].score == score && cur.forwards[i].v == v {
				return cur.forwards[i]
			} else if cur.forwards[i].score > score {
				break
			}
			cur = cur.forwards[i]
		}
	}

	return nil
}

//删除节点
func (sl *SkipList) Delete(v interface{}, score int) {
	if nil == v {
		return
	}

	//查找前驱节点
	cur := sl.head
	//记录前驱路径
	update := make([]*skipListNode,sl.level)
	for i := sl.level - 1; i >= 0; i-- {
		update[i] = sl.head
		for nil != cur.forwards[i] {
			if cur.forwards[i].score == score && cur.forwards[i].v == v {
				update[i] = cur
				break
			}
			cur = cur.forwards[i]
		}
	}

	cur = update[0].forwards[0]
	for i := cur.level - 1; i >= 0; i-- {
		if update[i] == sl.head && cur.forwards[i] == nil {
			sl.level = i
		}

		if nil == update[i].forwards[i] {
			update[i].forwards[i] = nil
		} else {
			update[i].forwards[i] = update[i].forwards[i].forwards[i]
		}
	}

	sl.length--
}

func (sl *SkipList) String() string {
	return fmt.Sprintf("level:%+v, length:%+v", sl.level, sl.length)
}

```