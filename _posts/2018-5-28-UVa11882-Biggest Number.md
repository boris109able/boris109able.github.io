---
layout: post
title: UVa11882 - Biggest Number
---
Iterative deepening DFS. Tricks:
1. Pre-compute neighbor of each position, and sort the neighbors from big to small.
2. Initialize the initial depth to be the max number of connected component, and search by decreasing depth.
3. Once found a solution, only need to compare with solution of current layer. 
But we do not need to wait until we found a solution, if the current path is path[0..d], 
if path[0..d] < solution[0..d], we can prune since we can no longer find a larger solution.


```cpp
//#define LOCAL
#include <cstdio>
#include <cstring>
#include <queue>
#include <vector>
#include <algorithm>
#include <iostream>
using namespace std;

typedef pair<int, int> PII;
bool vis[16 * 16];
int R, C;
char path[40];
char board[16][16];
vector<int> ngh[40];
vector<int> pos[10];
int maxd;

string result;

const int di[4] = {1, -1, 0, 0};
const int dj[4] = {0, 0, 1, -1};

bool myFunc(const PII& p1, const PII& p2) {
    return p1.first > p2.first;
}
bool valid(int i, int j) {
    return 0 <= i && i < R && 0 <= j && j < C && board[i][j] != '#';
}

int init_depth() {
    int d = 0;
    memset(vis, false, sizeof(vis));
    for (int i = 0; i < R; i++) {
        for (int j = 0; j < C; j++) {
            if (vis[i * C + j] || board[i][j] == '#') continue;
            queue<int> q;
            q.push(i * C + j); vis[i * C + j] = true;
            int cnt = 0;
            while (!q.empty()) {
                int u = q.front(); q.pop(); cnt++;
                for (int v: ngh[u]) {
                    if (!vis[v]) {
                        q.push(v);
                        vis[v] = true;
                    }
                }
            }
            d = max(d, cnt);
        }
    }
    memset(vis, false, sizeof(vis));
    return d;
}

bool less_than(char* path, int d) {
    // return true if path [0, d] is less than [0, d] of result
    // return false if equal or greater
    for (int i = 0; i <= d; i++) {
        if (result[i] != path[i]) return path[i] < result[i];
    }
    return false;
}

void dfs(int u, int d) {
    path[d] = board[u / C][u % C];
    if (!result.empty() && less_than(path, d)) return;
    if (d == maxd - 1) {
        path[d + 1] = '\0';
        result = string(path);
        return;
    }
    for (int v: ngh[u]) {
        int ni = v / C, nj = v % C;
        if (vis[v]) continue;
        vis[v] = true;
        dfs(v, d + 1);
        vis[v] = false;
    }
}

void ida_star() {
    for (maxd = init_depth(); maxd >= 1; maxd--) {
        for (int begin = 9; begin >= 1; begin--) {
            for (int p: pos[begin]) {
                vis[p] = true;
                dfs(p, 0);
                vis[p] = false;
            }
            if (!result.empty()) return;
        }
    }
}

int main() {
    #ifdef LOCAL
    freopen("data.in", "r", stdin);
    freopen("data.out", "w", stdout);
    #endif 
    
    char s[20];
    while (scanf("%d%d", &R, &C) == 2 && R && C) {
        for (int i = 0; i < R; i++) {
            scanf("%s", board[i]);
        }
        
        for (int i = 1; i <= 9; i++) pos[i].clear();
        for (int i = 0; i < R * C; i++) ngh[i].clear();
        result.clear();
        for (int i = 0; i < R; i++) {
            for (int j = 0; j < C; j++) {
                if (board[i][j] == '#') continue;
                pos[board[i][j] - '0'].push_back(i * C + j);
                vector<PII> tmp;
                for (int k = 0; k < 4; k++) {
                    int ni = i + di[k], nj = j + dj[k];
                    if (valid(ni, nj)) {
                        tmp.push_back({board[ni][nj] - '0', ni * C + nj});
                    }
                }
                sort(tmp.begin(), tmp.end(), myFunc);
                for (PII p: tmp) {
                    ngh[i * C + j].push_back(p.second);
                }
            }
        }
        
        ida_star();
        cout << result << endl;
    }
    return 0;
}
```