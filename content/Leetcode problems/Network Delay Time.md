---
title: Network Delay Time
date: 2023-04-24
lastmod: 2025-03-23
author:
  - Jimmy Lin
tags:
  - shortest_path
  - Dijkstra
  - bellman_ford
draft: false
---

## Description

You are given a network of `n` nodes, labeled from `1` to `n`. You are also given `times`, a list of travel times as directed edges `times[i] = (ui, vi, wi)`, where `ui` is the source node, `vi` is the target node, and `wi` is the time it takes for a signal to travel from source to target.

We will send a signal from a given node `k`. Return _the **minimum** time it takes for all the_ `n` _nodes to receive the signal_. If it is impossible for all the `n` nodes to receive the signal, return `-1`.

**Example 1:**

![](https://assets.leetcode.com/uploads/2019/05/23/931_example_1.png)

**Input:** times = \[\[2,1,1\],\[2,3,1\],\[3,4,1\]\], n = 4, k = 2
**Output:** 2

**Example 2:**

**Input:** times = \[\[1,2,1\]\], n = 2, k = 1
**Output:** 1

**Example 3:**

**Input:** times = \[\[1,2,1\]\], n = 2, k = 2
**Output:** -1

**Constraints:**

*   `1 <= k <= n <= 100`
*   `1 <= times.length <= 6000`
*   `times[i].length == 3`
*   `1 <= ui, vi <= n`
*   `ui != vi`
*   `0 <= wi <= 100`
*   All the pairs `(ui, vi)` are **unique**. (i.e., no multiple edges.)

## Code 

same as [[Cheapest Flights Within K Stops]], 都是 shortest path 的題目。

### Dijkstra

$O((V+E)(\log V))$

```python
class Solution:
    def networkDelayTime(self, times: List[List[int]], n: int, k: int) -> int:
        # Build adjacency list
        adj_list = [[] for _ in range(n)]
        for u, v, t in times:
            adj_list[u - 1].append([v - 1, t])

        # Dijkstra's Algorithm
        time = [float('inf') for _ in range(n)]
        time[k - 1] = 0
        min_heap = [[0, k - 1]] # time, node

        while min_heap:
            cur_time, node = heappop(min_heap)
            for neighbor, t in adj_list[node]:
                if cur_time + t < time[neighbor]:
                    time[neighbor] = cur_time + t
                    heappush(min_heap, [time[neighbor], neighbor])

        min_time = max(time)
        return min_time if min_time != float('inf') else -1
```

```cpp
typedef pair<int, int> pa;
class Solution {
public:
    int networkDelayTime(vector<vector<int>>& times, int n, int k) {
        vector<vector<pair<int, int>>> adj(n + 1); 
        priority_queue<pa, vector<pa>, greater<pa>> pq;
        for(auto time: times) {
            adj[time[0]].push_back({time[1], time[2]});
        }
        vector<int> dist(n + 1, INT_MAX);
        dist[k] = 0;
        pq.push({0, k});
        while(!pq.empty()) {
            auto [cost, u] = pq.top(); pq.pop();
            for(auto [v, w]: adj[u]) {
                if(dist[v] > dist[u] + w) {
                    dist[v] = dist[u] + w;
                    pq.push({dist[v], v});
                }
            }
        }   

        int ans = *max_element(dist.begin() + 1, dist.end());
        return ans == INT_MAX ? -1 : ans;
    }
};
```

### Bellman Ford

$O(VE)$

```python
class Solution:
    def networkDelayTime(self, times: List[List[int]], n: int, k: int) -> int:
        # Bellman-Ford
        # Total of n nodes, so at most n - 1 edges -> relax n - 1 times
        time = [float('inf')] * n # 0-based
        time[k - 1] = 0

        for i in range(n - 1):
            temp = time[:]
            for u, v, t in times:
                if time[u - 1] == float('inf'):
                    continue
                temp[v - 1] = min(temp[v - 1], time[u - 1] + t)
            time = temp
        
        min_time = 0
        for t in time:
            if t == float('inf'):
                return -1
            min_time = max(min_time, t)
        return min_time
```

```cpp
class Solution {
public:
    int networkDelayTime(vector<vector<int>>& times, int n, int k) {
        vector<int> dp(n + 1, INT_MAX);
        dp[k] = 0;
        for(int i = 0; i < n; i++) {
            vector<int> temp(dp);
            for(auto time: times) {
                if(dp[time[0]] != INT_MAX)
                    temp[time[1]] = min(temp[time[1]], dp[time[0]] + time[2]);
            }
            dp = temp;
        }

        int ans = *max_element(dp.begin() + 1, dp.end());
        return ans == INT_MAX ? -1 : ans;

    }
};
```



## Source
- [Network Delay Time - LeetCode](https://leetcode.com/problems/network-delay-time/)