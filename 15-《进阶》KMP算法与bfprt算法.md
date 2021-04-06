[TOC]

# 1 KMP算法

`大厂劝退，面试高频^_^`

## 1.1 KMP算法分析

查找字符串问题：例如我们有一个字符串str="abc1234efd"和match="1234"。我们如何查找str字符串中是否包含match字符串的子串？


> 暴力解思路：循环str和match，挨个对比，最差情况为O(N*M)。时间复杂度为O(N*M)

> KMP算法，在N大于M时，可以在时间复杂度为O(N)解决此类问题


我们对str记录字符坐标前的前缀后缀最大匹配长度，例如str="abcabck"

1、对于k位置前的字符，前后缀长度取1时，前缀为"a"后缀为"c"不相等

2、对于k位置前的字符，前后缀长度取2时，前缀为"ab"后缀为"bc"不相等

3、对于k位置前的字符，前后缀长度取3时，前缀为"abc"后缀为"abc"相等

4、对于k位置前的字符，前后缀长度取4时，前缀为"abca"后缀为"cabc"不相等

5、对于k位置前的字符，前后缀长度取5时，前缀为"abcab"后缀为"bcabc"不相等

==注意前后缀长度不可取k位置前的整体长度6。那么此时k位置前的最大匹配长度为3==

所以，例如"aaaaaab","b"的坐标为6，那么"b"坐标前的前后缀最大匹配长度为5


我们对match建立坐标前后缀最大匹配长度数组，概念不存在的设置为-1，例如0位置前没有字符串，就为-1，1位置前只有一个字符，前后缀无法取和坐标前字符串相等，规定为0。例如"aabaabc"，nextArr[]为[-1,0,1,0,1,2,3]


> 暴力方法之所以慢，是因为每次比对，如果match的i位置前都和str匹配上了，但是match的i+1位置没匹配成功。那么str会回退到第一次匹配的下一个位置，match直接回退到0位置再次比对。str和match回退的位置太多，之前的信息全部作废，没有记录


> 而KMP算法而言，如果match的i位置前都和str匹配上了，但是match的i+1位置没匹配成功，那么str位置不回跳，match回跳到当前i+1位置的最大前后缀长度的位置上，去和当前str位置比对。

原理是如果我们当前match位置i+1比对失败了，我们跳到最大前后缀长度的下一个位置去和当前位置比对，如果能匹配上，由于i+1位置之前都匹配的上，那么match的最大后缀长度也比对成功，可以被我们利用起来。替换成match的前缀长度上去继续对比，起到加速的效果


那么为什么str和match最后一个不相等的位置，之前的位置无法配出match，可以反证，如果可以配置出来，那么该串的头信息和match的头信息相等，得出存在比match当前不等位置最大前后缀还要大的前后缀，矛盾

Code:

```Java
public class Code01_KMP {
    	// O(N)
	public static int getIndexOf(String s, String m) {
		if (s == null || m == null || m.length() < 1 || s.length() < m.length()) {
			return -1;
		}
		char[] str = s.toCharArray();
		char[] match = m.toCharArray();
		int x = 0; // str中当前比对到的位置
		int y = 0; // match中当前比对到的位置
		// match的长度M，M <= N   O(M)
		int[] next = getNextArray(match); // next[i]  match中i之前的字符串match[0..i-1],最长前后缀相等的长度
		// O(N)
        	// x在str中不越界，y在match中不越界
		while (x < str.length && y < match.length) {
            		// 如果比对成功，x和y共同往各自的下一个位置移动
			if (str[x] == match[y]) {
				x++;
				y++;
			} else if (next[y] == -1) { // 表示y已经来到了0位置 y == 0
                		// str换下一个位置进行比对
				x++;
			} else { // y还可以通过最大前后缀长度往前移动
				y = next[y];
			}
		}
	    	// 1、 x越界，y没有越界，找不到，返回-1
	    	// 2、 x没越界，y越界，配出
	    	// 3、 x越界，y越界 ，配出，str的末尾，等于match
	    	// 只要y越界，就配出了，配出的位置等于str此时所在的位置x，减去y的长度。就是str存在匹配的字符串的开始位置
		return y == match.length ? x - y : -1;
	}

	// M   O(M)
	public static int[] getNextArray(char[] match) {
    		// 如果match只有一个字符，人为规定-1
		if (match.length == 1) {
			return new int[] { -1 };
		}
    		// match不止一个字符，人为规定0位置是-1，1位置是0
		int[] next = new int[match.length];
		next[0] = -1;
		next[1] = 0;
		int i = 2;
		// cn代表，cn位置的字符，是当前和i-1位置比较的字符
		int cn = 0;
		while (i < next.length) {
			if (match[i - 1] == match[cn]) { // 跳出来的时候
        		// next[i] = cn+1;
        		// i++;
        		// cn++;
        		// 等同于
				next[i++] = ++cn;
      			// 跳失败，如果cn>0说明可以继续跳
			} else if (cn > 0) {
				cn = next[cn];
      			// 跳失败，跳到开头仍然不等
			} else {
				next[i++] = 0;
			}
		}
		return next;
	}

	// for test
	public static String getRandomString(int possibilities, int size) {
		char[] ans = new char[(int) (Math.random() * size) + 1];
		for (int i = 0; i < ans.length; i++) {
			ans[i] = (char) ((int) (Math.random() * possibilities) + 'a');
		}
		return String.valueOf(ans);
	}

	public static void main(String[] args) {
		int possibilities = 5;
		int strSize = 20;
		int matchSize = 5;
		int testTimes = 5000000;
		System.out.println("test begin");
		for (int i = 0; i < testTimes; i++) {
			String str = getRandomString(possibilities, strSize);
			String match = getRandomString(possibilities, matchSize);
			if (getIndexOf(str, match) != str.indexOf(match)) {
				System.out.println("Oops!");
			}
		}
		System.out.println("test finish");
	}

}
```

## 1.2 KMP算法应用

### 题目1：旋转词

例如Str1="123456",对于Str1的旋转词，字符串本身也是其旋转词，Str1="123456"的旋转词为，"123456","234561","345612","456123","561234","612345"。给定Str1和Str2，那么判断这个两个字符串是否互为旋转词？是返回true，不是返回false


暴力解法思路：把str1的所有旋转词都列出来，看str2是否在这些旋转词中。挨个便利str1，循环数组的方式，和str2挨个比对。O(N*N)

KMP解法：str1拼接str1得到str',"123456123456"，我们看str2是否是str'的子串


### 题目2：子树问题

给定两颗二叉树头结点，node1和node2，判断node2为头结点的树，是不是node1的某个子树？



# 2 bfprt算法

`面试常见`

情形：在一个无序数组中，怎么求第k小的数。如果通过排序，那么排序的复杂度为O(n*logn)。问，如何O(N)复杂度解决这个问题？

思路1：我们利用快排的思想，对数组进行荷兰国旗partion过程，每一次partion可以得到随机数m小的区域，等于m的区域，大于m的区域。我们看我们m区域是否包含我们要找的第k小的树，如果没有根据比较，在m左区间或者m右区间继续partion，直到第k小的数在我们的的中间区域。

快排是左右区间都会再进行partion，而该问题只会命中大于区域或小于区域，时间复杂度得到优化。T(n)=T(n/2)+O(n)，时间复杂度为O(N)，由于m随机选，概率收敛为O(N)


思路2：bfprt算法，不使用概率求期望，复杂度仍然严格收敛到O(N)

## 2.1 bfprt算法分析

通过上文，利用荷兰国旗问题的思路为：

1、随机选一个数m

2、进行荷兰国旗，得到小于m区域，等于m区域，大于m区域

3、index命中到等于m区域，返回等于区域的左边界，否则比较，进入小于区域，或者大于区域，只会进入一个区域


bfprt算法，再此基础上唯一的区别是，第一步，如何选择m。快排的思想是随机选择一个

bfprt如何选择m？

- 1、对arr分组，5个一组，所以0到4为一组，5到9为一组，最后不够一组的当成最后一组
- 2、对各个小组进行排序。第一步和第二步进行下来，时间复杂度为O(N)
- 3、把每一小组排序后的中间位置的数拿出来。放入一个数组中m[]。前三步统称为bfprt方法
- 4、对m数组，取中位数，这个数就是我们需要的m

```math
T(N) = T(N/5) + T(?) + O(N)
```

建议画图分析：

T(?)在我们随机选取m的时候，是不确定的，但是在bfprt中，m的左侧范围最多有多少个数，等同于m右侧最少有几个数。

假设我们经过分组拿到的m数组有5个数，中位数是我们的m，在m[]数组中，大于m的有2个，小于m的有2个。对于整的数据规模而言，m[]的规模是n/5。大于m[]中位数的规模为m[]的一半,也就是整体数据规模的n/10。


由于m[]中的每个数都是从小组中选出来的，那么对于整体数据规模而言，大于m的数整体为3n/10（每个n/10规模的数回到自己的小组，大于等于的每小组有3个）


那么最少有3n/10的规模是大于等于m的，那么对于整体数据规模而言最多有7n/10的小于m的。同理最多有7n/10的数据是大于m的

可得：

```math
T(N) = T(N/5) + T(7n/10) + O(N)
```

数学证明，以上公式无法通过master来算复杂度，但是数学证明复杂度严格O(N)，证明略(算法导论第九章第三节)


> bfprt算法在算法上的地位非常高，它发现只要涉及到我们随便定义的一个常数分组，得到一个表达式，最后收敛到O(N)，那么就可以通过O(N)的复杂度测试


```Java
public class Code01_FindMinKth {

	public static class MaxHeapComparator implements Comparator<Integer> {

		@Override
		public int compare(Integer o1, Integer o2) {
			return o2 - o1;
		}

	}

	// 利用大根堆，时间复杂度O(N*logK)
	public static int minKth1(int[] arr, int k) {
		PriorityQueue<Integer> maxHeap = new PriorityQueue<>(new MaxHeapComparator());
		for (int i = 0; i < k; i++) {
			maxHeap.add(arr[i]);
		}
		for (int i = k; i < arr.length; i++) {
			if (arr[i] < maxHeap.peek()) {
				maxHeap.poll();
				maxHeap.add(arr[i]);
			}
		}
		return maxHeap.peek();
	}

	// 改写快排，时间复杂度O(N)
	public static int minKth2(int[] array, int k) {
		int[] arr = copyArray(array);
		return process2(arr, 0, arr.length - 1, k - 1);
	}

	public static int[] copyArray(int[] arr) {
		int[] ans = new int[arr.length];
		for (int i = 0; i != ans.length; i++) {
			ans[i] = arr[i];
		}
		return ans;
	}

	// arr 第k小的数: process2(arr, 0, N-1, k-1) 
	// arr[L..R]  范围上，如果排序的话(不是真的去排序)，找位于index的数
	// index [L..R]
  	// 通过荷兰国旗的优化，概率期望收敛于O(N)
	public static int process2(int[] arr, int L, int R, int index) {
		if (L == R) { // L == R ==INDEX
			return arr[L];
		}
		// 不止一个数  L +  [0, R -L]，随机选一个数
		int pivot = arr[L + (int) (Math.random() * (R - L + 1))];
		
    		// 返回以pivot为划分值的中间区域的左右边界
		// range[0] range[1]
		//  L   ..... R     pivot 
		//  0         1000     70...800
		int[] range = partition(arr, L, R, pivot);
    		// 如果我们第k小的树正好在这个范围内，返回区域的左边界
		if (index >= range[0] && index <= range[1]) {
			return arr[index];
      		// index比该区域的左边界小，递归左区间
		} else if (index < range[0]) {
			return process2(arr, L, range[0] - 1, index);
      		// index比该区域的右边界大，递归右区间
		} else {
			return process2(arr, range[1] + 1, R, index);
		}
	}

	public static int[] partition(int[] arr, int L, int R, int pivot) {
		int less = L - 1;
		int more = R + 1;
		int cur = L;
		while (cur < more) {
			if (arr[cur] < pivot) {
				swap(arr, ++less, cur++);
			} else if (arr[cur] > pivot) {
				swap(arr, cur, --more);
			} else {
				cur++;
			}
		}
		return new int[] { less + 1, more - 1 };
	}

	public static void swap(int[] arr, int i1, int i2) {
		int tmp = arr[i1];
		arr[i1] = arr[i2];
		arr[i2] = tmp;
	}

	// 利用bfprt算法，时间复杂度O(N)
	public static int minKth3(int[] array, int k) {
		int[] arr = copyArray(array);
		return bfprt(arr, 0, arr.length - 1, k - 1);
	}

	// arr[L..R]  如果排序的话，位于index位置的数，是什么，返回
	public static int bfprt(int[] arr, int L, int R, int index) {
		if (L == R) {
			return arr[L];
		}
    	// 通过bfprt分组，最终选出m。不同于随机选择m作为划分值
		int pivot = medianOfMedians(arr, L, R);
		int[] range = partition(arr, L, R, pivot);
		if (index >= range[0] && index <= range[1]) {
			return arr[index];
		} else if (index < range[0]) {
			return bfprt(arr, L, range[0] - 1, index);
		} else {
			return bfprt(arr, range[1] + 1, R, index);
		}
	}

	// arr[L...R]  五个数一组
	// 每个小组内部排序
	// 每个小组中位数拿出来，组成marr
	// marr中的中位数，返回
	public static int medianOfMedians(int[] arr, int L, int R) {
		int size = R - L + 1;
    		// 是否需要补最后一组，例如13，那么需要补最后一组，最后一组为3个数
		int offset = size % 5 == 0 ? 0 : 1;
		int[] mArr = new int[size / 5 + offset];
		for (int team = 0; team < mArr.length; team++) {
			int teamFirst = L + team * 5;
			// L ... L + 4
			// L +5 ... L +9
			// L +10....L+14
			mArr[team] = getMedian(arr, teamFirst, Math.min(R, teamFirst + 4));
		}
		// marr中，找到中位数，原问题是arr拿第k小的数，这里是中位数数组拿到中间位置的数（第mArr.length / 2小的数），相同的问题
   		// 返回值就是我们需要的划分值m
		// marr(0, marr.len - 1,  mArr.length / 2 )
		return bfprt(mArr, 0, mArr.length - 1, mArr.length / 2);
	}

	public static int getMedian(int[] arr, int L, int R) {
		insertionSort(arr, L, R);
		return arr[(L + R) / 2];
	}

  	// 由于确定是5个数排序，我们选择一个常数项最低的排序-插入排序
	public static void insertionSort(int[] arr, int L, int R) {
		for (int i = L + 1; i <= R; i++) {
			for (int j = i - 1; j >= L && arr[j] > arr[j + 1]; j--) {
				swap(arr, j, j + 1);
			}
		}
	}

	// for test
	public static int[] generateRandomArray(int maxSize, int maxValue) {
		int[] arr = new int[(int) (Math.random() * maxSize) + 1];
		for (int i = 0; i < arr.length; i++) {
			arr[i] = (int) (Math.random() * (maxValue + 1));
		}
		return arr;
	}

	public static void main(String[] args) {
		int testTime = 1000000;
		int maxSize = 100;
		int maxValue = 100;
		System.out.println("test begin");
		for (int i = 0; i < testTime; i++) {
			int[] arr = generateRandomArray(maxSize, maxValue);
			int k = (int) (Math.random() * arr.length) + 1;
			int ans1 = minKth1(arr, k);
			int ans2 = minKth2(arr, k);
			int ans3 = minKth3(arr, k);
			if (ans1 != ans2 || ans2 != ans3) {
				System.out.println("Oops!");
			}
		}
		System.out.println("test finish");
	}

}
```

## 2.2 bfprt算法应用

题目：求一个数组中，拿出所有比第k小的数还小的数

可以通过bfprt拿到第k小的数，再对原数组遍历一遍，小于该数的拿出来，不足k位的，补上第k小的数

> 对于这类问题，笔试的时候最好选择随机m，进行partion。而不是选择bfprt。bfprt的常数项高。面试的时候可以选择bfprt算法


