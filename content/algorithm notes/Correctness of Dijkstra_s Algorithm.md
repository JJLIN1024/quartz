---
title: Correctness of Dijkstra's Algorithm
date: 2022-06-17
author:
  - Jimmy Lin
tags:
  - Dijkstra
  - shortest_path
---


## 證明

- Invariant
	- 當一個 node 在 Dijkstra's Algorithm 中被 extract-min 從 heap 中抓出來時，它和 source node ($s$) 的距離就是從 $s$ 到此 node 的最短距離。

## Proof by Induction (Fig. 1)

首先圈圈中只有裝 node $s$ 時，invariant 成立，距離和最短距離都是 $0$。假設當圈圈中有 $k- 1$ 個 node 時成立。假設 node $v$ 為第 $k$ 個被 extract-min 的 node，而經過 relaxation，$v$ 的 parent 會是 $u$，代表 $d(s, u) + w(u, v) \le d(s, a) + w(a, v)$。至此，$d(s, v)$ 已確定，長度等於 $s$ 到 $v$ 的最短路徑。假設還有另一條到 $v$ 的最短路徑通過 $b$ 到 $v$，我們可以證明這條路徑不是最短路徑，因為根據三角不等式：$w(a,v) \lt w(a, b) + w(b, v)$，結合上面的結論 $d(s, u) + w(u, v) \le d(s, a) + w(a, v)$，可證明出 
$$ d(s, u) + w(u, v) \le d(s, a) + w(a, v) \lt d(s, a) + w(a, b) + w(b, v)$$ 
因此通過 $b$ 的這條路徑並非最短路徑，所以原本通過 $u$ 的那條路確實是最短路徑。

## Proof by contradiction (Fig. 2)

Invariant 不變，假設 node $b$ 為現在被 extract-min 抓出來的 node，但是反證法假設還有另一個 node $v$ 在到 $b$ 的 shortest path 上。根據 invariant，$d(s, b) \le d(s, v)$，而 
$$d(s, v) = d(s, u) + w(u, v) \lt d(s, u) + w(u, v) + w(v,b) = d(s, b)$$
故 $d(s, b) \lt d(s, b)$，造成矛盾，故證明出不會有其他 node 在通往 $b$ 的 shortest path 上，所以經過 relax 後確實會得到通往 $b$ 的 shortest path。

## Note
Dijkstra's Algorithm 只適用於 edge weight 為正的 graph 的理由就在於上面兩個證明的不等式，這兩個不等式都會需要 edge weight 為正才會成立。

## Figures
### Fig. 1
![](https://i.imgur.com/wiFmeYL.png)
### Fig.2
![](https://i.imgur.com/dYpvlc1.png)

## Reference
- 台大電機 Algorithm Class 江蕙如 投影片