---
title: NOIP退役整理 3 数据结构
date: 2018-11-08 22:27:25
categories: 编程
tags: NOIP
---

# NOIP退役整理 3 数据结构

> 看完保证你, 退役!
>
> 感觉NOIP的数据结构并不是很多的说...233

## 0. 并查集

## 1. 树状数组

随便贴一个区间加的

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
#define re register int
#define lowbit(x) (x & -x)
const int maxn = 5e5 + 10;
typedef long long lld;
inline lld read() {
    lld x = 0, f = 1; char c; while ((c = getchar()) > '9' || c < '0') if (c == '-') f = 0; x = c - 48;
    while ((c = getchar()) >= '0' && c <= '9') x = (x << 1) + (x << 3) + c - 48; return f ? x : ~x + 1;
}

lld a[maxn], n, m;

inline void Add(lld x, lld k) {
    for (; x <= n; x += lowbit(x)) a[x] += k;
}

inline lld get(lld x) {
    lld r = 0;
    for (; x; x -= lowbit(x)) r += a[x];
    return r;
}

int main() {
    n = read(); m = read();
    lld now, last = 0;
    for (re i = 1; i <= n; ++ i) {
        now = read(); Add(i, now - last);
        last = now;
    }
    for (re i = 1; i <= m; ++ i) {
        lld op = read(), x = read();
        if (op == 1) {
            lld y = read(), k = read();
            Add(x, k); Add(y + 1, -k);			
        }
        else {
            printf("%lld\n", get(x));
        }		
    }
    return 0;	
}
```



## 2. 线段树

随便写写就好

## 3.ST表

主要用来解决RMQ(区间最值问题)的一种算法, 主要思想竟然是**动态规划**(区间动规)和倍增.支持``O(nlog(n))``预处理, ``O(1)``查询, 而且常数非常小, ~~跑得跟记者一样快~~但是不支持修改,极大地限制了它的运用范围.

方程:
$$
f[i,j] = max(f[i,j-1], f[i + 2^{j-1}, j-1]) \\
其中f[i,j]表示a[i]到a[2^j-1]区间内的最大值
$$
求解的时候
$$
k = log_2(r - l + 1)\\
ans = max(f[l, k], f[r - 2^k + 1, k])
$$
板子

```cpp
// 初始化部分
for (log[0] = -1, register int i = 1; i <= n; ++ i) log[i] = log[i >> 1] + 1; //递推log,在询问较多时可以卡卡常
for (register int i = 1; i <= n; ++ i) st[i][0] = dat[i]; //dp初始化
for (register int j = 1; j <= 22; ++ j) {
	for (register int i = 1; i + (1 << j) - 1 <= n; ++ i) {
		st[i][j] = std::max(st[i][j-1], st[i+(1<<j-1)][j-1]);
	}
}
//查询
inline int query_max(const int & l, const int & r) {
	int k = log[r - l + 1];
	return std::max(st[i][k], st[r - (1 << k) + 1][k]);
}
```



## 4. 平衡树大法!!!

这个不会用到的. 只是个人兴趣而已

## 5. Trie

## 6. 平板电视(ext/pb_ds/*.hpp)

## 7. 差分 前缀和

[这个玩意儿写得不错, 抄下来了.](https://blog.csdn.net/Taunt_/article/details/78478526)

### 前缀和
#### 1.一维前缀和
对于数组A[], 前缀和SUM[i]表示的就是A[1]+A[2]+…+A[i].

```cpp
int init() {
    for(int i = 1; i <= n; i++) sum[i] = sum[i-1] + a[i];
}
int get(int l, int r) {
    return sum[r] - sum[l-1];
}
```



#### 2.二维前缀和
对于二维数组, 前缀和SUM\[i\]\[k\]表示的是所有A\[i’\]\[k’\](1< = i’<=i,i <= k’<=k)的和.

```cpp
int init() {
    for(int i = 1; i <= n; i++) {
        for(int j = 1; j <= m; j++) {
            sum[i][j] = sum[i][j-1] + sum[i-1][j] - sum[i-1][j-1] + a[i][j];
        }
    }
}
int get(int x1, int y1, int x2, int y2) {
    return sum[x1][y1] - sum[x1][y2 - 1] - sum[x2 - 1][y1] + sum[x2 - 1][y2 - 1];
}
```

#### 3.%k时的优化
（p - q）% k= 0 ==> p % k = q % k 
统计q % k 和 p % k 相等的数 
详细见T1

### 差分
#### 1.一维差分
我们对[L,R]区间进行加num操作，在C[L]处加上num，在C[R+1]处减去num 

```cpp
void init(int l,int r,int num) {
    dis[l] += num, dis[r + 1] -= num;
}

int get() {
    for(int i = 1; i <= n; i++) {
        val[i] = val[i-1] + dis[i];
    }
}
```


#### 2.二维差分
其实也挺简单，和二维前缀和一样

```cpp
void init(int x1, int y1, int x2, int y2, int num) {
    sumx1 += num;
    sumx1 -= num;
    sumx2 + 1 -= num;
    sumx2 + 1 += num;
}

void get() {
    for(int i = 1; i <= n; i++) {
        for(int j = 1; j <= n; j++) {
            sumi += sumi + sumi-1 - sumi-1;
        }
    }
}
```

#### 3.树上差分
##### (1)点差分
对 u 到 v 的路径上的点 +num 
用来求 - 已知路径求树上所有节点被路径覆盖次数

```cpp
int init(int u, int v, int num) {
    dis[u] += num;
    dis[v] += num;
    dis[lca(u,v)] -= num;
    dis[f[lca(u,v)]] -= num;
}
```

##### (2)边差分
对 u 到 v 的路径上的边 +num 
用来求 - 已知路径求被所有路径覆盖的边 
dis[i] 表示以i节点为儿子的边

```cpp
int init(int u, int v, int num) {
    dis[u] += num;
    dis[v] += num;
    dis[lca(u,v)] -= 2 *num;
}
```



最后dfs遍历一遍

```cpp
void dfs(int x) {
    for(int i = head[u]; i; i = e[i].next) {
        int v = e[i].v;
        if(v != fx) {    //fx 为倍增数组 
            dfs(v);
            dis[x] += dis[v];
        }
    }
}    
```



