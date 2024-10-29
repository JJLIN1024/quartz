---
title: Count Square Submatrices with All Ones
date: 2024-10-27
lastmod: 2024-10-27
author:
  - Jimmy Lin
tags:
  - DP
draft: false
---

## Description

Given a `m * n` matrix of ones and zeros, return how many **square** submatrices have all ones.

**Example 1:**

**Input:** matrix =
\[
  \[0,1,1,1\],
  \[1,1,1,1\],
  \[0,1,1,1\]
\]
**Output:** 15
**Explanation:** 
There are **10** squares of side 1.
There are **4** squares of side 2.
There is  **1** square of side 3.
Total number of squares = 10 + 4 + 1 = **15**.

**Example 2:**

**Input:** matrix = 
\[
  \[1,0,1\],
  \[1,1,0\],
  \[1,1,0\]
\]
**Output:** 7
**Explanation:** 
There are **6** squares of side 1.  
There is **1** square of side 2. 
Total number of squares = 6 + 1 = **7**.

**Constraints:**

*   `1 <= arr.length <= 300`
*   `1 <= arr[0].length <= 300`
*   `0 <= arr[i][j] <= 1`

## Code 

Thinking Process:
1. 當 iterate 到 `matrix[i][j]` 時，如果他本身是 1，就可以組成一個 1 x 1 的正方格，而要判斷它是否可以組成一個 size > 1 都是 1 的正方格，就要看他左邊右邊左斜上方的三個格子
2. `dp[i][j] = min(dp[i - 1][j], min(dp[i][j - 1], dp[i - 1][j - 1])) + 1;`
3. 當 `dp[i][j] = n`，代表這格可以組成 1 x 1、2 x 2、一路到 n x n 都是 1 的正方形，因此 `res += n`

Time Complexity: $O(mn)$, Space Complexity: $O(mn)$

```cpp
class Solution {
public:
    int countSquares(vector<vector<int>>& matrix) {
        int m = matrix.size(), n = matrix[0].size();
        vector<vector<int>> dp(m, vector<int>(n, 0));
        int res = 0;
        // base case
        for(int i = 0; i < m; i++) {
            dp[i][0] = matrix[i][0];
            res += dp[i][0];
        }
        for(int j = 0; j < n; j++) {
            dp[0][j] = matrix[0][j];
            if (j != 0)
                res += dp[0][j];
        }
        // dp: dp[i][j] = min(dp[i - 1][j], min(dp[i][j - 1], dp[i - 1][j - 1])) + 1;
        for(int i = 1; i < m; i++) {
            for(int j = 1; j < n; j++) {
                if (matrix[i][j] == 1) {
                    dp[i][j] = min(dp[i - 1][j], min(dp[i][j - 1], dp[i - 1][j - 1])) + 1;
                }
                res += dp[i][j];
            }
        }
        return res;
    }
};
```

## Source
- [Count Square Submatrices with All Ones - LeetCode](https://leetcode.com/problems/count-square-submatrices-with-all-ones/description/?envType=daily-question&envId=2024-10-27)