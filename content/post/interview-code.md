---
title: "面试常见问题-算法编程题"
date: 2020-06-21T10:25:18+08:00
draft: false
tags: 
- code
categories: 
- 面试经验
---

# 腾讯2020校园招聘-后台

## A

### 题目描述

小Q想要给他的朋友发送一个神秘字符串，但是他发现字符串的过于长了，于是小Q发明了一种压缩算法对字符串中重复的部分进行了压缩，对于字符串中连续的m个相同字符串S将会压缩为\[m|S](m为一个整数且1<=m<=100)，例如字符串ABCABCABC将会被压缩为[3|ABC]，现在小Q的同学收到了小Q发送过来的字符串，你能帮助他进行解压缩么？ 

### 代码

```cpp
#include <algorithm>
#include <bitset>
#include <cassert>
#include <cmath>
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <ctime>
#include <iostream>
#include <map>
#include <queue>
#include <random>
#include <set>
#include <stack>
#include <string>
#include <unordered_map>
#include <unordered_set>
#include <vector>
using namespace std;
typedef long long ll;
const double PI = acos(-1.);
//mt19937 myrand(time(0));
string s;
int tot = 0, len;
string get()
{
    tot++;
    int res = 0;
    string ss;
    while (tot < len && s[tot] != '|')
    {
        res = res * 10 + s[tot] - '0';
        tot++;
    }
    tot++;
    while (tot < len)
    {
        if (s[tot] == '[')
        {
            ss += get();
        }
        else if (s[tot] == ']')
        {
            break;
        }
        else
        {
            ss += s[tot];
        }
        tot++;
    }
    string tmp = "";
    for (int i = 0; i < res; i++)
    {
        tmp += ss;
    }
    return tmp;
}
string ans;
int main()
{
    //freopen("in.txt","r",stdin);
    //freopen("out.txt","w",stdout);
 
    cin >> s;
    len = s.size();
    while (tot < len)
    {
        if (s[tot] != '[')
        {
            ans += s[tot];
        }
        else
        {
            ans += get();
        }
        tot++;
    }
    cout << ans;
    return 0;
}
```

## B

### 题目描述

小Q在周末的时候和他的小伙伴来到大城市逛街，一条步行街上有很多高楼，共有n座高楼排成一行。 

小Q从第一栋一直走到了最后一栋，小Q从来都没有见到这么多的楼，所以他想知道他在每栋楼的位置处能看到多少栋楼呢？（当前面的楼的高度大于等于后面的楼时，后面的楼将被挡住） 

### 代码

```cpp
#include <algorithm>
#include <bitset>
#include <cassert>
#include <cmath>
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <ctime>
#include <iostream>
#include <map>
#include <queue>
#include <random>
#include <set>
#include <stack>
#include <string>
#include <unordered_map>
#include <unordered_set>
#include <vector>
using namespace std;
typedef long long ll;
const double PI = acos(-1.);
//mt19937 myrand(time(0));
 
int a[100010];
int ans[100010];
stack<int> stk;
int main()
{
    //freopen("in.txt","r",stdin);
    //freopen("out.txt","w",stdout);
    int n;
    scanf("%d", &n);
    for (int i = 0; i < n; i++)
    {
        scanf("%d", &a[i]);
    }
    for (int i = 0; i < n; i++)
    {
        ans[i] += stk.size();
        while (stk.size() && stk.top() <= a[i])
        {
            stk.pop();
        }
        stk.push(a[i]);
    }
    while (stk.size())
        stk.pop();
    for (int i = n - 1; i >= 0; i--)
    {
        ans[i] += stk.size();
        while (stk.size() && stk.top() <= a[i])
        {
            stk.pop();
        }
        stk.push(a[i]);
    }
    for (int i = 0; i < n; i++)
    {
        printf("%d%c", ans[i] + 1, i == n - 1 ? '\n' : ' ');
    }
    return 0;
}
```

## C

### 题目描述

作为程序员的小Q，他的数列和其他人的不太一样，他有2n2^n2n个数。

  老板问了小Q一共 m次，每次给出一个整数qi(1<=i<=m)q_i (1 <= i <= m)qi(1<=i<=m), 要求小Q把这些数每2qi2^{q_i}2qi分为一组，然后把每组进行翻转，小Q想知道每次操作后整个序列中的逆序对个数是多少呢？ 

例如: 

对于序列1 3 4 2，逆序对有(4, 2),(3, 2),总数量为2。 

翻转之后为2 4 3 1，逆序对有(2, 1),(4, 3), (4, 1), (3, 1),总数量为4。

### 代码

```cpp
#include <algorithm>
#include <bitset>
#include <cassert>
#include <cmath>
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <ctime>
#include <iostream>
#include <map>
#include <queue>
#include <random>
#include <set>
#include <stack>
#include <string>
#include <unordered_map>
#include <unordered_set>
#include <vector>
using namespace std;
typedef long long ll;
const double PI = acos(-1.);
//mt19937 myrand(time(0));
int a[1 << 21];
int b[1 << 21];
int d[1 << 21];
long long ans[25] = {0};
long long aans[25] = {0};
void merge_sort(int a[], int l, int r, int dep, long long c[])
{
    if (l == r)
    {
        return;
    }
    int mid = (l + r) >> 1;
    merge_sort(a, l, mid, dep - 1, c);
    merge_sort(a, mid + 1, r, dep - 1, c);
    int i = l, j = mid + 1, k = l;
    while (i <= mid && j <= r)
    {
        if (a[i] <= a[j])
        {
            b[k++] = a[i++];
        }
        else
        {
            c[dep] += mid - i + 1;
            b[k++] = a[j++];
        }
    }
    while (i <= mid)
    {
        b[k++] = a[i++];
    }
    while (j <= r)
    {
        b[k++] = a[j++];
    }
    for (i = l; i <= r; i++)
    {
        a[i] = b[i];
    }
}
int main()
{
    //freopen("in.txt","r",stdin);
    //freopen("out.txt","w",stdout);
    int n;
    scanf("%d", &n);
    int res = 1 << n;
    for (int i = 0; i < res; i++)
    {
        scanf("%d", &a[i]);
        d[i] = a[i];
    }
    merge_sort(a, 0, res - 1, n, ans);
    reverse(d, d + res);
    merge_sort(d, 0, res - 1, n, aans);
    int q;
    scanf("%d", &q);
    while (q--)
    {
        int m;
        scanf("%d", &m);
        long long temp = 0;
        for (int i = 1; i <= n; i++)
        {
            if (i <= m)
            {
                swap(aans[i], ans[i]);
            }
            temp += ans[i];
        }
        printf("%lld\n", temp);
    }
    return 0;
}
```



## D

### 题目描述

由于业绩优秀，公司给小Q放了 n  天的假，身为工作狂的小Q打算在在假期中工作、锻炼或者休息。他有个奇怪的习惯：不会连续两天工作或锻炼。只有当公司营业时，小Q才能去工作，只有当健身房营业时，小Q才能去健身，小Q一天只能干一件事。给出假期中公司，健身房的营业情况，求小Q最少需要休息几天。

### 代码

```cpp
#include <algorithm>
#include <bitset>
#include <cassert>
#include <cmath>
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <ctime>
#include <iostream>
#include <map>
#include <queue>
#include <random>
#include <set>
#include <stack>
#include <string>
#include <unordered_map>
#include <unordered_set>
#include <vector>
using namespace std;
typedef long long ll;
const double PI = acos(-1.);
//mt19937 myrand(time(0));
const int N = 100010;
int a[N], b[N], dp[N][3];
int main()
{
    //freopen("in.txt","r",stdin);
    //freopen("out.txt","w",stdout);
    int n;
    scanf("%d", &n);
    for (int i = 0; i < n; i++)
    {
        scanf("%d", &a[i]);
    }
    for (int i = 0; i < n; i++)
    {
        scanf("%d", &b[i]);
    }
    if (a[0] == 1)
    {
        dp[0][1] = 1;
    }
    if (b[0] == 1)
    {
        dp[0][2] = 1;
    }
    for (int i = 1; i < n; i++)
    {
        dp[i][0] = max(max(dp[i - 1][0], dp[i - 1][1]), dp[i - 1][2]);
        if (a[i] == 1)
            dp[i][1] = max(dp[i - 1][0], dp[i - 1][2]) + 1;
        if (b[i] == 1)
            dp[i][2] = max(dp[i - 1][0], dp[i - 1][1]) + 1;
    }
    int ans = n - max(max(dp[n - 1][0], dp[n - 1][1]), dp[n - 1][2]);
    printf("%d\n", ans);
    return 0;
}
```

## E

### 题目描述

小Q在进行一场竞技游戏,这场游戏的胜负关键就在于能否能争夺一条长度为L的河道,即可以看作是[0,L]的一条数轴。 

这款竞技游戏当中有n个可以提供视野的道具−真视守卫,第i个真视守卫能够覆盖区间[xi,yi]。现在小Q想知道至少用几个真视守卫就可以覆盖整段河道。 

### 代码

``` cpp
#include <algorithm>
#include <bitset>
#include <cassert>
#include <cmath>
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <ctime>
#include <iostream>
#include <map>
#include <queue>
#include <random>
#include <set>
#include <stack>
#include <string>
#include <unordered_map>
#include <unordered_set>
#include <vector>
using namespace std;
typedef long long ll;
const double PI = acos(-1.);
//mt19937 myrand(time(0));
const int N = 100010;
struct node
{
    int x, y;
    bool operator<(const node &tmp) const
    {
        return y < tmp.y;
    }
} no[N];
priority_queue<node> q;
int main()
{
    //freopen("in.txt","r",stdin);
    //freopen("out.txt","w",stdout);
    int n, l;
    scanf("%d%d", &n, &l);
    for (int i = 0; i < n; i++)
    {
        scanf("%d%d", &no[i].x, &no[i].y);
    }
    sort(no, no + n, [](const node &x, const node &y) { return x.x < y.x; });
    if (no[0].x != 0)
    {
        puts("-1");
        return 0;
    }
    int r = 0;
    int now = 0;
    int ans = 0;
    while (now < n && no[now].x <= r)
    {
        while (now < n && no[now].x <= r)
        {
            q.push(no[now]);
            now++;
        }
        node temp = q.top();
        q.pop();
        ans++;
        if (r >= temp.y)
        {
            puts("-1");
            return 0;
        }
        else
        {
            r = temp.y;
        }
        if (r >= l)
            break;
    }
    if (r < l)
    {
        puts("-1");
        return 0;
    }
    printf("%d\n", ans);
    return 0;
}
```

