---
title: Cheapest Flights Within K Stops
date: 2023-04-23
lastmod: 2023-04-23
author:
  - Jimmy Lin
tags:
  - shortest_path
  - Dijkstra
  - bellman_ford
draft: false
---

## Description

There are `n` cities connected by some number of flights. You are given an array `flights` where `flights[i] = [fromi, toi, pricei]` indicates that there is a flight from city `fromi` to city `toi` with cost `pricei`.

You are also given three integers `src`, `dst`, and `k`, return _**the cheapest price** from_ `src` _to_ `dst` _with at most_ `k` _stops._ If there is no such route, return `-1`.

**Example 1:**

![](https://assets.leetcode.com/uploads/2022/03/18/cheapest-flights-within-k-stops-3drawio.png)

**Input:** n = 4, flights = \[\[0,1,100\],\[1,2,100\],\[2,0,100\],\[1,3,600\],\[2,3,200\]\], src = 0, dst = 3, k = 1
**Output:** 700
**Explanation:**
The graph is shown above.
The optimal path with at most 1 stop from city 0 to 3 is marked in red and has cost 100 + 600 = 700.
Note that the path through cities \[0,1,2,3\] is cheaper but is invalid because it uses 2 stops.

**Example 2:**

![](https://assets.leetcode.com/uploads/2022/03/18/cheapest-flights-within-k-stops-1drawio.png)

**Input:** n = 3, flights = \[\[0,1,100\],\[1,2,100\],\[0,2,500\]\], src = 0, dst = 2, k = 1
**Output:** 200
**Explanation:**
The graph is shown above.
The optimal path with at most 1 stop from city 0 to 2 is marked in red and has cost 100 + 100 = 200.

**Example 3:**

![](https://assets.leetcode.com/uploads/2022/03/18/cheapest-flights-within-k-stops-2drawio.png)

**Input:** n = 3, flights = \[\[0,1,100\],\[1,2,100\],\[0,2,500\]\], src = 0, dst = 2, k = 0
**Output:** 500
**Explanation:**
The graph is shown above.
The optimal path with no stops from city 0 to 2 is marked in red and has cost 500.

**Constraints:**

*   `1 <= n <= 100`
*   `0 <= flights.length <= (n * (n - 1) / 2)`
*   `flights[i].length == 3`
*   `0 <= fromi, toi < n`
*   `fromi != toi`
*   `1 <= pricei <= 104`
*   There will not be any multiple flights between two cities.
*   `0 <= src, dst, k < n`
*   `src != dst`

## Code 
### Bellman Ford

$O(VE)$.

see [[Bellman-Ford Algorithm]].

```python
class Solution:
    def findCheapestPrice(self, n: int, flights: List[List[int]], src: int, dst: int, k: int) -> int:
        # Bellman-Ford algorithm
        # at most k stops -> relax edges for k + 1 times
        dist = [float('inf')] * n
        dist[src] = 0

        for i in range(k + 1):
            temp = dist[:]
            for u, v, w in flights:
                temp[v] = min(temp[v], dist[u] + w)
            dist = temp
        
        return -1 if dist[dst] == float('inf') else dist[dst]
```

```cpp
class Solution {
public:
    int findCheapestPrice(int n, vector<vector<int>>& flights, int src, int dst, int k) {
        vector<int> dp(n, INT_MAX);
        dp[src] = 0;
        for(int i = 0; i < k + 1; i++) {
            vector<int> temp(dp);
            for(auto flight: flights) {
                int u = flight[0];
                int v = flight[1];
                int w = flight[2];
                if(dp[u] != INT_MAX)
                    temp[v] = min(temp[v], dp[u] + w);
            }
            dp = temp;
        }

        return dp[dst] == INT_MAX ? -1 : dp[dst];
    }
};
```

### Dijkstra (TLE now)

$O((V+E) \log V)$

```cpp
typedef tuple<int, int, int> tp;
class Solution {
public:
    int findCheapestPrice(int n, vector<vector<int>>& flights, int src, int dst, int k) {
        vector<vector<pair<int, int>>> adj(n); 
        priority_queue<tp, vector<tp>, greater<tp>> pq;
        for(auto flight: flights) {
            adj[flight[0]].emplace_back(flight[1], flight[2]);
        }

        pq.emplace(src, 0, k + 1);
        while(!pq.empty()) {
            auto [node, cost, stops] = pq.top();
            pq.pop();
            if(node == dst) return cost;
            if(!stops) continue;
            for(auto neighbor_info: adj[node]) {
                auto [neighbor, w] = neighbor_info;
                pq.push(make_tuple(neighbor, cost + w, stops - 1));
            }
        }
        return -1;
    }
};
```





## Source
- [Cheapest Flights Within K Stops - LeetCode](https://leetcode.com/problems/cheapest-flights-within-k-stops/)