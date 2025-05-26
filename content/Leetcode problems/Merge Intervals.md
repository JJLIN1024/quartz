---
title: Merge Intervals
date: 2023-03-25
lastmod: 2023-03-25
author:
  - Jimmy Lin
tags:
  - array
  - review
  - greedy
  - interval
draft: false
---

## Description

Given an array of `intervals` where `intervals[i] = [starti, endi]`, merge all overlapping intervals, and return _an array of the non-overlapping intervals that cover all the intervals in the input_.

**Example 1:**

**Input:** intervals = \[\[1,3\],\[2,6\],\[8,10\],\[15,18\]\]
**Output:** \[\[1,6\],\[8,10\],\[15,18\]\]
**Explanation:** Since intervals \[1,3\] and \[2,6\] overlap, merge them into \[1,6\].

**Example 2:**

**Input:** intervals = \[\[1,4\],\[4,5\]\]
**Output:** \[\[1,5\]\]
**Explanation:** Intervals \[1,4\] and \[4,5\] are considered overlapping.

**Constraints:**

*   `1 <= intervals.length <= 104`
*   `intervals[i].length == 2`
*   `0 <= starti <= endi <= 104`

## Code 

類似 [[Insert Interval]]。關鍵都在於看後者的開頭是否小於前者的尾巴，再去做處理。

```cpp
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        if(intervals.size() <= 1) return intervals;
        vector<vector<int>> answer;
        sort(begin(intervals), end(intervals));

        answer.push_back(intervals[0]);
        for(int i = 1; i < intervals.size(); i++){
            if(answer.back()[1] < intervals[i][0]) answer.push_back(intervals[i]);
            else answer.back()[1] = max(intervals[i][1], answer.back()[1]);
        }
        return answer;
    }
};
```


```cpp
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        int i = 1, n = intervals.size();
        sort(intervals.begin(), intervals.end());

        vector<vector<int>> res;
        res.push_back(intervals[0]);

        while(i < n) {
            while(i < n && res.back()[1] < intervals[i][0]) {
                res.push_back(intervals[i]);
                i++;
            }

            while(i < n && res.back()[1] >= intervals[i][0]) {
                res.back()[0] = min(res.back()[0], intervals[i][0]);
                res.back()[1] = max(res.back()[1], intervals[i][1]);
                i++;
            }
        }
        return res;
    }
};
```
## Source
- [Merge Intervals - LeetCode](https://leetcode.com/problems/merge-intervals/description/)