---
layout: post
title: 动态规划
categories: 算法
description: 动态规划
keywords: 动态规划
---

动态规划

## 基本思想

若要解一个给定问题，我们需要解其不同部分（即子问题），再合并子问题的解以得出原问题的解。 通常许多子问题非常相似，为此动态规划法试图仅仅解决每个子问题一次，从而减少计算量： 一旦某个给定子问题的解已经算出，则将其记忆化存储，以便下次需要同一个子问题解之时直接查表。 这种做法在重复子问题的数目关于输入的规模呈指数增长时特别有用。

## 分治与动态规划

共同点：二者都要求原问题具有最优子结构性质,都是将原问题分而治之,分解成若干个规模较小(小到很容易解决的程序)的子问题.然后将子问题的解合并,形成原问题的解.

不同点：分治法将分解后的子问题看成相互独立的，通过用递归来做。动态规划将分解后的子问题理解为相互间有联系,有重叠部分，需要记忆，通常用迭代来做。

## 问题特征

最优子结构：当问题的最优解包含了其子问题的最优解时，称该问题具有最优子结构性质。

重叠子问题：在用递归算法自顶向下解问题时，每次产生的子问题并不总是新问题，有些子问题被反复计算多次。动态规划算法正是利用了这种子问题的重叠性质，对每一个子问题只解一次，而后将其解保存在一个表格中，在以后尽可能多地利用这些子问题的解。

## 解题步骤

1.找出最优解的性质，刻画其结构特征和最优子结构特征，将原问题分解成若干个子问题；

把原问题分解为若干个子问题，子问题和原问题形式相同或类似，只不过规模变小了。子问题都解决，原问题即解决，子问题的解一旦求出就会被保存，所以每个子问题只需求解一次。

2.递归地定义最优值，刻画原问题解与子问题解间的关系，确定状态；

 在用动态规划解题时，我们往往将和子问题相关的各个变量的一组取值，称之为一个“状态”。一个“状态”对应于一个或多个子问题， 所谓某个“状态”下的“值”，就是这个“状 态”所对应的子问题的解。所有“状态”的集合，构成问题的“状态空间”。“状态空间”的大小，与用动态规划解决问题的时间复杂度直接相关。

3.以自底向上的方式计算出各个子问题、原问题的最优值，并避免子问题的重复计算；

定义出什么是“状态”，以及在该“状态”下的“值”后，就要找出不同的状态之间如何迁移――即如何从一个或多个“值”已知的 “状态”，求出另一个“状态”的“值”(递推型)。

4.根据计算最优值时得到的信息，构造最优解，确定转移方程;

状态的迁移可以用递推公式表示，此递推公式也可被称作“状态转移方程”。

## 案例

### 1. 322. Coin Change

`You are given coins of different denominations and a total amount of money amount. Write a function to compute the fewest number of coins that you need to make up that amount. If that amount of money cannot be made up by any combination of the coins, return -1.`

自底向上动态规划：

For the iterative solution, we think in bottom-up manner. Before calculating F(i)F(i), we have to compute all minimum counts for amounts up to ii. On each iteration ii of the algorithm F(i)F(i) is computed as min <sub>j=0…n−1</sub>  F(i−c <sub>j </sub>)+1 。

比如：
F(3) 
=min{F(3−c<sub>1</sub> ),F(3−c<sub>2</sub>),F(3−<sub>c3</sub>)}+1
=min{F(3−1),F(3−2),F(3−3)}+1
=min{F(2),F(1),F(0)}+1
=min{1,1,0}+1
=1

代码：

```java 
public class Solution {
    public int coinChange(int[] coins, int amount) {
        int max = amount + 1;             
        int[] dp = new int[amount + 1];  
        Arrays.fill(dp, max);  
        dp[0] = 0;   
        for (int i = 1; i <= amount; i++) {
            for (int j = 0; j < coins.length; j++) {
                if (coins[j] <= i) {
                    dp[i] = Math.min(dp[i], dp[i - coins[j]] + 1);
                }
            }
        }
        return dp[amount] > amount ? -1 : dp[amount];
    }
}
```

### 2. 300. Longest Increasing Subsequence

`Given an unsorted array of integers, find the length of longest increasing subsequence.`

有一个序列有 N 个数，A[1],A[2],…,A[N]. 求出其最长递增子序列的长度。

我们将这个问题分解，一个序列有 i 个数 A[1],A[2],…,A[i], 其中 i<N.

那么这个问题就变成了一个子问题，然后我们定义 d(i), 表示前 i 个数中以 A[i] 结尾的最长递增子序列的长度。

当 i→N 时，我们把 d(1) 到 d(N) 都计算出来，我们要找的答案就是这里面最大的一个。

状态找到了，下一步来找状态转移方程。举个例子，我们要求的这 N 个数的序列是：

`5, 3, 4, 8, 6, 7`

根据上面的状态，我们得到

d(1)=1 (5) // 5前面没有比它小的 d(1)=1

d(2)=1 (3) // 3前面没有比它小的 d(2)=1

d(3)=2 (3 4) // 4前面有1个比它小的，所以 d(3)=d(2)+1=2

d(4)=3 (3 4 8) // 8前面比他小的有3个数, 所以 d(4)=max{d(1),d(2),d(3)}+1=3

d(5)=3 (3 4 6) // 6前面比他小的有3个数，所以 d(5)=max{d(1),d(2),d(3)}+1=3

d(6)=4 (3 4 6 7) // 7前面比他小的有4个数，所以 d(6)=max{d(1),d(2),d(4),d(5)}+1=4

根据 d(i) 和 d(i−1) 我们可以得到

`d(i)=max{1,d(j)+1}(j<i,A[j]<A[i])`

解释一下，要找到 d(i), 我们要先找到所有 A[j] 小于 A[i] 的数，分别是 1→j，并且， 然后找到他们中最大的那一个 d(j), 加上1, 就得到了我们想要的序列 1→i 的LIS长度 d(i) 了。

​代码：

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        
        if (nums == null || nums.length == 0){
			return 0;
		}
        int[] dp = new int[nums.length];
		int maxLength = 1;
		
		for (int i = 0; i < nums.length; i++){
			dp[i] = 1;
			for (int j = 0; j < i; j++){
				if (nums[j] < nums[i] && dp[j] + 1 > dp[i]){
					dp[i] = dp[j] + 1;
				}
			}
			if (dp[i] > maxLength){
				maxLength = dp[i];
			}
		}
		return maxLength;
    }
}
```

> 声明：本站采用开放的[知识共享署名-非商业性使用-相同方式共享 许可协议](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)进行许可。
