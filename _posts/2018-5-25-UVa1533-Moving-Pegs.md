---
layout: post
title: UVa1533 - Moving Pegs
---
Use BFS and compress each state into an int. 
The problem description is not good. If multiple solutions exist, should return the smallest one.

```cpp
//#define LOCAL
#include <cstdio>
#include <cstring>
#include <vector>
#include <string>
#include <iostream>
#include <algorithm>
#include <queue>
using namespace std;

typedef pair<int, int> PII;
const int maxn = (1 << 15);

int parent[maxn];
PII action[maxn];
bool vis[maxn];
vector<int> step[15];
vector<int> line[15][6];
string ans[15];

bool valid(int lv, int i) {
    return 0 <= lv && lv <= 4 && 0 <= i && i <= lv;
}

int get_num(int lv, int i) {
    return lv * (lv + 1) / 2 + i;
}

bool empty(int u, int i) { // return true if i th position in u is empty (0)
    return (u & (1 << i)) ? false : true;
}

void bfs(int s, int t) {
    parent[s] = -1; 
    parent[t] = -1;
    memset(vis, 0, sizeof(vis));
    queue<int> q;
    q.push(s); vis[s] = true;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        if (u == t) return;
        for (int i = 0; i < 15; i++) { 
            if (empty(u, i)) continue;
            // i th position not empty
            for (int j = 0; j < 6; j++) {
                if (line[i][j].size() > 1 && !empty(u, line[i][j][0])) { // can jump to
                    int l = 1;
                    for (; l < line[i][j].size() && !empty(u, line[i][j][l]); l++);
                    if (l == line[i][j].size()) continue; //no empty position can jump to
                    // jump i to line[i][j][l]
                    int v = u;
                    v |= (1 << line[i][j][l]);
                    int mask = 0;
                    mask |= (1 << i);
                    for (int k = 0; k < l; k++) {
                        mask |= (1 << line[i][j][k]);
                    }
                    mask = ((~mask) & ((1 << 15) - 1));
                    v &= mask;
                    if (!vis[v]) {
                        parent[v] = u;
                        action[v] = {i, line[i][j][l]};
                        if (v == t) return;
                        vis[v] = true;
                        q.push(v);
                    }
                    
                }
            }
        }
    }
}

void init() {
    //upper left, upper right, left, right, lower left, lower right
    int dlv[6] = {-1, -1, 0, 0, 1, 1};
    int di[6] = {-1, 0, -1, 1, 0, 1};
    
    for (int lv = 0; lv < 5; lv++) {
        for (int i = 0; i <= lv; i++) {
            int x = get_num(lv, i);
            for (int j = 0; j < 6; j++) {
                int curr_lv = lv, curr_i = i;
                while (valid(curr_lv + dlv[j], curr_i + di[j])) {
                    curr_lv += dlv[j];
                    curr_i += di[j];
                    line[x][j].push_back(get_num(curr_lv, curr_i));
                }
            }
        }
    }    
    
    for (int i = 0; i < 15; i++) {
        int s = ((1 << 15) - 1) & (~(1 << i));
        int t = (~s) & ((1 << 15) - 1);
        bfs(s, t);
        if (parent[t] < 0) {
            ans[i] = "IMPOSSIBLE";
        } else {
            int now = t;
            while (parent[now] >= 0) {
                PII act = action[now];
                step[i].push_back(act.second);
                step[i].push_back(act.first);
                now = parent[now];
            }
            reverse(step[i].begin(), step[i].end());
            ans[i] += to_string(step[i].size() / 2);
            ans[i] += "\n";
            for (int j = 0; j < step[i].size(); j++) {
                if (j > 0) {
                    ans[i] += " ";
                }
                ans[i] += to_string(step[i][j] + 1);
            }
        }
    }
}

int main() {
    #ifdef LOCAL
    freopen("data.in", "r", stdin);
    freopen("data.out", "w", stdout);
    #endif
    
    init();
    
//    for (int i = 0; i < 15; i++) {
//        cout << ans[i] << endl;
//    }
    
    int T;
    scanf("%d", &T);
    while (T--) {
        int x;
        scanf("%d", &x); x--;
        cout << ans[x] << endl;
    }
    return 0;
}
```