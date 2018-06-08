---
layout: post
title: UVa10384 - The Wall Pushers
---
Compress states into integer and BFS.
Horizontal walls numbered from [1, 29].
Vertical walls numbered from [30, 57].
Row number can be stored in [58, 59].
Column number can be stored in [60, 63] (actually [63] bit is not taken, so we can return -1 when we found a solution (no collision with all possible states)).
Notice details and orders of directions.
```cpp
//#define LOCAL
#include <cstdio>
#include <vector>
#include <algorithm>
#include <unordered_map>
#include <unordered_set>
#include <queue>
using namespace std;

typedef long long LL;

const char dir[4] = {'W', 'N', 'E', 'S'};
const int dr[4] = {0, -1, 0, 1};
const int dc[4] = {-1, 0, 1, 0};

unordered_map<LL, LL> parent;
unordered_map<LL, char> action;

int wallNum(LL r, LL c, int i) {
    if (i == 0) return 30 + r * 7 + c; // west wall
    else if (i == 1) return r * 6 + c; //north wall
    else if (i == 2) return 30 + r * 7 + c + 1; // east wall
    else if (i == 3) return (r + 1) * 6 + c; // south wall
    return -1;
}

bool isValid(LL r, LL c) { return 0 <= r && r < 4 && 0 <= c && c < 6;}

bool hasWall (LL u, LL r, LL c, int k) {
    int num = wallNum(r, c, k);
    return (u >> num) & 1LL;
}

void bfs(LL s) {
    unordered_set<LL> vis;
    queue<LL> q;
    parent.clear(); action.clear();
    q.push(s); vis.insert(s);
    while (!q.empty()) {
        LL u = q.front(); q.pop();
        LL ur = (u >> 58) & 3LL, uc = (u >> 60) & 15LL;
        for (int k = 0; k < 4; k++) {
            LL vr = ur + dr[k], vc = uc + dc[k];
            if (!hasWall(u, ur, uc, k) || (isValid(vr, vc) && !hasWall(u, vr, vc, k))) {
                if (!isValid(vr, vc)) {
                    parent[-1] = u;
                    action[-1] = dir[k];
                    return;
                }
                // go from (ur, uc) to (vr, vc)
                LL v = u & (~(63LL << 58));
                v = v | (vr << 58) | (vc << 60);
                if (hasWall(u, ur, uc, k)) { // need to push wall
                    int from = wallNum(ur, uc, k);
                    int to = wallNum(vr, vc, k);
                    v = v & (~(1LL << from)) | (1LL << to);
                }
                if (!vis.count(v)) {
                    q.push(v);
                    vis.insert(v);
                    parent[v] = u;
                    action[v] = dir[k];
                }
            }
        }
    }
}

int main() {
    #ifdef LOCAL
    freopen("data.in", "r", stdin);
    freopen("data.out", "w", stdin);
    #endif
    
    LL x, y, tmp;
    while (scanf("%lld%lld", &y, &x) == 2 && x && y) {
        x--; y--;
        LL s = 0;
        s = s | (x << 58) | (y << 60);
        for (int i = 0; i < 4; i++) {
            for (int j = 0; j < 6; j++) {
                scanf("%lld", &tmp);
                for (int k = 0; k < 4; k++) {
                    s |= (((tmp >> k) & 1LL) << wallNum(i, j, k));
                }
            }
        }
        bfs(s);
        LL t = -1;
        vector<char> path;
        while (parent.count(t)) {
            path.push_back(action[t]);
            t = parent[t];
        }
        reverse(path.begin(), path.end());
        for (char c: path) 
            putchar(c);
        putchar('\n');
    }
    return 0;
}
```