---
layout: post
title: UVa11846 - Finding Seats Again
---
Exact covering problem. Use DLX. Notice details.
```cpp
//#define LOCAL
#include <cstdio>
#include <algorithm>
#include <vector>
#include <cstring>
#include <map>
using namespace std;

typedef pair<int, int> PII;
vector<PII> shape[10];
int mat[10000][20 * 20];
const int max_sz = 10000 * 20 * 20;
int L[max_sz], R[max_sz], U[max_sz], D[max_sz], S[20 * 20];
int r, n, len;
char board[21][21];

void init() {
    for (int i = 1; i <= 9; i++) {
        for (int j = 1; j <= i; j++) {
            if (i % j == 0) {
                shape[i].push_back({j, i / j});
            }
        }
    }
}

void fill(int row, int r1, int c1, int r2, int c2) {
    for (int i = r1; i <= r2; i++) {
        for (int j = c1; j <= c2; j++) {
            mat[row][i * n + j] = 1;
        }
    }
}

void init_dl() {
    // dancing link initialization
    memset(mat, 0, sizeof(mat));
    r = 0;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            if (board[i][j] == '.') continue;
            int num = board[i][j] - '0';
            for (PII p: shape[num]) {
                for (int shape_r = max(0, i - p.first + 1); shape_r <= min(i, n - p.first); shape_r++) {
                    for (int shape_c = max(0, j - p.second + 1); shape_c <= min(j, n - p.second); shape_c++) {
                        fill(r++, shape_r, shape_c, shape_r + p.first - 1, shape_c + p.second - 1);
                    }
                }
            }
        }
    }
    len = n * n + 1;
    // header row
    for (int i = 0; i < len; i++) {
        int next = (i + 1) % len;
        R[i] = next;
        L[next] = i;
    }
    // connect rows, LR
    for (int i = 0; i < r; i++) {
        vector<int> vj;
        for (int j = 0; j < n * n; j++) {
            if (mat[i][j]) {
                vj.push_back((i + 1) * len + j + 1);
            }
        }
        int vj_sz = vj.size();
        for (int j = 0; j < vj_sz; j++) {
            int now = vj[j];
            int next = vj[(j + 1) % vj_sz];
            R[now] = next;
            L[next] = now;
        }
    }
    // connect columns, UD
    memset(S, 0, sizeof(S));
    for (int j = 0; j < n * n; j++) {
        vector<int> vi;
        vi.push_back(0 * len + j + 1);
        for (int i = 0; i < r; i++) {
            if (mat[i][j]) {
                vi.push_back((i + 1) * len + j + 1);
            }
        }
        int vi_sz = vi.size();
        for (int i = 0; i < vi_sz; i++) {
            int now = vi[i];
            int next = vi[(i + 1) % vi_sz];
            D[now] = next;
            U[next] = now;
        }
        S[j + 1] = vi_sz;
    }
}

void coverCol(int c) {
    L[R[c]] = L[c]; R[L[c]] = R[c];
    for (int i = D[c]; i != c; i = D[i]) {
        for (int j = R[i]; j != i; j = R[j]) {
            D[U[j]] = D[j]; U[D[j]] = U[j];
            S[j % len]--;
        }
    }
}

void uncoverCol(int c) {
    for (int i = U[c]; i != c; i = U[i]) {
        for (int j = L[i]; j != i; j = L[j]) {
            D[U[j]] = j; U[D[j]] = j;
            S[j % len]++;
        }
    }
    L[R[c]] = c; R[L[c]] = c;
}

vector<int> path;
bool dlx() {
    if (!R[0]) return true;
    int minc = 10000, minv = 10000;
    int c;
    for (c = R[0]; c != 0; c = R[c]) {
        if (S[c] < minv) {
            minc = c;
            minv = S[c];
        }
    }
    c = minc;
    coverCol(c);
    for (int row = D[c]; row != c; row = D[row]) {
        path.push_back(row);
        for (int j = R[row]; j != row; j = R[j]) {
            coverCol(j % len);
        }
        if (dlx()) return true;
        
        for (int j = L[row]; j != row; j = L[j]) {
            uncoverCol(j % len);
        }
        path.pop_back();
    }
    uncoverCol(c);
    return false;
}

void fill_board(int num, char c) {
    int col = num % len - 1;
    board[col / n][col % n] = c;
}

void print_solution() {
    for (int i = 0; i < path.size(); i++) {
        char c = (char)('A' + i);
        int num = path[i];
        fill_board(num, c);
        for (int j = R[num]; j != num; j = R[j]) {
            fill_board(j, c);
        }
    }

    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            putchar(board[i][j]);
        }
        putchar('\n');
    }
}

int main() {
    #ifdef LOCAL
    freopen("data.in", "r", stdin);
    freopen("data.out", "w", stdout);
    #endif
    
    init();    
    int K;
    while (scanf("%d%d", &n, &K) == 2 && n && K) {
        path.clear();
        for (int i = 0; i < n; i++) {
            scanf("%s", board[i]);
        }      
        init_dl();
        dlx();
        print_solution();
    }
    
    return 0;
}
```