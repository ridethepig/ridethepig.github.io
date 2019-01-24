---
title: NOIP退役整理 1 数学相关
date: 2018-11-05 18:29:25
categories: 编程
tags: NOIP
---
# NOIP退役整理 1  数学相关

> 看完保证你,退役
>
> 这篇笔记里从不写证明

[TOC]

### 0.更加基础的算法

这里贴几个最最最最基础的算法

#### 1.√n求因数

```cpp
int cnt = 0, fac[maxn];
inline void get_factor(int x)  {    
	for (register int i = 1; i * i <= x; ++ i) {
		if (x % i == 0) {
			fac[++ cnt] = i; 
			if (i != x / i) fac[++ cnt] = x / i;
		}
	}
}
```

#### 2.gcd

```cpp
int gcd(int a, int b) {
    return b == 0 ? a : gcd(b, a % b);
}
```

#### 3.√n分解质因数

```cpp
int cnt = 0; prime[maxn], index[maxn];
inline void prime_factor(int x) {    
    for (register int i = 2; i * i <= n; ++ i)
        if (x % i == 0) {
            prime[++ cnt] = i, index[cnt] = 0;
            while (n % i == 0) x /= i, index[cnt] ++;
    	}
    if (x > 1) prime[++ cnt] = x, index[cnt] = 1;
}
```



### 1. 裴蜀定理 (Bezout)

$$
ax + by = m
$$

对于这样一个不定方程,当且仅当``gcd(a,b) | m``的时候,有无数多个整数解.特别的,对于 ``ax + by = 1``, 当且仅当a和b互质时有如上结论.

另: 对于任意多个未知数和任意多个系数, 这个结论也是成立的. 也就是
$$
a_{1}x_{1} + a_{2}x_{2} + \cdots + a_{n}x_{n} = d \\
其中d = (a_{1}, a_{2}, \cdots,a_{n})
$$
这样一个式子. 

似乎简单的用法就是推出系数和后面的解的关系,然后用``gcd``或者``整数相关的东西``瞎搞.

例题:

1. BZOJ 1441. 显然就是推广结论的裸题
2. <JSOI 2009 瓶子和燃料> 虽然感觉和Bezout没有什么关系...个人认为,这就是个结论题, 似乎用辗转相减更好理解一点
3. <HAOI2011 向量> 这个比较难,要多推几步.也是一条Bezout定理.(从某个聚铑博客里发现的题) (这个我不会写)
4. NOIP2014 Day2T3 解方程. 这个似乎就是直接枚举(因为1e6好像可以过去),然后Bezout定理去检验.(话说这题好皮啊...)(然而我也不会写)

### 2.拓展欧几里得 (Ex_Gcd)

#### 1.基本内容

这个算是一个比较基础的算法,用来解线性同余方程.

这个方程大概长这个样子:
$$
ax \equiv b\pmod {m}
$$
我们需要求整数解.

把这个玩意儿变个形, 就得到了
$$
ax + my = b
$$
这不就是个二元不定方程吗.这个可以用exgcd来解.显然, 由Bezout定理可知, 如果有整数解,那就一定有无数个这样的解. 当且仅当``(a,m) | b``时有整数解

所以在解方程之前需要判定合法. 也就是判断``(a, m)|b``是否成立.

先上代码

```cpp
void ex_gcd(int a, int b, int & d, int & x, int &y) {
    if (b == 0) {
        x = 1, y = 0, d = a;
    }
    else {
        ex_gcd(b, a % b, d, y, x); // 划重点. x和y不要打反了
        y -= x * (a / b); //这个括号一定要打。否则会先乘溢出
    }
}
```

解出来以后,这只是一个解.

所有的解可以由一种构造方法得到, 也就是通解.
$$
x_{i} = x_{0} + i*b/(a,m),i\in Z \\
y_{i} = y_{0} - i*b/(a,m),i\in Z
$$
还有最小整数解:
$$
x_{min} = (x*b/d \bmod (m / d) + m / d) \bmod (m/d) \\
其中d = (a, m); \\
$$
似乎还可以这么写: (不是很清楚这两个有什么区别) 总而言之,求逆元用下面的,最小整数解用上面的
$$
x_{min} = (x \bmod m + m) \bmod m
$$

#### 2. 中国剩余定理

用来解线性同余方程组.背结论好了.
$$
设m_1, m_2, m_3, \cdots, m_n 是两两互质的数\\
设 m = \Pi_{i=1}^nm_i\\
M_i = \frac{m}{m_i}\\
t_i是同余方程 M_i*t_i \equiv 1 \pmod m_i的一个解\\

$$

$$
则对于任意n个整数a_1,a_2,\cdots,a_n,\\
\begin{equation}
\left\{
\begin{array}{lr}
x \equiv a_1 \pmod{m_1} \\
x \equiv a_2 \pmod{m_2} \\
\cdots \\
x \equiv a_n \pmod {m_n}
\end{array}
\right.
\end{equation} \\
该方程组有整数解, 为x = \Sigma_{i=1}^{n}a_iM_it_i
$$
同样的,这也只是一个特解. 最小整数解需要%一下.

终于到例题了:

1. Poj1061 青蛙的约会. 经典老题
2. NOIP2012 同余方程. 裸的拓欧.
3. 没了. 附赠:[一个不错的讲解](http://www.cnblogs.com/frog112111/archive/2012/08/19/2646012.html)

### 3.各种筛 (把你打成筛子)

这个相信大家都很清楚是什么. 就当板子放在这里.

#### 1.素数筛

```cpp
inline void get_prime(int n) {
	memset(vis, 0, sizeof(vis));
	int cnt = 0;
	for (register int i = 2; i <= n; ++ i) {
		if (!vis[i]) {
			vis[i] = prime[++ cnt] = i;
		} 
		for (int j = 1; j <= cnt; ++ j)  {
			if (prime[j] > vis[i] || prime[j] > n / i) 
				break;
			vis[i * prime[j]] = prime[j];
		}		
	}
}
```

顺手判素数

```cpp
inline bool is_prime(int x) {
    for (register int i = 1; i * i <= x; ++ i) {
        if (x % i == 0) {
            return false;
        }
    }
    return true;
}
```



#### 2.欧拉筛

```cpp
inline void get_phi(int n) {
	memset(vis, 0, sizeof(vis));
	int cnt = 0;
	for (register int i = 2; i <= n; ++ i) {
		if (!vis[i]) {
			vis[i] = prime[++ cnt] = i;
			phi[i] = i - 1;
		} 
		for (int j = 1; j <= cnt; ++ j)  {
			if (prime[j] > vis[i] || prime[j] > n / i) 
				break;
			vis[i * prime[j]] = prime[j];
			phi[i * prime[j]] = phi[i] * (i % prime[j] ? prime[j] - 1 : prime[j]);
		}		
	}
}
```

求一个phi. 长得和分解质因数一毛一样

```cpp
int get_phi(int n) {
    int ans = n;
    for (register int i = 2; i * i <= n; ++ i) {
        if (n % i == 0) {
            ans = ans / i * (i - 1);
            while (n % i == 0) n /= i;
        }
    }
    if (n > 1) ans = ans / n * (n - 1);
    return ans;
}
```

顺便一提, phi(n)表示小于等于n的素数个数

积性函数什么的反正考不到我也不会,就不管了.

大概的表是 0, 1, 2, 2, 4, 2, 6, 4, 10 ...... 这个可以打表想到

例题: Poj3090.可以由暴力算法观察结果得出,这个是个欧拉函数(反正当时做的时候是打表出来的).至于具体证明,呵呵

### 4.逆元

逆元就是用来做除法取模的东西. 一个数处以另一个数等于被除数乘上除数的逆元.(因为在取模的时候,除法不满足结合律, 所以要用逆元)我们有多种方法求逆元,直接给出代码.

#### 1.exgcd

这个其实就是线性同余方程右边是1的情况下的解, 很明显它是唯一的,而且可以用exgcd解.

```cpp
int get_inv(int n, int p) {
	int x, y, d;
	ex_gcd(n, p, x, y, d);
	return (x%p+p)%p;
}
```

复杂度 ``O(log(n))``



#### 2.费马小定理

这个只有在a,p互素的时候才能用

由
$$
a^{p-1} \equiv 1 \pmod {p}, p是素数
$$
可以随便瞎推得
$$
inv(i) = i^{p-2} \pmod{p}, p是素数
$$
因为用了一个快速幂,所以复杂度是``O(log(n))``

```cpp
int get_inv_fm(int x, int p) {
	int k = p - 2, ans = 1;
	x %= p;
	for (; k; k >>= 1) {
		if (k & 1) ans = ans * x % p;
		x = x * x % p;
	}
	return ans % p;
}
```

其实还有什么Euler定理,当n垂直于a的时候,a^φ(n) 同余于 1(在模n意义下).这样的玩意儿



#### 3.线性递推

```cpp
inline void get_inv_arr(int n, int p){
	inv[1] = 1;
	for (register int i = 2; i <= n; ++ i) {
		inv[i] =(long long)(p - p/i) * inv[p % i] % p;	//防溢出
	}
}
```

这个没什么好说的,反正不长,背下来就好.

### 5.组合相关

#### 1.基本内容

* 加法, 乘法原理

* 排列数

$$
P_{n}^{m} = \frac{n!}{(n-m)!}
$$

* 组合数

$$
C_{n}^{m} = \frac{n!}{m!(n-m)!} = \frac{P_{n}^{m}}{m!}
$$

* 相关计算性质

$$
C_{n}^{m} = C_{n}^{n-m} \\
C_{n}^{m} = C_{n-1}^{m} + C_{n-1}^{m-1} \\
\Sigma_{i = 0}^{n}C_{n}^{i} = 2^{n}
$$

这些东西可以用来递推或者化简.

虽然不知道是什么,但是还是写一下多重集的组合数吧233
$$
对于一个多重集S = \{n_{1}*a_{1},n_2*a_2,\dots,n_k*a_k\} \\
设整数r<=n_i(i\in[1,k]),从S中取出r个元素的不同多重集数量是C_{k+r-1}^{k-1}
$$


有时候需要求组合数,接下来给出相关代码

1. 求一个组合数

   ```cpp
   inline int comb(int n, int m, int p) {
   	int n_1 = 1, m_1 = 1;
   	for (int i = n; i >= n - m + 1; -- i) n_1 = n_1 * i % p;
   	for (int i = m; i >= m; -- i) m_1 = m_1 * i % p;
   	return n_1 * get_inv(m_1, p) % p; //假装我们这里有一个求逆元的函数
   }
   ```

   这个比较慢,``O(n)``的

2. 求很多个

   ```cpp
   //假设我们有了所有阶乘的取模的值和其逆元的值
   return fac[n] * fac_inv[m] * fac[n - m] % p;
   ```

* 二项式定理(杨辉三角形)

就是那个noip2011 day2t1.
$$
(a+b)^{n} = \Sigma_{i = 0}^{n}C_{n}^{i}a^{i}b^{n-i}
$$

```cpp
for (register int i = 1; i <= k; ++ i) {
		comb[i][0] = comb[i][i] = 1;
}	
for (register int i = 2; i <= k; ++ i) {
	for (register int j = 1; j <= n; ++ j) {
		comb[i][j] = (comb[i-1][j] + comb[i-1][j-1]) % hhr;
	}
}
```

#### 2.Lucas定理

$$
C_{n}^{m} \equiv C_{n \bmod p}^{m \bmod p} * C_{n/p}^{m/p} \pmod {p}
$$

注意,这里的p是**素数**, 如果是合数的情况,似乎是可以分解来做. 我给的地址里面有讲到.

唯一的~~例题似乎是BJZOJ1951的古代猪文.比较难,还要合并线性同余方程组(中国剩余定理).不会写

带Lucas定理的求组合数. 据说在模数比较大的时候不需要Lucas(来自某位dalao)

```cpp
typedef long long ll;

inline ll fast_pow(ll a, ll b, ll &p) {
	ll ans = 1; a %= p;
	for (; b; b >>= 1) {
		if (b & 1) ans = ans * a % p;
		a = a * a % p;
	}
	return ans % p;
}

ll Comb(ll n, ll m, ll &p) {
	if (m > n) return 0;
	ll ans = 1;
	for(register int i = 1; i <= m; ++ i) {
		ll a = (n - m + i) % p;
		ll b = i % p;
		ans = ans * (a * fast_pow(b, p-2, p) % p) % p; // 费马小定理逆元 
	}
	return ans;
}

ll Lucas(ll n, ll m, ll &p) {
	if (m == 0) return 1;
	return Comb(n % p, m % p, p) * Lucas(n / p, m / p, p) % p;
}
```

其实还有拓展Lucas定理...不管了.

赠送:[一个很好的文章](https://blog.csdn.net/acdreamers/article/details/8037918)

#### 3.容斥原理

这个东西说简单也很简单, 但是不是非常好说明.那么一大堆数学公式不但难打,而且看着也嫌烦. **举个例子**,就是那个高一数学做的集合题, 那个什么参加乒乓球, 篮球还有什么足球的同学一共多少个什么的东西.这个东西在竞赛中的应用大概是把一类计数类问题进行转换,转换成比较便于计算的形式,然后得出结果,通常是什么逆向思维.也有分类之后算总和的时候进行容斥以得出正确的结论.

因为我自己写得比较烂,理解也非常浅陋,下面贴几个Blog:

[1](https://blog.csdn.net/m0_37286282/article/details/78869512)

#### 4.隔板法

**隔板法**就是在n个元素间的(n-1)个空中插入k个板,可以把n个元素分成k+1组的方法.

这个东西讲起来比较麻烦...2333继续放blog(其实是我不会)

[1](https://blog.csdn.net/sdz20172133/article/details/81431066)

### 6.矩阵相关

矩阵的基本计算就不说了.

矩阵考的最多的就是用矩阵来加速多项式的递推计算,也就是矩阵快速幂

#### 1.矩阵快速幂

先上板子

```cpp
struct matrix {
	ll dat[maxn][maxn]; 
	int n, m;
	matrix(int sz1, int sz2) : n(sz1), m(sz2) {
		memset(dat, 0, sizeof(dat));
	}
	inline friend matrix operator + (matrix a, matrix b) {
		matrix c(a.n, a.m);				
		for (register int i = 1; i <= a.n; ++ i) {
			for (register int j = 1; j <= a.m; ++ j) {
				c.dat[i][j] = a.dat[i][j] + b.dat[i][j];
			}
		}
		return c;
	}
	inline friend matrix operator * (matrix a, matrix b) {
		matrix c(a.n, b.m);				
		for (register int i = 1; i <= c.n; ++ i) {
			for (register int j = 1; j <= c.m; ++ j) {
				for (register int k = 1; k <= a.m; ++ k) {
					c.dat[i][j] = (c.dat[i][j] + a.dat[i][k] * b.dat[k][j] % p) % p;
				}
			}
		}
		return c;
	}
};
inline matrix fast_pow(matrix a, ll k) {
	matrix ans(a.n, a.m);
	for (register int i = 1; i <= ans.n; ++ i) {
		ans.dat[i][i] = 1;
	}
	for (; k; k >>= 1) {
		if (k & 1) ans = ans * a;
		a = a * a;
	}
	return ans;
}
```

#### 2.矩阵加速递推

对于一个递推式,我们可以把它放在某一个矩阵里面.然后它的每一次递推操作可以使用矩阵运算来解决,套上快速幂就会像记者一样.难点主要在于构建单位矩阵.

以Fibonacci为例:

我们有
$$
F[n] = F[n-1] + F[n-2]
$$
然后把它放到一个1*2的矩阵里面(我喜欢横着的)
$$
\left[
\begin{matrix}
F[n-1] & F[n]
\end{matrix}
\right] \\
\left[
\begin{matrix}
F[n-2] & F[n-1]
\end{matrix}
\right]
$$
我们希望从上面一个推到下面一个, 于是乎由于``F[n-1] = 0 * F[n-2] + 1 * F[n-1]``, ``F[n] = 1 * F[n-1]  + 1 * F[n-2]``,可以得出如下单位矩阵
$$
\left[
\begin{matrix}
0 & 1 \\
1 & 1 \\
\end{matrix}
\right]
$$
我们只要把这个矩阵做上n-1次乘法就可以得到结果了.

例题很多,就不一一列举了.

### 7.线性基(虽然可能不考)

这个比较简单，用来求一堆数异或起来的最值问题。主要的做法就是把这一堆数字的二进制给存起来，然后去一个一个异或。用到了向量的思想，不多说，背板子。

```cpp
const int maxbit = 63;
struct linear_base {
	linear_base() {
		memset(dat, 0, sizeof(dat));
	}
	long long dat[maxbit + 1];
	bool insert(long long n) {
		for (register int i = maxbit; i >= 0; -- i) {
			if (n & (1LL << i)) {
				if (!dat[i]){
					dat[i] = n;
					break;
				}
				n ^= dat[i];
			}
		}
		return n > 0;
	}
	long long get_min() {		
		for (register int i = maxbit; i >= 0; -- i) {
			if (dat[i]) return dat[i];
		}
		return 0;
	}
	long long get_max() {
		long long ret = 0;
		for (register int i = maxbit; i >= 0; -- i) {
			if ((ret ^ dat[i]) > ret) {
				ret ^= dat[i];
			}
		}
		return ret;
	}
};
```

附赠:[一个很好的文章](https://blog.csdn.net/qaq__qaq/article/details/53812883)

### 8.概率期望(碰到就放弃系列)

#### 1.基本内容

#### 2.期望DP

### 9.神奇的东西

#### 1.题目收集

1. Luogu: 斐波那契公约数 gcd(F[n], F[m]) = F[gcd(n,m)]
2. Luogu: P4388 phi

#### 2.GCD相关性质

并不清楚有什么

#### 3.几个数列

##### 1. Catalan

$$
Cat_n = \frac{C_{2n}^{n}}{n + 1}
$$

几个常见的情况:

1. 合法括号匹配序列数(n左n右 -> Cat(n))
2. 合法出栈序列数(n个数 -> Cat(n))
3. n个节点构成的不同二叉树的数量

给个表:1, 1, 2, 5, 14, 42, 132, 429, 1430, 4862, 16796, 58786, 208012......

##### 2. Fibonacci

这个就不说了

##### 3.欧拉函数

前面有

##### 4.约数个数

####  4. 数论分块

这个详见余数求和那道题

#### 5.一些结论

$$
(x + 1)^p \equiv x^p + 1 \pmod{p}
$$

可以用来证明Lucas定理(对于OI来说没什么用)
$$
对于a \perp b, 形如k*a + b的素数有无数个
$$
虽然不知道有什么用



### 10.致谢

百度 cnblogs csdn luogu <算法竞赛进阶指南> <信息学奥赛一本通 提高> <具体数学>

