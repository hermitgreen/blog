# Kick Start 2020 Round A解题报告

一个小时切到D题，然后D题自闭两小时也没做出来orz

## A:Allocation

### 题意

有N个数，权值为1，花费为 $A_i$ ，现有预算B，求最大能选到多少个数

### 思路

贪心，每次选花费最低的就好了

```C++
#include <bits/stdc++.h>
using namespace std;

int main(void) {
    int T;
    scanf("%d", &T);
    for (int t = 1; t <= T; t++) {
        int n, b;
        priority_queue<int, vector<int>, greater<int>> q;
        scanf("%d%d", &n, &b);
        int k;
        for (int i = 0; i < n; i++) {
            scanf("%d", &k);
            q.push(k);
        }
        int ans = 0;
        while (!q.empty()) {
            k = q.top();
            if (k <= b) {
                b -= k;
                ans++;
            } else {
                break;
            }
            q.pop();
        }
        printf("Case #%d: %d\n", t, ans);
    }
    return 0;
}

```

## B:Plates

### 题意

有N堆盘子，每堆有K个，每个盘子有美观度，取盘子的时候，必须从上往下取，也就是每次只能从堆顶取，现在要求取恰好P个盘子，使美观度最大。

### 思路

DP，计算每堆盘子各自的前缀和a\[N][K]，设dp\[i][j]为取到第i堆时已经取了j个盘子，dp公式如下

$dp[i][j] = max(dp[i][j],dp[i-1][j-h]+a[i][h]) |h<=min(j,K)$ 

```c++
#include <bits/stdc++.h>
using namespace std;
int a[55][55];
int dp[55][5500];
int n, k, p;
int main(void) {
    int T;
    cin >> T;
    for (int t = 1; t <= T; t++) {
        memset(a, 0, sizeof(a));
        memset(dp, 0, sizeof(dp));
        int c, sum;
        scanf("%d%d%d", &n, &k, &p);
        for (int i = 1; i <= n; i++) {
            sum = 0;
            for (int j = 1; j <= k; j++) {
                scanf("%d", &c);
                sum += c;
                a[i][j] = sum;
            }
        }
        for (int i = 1; i <= n; i++) {
            for (int j = 0; j <= p; j++) {
                for (int h = 0; h <= min(j, k); h++) {
                    dp[i][j] = max(dp[i][j], dp[i - 1][j - h] + a[i][h]);
                }
            }
        }
        printf("Case #%d: %d\n", t, dp[n][p]);
    }
    return 0;
}

```



## C:Workout

### 题意

有N个数，可以往这N个数中间插K个数，使得相邻两数的绝对差最小，求最小绝对差

### 思路

模拟+贪心，用优先队列维护绝对差，每次取最大的绝对差，插一个数以后再把他放回去，模拟K次，堆顶的绝对差即为所求

```c++
#include <bits/stdc++.h>
using namespace std;
struct node {
    int l;
    int d;
    node(int l, int d) : l(l), d(d) {}
    node() = default;
    bool operator<(const node b) const {
        int d1 = l / d + (l % d ? 1 : 0);
        int d2 = b.l / b.d + (b.l % b.d ? 1 : 0);
        if (d1 == d2) {
            return d < b.d;
        } else {
            return d1 < d2;
        }
    }
};
priority_queue<node, vector<node>> q;
int main(void) {
    int T;
    scanf("%d", &T);
    for (int t = 1; t <= T; t++) {
        while (!q.empty()) {
            q.pop();
        }
        int n, k;
        int pre, next;
        scanf("%d%d", &n, &k);
        scanf("%d", &pre);
        for (int i = 1; i < n; i++) {
            scanf("%d", &next);
            q.push(node(next - pre, 1));
            pre = next;
        }
        node nod;
        while (k--) {
            nod = q.top();
            q.pop();
            nod.d++;
            q.push(nod);
        }
        nod = q.top();
        int ans = nod.l / nod.d + (nod.l % nod.d ? 1 : 0);
        printf("Case #%d: %d\n", t, ans);
    }
    return 0;
}

```