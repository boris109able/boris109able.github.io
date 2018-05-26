---
layout: post
title: UVa817 - According to Bartjens
---
We have three operations "*", "+" and "-". "*" has high priority. "+" and "-" has same low priority. When evaluate a equation, we can first calculate "*" operations, then "+" or "-" each segment result. 
So we can first find the combination of only using "*" for each segment or interval. Use DP. Define prod[i][j] to a map of <string of equation, int of equation result>.
For k in (i, j), each <leq, lv> in prod[i][k], each <req, rv> in prod[k + 1][j], prod[i][j][leq + "*" + req] = lv * rv.

For "+" and "-", when we calculate, we are from left to right. So when recover, we are from right to left. 
eq + a = c, eq = c - a; eq - a = c, eq = c + a. Use DFS.

```cpp
//#define LOCAL
#include <cstdio>
#include <iostream>
#include <vector>
#include <string>
#include <map>
#include <algorithm>
using namespace std;

char s[20];
int len;
string u;
map<string, int> prod[10][10];
string op[3] = {"+", "-", "*"};

bool valid(int i, int j) {
    return u[i] != '0' || i == j;
}

void init() {
    for (int i = 0; i < len; i++) {
        for (int j = 0; j < len; j++) {
            prod[i][j].clear();
        }
    }
    for (int i = 0; i < len; i++) {
        prod[i][i][u.substr(i, 1)] = u[i] - '0';
    }
    for (int l = 2; l <= len; l++) {
        for (int i = 0; i + l - 1 < len; i++) {
            int j = i + l - 1;
            if (valid(i, j)) {
                prod[i][j][u.substr(i, l)] = stoi(u.substr(i, l));
            }
            for (int k = 1; k <= l - 1; k++) {
                map<string, int> lprod = prod[i][i + k - 1];
                map<string, int> rprod = prod[i + k][j];
                for (map<string, int>::iterator it1 = lprod.begin(); it1 != lprod.end(); it1++) {
                    for (map<string, int>::iterator it2 = rprod.begin(); it2 != rprod.end(); it2++) {
                        prod[i][j][it1->first + "*" + it2->first] = it1->second * it2->second;
                    }
                }
            }
        }
    }
}

vector<string> solve(int a, int b, int v) {
    vector<string> ans;
    if (a > b) return ans;
    for (map<string, int>::iterator it = prod[a][b].begin(); it != prod[a][b].end(); it++) {
        if (it->second == v) {
            ans.push_back(it->first);
        }
    }
    
    for (int i = 0; i < 2; i++) {
        for (int j = 1; j <= b - a; j++) {
            map<string, int> r = prod[b - j + 1][b];
            for (map<string, int>::iterator it = r.begin(); it != r.end(); it++) {
                vector<string> l = solve(a, b - j, v + (2 * i - 1) * it->second);
                for (string ls: l) {
                    ans.push_back(ls + op[i] + it->first);
                }
            }
        }
    }
    return ans;
}


int main() {
    #ifdef LOCAL
    freopen("data.in", "r", stdin);
    freopen("data.out", "w", stdout);
    #endif
    
    int kase = 0;
    
    while (scanf("%s", s) == 1 && s[0] != '=') {
        len = 0;
        for (int i = 0; i < 20 && s[i] != '='; i++, len++);
        u = string(s);
        u = u.substr(0, len);
        init();
        vector<string> ans = solve(0, len - 1, 2000);
        sort(ans.begin(), ans.end());
        printf("Problem %d\n", ++kase);
        if (ans.size() == 0 || (ans.size() == 1 && ans[0] == u)) {
            printf("  IMPOSSIBLE\n");
        } else {
            for (string t: ans) {
                if (t == u) continue;
                cout << "  " << t << "=" << endl;
            }
        }
    }
    
    return 0;
}
```