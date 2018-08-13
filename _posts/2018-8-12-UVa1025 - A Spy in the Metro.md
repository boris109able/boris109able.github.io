---
layout: post
title: UVa 1025 - A Spy in the Metro (DP)
---
设dp(i,t)为在时刻t在站点i时所需的最少等待时间。因为需要判断是否最终能够完成，dp(i,t)为inf(=10000)就表示dp(i,t)状态无法完成会见。
1. 初始条件：dp(N,t)=0；dp(i,t)=inf, 1<=i<=N-1。
2. 状态转移：dp(i,t)=min(dp(i,t+1)+1, dp(i-1, t+t[i-1]), dp(i+1, t+t[i]))。表示等待，向左走，向右走。注意并不是每个状态都可以向左右走，需要判断当前时刻是否有车，以及当前车站否是在最左或最右，以及是否无法会面(t+t[i-1] > T或者t+t[i]>T)。
3. 解：dp(1,0)

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

const int maxn = 50 + 5;
const int maxm = 50 + 5;
const int maxt = 200 + 5;
const int inf = 10000;
int t[maxn], d[maxm], e[maxm];
int l_train[maxn][maxt], r_train[maxn][maxt];
int dp[maxn][maxt];

int main() {  
    int N, T, M1, M2;
    int kase = 1;
    while (scanf("%d", &N) == 1 && N) {
        scanf("%d", &T);
        for (int i = 1; i <= N - 1; i++) {
            scanf("%d", t + i);
        }
        scanf("%d", &M1);
        for (int i = 1; i <= M1; i++) {
            scanf("%d", d + i);
        }
        scanf("%d", &M2);
        for (int i = 1; i <= M2; i++) {
            scanf("%d", e + i);
        }
        
        memset(r_train, 0, sizeof(r_train));
        for (int i = 1; i <= M1; i++) {
            if (d[i] <= T) r_train[1][d[i]] = 1;
        }
        for (int j = 2; j <= N; j++) {
            for (int i = 0; i <= T; i++) {
                if (i - t[j - 1] >= 0) r_train[j][i] = r_train[j - 1][i - t[j - 1]];
            }
        }
        
        memset(l_train, 0, sizeof(l_train));
        for (int i = 1; i <= M2; i++) {
            if (e[i] <= T) l_train[N][e[i]] = 1;
        }
        for (int j = N - 1; j >= 1; j--) {
            for (int i = 0; i <= T; i++) {
                if (i - t[j] >= 0) l_train[j][i] = l_train[j + 1][i - t[j]];
            }
        }
        
        for (int i = 1; i <= N - 1; i++) {
            dp[i][T] = inf;
        }
        dp[N][T] = 0;
        
        for (int i = T - 1; i >= 0; i--) {
            for (int j = 1; j <= N; j++) {
                dp[j][i] = dp[j][i + 1] + 1; // wait
                if (l_train[j][i] && j > 1 && i + t[j - 1] <= T) dp[j][i] = min(dp[j][i], dp[j - 1][i + t[j - 1]]); // go left
                if (r_train[j][i] && j < N && i + t[j] <= T) dp[j][i] = min(dp[j][i], dp[j + 1][i + t[j]]); // go right
            }
        }
        printf("Case Number %d: ", kase++);
        if (dp[1][0] >= inf) printf("impossible\n");
        else printf("%d\n", dp[1][0]);
        
    }
    return 0;  
}
```