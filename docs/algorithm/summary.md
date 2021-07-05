# 经典算法
# Dijkstra算法

# FLoyd算法
- [399. 除法求值](https://leetcode-cn.com/problems/evaluate-division/)
- [最短路径模板+解析——（FLoyd算法）](https://blog.csdn.net/ytuyzh/article/details/88617987)

# 求余问题
- [剑指 Offer 14- II. 剪绳子 II](https://leetcode-cn.com/problems/jian-sheng-zi-ii-lcof/)

大数求余解法
- 大数越界： 当 a 增大时，最后返回的 3^a大小以指数级别增长，可能超出 int32 甚至 int64 的取值范围，导致返回值错误。
- 大数求余问题： 在仅使用 int32 类型存储的前提下，正确计算 x^a
-  循环求余 、 快速幂求余 ，其中后者的时间复杂度更低，两种方法均基于以下求余运算规则推出：
    (xy)⊙p=[(x⊙p)(y⊙p)]⊙p

```go 
//求 (x^a) % p —— 循环求余法
func remainder(x,a,p int)int  {
	rem:=1
	for i := 0; i <a ; i++ {
		rem = (rem * x) % p
	}
	return rem
}




//求 (x^a) % p —— 快速幂求余
// 本算法的时间复杂度为O（logb），能在几乎所有的程序设计（竞赛）过程中通过
func remainder(x, a, p int) int {
	rem := 1
	for a > 0 {
		if a & 1>0 {//a的二进制和1的二进制进行位的按位与运算，其实等价于if(a%2==1)起到判断奇偶的功能，但计算机的位运算比较快。            
			rem = (rem * x) % p
		}
		x = (x * x) % p
        a >>= 1//a=a>>1 a的二进制数右移一位赋值给a，右移时高位空缺补零。类比十进制数100右移相当于100/10=10;a>>1 等价于a/2;但计算机的位运算比较快
	}
	return rem
}


```

# 离散化树状数组
```go
func countSmaller(nums []int) []int {
	n:=len(nums)
	set := map[int]struct{}{}
	for _, num := range nums {
		set[num] = struct{}{}
	}
	tmp := make([]int, 0,n)
	for num := range set {
		tmp = append(tmp, num)
	}
	sort.Ints(tmp)
	for i := 0; i < n; i++ {
		nums[i] = sort.SearchInts(tmp, nums[i]) + 1
	}

	bit := BIT{
		n: len(nums),
		tree: make([]int, n + 1),
	}
	resultList := []int{}
	for i := len(nums) - 1; i >= 0; i-- {
		id := nums[i]
		resultList = append(resultList, bit.query(id - 1))
		bit.update(id)
	}
	for i := 0; i < len(resultList)/2; i++ {
		resultList[i], resultList[len(resultList)-1-i] = resultList[len(resultList)-1-i], resultList[i]
	}
	return resultList
}


type BIT struct {
	n    int
	tree []int
}

func (b BIT) lowBit(x int) int {
	return x & (-x)
}
func (b BIT) query(x int) int {
	ret := 0
	for x > 0 {
		ret += b.tree[x]
		x -= b.lowBit(x)
	}
	return ret
}

func (b BIT) update(x int) {
	for x <= b.n {
		b.tree[x]++
		x += b.lowBit(x)
	}
}
```
- [315. 计算右侧小于当前元素的个数](https://leetcode-cn.com/problems/count-of-smaller-numbers-after-self/)
- [剑指 Offer 51. 数组中的逆序对](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/submissions)

# 背包问题
## 01 背包问题：
    最基本的背包问题就是 01 背包问题：一共有 N 件物品，第 i（i 从 1 开始）件物品的重量为 w[i]，价值为 v[i]。在总重量不超过背包承载上限 W 的情况下，能够装入背包的最大价值是多少？
## 完全背包问题
    完全背包与 01 背包不同就是每种物品可以有无限多个：一共有 N 种物品，每种物品有无限多个，第 i（i 从 1 开始）种物品的重量为 w[i]，价值为 v[i]。在总重量不超过背包承载上限 W 的情况下，能够装入背包的最大价值是多少？
    可见 01 背包问题与完全背包问题主要区别就是物品是否可以重复选取。
## 背包问题具备的特征
    是否可以根据一个 target（直接给出或间接求出），target 可以是数字也可以是字符串，再给定一个数组 arrs，问：能否使用 arrs 中的元素做各种排列组合得到 target。
    
## 背包问题解法：
#### 01 背包问题：
 如果是 01 背包，即数组中的元素不可重复使用，外循环遍历 arrs，内循环遍历 target，且内循环倒序:

#### 完全背包问题：
（1）如果是完全背包，即数组中的元素可重复使用并且不考虑元素之间顺序，arrs 放在外循环（保证 arrs 按顺序），target在内循环。且内循环正序。
（2）如果组合问题需考虑元素之间的顺序，需将 target 放在外循环，将 arrs 放在内循环，且内循环正序。

## 例题
#### 01 背包问题：
- [416. 分割等和子集](https://leetcode-cn.com/problems/partition-equal-subset-sum/)
- [494. 目标和](https://leetcode-cn.com/problems/target-sum/)
#### 完全背包问题
- [139. 单词拆分](https://leetcode-cn.com/problems/word-break/)
- [279. 完全平方数](https://leetcode-cn.com/problems/perfect-squares/)


# 卡塔兰数

# 并查集
