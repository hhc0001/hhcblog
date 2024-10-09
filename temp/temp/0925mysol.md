Linked to [[0925]].

# A 变幻

易得连续两个数不会一起操作。

所以就可以设 $dp_{i, j}$ 表示考虑前 $i$ 个，操作了 $j$ 次的最大权值。

由于最后一个数不会被操作，转移易得：

当 $a_{i + 1}$ 是山谷数时，$(i, j) \to (i + 2, j) + a_{i + 1}$

否则：$(i, j) \to (i + 1, j), (i, j) \to (i + 2, j + 1) + (\min(a_i, a_{i + 2}) - 1)$

```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;

int n, k, arr[2020], dp[2020][2020];

signed main() {
  freopen("change.in", "r", stdin);
  freopen("change.out", "w", stdout);
  cin >> n >> k;
  for(int i = 1; i <= n; i++) {
    cin >> arr[i];
  }
  memset(dp, 0x80, sizeof(dp));
  dp[0][0] = 0;
  for(int i = 0; i < n; i++) {
    for(int j = 0; j <= k; j++) {
      if(arr[i] > arr[i + 1] && arr[i + 2] > arr[i + 1]) {
        dp[i + 2][j] = max(dp[i + 2][j], dp[i][j] + arr[i + 1]);
      }else {
        dp[i + 1][j] = max(dp[i + 1][j], dp[i][j]);
        dp[i + 2][j + 1] = max(dp[i + 2][j + 1], dp[i][j] + (min(arr[i], arr[i + 2]) - 1));
      }
    }
  }
  int ans = 0x8080808080808080;
  for(int j = 0; j <= k; j++) {
    ans = max(ans, dp[n][j]);
  }
  cout << ans << '\n';
  return 0;
}
```

# B 交替

结论题。

记 $l = \lceil \frac{n}{2} \rceil - 1$。

当 $n$ 是奇数，答案为 $\sum \limits_{i = 0}^l (-1)^i \binom{l}{i} a_{2i}$；

否则，答案为 $\sum \limits_{i = 0}^l (-1)^i \binom{l}{i} (a_{2i} + a_{2i + 1})$

处理一下组合数直接算就行了。至于这个规律咋来的，我是手玩了 $n$ 比较小的情况。

```cpp
#include <bits/stdc++.h>
#define int ll
using namespace std;
using ll = long long;

const int kMod = int(1e9) + 7;

int n, arr[100010], fac[100010], inv[100010], rl, ans, val;

int qinv(int x) {
  int rs = 1;
  for(int tmp = kMod - 2; tmp; tmp >>= 1) {
    if(tmp & 1) {
      rs = 1ll * rs * x % kMod;
    }
    x = 1ll * x * x % kMod;
  }
  return rs;
}

void init() {
  fac[0] = 1;
  for(int i = 1; i <= 100000; i++) {
    fac[i] = 1ll * fac[i - 1] * i % kMod;
  }
  inv[100000] = qinv(fac[100000]);
  for(int i = 99999; i >= 0; i--) {
    inv[i] = 1ll * inv[i + 1] * (i + 1) % kMod;
  }
}

long long c(int x, int y) {
  if(x < 0 || y < 0 || x < y) {
    return 0;
  }
  return 1ll * fac[x] * inv[x - y] % kMod * inv[y] % kMod;
}

signed main() {
  freopen("alternate.in", "r", stdin);
  freopen("alternate.out", "w", stdout);
  init();
  cin >> n;
  for(int i = 0; i < n; i++) {
    cin >> arr[i];
  }
  rl = (n + 1) / 2 - 1, val = 1;
  if(n & 1) {
    for(int i = 0; i <= rl; i++) {
      ans = (ans + val * c(rl, i) * arr[i << 1] % kMod + kMod) % kMod;
      val = val * -1;
    }
  }else {
    for(int i = 0; i <= rl; i++) {
      ans = (ans + val * c(rl, i) * (arr[i << 1] + arr[(i << 1) | 1]) % kMod + kMod) % kMod;
      val = val * -1;
    }
  }
  cout << ans << '\n';
  return 0;
}

/*
auto [x, y, z] = ...
*/
```

# C 打拳

Related to [[1009mysol]].

为了高效的求 LIS，我们需要从值从小往大求以每一个数结尾的 LIS 长度。

比如说：

$a = [1, 1, 4, 5, 1, 4, 1, 9]$，LIS 的新求法会这样求：

$[0, 0, 0, 0, 0, 0, 0, 0] \to$

$p = [0, 0, 0, 0, 0, 0, 1, 0] \to$
$[0, 0, 0, 0, 1, 0, 1, 0] \to$
$[0, 1, 0, 0, 1, 0, 1, 0] \to$
$[1, 1, 0, 0, 1, 0, 1, 0] \to$
$[1, 1, 0, 0, 1, 2, 1, 0] \to$
$[1, 1, 2, 0, 1, 2, 1, 0] \to$
$[1, 1, 2, 3, 1, 2, 1, 0] \to$
$[1, 1, 2, 3, 1, 2, 1, 4]$

最后的 LIS 就是 $\max \limits_{i = 1}^n p_i$。

这样子，我们就可以从小往大考虑每一个人，并将这些人分配。

写一个记搜加上三个剪枝即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

struct my_hash {
  int operator ()(const pair<int, int> &x) const {
    return (((x.first * 1145141 % 998244353) << 1) ^ 1919810) + ((((x.second % 1145141) << 3) & 1919810) ^ 11451419) % 998244353;
  }
};

int n, m, k, kMod, fac[2020], inv[2020], arr[20], pt[20][10], cc[2020][2020];
//unordered_map<pair<int, int>, int, my_hash> dp;
map<pair<int, int>, int> dp;
string lis;

int qinv(int x) {
  int rs = 1;
  for(int tmp = kMod - 2; tmp; tmp >>= 1) {
    if(tmp & 1) {
      rs = 1ll * rs * x % kMod;
    }
    x = 1ll * x * x % kMod;
  }
  return rs;
}

void init() {
  cc[0][0] = 1;
  fac[0] = 1;
  for(int i = 1; i <= 2000; i++) {
    fac[i] = 1ll * fac[i - 1] * i % kMod;
    cc[i][0] = 1;
    for(int j = 1; j <= i; j++) {
      cc[i][j] = (cc[i - 1][j] + cc[i - 1][j - 1]) % kMod;
    }
  }
}

inline int c(int x, int y) {
  if(x < 0 || y < 0) {
    return 0;
  }
  return cc[x][y];
}

int DFS(int cur, int lls, int val, int cbest) {
  if(cur > m) {
    for(auto i : lis) {
      if(i == '0') {
        return 0;
      }
    }
    for(auto i : lis) {
      if(i - '0' >= k) {
        return 1;
      }
    }
    return 0;
  }
  if(__builtin_popcount(val) + m - cur + 1 < n) {
    return 0;
  }
  if(cbest + n - __builtin_popcount(val) < k) {
    return 0;
  }
  if(dp.find({cur, lls}) != dp.end()) {
    return dp[{cur, lls}];
  }
  int rs = DFS(cur + 1, lls, val, cbest);
  for(int i = 0; i < n; i++) {
    if(lis[i] == '0') {
      int mx = 0;
      for(int j = 0; j < i; j++) {
        mx = max(signed(mx), lis[j] - '0');
      }
      lis[i] = mx + '1';
      if(c(arr[cur] - val - 2, (1 << i) - 1) && fac[1 << i]) {
        rs = (rs + 1ll * DFS(cur + 1, lls + pt[i][mx + 1], val | (1 << i), max(cbest, lis[i] - '0')) * c(arr[cur] - val - 2, (1 << i) - 1) % kMod * fac[1 << i]) % kMod;
      }
      lis[i] = '0';
    }
  }
  return dp[{cur, lls}] = rs;
}

int main() {
  freopen("box.in", "r", stdin);
  freopen("box.out", "w", stdout);
  cin >> n >> m >> k >> kMod;
  init();
  for(int i = 1; i <= m; i++) {
    cin >> arr[i];
  }
  sort(arr + 1, arr + m + 1);
  for(int i = 0; i < n; i++) {
    lis += '0';
  }
  pt[0][1] = 1;
  for(int i = 1; i <= 8; i++) {
    pt[i][1] = pt[i - 1][1] * 10;
    for(int j = 2; j <= 9; j++) {
      pt[i][j] = pt[i][j - 1] + pt[i][1];
    }
  }
  cout << ((DFS(1, 0, 0, 0) + 0ll) << n) % kMod << '\n';
  return 0;
}
/*
3 3 2 872207251
8 5 4
*/
```

In [[Sol]].