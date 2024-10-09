Linked to [[1009]].

# A 矩阵交换

1. 可以 $m$ 关键字排序。
2. 但是我不会，写了一个弱智的逐级满足。

```cpp
#include <bits/stdc++.h>
using namespace std;

int t, n, m;
vector<int> arr[110];

void solve() {
  cin >> n >> m;
  for(int i = 1; i <= n; i++) {
    arr[i].resize(m + 1);
    for(int j = 1; j <= m; j++) {
      cin >> arr[i][j];
    }
  }
  for(int j = 1; j <= m; j++) {
    stable_sort(arr + 1, arr + n + 1, [j](vector<int> a, vector<int> b) {return a[j] < b[j];});
    for(int i = 2; i <= n; i++) {
      for(int k = 1; k <= j; k++) {
        if(arr[i][k] < arr[i - 1][k]) {
          cout << "NO" << '\n';
          return ;
        }
      }
    }
  }
  cout << "YES" << '\n';
  /*
  for(int i = 1; i <= n; i++) {
    for(int j = 1; j <= m; j++) {
      cout << arr[i][j] << ' ';
    }
    cout << '\n';
  }
  //*/
}

int main() {
  freopen("exchange.in", "r", stdin);
  freopen("exchange.out", "w", stdout);
  for(cin >> t; t--; solve()) {
  }
  return 0;
}
```

# B 砖块摆放

表格，启动！

| A\B | 0   | 1   | 2   |
| --- | --- | --- | --- |
| 0   | 0   | X   | X   |
| 1   | 2   | 1   | X   |
| 2   | 1   | 0   | 2   |
在 ~~把你的眼睛抠出来~~ 大眼观察法后，可以发现 $A \otimes B = -(A + B) \bmod 3$。

然后就可以显然发现这是一个杨辉三角。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int c[3][3] = {{1, 0, 0}, {1, 1, 0}, {1, 2, 1}};

int t, n, arr[200020], ans;
string s;

int C(int x, int y) {
  return x < 0 || y < 0? 0 : c[x][y];
}

int CC(int x, int y) {
  if(x < 3 && y < 3) {
    return C(x, y);
  }
  return C(x % 3, y % 3) * CC(x / 3, y / 3) % 3;
}

int main() {
  freopen("brick.in", "r", stdin);
  freopen("brick.out", "w", stdout);
  for(cin >> t; t--; ) {
    cin >> n >> s;
    for(int i = 0; i < s.size(); i++) {
      arr[i + 1] = s[i] - 'A';
    }
    ans = 0;
    for(int i = 1; i <= n; i++) {
      ans = ans + CC(n - 1, i - 1) * arr[i];
    }
    ans %= 3;
    if(!(n & 1)) {
      ans = (3 - ans) % 3;
    }
    cout << char(ans + 'A') << '\n';
  }
  return 0;
}
```

# C 学习 LIS

## 10pts

显然可以枚举每一个数，$O(m^n n^2)$，但是常数小跑不满。

## 20pts

有脑干的人都会发现我们可以边跑边判断，跑得更快了。

## 30pts

前置：[离散化]([离散化 - OI Wiki (oi-wiki.org)](https://oi-wiki.org/misc/discrete/))

在求 LIS 的时候，我们显然不用关心每一个数的具体的值——我们只关心它们的相对大小。

所以值域降为 $O(N)$。然后就可以用 20pts 的办法了。

## 70pts

Related to [[0925mysol]].

场上我是想到了这个的做法的，但是被否了 >:(

一般 LIS 有一个致命缺陷。这个致命缺陷就是，我们一定得知道一个前缀的所有值（的相对顺序），才能得到最后一个数的 LIS。

所以我们要改变顺序。

从值域从小往大一个一个求，当我们在求 $i$ 时直接 $p_i = \max \limits_{j = 1}^{i - 1} p_j + 1$。\

然后就可以从小往大考虑每一个元素，然后转移了。

时间 $O(3^n n^2)$。

## 100pts

Solve them independently... That's it.

所有的过程是独立的... 子集枚举是没有必要的。直接一个一个的选即可。

In [[Sol]].