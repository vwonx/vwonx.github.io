---
title: 0-1 BFS
subtitle: BFS变种，可解决边权为0或1时候的最短路搜索问题，常见于迷宫问题
catalog: true
tags:
  - 算法-图论
  - 算法-搜索
categories:
  - 算法
abbrlink: 9511e3af
date: 2022-12-02 23:27:09
---
# 0-1 BFS

0-1 BFS 是 BFS 的一种变体，可以解决边权仅为 0 或 1 时候的最短路搜索问题。实际上，当边权仅有 0 和另一固定值 $n$ 的时候，同样适用该算法。因为当边权 0 和 $n$ 同除以 $n$ 的时候，就自然转化为了 0 和 1。

# 正确性验证
考虑有边权仅为 1 的图，从起点出发，遍历完图后，会生成一颗最短路径树。
![](https://raw.githubusercontent.com/vwonx/blog-imgs/master/01-BFS/1.png)

对于该树来说，每一层的兄弟节点的路径边权和是相等的，如点 $b$ 和点 $c$，最短路的边权和均为1。而 BFS 也具有层次遍历的特点，即当开始遍历深度为 $k$ 的节点的时候，它的兄弟节点一定在队列中，且位于队列前端。只有当遍历完深度为 $k$ 的节点后才会遍历深度为 $k+1$ 的节点，并且 $k+1$ 深度节点的边权和一定大于深度为 $k$ 的节点的边权和。**即队列中节点的边权和一定是单调不减的，由此保证了 BFS 算法的正确性。**

![](https://raw.githubusercontent.com/vwonx/blog-imgs/master/01-BFS/2.png)

现在再考虑加入边权0的情况，假设点 $b$ 到点 $d$ 有两种边权，一种为0，一种为1。那么根据先前的分析，边权为0的时候，其实也就意味着点 $b$ 和点 $d$ 可以被当作是同一层的节点。而边权为1的时候，点 $d$ 的深度就比点 $b$ 大1。

![](https://raw.githubusercontent.com/vwonx/blog-imgs/master/01-BFS/3.png)

当我们在遍历到点 $b$，去做下一步的扩展的时候。让边权为 0 的点插入队列头部（下一次迭代等同于在遍历兄弟节点），而让边权为1的点插入到队列尾部（当前层的兄弟节点一定已经在队列中，所以下一层的节点要么没有，要么就一定在队列尾部）。这样扩展后的队列依旧满足单调不减的特效，所以可以继续保证 BFS 算法的正确性。

# 代码

[Leetcode2290](https://leetcode.cn/problems/minimum-obstacle-removal-to-reach-corner)

```cpp
class Solution {
public:
    using pii = pair<int,int>;
    int minimumObstacles(vector<vector<int>>& grid) {
        int rows = grid.size();
        int cols = grid[0].size();

        int dirX[4] = {0, 0, 1, -1};
        int dirY[4] = {1, -1, 0, 0};

        vector<vector<int>> cost(rows, vector<int>(cols, -1));

        deque<pii> de;
        de.push_back({0, 0});
        cost[0][0] = 0;
        while (!de.empty()) {
            pii cur = de.front();
            de.pop_front();
            for (int i = 0; i < 4; ++i) {
                int nx = dirX[i] + cur.first;
                int ny = dirY[i] + cur.second;
                if (nx < 0 || ny < 0 || nx >= rows || ny >= cols || cost[nx][ny] >= 0) continue;
                cost[nx][ny] = cost[cur.first][cur.second]+grid[nx][ny];
                if (grid[nx][ny] == 1) {
                    de.push_back({nx, ny});
                } else {
                    de.push_front({nx, ny});
                }
            }
        }
        return cost[rows-1][cols-1];
    }
};
```