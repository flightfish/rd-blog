# 经典排序
排序 | 平均时间复杂度|最佳时间复杂度|最差时间复杂度|空间复杂度|排序方式|稳定性
---|---|---|---|---|---|---
冒泡排序 | O(N^2）|O(N）|O(N^2）|O(1）|In-place|稳定
插入排序 | O(N^2）|O(N）|O(N^2）|O(1）|In-place|稳定
选择排序 | O(N^2）|O(N）|O(N^2）|O(1）|In-place|不稳定
希尔排序 | O(NlogN)|O(N）|O(N^2）|O(1）|In-place|不稳定


```
   /*
   发现无论什么排序。都需要对满足条件的元素进行位置置换。
   所以可以把这部分相同的代码提取出来，单独封装成一个函数。
   */
    public static void swap(int[] arr,int a,int b)
    {
        int temp = arr[a];
        arr[a] = arr[b];
        arr[b] = temp;
    }
```



# 冒泡排序
## Java语言版本
```java
public static int[] bubbleSort(int[] arr) {
        if(arr == null || arr.length <= 1){
            return arr;
        }
        int len = arr.length;
        int temp;
        for (int i = 0; i < len-1; i++) {
            for (int j = 0; j < len - 1 - i; j++) {
                if (arr[j] > arr[j + 1]) {
                    swap(arr,j,j+1)
                }
            }
        }
        return arr;
    }
    
    //通过设置标志位来记录此次遍历有无数据交换，进而可以判断是否要继续循环，设置一个flag标记，
    //当在一趟序列中没有发生交换，则该序列已排序好，但优化后排序的时间复杂度没有发生量级的改变。

    public static int[] bubbleSort1(int[] arr) {
        if(arr == null || arr.length <= 1){
            return arr;
        }
        int len = arr.length;
        int temp;
        for (int i = 0; i < len - 1; i++) {
            boolean flag = true;
            for (int j = 0; j < len - 1 - i; j++) {
                if (arr[j] > arr[j + 1]) {
                    swap(arr,j,j+1)
                    flag = false;
                }
            }
            if (flag) break;
        }
        return arr;
    }
    
    //记录某次遍历时最后发生数据交换的位置pos，这个位置之后的数据显然已经有序了。
    //因此通过记录最后发生数据交换的位置就可以确定下次循环的范围了。由于pos位置之后的记录均已交换到位,故在进行下一趟排序时只要扫描到pos位置即可。

    public static int[] bubbleSort2(int[] arr) {
        if(arr == null || arr.length <= 1){
            return arr;
        }
        int len = arr.length;
        int temp, flag, k;
        flag = len;
        while (flag > 0) {
            k = flag;
            flag = 0;
            for (int j = 0; j < k - 1; j++) {
                if (arr[j] > arr[j + 1]) {
                     swap(arr,j,j+1)
                    flag = j + 1;
                }
            }
        }
        return arr;
    }

```

## go语言版本

# 插入排序
## Java语言版本

```java
public static int[]  insertSort(int[] arr){
        if (arr == null || arr.length <= 1) {
            return arr;
        }
        int len = arr.length;
        int temp, j;
        for (int i = 1; i < len; i++) {
            temp = arr[i];
            for (j = i - 1; j >= 0; j--) {
                //如果比tmp大把值往后移动一位
                if (arr[j] > temp) {
                    arr[j + 1] = arr[j];
                } else {
                    break;
                }
            }
            arr[j + 1] = temp;
        }
        return arr;
    }
    
① 从第一个元素开始，该元素可以认为已经被排序
② 取出下一个元素，在已经排序的元素序列中二分查找到第一个比它大的数的位置
③ 将新元素插入到该位置后
④ 重复上述两步
//二分插入排序相对直接插入排序而言：平均性能更快，时间复杂度降至O(NlogN)，排序是稳定的，
//但排序的比较次数与初始序列无关，相比直接插入排序，在速度上有一定提升。

    public static int[] BinaryInsertSort(int[] arr) {
        if (arr == null || arr.length <= 1) {
            return arr;
        }
        int len = arr.length;
        int key,left,right,middle;

        for (int i = 1; i <len ; i++) {
            key=arr[i];
            left=0;
            right=i-1;
            while (left<=right){
                middle=(left+right)/2;
                if(arr[middle]>key){
                    right=middle-1;
                }else{
                    left=middle+1;
                }
            }
            for(int j=i-1; j>=left; j--)
            {
                arr[j+1] = arr[j];
            }
            arr[left] = key;
        }
        return arr;
    }

```
## go语言版本

# 希尔排序
## java语言版本

```java
    private static int[] shellShort(int[] arr){
        if(arr == null || arr.length <= 1){
            return arr;
        }
        int i, j, len = arr.length;
        int temp;

        for (int gap = len / 2; gap > 0; gap /= 2) {
            System.out.println("增量取值:" + gap);
            for (i = gap; i < len; i++) {
                temp = arr[i];
                for (j = i - gap; j >= 0 ; j -= gap) {
                    if(arr[j] > temp)
                    arr[j + gap] = arr[j];
                }
                arr[j + gap] = temp;
            }
        }
        return arr;
    }
```
## go语言版本

# 选择排序
## java语言版本
```java
 public static  int[] selectionSort(int[] arr){
        if (arr == null || arr.length <= 1) {
            return arr;
        }
        int i, j, min, temp, len = arr.length;
        for (i = 0; i < len - 1; i++) {
            min = i;
            for (j = i + 1; j < len; j++) {
                if (arr[min] > arr[j]) {
                    min = j;
                }
            }
            temp = arr[min];
            arr[min] = arr[i];
            arr[i] = temp;
        }
        return arr;
    }

①二元选择排序

改进思路： 简单选择排序，每趟循环只能确定一个元素排序后的定位。根据之前冒泡排序的经验，我们可以考虑改进为每趟循环确定两个元素（当前趟最大和最小记录）的位置,从而减少排序所需的循环次数。改进后对n个数据进行排序，最多只需进行[n/2]趟循环即可。

②堆排序

堆排序是一种树形选择排序，是对直接选择排序的有效改进。具体的分析我们留到后面讲堆排序时再详细说明。
```
## go语言版本

# 快速排序

## php语言版本
```php
function quickSort($arr)
{
    $count = count($arr);

    if ($count < 2) {
        return $arr;
    }

    $leftArray = $rightArray = array();
    $middle = $arr[0];// 基准值

    for ($i = 1; $i < $count; $i++) {
        // 小于基准值，存入左边；大于基准值，存入右边
        if ($arr[$i] < $middle) {
            $leftArray[] = $arr[$i];
        } else {
            $rightArray[] = $arr[$i];
        }
    }

    $leftArray = quickSort($leftArray);
    $rightArray = quickSort($rightArray);

    return array_merge($leftArray, array($middle), $rightArray);
    // 倒序
    // return array_merge($rightArray, array($middle), $leftArray);
}
```
## go语言版本
```go
//递归 
func quickSort(arr []int,low,high int)  {
	if low>=high{
		return
	}
	mid:=partition(arr,low,high)
	quickSort(arr,low,mid-1)
	quickSort(arr,mid+1,high)
}
//迭代
func quickSort(arr []int,low,high int)  {
	var stack []int
	stack=append(stack,0,len(arr)-1)
	for len(stack)>0 {
		low:=stack[0]
		high:=stack[1]
		stack=stack[2:]
		if low >= high{
			continue
		}
		mid:=partition(arr,low,high)
		stack=append(stack,low,mid-1)
		stack=append(stack,mid+1,high)
	}
}


func partition(arr []int,low,high int)int  {
	p:=arr[low]
	i,j:=low,high
	for i<j{
		for arr[j]>=p&&i<j {
			j--
		}
		for arr[i]<=p&&i<j {
			i++
		}
		arr[i],arr[j]=arr[j],arr[i]
	}
	arr[low],arr[i]=arr[i],arr[low]
	return i
}
```
## java语言版本
```java
//迭代
public static void quickSort2(int[] arr){
        int len= arr.length;
        Stack<Integer> stack = new Stack<>();
        stack.push(0);
        stack.push(len-1);
        while (!stack.isEmpty()){
            int end=stack.pop();
            int start=stack.pop();

            if(low >= high){
                continue;
            }
            //int mid=(start+end)/2;
            int mid;
            mid=partition(arr,start,end);
            stack.push(mid + 1);
            stack.push(end);

            stack.push(start);
            stack.push(mid-1);
        }
    }
    //递归
    public static void quickSort1(int[] arr,int low,int high){
        if (low >= high) {
            return;
        }
        int mid = partition(arr, low, high);
        quickSort1(arr, low, mid - 1);
        quickSort1(arr, mid + 1, high);
    }
    /*
     * Utility method to partition the array into smaller array, and
     * comparing numbers to rearrange them as per quicksort algorithm.
     */
    private static int partition(int[] arr, int low, int high) {
        int p,i,j;
        p = arr[low];
        i = low;
        j = high;
        while(i < j) {
            //右边当发现小于p的值时停止循环
            while(arr[j] >= p && i < j) {
                j--;
            }
            //这里一定是右边开始，上下这两个循环不能调换（下面有解析，可以先想想）

            //左边当发现大于p的值时停止循环
            while(arr[i] <= p && i < j) {
                i++;
            }

            swap(arr,i,j);
        }
        if(arr[low] > arr[i]){
            arr[low] = arr[i];//这里的arr[i]一定是停小于p的，经过i、j交换后i处的值一定是小于p的(j先走)
            arr[i] = p;
        }
        return i;
    }


```

# 归并排序

## java语言版本
```java
递归
 public static void mergeSort(int[] a, int low, int high) {
        int mid = (low + high) / 2;
        if (low < high) {
            // 左边
            mergeSort(a, low, mid);
            // 右边
            mergeSort(a, mid + 1, high);

            // 左右归并
            int[] temp = new int[high - low + 1];
            int i = low;// 左指针
            int j = mid + 1;// 右指针
            int k = 0;
            // 把较小的数先移到新数组中
            while (i <= mid && j <= high) {
                if (a[i] < a[j]) {
                    temp[k++] = a[i++];
                } else {
                    temp[k++] = a[j++];
                }
            }
            // 把左边剩余的数移入数组
            while (i <= mid) {
                temp[k++] = a[i++];
            }
            // 把右边边剩余的数移入数组
            while (j <= high) {
                temp[k++] = a[j++];
            }
            // 把新数组中的数覆盖nums数组
            for (int k2 = 0; k2 < temp.length; k2++) {
                a[k2 + low] = temp[k2];
            }
            System.out.println(Arrays.toString(a));
        }

    }
    
    // 归并排序（Java-迭代版）
public static void merge_sort(int[] arr) {
    int len = arr.length;
    int[] result = new int[len];
    int block, start;

    // 原版代码的迭代次数少了一次，没有考虑到奇数列数组的情况
    for(block = 1; block < len; block *= 2) {
        for(start = 0; start <len; start += 2 * block) {
            int low = start;
            int mid = (start + block) < len ? (start + block) : len;
            int high = (start + 2 * block) < len ? (start + 2 * block) : len;
            //两个块的起始下标及结束下标
            int start1 = low, end1 = mid;
            int start2 = mid, end2 = high;
            //开始对两个block进行归并排序
            while (start1 < end1 && start2 < end2) {
            result[low++] = arr[start1] < arr[start2] ? arr[start1++] : arr[start2++];
            }
            while(start1 < end1) {
            result[low++] = arr[start1++];
            }
            while(start2 < end2) {
            result[low++] = arr[start2++];
            }
        }
    int[] temp = arr;
    arr = result;
    result = temp;
    }
    result = arr;       
}
```
## go语言版本
```go
//递归
func mergeSort(arr []int,low,high int)  {
	if low>=high{
		return
	}
	mid:=(low+high)>>1
	mergeSort(arr,low,mid)
	mergeSort(arr,mid+1,high)
	merge(arr,low,mid,high)
}
func merge(arr []int,low,mid,high int)  {
	temp:=make([]int,high-low+1)
	i,j,k:=low,mid+1,0
	for i<=mid&&j<=high{
		if arr[i]<=arr[j]{
			temp[k]=arr[i]
			i++
		}else{
			temp[k]=arr[j]
			j++
		}
		k++
	}
	for i<=mid{
		temp[k]=arr[i]
		i++
		k++
	}
	for j<=high{
		temp[k]=arr[j]
		j++
		k++
	}
	for i := low; i <=high ; i++ {
		arr[i]=temp[i-low]
	}
	fmt.Println(arr)
}
//迭代
func mergeSort(arr []int,low,high int)  {
	if low>=high{
		return
	}

	n:=high-low+1
	result:=make([]int,n)
	for block := 1; block <n ; block*=2 {
		for start := 0; start <n ; start+= 2 * block {
			low:=start
			mid:=start + block
			if mid>n{
				mid=n
			}
			high:=start + 2*block
			if high>n{
				high=n
			}
			 start1,end1 := low, mid
			 start2,end2:= mid,high
			for start1 < end1 && start2 < end2 {
				if arr[start1] < arr[start2]{
					result[low]=arr[start1]
					start1++
				}else{
					result[low]=arr[start2]
					start2++
				}
				low++
			}
			for start1 < end1 {
				result[low] = arr[start1]
				start1++
				low++
			}
			for start2 < end2 {
				result[low] = arr[start2]
				start2++
				low++
			}
			fmt.Println(result)
		}
		copy(arr,result)

	}
}

```

# 桶排序

## java语言版本
```java
public static void bucketSort(int[] arr){
	
    int max = Integer.MIN_VALUE;
    int min = Integer.MAX_VALUE;
    for(int i = 0; i < arr.length; i++){
        max = Math.max(max, arr[i]);
        min = Math.min(min, arr[i]);
    }
	
    //桶数
    int bucketNum = (max - min) / arr.length + 1;
    ArrayList<ArrayList<Integer>> bucketArr = new ArrayList<>(bucketNum);
    for(int i = 0; i < bucketNum; i++){
        bucketArr.add(new ArrayList<Integer>());
    }
	
    //将每个元素放入桶
    for(int i = 0; i < arr.length; i++){
        int num = (arr[i] - min) / (arr.length);
        bucketArr.get(num).add(arr[i]);
    }
	
    //对每个桶进行排序
    for(int i = 0; i < bucketArr.size(); i++){
        Collections.sort(bucketArr.get(i));
    }
	
    System.out.println(bucketArr.toString());
	
}
```
## go语言版本
```go
func bucketSort(arr []int)  {
	minNum,maxNum,n:=math.MaxInt32,math.MinInt32,len(arr)
	for i := 0; i <n ; i++ {
		maxNum=max(maxNum,arr[i])
		minNum=min(minNum,arr[i])
	}

	//桶数
	bucketNum:=(maxNum-minNum)/n+1
	bucketArr:=make([][]int,bucketNum)
	for i := 0; i <bucketNum ; i++ {
		bucketArr[i]=[]int{}
	}

	//将每个元素放入桶
	for i := 0; i < n; i++ {
		num := (arr[i] - minNum) / n
		bucketArr[num] = append(bucketArr[num], arr[i])
	}

	k:=0
	//对每个桶进行排序
	for i := 0; i <bucketNum ; i++ {
		//归并
		mergeSort(bucketArr[i],0,len(bucketArr[i])-1)
		for j := 0; j <len(bucketArr[i]) ; j++ {
			arr[k]=bucketArr[i][j]
			k++
		}
	}
}
func max (x,y int)int{
	if x>y {
		return x
	}
	return y
}
func min (x,y int)int{
	if x>y {
		return y
	}
	return x
}
```

# 计数排序
## java语言版本
```java
// 计数排序。假设数组中存储的都是非负整数。
  public static void countSort(int[] arr){
        int len = arr.length;
        if (len <= 1) return;
        int max = arr[0];

        for (int value : arr) {
            if (max < value) max = value;
        }

        int[] count = new int[max+1];
        for (int i = 0; i < max; i++) {
            count[i] = 0;
        }


        for (int value : arr) {
            count[value]++;
        }

        for (int i = 1; i < max+1; i++) {
            count[i] = count[i] + count[i - 1];
        }

        int[] newArr=new int[len];
        for (int i = len-1; i >=0 ; i--) {
            int index=count[arr[i]]-1;
            newArr[index]=arr[i];
            count[arr[i]]--;
        }
        for (int i = 0; i <len ; i++) {
            arr[i]=newArr[i];
        }
    }
```
## go语言版本
```go
func countSort(arr []int)  {
	n:=len(arr)
	if n<=1{
		return
	}
	maxNum:=arr[0]
	for i := 0; i <n ; i++ {
		maxNum=max(maxNum,arr[i])
	}
	c:=make([]int,maxNum+1)
	for _, v := range arr {
		c[v]++
	}
	for i := 0; i <maxNum ; i++ {

	}
	for i := 1; i <=maxNum; i++ {
		c[i]+=c[i-1]
	}
	r:=make([]int,n)
	for i := n-1; i >=0 ; i-- {
		index:=c[arr[i]]-1
		r[index]=arr[i]
		c[arr[i]]--
	}
	copy(arr,r)
}

```

# 基数排序
## java语言版本
```java
 public static void radixSort(int[] arr) {
        int len = arr.length;
        int d = maxbit(arr);
        System.out.println(d);

        int[] count = new int[10];
        int[] newArr = new int[len];
        int radix = 1, k;
        while (d >= 1) {

            for (int i = 0; i < 10; i++) {
                count[i] = 0;
            }
            for (int i = 0; i < len; i++) {
                k = (arr[i] / radix) % 10;
                count[k]++;
            }
            for (int i = 1; i < 10; i++) {
                count[i] = count[i] + count[i - 1];
            }

            for (int i = len - 1; i >= 0; i--) {
                k = (arr[i] / radix) % 10;
                newArr[count[k] - 1] = arr[i];
                count[k]--;

            }
            for (int i = 0; i < len; i++) {
                arr[i] = newArr[i];
            }
            System.out.println(Arrays.toString(arr));
            d--;
            radix *= 10;
        }
    }

    private static int maxbit(int[] arr) {
        int max = arr[0];
        for (int val : arr) {
            if (val > max) max = val;
        }
        int d = 1;
        while (max >= 10) {
            max /= 10;
            d++;
        }
        return d;
    }
```
## go语言版本
```go
func radixSort(arr []int)  {
	n:=len(arr)
	if n<=1{
		return
	}
	d:=maxbit(arr)
	radix:=1
	var k int
	count,newArr := make([]int,10),make([]int,n)
	for d>=1 {
		for  i:= 0; i < 10; i++ {
			count[i] = 0
		}
		for i := 0; i <n ; i++ {
			k = (arr[i] / radix) % 10
			count[k]++
		}
		for i := 1; i < 10; i++ {
			count[i]+=count[i-1]
		}
		for i := n-1; i >=0 ; i-- {
			k = (arr[i] / radix) % 10
			newArr[count[k]-1]=arr[i]
			count[k]--
		}
		d--
		radix*=10
		copy(arr,newArr)
	}

}
func maxbit(arr []int) int {
	maxNum:=math.MinInt32
	for _, val := range arr {
		if val > maxNum{
			maxNum=val
		}
	}
	d:=1
	for maxNum>=10 {
		d++
		maxNum/=10
	}
	return d
}

```

# 堆排序
## java语言版本
```java
  public static void buildHeap(int[] arr){
        int n=arr.length-1;
        for (int i = (n-1)/2; i >=0 ; i--) {
            heapify(arr,n,i);

        }
    }

    private static void heapify(int[] a, int n, int i) {
        while (true) {
            int maxPos = i;
            if (i * 2+1 <= n && a[i] < a[i * 2+1]) maxPos = i * 2+1;
            if (i * 2 + 2 <= n && a[maxPos] < a[i * 2 + 2]) maxPos = i * 2 + 2;
            if (maxPos == i) break;
            swap(a, i, maxPos);
            i = maxPos;
        }

    }

    public static void heapSort(int[] arr){
        buildHeap(arr);//建堆

        int k=arr.length-1;
        while(k>0){
            swap(arr,0,k);
            k--;
            heapify(arr, k, 0);//堆化
        }
    }
```
## go语言版本
```go
func heapSort(arr []int)  {
	n:=len(arr)
	if n<=1{
		return
	}
	buildHeap(arr)
	k := n-1
	for k >0 {
		arr[0],arr[k]=arr[k],arr[0]
		k--
		heapify(arr,k,0)
	}
}
func buildHeap(arr []int)  {
	n:=len(arr)-1
	for i := (n-1)/2; i >=0 ; i-- {
		heapify(arr,n,i)
	}
}
func heapify(arr []int,n,i int)  {
	for  {
		maxPos:=i
		if 2*i+1<=n&&arr[maxPos]<arr[2*i+1]{
			maxPos=2*i+1
		}
		if 2*i+2<=n&&arr[maxPos]<arr[2*i+2]{
			maxPos=2*i+2
		}
		if maxPos==i{
			break
		}
		arr[i],arr[maxPos]=arr[maxPos],arr[i]
		i=maxPos
	}
}

```








