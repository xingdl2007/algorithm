# 第八章 动态规划

> 思想，就像幽灵一样…………在它自己解释自己之前，必须先告诉它些什么。  ——查尔斯●狄更斯

## 习题8.1

**10. 有向无环图的最长路径 **

a.设计一个高效的算法找出有向无环图中最长路径的长度。（这个问题无论是作为其他动态规划应用的原型，或者就其自身而言都是很重要的。当一个项目由若干具有前驱依赖的认为组成时，它决定了完成该项目所需的最少时间。）

b.将币值最大化问题变换为寻找有向五环图的最长路径问题。

_**答：**暂时先跳过，用不同方法解决同一个问题的示例_

**11. 最大子方阵 **对一个$$m×n$$布尔矩阵$$B$$，找出其元素均为$$0$$的最大子方阵。设计一个动态规划算法并给出时间效率。（该算法可能会用于在计算机屏幕中找寻最大空白方形区域或者是选择建筑地点。）

_**答：**参见 _[leetcode 221. Maximal Square](https://leetcode.com/problems/maximal-square/description/)。_方法一定义一个类型，记录当前位置上水平和垂直方向上1的个数，再跟对角方向上的比较，决定是否构成方阵，比较好理解。第二种方法的思路还是很巧妙的，源自深刻的观察，另外空间开销也较小为_$$O(n)$$，方法一则是$$O(mn)$$，不过两种方法的时间效率都是$$O(mn)$$，都需要访问矩阵所有的元素。

```go
func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}

type info struct {
    i, j  int
    valid int
}

// verbose version, not very elegant, but the method is easy to understand
// Runtime: 6 ms, 15%
func maximalSquare(matrix [][]byte) int {
	if len(matrix) == 0 {
		return 0
	}
	row, col := len(matrix), len(matrix[0])
	if col == 0 {
		return 0
	}

	var area = 0
	infos := make([][]info, row+1)
	for i := 0; i <= row; i++ {
		infos[i] = make([]info, col+1)
	}
	// the rest of matrix
	for i := 1; i <= row; i++ {
		for j := 1; j <= col; j++ {
			if matrix[i-1][j-1] == '1' {
				infos[i][j].i = 1 + infos[i][j-1].i
				infos[i][j].j = 1 + infos[i-1][j].j
				infos[i][j].valid = 1

				tmp := min(infos[i][j].i, infos[i][j].j)
				if infos[i-1][j-1].valid > 0 {
					if tmp >= infos[i-1][j-1].valid+1 {
						infos[i][j].valid = infos[i-1][j-1].valid + 1
					} else {
						infos[i][j].valid = tmp
					}
				}
				tmp = infos[i][j].valid * infos[i][j].valid
				if area < tmp {
					area = tmp
				}
			}
		}
	}
	return area
}

// clever
func maximalSquare2(matrix [][]byte) int {
	if len(matrix) == 0 {
		return 0
	}
	row, col := len(matrix), len(matrix[0])
	if col == 0 {
		return 0
	}

	var side, prev = 0, 0
	dp := make([]int, col+1)

	for i := 1; i <= row; i++ {
		for j := 1; j <= col; j++ {
			tmp := dp[j]
			if matrix[i-1][j-1] == '1' {
				dp[j] = min(min(dp[j-1], dp[j]), prev) + 1
				side = max(side, dp[j])
			} else {
				dp[j] = 0
			}
			prev = tmp
		}
	}
	return side * side
}


// better solution:
public class Solution {
    public int maximalSquare(char[][] matrix) {
        int rows = matrix.length, cols = rows > 0 ? matrix[0].length : 0;
        int[][] dp = new int[rows + 1][cols + 1];
        int maxsqlen = 0;
        for (int i = 1; i <= rows; i++) {
            for (int j = 1; j <= cols; j++) {
                if (matrix[i-1][j-1] == '1'){
                    dp[i][j] = Math.min(Math.min(dp[i][j - 1], dp[i - 1][j]), dp[i - 1][j - 1]) + 1;
                    maxsqlen = Math.max(maxsqlen, dp[i][j]);
                }
            }
        }
        return maxsqlen * maxsqlen;
    }
}

// too clever
public class Solution {
    public int maximalSquare(char[][] matrix) {
        int rows = matrix.length, cols = rows > 0 ? matrix[0].length : 0;
        int[] dp = new int[cols + 1];
        int maxsqlen = 0, prev = 0;
        for (int i = 1; i <= rows; i++) {
            for (int j = 1; j <= cols; j++) {
                int temp = dp[j];
                if (matrix[i - 1][j - 1] == '1') {
                    dp[j] = Math.min(Math.min(dp[j - 1], prev), dp[j]) + 1;
                    maxsqlen = Math.max(maxsqlen, dp[j]);
                } else {
                    dp[j] = 0;
                }
                prev = temp;
            }
        }
        return maxsqlen * maxsqlen;
    }
}
```

**12. 世界大赛的胜率 **考虑$$A$$和$$B$$两支队伍，正在进行一系列比赛，直到一个队赢得了$$n$$场比赛为止。假设$$A$$队赢得每场比赛的概率都是相同的，即等于$$p$$，而A队丢掉比赛的概率是$$q=1-p$$（因此，就不存在平局了）。当$$A$$队还需要$$i$$场胜利才能赢得系列赛，$$B$$队还需要$$j$$场胜利才能赢得系列赛时，$$A$$队赢得系列赛的概率$$P(i,j)$$。

a. $$为P(i,j$$\)建立一个可以在动态规划算法中使用的递推关系。

b. 如果$$A$$队赢得一场比赛的概率是0.4，该队赢得一个7场系列赛的概率是多少？

c.为解决该问题写一个动态规划算法的伪代码，并确定它的时间和空间效率。

_**答： **递推关系的解释：这场赢的概率乘以还需要赢_$$i-1$$场_的概率，加上这场输乘以之后赢_$$i$$_场的概率。_

_a. _$$P(i,j) = q × P(i-1,j) + p×P(i,j-1)$$,  $$P(i,0) = 0, P(0,j) = 1$$

b. $$P(4,4) = 0.31744$$

c. _代码如下：_

```go
func cal(n int) float64 {
    row, col := (n+1)/2, (n+1)/2
    var pro = make([][]float64, row)
    for i := 0; i < row; i++ {
        pro[i] = make([]float64, col)
        pro[i][0] = 1.0
    }

    for i := 1; i < row; i++ {
        for j := 1; j < row; j++ {
            pro[i][j] = 0.6*pro[i-1][j] + 0.4*pro[i][j-1]
        }
    }
    return pro[row-1][col-1]
}
```

## 重要结论

* **动态规划**方法是一种对具有交叠子问题的问题进行求解的技术。一般来说，这样的子问题出现在求解给定问题的递推关系中，这个递推关系中包含了相同类型的更小子问题的解。动态规划法建议，与其对交叠的子问题一次又一次地求解，还不如对每个较小的子问题只解一次并把结果记录在表中，这样就可以从表中得出原始问题的解。
* 对一个最优问题应用动态规划方法要求该问题满足**最优化法则：**一个最优问题的任何实例的最优解是由该实例的子实例的最优解组成的。



