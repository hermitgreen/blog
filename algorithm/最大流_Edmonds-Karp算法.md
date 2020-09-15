# 最大流_Edmonds-Karp算法

最大流问题是指求源点s到汇点d的最大流量的一类问题，下面是基于增广路的**Edmonds-Karp**算法

```c++
const int INF = 0x3f3f3f3f;
const int MAX = 205;
struct edge {
    int from;  //头
    int to;    //目标
    int cap;   //容量
    int flow;  //流量
    edge(int u, int v, int c, int f) : from(u), to(v), cap(c), flow(f) {}
};
struct edmondKarp {
    int n;
    int m;
    vector<edge> edges;
    vector<int> G[MAX];  //邻接表
    int a[MAX];          //起点到i的可改进量
    int p[MAX];          //起点的前驱节点

    void init(int n) {
        for (int i = 0; i < n; i++) {
            G[i].clear();
        }
        edges.clear();
    }

    void AddEdge(int from, int to, int cap) {
        edges.push_back(edge(from, to, cap, 0));
        edges.push_back(edge(to, from, 0, 0));  //反向边
        m = edges.size();
        G[from].push_back(m - 2);
        G[to].push_back(m - 1);
    }

    int max_flow(int s, int t) {
        int flow = 0;
        while (true) {
            memset(a, 0, sizeof(a));
            queue<int> q;
            q.push(s);
            a[s] = INF;
            while (!q.empty()) {
                int x = q.front();
                q.pop();
                for (int i = 0; i < G[x].size(); i++) {
                    edge& e = edges[G[x][i]];
                    //如果e边的目标点没被优化过并且有优化空间
                    if (!a[e.to] && e.cap > e.flow) {
                        p[e.to] = G[x][i];
                        //值为前驱节点或者容量限制
                        a[e.to] = min(a[x], e.cap - e.flow);
                        q.push(e.to);
                    }
                }
                //优化到汇点了
                if (a[t]) break;
            }
            //汇点没被优化,说明已经优化到头了
            if (!a[t]) break;
            //从后向前遍历
            for (int u = t; u != s; u = edges[p[u]].from) {
                //出边的流增加
                edges[p[u]].flow += a[t];
                //入边的流减小
                edges[p[u] ^ 1].flow -= a[t];
            }
            //得到的a[t]是整条路的最小残量
            flow += a[t];
        }
        return flow;
    }
};
```