# 第三章 蛮力法

> 就像宝剑不是撬棍一样，科学也很少使用蛮力。——爱德华●莱顿
>
> 把事情做“好”常常是浪费时间。——罗伯特●伯恩

## 习题3.1

**14. 交替放置的碟子 **我们有数量为$$2n$$的一排碟子，$$n$$黑$$n$$白交替放置：黑、白、黑、……现在要把黑蝶子都放在右边，白蝶子都放在左边，但只允许通过互换相邻的碟子的位置来实现。为该谜题写个算法，并确定该算法需要执行的换位次数。

**答：**_问题并不复杂，由于只允许互换相邻的碟子，可以从最厚一个黑蝶子开始，它仅需要换1次，导数第二个需要换2次（它后面有两个白蝶子）以此类推，导数第_$$n$$_个黑蝶子需要换_$$n$$_次，所以需要执行的换位总次数是_$$n(n+1)/2$$_。_

## 习题3.4

**9. 八皇后问题 **这是一个经典游戏：在一个8x8的棋盘上，摆放8个皇后使得任意的两个皇后不在同一行或者同一列或者同一个对角线上。那么对下列各种情形，各有多少种不同的摆放方法？

**a. **任意两个皇后不在同一块方格上

**b. **任意两个皇后不在同一个行上

**c. **任意两个皇后不在同一行或者同一列上

同时估计在没秒能检查100亿个位置的计算机上，穷举查找方针针对上述各种情形找到问题的所有解需要多少时间？

**答：**_**a.** _$$C_{64}^8$$_种; **b.** _$$8^8$$_种; _**c.** $$8!$$_种_

_原始的“不在同一行或者同一列或者同一个对角线上”的摆法，一共有**92**种，并且很容易扩展到N皇后，另外isValid2也验证了，如果不需要对角的限制，则有_$$N!$$_种摆法。_

```go
// invalid: false
// O(n): i=> row, res[i]=> column
func isValid(res []int, k int) bool {
    for i := 0; i < k; i++ {
        if res[i] == res[k] ||
            i+res[i] == k+res[k] ||
            i-res[i] == k-res[k] {
            return false
        }
    }
    return true
}

// if only restrict row/column, then will have N! solution
func isValid2(res []int, k int) bool {
    for i := 0; i < k; i++ {
        if res[i] == res[k] {
            return false
        }
    }
    return true
}

// 8-queens problems
// space: O(n)
func eightQueens() (count int) {
    const N = 8
    res := [...]int{-1, -1, -1, -1, -1, -1, -1, -1}

    for k := 0; k >= 0; k-- {
        // find one solution
        for k != N && k >= 0 {
            res[k]++
            // subtle: backtrace
            if res[k] == N {
                res[k] = -1
                k--
                continue
            }
            if isValid(res[:], k) {
                k++
            }
        }
        // counter
        if k == N {
            count++
            fmt.Println(res[:])
        }
    }
    return
}
```

**10. 幻方问题 **$$n$$阶幻方是把1到$$n^2$$的整数填入一个$$n$$阶方阵，每个整数只出现一次，使得每一行、每一列、每一条主对角线上各数之和都相等。

**a.** 证明：如果$$n$$阶幻方存在的话，所讨论的这个和一定等于$$n(n^2+1)/2$$。

**b.** 设计一个穷举算法，生成阶数为$$n$$的所有幻方。

**c.** 在internet或者图书馆查找一个更好的生成幻方的算法。

**d.** 实现这两个算法——穷举查找算法以及在internet上找到的算法，然后在自己的计算机上做一个实验，确定在一分钟之内，这两个算法能够求出的幻方的最大阶数。

**答：**a. 和为$$1+2+3+……+n^2$$，显然为$$n(n^2+1)/2$$。

b. 生成$$n^2$$个数的全排列，然后一个一个的检查

```go
// recursive version
func permute(nums []int) [][]int {
    var ret [][]int
    size := len(nums)
    if size == 1 {
        ret = append(ret, nums)
    } else {
        p := permute(nums[:size-1])
        for i := 0; i < len(p); i++ {
            for j := 0; j < size; j++ {
                // must be new each iteration
                var tmp []int
                tmp = append(tmp, p[i][:j]...)
                tmp = append(tmp, nums[size-1])
                tmp = append(tmp, p[i][j:]...)

                ret = append(ret, tmp)
            }
        }
    }
    return ret
}

func judge(nums []int, n int) bool {
    var sum int

    for i := 0; i < n; i++ {
        sum += nums[i]
    }

    // row
    for i := 1; i < n; i++ {
        var tmp int
        for j := 0; j < n; j++ {
            tmp += nums[i*n+j]
        }
        if tmp != sum {
            return false
        }
    }

    // column
    for i := 0; i < n; i++ {
        var tmp int
        for j := 0; j < n; j++ {
            tmp += nums[i+j*n]
        }
        if tmp != sum {
            return false
        }
    }

    // diagonal
    var tmp int
    for i := 0; i < n; i++ {
        tmp += nums[i+i*n]
    }
    if tmp != sum {
        return false
    }

    tmp = 0
    for i := 0; i < n; i++ {
        tmp += nums[n-1-i+i*n]
    }
    if tmp != sum {
        return false
    }

    return true
}

func magicSquare(n int) (count int) {
    defer trace("magicSquare")()
    var arr = make([]int, n*n)
    for i := 0; i < n*n; i++ {
        arr[i] = i + 1
    }
    perms := permute(arr)

    for _, p := range perms {
        if judge(p, n) {
            fmt.Println(p)
            count++
        }
    }
    return
}
```

c. 参考wiki [_https://en.wikipedia.org/wiki/Magic\_square_](https://en.wikipedia.org/wiki/Magic_square)

d. 穷举法效率低下，$$O(n^2!)$$复杂度，基本上一分钟之内只能算3阶。

**11. 字母算术** 有一种称为密码算术（cryptarithm）的算式谜题，它的算式（例如加法算式）中，所有的数字都被字母所代替。如果该算式中的单词是有意义的，那么这种算式被称为字母算术题（alphametic）。最著名的字母算术题是由大名鼎鼎的英国谜题大师亨利●E.杜德尼（Henry E.Dudeney，1857-1930）给出的：


$$
SEND+MORE = MONEY
$$


这里有两个前提假设：第一，字母和十进制之间是一一对应关系，也就是说，每个字母只代表一个数字，而且不同的字母代表不同的数字；第二，数字0不出现在任何数的最左边。求解一个字母算术以为着找到每个字母代表的是哪个数字。请注意，解可能并不是唯一的，不同人的解可能并不相同。

a. 写一个程序用穷举查找解密码算术谜题。假设给定的算式是两个单词的加法算式。

b. 杜德尼的谜题发表于1924年，请用你认为合理的方法解该谜题。

**答：先跳过**

## 习题3.5

**8. 二分图 **如果图中的顶点可以分为两个不相交的子集X和Y，使得每条连接X中顶点的边都连接着Y中的顶点，这样的图是二分（bipartite）图；也可以这样认为：如果只用两种颜色对顶点进行z着色，就能使得每一条边上的两个顶点是不同的颜色，这样的图是二分d额，也称为二色（2-colorable）图。

a. 设计一个基于DFS的算法来检查一个图是否是二分图。

b. 设计一个基于BFS的算法来检查一个图是否是二分图。

**答：**检测方法跟检测无向图中是否有环很像，思路是类似的。

```go
// a. DFS
func bipartiteDFS(graph [][]int) bool {
    var m = make(map[int]bool)
    var color = make([]bool, len(graph))
    isBipartite := true

    // DFS
    var dfs func(n int)
    dfs = func(n int) {
        m[n] = true
        for _, a := range graph[n] {
            if !m[a] {
                color[a] = !color[n]
                dfs(a)
            } else {
                if color[a] == color[n] {
                    isBipartite = false
                }
            }
        }
    }

    // scan all vertex
    for i := 0; i < len(graph); i++ {
        dfs(i)
    }

    return isBipartite
}


// b. BFS
func bipartiteBFS(graph [][]int) bool {
    var m = make(map[int]bool)
    var color = make([]bool, len(graph))
    isBipartite := true

    var scanBuf []int

    // BFS
    var bfs func(n int)
    bfs = func(n int) {
        scanBuf = append(scanBuf, n)
        for len(scanBuf) > 0 {
            n := scanBuf[0]
            if !m[n] {
                m[n] = true
                for _, v := range graph[n] {
                    if !m[v] {
                        scanBuf = append(scanBuf, v)
                        color[v] = !color[n]
                    } else {
                        if color[v] == color[n] {
                            isBipartite = false
                        }
                    }
                }
            }
            scanBuf = scanBuf[1:]
        }
    }

    // scan all vertex
    for i := 0; i < len(graph); i++ {
        if !m[i] {
            bfs(i)
        }
    }

    return isBipartite
}
```

**9. **设计一个程序，对于一个给定的图，它能够输出

a. 每一个连通分量

b. 图的回路，或者返回一个消息表明图是无环的。

**答：**

```go
// a
// ref: Algorithms 4ed (Robert Sedgewick, p.350)
func CC(graph [][]int) (ret [][]int) {
    m := make(map[int]bool)
    id := make([]int, len(graph))
    var counter int

    var dfs func(n int)
    dfs = func(n int) {
        m[n] = true
        id[n] = counter
        for _, v := range graph[n] {
            if !m[v] {
                dfs(v)
            }
        }
    }

    for i := 0; i < len(graph); i++ {
        if !m[i] {
            dfs(i)
            counter++
        }
    }

    ret = make([][]int, counter)
    for n, v := range id {
        ret[v] = append(ret[v], n)
    }

    return ret
}

// b
func loopDetect(graph [][]int) (has bool) {
    m := make(map[int]bool)

    var dfs func(n, f int)
    dfs = func(n, f int) {
        m[n] = true
        for _, v := range graph[n] {
            if !m[v] {
                dfs(v, n)
            } else {
                if v != f {
                    has = true
                }
            }
        }
    }
    for i := 0; i < len(graph); i++ {
        if !m[i] {
            dfs(i, i)
        }
    }
    return
}
```

**10. **我们可以用一个代表起点的顶点、一个代表终点的顶点、若干个代表死胡同和通道的顶点来对迷宫建模，迷宫中的通道不止一条，我们必须求出连接起点和终点的迷宫道路。

a. 为下面的迷宫构造一个图

b. 如果你发现自己身处一个迷宫中，你会选用DFS遍历还是BFS遍历？为什么？

**答：**DFS倾向于快速的出口，因为每次搜索是距离起点更远的节点，会更快的找到出口。

**11. 三壶问题 **西蒙●丹尼斯●泊松（Simeon Denis Poisson, 1781-1840）是著名的法国数学家和物理学家。据说他遇到某个古老的谜题之后，就开始对数学感兴趣了，这个谜题是这样的：给定一个装满水的8品脱壶以及两个分别为5品脱和3品脱的空壶，如何通过完全灌满或者倒空这些壶从而使的某个壶精确的装有4品脱的水？用广度优先查找来解这个谜题。

**答：**先用8品脱的壶，把5品脱的空壶灌满，然后用5品脱的壶把3品脱的空壶灌满，然后将3品脱的壶倒到8品脱的壶中；接着将5品脱的壶剩下的2品脱倒空到3品脱的壶中。然后再将5品脱的壶灌满，再将5品脱向3品脱的壶中倒水，直到灌满3品脱的壶。这时，5品脱的壶中，将含有精确的4品脱水。

## 重要结论

* 许多重要的问题要求在一个复杂度随实例规模指数增长（或者更快）的域中，查找一个具有特定属性的元素。无论明指还是暗指，一般来说，这种问题往往涉及组合对象，例如排列、组合以及一个给定集合的子集。许多这样的问题都是最优问题：它们要求找到一个元素，能使某些期望的特性最大化或者最小化。

* 对于组合问题来说，**穷举查找（exhaustive seach）**是一种简单的蛮力方法。它要求生成问题域种的每一个元素，选出其中满族问题约束的元素，然后再找出一个期望元素。

* **旅行商问题: **可能的路径有$$(n-1)!/2$$种

* **背包问题: **可能的方案有$$2^n$$** **种，子集问题

* **分配问题：**可能的的分配方案有$$n!$$种

以上三个问题，就是所谓的**NP困难问题（NP-hard problem）**中的一些例子。对于NP困难问题，目前没有已知的效率可以用多项式来表示的算法。而且，大多数计算机科学家相信，这样的算法是不存在的，虽然这个非常重要的猜想从来没有被证实过。一些更加复杂的方法——回溯法和分支界限法使我们可以在优于指数级的效率下解决该问题（以及类似问题）的部分实例。或者也可以使用一些近似算法。

