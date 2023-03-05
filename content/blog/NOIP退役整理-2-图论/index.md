+++
title = 'NOIP退役整理 2 图论'
date = '2018-11-08'
categories = ['笔记']
tags = ['NOIP']
math = true
+++
# NOIP退役整理 2 图论

> 看完保证你, 退役!
>
> 继续没有任何证明的笔记

[TOC]

## 0. 基础的算法

### -1. 链式前向星

最最最最重要的存图方法.

```cpp
int head[maxn], cnt = 1;
struct Edge { int next, to, w; } edge[maxn << 1];
inline void add_edge(int u, int v, int w) {
    edge[cnt].next = head[u]; edge[cnt].to = v; edge[cnt].w = w; head[u] = cnt ++;
}

for (int i = head[u]; i; i = edge[i].next) {
    int v = edge[cnt].to;
    //...
}
```



### 0. 百分数(BFS)和电风扇(DFS)

这个我不想说什么. 不过什么DFS的手工栈估计也是不会考,我虽然会写但是只用过一次还炸了

### 1.拓扑排序

拓扑序是一个比较重要的顺序,可以用来做各种事情,比如在图上递推, 或者直接解题什么的.

主要就是在DAG上分析依赖关系. 还可以判环.主要思想就是:每次找入度为0的节点,找到后删除该节点和该节点的出度边.

```cpp
// deg[]是入度. vis[]呵呵. 以下默认使用链星. (生成树除外)
void Topo(int u) {
    vis[u] = 1;
    for (register int i = head[u]; i; i = edge[i].next) {
        int v = edge[i].to;
        deg[v] --;
        if (!deg[v]) Topo(v);
    }
}

//main
for (register int i = 1; i <= n; ++ i) {
    if (!vis[i] && !deg[i]) Topo(i);
}
```

可以在这个主要模板上做各种操作. 判环的话, 记一个color标记(比如:记正在访问为-1, 已经访问为1, 没有访问为0), 如果祖先节点没有返回但是子孙节点又访问到了它, 于是就有环了.

其实还有BFS的写法.估计不考, 就不管了.[随便找了一个板子](https://blog.csdn.net/baodream/article/details/80368764)

## 1.最短路

### 1. Dijkstra

先膜一波: %%%Dijkstra%%%

```cpp
#include<cstdio>
#include<algorithm>
#include<cstring>
#include<queue>
using std::priority_queue;
const int maxn = 1e5 + 10;
const int inf = 0x7fffffff;
inline int read() {
    int x = 0, f = 1; char c; while((c = getchar()) > '9' || c < '0') if (c == '-') f = 0; x = c - 48;
    while((c = getchar()) >= '0' && c <= '9') x = (x << 1) + (x << 3) + c - 48; return f ? x : ~x + 1;
}
int head[maxn], cnt = 1;
struct T_Edge{ int to, next, w; } edge[maxn << 1];
inline void AddEdge(int u, int v, int w) {
    edge[cnt].next = head[u]; edge[cnt].to = v; edge[cnt].w = w; head[u] = cnt ++;
}
struct T_Node {
    int i, d;
    T_Node(int a, int b): i(a), d(b) { }
    inline friend bool operator < (T_Node a, T_Node b) {
        return a.d > b.d;
    }
};
priority_queue<T_Node> pq;
int n, m, s;
int dis[maxn + 10], vis[maxn + 10];
int main() {
    n = read(); m = read(); s = read();
    register int ui, vi, wi;
    for (register int i = 1; i <= m; ++ i){
        ui = read(); vi = read(); wi = read();
        AddEdge(ui, vi, wi);
    }
    for (int i = 1; i <= n; ++ i) {
        dis[i] = inf;
    }
    T_Node now(s, 0);
    now.i = s; now.d = dis[s]  = 0;
    pq.push(now);
    while (!pq.empty()) {
        now = pq.top(); pq.pop();
        int u = now.i;
        if(vis[u]) continue;
        vis[u] = 1;
        for (register int i = head[u]; i; i = edge[i].next) {
            int v = edge[i].to;
            if (!vis[v] && dis[v] > dis[u] + edge[i].w) {
                dis[v] = dis[u] + edge[i].w;
                pq.push(T_Node(v, dis[v]));
            }
        }
    }
    for (register int i = 1; i <= n; ++ i){
        printf("%d ", dis[i]);
    }
    return 0;
}
```

这个的复杂度是严格``O(nlog(n))``的.(其实跑n遍dj比一遍floyd要快(不考虑常数) // 笑)

### 2. SPFA

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
inline int read() { /*...*/}

int head[maxn], cnt = 1;
struct Edge{ int next, to, w; } edge[maxn];
inline void add_edge (int u, int v, int w) {/*...*/}

int vis[maxn], dis[maxn];
inline void spfa(int s) {
    int q[maxn]; 
    memset(q, 0, sizeof(q));
    memset(dis, 0x3f, sizeof(dis));
    int l = 1, r = 1;
    q[l] = s, dis[s] = 0;
    while (l <= r) {
        int u = q[l ++];
        vis[u] = [0];
        for (register int i = head[u]; i; i = edge[i].next) {
            int &v = edge[i].to;
            if (dis[v] > dis[u] + edge[i].w) {
                dis[v] = dis[u] + edge[i].w;
                if (!vis[v]) vis[v] = 1, q[++r] = v;                
            }
        }
    }
}
//main
```

这个写起来比较方便, 虽然有可能会被卡掉.

### 3. Floyd

这个的主要思想是DP. 所以也是可以在上面加个一位瞎搞的. 复杂度很高, ``O(n^3)``

状态转移方程
$$
D[k,i,j] 表示经过几个编号不超过k的节点,从i到j的最短路.我们把这个k作为一个中转点一样的东西 \\\\
显然, D[k, i, j] = min(D[k-1, i, j], D[k-1, i, k] + D[k-1, k, j]) \\\\
再显然,k这一维可以被推掉 \\\\
就变成了: D[i, j] = min(D[i, j], D[i,k] + D[k,j])
$$


```cpp
//init
memset(d, 0x3f, sizeof(d));
for (register int i = 1; i <= n; ++ i) d[i][i] = 0; //自己到自己
// read in...
for (register int k = 1; k <= n; ++ k) {
    for (register int i = 1; i <= n; ++ i) {
        for (register int j = 1; j <= n; ++ j) {
            d[i][j] = std::min(d[i][j], d[i][k] + d[k][j]);
        }
    }
}
```

这个玩意儿似乎可以用来求最小环, 最后再写吧

然而更加牛逼的是, 这个玩意儿的求解过程比较类似于矩阵乘法, 有些看似不可做的题目可以用类似于这样的矩阵快速幂跑.

### 4. 相关题目

1. Luogu P1144 最短路计数. 在跑Dj/ Spfa的时候随便统计一下
2. Luogu P2384 把加法换成了乘法, 保证你退役2333. 其实打个log就好了
3. Luogu P1613 在Floyd上加一维乱搞

### 5. 我是不是应该皮一把k短路

A*可做. 复杂度上界``O(nklog(nk)``

我不会写红红火火恍恍惚惚

## 2.生成树

### 1. Kruskal

```cpp
#include <cstdio> 
#include <cstring>
#include <algorithm>
const int maxn = 1e5 + 10;
inline int read() {
	int x = 0, f = 1; char c; while ((c = getchar()) > '9' || c < '0') if (c == '-') f = 0; x = c - 48;
	while ((c = getchar()) >= '0' && c <= '9') x = (x << 1) + (x << 3) + c - 48; return f ? x : ~x + 1;
}
int fa[maxn];
int findfa(int x) {
	return fa[x] == x ? fa[x] : fa[x] = findfa(fa[x]);
}

struct Edge{
	int from, to, w;
	inline friend bool operator < (Edge a, Edge b) {
		return a.w < b.w;
	}
} edge[maxn];

int main() {
	int n = read(); int m = read();
	for (int i = 1; i <= m; ++ i) 
		edge[i].from = read(), edge[i].to = read(), edge[i].w = read();
	std::sort(edge + 1, edge + m + 1);
	int cnt = 0, ans = 0;
	for (int i = 1; i <= n; ++ i) fa[i] = i;
	for (int i = 1; i <= m; ++ i) {
		int fau;
		if ((fau = findfa(edge[i].from)) != findfa(edge[i].to)){
			fa[edge[i].to] = fau;
			ans += edge[i].w;
			cnt ++;
		}
	}
	if (cnt < n-1) printf("fuck"); //如果边数不对就说明生成失败了
	else printf("%d",ans);
}
```



### 2. Prim 不讲

这个基本用不到. 只有在稠密图上才会优于KS.

### 3. 变种生成树

* 最大生成树就把排序的顺序改一下就好了.

* 次大/小生成树有点难写.

  先求最小生成树T,枚举添加不在T中的边,则添加后一定会形成环,找到环上边值第二大的边,把它删掉,计算当前生成树的权值,取所有枚举修改的生成树的最小值,即为次小生成树.这种方法的实现更为简单,首先求最小生成树T,然后从每个结点u,遍历最小生成树T,用一个二维的数组max\[u\]\[v\]记录结点u到结点v的路径上边的最大值,然后枚举不在T中的边(u,v),计算T-max\[u\]\[v\]+w(u,v)的最小值,即为次小生成树的权值 ,这种方法的时间复杂度为``O(n^2+e)``.

  因为没什么题,就不多说了.

考生成树的时候,一般是Day2T1的难度.基本上不会有什么特别的地方,一旦看出来能敲对就可以了.题目都比较弱智,不贴了

## 3.Tarjan

先膜一波为敬: %%%Tarjan%%%

### 1. 强连通分量

概念就不说了. 主要用来缩点,缩完以后就把一个有环图变成DAG,就可以随便瞎搞了.

求这个玩意儿,其实有另外一个奇葩的跑两边的算法.(在接受FanDalao的指导之前,我一直写的是那个k打头的算法).

先上板子

```cpp
int low[maxn], dfn[maxn], stk[maxn], instk[maxn], tim, top, scc[maxn], num;
// scc[u]代表u属于的强连通分量的编号
// low[u]代表u能到达的最小的dfn,似乎也就是最老的祖先
// dfn代表的似乎是dfs序列
void Tarjan(int u) {
	dfn[u] = low[u] = ++tim;
	instk[stk[++top] = u] = 1;
	for (register int i = head[u]; i; i = edge[i].next) {
		int &v = edge[i].to;
		if (!dfn[v]) {
			Tarjan(v);
			low[u] = std::min(low[u], low[v]);
		}
		else if (instk[v]) {
			low[u] = std::min(low[u], dfn[v]);
		}
	}
	if (low[u] == dfn[u]) { //完了,又回到自己了
		num ++;
		while(1) {
			int v = stk[top --]; 
			instk[v] = 0,
			scc[v] = num;
			if (v == u) break;
		}
	}
}
```

还是有一些有趣的题目的.不过感觉都比较趋同,缩点->搞一搞连通性,最短路,出度入度->没了

### 2. 割点和桥

概念不说.

割点

```cpp
inline void dfs(int u,int root)
{
    int sz = 0;
    dfn[u] = low[u] = ++cnt;
    for (register int i = head[u];i;i = edge[i].next)
    {
        int v = edge[i].to;
        if (dfn[v]) low[u] = min(low[u],dfn[v]);
        else
        {
            sz ++;
            dfs(v,root);
            if (low[v] >= dfn[u]) iscut[u] = true;
            low[u] = min(low[u],low[v]);
        }
    }
    if (u == root && sz < 2)
        iscut[u] = false;
}
// may be more beautiful
void point(int u, int rt) {
	int sz = 0; dfn[u] = low[u] = ++tim;
	for (register int i = head[u]; i; i = edge[i].next ) {
		int &v = edge[i].to;
		if (!dfn[v])  {
			sz ++;
			point(v, rt);
			if (low[v] >= dfn[u]) iscut[u] = 1;
			low[u] = std::min(low[u], low[v]);
		}
		else {
			low[u] = std::min(low[u], dfn[v]);
		}		
	}
	if (u == rt && sz < 2) iscut[u] = 0;
}
```

桥 //正确性不明

```cpp
void bridge(int u, int fa) {
	low[u] = dfn[u] = ++tim;
	for (register int i = head[u]; i; i = edge[i].next){
		int &v = edge[i].to;
		if (!dfn[v]) {
			bridge(v, u);
			low[u] = std::min(low[u], low[v]) ;
			if (low[v] > dfn[u]) { // ">"
				isbridge[u][v] = 1;
			}
		}
		else if (fa != v) {
			low[u] = std::min(low[u], dfn[v]);
		}
	}
}
```

似乎没有什么题目。

### 3. 双联通分量

"点双连通图的定义等价于任意两条边都同在一个简单环中,而边双连通图的定义等价于任意一条边至少在一个简单环中."这个是从网上抄的."不同双连通分量最多只有一个公共点,即某一个割顶,任意一个割顶都是至少两个点双连通的公共点.不同边双连通分量没有公共点,而桥不在任何一个边双连通分量中,点双连通分量一定是一个边双连通分量."

这个东西的求解跟上面的割点和桥差不多,虽然我没写过.(有一条HNOI2012)

过天把代码补上

## 4. LCA

"对呀对呀..求LCA有六种方法,你知道吗?" --fan乙己 %%%%%%

### 1. 暴力

我表示不会写. 这个应该跟倍增差不多思想.就是对于两个点,轮流向上面跳,直到碰起来这个样子.

### 2. ST表/RMQ

这个跟tarjan一样不是很常用.就不打了,省的浪费时间和记忆力

### 3. 倍增

最最最常用而且很好写的lca.大家都在写它.(虽然很容易被卡掉,但是NOIP级别的还没有毒瘤到去卡这个)

主要思想就是先预处理出每个点的深度,然后对于两个点的深度差倍增的向上跳,因为是倍增所以比一般的跳快一点.

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>

const int maxn = 1e6 + 10;

inline int read() {
	int x = 0, f = 1; char c; while ((c = getchar()) > '9' || c < '0') if (c == '-') f = 0; x = c - 48;
	while ((c = getchar()) >= '0' && c <= '9') x = (x << 1) + (x << 3) + c - 48; return f ? x : ~x + 1;
}

int head[maxn], cnt = 1;
struct Edge { int next, to; } edge[maxn<<1];
inline void AddEdge(int u, int v) { edge[cnt].next = head[u]; edge[cnt].to = v; head[u] = cnt++; }
int deep[maxn], l[maxn][30];

void dfs(int u) { //预处理
	for (register int i = 1; i <= 20; ++ i) {
		l[u][i] = l[l[u][i-1]][i-1];
	}
	for (register int i = head[u]; i; i = edge[i].next) {
		int v = edge[i].to;
		if (v == l[u][0]) continue; // 什么,儿子变成父亲了233
		l[v][0] = u; // v的父亲是u,就是从v向上跳2^0,即1步到达u
		deep[v] = deep[u] + 1;
		dfs(v);
	}
}

inline int getlca(int u, int v) {
	if (deep[u] < deep[v]) u ^= v ^= u ^= v;	
	for (register int i = 20; i >= 0; -- i) {
		if (deep[l[u][i]] >= deep[v]) {
			u = l[u][i];
		}
	}//把u和v跳到同一高度
	if  (u == v) return u; //到了一个点上,说明这两个点具有祖先关系
	for (register int i = 20; i >= 0; -- i)
		if (l[u][i] != l[v][i])//一起跳
			u = l[u][i], v = l[v][i];	
	return l[u][0];
}
int n,m,s;
int main()
{    
    n = read(); m = read(); s = read();    
    int tmp1,tmp2;
    for (register int i = 1; i <= n-1;i ++)
    {
        tmp1 = read(); tmp2 = read();
        AddEdge(tmp1,tmp2); AddEdge(tmp2,tmp1);
    }
    deep[s] = 1; dfs(s);
    for (register int i = 1;i <= m; i++)
    {
        tmp1 = read(); tmp2 = read();
        printf("%d\n",getlca(tmp1,tmp2));
    }
}
```



### 4. 树链剖分

树剖是前置技能.不说.

主要思想是把u和v所在链的顶端跳到一起

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
const int maxn = 1e6 + 10;
inline int read() {
    int x = 0, f = 1; char c; while ((c = getchar()) > '9' || c < '0') if (c == '-') f = 0; x = c - 48;
    while ((c = getchar()) >= '0' && c <= '9') x = (x << 1) + (x << 3) + c - 48; return f ? x : ~x + 1;
}

int head[maxn], cnt = 1;
struct Edge { int next, to; } edge[maxn<<1];
inline void AddEdge(int u, int v) { edge[cnt].next = head[u]; edge[cnt].to = v; head[u] = cnt++; }

int sz[maxn], son[maxn], top[maxn], deep[maxn], fa[maxn];
void dfs1(int u) {
	sz[u] = 1, son[u] = 0;
	for (register int i = head[u]; i; i = edge[i].next) {
		int &v = edge[i].to;
		if (v == fa[u]) continue;
		fa[v] = u, deep[v] = deep[u] + 1;
		dfs1(v);
		if (!son[u] || sz[v] > sz[son[u]]) son[u] = v;
		sz[u] += sz[v];
	}
} //处理儿子

void dfs2(int u, int tp) {
	top[u] = tp;
	if (son[u]) dfs2(son[u], tp);
	for (register int i = head[u]; i; i = edge[i].next) {
		int &v = edge[i].to;
		if (v != fa[u] && v != son[u]) dfs2(v, v);
	}
}//处理链顶

inline int getlca(int u, int v) {
	while (top[u] != top[v]) {
		if (deep[top[u]] < deep[top[v]]) u ^= v ^= u ^= v;
		u = fa[top[u]];
	}
	if (deep[u] > deep[v]) u ^= v ^= u ^= v;
	return u;
}

int main() {
	int n = read(), m = read(), s = read();
	for (register int i = 1; i < n; ++ i) {
		int u = read(), v = read();
		AddEdge(u, v); AddEdge(v, u);
	}
	dfs1(s); dfs2(s, s);
	for (register int i = 1; i <= m; ++ i) {
		int ui = read(), vi = read();
		printf("%d\n", getlca(ui, vi));
	}
	return 0;
} 
```



### 5. 离线Tarjan

不是很常用.贴一个久远的板子.

```cpp
#include<cstdio>
#include<iostream>

using namespace std;
const int maxn = 3e6+50;
int n,m,s,cnt1 = 1, cnt2 = 1;
int head1[maxn], head2[maxn], ans[maxn], fa[maxn];
bool vis[maxn];
struct t_edge{
    int next, to;
}edge[maxn];
struct t_query{
    int next, to, num, vis;
}query[maxn];
inline void AddEdge(int,int);
inline void AddQuery(int,int,int);
inline int read();
int father(int);
inline void combine(int,int);
void dfs(int);

int main()
{
    //freopen("testdata.in","r",stdin);
    n = read(), m = read(), s = read();
    //scanf("%d%d%d",&n,&m,&s);
    int ui,vi;
    for (int i = 1; i <= n-1; ++i)
    {
        int ui = read(),vi = read();
        //scanf("%d%d",&ui,&vi);
        AddEdge(ui,vi); AddEdge(vi,ui);
    }
    for (int i = 1; i <= m; ++i)
    {
    	int ui = read(),vi = read();
        //scanf("%d%d",&ui,&vi);
        AddQuery(ui,vi,i);AddQuery(vi,ui,i);
    }
    //fclose(stdin);
    for (int i = 1; i <= n; ++i)
    	fa[i] = i;
    dfs(s);
    for (int i = 1;i <= m; ++i)
        printf("%d\n",ans[i]);
    return 0;
}

inline void AddEdge(int u,int v)
{
    edge[cnt1].next = head1[u];
    edge[cnt1].to = v;
    head1[u] = cnt1 ++;
}

inline void AddQuery(int u,int v,int num)
{
    query[cnt2].next = head2[u];
    query[cnt2].to = v;
    query[cnt2].num = num;
    head2[u] = cnt2 ++;
}

inline int read()
{
    int x = 0, f = 1; char c;
    while ((c = getchar()) > '9' || c < '0') if (c == '-') f = 0; x = c - 48;
    while ((c = getchar()) >= '0' && c <= '9') x = (x << 3) + (x << 1) + c - 48;
    return f ? x : -x;
}

int father(int x)
{
    if (fa[x] != x) fa[x] = father(fa[x]);
    return fa[x];
}

inline void combine(int x,int y)
{
    fa[father(x)] = father(y);
}

void dfs(int u)
{
    vis[u] = true;
    for (int i = head1[u]; i ; i = edge[i].next)
    {
        int &v = edge[i].to;
        if (vis[v]) continue;
        dfs(v);
        combine(v,u);
    }
    for (int i = head2[u]; i ; i = query[i].next)
    {
        int &v = query[i].to;
        if (vis[v] && !query[i].vis)
        {
            ans[query[i].num] = father(v);
            query[i].vis = true;
        }
    }
}
```



### 6. 笛卡尔树

这个不会写, 巨难

## 5. 二分图

### 1. 二分染色
这个算法似乎还是考过的,主要用来判断一个图是否是二分图,还可以顺带做一些操作.看到有某些互斥操作或者两边分的比较明显的就可以考虑二分图相关的东西.染色很简单,暴力遍历一边就结束了.

题目:
1. Luogu P1155. 其实也可以不用二分图.也比较不好想到是二分图
2. Luogu P1330. 这个比较明显. 

蒟蒻表示自己基本只刷过luogu...没有什么别的题库.tcl

### 2. 二分图匹配
这个里面有许多奇奇怪怪的概念:  
1. 定义：给定一个二分图G，在G的一个子图M中，M的边集{E}中的任意两条边都不依附于同一个顶点，则称M是一个匹配。  
   匹配点：匹配边上的两点
2. 极大匹配(Maximal Matching)：是指在当前已完成的匹配下,无法再通过增加未完成匹配的边的方式来增加匹配的边数。
3. 最大匹配(maximum matching)：是所有极大匹配当中边数最大的一个匹配,设为M。选择这样的边数最大的子集称为图的最大匹配问题。
4. 完美匹配（完备匹配）：一个图中所有的顶点都是匹配点的匹配，即2|M| = |V|。完美匹配一定是最大匹配，但并非每个图都存在完美匹配。
5. 最优匹配：最优匹配又称为带权最大匹配，是指在带有权值边的二分图中，求一个匹配使得匹配边上的权值和最大。一般X和Y集合顶点个数相同，最优匹配也是一个完备匹配，即每个顶点都被匹配。如果个数不相等，可以通过补点加0边实现转化。一般使用KM算法解决该问题。（KM（Kuhn and Munkres）算法，是对匈牙利算法的一种贪心扩展。）
6. 最小覆盖 二分图的最小覆盖分为最小顶点覆盖和最小路径覆盖：  
   ①最小顶点覆盖是指最少的顶点数使得二分图G中的每条边都至少与其中一个点相关联  
   注：二分图的最小顶点覆盖数=二分图的最大匹配数

   ②最小路径覆盖也称为最小边覆盖，是指用尽量少的不相交简单路径覆盖二分图中的所有顶点。   
   注：二分图的最小路径覆盖数=|V|-二分图的最大匹配数
7. 最大独立集:最大独立集是指寻找一个点集，使得其中任意两点在图中无对应边。对于一般图来说，最大独立集是一个NP完全问题，对于二分图来说最大独立集=|V|-二分图的最大匹配数。最大独立集S 与 最小覆盖集T 互补
8. [这个当然是抄的](https://blog.csdn.net/ling_wang/article/details/79830980)

#### 1. 二分图最大匹配
就是找到最多的匹配个数. 经典模型为稳定婚姻问题.

二分图匹配本质上是一个网络流问题,只要再左边一列的左边加一个源点,右边一列右边再加一个汇点,跑最大流就好了.但是对于二分图来说其实不必要这个样子,可以简单一点.

接下来是板子. 其实代码并不是很难, 难在发现这是一个二分图,再把模型建起来.这一点在网络流里面也是一个巨大的问题.233

算法为匈牙利算法.主要思想就是先随便匹配,遇到有重合的就考虑拆掉原来的,再给原来的重新配一个.知道再也配不到为止.(一个无比凶残的算法)
```cpp
// Luogu P3386
#include <cstdio>
#include <cstring>
#include <algorithm>

const int maxn = 1e6 + 10;

inline int read() {
	int x = 0, f = 1; char c; while ((c = getchar()) > '9' || c < '0') if (c == '-') f = 0; x = c - 48;
	while ((c = getchar()) >= '0' && c <= '9') x = (x << 1) + (x << 3) + c - 48; return f ? x : ~x + 1;
}

int head[maxn], cnt = 1;
struct Edge { int to, next; } edge[maxn << 1];
inline void add_edge(int u, int v) {
	edge[cnt].next = head[u]; edge[cnt].to = v; head[u] = cnt ++;
}

int pre[maxn], vis[maxn], tim, ans;

int dfs(int u) {
	for (register int i = head[u]; i; i = edge[i].next) {
		int & v = edge[i].to;
		if (vis[v] == tim) continue; //神奇的常数优化,不需要memset了
		vis[v] = tim;
		if (!pre[v] || dfs(pre[v])) { // 自己未被匹配 || 可以腾出一个位置; 这个写得顺序也是一个常数优化
			pre[v] = u; // 重新匹配成功
			return 1;
		}
	}
	return 0;
}

int main() {
	int n = read(), m = read(), e = read();
	for (register int i = 1; i <= e; ++ i) {
		int ui = read(), vi = read();
		if (ui > n || vi > m) continue;
		add_edge(ui, vi);
	}
	for (register int i = 1; i <= n; ++ i) {
		tim ++;
		if (dfs(i)) ans ++;
	}
	printf("%d\n", ans);
	return 0;
}
```
#### 2. 二分图最大带权匹配(最优匹配)

就是给每一个匹配加一个权值,然后求最大匹配,同时使权值最优.

同样可以跑网络流. 不过主要使KM算法.这个东西可以用,但是似乎从来没有刻意去考过, 估计也暂时不将去考.

鉴于我自己只是了解大概思想, 板子就没有了.

## 6. 各种奇怪算法

### 1. 差分约束

有一大堆形如 
$$
x{i} - x{j} \leq c_{k}
$$
 的不等式, 因为它们长得非常像SPFA / DJ中的三角形不等式, 于是乎可以用图论方法来求解这一堆东西的关系. 

如果出现小于, 就在后面-1就好了, (至于double请自求多福), 如果是大于等于之类的, 打个负号然后考场上现场瞎编就好了.

如果存在负环, 表明不满足条件 (貌似很多题里面只要有环就不成立, 到时候随机应变就好了)

如果跑出来是个inf, 就表明没有任何的限制. 

dis的结果就是每一条约束链的最小花费, (一组解?)

下面是个板子

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>

const int maxn = 10001;

inline int read() {
    int x = 0, f = 1; char c; while ((c = getchar()) > '9' || c < '0') if (c == '-') f = 0; x = c - 48;
    while ((c = getchar()) >= '0' && c <= '9') x = (x << 1) + (x << 3) + c - 48; return f ? x : ~x + 1;
}

struct t_edge{ int to, next, w; } edge[maxn << 1];
int head[maxn], cnt = 1;
inline void add_e(int u, int v, int w) {
    edge[cnt].next = head[u]; edge[cnt].to = v; edge[cnt].w = w; head[u] = cnt ++;
}
int dis[maxn], vis[maxn], flg;
int n, m;
void spfa(int s) {
    if (flg) return;
    vis[s] = 1;
    for (register int i = head[s]; i; i = edge[i].next) {
        int v = edge[i].to;
        if (flg) return;
        if (dis[v] < dis[s] + edge[i].w) {
            dis[v] = dis[s] + edge[i].w;
            if (!vis[v]) spfa(v);
            else { flg = 1; return; }
        }
    }
    vis[s] = 0;
}

int main() {
    n = read(); m = read();
    int op, ai, bi, ci;
    for (register int i = 1; i <= m; ++ i) {
        op = read(); ai = read(); bi = read();
        switch(op) {
            case 1:{
                ci = read();
                add_e(bi, ai, ci);
                break;
            }
            case 2: {
                ci = read();
                add_e(ai, bi, -ci);
                break;
            }
            case 3: {
                add_e(ai, bi, 0); add_e(bi, ai, 0);
                break;
            }
        }
    }
    for (register int i = 1; i <= n; ++ i) {
        spfa(i); if (flg) break;
    }
    printf(flg ? "No\n" : "Yes\n");
    return 0;
}
```



### 2. 负环

深搜或者SPFA都可以用来判负环

至于正环, 随便写一写就好了

上代码

大法师:

```cpp
// luogu-judger-enable-o2
#include <cstdio>
#include <cstring>
#include <algorithm>

const int maxn = 3001;

inline int read() {
    int x = 0, f = 1; char c; while ((c = getchar()) > '9' || c < '0') if (c == '-') f = 0; x = c - 48;
    while ((c = getchar()) >= '0' && c <= '9') x = (x << 1) + (x << 3) + c - 48; return f ? x : ~x + 1;
}

struct t_edge{ int to, next, w; } edge[maxn << 1];
int head[maxn], cnt = 1;
inline void add_e(int u, int v, int w) {
    edge[cnt].next = head[u]; edge[cnt].to = v; edge[cnt].w = w; head[u] = cnt ++;
}
int dis[maxn], vis[maxn], flg;
int n, m;
void spfa(int s) {
    if (flg) return;
    vis[s] = 1;
    for (register int i = head[s]; i; i = edge[i].next) {
        int v = edge[i].to;
        if (flg) return;
        if (dis[v] > dis[s] + edge[i].w) {
            dis[v] = dis[s] + edge[i].w;
            if (!vis[v]) spfa(v);
            else { flg = 1; return; }
        }
    }
    vis[s] = 0;
}

int main() {
    int t = read();
    while (t --) {
        n = read(); m = read(); int ui, vi, wi;
        flg = 0; cnt = 1; 
        memset(vis, 0, sizeof(vis));
        memset(head, 0, sizeof(head));
        memset(dis, 0, sizeof(dis));
        for (register int i = 1; i <= m; ++ i) {
            ui = read(); vi = read(); wi = read();
            if (wi < 0) add_e(ui, vi, wi);			
            else { add_e(ui, vi, wi); add_e(vi, ui, wi); }
        }
        for (register int i = 1; i <= n; ++ i) {
            spfa(i); if (flg) break;
        }
        printf(flg ? "YE5\n" : "N0\n");
    }
    return 0;
}
```

百分数:

```cpp
#include<bits/stdc++.h>
#define IL inline
#define RI register int
#define N 100086
#define clear(a) memset(a,0,sizeof a)
#define rk for(RI i=1;i<=n;i++)
using namespace std;
IL void read(int &x)
{
    int f=1;x=0;char s=getchar();
    while(s>'9'||s<'0'){if(s=='-')f=-1;s=getchar();}
    while(s<='9'&&s>='0'){x=x*10+s-'0';s=getchar();}
    x*=f;
}
int n,m,T;
struct code{int u,v,w;}edge[N];
bool vis[N];
int head[N],tot,dis[N],cnt[N];
IL void add(int x,int y,int z){edge[++tot].u=head[x];edge[tot].v=y;edge[tot].w=z;head[x]=tot;}
IL bool spfa(int now)
{
    rk vis[i]=false,dis[i]=2147483647,cnt[i]=false;
    queue<int>q;
    q.push(now);
    vis[now]=true;
    dis[now]=0;
    while(!q.empty())
    {
        int u=q.front();q.pop();vis[u]=false;
        if(cnt[u]>=n)return true;
        for(RI i=head[u];i;i=edge[i].u)
        {
            if(dis[edge[i].v]>dis[u]+edge[i].w)
            {
                dis[edge[i].v]=dis[u]+edge[i].w;
                if(!vis[edge[i].v])
                {
                    q.push(edge[i].v);
                    vis[edge[i].v]=true;
                    cnt[edge[i].v]++;
                    if(cnt[edge[i].v]>=n)return true;
                }
            }
        }
    }
    return false;
}
int main()
{
    read(T);
    while(T--)
    {
        read(n),read(m);
        tot=0;clear(head);
        for(RI i=1,u,v,w;i<=m;i++)
        {
            read(u),read(v),read(w);
            if(w<0)add(u,v,w);
            else add(u,v,w),add(v,u,w);
        }
        puts(spfa(1)?"YE5":"N0");
    }
}
```



### 3.  最小环

可以用Floyd来求, 似乎dj也可以做. 不过网上似乎只有Floyd的做法, 于是乎抄写一波.

### 4. 反图

这个主要是用来解决不能到达终点的情况的. 很多时候正反跑一遍 或者直接反着跑就会奇迹再现

### 5. 欧拉路

### 6. 哈密顿路

不管了

### 7. 倍增

这个可以用来优化图上 / 树上的长度问题. 具体写得话, 随缘了.

**记得不要把数组开太小**. 还有这个玩意儿有点耗空间. (不过这几年不卡空间就是了)

## 7.一些奇特的东西

### 1.DFS序和DFS树

咕咕咕...

## 8. 一些奇葩题

1. POJ3613 看起来是图论题的矩阵快速幂
2. NOIP2013 华容道: 非常牛逼的一条图论建模