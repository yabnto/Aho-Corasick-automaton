# AC 自动坤
## 啊不是，是 AC 自动机
### 引入
trie 都知道吧，不知道也没关系啊，出门左转自己去搜（~~说实话，这东西和 trie 有个毛关系啊~~），然后我们知道 trie 是用来将模式串与文本串匹配然后匹最长前缀或后缀，而不能看文本串里面有没有，而 KMP 则是用来将模式串与文本串匹配出文本串中有没有模式串（不会没关系，出门左转），但是，KMP 只能单个单个的算，效率太低，所以我们要使用一种新的东西来计算。

假如说现在有一堆模式串，和一个文本串：

```
模式串：
aaa
aaaabbb
abac
文本串：
aaaabbbabac
```

我们用 trie 搜到 aaaabbb 是 aaaabbbabac 的前缀时，由于我们要的是一堆模式串在文本串里匹配，所以此时的 trie 就需要重新跑回根然后重搜，这时间消耗太大了，于是我们思考，是否可以通过一个更快捷的办法，跑到另一个可以匹配的位置，或者说是一个可匹配的位置的离终点最近的位置呢，由于我们不可能随机去找，所以必然得与已经搜了的有点关联，所以我们可以取其它串与之的最长前缀，这样就可以快速枚举与这个串有前缀的所有模式串，并保证是可以搜到可以匹配的位置的节点（哇好难理解啊，不过学过 KMP 的就用 KMP 来理解吧），那么我们如何维护最长前缀呢，首先我们可以定义一下 fail 为最长前缀。

哦对了，我们需要以 trie 为基础，然后实现 fail 等操作

考虑递推，而且由于 fail 不仅仅是往自己祖先找，所以得用广搜，根节点的 fail 值肯定指向根，或者说没有，然后找到所有可能的出边，设当前点为 u，下一个要处理的点为 v，那么如果 v 这条边存在，那么我肯定要处理，所以加入队列，并且我肯定存在 fail，那么我的 fail 将会是我父亲的 fail 的下一个与我的花色相同的（如果存在父亲的话，如果没有那就是 0），不过我怎么保证我的父亲的 fail 一定存在我的花色的边呢？那么这个问题就转换为当前要处理的点不存在这条边的情况，那不存在这条边，也就没有 fail，可是根据上面的如果这个的父亲没有这条出边，就需要继续往 fail 找，时间复杂度又巨大了（啊不是，就是正常来说，我们需要继续找父亲的 fail，所以我这里记忆化掉了，直接指向），所以我们考虑记忆化搜索，直接将这个出边指向父亲的 fail 的这条出边（同理前提是有父亲），那么这段代码如下（好好理解，这是终点，没写错，就是终点！）：

```cpp
bool Record(int &x, int c, int fa) {
  return ((~x) ? (q[++r] = x, fail[x]) : (x)) = fa ? nxt[fail[fa]][c] : 0;
}
```

x 为这条出边的指向，c 为指向的花色，u 为指向的源头也就是 x 的父亲

那么这段代码是说，如果这条出边存在，那么处理 x 并更新 fail，并将 fail 指向父亲的 fail 的同花色的边，如果这条出边不存在，那么将这条出边指向父亲的 fail 的同花色的边。

那么其他代码也非常容易，学过 trie 的都能理解（bushi，直接上代码吧：

```cpp
struct Tree {
  int nxt[MaxN][27], fail[MaxN], q[MaxN], h[MaxN], cnt[MaxN], l, r, tot;

  Tree() {
    fill(nxt[0], nxt[MaxN], -1), fill(fail, fail + MaxN, 0), fill(cnt, cnt + MaxN, 0), tot = 0;
  }

  void insert(string s, int id) {
    int p = 0;
    for (int i = 0; i < s.size(); i++) {
      int c = s[i] - 'a';
      (~nxt[p][c]) || (nxt[p][c] = ++tot);
      p = nxt[p][c];
    }
    h[id] = p, cnt[p]++;
  }

  bool Record(int &x, int c, int fa) {
    return ((~x) ? (q[++r] = x, fail[x]) : (x)) = fa ? nxt[fail[fa]][c] : 0;
  }

  void get_fail() {
    r = 0, l = r + 1;
    for (int i = 0; i < 26; i++) {
      (~nxt[0][i]) ? (Record(nxt[0][i], i, 0)) : (nxt[0][i] = 0);
    }
    while (l <= r) {
      int u = q[l++];
      for (int i = 0; i < 26; i++) {
        Record(nxt[u][i], i, u);
      }
    }
  }

  void query(string x) {
    long long res[MaxN] = {0}, p = 0;
    for (int i = 0; i < x.size(); i++) {
      int c = x[i] - 'a';
      p = nxt[p][c], res[p]++;
    }
    for (int i = r; i >= 1; i--) {
      res[fail[q[i]]] += res[q[i]];
    }
    for (int i = 1; i <= n; i++) {
      cout << res[h[i]] << '\n';
    }
  }
} st;
```

啊，多么奇怪高级而又丑陋的代码啊！
