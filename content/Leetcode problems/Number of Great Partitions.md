---
title: Number of Great Partitions
date: 2023-09-25
lastmod: 2023-09-25
author:
  - Jimmy Lin
tags:
  - knapsack
  - DP
draft: false
sr-due: 2024-03-30
sr-interval: 4
sr-ease:
---

## Description

You are given an array `nums` consisting of **positive** integers and an integer `k`.

**Partition** the array into two ordered **groups** such that each element is in exactly **one** group. A partition is called great if the **sum** of elements of each group is greater than or equal to `k`.

Return _the number of **distinct** great partitions_. Since the answer may be too large, return it **modulo** `109 + 7`.

Two partitions are considered distinct if some element `nums[i]` is in different groups in the two partitions.

**Example 1:**

**Input:** nums = \[1,2,3,4\], k = 4
**Output:** 6
**Explanation:** The great partitions are: (\[1,2,3\], \[4\]), (\[1,3\], \[2,4\]), (\[1,4\], \[2,3\]), (\[2,3\], \[1,4\]), (\[2,4\], \[1,3\]) and (\[4\], \[1,2,3\]).

**Example 2:**

**Input:** nums = \[3,3,3\], k = 4
**Output:** 0
**Explanation:** There are no great partitions for this array.

**Example 3:**

**Input:** nums = \[6,6\], k = 2
**Output:** 2
**Explanation:** We can either put nums\[0\] in the first partition or in the second partition.
The great partitions will be (\[6\], \[6\]) and (\[6\], \[6\]).

**Constraints:**

*   `1 <= nums.length, k <= 1000`
*   `1 <= nums[i] <= 109`

## Code 

### Naive
按照題意，但是這樣 solution space 太大，k 可以很大。

```cpp
class Solution {
public:
    int mod = 1e9 + 7;
    int k_;
    int countPartitions(vector<int>& nums, int k) {
        k_ = k;
        return partition(nums, 0, 0, 0);
    }

    int partition(vector<int>& nums, int i, long long group1, long long group2) {
        if(i == nums.size()) {
            return group1 >= k_ && group2 >= k_;
        }
        int toG1 = partition(nums, i + 1, group1 + nums[i], group2) % mod;
        int toG2 = partition(nums, i + 1, group1, group2 + nums[i]) % mod;
        return (toG1 + toG2) % mod;
    }
};
```


所以我們要反過來想，既然找比 k 大的不行，就找比ｋ小的。

### Top Down DP

和 [[Find the Sum of the Power of All Subsequences]] 類似，也是 knapsack 類型的題目（take or not take）。

```      
Suppose we partitioned our array into subsets s1 and s2 :-
    
Final subsets -> s1 and s2

condition to be satisfied :  s1 >= k and s2 >= k
            
s1+s2 >= 2*k
            
s1+s2 = sum of array = s
            
so sum of array i.e s must be >= 2*k
            
Converse of the above condition : s1>=k and s2>=k
            
is :
            
(1) s1<k and s2<k
(2) s1<k and s2>=k
(3) s1>=k and s2<k
            
if(sum < 2 * k) {
	return 0;
}

will eliminate (1), and (2) (3) 是對稱的，所以只需要計算 (2)（或 (3))。


Final ans= 2^n - count in (2) - count in (3)
         = 2^n - 2*(count in (2))   [as count in (2) = count in (3)]
```

```cpp
class Solution {
public:
    int mod = 1e9 + 7;
    int k_;
    int countPartitions(vector<int>& nums, int k) {
        k_ = k;
        long long validWays = 1;
        long long sum = 0;
        for(auto& n: nums) {
            sum += n;
            validWays = (validWays * 2LL) % mod;
        }
        if(sum < 2 * k) {
            return 0;
        }
        int n = nums.size();
        vector<vector<int>> dp(n + 1, vector<int>(k + 1, -1));

        int invalidWays = findLessThanK(nums, 0, 0, dp) % mod;
        invalidWays = invalidWays * 2LL % mod;
        long long res = (validWays - invalidWays + mod) % mod;
        return res;
    }

    int findLessThanK(vector<int>& nums, int i, long long sum, vector<vector<int>>& dp) {
        if(sum >= k_)
            return 0;
        if(i == nums.size()) 
            return sum < k_;
        if(dp[i][sum] != -1)
            return dp[i][sum];
        
        int take = findLessThanK(nums, i + 1, sum + nums[i], dp) % mod;
        int notTake = findLessThanK(nums, i + 1, sum, dp) % mod;
        return dp[i][sum] = (take + notTake) % mod;
    }
};
```

### DP - 2D knapsack
Time Complexity: $O(nk)$, Space Complexity: $O(nk)$

注意 `total = (total % mod + mod) % mod;` ，因為有取 mod，所以在 `total -= 2*ways;` 之後值有可能是負的。就像是在計算 `-3 mod 7` 時，要先做 `-3 + 7 = 4`，再做 `4 % 7 = 4`。（ `%` 代表除法取餘數）

和 [[Partition Equal Subset Sum|Partition Equal Subset Sum]] 一樣屬於 knapsack problem 系列。只是在這題中我們要找的是 `sum < k` 的組合。

```markdown
we are asked to find the number of arrangement that both have size > k.
We can find out the number of cases that have size <= k, then that a more regular Knapsack problem.
```


```cpp
#define ll long long
class Solution {
public:
    int countPartitions(vector<int>& nums, int k) {

        ll sum = 0;
        for(int &n: nums) {
            sum += n;
        }
        if(sum < 2 * 1ll * k || nums.size() <= 1) return 0;

        ll n = nums.size();
        ll mod = 1e9 + 7;
        ll total = mpow(2, n, mod);    

        vector<vector<ll>> dp(n + 1, vector<ll>(k, 0));
        
        // base case
        for(int i = 0; i < n + 1; i++) {
            dp[i][0] = 1;
        }

        for(int i = 1; i < n + 1; i++) {
            for(int j = 1; j < k; j++) {
                if(j >= nums[i - 1]) {
                    dp[i][j] = (dp[i - 1][j] % mod) + (dp[i - 1][j - nums[i - 1]] % mod);
                } else {
                    dp[i][j] = dp[i - 1][j];
                }
                
                dp[i][j] = dp[i][j] % mod;
            }
        }

        ll ways = 0;
        for(int i = 0; i < k; i++) {
            ways = (ways + dp[n][i]) % mod;
        }

        ways %= mod;

        total -= 2*ways;
        total = (total % mod + mod) % mod;

        return total;

    }

    ll mpow(ll a, ll b, ll m){
        if (b == 0)
            return 1;
        ll x = mpow(a, b / 2, m);
        x = (x * x) % m;
        if (b % 2)
        {
            x = (x * a) % m;
        }
        return x;
    }
};
```

## Source
- [Number of Great Partitions - LeetCode](https://leetcode.com/problems/number-of-great-partitions/description/)