---
layout: post
title: UVa12569 - Planning mobile robot on Tree (EASY Version)
---
Use BFS and notice state compression. The state is composed of the position of robot and the state (taken/empty) of each node. The robot position has up to 15 choices and nodes state can be up to 2^15. 
If these two are combined together, the states are too many. We can use an int to represent the states. The lower 4 bits to store the robot position, the upper bits (at most 15) to store each node is taken or empty.

```cpp
//#define LOCAL
#include<cstdio>
#include<vector>
#include<map>
#include<queue>
#include<cstring>
#include<unordered_set>
#include<unordered_map>
#include<iostream>
#include<stack>
using namespace std;

typedef pair<int, int> PII;
vector<int> ngh[16];

//unordered_map<int, int> parent;
//unordered_map<int, PII> action;
const int maxn = 15;
const int max_state = (1 << 15);
int parent[maxn][max_state];
int action[maxn][max_state];
bool vis[maxn][max_state];

class ReturnType {
	public:
		int step;
		int state;
		ReturnType(int a, int b) {
			step = a;
			state = b;
		}
};

// state representation: n is at most 15, use lowest 4bits to store robot position, use higher bits to store whether that position is empty or not	
ReturnType bfs(int s, int t, int n) {
	//cout<<s<<" "<<t<<" "<<n<<endl;	
	memset(vis, false, sizeof(vis));
	parent[s & 15][s >> 4] = -1;
	queue<int> q;
	//unordered_set<int> vis;
	q.push(s); vis[s & 15][s >> 4] = true;
	int step = 0;
	while (!q.empty()) {
		int sz = q.size();
		while (sz--) {
			int u = q.front(); q.pop();
			if ((u & 15) == t) return ReturnType(step, u);
			for (int i = 0; i < n; i++) {
				if (!((u >> (4 + i)) & 1)) continue;  // i is not empty
				for (int j: ngh[i]) {
					if (((u >> (4 + j)) & 1)) continue; //j is empty
					// move item in i to j
					int v = u;
					if ((u & 15) == i) { //robot
						v = u - i + j;
					}
					// set i to empty(0), j to filled(1)
					//printf("before clear: %d\n", v);
					v &= (~(1 << (4 + i)));
					//printf("after clear: %d\n", v);
					//printf("before set: %d\n", v);
					v |= (1 << (4 + j));
					//printf("before set: %d\n", v);
					//printf("%d, %d, %d\n", i, j, v);
					if (!vis[v & 15][v >> 4]) {
						action[v & 15][v >> 4] = i * maxn + j;
						parent[v & 15][v >> 4] = u;
						if ((v & 15) == t) return ReturnType(step + 1, v);
						vis[v & 15][v >> 4] = true;
						q.push(v);
					}
				}
			}
		}
		step++;
	}
	return ReturnType(-1, -1);
}

int main() {
    #ifdef LOCAL
    freopen("data.in", "r", stdin);
	freopen("data.out", "w", stdout);
	#endif
	
	int T, n, m, s, t;
	scanf("%d", &T);
	
	for (int kase = 0; kase < T; kase++) {
		scanf("%d%d%d%d", &n, &m, &s, &t);
		for (int i = 0; i < n; i++) {
			ngh[i].clear();
		}
		s--; t--;
		int init = s;
		init |= (1 << (4 + s));
		for (int i = 0; i < m; i++) {
			int tmp;
			scanf("%d", &tmp);
			init |= (1 << (4 + tmp - 1));
		}
		for (int i = 0; i < n - 1; i++) {
			int u, v;
			scanf("%d%d", &u, &v);
			ngh[u - 1].push_back(v - 1);
			ngh[v - 1].push_back(u - 1);
		}
		ReturnType ans = bfs(init, t, n);
		printf("Case %d: %d\n", kase + 1, ans.step);
		
		int state = ans.state;
		if (state >= 0) {
			vector<int> st;
			while (parent[state & 15][state >> 4] >= 0) {
				st.push_back(action[state & 15][state >> 4]);
				state = parent[state & 15][state >> 4];
			}
			
			for (int i = st.size() - 1; i >= 0; i--)
				printf("%d %d\n", st[i] / maxn + 1, st[i] % maxn + 1);
		}

		putchar('\n');
	}
	return 0;
}
```