---
layout: post
title: UVa11694 - Gokigen Naname
---
从左向右，从上到下，逐次填好。剪枝条件：1. cell (i, j) 必须满足 dot (i, j) 的连接数，因为以后没有格子能连它 2. 其他三个顶点必须小于满足数 3. 最后一行，每行最后一格都要特殊对待 4. 不能形成环 （无向图找环 DFS）
```cpp
//#define LOCAL
#include <cstdio>
#include <iostream>
#include <cstring>
#include <string>
#include <vector>
#include <cctype>
using namespace std;

int n;
char dot[8][8];
char board[7][7];
bool hasEdge[8][8][8][8];
int connected[8][8];
int cnt;

const int go[4][2] = {1,1, 1,-1,  -1,-1, -1,1};

bool isValidDot(int i, int j) {
    return 0 <= i && i < n + 1 && 0 <= j && j < n + 1;
}

void printBoard() {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            putchar(board[i][j]);
        }
        putchar('\n');
    }
}

bool vis[8][8];

bool detectCycle(int ni, int nj, int pi, int pj) { // ni, nj, pi, pj is corner index, pi, pj is parent of ni, nj
    vis[ni][nj] = true;
    for (int l = 0; l < 4; l++) {
        int nni = ni + go[l][0], nnj = nj + go[l][1];
        if (nni == pi && nnj == pj) continue;
        if (isValidDot(nni, nnj) && hasEdge[ni][nj][nni][nnj]) {
            if (vis[nni][nnj] || detectCycle(nni, nnj, ni, nj)) return true;
        }
    }
    return false;
}

bool hasCycle(int ni, int nj) {
    memset(vis, false, sizeof(vis));
    vis[ni][nj] = true;
    for (int l = 0; l < 4; l++) {
        int nni = ni + go[l][0], nnj = nj + go[l][1];
        if (isValidDot(nni, nnj) && hasEdge[ni][nj][nni][nnj]) {
            if (detectCycle(nni, nnj, ni, nj)) return true;
        }
    }
    return false;
}

bool dfs(int ci, int cj) { // ci, cj is cell index
    if (cj == n) {
        if (isdigit(dot[ci][n]) && connected[ci][n] != dot[ci][n] - '0') return false;
        ci++;
        cj = 0;
        if (ci == n) {
            for (int j = 0; j < n + 1; j++) {
                if (isdigit(dot[n][j]) && connected[n][j] != dot[n][j] - '0') return false;
            }
            return true;
        }
        else {
            return dfs(ci, cj);
        }
    }
    else {
        board[ci][cj] = '\\';
        connected[ci][cj]++;
        connected[ci + 1][cj + 1]++;
        hasEdge[ci][cj][ci + 1][cj + 1] = true;
        hasEdge[ci + 1][cj + 1][ci][cj] = true;
        if (isdigit(dot[ci][cj]) && connected[ci][cj] != dot[ci][cj] - '0') {
            ;
        }
        else if (isdigit(dot[ci + 1][cj + 1]) && connected[ci + 1][cj + 1] > dot[ci + 1][cj + 1] - '0') {
            ;
        }
        else if (!hasCycle(ci, cj) && dfs(ci, cj + 1)) {
            return true;
        }
        hasEdge[ci][cj][ci + 1][cj + 1] = false;
        hasEdge[ci + 1][cj + 1][ci][cj] = false;
        connected[ci][cj]--;
        connected[ci + 1][cj + 1]--;
        
        board[ci][cj] = '/';
        connected[ci + 1][cj]++;
        connected[ci][cj + 1]++;
        hasEdge[ci + 1][cj][ci][cj + 1] = true;
        hasEdge[ci][cj + 1][ci + 1][cj] = true;
        if (isdigit(dot[ci][cj]) && connected[ci][cj] != dot[ci][cj] - '0') {
            ;
        }
        else if (isdigit(dot[ci + 1][cj]) && connected[ci + 1][cj] > dot[ci + 1][cj] - '0') {
            ;
        }
        else if (isdigit(dot[ci][cj + 1]) && connected[ci][cj + 1] > dot[ci][cj + 1] - '0') {
            ;
        }
        else if (!hasCycle(ci + 1, cj) && dfs(ci, cj + 1)) {
            return true;
        }
        hasEdge[ci + 1][cj][ci][cj + 1] = false;
        hasEdge[ci][cj + 1][ci + 1][cj] = false;
        connected[ci + 1][cj]--;
        connected[ci][cj + 1]--;
        
        return false;
    }
}

int main() {
    #ifdef LOCAL
    freopen("data.in", "r", stdin);
    freopen("data.out", "w", stdout);
    #endif
    
    int N;
    string s;
    getline(cin, s);
    N = stoi(s);
    while (N--) {
        getline(cin, s);
        n = stoi(s);
        for (int i = 0; i < n + 1; i++) {
            getline(cin, s);
            for (int j = 0; j < n + 1; j++) {
                dot[i][j] = s[j];
            }
        }
        memset(board, ' ', sizeof(board));
        memset(hasEdge, false, sizeof(hasEdge));
        memset(connected, 0, sizeof(connected));
        cnt = 0;
        dfs(0, 0);
        printBoard();
    }
    return 0;
}
```