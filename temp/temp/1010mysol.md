布什，戈门，AB 两题都是单调栈啊？！

# A 植物收集

首先把数组 Rotate 至以最小值为第一个（这不影响答案），方便后续断环成链。

然后，对于一个固定的科技使用量，这是一个 RMQ 问题！

然后傻傻的 hhc 写了个单调树上倍增，$O(N \log^2 N)$，可过。

```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;

struct node {
  int id, t;
  
  bool operator >(const node &b) const {
    return t > b.t;
  }
};

int n, k, arr[100010], fa[100010][22], fw[100010][22];
vector<int> v;

long long check(int x) {
  long long rs = 0;
  for(int i = 1; i <= n; i++) {
    int c = 0, t = i;
    for(int j = 20; j >= 0; j--) {
      if(c + fw[t][j] <= x) {
        c += fw[t][j];
        t = fa[t][j];
      }
    }
    rs += arr[t];
  }
  return rs + k * x;
}

long long lmrm() {
  int l = 0, r = n + 2;
  long long ans = LLONG_MAX;
  for(; l <= r; ) {
    int len = (r - l) / 3;
    int lmid = l + len, rmid = r - len;
    if(check(lmid) <= check(rmid)) {
      ans = min(ans, check(lmid));
      r = rmid - 1;
    }else {
      ans = min(ans, check(rmid));
      l = lmid + 1;
    }
  }
  return ans;
}

signed main() {
  freopen("collect.in", "r", stdin);
  freopen("collect.out", "w", stdout);
  cin >> n >> k;
  for(int i = 1; i <= n; i++) {
    cin >> arr[i];
  }
  rotate(arr + 1, min_element(arr + 1, arr + n + 1), arr + n + 1);
  for(int i = n; i >= 1; i--) {
    fw[i][0] = 100000000;
    for(; v.size() && arr[v.back()] > arr[i]; v.pop_back()) {
      fw[v.back()][0] = v.back() - i;
      fa[v.back()][0] = i;
    }
    v.push_back(i);
  }
  for(int i = 1; i <= n; i++) {
    for(int j = 1; j <= 20; j++) {
      fa[i][j] = fa[fa[i][j - 1]][j - 1];
      fw[i][j] = fw[i][j - 1] + fw[fa[i][j - 1]][j - 1];
    }
  }
  cout << lmrm() << '\n';
  return 0;
}
```

# B 美丽子区间

容斥。

首先显然可以建出两个笛卡尔树处理掉最小/大值出现在了开头或结尾的情况。

如何处理最大值和最小值一起出现的情况？

建出两个 $(\text{Left}, >), (\text{Right}, >)$ 的单调树，然后在搜最小值时，通过倍增减掉这种算重的情况。

```cpp
#include <bits/stdc++.h>
#define int long long
using namespace std;

struct node {
  int lc, rc;
}ctl[200020], ctg[200020];

int n, arr[200020], fl[200020][22], fr[200020][22], rcl, rcg;
long long ans;
vector<int> v;

// 5.1 Minimum, except in both
void TVl(int x, int l, int r) {
  if(!x) {
    return ;
  }
  ans -= (r - l);
  int cl = x, cr = x;
  for(int i = 20; i >= 0; i--) {
    if(fl[cl][i] >= l) {
      cl = fl[cl][i];
      ans += (1 << i);
    }
    if(fr[cr][i] <= r) {
      cr = fr[cr][i];
      ans += (1 << i);
    }
  }
  TVl(ctl[x].lc, l, x - 1);
  TVl(ctl[x].rc, x + 1, r);
}

// 5.2 Maximum
void TVg(int x, int l, int r) {
  if(!x) {
    return ;
  }
  ans -= (r - l);
  TVg(ctg[x].lc, l, x - 1);
  TVg(ctg[x].rc, x + 1, r);
}

signed main() {
  freopen("interval.in", "r", stdin);
  freopen("interval.out", "w", stdout);
  cin >> n;
  for(int i = 1; i <= n; i++) {
    cin >> arr[i];
  }
  
  // 1. Cartesian, <
  for(int i = 1; i <= n; i++) {
    if(!v.size()) {
      v.push_back(i);
      continue;
    }
    for(; v.size() > 1 && arr[v.back()] > arr[i]; v.pop_back()) {
      ctl[i].lc = v.back();
    }
    if(arr[v.back()] > arr[i]) {
      ctl[i].lc = v.back();
      v.pop_back();
    }else {
      ctl[v.back()].rc = i;
    }
    v.push_back(i);
  }
  rcl = v[0];
  v.clear();
  
  // 2. Cartesian, >
  for(int i = 1; i <= n; i++) {
    if(!v.size()) {
      v.push_back(i);
      continue;
    }
    for(; v.size() > 1 && arr[v.back()] < arr[i]; v.pop_back()) {
      ctg[i].lc = v.back();
    }
    if(arr[v.back()] < arr[i]) {
      ctg[i].lc = v.back();
      v.pop_back();
    }else {
      ctg[v.back()].rc = i;
    }
    v.push_back(i);
  }
  rcg = v[0];
  v.clear();
  
  // 3. Left-first, >
  for(int i = n; i >= 1; i--) {
    for(; v.size() && arr[v.back()] < arr[i]; v.pop_back()) {
      fl[v.back()][0] = i;
    }
    v.push_back(i);
  }
  v.clear();
  for(int i = 1; i <= n; i++) {
    for(int j = 1; j <= 20; j++) {
      fl[i][j] = fl[fl[i][j - 1]][j - 1];
    }
  }
  /*
  for(int i = 1; i <= n; i++) {
    cout << fl[i][0] << ' ';
  }
  cout << '\n';
  //*/
  
  // 4. Right-first, >
  for(int i = 0; i <= 20; i++) {
    fr[n + 1][i] = n + 1;
  }
  for(int i = 1; i <= n; i++) {
    for(; v.size() && arr[v.back()] < arr[i]; v.pop_back()) {
      fr[v.back()][0] = i;
    }
    v.push_back(i);
  }
  for(int i = n; i >= 1; i--) {
    if(!fr[i][0]) {
      fr[i][0] = n + 1;
    }
    for(int j = 1; j <= 20; j++) {
      fr[i][j] = fr[fr[i][j - 1]][j - 1];
    }
  }
  /*
  for(int i = 1; i <= n; i++) {
    cout << fr[i][0] << ' ';
  }
  cout << '\n';
  //*/
  
  // 5. Cal ans
  ans = 1ll * n * (n - 1) / 2;
  TVl(rcl, 1, n);
  TVg(rcg, 1, n);
  cout << ans << '\n';
  return 0;
}

//Cartesian (<, >), Left-first with fa[200020][20], Right-first with fa[200020][20]
```

In [[Sol]], linked to [[1010]].