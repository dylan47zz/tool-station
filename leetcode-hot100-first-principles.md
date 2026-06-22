# yLeetCode Hot 100：第一性原理拆解草稿

> 目标：把每类题先抽象成统一模板，再让每道例题显式套模板，从核心原则出发理解题目，最后结合题目特性给出优化解法。

## 使用方式

1. 先看每个类型的「第一性原理」和「统一模板」。
2. 做例题时，先回答模板的 5 个问题，再看代码。
3. 代码不是背诵对象，重点是状态、选择、边界、剪枝和优化来源。

## 全局方法论

所有算法题都可以先拆成四件事：

1. **对象**：题目里的基本对象是什么，数组元素、字符、节点、区间还是坐标？
2. **状态**：为了继续推理，最少需要保留哪些信息？
3. **转移 / 选择**：从当前状态能走向哪些新状态？
4. **不变量**：整个过程中必须一直成立的约束是什么？

如果能写清楚这四件事，代码通常只是把它们翻译出来。

## 信息的前向流动 (动态规划)

### 如何识别这是动态规划题

优先考虑动态规划，不是因为题目看起来“复杂”，而是因为它同时满足下面几个信号：

1. **求最优值 / 方案数 / 可行性**：例如最大、最小、多少种、能不能。
2. **当前选择会影响未来**：不能只看眼前一步，否则可能破坏后续最优。
3. **存在重复子问题**：暴力递归会反复计算同一段、同一位置、同一容量或同一状态。
4. **状态能被有限信息表示**：例如下标 `i`、金额 `amount`、容量 `j`、两个字符串前缀 `(i,j)`、树节点状态等。
5. **状态之间有依赖顺序**：可以从小到大、从左到右、从子树到父节点，或按区间长度推进。

判断公式：

> 如果题目问“最优 / 计数 / 可行”，并且暴力搜索中会多次遇到同一个子问题，就尝试定义 `dp`。

反过来，如果每一步局部最优都能被证明不会影响全局，优先考虑贪心；如果只是连续区间的约束维护，优先考虑滑动窗口；如果答案空间有单调性，可能是二分。

### 一句话本质

把指数级重复搜索压缩成有序状态表。

### 第一性原理

动态规划的第一性原理是：如果一个大问题可以被拆成会反复出现的子问题，并且子问题之间存在明确的依赖顺序，那么就不要重复递归，而是把每个子问题的答案存成状态，让信息按依赖方向流动。

### DP 解题标准流程

1. **先写暴力递归含义**：当前问题可以拆成哪些子问题？
2. **找重复子问题**：哪些参数组合会被反复计算？
3. **定义 `dp` 状态**：`dp[...]` 必须能完整表达一个子问题的答案。
4. **写信息流动方向**：当前状态从哪些前置状态来？遍历顺序必须保证前置状态已经算完。
5. **先写完整 `dp` 数组/表**：不要一上来压缩空间，先让信息流动可见。
6. **再做空间压缩**：只有当状态依赖范围很小，才用滚动变量或滚动数组。

### 通用代码骨架：先数组，后压缩

一维 DP 数组：

```python
dp = [初始值] * (n + 1)
dp[base] = base_value

for i in range(状态起点, n + 1):
    dp[i] = 从 dp[更小状态] 推出

return dp[答案位置]
```

二维 DP 表：

```python
dp = [[初始值] * (n + 1) for _ in range(m + 1)]
初始化第一行 / 第一列 / 空状态

for i in range(1, m + 1):
    for j in range(1, n + 1):
        dp[i][j] = 从左、上、左上或其他前置状态推出

return dp[m][n]
```

空间压缩只在最后做：

```python
# 当 dp[i] 只依赖 dp[i-1] 和 dp[i-2]
prev2, prev1 = base2, base1
for i in range(...):
    cur = f(prev1, prev2)
    prev2 = prev1
        prev1 = cur
return prev1
```

### 常见状态变量

- `dp[i]`：前 `i` 个元素、金额 `i`、长度 `i` 的最优值/方案数/可行性。
- `dp[i][j]`：两个维度共同决定的状态，如字符串匹配、编辑距离、背包容量、网格坐标。
- `dp[l][r]`：区间问题中，子区间 `[l,r]` 或 `(l,r)` 的最优值。
- 树形 DP 返回值：把每棵子树的多个状态作为递归返回值。

### 本类题目总览

- 70. 爬楼梯（Easy）
- 198. 打家劫舍（Medium）
- 213. 打家劫舍II（Medium）
- 337. 打家劫舍III（Medium）
- 322. 零钱兑换（Medium）
- 300. 最长递增子序列（Medium）
- 139. 单词拆分（Medium）
- 152. 乘积最大子数组（Medium）
- 416. 分割等和子集（Medium）
- 72. 编辑距离（Medium）
- 5. 最长回文子串（Medium）
- 53. 最大子数组和（Medium）
- 64. 最小路径和（Medium）
- 91. 解码方法（Medium）
- 221. 最大正方形（Medium）
- 279. 完全平方数（Medium）
- 10. 正则表达式匹配（Hard）
- 32. 最长有效括号（Hard）
- 312. 戳气球（Hard）
- 62. 不同路径（Medium）

### 70. 爬楼梯（Easy）

**题目描述**：每次可以爬1或2级台阶，求爬到第n级台阶有多少种不同方法。n为正整数。

**为什么属于这个类型**：题目问“到达第 n 阶有多少种方法”，这是计数问题；到达第 i 阶只依赖更小的第 i-1 和 i-2 阶，暴力递归会反复计算同一个 f(i)，所以适合 DP。

**从暴力思路出发**：暴力递归会写成 `f(i)=f(i-1)+f(i-2)`，但 `f(5)` 会重复调用 `f(3)`、`f(2)` 等子问题。

**发现可复用结构 / 单调性 / 选择树**：`dp[i]` 表示到达第 i 阶的方法数。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def climbStairs(self, n):
    if n <= 2:
        return n
    dp = [0] * (n + 1)
    dp[1], dp[2] = 1, 2
    for i in range(3, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]
    return dp[n]
```

**信息流动 / 窗口变化 / 指针移动过程**：

信息从小台阶流向大台阶：`dp[1]` 和 `dp[2]` 先确定，再推出 `dp[3]`，一路推出 `dp[n]`。

**优化代码**：

```python
def climbStairs(self, n):
    if n <= 2:
        return n
    a = 1  # f(1)
    b = 2  # f(2)
    for _ in range(3, n + 1):
        next_value = a + b
        a = b
        b = next_value
    return b
```

**易错点**：因为 `dp[i]` 只依赖前两个状态，不需要保留整张表，可压缩为两个变量 `a,b`。

### 198. 打家劫舍（Medium）

**题目描述**：给定一个非负整数数组表示每个房屋的金额，相邻房屋不能同时偷，求能偷到的最高金额。

**为什么属于这个类型**：题目问最大收益，且偷第 i 间会影响第 i+1 间能否偷；局部贪心不能保证全局最优，适合用 DP 保存“考虑到当前位置时的最优收益”。

**从暴力思路出发**：每个房子都有偷/不偷两种选择，暴力搜索是二叉决策树；不同路径会反复遇到“从第 i 间开始还能偷多少”的同一子问题。

**发现可复用结构 / 单调性 / 选择树**：`dp[i]` 表示考虑前 i 间房屋时能偷到的最高金额。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def rob(self, nums):
    n = len(nums)
    dp = [0] * (n + 1)
    dp[1] = nums[0]
    for i in range(2, n + 1):
        dp[i] = max(dp[i - 1], dp[i - 2] + nums[i - 1])
    return dp[n]
```

**信息流动 / 窗口变化 / 指针移动过程**：

从前往后流动：第 i 间的最优值只需要知道“不偷它”的 `dp[i-1]` 和“偷它”的 `dp[i-2]+nums[i-1]`。

**优化代码**：

```python
def rob(self, nums):
    if not nums:
        return 0
    prev_no = 0  # 前一家不偷时的最大收益
    prev_yes = 0  # 前一家偷时的最大收益
    for num in nums:
        # 当前不偷：前一家偷不偷都行，取最大
        # 当前偷：前一家必须不偷
        new_prev_no = max(prev_no, prev_yes)
        new_prev_yes = prev_no + num
        prev_no = new_prev_no
        prev_yes = new_prev_yes
    return max(prev_no, prev_yes)
```

**易错点**：只依赖 `dp[i-1]` 和 `dp[i-2]`，可用两个变量滚动。

### 213. 打家劫舍II（Medium）

**题目描述**：房屋围成一圈，相邻房屋不能同时偷，首尾也算相邻，求最高金额。

**为什么属于这个类型**：仍然是打家劫舍的最优值问题，但首尾相邻让线性 DP 多了一个环形约束。

**从暴力思路出发**：不能同时偷第一间和最后一间。暴力枚举仍有大量重复子问题，但关键是先拆掉环。

**发现可复用结构 / 单调性 / 选择树**：拆成两个线性问题：偷 `[0,n-2]` 或偷 `[1,n-1]`，每段内部使用 `dp[i]` 表示前 i 间最优收益。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def rob(self, nums):
    if not nums:
        return 0
    if len(nums) == 1:
        return nums[0]

    def rob_line(arr):
        n = len(arr)
        dp = [0] * (n + 1)
        dp[1] = arr[0]
        for i in range(2, n + 1):
            skip_current = dp[i - 1]
            rob_current = dp[i - 2] + arr[i - 1]
            dp[i] = max(skip_current, rob_current)
        return dp[n]

    skip_last = rob_line(nums[:-1])
    skip_first = rob_line(nums[1:])
    return max(skip_last, skip_first)
```

**信息流动 / 窗口变化 / 指针移动过程**：

环无法直接从左到右流动；先排除首或尾后，信息就重新变成线性前向流动。

**优化代码**：

```python
def rob(self, nums):
    if not nums:
        return 0
    if len(nums) == 1:
        return nums[0]

    def rob_line(start, end):
        prev_two = 0
        prev_one = 0

        for index in range(start, end + 1):
            skip_current = prev_one
            rob_current = prev_two + nums[index]
            current = max(skip_current, rob_current)
            prev_two = prev_one
            prev_one = current

        return prev_one

    skip_last = rob_line(0, len(nums) - 2)
    skip_first = rob_line(1, len(nums) - 1)
    return max(skip_last, skip_first)
```

**易错点**：优化点不是新的转移，而是“破环成链”：答案为 `max(rob_line(nums[:-1]), rob_line(nums[1:]))`。

### 337. 打家劫舍III（Medium）

**题目描述**：二叉树结构的房屋，相邻节点（父子）不能同时偷，求最高金额。

**为什么属于这个类型**：这是树上的最优值问题；偷当前节点会限制左右孩子，状态依赖子树结果，所以适合树形 DP。

**从暴力思路出发**：对每个节点递归考虑偷/不偷，会反复计算孙子、子树等结果。

**发现可复用结构 / 单调性 / 选择树**：每个节点返回二元组 `(not_rob, rob)`：不偷当前节点的最大收益、偷当前节点的最大收益。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def rob(self, root):
    memo = {}

    def dfs(node):
        if not node:
            return 0
        if node in memo:
            return memo[node]

        # 选择偷当前节点：左右孩子不能偷，只能从孙子节点继续
        rob_cur = node.val
        if node.left:
            rob_cur += dfs(node.left.left) + dfs(node.left.right)
        if node.right:
            rob_cur += dfs(node.right.left) + dfs(node.right.right)

        # 选择不偷当前节点：左右孩子可偷可不偷，交给递归取最优
        not_rob_cur = dfs(node.left) + dfs(node.right)

        memo[node] = max(rob_cur, not_rob_cur)
        return memo[node]

    return dfs(root)
```

**信息流动 / 窗口变化 / 指针移动过程**：

信息自底向上流动：左右子树先算出偷/不偷两种状态，父节点再合并。

**优化代码**：

```python
def rob(self, root):
    def dfs(node):
        if not node:
            return (0, 0)  # (不偷当前, 偷当前)
        left = dfs(node.left)
        right = dfs(node.right)
        # 不偷当前：子节点偷不偷都行
        not_rob = max(left) + max(right)
        # 偷当前：子节点必须不偷
        rob = node.val + left[0] + right[0]
        return (not_rob, rob)
    return max(dfs(root))
```

**易错点**：这是树形 DP 的压缩形式：不建全局数组，而把每个子树状态作为递归返回值。

### 322. 零钱兑换（Medium）

**题目描述**：给定不同面额的硬币coins和总金额amount，求凑成总金额所需的最少硬币数，每种硬币可无限使用。无法凑出返回-1。

**为什么属于这个类型**：题目问凑出金额的最少硬币数，是最优值问题；凑出金额 i 的答案依赖 i-c，很多路径会反复遇到同一个金额。

**从暴力思路出发**：把每次选硬币看作搜索树，目标从 amount 不断减少；同一个剩余金额会从不同硬币路径到达。

**发现可复用结构 / 单调性 / 选择树**：`dp[i]` 表示凑出金额 i 所需的最少硬币数。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def coinChange(self, coins, amount):
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0

    for i in range(1, amount + 1):
        for coin in coins:
            if i >= coin and dp[i - coin] != float('inf'):
                candidate = dp[i - coin] + 1
                if candidate < dp[i]:
                    dp[i] = candidate

    if dp[amount] == float('inf'):
        return -1
    return dp[amount]
```

**信息流动 / 窗口变化 / 指针移动过程**：

金额从小到大流动：先知道小金额的最优解，再用一个硬币扩展到更大金额。

**优化代码**：

```python
def coinChange(self, coins, amount):
    from collections import deque

    if amount == 0:
        return 0

    q = deque([amount])
    visited = {amount}
    steps = 0

    while q:
        steps += 1
        for _ in range(len(q)):
            cur = q.popleft()
            for coin in coins:
                nxt = cur - coin
                if nxt == 0:
                    return steps
                if nxt > 0 and nxt not in visited:
                    visited.add(nxt)
                    q.append(nxt)

    return -1
```

**易错点**：完整 DP 从小金额推到大金额；替代优化可以把金额看成图节点，每次减去一个硬币是一条边，BFS 第一次到达 0 的层数就是最少硬币数。DP 更稳定通用，BFS 在较早找到答案时可能提前结束。

### 300. 最长递增子序列（Medium）

**题目描述**：给定整数数组nums，找到最长严格递增子序列的长度。子序列不要求连续。

**为什么属于这个类型**：题目问最长长度；以每个位置结尾的 LIS 可以由前面更小元素的 LIS 推出，存在明确的前向依赖。

**从暴力思路出发**：枚举每个元素选或不选会产生指数级子序列；许多路径共享“以 j 结尾的最长递增子序列”信息。

**发现可复用结构 / 单调性 / 选择树**：`dp[i]` 表示以 `nums[i]` 结尾的最长递增子序列长度。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def lengthOfLIS(self, nums):
    n = len(nums)
    dp = [1] * n
    for i in range(n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)
```

**信息流动 / 窗口变化 / 指针移动过程**：

信息沿着“小于关系”流动：所有 `j<i 且 nums[j]<nums[i]` 都能把自己的长度传给 i。

**优化代码**：

```python
def lengthOfLIS(self, nums):
    import bisect

    tails = []
    for num in nums:
        # 找到第一个 >= num 的尾值，用 num 替换它
        # 这样同样长度的递增子序列拥有更小尾巴，更容易接上后续元素
        idx = bisect.bisect_left(tails, num)
        if idx == len(tails):
            tails.append(num)
        else:
            tails[idx] = num

    return len(tails)
```

**易错点**：基础 DP 的状态是“以每个位置结尾的 LIS 长度”；优化版换了状态含义，`tails[k]` 表示长度为 `k+1` 的递增子序列的最小可能尾值。`tails` 单调递增，所以可以二分替换，时间从 O(n²) 降到 O(nlogn)。

### 139. 单词拆分（Medium）

**题目描述**：给定字符串s和字典wordDict，判断s是否可以由字典中的单词（可重复使用）拼接而成。

**为什么属于这个类型**：题目问字符串能否被拆分，是可行性问题；前缀 s[:i] 能否拆分取决于更短前缀 s[:j] 和单词 s[j:i]。

**从暴力思路出发**：从每个位置尝试切一刀，递归判断剩余字符串；同一个起点会被多次递归访问。

**发现可复用结构 / 单调性 / 选择树**：`dp[i]` 表示前 i 个字符 `s[:i]` 是否可以由词典拼出。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def wordBreak(self, s, wordDict):
    word_set = set(wordDict)
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True

    for i in range(1, n + 1):
        for j in range(i):
            if dp[j] and s[j:i] in word_set:
                dp[i] = True
                break

    return dp[n]
```

**信息流动 / 窗口变化 / 指针移动过程**：

信息从短前缀流向长前缀：如果 `dp[j]` 为真且 `s[j:i]` 在词典中，那么 `dp[i]` 为真。

**优化代码**：

```python
def wordBreak(self, s, wordDict):
    word_set = set(wordDict)
    max_len = max(map(len, wordDict), default=0)
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True

    for i in range(1, n + 1):
        for length in range(1, min(max_len, i) + 1):
            j = i - length
            if dp[j] and s[j:i] in word_set:
                dp[i] = True
                break

    return dp[n]
```

**易错点**：基础 DP 枚举所有切分点；优化实现用词典最大单词长度限制枚举范围，避免检查不可能出现在词典中的超长子串。

### 152. 乘积最大子数组（Medium）

**题目描述**：给定整数数组nums，找出乘积最大的非空连续子数组，返回该乘积。

**为什么属于这个类型**：题目问连续子数组最大乘积，但负数会让最大变最小、最小变最大，需要 DP 同时保留两类历史信息。

**从暴力思路出发**：枚举所有子数组乘积是 O(n²)，每个右端点都重复计算大量前缀乘积。

**发现可复用结构 / 单调性 / 选择树**：`max_dp[i]` 表示以 `nums[i]` 结尾的最大乘积，`min_dp[i]` 表示以 `nums[i]` 结尾的最小乘积。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def maxProduct(self, nums):
    n = len(nums)
    max_dp = [0] * n
    min_dp = [0] * n

    max_dp[0] = nums[0]
    min_dp[0] = nums[0]
    ans = nums[0]

    for i in range(1, n):
        x = nums[i]
        max_dp[i] = max(x, max_dp[i - 1] * x, min_dp[i - 1] * x)
        min_dp[i] = min(x, max_dp[i - 1] * x, min_dp[i - 1] * x)
        ans = max(ans, max_dp[i])

    return ans
```

**信息流动 / 窗口变化 / 指针移动过程**：

从左到右流动：当前数可以自己开头，也可以乘以前一位置的最大/最小乘积。

**优化代码**：

```python
def maxProduct(self, nums):
    res = max(nums)
    cur_max = 1
    cur_min = 1
    for num in nums:
        # 负数会让最大变最小，先暂存
        tmp = cur_max
        cur_max = max(num, cur_max * num, cur_min * num)
        cur_min = min(num, tmp * num, cur_min * num)
        res = max(res, cur_max)
    return res
```

**易错点**：这里天然已经是空间压缩 DP，因为每个位置只依赖前一个位置。

### 416. 分割等和子集（Medium）

**题目描述**：给定只含正整数的非空数组，判断能否将数组分割成两个子集，使两个子集的和相等。

**为什么属于这个类型**：题目问能否分成等和子集，本质是能否选出若干数凑成 sum/2，是 0-1 背包可行性问题。

**从暴力思路出发**：每个数选或不选会产生 2^n 种子集，很多路径共享“能否凑出容量 j”的状态。

**发现可复用结构 / 单调性 / 选择树**：`dp[j]` 表示在已经处理过的数字中，是否能凑出和 j。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def canPartition(self, nums):
    total = sum(nums)
    if total % 2:
        return False
    target = total // 2
    dp = [[False] * (target + 1) for _ in range(len(nums) + 1)]
    dp[0][0] = True
    for i, num in enumerate(nums, 1):
        for j in range(target + 1):
            dp[i][j] = dp[i - 1][j]
            if j >= num:
                dp[i][j] = dp[i][j] or dp[i - 1][j - num]
    return dp[len(nums)][target]
```

**信息流动 / 窗口变化 / 指针移动过程**：

逐个数字处理；每个数字只能用一次，所以容量必须从大到小更新，避免同一轮重复使用当前数字。

**优化代码**：

```python
# 1D DP —— 从大到小更新，禁止信息自环
def canPartition(self, nums):
    total = sum(nums)
    if total % 2:
        return False
    target = total // 2
    dp = [False] * (target + 1)
    dp[0] = True
    for num in nums:
        for j in range(target, num - 1, -1):  # 从大到小，保证每个num只用一次
            dp[j] = dp[j] or dp[j - num]
    return dp[target]

# 2D DP —— 行间隔离，顺序无关，更直观
def canPartition(self, nums):
    total = sum(nums)
    if total % 2:
        return False
    target = total // 2
    n = len(nums)
    dp = [[False] * (target + 1) for _ in range(n + 1)]
    dp[0][0] = True  # 0个元素凑出和0
    for i in range(1, n + 1):
        num = nums[i - 1]
        for j in range(target + 1):  # 顺序随意，读的是上一行
            dp[i][j] = dp[i-1][j]  # 不选
            if j >= num:
                dp[i][j] = dp[i][j] or dp[i-1][j - num]  # 选
    return dp[n][target]
```

**易错点**：二维表可压缩成一维 `dp[j]`，但必须倒序遍历 j，这是 0-1 背包最重要的细节。

### 72. 编辑距离（Medium）

**题目描述**：给定两个字符串word1和word2，允许插入、删除、替换一个字符，求将word1转换成word2的最少操作数。

**为什么属于这个类型**：题目问把一个前缀变成另一个前缀的最少操作数，是典型二维最优值 DP。

**从暴力思路出发**：递归尝试插入、删除、替换三种操作会分叉，重复比较相同的 `(i,j)` 前缀状态。

**发现可复用结构 / 单调性 / 选择树**：`dp[i][j]` 表示 `word1[:i]` 转成 `word2[:j]` 的最少操作数。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def minDistance(self, word1, word2):
    m = len(word1)
    n = len(word2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    for i in range(m + 1):
        dp[i][0] = i
    for j in range(n + 1):
        dp[0][j] = j

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if word1[i - 1] == word2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1]
            else:
                dp[i][j] = min(
                    dp[i - 1][j],      # 删除 word1[i-1]
                    dp[i][j - 1],      # 插入 word2[j-1]
                    dp[i - 1][j - 1]   # 替换
                ) + 1

    return dp[m][n]
```

**信息流动 / 窗口变化 / 指针移动过程**：

二维前缀信息流动：`dp[i][j]` 来自左、上、左上三个已经计算过的状态。

**优化代码**：

```python
def minDistance(self, word1, word2):
    m = len(word1)
    n = len(word2)
    prev = list(range(n + 1))

    for i in range(1, m + 1):
        curr = [i] + [0] * n
        for j in range(1, n + 1):
            if word1[i - 1] == word2[j - 1]:
                curr[j] = prev[j - 1]
            else:
                curr[j] = min(prev[j], curr[j - 1], prev[j - 1]) + 1
        prev = curr

    return prev[n]
```

**易错点**：二维编辑距离最适合理解三种操作的信息来源；优化实现只保留上一行 `prev` 和当前行 `curr`，空间从 O(mn) 降到 O(n)。

### 5. 最长回文子串（Medium）

**题目描述**：给定字符串s，找到s中最长回文子串的长度或子串本身。s长度最大1000。

**为什么属于这个类型**：最长回文子串可以用区间 DP 判断 `s[i:j]` 是否回文；状态依赖更短的内部区间。

**从暴力思路出发**：枚举所有子串再判断回文是 O(n³)，重复检查内部字符。

**发现可复用结构 / 单调性 / 选择树**：`dp[i][j]` 表示子串 `s[i:j+1]` 是否为回文。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def longestPalindrome(self, s):
    n = len(s)
    dp = [[False] * n for _ in range(n)]
    start, max_len = 0, 1
    for length in range(1, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            if s[i] == s[j] and (length <= 2 or dp[i + 1][j - 1]):
                dp[i][j] = True
                if length > max_len:
                    start, max_len = i, length
    return s[start:start + max_len]
```

**信息流动 / 窗口变化 / 指针移动过程**：

区间长度从短到长流动：先知道长度 1、2 的回文，再推出更长区间。

**优化代码**：

```python
def longestPalindrome(self, s):
    def expand(left, right):
        while left >= 0 and right < len(s) and s[left] == s[right]:
            left -= 1
            right += 1
        return left + 1, right - 1

    start, end = 0, 0
    for i in range(len(s)):
        l1, r1 = expand(i, i)      # 奇数长度中心
        l2, r2 = expand(i, i + 1)  # 偶数长度中心
        if r1 - l1 > end - start:
            start, end = l1, r1
        if r2 - l2 > end - start:
            start, end = l2, r2

    return s[start:end + 1]
```

**易错点**：DP 表能清楚表达“长回文依赖内部短回文”；优化实现使用中心扩展，利用回文的对称性，不保存整张 `dp` 表，空间降为 O(1)。

### 53. 最大子数组和（Medium）

**题目描述**：给定整数数组nums，找到具有最大和的连续子数组，返回其和。数组至少含一个元素。

**为什么属于这个类型**：题目问连续子数组最大和；以当前位置结尾的最大和只依赖前一位置，是线性 DP。

**从暴力思路出发**：枚举所有子数组求和是 O(n²) 或 O(n³)。

**发现可复用结构 / 单调性 / 选择树**：`dp[i]` 表示必须以 `nums[i]` 结尾的最大子数组和。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def maxSubArray(self, nums):
    dp = [0] * len(nums)
    dp[0] = nums[0]
    ans = dp[0]
    for i in range(1, len(nums)):
        dp[i] = max(nums[i], dp[i - 1] + nums[i])
        ans = max(ans, dp[i])
    return ans
```

**信息流动 / 窗口变化 / 指针移动过程**：

从左到右流动：前面的正收益可以接上当前数，负收益就不如从当前数重新开始。

**优化代码**：

```python
# Kadane 算法 —— 空间压缩版，只求最大和
def maxSubArray(self, nums):
    cur = nums[0]
    global_max = nums[0]
    for num in nums[1:]:
        cur = max(num, cur + num)  # 重新开始 or 延续前段
        global_max = max(global_max, cur)
    return global_max

# 完整 1D DP —— 可还原子数组起止位置
def maxSubArray(self, nums):
    n = len(nums)
    dp = [0] * n
    dp[0] = nums[0]
    start = 0
    best_start, best_end = 0, 0
    best_val = nums[0]
    for i in range(1, n):
        if dp[i-1] < 0:            # 负收益 → 信息断点，重新开始
            dp[i] = nums[i]
            start = i
        else:                      # 正收益 → 延续前段
            dp[i] = dp[i-1] + nums[i]
        if dp[i] > best_val:       # 更新全局最优
            best_val = dp[i]
            best_start, best_end = start, i
    return best_val  # 如需区间: (best_val, best_start, best_end)
```

**易错点**：只依赖前一个 `dp`，可压缩成 Kadane 算法中的 `cur`。

### 64. 最小路径和（Medium）

**题目描述**：给定m×n非负整数矩阵grid，从左上角到右下角，每次只能右移或下移，求路径上数字之和的最小值。

**为什么属于这个类型**：题目问从左上到右下的最小路径和；每个格子只能从上或左到达，有天然的二维前向依赖。

**从暴力思路出发**：递归枚举所有路径数量指数级，许多路径共享同一格子的最小路径和。

**发现可复用结构 / 单调性 / 选择树**：`dp[i][j]` 表示到达格子 `(i,j)` 的最小路径和。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def minPathSum(self, grid):
    m = len(grid)
    n = len(grid[0])
    dp = [[0] * n for _ in range(m)]
    dp[0][0] = grid[0][0]

    for i in range(1, m):
        dp[i][0] = dp[i - 1][0] + grid[i][0]
    for j in range(1, n):
        dp[0][j] = dp[0][j - 1] + grid[0][j]

    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j]

    return dp[m - 1][n - 1]
```

**信息流动 / 窗口变化 / 指针移动过程**：

信息从左上角向右、向下流动；当前格子只接收上方和左方的信息。

**优化代码**：

```python
def minPathSum(self, grid):
    m = len(grid)
    n = len(grid[0])
    dp = [float('inf')] * n
    dp[0] = 0

    for i in range(m):
        for j in range(n):
            if j == 0:
                dp[j] = dp[j] + grid[i][j]
            else:
                dp[j] = min(dp[j], dp[j - 1]) + grid[i][j]

    return dp[-1]
```

**易错点**：二维表中的 `dp[i][j]` 只依赖上方和左方；优化实现用一维 `dp[j]` 表示当前行处理到第 j 列时的最小路径和。

### 91. 解码方法（Medium）

**题目描述**：给定仅含数字的字符串s，按照1->A, 2->B, ..., 26->Z的映射，求解码方法总数。

**为什么属于这个类型**：题目问解码方案数，是计数 DP；当前位置可以单独解码，也可能和前一位组合解码。

**从暴力思路出发**：递归每次切 1 位或 2 位会重复访问同一个下标。

**发现可复用结构 / 单调性 / 选择树**：`dp[i]` 表示前 i 个字符的解码方法数。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def numDecodings(self, s):
    n = len(s)
    dp = [0] * (n + 1)
    dp[0] = 1

    if s[0] != '0':
        dp[1] = 1
    else:
        dp[1] = 0

    for i in range(2, n + 1):
        if s[i - 1] != '0':
            dp[i] += dp[i - 1]
        two = int(s[i - 2:i])
        if 10 <= two <= 26:
            dp[i] += dp[i - 2]

    return dp[n]
```

**信息流动 / 窗口变化 / 指针移动过程**：

从左到右流动：`dp[i]` 接收一位解码的 `dp[i-1]` 和两位解码的 `dp[i-2]`。

**优化代码**：

```python
def numDecodings(self, s):
    if not s or s[0] == '0':
        return 0

    prev2 = 1  # dp[0]
    prev1 = 1  # dp[1]

    for i in range(2, len(s) + 1):
        cur = 0
        if s[i - 1] != '0':
            cur += prev1
        if 10 <= int(s[i - 2:i]) <= 26:
            cur += prev2
        prev2 = prev1
        prev1 = cur

    return prev1
```

**易错点**：`dp[i]` 只依赖 `dp[i-1]` 和 `dp[i-2]`，所以优化实现改用 `prev1/prev2` 滚动变量；易错点仍然是 `0` 不能单独解码，只能出现在 `10` 或 `20` 中。

### 221. 最大正方形（Medium）

**题目描述**：给定0-1矩阵，找到只包含1的最大正方形面积，返回其面积。

**为什么属于这个类型**：题目问最大正方形面积；以某格为右下角的正方形大小取决于左、上、左上三个相邻状态。

**从暴力思路出发**：枚举每个左上角和边长再验证会重复检查大量区域。

**发现可复用结构 / 单调性 / 选择树**：`dp[i][j]` 表示以 `(i,j)` 为右下角的最大正方形边长。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def maximalSquare(self, matrix):
    if not matrix:
        return 0
    m = len(matrix)
    n = len(matrix[0])
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    max_side = 0

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if matrix[i - 1][j - 1] == '1':
                dp[i][j] = min(
                    dp[i - 1][j],
                    dp[i][j - 1],
                    dp[i - 1][j - 1]
                ) + 1
                max_side = max(max_side, dp[i][j])

    return max_side * max_side
```

**信息流动 / 窗口变化 / 指针移动过程**：

信息从左上向右下流动，当前正方形能扩多大取决于三个方向的最小边长。

**优化代码**：

```python
def maximalSquare(self, matrix):
    if not matrix:
        return 0
    m = len(matrix)
    n = len(matrix[0])
    dp = [0] * (n + 1)
    max_side = 0

    for i in range(1, m + 1):
        prev_diag = 0
        for j in range(1, n + 1):
            temp = dp[j]
            if matrix[i - 1][j - 1] == '1':
                dp[j] = min(dp[j], dp[j - 1], prev_diag) + 1
                max_side = max(max_side, dp[j])
            else:
                dp[j] = 0
            prev_diag = temp

    return max_side * max_side
```

**易错点**：二维表最清楚展示左、上、左上三处信息汇聚；优化实现用一维数组保存上一行信息，并用 `prev_diag` 保存左上角旧值。

### 279. 完全平方数（Medium）

**题目描述**：给定正整数n，返回和为n的最少完全平方数数量。

**为什么属于这个类型**：题目问最少数量，且 `i` 可以由 `i - square` 加一个平方数得到，是完全背包最小值 DP。

**从暴力思路出发**：递归尝试减去每个平方数会反复到达同一个剩余值。

**发现可复用结构 / 单调性 / 选择树**：`dp[i]` 表示凑出 i 的最少完全平方数个数。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def numSquares(self, n):
    dp = [float('inf')] * (n + 1)
    dp[0] = 0

    for i in range(1, n + 1):
        j = 1
        while j * j <= i:
            dp[i] = min(dp[i], dp[i - j * j] + 1)
            j += 1

    return dp[n]
```

**信息流动 / 窗口变化 / 指针移动过程**：

从小数值流向大数值：先解决较小的 `i-square`，再补一个平方数。

**优化代码**：

```python
def numSquares(self, n):
    from collections import deque

    squares = []
    i = 1
    while i * i <= n:
        squares.append(i * i)
        i += 1

    q = deque([n])
    visited = {n}
    steps = 0

    while q:
        steps += 1
        for _ in range(len(q)):
            cur = q.popleft()
            for sq in squares:
                nxt = cur - sq
                if nxt == 0:
                    return steps
                if nxt < 0:
                    break
                if nxt not in visited:
                    visited.add(nxt)
                    q.append(nxt)
```

**易错点**：完整 DP 是最通用写法；优化/替代实现可以把每个数字看成图节点，每次减去一个平方数是一条边，BFS 第一次到达 0 的层数就是最少个数。

### 10. 正则表达式匹配（Hard）

**题目描述**：给定字符串s和模式p，实现支持'.'匹配任意单字符、'*'匹配前一个字符零次或多次的正则匹配，判断s是否完全匹配p。

**为什么属于这个类型**：`*` 让匹配出现分支；同一个 `(i,j)` 字符串前缀和模式前缀会被重复判断，适合二维 DP。

**从暴力思路出发**：递归处理 `*` 的零次/多次匹配会大量重复访问相同前缀状态。

**发现可复用结构 / 单调性 / 选择树**：`dp[i][j]` 表示 `s[:i]` 是否能被 `p[:j]` 完全匹配。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def isMatch(self, s, p):
    m = len(s)
    n = len(p)
    dp = [[False] * (n + 1) for _ in range(m + 1)]
    dp[0][0] = True

    for j in range(2, n + 1):
        if p[j - 1] == '*':
            dp[0][j] = dp[0][j - 2]

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if p[j - 1] == '*':
                dp[i][j] = dp[i][j - 2]
                if p[j - 2] == '.' or p[j - 2] == s[i - 1]:
                    dp[i][j] = dp[i][j] or dp[i - 1][j]
            elif p[j - 1] == '.' or p[j - 1] == s[i - 1]:
                dp[i][j] = dp[i - 1][j - 1]

    return dp[m][n]
```

**信息流动 / 窗口变化 / 指针移动过程**：

前缀从短到长流动；普通字符看左上，`*` 看跳过 `x*` 或继续消耗一个字符。

**优化代码**：

```python
def isMatch(self, s, p):
    m = len(s)
    n = len(p)
    dp = [[False] * (n + 1) for _ in range(m + 1)]
    dp[0][0] = True
    # 初始化：a*b*c*可以匹配空串
    for j in range(2, n + 1):
        if p[j-1] == '*':
            dp[0][j] = dp[0][j-2]
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if p[j-1] == '*':
                # 匹配零次 or 匹配一次以上
                dp[i][j] = dp[i][j-2] or (dp[i-1][j] and (p[j-2] == s[i-1] or p[j-2] == '.'))
            elif p[j-1] == '.' or p[j-1] == s[i-1]:
                dp[i][j] = dp[i-1][j-1]
    return dp[m][n]
```

**易错点**：二维表最清晰；可压缩但边界更容易出错。

### 32. 最长有效括号（Hard）

**题目描述**：给定只含'('和')'的字符串，找到最长有效（合法）括号子串的长度。

**为什么属于这个类型**：题目问最长合法括号子串；当前位置为右括号时，能否形成更长合法段取决于前面已知合法段长度。

**从暴力思路出发**：枚举所有子串再判断括号合法是 O(n³) 或 O(n²)。

**发现可复用结构 / 单调性 / 选择树**：`dp[i]` 表示以 `s[i]` 结尾的最长有效括号长度。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def longestValidParentheses(self, s):
    n = len(s)
    dp = [0] * n
    ans = 0

    for i in range(1, n):
        if s[i] == ')':
            if s[i - 1] == '(':
                if i >= 2:
                    previous_len = dp[i - 2]
                else:
                    previous_len = 0
                dp[i] = previous_len + 2
            else:
                pre = i - dp[i - 1] - 1
                if pre >= 0 and s[pre] == '(':
                    dp[i] = dp[i - 1] + 2
                    if pre > 0:
                        dp[i] += dp[pre - 1]
            if dp[i] > ans:
                ans = dp[i]

    return ans
```

**信息流动 / 窗口变化 / 指针移动过程**：

只在 `s[i] == )` 时更新；信息从前一个位置和匹配左括号之前的位置流来。

**优化代码**：

```python
def longestValidParentheses(self, s):
    stack = [-1]
    ans = 0

    for i, ch in enumerate(s):
        if ch == '(':
            stack.append(i)
        else:
            stack.pop()
            if not stack:
                # 当前右括号无法匹配，作为新的无效边界
                stack.append(i)
            else:
                # 当前合法段从 stack[-1] + 1 到 i
                ans = max(ans, i - stack[-1])

    return ans
```

**易错点**：完整 DP 关注“以 i 结尾的最长合法长度”；替代实现用栈保存尚未匹配的左括号下标，以及最近一个无法匹配的位置。两者信息含义不同：DP 存长度，栈存边界。

### 312. 戳气球（Hard）

**题目描述**：n个气球排成一行，戳破第i个气球获得nums[left]*nums[i]*nums[right]硬币，求最大硬币数。戳破后相邻气球改变。

**为什么属于这个类型**：这题直接按“先戳哪个”思考会很混乱，因为戳破一个气球后，它的左右邻居会变化，后续收益依赖历史顺序。关键转化是反过来想“某个区间里最后戳哪个”。最后戳的气球左右邻居已经固定为区间边界，区间左右两边也就变成互不影响的子问题。

**从暴力思路出发**：暴力枚举所有戳破顺序是阶乘级。问题难点在于：如果先戳一个气球，它会改变相邻关系；但如果在区间 `(left, right)` 里最后戳 `k`，那么 `k` 被戳时，区间内部其他气球都已经消失，`k` 的左右邻居一定是 `left` 和 `right`，收益固定为 `vals[left] * vals[k] * vals[right]`。

**发现可复用结构 / 单调性 / 选择树**：先在数组两端补虚拟气球 `1`，得到 `vals = [1] + nums + [1]`。`dp[left][right]` 表示：只戳破开区间 `(left, right)` 里的所有气球，且保留 `left` 和 `right` 两个边界气球不戳时，能获得的最大硬币数。注意 `left/right` 是边界，不属于被戳破的集合。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def maxCoins(self, nums):
    vals = [1] + nums + [1]
    n = len(vals)
    dp = [[0] * n for _ in range(n)]

    for length in range(2, n):
        for left in range(0, n - length):
            right = left + length
            for k in range(left + 1, right):
                gain = vals[left] * vals[k] * vals[right]
                dp[left][right] = max(
                    dp[left][right],
                    dp[left][k] + gain + dp[k][right]
                )

    return dp[0][n - 1]
```

**信息流动 / 窗口变化 / 指针移动过程**：

区间 DP 的信息按“区间长度”从短到长流动。要计算大区间 `(left, right)`，枚举其中最后被戳破的气球 `k`。这时 `(left, k)` 和 `(k, right)` 两个小区间已经先被戳完，收益分别是 `dp[left][k]` 和 `dp[k][right]`；最后戳 `k` 的收益是 `vals[left] * vals[k] * vals[right]`。所以大区间依赖两个更短区间。

**优化代码**：

```python
def maxCoins(self, nums):
    from functools import lru_cache

    vals = [1] + nums + [1]

    @lru_cache(None)
    def solve(left, right):
        # 开区间为空，没有气球可戳
        if right == left + 1:
            return 0

        best = 0
        for k in range(left + 1, right):
            best = max(
                best,
                solve(left, k) + vals[left] * vals[k] * vals[right] + solve(k, right)
            )
        return best

    return solve(0, len(vals) - 1)
```

**易错点**：表格版是自底向上的区间 DP；这里给出自顶向下记忆化版本，状态含义完全相同，但更贴近“先问大区间，再递归拆成小区间”的思考过程。戳气球不能简单滚动压缩，因为 `dp[left][right]` 依赖大量不同分割点的二维区间状态。

### 62. 不同路径（Medium）

**题目描述**：机器人从m×n网格左上角到右下角，每次只能右移或下移，求不同路径数。

**为什么属于这个类型**：题目问路径数量；每个格子的路径数只来自上方和左方，是二维计数 DP。

**从暴力思路出发**：递归向右/向下枚举所有路径会重复计算同一格子到终点的路径数。

**发现可复用结构 / 单调性 / 选择树**：`dp[i][j]` 表示到达 `(i,j)` 的不同路径数。

**套用统一模板**：套用 DP 模板：定义状态 → 写初始化 → 写状态转移 → 确定遍历顺序 → 找答案位置。

**基础模板代码**：

```python
def uniquePaths(self, m, n):
    dp = [[1] * n for _ in range(m)]
    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = dp[i - 1][j] + dp[i][j - 1]
    return dp[m - 1][n - 1]
```

**信息流动 / 窗口变化 / 指针移动过程**：

从左上向右下流动，路径数在网格中累加传播。

**优化代码**：

```python
def uniquePaths(self, m, n):
    dp = [1] * n
    for _ in range(1, m):
        for j in range(1, n):
            dp[j] += dp[j - 1]
    return dp[-1]
```

**易错点**：二维表中的每格只依赖上方和左方；优化实现用一维数组，`dp[j]` 原值代表上方，`dp[j-1]` 代表左方。

## 区间的增量维护 (滑动窗口)

### 如何识别这是滑动窗口题

滑动窗口适用于“连续区间 + 可增量维护”的问题。识别信号是：

1. 题目对象是连续子数组或连续子串。
2. 需要维护窗口内的频次、和、种类数、最大值或匹配情况。
3. 右端加入一个元素、左端移除一个元素后，状态可以 O(1) 或接近 O(1) 更新。
4. 目标通常是最长、最短、是否包含某种排列、所有固定长度窗口。

判断公式：

> 如果答案必须是连续区间，并且区间移动时能复用上一个区间的信息，就考虑滑动窗口。

### 一句话本质

在连续区间上维护一个随左右边界增量变化的状态。

### 第一性原理

滑动窗口的第一性原理是：连续区间从 `[left, right)` 变到相邻区间时，大部分元素不变。因此不要每次重新扫描窗口，而是只处理“新进入窗口的元素”和“离开窗口的元素”。

### 标准滑动窗口模板

```python
def slidingWindow(s: str):
    # 用合适的数据结构记录窗口中的数据，根据具体场景变通
    # 例如：记录元素出现次数用 dict / Counter
    # 例如：记录窗口元素和用 int
    # 例如：记录窗口最大值用单调队列
    window = ...

    left, right = 0, 0
    while right < len(s):
        # c 是将移入窗口的字符
        c = s[right]
        # 增大窗口
        right += 1
        # 更新窗口内数据
        window.add(c)
        ...

        # debug 输出的位置，最终提交代码不要 print，IO 可能导致超时
        # print(f"window: [{left}, {right})")

        # 判断左侧窗口是否要收缩
        while left < right and window_needs_shrink:
            # d 是将移出窗口的字符
            d = s[left]
            # 缩小窗口
            left += 1
            # 更新窗口内数据
            window.remove(d)
            ...
```

### 两种常见窗口

- **可变长度窗口**：右边扩张探索，左边在不合法或已经满足条件时收缩。常见于最长无重复、最小覆盖。
- **固定长度窗口**：窗口大小固定为 `k`，每次右扩后如果长度超过 `k` 就左缩一格。常见于异位词、排列包含。

### 本类题目总览

- 3. 无重复字符的最长子串（Medium）
- 76. 最小覆盖子串（Hard）
- 438. 找到字符串中所有字母异位词（Medium）
- 239. 滑动窗口最大值（Hard）
- 567. 字符串的排列（Medium）

### 3. 无重复字符的最长子串（Medium）

**题目描述**：给定字符串s，找出不含重复字符的最长子串的长度。

**为什么属于这个类型**：题目要求“无重复字符的最长子串”，答案必须是连续子串；窗口内需要维护字符是否重复，右扩和左缩都能增量更新。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：`window` 用哈希表/集合记录窗口内每个字符出现次数。

**套用统一模板**：套用滑窗模板：初始化 `window/left/right` → 右扩加入元素 → 更新窗口状态 → 判断是否左缩 → 移出元素 → 更新答案。

**基础模板代码**：

```python
def lengthOfLongestSubstring(self, s):
    window = {}
    left = 0
    right = 0
    max_len = 0

    while right < len(s):
        char_in = s[right]
        right += 1

        old_count = window.get(char_in, 0)
        window[char_in] = old_count + 1

        while window[char_in] > 1:
            char_out = s[left]
            left += 1
            window[char_out] -= 1

        current_len = right - left
        if current_len > max_len:
            max_len = current_len

    return max_len
```

**信息流动 / 窗口变化 / 指针移动过程**：

`window` 用哈希表/集合记录窗口内每个字符出现次数。

右指针不断把新字符加入窗口。

当某个字符次数大于 1，说明窗口不合法，左指针持续右移直到重新无重复。

窗口恢复合法后，用 `right-left` 更新最长长度。

这题使用可变长度窗口。窗口 `[left, right)` 始终尝试保持“没有重复字符”。右扩加入 `s[right]` 后，如果这个字符出现次数超过 1，说明窗口不合法；于是按照模板从左侧逐个移出字符，直到重复被消除。窗口重新合法后，再更新最大长度。这里故意不用 `last_seen` 跳跃写法，先保留标准模板，便于理解滑窗的扩张和收缩。

**优化代码**：

基础模板保证可理解；优化通常是用 `valid/diff/单调队列/跳跃 left` 降低比较或收缩成本。

**易错点**：标准模板是一格一格左缩；熟练后可以用 `last_seen` 把 `left` 直接跳到重复字符上次出现位置之后。但教学上先理解模板：右扩导致不合法，左缩恢复合法，合法后更新答案。

### 76. 最小覆盖子串（Hard）

**题目描述**：给定字符串s和t，找到s中涵盖t所有字符（含重复）的最小子串。不存在返回空串。

**为什么属于这个类型**：题目要求包含 t 所有字符的最短连续子串；窗口内字符频次可以增量维护，满足条件后再尽量收缩。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：`need` 记录 t 的需求频次，`window` 记录当前窗口频次，`valid` 表示已有多少种字符满足需求。

**套用统一模板**：套用滑窗模板：初始化 `window/left/right` → 右扩加入元素 → 更新窗口状态 → 判断是否左缩 → 移出元素 → 更新答案。

**基础模板代码**：

```python
def minWindow(self, s, t):
    from collections import Counter

    need = Counter(t)
    window = {}
    valid = 0

    left = 0
    right = 0
    best_start = 0
    best_len = float('inf')

    while right < len(s):
        char_in = s[right]
        right += 1

        if char_in in need:
            old_count = window.get(char_in, 0)
            window[char_in] = old_count + 1
            if window[char_in] == need[char_in]:
                valid += 1

        while valid == len(need):
            current_len = right - left
            if current_len < best_len:
                best_start = left
                best_len = current_len

            char_out = s[left]
            left += 1

            if char_out in need:
                if window[char_out] == need[char_out]:
                    valid -= 1
                window[char_out] -= 1

    if best_len == float('inf'):
        return ''
    return s[best_start:best_start + best_len]
```

**信息流动 / 窗口变化 / 指针移动过程**：

`need` 记录 t 的需求频次，`window` 记录当前窗口频次，`valid` 表示已有多少种字符满足需求。

右扩时，如果字符在 `need` 中，更新窗口频次；当某字符频次刚好等于需求，`valid += 1`。

当 `valid == len(need)` 时，窗口已经覆盖 t，开始左缩寻找更短答案。

在每次左缩之前更新最短窗口，因为此时窗口仍然满足覆盖条件。

这题使用可变长度窗口。右扩负责把字符纳入窗口，并更新 `window` 中的频次；当 `valid == len(need)` 时，说明窗口已经覆盖了 `t` 的所有字符，此时进入左缩循环。左缩前先更新答案，因为当前窗口仍然有效；移出左端字符后，如果某类必需字符不再满足需求，`valid` 减一，窗口重新变为无效，停止收缩并继续右扩。

**优化代码**：

基础模板保证可理解；优化通常是用 `valid/diff/单调队列/跳跃 left` 降低比较或收缩成本。

**易错点**：`valid` 统计的是“满足需求的字符种类数”，不是已匹配字符总数。更新最短答案必须发生在左缩之前，因为左缩之后窗口可能立刻失效。

### 438. 找到字符串中所有字母异位词（Medium）

**题目描述**：给定字符串s和p，找到s中所有p的字母异位词（排列相同）子串的起始索引。

**为什么属于这个类型**：题目找 s 中所有 p 的异位词，答案是固定长度连续窗口；窗口频次与 p 频次相同即可。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：`need` 是 p 的频次，`window` 是长度接近 len(p) 的当前窗口频次。

**套用统一模板**：套用滑窗模板：初始化 `window/left/right` → 右扩加入元素 → 更新窗口状态 → 判断是否左缩 → 移出元素 → 更新答案。

**基础模板代码**：

```python
def findAnagrams(self, s, p):
    from collections import Counter

    need = Counter(p)
    window = {}
    result = []

    left = 0
    right = 0
    target_len = len(p)

    while right < len(s):
        char_in = s[right]
        right += 1

        old_count = window.get(char_in, 0)
        window[char_in] = old_count + 1

        while right - left > target_len:
            char_out = s[left]
            left += 1
            window[char_out] -= 1
            if window[char_out] == 0:
                del window[char_out]

        current_len = right - left
        if current_len == target_len and window == need:
            result.append(left)

    return result
```

**信息流动 / 窗口变化 / 指针移动过程**：

`need` 是 p 的频次，`window` 是长度接近 len(p) 的当前窗口频次。

右扩加入字符并更新频次。

窗口长度超过 len(p) 时固定左缩一位。

当窗口长度等于 len(p)，且频次匹配时记录 `left`。

这题使用固定长度窗口。右扩加入一个字符后，如果窗口长度超过 `len(p)`，就从左侧移出一个字符，使窗口长度始终不超过目标长度。每当窗口长度刚好等于 `len(p)`，比较窗口频次与 `need` 是否一致，一致就记录起点。

**优化代码**：

基础模板保证可理解；优化通常是用 `valid/diff/单调队列/跳跃 left` 降低比较或收缩成本。

**易错点**：固定长度窗口不需要在“满足条件”时反复收缩，只需要保证长度等于 `len(p)`。进一步优化可维护 `valid/diff`，避免每次比较两个字典。

### 239. 滑动窗口最大值（Hard）

**题目描述**：给定整数数组nums和窗口大小k，滑动窗口从左到右，返回每个窗口中的最大值。

**为什么属于这个类型**：题目要求每个固定长度窗口的最大值；窗口连续移动，最大值需要动态维护。普通哈希/计数无法 O(1) 得到最大值，所以用单调队列。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：`deque` 存下标，并保持对应值单调递减；队头永远是当前窗口最大值。

**套用统一模板**：套用滑窗模板：初始化 `window/left/right` → 右扩加入元素 → 更新窗口状态 → 判断是否左缩 → 移出元素 → 更新答案。

**基础模板代码**：

```python
def maxSlidingWindow(self, nums, k):
    from collections import deque

    window = deque()
    result = []

    right = 0
    while right < len(nums):
        current_value = nums[right]

        while window and nums[window[-1]] <= current_value:
            window.pop()
        window.append(right)

        left_boundary = right - k + 1
        if window[0] < left_boundary:
            window.popleft()

        if right >= k - 1:
            max_index = window[0]
            max_value = nums[max_index]
            result.append(max_value)

        right += 1

    return result
```

**信息流动 / 窗口变化 / 指针移动过程**：

`deque` 存下标，并保持对应值单调递减；队头永远是当前窗口最大值。

右扩时，把队尾所有小于等于当前值的下标弹出，再加入当前下标。

如果队头下标已经滑出窗口 `i-k`，从队头弹出。

当 `i >= k-1`，队头就是当前窗口最大值。

这题也是固定长度窗口，但窗口状态不是普通频次，而是一个单调递减队列。右扩时，新元素进入窗口；为了让队头始终是最大值，要从队尾移除所有不可能再成为最大值的元素。随后如果队头已经离开窗口，就从队头移除。窗口长度达到 `k` 后，队头对应的值就是当前窗口最大值。

**优化代码**：

基础模板保证可理解；优化通常是用 `valid/diff/单调队列/跳跃 left` 降低比较或收缩成本。

**易错点**：单调队列里存下标，不存值，因为需要判断元素是否已经滑出窗口。队尾弹出的是“被当前更大元素支配”的旧元素。

### 567. 字符串的排列（Medium）

**题目描述**：给定字符串s1和s2，判断s2是否包含s1的某个排列（字母异位词）作为子串。

**为什么属于这个类型**：题目判断 s2 是否包含 s1 的排列，本质是是否存在一个长度为 len(s1) 的连续窗口，字符频次等于 s1。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：`need` 记录 s1 频次，`window` 记录 s2 当前窗口频次。

**套用统一模板**：套用滑窗模板：初始化 `window/left/right` → 右扩加入元素 → 更新窗口状态 → 判断是否左缩 → 移出元素 → 更新答案。

**基础模板代码**：

```python
def checkInclusion(self, s1, s2):
    from collections import Counter

    need = Counter(s1)
    window = {}
    left = 0
    right = 0
    target_len = len(s1)

    while right < len(s2):
        char_in = s2[right]
        right += 1

        old_count = window.get(char_in, 0)
        window[char_in] = old_count + 1

        while right - left > target_len:
            char_out = s2[left]
            left += 1
            window[char_out] -= 1
            if window[char_out] == 0:
                del window[char_out]

        current_len = right - left
        if current_len == target_len and window == need:
            return True

    return False
```

**信息流动 / 窗口变化 / 指针移动过程**：

`need` 记录 s1 频次，`window` 记录 s2 当前窗口频次。

右扩加入 s2[right]。

窗口长度超过 len(s1) 时左缩，保持固定长度。

窗口长度等于 len(s1) 且频次匹配时返回 True。

这题使用固定长度窗口。目标是判断 `s2` 中是否存在长度等于 `len(s1)` 的窗口，并且字符频次与 `s1` 完全一致。右扩加入字符，窗口超长就左缩一格；每次窗口长度达标后比较频次。

**优化代码**：

基础模板保证可理解；优化通常是用 `valid/diff/单调队列/跳跃 left` 降低比较或收缩成本。

**易错点**：排列不关心顺序，只关心字符频次。进一步优化可像最小覆盖子串一 样维护 `valid`，但模板版先保持“右扩、超长左缩、达标比较”的结构清晰。

## 对称与夹逼 (双指针)

### 如何识别这是双指针题

双指针适用于“两个位置共同决定答案，并且移动某一侧能排除一批候选”的问题。识别信号是：

1. 数组有序，或可以排序后不破坏答案含义。
2. 答案由左右边界、两个数、读写位置或快慢距离决定。
3. 根据当前和、面积、重复关系或位置关系，可以确定移动哪个指针。
4. 暴力枚举是 O(n²)，但有规则能一次排除一整片候选。

判断公式：

> 如果两个端点/两个位置决定候选答案，并且有办法证明移动一个指针不会错过最优解，就考虑双指针。

### 一句话本质

利用顺序、对称或位置关系减少枚举。

### 第一性原理

双指针的第一性原理是：当答案由两个位置或一段边界决定时，如果移动某个指针能确定排除一批不可能答案，就不必枚举所有组合。

### 统一模板：先问 5 个问题

1. 两个指针分别代表什么边界或角色？
2. 移动哪个指针会丢弃一批不可能答案？
3. 移动依据来自有序性、面积瓶颈、还是读写分离？
4. 是否需要跳过重复元素？
5. 答案是在移动前更新还是移动后更新？

### 通用代码骨架

```python
left = 0
    right = len(nums) - 1
while left < right:
    根据 nums[left], nums[right] 计算当前候选
    更新答案
    if 应该移动 left:
        left += 1
    else:
        right -= 1
```

### 常见状态变量

- 相向指针：两端夹逼，如两数之和、盛水容器
- 快慢指针：读写分离，如移除元素、移动零
- 多指针：固定一个数后，剩余部分双指针夹逼

### 常见剪枝 / 优化

- 有序数组中和太小移左，和太大移右
- 面积题移动短板，因为长板不是当前瓶颈
- 去重题在同层跳过相同值

### 本类题目总览

- 11. 盛最多水的容器（Medium）
- 15. 三数之和（Medium）
- 42. 接雨水（Hard）
- 283. 移动零（Easy）
- 26. 删除有序数组中的重复项（Easy）
- 27. 移除元素（Easy）
- 167. 两数之和II（Medium）

### 11. 盛最多水的容器（Medium）

**题目描述**：给定非负整数数组height表示垂线高度，找两条线使其与x轴构成的容器盛水最多，返回最大面积。

**为什么属于这个类型**：题目可以用两个位置、两个边界或读写指针表达状态，并且移动某个指针能排除一批候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：面积由短板决定——移动长板不可能增大面积，只有移动短板才有可能找到更大值。

**套用统一模板**：每轮计算当前候选，再根据有序性、瓶颈或读写规则移动指针。

**基础模板代码**：

```python
def maxArea(self, height):
    left = 0
    right = len(height) - 1
    max_water = 0

    while left < right:
        left_height = height[left]
        right_height = height[right]
        width = right - left
        current_height = min(left_height, right_height)
        current_water = current_height * width

        if current_water > max_water:
            max_water = current_water

        if left_height < right_height:
            left += 1
        else:
            right -= 1

    return max_water
```

**信息流动 / 窗口变化 / 指针移动过程**：

双指针贪心。左右指针分别位于数组两端，面积=min(height[left], height[right])*(right-left)。每次移动较矮的指针向内收缩：因为面积受限于较短边，若移动较高边，宽度减小且高度不会超过较短边，面积一定变小；只有移动较短边才有可能遇到更高的线使面积增大。时间O(n)，空间O(1)。贪心正确性：排除的方案一定不是最优。

**优化代码**：

优化重点是跳过重复、减少无效枚举、或把 O(n²) 枚举压缩为 O(n) 夹逼。

**易错点**：排序后跳过重复值；读写分离时只写需要保留的元素；夹逼题证明被移动的一侧不可能产生更优解。

### 15. 三数之和（Medium）

**题目描述**：给定整数数组nums，找出所有和为0的三元组，结果不可包含重复三元组。

**为什么属于这个类型**：题目可以用两个位置、两个边界或读写指针表达状态，并且移动某个指针能排除一批候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：排序后固定一个数，剩下两个数用双指针夹逼——有序性让枚举从O(n³)降到O(n²)。

**套用统一模板**：每轮计算当前候选，再根据有序性、瓶颈或读写规则移动指针。

**基础模板代码**：

```python
def threeSum(self, nums):
    nums.sort()
    result = []
    n = len(nums)

    for i in range(n - 2):
        if i > 0 and nums[i] == nums[i - 1]:
            continue

        left = i + 1
        right = n - 1

        while left < right:
            current_sum = nums[i] + nums[left] + nums[right]

            if current_sum == 0:
                triple = [nums[i], nums[left], nums[right]]
                result.append(triple)

                left += 1
                right -= 1

                while left < right and nums[left] == nums[left - 1]:
                    left += 1
                while left < right and nums[right] == nums[right + 1]:
                    right -= 1
            elif current_sum < 0:
                left += 1
            else:
                right -= 1

    return result
```

**信息流动 / 窗口变化 / 指针移动过程**：

排序+双指针。先排序O(nlogn)，固定第一个数nums[i]（跳过重复），在i+1到n-1范围内用左右指针找两数之和等于-nums[i]。若和偏小则左指针右移，偏大则右指针左移。找到后左右指针同时移动并跳过重复值。关键去重：外层i跳过重复，内层找到解后左右都跳过重复。时间O(n²)，空间O(1)不计排序栈。

**优化代码**：

优化重点是跳过重复、减少无效枚举、或把 O(n²) 枚举压缩为 O(n) 夹逼。

**易错点**：排序后跳过重复值；读写分离时只写需要保留的元素；夹逼题证明被移动的一侧不可能产生更优解。

### 42. 接雨水（Hard）

**题目描述**：给定非负整数数组表示柱子高度，计算下雨后能接多少雨水。

**为什么属于这个类型**：题目可以用两个位置、两个边界或读写指针表达状态，并且移动某个指针能排除一批候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：每个位置能接的雨水由其左右最高柱的较小值决定——信息从两端向中间汇聚取瓶颈。

**套用统一模板**：每轮计算当前候选，再根据有序性、瓶颈或读写规则移动指针。

**基础模板代码**：

```python
def trap(self, height):
    left = 0
    right = len(height) - 1
    left_max = 0
    right_max = 0
    total_water = 0

    while left < right:
        left_height = height[left]
        right_height = height[right]

        if left_height < right_height:
            if left_height >= left_max:
                left_max = left_height
            else:
                water_here = left_max - left_height
                total_water += water_here
            left += 1
        else:
            if right_height >= right_max:
                right_max = right_height
            else:
                water_here = right_max - right_height
                total_water += water_here
            right -= 1

    return total_water
```

**信息流动 / 窗口变化 / 指针移动过程**：

方法一：双指针。对每个位置，接水量=min(左侧最高, 右侧最高)-当前高度。用左右指针从两端向中间移动，维护left_max和right_max。若left_max<right_max，则左指针位置的接水量由left_max决定（因为右侧一定有更高的柱子），反之同理。时间O(n)，空间O(1)。方法二：单调栈，维护递减栈，遇到更高柱子时弹出栈顶计算接水量。方法三：预处理左右最大值数组，时间O(n)空间O(n)。双指针最优。

**优化代码**：

优化重点是跳过重复、减少无效枚举、或把 O(n²) 枚举压缩为 O(n) 夹逼。

**易错点**：排序后跳过重复值；读写分离时只写需要保留的元素；夹逼题证明被移动的一侧不可能产生更优解。

### 283. 移动零（Easy）

**题目描述**：给定数组nums，将所有0移到末尾，同时保持非零元素的相对顺序，必须原地操作。

**为什么属于这个类型**：题目可以用两个位置、两个边界或读写指针表达状态，并且移动某个指针能排除一批候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：同向双指针——快指针找非零元素，慢指针指向待填充位置，信息从快指针向慢指针传递。

**套用统一模板**：每轮计算当前候选，再根据有序性、瓶颈或读写规则移动指针。

**基础模板代码**：

```python
def moveZeroes(self, nums):
    slow = 0  # 下一个非零元素应放的位置
    for fast in range(len(nums)):
        if nums[fast] != 0:
            nums[slow], nums[fast] = nums[fast], nums[slow]
            slow += 1
```

**信息流动 / 窗口变化 / 指针移动过程**：

双指针。慢指针指向下一个非零元素应放置的位置，快指针遍历数组。当快指针遇到非零元素时，与慢指针位置交换（或直接赋值），然后慢指针前进一步。这样所有非零元素按原顺序紧凑排列在数组前部，剩余位置自然全为0（若赋值而非交换则需最后补零）。时间O(n)，空间O(1)。本质是稳定分区操作。

**优化代码**：

优化重点是跳过重复、减少无效枚举、或把 O(n²) 枚举压缩为 O(n) 夹逼。

**易错点**：排序后跳过重复值；读写分离时只写需要保留的元素；夹逼题证明被移动的一侧不可能产生更优解。

### 26. 删除有序数组中的重复项（Easy）

**题目描述**：给定非严格递增数组nums，原地删除重复元素使每个元素只出现一次，保持相对顺序，返回新长度。

**为什么属于这个类型**：题目可以用两个位置、两个边界或读写指针表达状态，并且移动某个指针能排除一批候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：同向双指针在有序数组上去重——快指针找新元素，慢指针写不重复序列。

**套用统一模板**：每轮计算当前候选，再根据有序性、瓶颈或读写规则移动指针。

**基础模板代码**：

```python
def removeDuplicates(self, nums):
    if not nums: return 0
    slow = 0  # 无重复部分的末尾
    for fast in range(1, len(nums)):
        if nums[fast] != nums[slow]:  # 找到新元素
            slow += 1
            nums[slow] = nums[fast]
    return slow + 1
```

**信息流动 / 窗口变化 / 指针移动过程**：

快慢指针。慢指针指向已处理区域的末尾（唯一元素的最后位置），快指针遍历数组。当nums[fast]!=nums[slow]时，先将slow+1，再将nums[slow]=nums[fast]。由于数组有序，相等元素一定相邻，只需比较相邻即可去重。时间O(n)，空间O(1)。返回slow+1为新长度。

**优化代码**：

优化重点是跳过重复、减少无效枚举、或把 O(n²) 枚举压缩为 O(n) 夹逼。

**易错点**：排序后跳过重复值；读写分离时只写需要保留的元素；夹逼题证明被移动的一侧不可能产生更优解。

### 27. 移除元素（Easy）

**题目描述**：给定数组nums和值val，原地移除所有等于val的元素，返回新长度。元素顺序可变。

**为什么属于这个类型**：题目可以用两个位置、两个边界或读写指针表达状态，并且移动某个指针能排除一批候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：同向双指针过滤——快指针找保留的元素，慢指针写入。

**套用统一模板**：每轮计算当前候选，再根据有序性、瓶颈或读写规则移动指针。

**基础模板代码**：

```python
def removeElement(self, nums, val):
    slow = 0
    for fast in range(len(nums)):
        if nums[fast] != val:
            nums[slow] = nums[fast]
            slow += 1
    return slow
```

**信息流动 / 窗口变化 / 指针移动过程**：

双指针。慢指针指向下一个不等于val的元素应放置的位置，快指针遍历数组。当nums[fast]!=val时，将其复制到nums[slow]并递增slow。遍历结束后slow即为新长度。也可用左右指针从两端交换（但会改变顺序）。时间O(n)，空间O(1)。

**优化代码**：

优化重点是跳过重复、减少无效枚举、或把 O(n²) 枚举压缩为 O(n) 夹逼。

**易错点**：排序后跳过重复值；读写分离时只写需要保留的元素；夹逼题证明被移动的一侧不可能产生更优解。

### 167. 两数之和II（Medium）

**题目描述**：给定1-indexed的非递减整数数组numbers和目标值target，找出两数之和等于target的两个下标（加1），恰好一个解。

**为什么属于这个类型**：题目可以用两个位置、两个边界或读写指针表达状态，并且移动某个指针能排除一批候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：有序数组的双指针夹逼——和太大右缩，和太小左扩，利用单调性排除无效搜索。

**套用统一模板**：每轮计算当前候选，再根据有序性、瓶颈或读写规则移动指针。

**基础模板代码**：

```python
def twoSum(self, numbers, target):
    left = 0
    right = len(numbers) - 1
    while left < right:
        s = numbers[left] + numbers[right]
        if s == target:
            return [left + 1, right + 1]  # 1-indexed
        elif s < target:
            left += 1
        else:
            right -= 1
```

**信息流动 / 窗口变化 / 指针移动过程**：

双指针。数组已排序，左右指针分别在首尾。若和偏小则左指针右移，偏大则右指针左移，直到和等于target。时间O(n)，空间O(1)。也可用哈希表O(n)时间空间但未利用有序性质。双指针是排序数组两数之和的标准做法。

**优化代码**：

优化重点是跳过重复、减少无效枚举、或把 O(n²) 枚举压缩为 O(n) 夹逼。

**易错点**：排序后跳过重复值；读写分离时只写需要保留的元素；夹逼题证明被移动的一侧不可能产生更优解。

## 单调性 (单调栈/队列)

### 如何识别这是单调栈/队列题

单调结构适用于“找附近第一个更大/更小”或“动态窗口最值”。识别信号是：

1. 题目问右边/左边第一个更大、更小、最近更高、更低。
2. 新元素会让某些旧元素永远失去成为答案的机会。
3. 需要在滑动窗口中持续得到最大值或最小值。
4. 暴力向左右扫描会重复比较大量元素。

判断公式：

> 如果一个元素的答案要等未来某个更大/更小元素来“裁决”，就考虑单调栈；如果窗口最值随边界移动，就考虑单调队列。

### 一句话本质

维护仍可能成为答案的候选集合。

### 第一性原理

单调结构的第一性原理是：新元素到来时，会让某些旧元素永远不可能再成为答案。把这些旧元素立刻弹出，栈/队列里只保留未来仍有价值的候选。

### 统一模板：先问 5 个问题

1. 要找的是左/右侧第一个更大、更小，还是窗口最值？
2. 栈/队列里存值还是下标？
3. 维护递增还是递减？
4. 新元素出现时，哪些旧元素被淘汰？
5. 被弹出的瞬间是否能确定答案？

### 通用代码骨架

```python
stack = []
for i, x in enumerate(nums):
    while stack and x 触发栈顶答案:
        j = stack.pop()
        计算 j 的答案
    stack.append(i)
```

### 常见状态变量

- 单调栈：处理最近更大/更小、矩形边界
- 单调队列：维护滑动窗口最大/最小值
- 辅助栈：每个栈元素携带历史最值

### 常见剪枝 / 优化

- 被新元素支配的旧元素直接弹出
- 窗口外下标从队头移除
- 高度相等时按题意选择是否弹出以控制边界

### 本类题目总览

- 84. 柱状图中最大的矩形（Hard）
- 85. 最大矩形（Hard）
- 155. 最小栈（Medium）
- 739. 每日温度（Medium）
- 496. 下一个更大元素I（Easy）

### 84. 柱状图中最大的矩形（Hard）

**题目描述**：给定非负整数数组heights表示柱状图高度，求柱状图中能勾勒出的最大矩形面积。

**为什么属于这个类型**：题目需要寻找附近第一个更大/更小元素，或动态维护窗口最值，新元素会淘汰一批旧候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：矩形面积=高×宽，高受最矮柱子限制——每根柱子的命运由它两侧更矮柱子的位置决定。

**套用统一模板**：栈/队列中通常存下标；while 弹出被支配元素；弹出瞬间或入栈后更新答案。

**基础模板代码**：

```python
def largestRectangleArea(self, heights):
    stack = []
    max_area = 0
    heights.append(0)

    for index, height in enumerate(heights):
        while stack and heights[stack[-1]] > height:
            top_index = stack.pop()
            rectangle_height = heights[top_index]

            if stack:
                left_boundary = stack[-1]
                width = index - left_boundary - 1
            else:
                width = index

            area = rectangle_height * width
            if area > max_area:
                max_area = area

        stack.append(index)

    heights.pop()
    return max_area
```

**信息流动 / 窗口变化 / 指针移动过程**：

单调递增栈。核心思想：每根柱子作为矩形高度时，需要找到其左右边界（左右第一个比它矮的柱子）。用栈存储下标，保持栈中对应高度递增。当当前高度<栈顶高度时，弹出栈顶，此时栈顶的右边界为当前下标，左边界为新栈顶下标+1（栈顶下面的元素就是左边第一个更矮的）。添加尾部哨兵0确保所有柱子都被处理。时间O(n)，空间O(n)。

**优化代码**：

优化来自每个元素最多入栈/出栈一次，整体从重复扫描降为 O(n)。

**易错点**：窗口题要及时移除过期下标；矩形题常加哨兵简化边界；最小栈可让每个元素携带历史最小值。

### 85. 最大矩形（Hard）

**题目描述**：给定0-1矩阵，找到只包含1的最大矩形面积。

**为什么属于这个类型**：题目需要寻找附近第一个更大/更小元素，或动态维护窗口最值，新元素会淘汰一批旧候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：逐行将二维问题转化为一维柱状图——信息从上一行向下一行流动，每行调用84题。

**套用统一模板**：栈/队列中通常存下标；while 弹出被支配元素；弹出瞬间或入栈后更新答案。

**基础模板代码**：

```python
def maximalRectangle(self, matrix):
    if not matrix:
        return 0

    cols = len(matrix[0])
    heights = [0] * cols
    max_area = 0

    def largest_rectangle(histogram):
        stack = []
        best = 0
        histogram.append(0)

        for index, height in enumerate(histogram):
            while stack and histogram[stack[-1]] > height:
                top_index = stack.pop()
                rectangle_height = histogram[top_index]

                if stack:
                    left_boundary = stack[-1]
                    width = index - left_boundary - 1
                else:
                    width = index

                area = rectangle_height * width
                if area > best:
                    best = area
            stack.append(index)

        histogram.pop()
        return best

    for row in matrix:
        for col in range(cols):
            if row[col] == '1':
                heights[col] += 1
            else:
                heights[col] = 0

        area = largest_rectangle(heights)
        if area > max_area:
            max_area = area

    return max_area
```

**信息流动 / 窗口变化 / 指针移动过程**：

在#84基础上逐行转化。对每一行，计算以该行为底的高度数组heights[j]（连续1的个数，遇0归零），然后对该高度数组用#84的单调栈方法求最大矩形。遍历所有行取最大值。时间O(m*n)，空间O(n)。关键洞察：将2D问题逐行降维为1D的柱状图最大矩形问题。

**优化代码**：

优化来自每个元素最多入栈/出栈一次，整体从重复扫描降为 O(n)。

**易错点**：窗口题要及时移除过期下标；矩形题常加哨兵简化边界；最小栈可让每个元素携带历史最小值。

### 155. 最小栈（Medium）

**题目描述**：设计一个支持push、pop、top和getMin操作的栈，所有操作均要求O(1)时间复杂度。

**为什么属于这个类型**：题目需要寻找附近第一个更大/更小元素，或动态维护窗口最值，新元素会淘汰一批旧候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：辅助栈同步跟踪最小值——每次push/pop时，最小值信息与主栈同步维护。

**套用统一模板**：栈/队列中通常存下标；while 弹出被支配元素；弹出瞬间或入栈后更新答案。

**基础模板代码**：

```python
class MinStack:
    def __init__(self):
        self.stack = []
        self.min_stack = []

    def push(self, val: int) -> None:
        self.stack.append(val)

        if self.min_stack:
            previous_min = self.min_stack[-1]
            current_min = min(val, previous_min)
        else:
            current_min = val
        self.min_stack.append(current_min)

    def pop(self) -> None:
        self.stack.pop()
        self.min_stack.pop()

    def top(self) -> int:
        return self.stack[-1]

    def getMin(self) -> int:
        return self.min_stack[-1]
```

**信息流动 / 窗口变化 / 指针移动过程**：

核心难点是getMin需O(1)。方案：维护一个辅助栈min_stack，与主栈同步push/pop，但min_stack栈顶始终保存当前主栈的最小值。push时若min_stack为空或新值<=min_stack栈顶则压入新值，否则压入当前栈顶（重复最小值）；pop时两栈同步弹出。另一种等价方案：push时min_stack只在新值<=栈顶时压入，pop时只在值等于min_stack栈顶时弹出，但需注意重复最小值的处理。时间O(1)，空间O(n)。

**优化代码**：

优化来自每个元素最多入栈/出栈一次，整体从重复扫描降为 O(n)。

**易错点**：窗口题要及时移除过期下标；矩形题常加哨兵简化边界；最小栈可让每个元素携带历史最小值。

### 739. 每日温度（Medium）

**题目描述**：给定整数数组temperatures表示每日温度，返回数组answer，answer[i]为等多少天后才有更高温度，没有则为0。

**为什么属于这个类型**：题目需要寻找附近第一个更大/更小元素，或动态维护窗口最值，新元素会淘汰一批旧候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：每个温度等待更高温度出现——单调递减栈维护"还在等待"的温度，新温度裁决栈中旧温度的命运。

**套用统一模板**：栈/队列中通常存下标；while 弹出被支配元素；弹出瞬间或入栈后更新答案。

**基础模板代码**：

```python
def dailyTemperatures(self, temperatures):
    n = len(temperatures)
    res = [0] * n
    stack = []  # 单调递减栈，存下标
    for i, t in enumerate(temperatures):
        while stack and temperatures[stack[-1]] < t:
            prev = stack.pop()
            res[prev] = i - prev  # 等待天数确定
        stack.append(i)
    return res
```

**信息流动 / 窗口变化 / 指针移动过程**：

单调递减栈。从左到右遍历，栈存储下标且对应温度递减。当当前温度>栈顶温度时，弹出栈顶，栈顶的答案=当前下标-栈顶下标。循环直到栈顶温度>=当前温度或栈空，然后将当前下标压栈。直觉：之前温度更低的日期在等待更高的温度，一旦遇到就确定了答案。每个下标最多入栈出栈各一次，时间O(n)，空间O(n)。

**优化代码**：

优化来自每个元素最多入栈/出栈一次，整体从重复扫描降为 O(n)。

**易错点**：窗口题要及时移除过期下标；矩形题常加哨兵简化边界；最小栈可让每个元素携带历史最小值。

### 496. 下一个更大元素I（Easy）

**题目描述**：给定两个无重复元素数组nums1和nums2（nums1是nums2子集），对nums1中每个元素找其在nums2中右边第一个更大元素，不存在则为-1。

**为什么属于这个类型**：题目需要寻找附近第一个更大/更小元素，或动态维护窗口最值，新元素会淘汰一批旧候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：在nums2上用单调栈预处理所有下一个更大元素，再通过哈希表查询nums1所需的结果。

**套用统一模板**：栈/队列中通常存下标；while 弹出被支配元素；弹出瞬间或入栈后更新答案。

**基础模板代码**：

```python
def nextGreaterElement(self, nums1, nums2):
    next_greater = {}
    stack = []  # 单调递减栈
    for num in nums2:
        while stack and stack[-1] < num:
            next_greater[stack.pop()] = num  # 栈顶的下一个更大元素是num
        stack.append(num)
    return [next_greater.get(n, -1) for n in nums1]
```

**信息流动 / 窗口变化 / 指针移动过程**：

单调栈+哈希表。先对nums2用单调递减栈预处理，从右向左遍历：栈中维护递减序列，弹出所有<=当前元素的栈顶，若栈非空则栈顶就是右边第一个更大元素，记录到哈希表中，当前元素入栈。然后对nums1中每个元素查哈希表即可。时间O(len(nums1)+len(nums2))，空间O(len(nums2))。

**优化代码**：

优化来自每个元素最多入栈/出栈一次，整体从重复扫描降为 O(n)。

**易错点**：窗口题要及时移除过期下标；矩形题常加哨兵简化边界；最小栈可让每个元素携带历史最小值。

## 决策树的穷举 (回溯)

### 如何识别这是回溯题

回溯适用于“需要构造所有可能答案”或“在路径中尝试选择”的问题。识别信号是：

1. 题目要求返回所有排列、组合、子集、路径或合法字符串。
2. 每一步有多个选择，选择之后还要继续选择。
3. 一条完整选择路径就是一个候选答案。
4. 选择有约束，需要走不通时撤销并换分支。
5. 输入规模通常不大，因为本质可能需要指数级枚举。

判断公式：

> 如果答案空间天然是一棵“选择树”，并且题目要枚举所有合法路径，就考虑回溯。

### 一句话本质

在选择树上 DFS，选择、深入、撤销。

### 第一性原理

回溯的第一性原理是：答案空间可以表示成一棵决策树。每个节点是当前状态，每条边是一次选择；DFS 走到底得到候选答案，再撤销选择回到父节点尝试其他分支。

### 统一模板：先问 5 个问题

1. 决策树每一层代表什么？
2. 当前状态有哪些变量？`path/start/used/index/visited/remain`？
3. 当前有哪些选择？
4. 选择是否合法，能否剪枝？
5. 何时收集答案，撤销哪些状态？

### 通用代码骨架

```python
def backtrack(state):
    if 到达结束条件:
        记录答案
        return
    for choice in 当前选择集合:
        if choice 不合法:
            continue
        做选择
        backtrack(新状态)
        撤销选择
```

### 常见状态变量

- `path`：当前构造出的路径
- `start`：组合类题固定搜索方向，避免顺序重复
- `used`：排列类题防止同一元素重复使用
- `visited`：网格/图路径防止走回头路
- `remain`：目标和剩余量

### 常见剪枝 / 优化

- 剩余目标小于 0 停止
- 长度达到上限停止
- 字符不匹配停止
- 排序后当前数过大可 `break`
- 同层重复元素跳过

### 本类题目总览

- 46. 全排列（Medium）
- 78. 子集（Medium）
- 39. 组合总和（Medium）
- 22. 括号生成（Medium）
- 17. 电话号码的字母组合（Medium）
- 79. 单词搜索（Medium）
- 51. N皇后（Hard）
- 77. 组合（Medium）
- 40. 组合总和II（Medium）
- 47. 全排列II（Medium）

### 46. 全排列（Medium）

**题目描述**：给定不含重复数字的数组nums，返回其所有可能的全排列。

**为什么属于这个类型**：题目要求枚举所有排列、组合、子集、路径或合法构造，答案空间天然是一棵选择树。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：每个位置选择一个未使用的元素——决策树的每一层是一个位置，每个分支是一个可用选择。

**套用统一模板**：状态是 `path + used`；选择是所有 `not used[i]` 的数字；结束条件是 `len(path)==n`；撤销时同时恢复 `path` 和 `used`。

**基础模板代码**：

```python
def permute(self, nums):
    res = []
    def backtrack(path, used):
        if len(path) == len(nums):
            res.append(path[:])
            return
        for i in range(len(nums)):
            if used[i]:
                continue
            used[i] = True
            path.append(nums[i])
            backtrack(path, used)
            path.pop()       # 撤销选择
            used[i] = False  # 撤销标记
    backtrack([], [False] * len(nums))
    return res
```

**信息流动 / 窗口变化 / 指针移动过程**：

直观上，`path` 就是正在填写的排列前缀。第 0 层决定第一个数，第 1 层决定第二个数。`used` 的意义不是剪枝优化，而是题目约束：同一个数字不能在一条路径中出现两次。

**优化代码**：

优化主要是剪枝、排序去重、提前返回，避免探索不可能产生答案的子树。

**易错点**：不重复数字时无需去重；如果变成全排列 II，要排序并跳过同层重复；若只求第 k 个排列，可用阶乘定位避免枚举全部。

### 78. 子集（Medium）

**题目描述**：给定不含重复元素的整数数组nums，返回其所有可能的子集（幂集），解集不可包含重复子集。

**为什么属于这个类型**：题目要求枚举所有排列、组合、子集、路径或合法构造，答案空间天然是一棵选择树。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：每个元素选或不选——决策树的每一层是一个元素，两个分支：放入子集或不放入。

**套用统一模板**：状态是 `path + start`；每个节点本身都是一个合法子集，所以进入节点就记录答案；选择是 `[start,n)` 中的元素；递归时传 `i+1`。

**基础模板代码**：

```python
def subsets(self, nums):
    res = []
    def backtrack(start, path):
        res.append(path[:])  # 每个节点都是一个有效子集
        for i in range(start, len(nums)):
            path.append(nums[i])
            backtrack(i + 1, path)  # 从i+1开始，避免重复
            path.pop()
    backtrack(0, [])
    return res
```

**信息流动 / 窗口变化 / 指针移动过程**：

子集和组合的区别是：子集不要求固定长度，所以不必等到叶子节点才收集答案；树上每个节点都对应一个子集。

**优化代码**：

优化主要是剪枝、排序去重、提前返回，避免探索不可能产生答案的子树。

**易错点**：标准写法记录所有节点；也可用二进制枚举。若数组有重复元素，需要排序并跳过同层重复。

### 39. 组合总和（Medium）

**题目描述**：给定无重复正整数数组candidates和目标数target，找出所有使数字和为target的组合，candidates中数字可无限制重复选取，解集不可包含重复组合。

**为什么属于这个类型**：题目要求枚举所有排列、组合、子集、路径或合法构造，答案空间天然是一棵选择树。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：无限制地选取元素使和为target——决策树每层选一个数，允许重复选择，通过start控制不回退。

**套用统一模板**：状态是 `path + start + remain`；选择是从 `start` 开始的候选数；`remain==0` 收集答案；允许重复使用所以递归传 `i` 而不是 `i+1`。

**基础模板代码**：

```python
def combinationSum(self, candidates, target):
    res = []
    def backtrack(start, path, remain):
        if remain == 0:
            res.append(path[:])
            return
        if remain < 0:
            return  # 剪枝
        for i in range(start, len(candidates)):
            path.append(candidates[i])
            backtrack(i, path, remain - candidates[i])  # 传i而非i+1，允许重复
            path.pop()
    backtrack(0, [], target)
    return res
```

**信息流动 / 窗口变化 / 指针移动过程**：

`start` 的作用是固定搜索方向，避免 `[2,2,3]`、`[2,3,2]`、`[3,2,2]` 被重复生成。递归传 `i` 表示当前数字还可以继续被选。

**优化代码**：

优化主要是剪枝、排序去重、提前返回，避免探索不可能产生答案的子树。

**易错点**：先排序，若 `candidates[i] > remain` 直接 `break`；如果候选数很多，可先过滤大于 target 的数。

### 22. 括号生成（Medium）

**题目描述**：给定整数n，生成所有包含n对括号的合法括号组合。

**为什么属于这个类型**：题目要求枚举所有排列、组合、子集、路径或合法构造，答案空间天然是一棵选择树。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：构建合法括号序列的过程——左括号随时可加，右括号只能在左括号有余量时加。

**套用统一模板**：状态是 `path + left + right`；左括号条件是 `left<n`；右括号条件是 `right<left`；长度到 `2n` 时收集。

**基础模板代码**：

```python
def generateParenthesis(self, n):
    res = []
    def backtrack(path, left, right):
        if len(path) == 2 * n:
            res.append(path)
            return
        if left < n:
            backtrack(path + '(', left + 1, right)   # 随时可加左括号
        if right < left:
            backtrack(path + ')', left, right + 1)   # 右括号不能超过左括号
    backtrack('', 0, 0)
    return res
```

**信息流动 / 窗口变化 / 指针移动过程**：

合法括号的本质约束是任意前缀中右括号不能超过左括号。因此 `right < left` 不是技巧，而是合法性的定义。

**优化代码**：

优化主要是剪枝、排序去重、提前返回，避免探索不可能产生答案的子树。

**易错点**：核心优化是生成过程中只走合法前缀，不先生成所有括号串再过滤。

### 17. 电话号码的字母组合（Medium）

**题目描述**：给定一个仅包含2-9的数字字符串，返回所有可能的字母组合。2-9分别映射3-4个字母，空字符串返回空列表。

**为什么属于这个类型**：题目要求枚举所有排列、组合、子集、路径或合法构造，答案空间天然是一棵选择树。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：每个数字对应多个字母——决策树每层选一个字母，穷举所有路径的组合。

**套用统一模板**：状态是 `index + path`；第 `index` 层选择 `digits[index]` 对应的所有字母；`index==len(digits)` 收集。

**基础模板代码**：

```python
class Solution:
    def letterCombinations(self, digits: str) -> List[str]:
        if not digits:
            return []
        mapping = {
            '2': 'abc', '3': 'def', '4': 'ghi', '5': 'jkl',
            '6': 'mno', '7': 'pqrs', '8': 'tuv', '9': 'wxyz'
        }
        res = []
        def backtrack(path, index):
            if index == len(digits):
                res.append(path)
                return
            for ch in mapping[digits[index]]:
                backtrack(path + ch, index + 1)
        backtrack('', 0)
        return res
```

**信息流动 / 窗口变化 / 指针移动过程**：

这题的决策树层数固定等于数字个数，每层的分支数等于该数字映射的字母数。没有复杂剪枝，因为所有组合都合法。

**优化代码**：

优化主要是剪枝、排序去重、提前返回，避免探索不可能产生答案的子树。

**易错点**：空字符串直接返回空数组；也可用队列迭代生成。

### 79. 单词搜索（Medium）

**题目描述**：在一个m×n的二维字符网格中判断给定单词是否存在。单词由相邻单元格的字母组成，相邻指水平或垂直相邻，同一单元格的字母不允许重复使用。

**为什么属于这个类型**：题目要求枚举所有排列、组合、子集、路径或合法构造，答案空间天然是一棵选择树。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：在网格上搜索路径——每个位置决定走向哪个邻居，走过的格子不能重走。

**套用统一模板**：状态是 `(r,c,index)`；选择是四个方向；合法条件是不越界、未访问、字符等于 `word[index]`；找到末尾返回 True；撤销访问标记。

**基础模板代码**：

```python
class Solution:
    def exist(self, board: List[List[str]], word: str) -> bool:
        rows = len(board)
        cols = len(board[0])

        def backtrack(row, col, index):
            if index == len(word):
                return True

            out_of_row = row < 0 or row >= rows
            out_of_col = col < 0 or col >= cols
            if out_of_row or out_of_col:
                return False
            if board[row][col] != word[index]:
                return False

            original_char = board[row][col]
            board[row][col] = '#'

            found_down = backtrack(row + 1, col, index + 1)
            found_up = backtrack(row - 1, col, index + 1)
            found_right = backtrack(row, col + 1, index + 1)
            found_left = backtrack(row, col - 1, index + 1)
            found = found_down or found_up or found_right or found_left

            board[row][col] = original_char
            return found

        for row in range(rows):
            for col in range(cols):
                if backtrack(row, col, 0):
                    return True
        return False
```

**信息流动 / 窗口变化 / 指针移动过程**：

这题不是收集所有路径，而是判断是否存在。因此回溯函数返回布尔值，找到答案就停止。访问标记只在当前路径内有效，回退后必须恢复。

**优化代码**：

优化主要是剪枝、排序去重、提前返回，避免探索不可能产生答案的子树。

**易错点**：先检查字符频次；从更稀有的一端开始搜索；一旦某条路径成功立即短路返回。

### 51. N皇后（Hard）

**题目描述**：在n×n的棋盘上放置n个皇后，使它们不能互相攻击（任意两个皇后不能在同一行、同一列、同一斜线上）。返回所有不同的解法。

**为什么属于这个类型**：题目要求枚举所有排列、组合、子集、路径或合法构造，答案空间天然是一棵选择树。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：每行放一个皇后，约束来自列和对角线——决策树的约束随深度累积，需要高效检测冲突。

**套用统一模板**：状态是 `row + path + cols + diag1 + diag2`；选择是当前行的每一列；合法条件是列、主对角线、副对角线都未占用；`row==n` 收集棋盘。

**基础模板代码**：

```python
class Solution:
    def solveNQueens(self, n: int) -> List[List[str]]:
        res = []
        cols = set()
        diag1 = set()
        diag2 = set()
        def backtrack(row, path):
            if row == n:
                res.append(['.' * c + 'Q' + '.' * (n - c - 1) for c in path])
                return
            for col in range(n):
                if col in cols or (row - col) in diag1 or (row + col) in diag2:
                    continue
                cols.add(col)
                diag1.add(row - col)
                diag2.add(row + col)
                path.append(col)
                backtrack(row + 1, path)
                path.pop()
                diag2.remove(row + col)
                diag1.remove(row - col)
                cols.remove(col)
        backtrack(0, [])
        return res
```

**信息流动 / 窗口变化 / 指针移动过程**：

N 皇后表面是棋盘题，本质是逐行决策树。因为每行必须且只放一个皇后，所以层数就是行号，选择空间就是列号。

**优化代码**：

优化主要是剪枝、排序去重、提前返回，避免探索不可能产生答案的子树。

**易错点**：用集合 O(1) 判断冲突；对角线用 `row-col` 和 `row+col` 表示；还可用位运算进一步压缩。

### 77. 组合（Medium）

**题目描述**：给定两个整数n和k，返回[1,n]中所有可能的k个数的组合。1 <= n <= 20，1 <= k <= n。

**为什么属于这个类型**：题目要求枚举所有排列、组合、子集、路径或合法构造，答案空间天然是一棵选择树。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：从1到n中选k个数——决策树每层选一个数，通过start参数保证递增顺序避免重复。

**套用统一模板**：状态是 `path + start`；选择是 `[start,n]`；`len(path)==k` 收集；递归传 `i+1`。

**基础模板代码**：

```python
class Solution:
    def combine(self, n: int, k: int) -> List[List[int]]:
        res = []
        def backtrack(start, path):
            if len(path) == k:
                res.append(path[:])
                return
            for i in range(start, n - (k - len(path)) + 2):
                path.append(i)
                backtrack(i + 1, path)
                path.pop()
        backtrack(1, [])
        return res
```

**信息流动 / 窗口变化 / 指针移动过程**：

组合题的关键不是 `used`，而是 `start`。一旦选择了较大的数，后面就不再回头选更小的数，从源头避免顺序重复。

**优化代码**：

优化主要是剪枝、排序去重、提前返回，避免探索不可能产生答案的子树。

**易错点**：剩余数字不足时提前停止：`i <= n - (k - len(path)) + 1`。

### 40. 组合总和II（Medium）

**题目描述**：给定候选数字数组candidates和目标数target，找出所有唯一组合使得组合中数字之和为target。每个数字在每个组合中只能使用一次，解集不能包含重复组合。candidates长度<=30。

**为什么属于这个类型**：题目要求枚举所有排列、组合、子集、路径或合法构造，答案空间天然是一棵选择树。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：39题加去重约束——同一位置的数字不能重复使用，相同数字的组合不能重复出现。

**套用统一模板**：状态是 `path + start + remain`；递归传 `i+1` 表示每个下标只能用一次；同层遇到重复值要跳过。

**基础模板代码**：

```python
class Solution:
    def combinationSum2(self, candidates: List[int], target: int) -> List[List[int]]:
        candidates.sort()
        res = []
        def backtrack(start, path, remain):
            if remain == 0:
                res.append(path[:])
                return
            if remain < 0:
                return
            for i in range(start, len(candidates)):
                if i > start and candidates[i] == candidates[i-1]:
                    continue
                path.append(candidates[i])
                backtrack(i + 1, path, remain - candidates[i])
                path.pop()
        backtrack(0, [], target)
        return res
```

**信息流动 / 窗口变化 / 指针移动过程**：

去重必须发生在同一层，而不是所有层。因为同一层选相同值会产生重复组合，但不同层选相同值可能代表使用不同下标，是合法路径。

**优化代码**：

优化主要是剪枝、排序去重、提前返回，避免探索不可能产生答案的子树。

**易错点**：排序后用 `if i > start and candidates[i] == candidates[i-1]: continue` 去重；当前数大于 remain 时 `break`。

### 47. 全排列II（Medium）

**题目描述**：给定一个可包含重复数字的序列nums，返回所有不重复的全排列。序列长度<=8。

**为什么属于这个类型**：题目要求枚举所有排列、组合、子集、路径或合法构造，答案空间天然是一棵选择树。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：46题加去重——含重复元素的排列去重，相同元素在同一层只选一次。

**套用统一模板**：状态是 `path + used`；选择未使用下标；排序后同层跳过相同值，避免重复分支。

**基础模板代码**：

```python
class Solution:
    def permuteUnique(self, nums: List[int]) -> List[List[int]]:
        nums.sort()
        res = []
        used = [False] * len(nums)
        def backtrack(path):
            if len(path) == len(nums):
                res.append(path[:])
                return
            for i in range(len(nums)):
                if used[i]:
                    continue
                if i > 0 and nums[i] == nums[i-1] and not used[i-1]:
                    continue
                used[i] = True
                path.append(nums[i])
                backtrack(path)
                path.pop()
                used[i] = False
        backtrack([])
        return res
```

**信息流动 / 窗口变化 / 指针移动过程**：

重复排列的核心是区分“相同值的不同下标”。排序后强制相同值按固定顺序被使用，就不会生成等价排列。

**优化代码**：

优化主要是剪枝、排序去重、提前返回，避免探索不可能产生答案的子树。

**易错点**：剪枝条件：`i>0 and nums[i]==nums[i-1] and not used[i-1]` 时跳过。

## 连通性的探索 (图遍历/并查集)

### 如何识别这是图遍历 / 并查集题

图题不一定直接给“图”，只要对象之间存在关系，就可以抽象成图。识别信号是：

1. 题目问是否连通、多少个连通块、能否从 A 到 B。
2. 网格中上下左右相邻可以传播或连接。
3. 课程、依赖、前置条件构成有向边。
4. 等价关系、合并集合、最长连续段可以用连通分量理解。

判断公式：

> 如果核心问题是“关系如何传播”或“哪些对象属于同一组”，就考虑图遍历或并查集。

### 一句话本质

回答点与点、块与块之间是否连通。

### 第一性原理

图问题的第一性原理是：数据之间存在邻接关系，算法要沿着关系传播信息。DFS/BFS 从局部起点扩散，并查集从全局维护连通分量。

### 统一模板：先问 5 个问题

1. 节点是什么？边是什么？
2. 要找连通块、最短层数、拓扑顺序还是等价关系？
3. 访问过的节点如何标记？
4. DFS、BFS、并查集哪个更贴近问题？
5. 是否存在方向、权值、环或多源起点？

### 通用代码骨架

```python
visited = set()
def dfs(node):
    if node in visited: return
    visited.add(node)
    for nxt in graph[node]:
        dfs(nxt)
```

### 常见状态变量

- `visited`：防止重复访问
- `queue`：BFS 层序扩散
- `parent/rank`：并查集维护集合代表元
- `indegree`：拓扑排序维护依赖入口

### 常见剪枝 / 优化

- 越界/水域/已访问直接返回
- BFS 找到目标即可提前结束
- 拓扑排序入度不为 0 的点暂不能处理

### 本类题目总览

- 200. 岛屿数量（Medium）
- 994. 腐烂的橘子（Medium）
- 207. 课程表（Medium）
- 210. 课程表II（Medium）
- 133. 克隆图（Medium）
- 130. 被围绕的区域（Medium）
- 399. 除法求值（Medium）
- 785. 判断二分图（Medium）
- 208. 实现Trie前缀树（Medium）
- 128. 最长连续序列（Medium）

### 200. 岛屿数量（Medium）

**题目描述**：给定一个由'1'(陆地)和'0'(水)组成的m×n二维网格，计算岛屿的数量。岛屿被水包围，由水平或垂直相邻的陆地连接而成。

**为什么属于这个类型**：题目核心是对象之间的连接、传播、依赖或同组关系，可以抽象成节点和边。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：网格中的连通分量计数：每次发现未访问的陆地，就是一个新岛屿的起点，BFS/DFS将整块连通陆地标记为已访问。

**套用统一模板**：遍历类维护 `visited`，BFS 维护队列层数，并查集维护 `parent`，拓扑排序维护 `indegree`。

**基础模板代码**：

```python
class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        if not grid:
            return 0

        rows = len(grid)
        cols = len(grid[0])

        def dfs(row, col):
            out_of_row = row < 0 or row >= rows
            out_of_col = col < 0 or col >= cols
            if out_of_row or out_of_col:
                return
            if grid[row][col] != '1':
                return

            grid[row][col] = '0'
            dfs(row + 1, col)
            dfs(row - 1, col)
            dfs(row, col + 1)
            dfs(row, col - 1)

        count = 0
        for row in range(rows):
            for col in range(cols):
                if grid[row][col] == '1':
                    dfs(row, col)
                    count += 1

        return count
```

**信息流动 / 窗口变化 / 指针移动过程**：

BFS/DFS/并查集均可。遍历网格，遇到'1'则启动DFS/BFS将相连的陆地全部标记为已访问，岛屿数+1。核心思想是'感染'——每发现一个新岛屿的起点，就将整个岛屿沉没（标记为'0'），后续遍历不再重复计数。DFS递归向四个方向扩展，越界或遇到水则返回。时间复杂度O(m*n)，空间复杂度O(m*n)最坏递归深度。

**优化代码**：

优化重点是避免重复访问、原地标记、多源 BFS、拓扑入度或路径压缩。

**易错点**：网格题可原地标记；最短传播用多源 BFS；等价关系用并查集；依赖环检测用拓扑排序。

### 994. 腐烂的橘子（Medium）

**题目描述**：在m×n网格中，0代表空，1代表新鲜橘子，2代表腐烂橘子。每分钟腐烂橘子使上下左右相邻的新鲜橘子腐烂。返回所有橘子腐烂的最少分钟数，不可能返回-1。

**为什么属于这个类型**：题目核心是对象之间的连接、传播、依赖或同组关系，可以抽象成节点和边。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：多源BFS的最短传播距离：所有腐烂橘子同时向外扩散，每分钟扩散一层，求最后一个新鲜橘子被感染的时间。

**套用统一模板**：遍历类维护 `visited`，BFS 维护队列层数，并查集维护 `parent`，拓扑排序维护 `indegree`。

**基础模板代码**：

```python
class Solution:
    def orangesRotting(self, grid: List[List[int]]) -> int:
        rows = len(grid)
        cols = len(grid[0])
        queue = []
        fresh = 0

        for row in range(rows):
            for col in range(cols):
                if grid[row][col] == 2:
                    queue.append((row, col))
                elif grid[row][col] == 1:
                    fresh += 1

        if fresh == 0:
            return 0

        minutes = 0
        directions = [(1, 0), (-1, 0), (0, 1), (0, -1)]

        while queue:
            next_queue = []
            for row, col in queue:
                for row_delta, col_delta in directions:
                    next_row = row + row_delta
                    next_col = col + col_delta

                    row_ok = 0 <= next_row < rows
                    col_ok = 0 <= next_col < cols
                    if row_ok and col_ok and grid[next_row][next_col] == 1:
                        grid[next_row][next_col] = 2
                        fresh -= 1
                        next_queue.append((next_row, next_col))

            queue = next_queue
            if queue:
                minutes += 1

        if fresh == 0:
            return minutes
        return -1
```

**信息流动 / 窗口变化 / 指针移动过程**：

多源BFS。初始将所有腐烂橘子入队作为第0层，同时统计新鲜橘子数量。BFS逐层扩展，每层代表一分钟，腐烂相邻新鲜橘子并减少新鲜计数。当BFS结束（队列为空）时，若新鲜橘子数为0则返回层数-1（最后一层不需要额外时间），否则返回-1。注意无新鲜橘子时直接返回0。时间复杂度O(m*n)，空间复杂度O(m*n)。

**优化代码**：

优化重点是避免重复访问、原地标记、多源 BFS、拓扑入度或路径压缩。

**易错点**：网格题可原地标记；最短传播用多源 BFS；等价关系用并查集；依赖环检测用拓扑排序。

### 207. 课程表（Medium）

**题目描述**：给定numCourses门课和先修课程列表prerequisites，判断是否可能完成所有课程（即判断有向图是否有环）。课程数<=2000。

**为什么属于这个类型**：题目核心是对象之间的连接、传播、依赖或同组关系，可以抽象成节点和边。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：有向图中是否存在环：课程依赖构成有向图，能完成所有课程等价于图中无环（是DAG）。

**套用统一模板**：遍历类维护 `visited`，BFS 维护队列层数，并查集维护 `parent`，拓扑排序维护 `indegree`。

**基础模板代码**：

```python
class Solution:
    def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
        from collections import deque
        adj = [[] for _ in range(numCourses)]
        indeg = [0] * numCourses
        for a, b in prerequisites:
            adj[b].append(a)
            indeg[a] += 1
        q = deque([i for i in range(numCourses) if indeg[i] == 0])
        cnt = 0
        while q:
            node = q.popleft()
            cnt += 1
            for nb in adj[node]:
                indeg[nb] -= 1
                if indeg[nb] == 0:
                    q.append(nb)
        return cnt == numCourses
```

**信息流动 / 窗口变化 / 指针移动过程**：

拓扑排序判断有向图是否有环。两种方法：

1. BFS（Kahn算法）：计算所有节点入度，入度为0的节点入队，依次移除并更新邻居入度，若最终访问节点数等于课程数则无环。
2. DFS三色标记：白色(0)未访问、灰色(1)正在访问、黑色(2)已完成。DFS中遇到灰色节点说明有环。

BFS更直观：构建邻接表和入度数组，初始入度为0的课程入队，每次出队一个课程并将其后续课程的入度减1，入度变0则入队。最终看处理课程数是否等于总数。时间复杂度O(V+E)，空间复杂度O(V+E)。

**优化代码**：

优化重点是避免重复访问、原地标记、多源 BFS、拓扑入度或路径压缩。

**易错点**：网格题可原地标记；最短传播用多源 BFS；等价关系用并查集；依赖环检测用拓扑排序。

### 210. 课程表II（Medium）

**题目描述**：给定numCourses门课和先修课程列表，返回一种完成所有课程的顺序（拓扑排序）。不可能完成返回空数组。课程数<=2000。

**为什么属于这个类型**：题目核心是对象之间的连接、传播、依赖或同组关系，可以抽象成节点和边。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：输出拓扑排序的具体序列：在207题基础上不仅判断是否有环，还要记录删除节点的顺序。

**套用统一模板**：遍历类维护 `visited`，BFS 维护队列层数，并查集维护 `parent`，拓扑排序维护 `indegree`。

**基础模板代码**：

```python
def findOrder(self, numCourses, prerequisites):
    from collections import deque

    graph = [[] for _ in range(numCourses)]
    indegree = [0] * numCourses

    for course, prerequisite in prerequisites:
        graph[prerequisite].append(course)
        indegree[course] += 1

    queue = deque()
    for course in range(numCourses):
        if indegree[course] == 0:
            queue.append(course)

    result = []
    while queue:
        course = queue.popleft()
        result.append(course)

        for next_course in graph[course]:
            indegree[next_course] -= 1
            if indegree[next_course] == 0:
                queue.append(next_course)

    if len(result) == numCourses:
        return result
    return []
```

**信息流动 / 窗口变化 / 指针移动过程**：

与课程表I相同的拓扑排序，额外记录出队顺序即为一种合法选课顺序。BFS拓扑排序：入度为0的课程入队，依次出队并加入结果，将邻居入度减1，新入度为0的继续入队。若结果长度等于课程数则返回结果，否则有环返回空数组。时间复杂度O(V+E)，空间复杂度O(V+E)。

**优化代码**：

优化重点是避免重复访问、原地标记、多源 BFS、拓扑入度或路径压缩。

**易错点**：网格题可原地标记；最短传播用多源 BFS；等价关系用并查集；依赖环检测用拓扑排序。

### 133. 克隆图（Medium）

**题目描述**：给定无向连通图中一个节点的引用，返回该图的深拷贝。每个节点包含val和邻居列表。节点数<=100。

**为什么属于这个类型**：题目核心是对象之间的连接、传播、依赖或同组关系，可以抽象成节点和边。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：图的深拷贝：遍历原图的同时创建新节点，用哈希表维护旧节点到新节点的映射以处理引用关系。

**套用统一模板**：遍历类维护 `visited`，BFS 维护队列层数，并查集维护 `parent`，拓扑排序维护 `indegree`。

**基础模板代码**：

```python
class Solution:
    def cloneGraph(self, node: 'Node') -> 'Node':
        if not node:
            return None
        visited = {}
        def dfs(n):
            if n in visited:
                return visited[n]
            clone = Node(n.val, [])
            visited[n] = clone
            for nb in n.neighbors:
                clone.neighbors.append(dfs(nb))
            return clone
        return dfs(node)
```

**信息流动 / 窗口变化 / 指针移动过程**：

DFS或BFS遍历原图，用字典存储原节点到新节点的映射。遍历时若邻居未被克隆则递归/入队克隆，然后建立新图中邻居关系。关键：用visited字典防止重复克隆和死循环。DFS递归较简洁，BFS用队列。时间复杂度O(V+E)，空间复杂度O(V)。

**优化代码**：

优化重点是避免重复访问、原地标记、多源 BFS、拓扑入度或路径压缩。

**易错点**：网格题可原地标记；最短传播用多源 BFS；等价关系用并查集；依赖环检测用拓扑排序。

### 130. 被围绕的区域（Medium）

**题目描述**：给定m×n的board含'X'和'O'，将被'X'围绕的'O'区域填充为'X'。任何与边界相连的'O'不被填充。board大小<=200×200。

**为什么属于这个类型**：题目核心是对象之间的连接、传播、依赖或同组关系，可以抽象成节点和边。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：只有与边界相连的'O'不会被包围：从边界出发的BFS/DFS标记所有不被包围的'O'，剩下的全变'X'。

**套用统一模板**：遍历类维护 `visited`，BFS 维护队列层数，并查集维护 `parent`，拓扑排序维护 `indegree`。

**基础模板代码**：

```python
def solve(self, board):
    if not board:
        return

    rows = len(board)
    cols = len(board[0])

    def dfs(row, col):
        out_of_row = row < 0 or row >= rows
        out_of_col = col < 0 or col >= cols
        if out_of_row or out_of_col:
            return
        if board[row][col] != 'O':
            return

        board[row][col] = 'T'
        dfs(row + 1, col)
        dfs(row - 1, col)
        dfs(row, col + 1)
        dfs(row, col - 1)

    for row in range(rows):
        dfs(row, 0)
        dfs(row, cols - 1)

    for col in range(cols):
        dfs(0, col)
        dfs(rows - 1, col)

    for row in range(rows):
        for col in range(cols):
            if board[row][col] == 'O':
                board[row][col] = 'X'
            elif board[row][col] == 'T':
                board[row][col] = 'O'
```

**信息流动 / 窗口变化 / 指针移动过程**：

逆向思维：不被围绕的'O'一定与边界相连。先从四条边界出发DFS/BFS标记所有与边界相连的'O'为特殊字符（如'T'），然后遍历整个board：将'O'（不与边界相连）改为'X'，将'T'恢复为'O'。核心洞察：与边界相连的'O'是安全的，其余'O'必然被X包围。时间复杂度O(m*n)，空间复杂度O(m*n)。

**优化代码**：

优化重点是避免重复访问、原地标记、多源 BFS、拓扑入度或路径压缩。

**易错点**：网格题可原地标记；最短传播用多源 BFS；等价关系用并查集；依赖环检测用拓扑排序。

### 399. 除法求值（Medium）

**题目描述**：给定变量对之间的除法结果equations和values，以及若干查询queries，返回每个查询的结果。若无法确定则返回-1.0。变量数<=20。

**为什么属于这个类型**：题目核心是对象之间的连接、传播、依赖或同组关系，可以抽象成节点和边。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：带权并查集或带权图的DFS：变量间的除法关系构成带权有向图，查询两变量的比值就是在图上找路径并累乘权重。

**套用统一模板**：遍历类维护 `visited`，BFS 维护队列层数，并查集维护 `parent`，拓扑排序维护 `indegree`。

**基础模板代码**：

```python
class Solution:
    def calcEquation(self, equations, values, queries):
        from collections import defaultdict, deque

        graph = defaultdict(list)
        for index, equation in enumerate(equations):
            numerator = equation[0]
            denominator = equation[1]
            value = values[index]
            graph[numerator].append((denominator, value))
            graph[denominator].append((numerator, 1.0 / value))

        def bfs(source, target):
            if source not in graph:
                return -1.0
            if target not in graph:
                return -1.0

            queue = deque()
            queue.append((source, 1.0))
            visited = {source}

            while queue:
                node, product = queue.popleft()
                if node == target:
                    return product

                for neighbor, weight in graph[node]:
                    if neighbor in visited:
                        continue
                    visited.add(neighbor)
                    next_product = product * weight
                    queue.append((neighbor, next_product))

            return -1.0

        result = []
        for query in queries:
            source = query[0]
            target = query[1]
            value = bfs(source, target)
            result.append(value)
        return result
```

**信息流动 / 窗口变化 / 指针移动过程**：

将除法关系建模为带权有向图：a/b=v表示a到b的边权为v，b到a的边权为1/v。查询a/c等价于找a到c的路径，路径上边权之积即为结果。BFS/DFS从起点搜索到终点，累积乘积。若起点或终点不在图中或不可达，返回-1.0。也可用并查集，维护到根的权值比。时间复杂度O(Q*(V+E))，空间复杂度O(V+E)。

**优化代码**：

优化重点是避免重复访问、原地标记、多源 BFS、拓扑入度或路径压缩。

**易错点**：网格题可原地标记；最短传播用多源 BFS；等价关系用并查集；依赖环检测用拓扑排序。

### 785. 判断二分图（Medium）

**题目描述**：给定无向图，判断是否为二分图（能否将节点分成两个集合，使得每条边的两个端点分别属于不同集合）。节点数<=100。

**为什么属于这个类型**：题目核心是对象之间的连接、传播、依赖或同组关系，可以抽象成节点和边。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：图能否二着色：相邻节点必须不同色，BFS/DFS交替染色，若冲突则非二分图。

**套用统一模板**：遍历类维护 `visited`，BFS 维护队列层数，并查集维护 `parent`，拓扑排序维护 `indegree`。

**基础模板代码**：

```python
class Solution:
    def isBipartite(self, graph: List[List[int]]) -> bool:
        from collections import deque
        n = len(graph)
        color = [0] * n
        for i in range(n):
            if color[i] != 0:
                continue
            q = deque([i])
            color[i] = 1
            while q:
                node = q.popleft()
                for nb in graph[node]:
                    if color[nb] == 0:
                        color[nb] = -color[node]
                        q.append(nb)
                    elif color[nb] == color[node]:
                        return False
        return True
```

**信息流动 / 窗口变化 / 指针移动过程**：

二分图等价于图可以被二着色（相邻节点不同色）。BFS/DFS遍历，用颜色数组（0未染色，1和-1两种颜色）标记。遍历每个未染色节点，染成1并BFS/DFS其邻居染成-1，交替染色。若发现邻居与自己同色则不是二分图。注意图可能不连通，需遍历所有节点。时间复杂度O(V+E)，空间复杂度O(V)。

**优化代码**：

优化重点是避免重复访问、原地标记、多源 BFS、拓扑入度或路径压缩。

**易错点**：网格题可原地标记；最短传播用多源 BFS；等价关系用并查集；依赖环检测用拓扑排序。

### 208. 实现Trie前缀树（Medium）

**题目描述**：实现前缀树（Trie），支持insert插入单词、search搜索单词、startsWith搜索前缀。单词仅由小写字母组成。

**为什么属于这个类型**：题目核心是对象之间的连接、传播、依赖或同组关系，可以抽象成节点和边。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：多叉树的路径编码字符串：从根到某节点的路径代表一个字符串前缀，每条边对应一个字符。

**套用统一模板**：遍历类维护 `visited`，BFS 维护队列层数，并查集维护 `parent`，拓扑排序维护 `indegree`。

**基础模板代码**：

```python
class Trie:
    def __init__(self):
        self.children = {}
        self.is_end = False

    def insert(self, word: str) -> None:
        node = self
        for ch in word:
            if ch not in node.children:
                node.children[ch] = Trie()
            node = node.children[ch]
        node.is_end = True

    def search(self, word: str) -> bool:
        node = self._find(word)
        return node is not None and node.is_end

    def startsWith(self, prefix: str) -> bool:
        return self._find(prefix) is not None

    def _find(self, prefix: str):
        node = self
        for ch in prefix:
            if ch not in node.children:
                return None
            node = node.children[ch]
        return node
```

**信息流动 / 窗口变化 / 指针移动过程**：

前缀树是多叉树结构，每个节点包含26个子节点指针和一个is_end标记。insert逐字符遍历，不存在则创建节点，末尾标记is_end=True。search逐字符遍历，路径中断或末尾is_end为False则返回False。startsWith与search类似但不检查is_end。每个操作时间复杂度O(L)，L为单词长度。空间复杂度O(总字符数)。

**优化代码**：

优化重点是避免重复访问、原地标记、多源 BFS、拓扑入度或路径压缩。

**易错点**：网格题可原地标记；最短传播用多源 BFS；等价关系用并查集；依赖环检测用拓扑排序。

### 128. 最长连续序列（Medium）

**题目描述**：给定未排序的整数数组nums，找出数字连续的最长序列的长度。要求算法时间复杂度为O(n)。数组长度<=10^5。

**为什么属于这个类型**：题目核心是对象之间的连接、传播、依赖或同组关系，可以抽象成节点和边。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：在无序数组中找最长连续整数序列：对每个可能的序列起点（num-1不存在），向后延伸计数。

**套用统一模板**：遍历类维护 `visited`，BFS 维护队列层数，并查集维护 `parent`，拓扑排序维护 `indegree`。

**基础模板代码**：

```python
class Solution:
    def longestConsecutive(self, nums: List[int]) -> int:
        num_set = set(nums)
        longest = 0
        for n in num_set:
            if n - 1 not in num_set:
                cur = n
                length = 1
                while cur + 1 in num_set:
                    cur += 1
                    length += 1
                longest = max(longest, length)
        return longest
```

**信息流动 / 窗口变化 / 指针移动过程**：

关键洞察：若要求O(n)时间，不能排序。用HashSet存储所有数字，对于每个数字n，只有当n-1不在集合中时才以n为序列起点开始向后枚举（n+1, n+2,...），这样每个数字最多被访问两次。如果n-1存在，则n不是序列起点，跳过，避免重复计算。这是O(n)的关键。时间复杂度O(n)，空间复杂度O(n)。

**优化代码**：

优化重点是避免重复访问、原地标记、多源 BFS、拓扑入度或路径压缩。

**易错点**：网格题可原地标记；最短传播用多源 BFS；等价关系用并查集；依赖环检测用拓扑排序。

## 树结构的递归分解 (二叉树)

### 如何识别这是二叉树递归题

二叉树题的识别通常很直接，但关键是判断递归返回什么。识别信号是：

1. 输入是树节点 `root`。
2. 一个节点的答案依赖左右子树答案。
3. 空节点天然提供 base case。
4. 题目要求高度、路径、合法性、序列化、构造或遍历。

判断公式：

> 如果一棵树的问题可以拆成“左子树答案 + 右子树答案 + 根节点处理”，就优先写递归。

### 一句话本质

每棵子树都是同类问题。

### 第一性原理

二叉树的第一性原理是递归定义：一棵树等于根节点加左子树加右子树。因此树题通常把问题拆成：当前根节点要做什么，左右子树分别返回什么。

### 统一模板：先问 5 个问题

1. 递归函数对一棵子树返回什么？
2. 空节点的 base case 是什么？
3. 当前根如何合并左右子树信息？
4. 答案是在自顶向下传参，还是自底向上返回？
5. 是否需要全局变量记录跨节点最优值？

### 通用代码骨架

```python
def dfs(root):
    if not root:
        return 空树答案
    left = dfs(root.left)
    right = dfs(root.right)
    return 根节点合并(left, right)
```

### 常见状态变量

- 返回高度、路径和、是否合法、序列化字符串等
- 全局答案：直径、最大路径和这类不一定经过根的最优值
- 队列：层序遍历按层处理

### 常见剪枝 / 优化

- BST 可用上下界提前判错
- 找到目标节点可提前返回
- 空节点统一作为递归边界

### 本类题目总览

- 104. 二叉树的最大深度（Easy）
- 226. 翻转二叉树（Easy）
- 101. 对称二叉树（Easy）
- 543. 二叉树的直径（Easy）
- 98. 验证二叉搜索树（Medium）
- 102. 二叉树的层序遍历（Medium）
- 124. 二叉树中的最大路径和（Hard）
- 236. 二叉树的最近公共祖先（Medium）
- 105. 从前序与中序遍历序列构造二叉树（Medium）
- 114. 二叉树展开为链表（Medium）
- 199. 二叉树的右视图（Medium）
- 297. 二叉树的序列化与反序列化（Hard）
- 617. 合并二叉树（Easy）
- 437. 路径总和III（Medium）

### 104. 二叉树的最大深度（Easy）

**题目描述**：给定二叉树的根节点，返回其最大深度（从根节点到最远叶子节点的最长路径上的节点数）。节点数<=10^4。

**为什么属于这个类型**：二叉树天然递归定义，整棵树的问题通常可以拆成根节点、左子树、右子树的同类问题。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：树的最大深度=1+max(左子树深度, 右子树深度)，递归定义天然就是解法。

**套用统一模板**：空节点给 base case；递归求左右；根节点合并左右信息；必要时用全局变量记录不一定经过根的答案。

**基础模板代码**：

```python
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0
        return 1 + max(self.maxDepth(root.left), self.maxDepth(root.right))
```

**信息流动 / 窗口变化 / 指针移动过程**：

递归DFS：最大深度 = max(左子树深度, 右子树深度) + 1。空节点深度为0。这是最自然的递归分治写法。也可用BFS层序遍历统计层数。时间复杂度O(n)，空间复杂度O(h)，h为树高。

**优化代码**：

优化重点是明确返回值和全局答案的区别，避免重复遍历子树。

**易错点**：BST 题用上下界；层序题用队列；构造题用哈希表定位中序下标；路径题区分返回给父节点的单边贡献和全局答案。

### 226. 翻转二叉树（Easy）

**题目描述**：给定二叉树的根节点，翻转该二叉树（交换每个节点的左右子树）。节点数<=100。

**为什么属于这个类型**：二叉树天然递归定义，整棵树的问题通常可以拆成根节点、左子树、右子树的同类问题。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：翻转一棵树=交换左右子树+分别翻转左右子树，递归定义即递归解法。

**套用统一模板**：空节点给 base case；递归求左右；根节点合并左右信息；必要时用全局变量记录不一定经过根的答案。

**基础模板代码**：

```python
class Solution:
    def invertTree(self, root: Optional[TreeNode]) -> Optional[TreeNode]:
        if not root:
            return None
        root.left, root.right = root.right, root.left
        self.invertTree(root.left)
        self.invertTree(root.right)
        return root
```

**信息流动 / 窗口变化 / 指针移动过程**：

递归：对每个节点交换其左右子树，然后递归翻转左右子树。也可BFS逐层交换。递归是最简洁的写法：先交换当前节点的左右子节点，再递归处理左右子树（顺序无所谓，因为交换后原左子树变成右子树，但递归参数已经确定）。时间复杂度O(n)，空间复杂度O(h)。

**优化代码**：

优化重点是明确返回值和全局答案的区别，避免重复遍历子树。

**易错点**：BST 题用上下界；层序题用队列；构造题用哈希表定位中序下标；路径题区分返回给父节点的单边贡献和全局答案。

### 101. 对称二叉树（Easy）

**题目描述**：给定二叉树的根节点，判断它是否轴对称（左右子树镜像对称）。节点数<=1000。

**为什么属于这个类型**：二叉树天然递归定义，整棵树的问题通常可以拆成根节点、左子树、右子树的同类问题。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：对称性=左子树镜像等于右子树，递归比较(左.左,右.右)和(左.右,右.左)。

**套用统一模板**：空节点给 base case；递归求左右；根节点合并左右信息；必要时用全局变量记录不一定经过根的答案。

**基础模板代码**：

```python
def isSymmetric(self, root):
    def check(left_node, right_node):
        if not left_node and not right_node:
            return True
        if not left_node or not right_node:
            return False
        if left_node.val != right_node.val:
            return False

        outside_same = check(left_node.left, right_node.right)
        inside_same = check(left_node.right, right_node.left)
        return outside_same and inside_same

    return check(root.left, root.right)
```

**信息流动 / 窗口变化 / 指针移动过程**：

递归判断：左子树和右子树镜像对称 ⟺ 左子树的左子树与右子树的右子树对称 且 左子树的右子树与右子树的左子树对称，且根节点值相等。定义辅助函数check(p, q)判断两棵树是否镜像对称。也可用BFS双队列实现迭代版本。时间复杂度O(n)，空间复杂度O(h)。

**优化代码**：

优化重点是明确返回值和全局答案的区别，避免重复遍历子树。

**易错点**：BST 题用上下界；层序题用队列；构造题用哈希表定位中序下标；路径题区分返回给父节点的单边贡献和全局答案。

### 543. 二叉树的直径（Easy）

**题目描述**：给定二叉树的根节点，返回其直径长度（任意两节点间最长路径的边数）。节点数<=10^4。

**为什么属于这个类型**：二叉树天然递归定义，整棵树的问题通常可以拆成根节点、左子树、右子树的同类问题。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：直径=某个节点左子树最大深度+右子树最大深度，遍历所有节点取最大值。

**套用统一模板**：空节点给 base case；递归求左右；根节点合并左右信息；必要时用全局变量记录不一定经过根的答案。

**基础模板代码**：

```python
class Solution:
    def diameterOfBinaryTree(self, root: Optional[TreeNode]) -> int:
        self.diameter = 0
        def depth(node):
            if not node:
                return 0
            left = depth(node.left)
            right = depth(node.right)
            self.diameter = max(self.diameter, left + right)
            return 1 + max(left, right)
        depth(root)
        return self.diameter
```

**信息流动 / 窗口变化 / 指针移动过程**：

直径可能不经过根节点。直径 = max(左子树最大深度 + 右子树最大深度)。递归计算每个节点的深度时，顺便用全局变量更新直径：经过当前节点的最长路径 = 左深度 + 右深度。深度函数返回以当前节点为根的最大深度。每个节点访问一次，时间复杂度O(n)，空间复杂度O(h)。

**优化代码**：

优化重点是明确返回值和全局答案的区别，避免重复遍历子树。

**易错点**：BST 题用上下界；层序题用队列；构造题用哈希表定位中序下标；路径题区分返回给父节点的单边贡献和全局答案。

### 98. 验证二叉搜索树（Medium）

**题目描述**：给定二叉树的根节点，判断其是否为有效的二叉搜索树（左子树所有节点值<根节点值<右子树所有节点值）。节点数<=10^4。

**为什么属于这个类型**：二叉树天然递归定义，整棵树的问题通常可以拆成根节点、左子树、右子树的同类问题。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：BST的中序遍历严格递增，或递归时传递有效值范围(min, max)约束。

**套用统一模板**：空节点给 base case；递归求左右；根节点合并左右信息；必要时用全局变量记录不一定经过根的答案。

**基础模板代码**：

```python
def isValidBST(self, root):
    def validate(node, lower_bound, upper_bound):
        if not node:
            return True

        if node.val <= lower_bound:
            return False
        if node.val >= upper_bound:
            return False

        left_valid = validate(node.left, lower_bound, node.val)
        right_valid = validate(node.right, node.val, upper_bound)
        return left_valid and right_valid

    return validate(root, float('-inf'), float('inf'))
```

**信息流动 / 窗口变化 / 指针移动过程**：

BST的中序遍历是严格递增的。方法1：中序遍历，检查当前值是否大于前一个值。方法2：递归时传递上下界（min, max），每个节点值必须在(min, max)开区间内。根节点的上下界为(-inf, +inf)，左子树上界更新为当前节点值，右子树下界更新为当前节点值。注意是严格小于/大于，不是<=。时间复杂度O(n)，空间复杂度O(h)。

**优化代码**：

优化重点是明确返回值和全局答案的区别，避免重复遍历子树。

**易错点**：BST 题用上下界；层序题用队列；构造题用哈希表定位中序下标；路径题区分返回给父节点的单边贡献和全局答案。

### 102. 二叉树的层序遍历（Medium）

**题目描述**：给定二叉树的根节点，返回其节点值的层序遍历（逐层从左到右访问）。节点数<=2000。

**为什么属于这个类型**：二叉树天然递归定义，整棵树的问题通常可以拆成根节点、左子树、右子树的同类问题。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：BFS按层处理：队列中每次处理一整层，记录层内所有节点值。

**套用统一模板**：空节点给 base case；递归求左右；根节点合并左右信息；必要时用全局变量记录不一定经过根的答案。

**基础模板代码**：

```python
class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []
        from collections import deque
        res = []
        q = deque([root])
        while q:
            level = []
            for _ in range(len(q)):
                node = q.popleft()
                level.append(node.val)
                if node.left:
                    q.append(node.left)
                if node.right:
                    q.append(node.right)
            res.append(level)
        return res
```

**信息流动 / 窗口变化 / 指针移动过程**：

BFS用队列逐层遍历。每层开始时记录队列长度（即该层节点数），一次性处理完一层再进入下一层。这是区分层级的关键——不能简单地一个一个出队，需要按层批量处理。也可以用DFS加level参数，按层级将节点值加入对应列表。时间复杂度O(n)，空间复杂度O(n)。

**优化代码**：

优化重点是明确返回值和全局答案的区别，避免重复遍历子树。

**易错点**：BST 题用上下界；层序题用队列；构造题用哈希表定位中序下标；路径题区分返回给父节点的单边贡献和全局答案。

### 124. 二叉树中的最大路径和（Hard）

**题目描述**：给定二叉树的根节点，返回其最大路径和（路径定义为从任一节点出发沿父-子连接到达任一节点的序列，路径至少包含一个节点）。节点值可为负。

**为什么属于这个类型**：二叉树天然递归定义，整棵树的问题通常可以拆成根节点、左子树、右子树的同类问题。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：路径和=某节点左单链最大贡献+右单链最大贡献+节点值，递归计算单链贡献同时更新全局最大路径和。

**套用统一模板**：空节点给 base case；递归求左右；根节点合并左右信息；必要时用全局变量记录不一定经过根的答案。

**基础模板代码**：

```python
class Solution:
    def maxPathSum(self, root: Optional[TreeNode]) -> int:
        self.max_sum = float('-inf')
        def gain(node):
            if not node:
                return 0
            left = max(gain(node.left), 0)
            right = max(gain(node.right), 0)
            self.max_sum = max(self.max_sum, node.val + left + right)
            return node.val + max(left, right)
        gain(root)
        return self.max_sum
```

**信息流动 / 窗口变化 / 指针移动过程**：

路径可以经过任意节点，不一定经过根。关键区分两个概念：1) 从当前节点向下的最大贡献（只能选左或右一条分支），用于给父节点使用；2) 经过当前节点的路径和 = 节点值 + 左贡献 + 右贡献（可以同时走左右两条分支），用于更新全局最大值。

递归函数返回以当前节点为起点的最大单路径贡献：max(0, left_gain, right_gain) + node.val。如果子树贡献为负则不取（取0更好）。在递归过程中用全局变量更新最大路径和。时间复杂度O(n)，空间复杂度O(h)。

**优化代码**：

优化重点是明确返回值和全局答案的区别，避免重复遍历子树。

**易错点**：BST 题用上下界；层序题用队列；构造题用哈希表定位中序下标；路径题区分返回给父节点的单边贡献和全局答案。

### 236. 二叉树的最近公共祖先（Medium）

**题目描述**：给定二叉树的根节点和两个节点p、q，找到它们的最近公共祖先（LCA）。所有节点值唯一，p和q存在于树中。

**为什么属于这个类型**：二叉树天然递归定义，整棵树的问题通常可以拆成根节点、左子树、右子树的同类问题。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：LCA是第一个使p和q分布在其两侧子树（或自身就是p/q）的节点，递归自底向上寻找。

**套用统一模板**：空节点给 base case；递归求左右；根节点合并左右信息；必要时用全局变量记录不一定经过根的答案。

**基础模板代码**：

```python
def lowestCommonAncestor(self, root, p, q):
    if not root:
        return None
    if root == p:
        return root
    if root == q:
        return root

    left_result = self.lowestCommonAncestor(root.left, p, q)
    right_result = self.lowestCommonAncestor(root.right, p, q)

    if left_result and right_result:
        return root
    if left_result:
        return left_result
    return right_result
```

**信息流动 / 窗口变化 / 指针移动过程**：

递归后序遍历：若当前节点为p或q或null，直接返回当前节点。递归左右子树，若左右都非空则当前节点就是LCA（p和q分别在两侧）；若只有一侧非空，则LCA在该侧子树中。核心逻辑：从底向上，第一个同时包含p和q的节点就是LCA。时间复杂度O(n)，空间复杂度O(h)。

**优化代码**：

优化重点是明确返回值和全局答案的区别，避免重复遍历子树。

**易错点**：BST 题用上下界；层序题用队列；构造题用哈希表定位中序下标；路径题区分返回给父节点的单边贡献和全局答案。

### 105. 从前序与中序遍历序列构造二叉树（Medium）

**题目描述**：给定 preorder 和 inorder 遍历序列（无重复元素），构造二叉树并返回根节点。序列长度<=3000。

**为什么属于这个类型**：二叉树天然递归定义，整棵树的问题通常可以拆成根节点、左子树、右子树的同类问题。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：前序定根，中序定左右子树边界：递归地在子区间内重复根-左-右的划分。

**套用统一模板**：空节点给 base case；递归求左右；根节点合并左右信息；必要时用全局变量记录不一定经过根的答案。

**基础模板代码**：

```python
class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        idx_map = {v: i for i, v in enumerate(inorder)}
        def build(pre_l, pre_r, in_l, in_r):
            if pre_l > pre_r:
                return None
            root_val = preorder[pre_l]
            root = TreeNode(root_val)
            in_idx = idx_map[root_val]
            left_size = in_idx - in_l
            root.left = build(pre_l + 1, pre_l + left_size, in_l, in_idx - 1)
            root.right = build(pre_l + left_size + 1, pre_r, in_idx + 1, in_r)
            return root
        return build(0, len(preorder) - 1, 0, len(inorder) - 1)
```

**信息流动 / 窗口变化 / 指针移动过程**：

前序遍历的第一个元素是根节点，在中序遍历中找到根节点位置，左边是左子树中序，右边是右子树中序。根据左子树大小可以在前序中划分出左子树前序和右子树前序，然后递归构造。用字典存储中序值到索引的映射实现O(1)查找根位置。注意递归参数的边界：前序起止和中序起止要对应。时间复杂度O(n)，空间复杂度O(n)。

**优化代码**：

优化重点是明确返回值和全局答案的区别，避免重复遍历子树。

**易错点**：BST 题用上下界；层序题用队列；构造题用哈希表定位中序下标；路径题区分返回给父节点的单边贡献和全局答案。

### 114. 二叉树展开为链表（Medium）

**题目描述**：给定二叉树的根节点，将其展开为单链表（所有节点右子节点指向下一个节点，左子节点为null），展开顺序与前序遍历一致。原地操作。

**为什么属于这个类型**：二叉树天然递归定义，整棵树的问题通常可以拆成根节点、左子树、右子树的同类问题。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：先序遍历的顺序重排节点：递归展开右子树后，将左子树插入根与右子树之间。

**套用统一模板**：空节点给 base case；递归求左右；根节点合并左右信息；必要时用全局变量记录不一定经过根的答案。

**基础模板代码**：

```python
class Solution:
    def flatten(self, root: Optional[TreeNode]) -> None:
        prev = None
        def dfs(node):
            nonlocal prev
            if not node:
                return
            dfs(node.right)
            dfs(node.left)
            node.right = prev
            node.left = None
            prev = node
        dfs(root)
```

**信息流动 / 窗口变化 / 指针移动过程**：

方法1：递归后序遍历（右-左-中），维护全局prev指针。先处理右子树再处理左子树，最后处理当前节点：将当前节点右指针指向prev，左指针置空，更新prev。这种逆前序的方式保证按前序遍历的逆序处理节点，每次将当前节点接在已处理链表前面。
方法2：迭代法，找到左子树最右节点，将右子树接过去，然后移动到左子树继续。
时间复杂度O(n)，空间复杂度O(h)或O(1)。

**优化代码**：

优化重点是明确返回值和全局答案的区别，避免重复遍历子树。

**易错点**：BST 题用上下界；层序题用队列；构造题用哈希表定位中序下标；路径题区分返回给父节点的单边贡献和全局答案。

### 199. 二叉树的右视图（Medium）

**题目描述**：给定二叉树的根节点，返回从右侧观察到的节点值（即每层最右边的节点值）。节点数<=100。

**为什么属于这个类型**：二叉树天然递归定义，整棵树的问题通常可以拆成根节点、左子树、右子树的同类问题。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：每层最后一个节点：BFS层序遍历中每层的最右节点，或DFS先右后左记录每层首次访问。

**套用统一模板**：空节点给 base case；递归求左右；根节点合并左右信息；必要时用全局变量记录不一定经过根的答案。

**基础模板代码**：

```python
class Solution:
    def rightSideView(self, root: Optional[TreeNode]) -> List[int]:
        if not root:
            return []
        from collections import deque
        res = []
        q = deque([root])
        while q:
            size = len(q)
            for i in range(size):
                node = q.popleft()
                if i == size - 1:
                    res.append(node.val)
                if node.left:
                    q.append(node.left)
                if node.right:
                    q.append(node.right)
        return res
```

**信息流动 / 窗口变化 / 指针移动过程**：

BFS层序遍历，每层最后一个节点即为右视图可见节点。也可以用DFS前序遍历（根-右-左），每层第一个被访问的节点就是右视图节点，用深度判断是否是该层第一个。BFS更直观：每层遍历完成后取最后一个值。时间复杂度O(n)，空间复杂度O(n)。

**优化代码**：

优化重点是明确返回值和全局答案的区别，避免重复遍历子树。

**易错点**：BST 题用上下界；层序题用队列；构造题用哈希表定位中序下标；路径题区分返回给父节点的单边贡献和全局答案。

### 297. 二叉树的序列化与反序列化（Hard）

**题目描述**：设计算法将二叉树序列化为字符串，并能将字符串反序列化为原始二叉树。不限制编码方式，但需要保证唯一对应。

**为什么属于这个类型**：二叉树天然递归定义，整棵树的问题通常可以拆成根节点、左子树、右子树的同类问题。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：序列化是遍历的逆操作：用先序遍历编码（含null标记），反序列化按相同顺序重建。

**套用统一模板**：空节点给 base case；递归求左右；根节点合并左右信息；必要时用全局变量记录不一定经过根的答案。

**基础模板代码**：

```python
class Codec:
    def serialize(self, root):
        vals = []
        def dfs(node):
            if not node:
                vals.append('None')
                return
            vals.append(str(node.val))
            dfs(node.left)
            dfs(node.right)
        dfs(root)
        return ','.join(vals)

    def deserialize(self, data):
        vals = iter(data.split(','))
        def dfs():
            val = next(vals)
            if val == 'None':
                return None
            node = TreeNode(int(val))
            node.left = dfs()
            node.right = dfs()
            return node
        return dfs()
```

**信息流动 / 窗口变化 / 指针移动过程**：

用前序遍历序列化：遇到null用特殊标记（如'None'），节点间用逗号分隔。反序列化时按前序递归重建：从列表头部取值，'None'则返回null，否则创建节点并递归构建左右子树。关键是null节点也要序列化，否则无法唯一确定树结构。BFS层序也可以，但前序递归更简洁。时间复杂度O(n)，空间复杂度O(n)。

**优化代码**：

优化重点是明确返回值和全局答案的区别，避免重复遍历子树。

**易错点**：BST 题用上下界；层序题用队列；构造题用哈希表定位中序下标；路径题区分返回给父节点的单边贡献和全局答案。

### 617. 合并二叉树（Easy）

**题目描述**：给定两个二叉树，将它们合并为一棵新树：重叠节点值相加，否则取非空节点。节点数<=2000。

**为什么属于这个类型**：二叉树天然递归定义，整棵树的问题通常可以拆成根节点、左子树、右子树的同类问题。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：两树对应位置节点值相加，缺位用对方节点补，递归同步遍历。

**套用统一模板**：空节点给 base case；递归求左右；根节点合并左右信息；必要时用全局变量记录不一定经过根的答案。

**基础模板代码**：

```python
class Solution:
    def mergeTrees(self, root1: Optional[TreeNode], root2: Optional[TreeNode]) -> Optional[TreeNode]:
        if not root1:
            return root2
        if not root2:
            return root1
        root1.val += root2.val
        root1.left = self.mergeTrees(root1.left, root2.left)
        root1.right = self.mergeTrees(root1.right, root2.right)
        return root1
```

**信息流动 / 窗口变化 / 指针移动过程**：

递归同步遍历两棵树：若两节点都存在，值相加作为新节点值，递归合并左右子树；若其中一个为null，返回另一个（整个子树直接使用）。可以原地修改root1以节省空间，也可创建新节点。时间复杂度O(min(m,n))，空间复杂度O(min(m,n))。

**优化代码**：

优化重点是明确返回值和全局答案的区别，避免重复遍历子树。

**易错点**：BST 题用上下界；层序题用队列；构造题用哈希表定位中序下标；路径题区分返回给父节点的单边贡献和全局答案。

### 437. 路径总和III（Medium）

**题目描述**：给定二叉树的根节点和整数targetSum，求路径总和等于targetSum的路径数目。路径不需要从根开始也不需要在叶子结束，但必须向下。节点数<=1000。

**为什么属于这个类型**：二叉树天然递归定义，整棵树的问题通常可以拆成根节点、左子树、右子树的同类问题。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：前缀和+哈希：树上路径和问题转化为前缀和的差，用哈希表记录祖先前缀和的频次。

**套用统一模板**：空节点给 base case；递归求左右；根节点合并左右信息；必要时用全局变量记录不一定经过根的答案。

**基础模板代码**：

```python
class Solution:
    def pathSum(self, root: Optional[TreeNode], targetSum: int) -> int:
        from collections import defaultdict
        prefix = defaultdict(int)
        prefix[0] = 1
        self.count = 0
        def dfs(node, curr_sum):
            if not node:
                return
            curr_sum += node.val
            self.count += prefix[curr_sum - targetSum]
            prefix[curr_sum] += 1
            dfs(node.left, curr_sum)
            dfs(node.right, curr_sum)
            prefix[curr_sum] -= 1
        dfs(root, 0)
        return self.count
```

**信息流动 / 窗口变化 / 指针移动过程**：

前缀和+DFS。路径从节点a到节点b（a是b的祖先）的和 = b的前缀和 - a的前缀和。用字典记录从根到当前节点路径上各前缀和出现的次数。DFS遍历时：1) 当前前缀和curr_sum += node.val；2) 检查curr_sum - targetSum是否在字典中，有多少个就有多少条以当前节点结尾的满足条件的路径；3) 将curr_sum加入字典；4) 递归左右子树；5) 回溯时将curr_sum从字典中移除。类似数组的前缀和思路，从一维扩展到树路径。时间复杂度O(n)，空间复杂度O(n)。

**优化代码**：

优化重点是明确返回值和全局答案的区别，避免重复遍历子树。

**易错点**：BST 题用上下界；层序题用队列；构造题用哈希表定位中序下标；路径题区分返回给父节点的单边贡献和全局答案。

## 局部最优的累积 (贪心)

### 如何识别这是贪心题

贪心适用的关键不是“看起来能贪”，而是能证明局部选择不会让全局变差。识别信号是：

1. 每一步可以做一个局部决策。
2. 当前最优选择可以通过交换论证替代其他选择。
3. 只需要维护少量边界、余额、最远位置、当前区间。
4. 不需要回头修改大量历史状态。

判断公式：

> 如果能证明“当前这样选，未来一定不吃亏”，就考虑贪心；证明不了时，不要硬贪，考虑 DP 或搜索。

### 一句话本质

证明当前选择不可替代，再累积局部最优。

### 第一性原理

贪心的第一性原理是：如果某个局部选择在任何全局最优解中都可以被替换成不更差的选择，那么每一步做这个选择就能得到全局最优。

### 统一模板：先问 5 个问题

1. 当前局部决策是什么？
2. 为什么这个选择不会破坏未来？
3. 有没有交换论证或反证法支持？
4. 状态只需要维护当前最优边界还是完整历史？
5. 失败条件什么时候出现？

### 通用代码骨架

```python
state = 初始
for item in 按策略排序或原序遍历:
    if 当前局部选择可行:
        接受并更新 state
    else:
        修正或返回失败
return 答案
```

### 常见状态变量

- 最远可达位置
- 当前收益/最低价格
- 区间右边界
- 排序后的局部约束

### 常见剪枝 / 优化

- 一旦当前位置超过最远可达，直接失败
- 区间已覆盖则合并，不重开新区间
- 局部约束不满足时只做必要修正

### 本类题目总览

- 55. 跳跃游戏（Medium）
- 45. 跳跃游戏II（Medium）
- 121. 买卖股票的最佳时机（Easy）
- 122. 买卖股票的最佳时机II（Medium）
- 763. 划分字母区间（Medium）
- 135. 分发糖果（Hard）
- 406. 根据身高重建队列（Medium）
- 56. 合并区间（Medium）

### 55. 跳跃游戏（Medium）

**题目描述**：给定非负整数数组nums，初始在第一个位置，每个元素代表该位置可跳跃的最大步数，判断是否能到达最后一个位置。数组长度<=10^4。

**为什么属于这个类型**：题目存在可以被证明安全的局部选择，局部最优累积后得到全局最优。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：维护当前可达的最远位置：如果某个位置超出了最远可达范围，则不可达；否则不断扩展最远范围。

**套用统一模板**：维护当前最优边界或资源状态，每步做不可替代的局部选择。

**基础模板代码**：

```python
class Solution:
    def canJump(self, nums: List[int]) -> bool:
        max_reach = 0
        for i, num in enumerate(nums):
            if i > max_reach:
                return False
            max_reach = max(max_reach, i + num)
            if max_reach >= len(nums) - 1:
                return True
        return True
```

**信息流动 / 窗口变化 / 指针移动过程**：

贪心：维护当前能到达的最远位置max_reach。遍历数组，若当前下标i > max_reach则说明无法到达位置i，返回False；否则更新max_reach = max(max_reach, i + nums[i])。若max_reach >= 最后下标则可到达。关键洞察：不需要模拟每一步跳跃，只需看最远能跳到哪里。时间复杂度O(n)，空间复杂度O(1)。

**优化代码**：

优化重点是排序、边界合并、最远可达、最低成本等状态压缩。

**易错点**：通常通过排序、维护最远边界、最低成本或区间合并来减少状态；关键是证明不是拍脑袋选。

### 45. 跳跃游戏II（Medium）

**题目描述**：给定非负整数数组nums，初始在第一个位置，每个元素代表该位置可跳跃的最大步数，返回到达最后一个位置的最少跳跃次数。假设总能到达。

**为什么属于这个类型**：题目存在可以被证明安全的局部选择，局部最优累积后得到全局最优。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：贪心跳跃：每步在当前可达范围内选择能使下一步跳得最远的位置，最少步数等于必要的跳跃次数。

**套用统一模板**：维护当前最优边界或资源状态，每步做不可替代的局部选择。

**基础模板代码**：

```python
class Solution:
    def jump(self, nums: List[int]) -> int:
        jumps = 0
        end = 0
        farthest = 0
        for i in range(len(nums) - 1):
            farthest = max(farthest, i + nums[i])
            if i == end:
                jumps += 1
                end = farthest
        return jumps
```

**信息流动 / 窗口变化 / 指针移动过程**：

贪心BFS思想：将跳跃过程看作逐层扩展。维护当前层的边界end和下一层的最远位置farthest。遍历到end时，说明当前层所有位置都处理完了，跳跃次数+1，更新end = farthest进入下一层。不需要逐个位置模拟跳跃，而是按层统计跳跃次数。保证每个位置最多访问一次。时间复杂度O(n)，空间复杂度O(1)。

**优化代码**：

优化重点是排序、边界合并、最远可达、最低成本等状态压缩。

**易错点**：通常通过排序、维护最远边界、最低成本或区间合并来减少状态；关键是证明不是拍脑袋选。

### 121. 买卖股票的最佳时机（Easy）

**题目描述**：给定股票每天价格的数组，选择某一天买入并在之后某一天卖出，求最大利润。价格数组长度<=10^5。

**为什么属于这个类型**：题目存在可以被证明安全的局部选择，局部最优累积后得到全局最优。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：最大利润=最高卖出价-之前的最低买入价，遍历时维护历史最低价。

**套用统一模板**：维护当前最优边界或资源状态，每步做不可替代的局部选择。

**基础模板代码**：

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        min_price = float('inf')
        max_profit = 0
        for p in prices:
            min_price = min(min_price, p)
            max_profit = max(max_profit, p - min_price)
        return max_profit
```

**信息流动 / 窗口变化 / 指针移动过程**：

一次遍历：维护到当前位置的最低价格min_price，当前利润 = prices[i] - min_price，更新最大利润。本质是对于每个卖出日，最优买入日是其之前价格最低的那天。不需要双重循环，一次遍历即可。时间复杂度O(n)，空间复杂度O(1)。

**优化代码**：

优化重点是排序、边界合并、最远可达、最低成本等状态压缩。

**易错点**：通常通过排序、维护最远边界、最低成本或区间合并来减少状态；关键是证明不是拍脑袋选。

### 122. 买卖股票的最佳时机II（Medium）

**题目描述**：给定股票每天价格的数组，可以进行多次交易（买入后必须卖出才能再次买入），求最大利润。价格数组长度<=3*10^4。

**为什么属于这个类型**：题目存在可以被证明安全的局部选择，局部最优累积后得到全局最优。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：可以多次交易时，所有上涨段的利润之和就是最大利润，贪心捕获每一段上升。

**套用统一模板**：维护当前最优边界或资源状态，每步做不可替代的局部选择。

**基础模板代码**：

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        profit = 0
        for i in range(1, len(prices)):
            if prices[i] > prices[i-1]:
                profit += prices[i] - prices[i-1]
        return profit
```

**信息流动 / 窗口变化 / 指针移动过程**：

贪心：只要今天比昨天涨了就交易（累加所有上涨区间），等价于在所有上升段买入卖出。数学证明：多次交易的最优解 = 所有相邻两天正差值之和。因为任何跨越多天的上涨都可以拆分为逐日交易，利润相同。不需要找波峰波谷。时间复杂度O(n)，空间复杂度O(1)。

**优化代码**：

优化重点是排序、边界合并、最远可达、最低成本等状态压缩。

**易错点**：通常通过排序、维护最远边界、最低成本或区间合并来减少状态；关键是证明不是拍脑袋选。

### 763. 划分字母区间（Medium）

**题目描述**：给定字符串S，将其划分为尽可能多的片段，使得同一字母最多出现在一个片段中，返回每个片段的长度。S长度<=500，仅小写字母。

**为什么属于这个类型**：题目存在可以被证明安全的局部选择，局部最优累积后得到全局最优。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：每个字母的最后出现位置决定了包含该字母的片段的最小右边界，贪心扩展到边界处切割。

**套用统一模板**：维护当前最优边界或资源状态，每步做不可替代的局部选择。

**基础模板代码**：

```python
class Solution:
    def partitionLabels(self, s: str) -> List[int]:
        last_position = {}
        for index, char in enumerate(s):
            last_position[char] = index

        result = []
        start = 0
        end = 0

        for index, char in enumerate(s):
            char_last_position = last_position[char]
            if char_last_position > end:
                end = char_last_position

            if index == end:
                part_len = end - start + 1
                result.append(part_len)
                start = index + 1

        return result
```

**信息流动 / 窗口变化 / 指针移动过程**：

贪心。第一步：统计每个字母的最后出现位置last[c]。第二步：遍历字符串，维护当前片段的end = max(end, last[S[i]])，当遍历到end位置时，说明当前片段中所有字母的最后出现位置都已包含在内，可以在此处切分。end初始为0，每遇到一个字符就扩展end，当i == end时记录片段长度并重置。时间复杂度O(n)，空间复杂度O(1)（26个字母）。

**优化代码**：

优化重点是排序、边界合并、最远可达、最低成本等状态压缩。

**易错点**：通常通过排序、维护最远边界、最低成本或区间合并来减少状态；关键是证明不是拍脑袋选。

### 135. 分发糖果（Hard）

**题目描述**：n个孩子站成一排，ratings数组表示每个孩子的评分。每个孩子至少分1颗糖果，相邻孩子中评分高的必须分更多糖果。求最少糖果总数。n<=2*10^4。

**为什么属于这个类型**：题目存在可以被证明安全的局部选择，局部最优累积后得到全局最优。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：两次遍历分别满足左规则和右规则：从左到右保证右边评分高的糖果更多，从右到左保证左边评分高的糖果更多，取两者最大值。

**套用统一模板**：维护当前最优边界或资源状态，每步做不可替代的局部选择。

**基础模板代码**：

```python
class Solution:
    def candy(self, ratings: List[int]) -> int:
        n = len(ratings)
        candies = [1] * n
        for i in range(1, n):
            if ratings[i] > ratings[i-1]:
                candies[i] = candies[i-1] + 1
        for i in range(n-2, -1, -1):
            if ratings[i] > ratings[i+1]:
                candies[i] = max(candies[i], candies[i+1] + 1)
        return sum(candies)
```

**信息流动 / 窗口变化 / 指针移动过程**：

两次遍历。从左到右：若ratings[i] > ratings[i-1]则candies[i] = candies[i-1] + 1，保证右边评分高的比左边多。从右到左：若ratings[i] > ratings[i+1]则candies[i] = max(candies[i], candies[i+1] + 1)，保证左边评分高的比右边多。取两次遍历的最大值，同时满足左右两边的约束。初始化全为1。时间复杂度O(n)，空间复杂度O(n)。

**优化代码**：

优化重点是排序、边界合并、最远可达、最低成本等状态压缩。

**易错点**：通常通过排序、维护最远边界、最低成本或区间合并来减少状态；关键是证明不是拍脑袋选。

### 406. 根据身高重建队列（Medium）

**题目描述**：people[i] = [h_i, k_i]表示第i个人的身高和前面身高大于等于自己的人数，重建队列使其满足条件。人数<=2000。

**为什么属于这个类型**：题目存在可以被证明安全的局部选择，局部最优累积后得到全局最优。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：先排高的人（不受矮人影响），再逐个插入矮的人到指定位置：高人已固定，矮人插入不影响高人的排数。

**套用统一模板**：维护当前最优边界或资源状态，每步做不可替代的局部选择。

**基础模板代码**：

```python
class Solution:
    def reconstructQueue(self, people: List[List[int]]) -> List[List[int]]:
        people.sort(key=lambda x: (-x[0], x[1]))
        res = []
        for p in people:
            res.insert(p[1], p)
        return res
```

**信息流动 / 窗口变化 / 指针移动过程**：

贪心。先按身高降序、k升序排列。然后依次将每个人插入结果列表的第k个位置。为什么这样做正确？身高高的人先插入，其k值就是它的绝对位置；身高矮的人后插入，不影响已插入的高个子前面的计数（矮个子不计入高个子的k值），且矮个子的k值只需要在前面的序列中找到正确位置即可。每次插入用list.insert操作。时间复杂度O(n^2)（insert操作），空间复杂度O(n)。

**优化代码**：

优化重点是排序、边界合并、最远可达、最低成本等状态压缩。

**易错点**：通常通过排序、维护最远边界、最低成本或区间合并来减少状态；关键是证明不是拍脑袋选。

### 56. 合并区间（Medium）

**题目描述**：给定区间的集合intervals，合并所有重叠区间。区间数<=10^4，每个区间[start_i, end_i]满足start_i <= end_i。

**为什么属于这个类型**：题目存在可以被证明安全的局部选择，局部最优累积后得到全局最优。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：排序后贪心合并：按左端点排序，相邻区间有重叠则合并为并集，无重叠则新区间开始。

**套用统一模板**：维护当前最优边界或资源状态，每步做不可替代的局部选择。

**基础模板代码**：

```python
class Solution:
    def merge(self, intervals: List[List[int]]) -> List[List[int]]:
        intervals.sort(key=lambda x: x[0])
        res = []
        for interval in intervals:
            if res and interval[0] <= res[-1][1]:
                res[-1][1] = max(res[-1][1], interval[1])
            else:
                res.append(interval)
        return res
```

**信息流动 / 窗口变化 / 指针移动过程**：

排序+贪心。先按区间左端点排序，然后遍历：若当前区间左端点 <= 结果列表最后一个区间的右端点，则重叠，合并（更新右端点为max）；否则不重叠，直接加入结果列表。排序保证所有可合并的区间一定相邻，一次遍历即可完成合并。时间复杂度O(n log n)，空间复杂度O(n)。

**优化代码**：

优化重点是排序、边界合并、最远可达、最低成本等状态压缩。

**易错点**：通常通过排序、维护最远边界、最低成本或区间合并来减少状态；关键是证明不是拍脑袋选。

## 区间的代数化 (前缀和/哈希)

### 如何识别这是前缀和 / 哈希题

前缀和适用于区间累计，哈希适用于快速查找历史状态。识别信号是：

1. 题目问子数组和、区间和、连续段满足某个目标。
2. 区间条件可以写成两个前缀状态的差。
3. 需要判断过去是否出现过某个值、签名或节点。
4. 暴力枚举所有区间是 O(n²)，但历史状态可查表。

判断公式：

> 如果当前答案依赖“过去是否出现过某个状态”，就考虑哈希；如果是连续区间累计，就把它前缀化。

### 一句话本质

把区间问题转化为端点差和查表。

### 第一性原理

前缀和的第一性原理是：区间信息可以由两个前缀状态相减得到；哈希表的第一性原理是：把寻找过去某个状态从线性扫描变成 O(1) 查找。

### 统一模板：先问 5 个问题

1. 当前前缀状态是什么？和、乘积、频次还是签名？
2. 目标区间条件能否写成 `prefix[j] - prefix[i] = k`？
3. 哈希表存的是值、下标、次数还是对象节点？
4. 查询发生在更新哈希表之前还是之后？
5. 是否需要处理 0、负数、重复键？

### 通用代码骨架

```python
seen = {初始前缀: 次数或位置}
prefix = 0
ans = 0
for x in nums:
    prefix += x
    ans += seen.get(prefix - target, 0)
    seen[prefix] = seen.get(prefix, 0) + 1
return ans
```

### 常见状态变量

- `prefix`：当前位置之前的累计信息
- `dict`：历史前缀状态到次数/位置的映射
- 签名 tuple/string：异位词分组等结构化哈希键

### 常见剪枝 / 优化

- 先查再存，避免把当前前缀当成过去
- 缓存容量超限时淘汰最久未使用项
- 频次为 0 时删除键保持状态干净

### 本类题目总览

- 1. 两数之和（Easy）
- 560. 和为K的子数组（Medium）
- 49. 字母异位词分组（Medium）
- 238. 除自身以外数组的乘积（Medium）
- 146. LRU缓存（Medium）
- 347. 前K个高频元素（Medium）

### 1. 两数之和（Easy）

**题目描述**：给定整数数组nums和目标值target，找出数组中和为目标值的两个整数的下标。每个输入只对应一个答案，同一元素不能使用两次。数组长度<=10^4。

**为什么属于这个类型**：题目需要快速查询过去是否出现过某个状态，或把区间条件转化为两个前缀状态的代数关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：查找补数：遍历时用哈希表记录已见数字，O(1)查询是否存在target-当前数的补数。

**套用统一模板**：边遍历边维护当前前缀；先查询需要的历史前缀，再把当前前缀加入表。

**基础模板代码**：

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        seen = {}
        for i, num in enumerate(nums):
            if target - num in seen:
                return [seen[target - num], i]
            seen[num] = i
        return []
```

**信息流动 / 窗口变化 / 指针移动过程**：

哈希表一次遍历。遍历数组，对于每个元素nums[i]，检查target - nums[i]是否在哈希表中：若在则返回两个下标；否则将nums[i]及其下标存入哈希表。关键：先查找再插入，避免同一元素被使用两次。哈希表查找O(1)，总时间复杂度O(n)，空间复杂度O(n)。

**优化代码**：

优化重点是选择正确哈希键、先查后存、删除无效键或维护双向链表。

**易错点**：计数题存次数，最短/最长题存位置，分组题存标准化签名，缓存题存键到节点的映射。

### 560. 和为K的子数组（Medium）

**题目描述**：给定整数数组nums和整数k，统计数组中连续子数组和为k的个数。数组长度<=2*10^4，元素可为负数。

**为什么属于这个类型**：题目需要快速查询过去是否出现过某个状态，或把区间条件转化为两个前缀状态的代数关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：前缀和之差等于K：遍历时记录每个前缀和出现次数，查找prefix-K是否存在。

**套用统一模板**：边遍历边维护当前前缀；先查询需要的历史前缀，再把当前前缀加入表。

**基础模板代码**：

```python
class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        from collections import defaultdict
        count = defaultdict(int)
        count[0] = 1
        curr = 0
        res = 0
        for num in nums:
            curr += num
            res += count[curr - k]
            count[curr] += 1
        return res
```

**信息流动 / 窗口变化 / 指针移动过程**：

前缀和+哈希表。子数组sum[i..j] = prefix[j] - prefix[i-1] = k，即prefix[j] - k = prefix[i-1]。遍历时用哈希表记录各前缀和出现的次数，对于当前前缀和curr，查找curr - k出现几次就对应几个和为k的子数组。注意初始化prefix[0] = 0出现1次。元素可为负数导致前缀和不单调，不能用双指针。时间复杂度O(n)，空间复杂度O(n)。

**优化代码**：

优化重点是选择正确哈希键、先查后存、删除无效键或维护双向链表。

**易错点**：计数题存次数，最短/最长题存位置，分组题存标准化签名，缓存题存键到节点的映射。

### 49. 字母异位词分组（Medium）

**题目描述**：给定字符串数组strs，将字母异位词（由相同字母重新排列组成的单词）分组。strs长度<=10^4，每个串长度<=100。

**为什么属于这个类型**：题目需要快速查询过去是否出现过某个状态，或把区间条件转化为两个前缀状态的代数关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：异位词的排序后字符串相同（或字母计数相同），以此作为哈希键分组。

**套用统一模板**：边遍历边维护当前前缀；先查询需要的历史前缀，再把当前前缀加入表。

**基础模板代码**：

```python
class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        from collections import defaultdict
        groups = defaultdict(list)
        for s in strs:
            key = ''.join(sorted(s))
            groups[key].append(s)
        return list(groups.values())
```

**信息流动 / 窗口变化 / 指针移动过程**：

字母异位词排序后字符串相同，或字符计数相同。方法1：排序法，将每个字符串按字母排序作为key，O(k log k)每串。方法2：计数法，用26个字母的计数元组作为key，O(k)每串。两种方法都用哈希表按key分组。排序法更简洁，计数法在字符串较长时更快。时间复杂度O(n*k log k)或O(n*k)，空间复杂度O(n*k)。

**优化代码**：

优化重点是选择正确哈希键、先查后存、删除无效键或维护双向链表。

**易错点**：计数题存次数，最短/最长题存位置，分组题存标准化签名，缓存题存键到节点的映射。

### 238. 除自身以外数组的乘积（Medium）

**题目描述**：给定一个整数数组，返回数组中除自身以外其余各元素的乘积。要求时间复杂度O(n)且不使用除法，额外空间O(1)（输出数组不计入）。

**为什么属于这个类型**：题目需要快速查询过去是否出现过某个状态，或把区间条件转化为两个前缀状态的代数关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：结果[i]=左侧所有元素的乘积×右侧所有元素的乘积，两次遍历分别累积左右乘积。

**套用统一模板**：边遍历边维护当前前缀；先查询需要的历史前缀，再把当前前缀加入表。

**基础模板代码**：

```python
def productExceptSelf(self, nums):
    n = len(nums)
    result = [1] * n
    # 第一遍：左积累积
    left = 1
    for i in range(n):
        result[i] = left
        left *= nums[i]
    # 第二遍：右积累积，与左积相乘
    right = 1
    for i in range(n-1, -1, -1):
        result[i] *= right
        right *= nums[i]
    return result
```

**信息流动 / 窗口变化 / 指针移动过程**：

核心约束是不用除法，因此需要用前后缀乘积来替代。设左侧累乘数组L[i]表示nums[0..i-1]的乘积，右侧累乘数组R[i]表示nums[i+1..n-1]的乘积，则answer[i]=L[i]*R[i]。优化空间：先从左到右将L[i]填入answer，再用一个变量从右到左累乘R部分直接乘到answer上，即可做到除输出数组外O(1)空间。两趟遍历，时间O(n)，空间O(1)（不计输出）。

**优化代码**：

优化重点是选择正确哈希键、先查后存、删除无效键或维护双向链表。

**易错点**：计数题存次数，最短/最长题存位置，分组题存标准化签名，缓存题存键到节点的映射。

### 146. LRU缓存（Medium）

**题目描述**：设计一个满足LRU（最近最少使用）缓存淘汰机制的数据结构，支持get和put操作，均要求O(1)时间复杂度。容量满时淘汰最久未使用的键。

**为什么属于这个类型**：题目需要快速查询过去是否出现过某个状态，或把区间条件转化为两个前缀状态的代数关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：哈希表+双向链表：哈希表O(1)查找，链表O(1)调整访问顺序，最近访问移到头部，满时淘汰尾部。

**套用统一模板**：边遍历边维护当前前缀；先查询需要的历史前缀，再把当前前缀加入表。

**基础模板代码**：

```python
class Node:
    def __init__(self, key=0, val=0):
        self.key = key
        self.val = val
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache = {}
        self.head = Node()
        self.tail = Node()
        self.head.next = self.tail
        self.tail.prev = self.head

    def _remove(self, node):
        prev_node = node.prev
        next_node = node.next
        prev_node.next = next_node
        next_node.prev = prev_node

    def _add_to_front(self, node):
        first_node = self.head.next
        node.prev = self.head
        node.next = first_node
        self.head.next = node
        first_node.prev = node

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1

        node = self.cache[key]
        self._remove(node)
        self._add_to_front(node)
        return node.val

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            old_node = self.cache[key]
            self._remove(old_node)

        node = Node(key, value)
        self.cache[key] = node
        self._add_to_front(node)

        if len(self.cache) > self.capacity:
            lru_node = self.tail.prev
            self._remove(lru_node)
            del self.cache[lru_node.key]
```

**信息流动 / 窗口变化 / 指针移动过程**：

O(1)的get和put要求用哈希表定位，而LRU淘汰需要维护访问顺序，因此采用哈希表+双向链表的经典组合。哈希表key→链表节点实现O(1)查找，双向链表按访问时间排序（最近访问移到头部，尾部为最久未使用）。get命中时将节点移到头部；put时若key已存在则更新值并移到头部，若不存在则新建节点插入头部，超容量则删除尾部节点。使用哨兵头尾节点简化边界处理。

**优化代码**：

优化重点是选择正确哈希键、先查后存、删除无效键或维护双向链表。

**易错点**：计数题存次数，最短/最长题存位置，分组题存标准化签名，缓存题存键到节点的映射。

### 347. 前K个高频元素（Medium）

**题目描述**：给定整数数组和一个整数k，返回出现频率前k高的元素。要求时间复杂度优于O(nlogn)。

**为什么属于这个类型**：题目需要快速查询过去是否出现过某个状态，或把区间条件转化为两个前缀状态的代数关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：哈希统计频率+堆/桶排序取TopK：统计频次后，用最小堆维护前K个高频元素。

**套用统一模板**：边遍历边维护当前前缀；先查询需要的历史前缀，再把当前前缀加入表。

**基础模板代码**：

```python
def topKFrequent(self, nums, k):
    from collections import Counter
    import heapq
    count = Counter(nums)
    # 最小堆维护前K个高频元素
    heap = []
    for num, freq in count.items():
        heapq.heappush(heap, (freq, num))
        if len(heap) > k:
            heapq.heappop(heap)  # 弹出频率最低的
    return [num for freq, num in heap]
```

**信息流动 / 窗口变化 / 指针移动过程**：

第一步用哈希表统计频率O(n)。第二步取前k高频有三种策略：(1)最小堆维护大小k的堆O(nlogk)；(2)快速选择算法基于partition找第k大O(n)平均；(3)桶排序，以频率为下标建立桶数组，频率最高不超过n，从高频桶往低频桶收集O(n)。桶排序方案严格O(n)且实现简洁：建立n+1个桶，将频率为f的元素放入桶[f]，从后往前收集直到满k个。

**优化代码**：

优化重点是选择正确哈希键、先查后存、删除无效键或维护双向链表。

**易错点**：计数题存次数，最短/最长题存位置，分组题存标准化签名，缓存题存键到节点的映射。

## 指针的操控 (链表)

### 如何识别这是链表指针题

链表题的核心限制是不能随机访问，只能沿 `next` 走。识别信号是：

1. 输入是链表节点。
2. 需要删除、反转、合并、重排、找环、找交点。
3. 操作目标是节点连接关系，而不是数组下标。
4. 需要处理头节点变化。

判断公式：

> 如果答案依赖节点之间的连接关系变化，就画指针图，用 `dummy/prev/cur/nxt` 解决。

### 一句话本质

通过重定向指针改变节点关系。

### 第一性原理

链表的第一性原理是：节点只知道相邻节点，不能随机访问。因此所有操作本质都是保存必要后继、断开旧连接、建立新连接。

### 统一模板：先问 5 个问题

1. 需要操作的是节点本身还是节点值？
2. 是否需要虚拟头节点统一头部修改？
3. 要保存哪些指针避免链断掉？
4. 快慢指针能否定位目标位置或环入口？
5. 反转/合并/删除后尾部如何接回？

### 通用代码骨架

```python
dummy = ListNode(0)
dummy.next = head
prev = dummy
cur = head
while cur:
    nxt = cur.next
    执行指针重连
    prev, cur = 更新
return dummy.next
```

### 常见状态变量

- `prev/cur/nxt`：局部重连三件套
- `slow/fast`：距离差、环检测、中点定位
- `dummy`：简化删除头节点和合并链表
- `tail`：维护新链表末尾

### 常见剪枝 / 优化

- 空链表或单节点直接返回
- 删除倒数节点用快慢指针保持固定间距
- 环题快慢相遇后再定位入口

### 本类题目总览

- 206. 反转链表（Easy）
- 141. 环形链表（Easy）
- 21. 合并两个有序链表（Easy）
- 2. 两数相加（Medium）
- 19. 删除链表的倒数第N个结点（Medium）
- 160. 相交链表（Easy）
- 148. 排序链表（Medium）
- 234. 回文链表（Easy）
- 142. 环形链表II（Medium）
- 92. 反转链表II（Medium）
- 25. K个一组翻转链表（Hard）
- 138. 随机链表的复制（Medium）
- 143. 重排链表（Medium）

### 206. 反转链表（Easy）

**题目描述**：给定单链表的头节点，反转链表并返回反转后的头节点。迭代和递归两种方式均要求掌握。

**为什么属于这个类型**：题目操作对象是链表节点，不能随机访问，核心是改变或利用 `next/random` 指针关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：逐节点翻转指针方向：用prev和curr双指针，每次将curr.next指向prev，然后双指针前进。

**套用统一模板**：用 dummy 统一头节点变化；每次修改前先保存后继；反转/删除/合并后保证链不断。

**基础模板代码**：

```python
def reverseList(self, head):
    prev = None
    curr = head
    while curr:
        next_temp = curr.next  # 暂存下一个节点
        curr.next = prev       # 翻转指针
        prev = curr            # prev前进
        curr = next_temp       # curr前进
    return prev  # prev是新的头节点
```

**信息流动 / 窗口变化 / 指针移动过程**：

迭代法：用三个指针prev、curr、next，遍历链表逐个反转指针方向。prev初始为None，curr指向head，每步保存curr.next，将curr.next指向prev，然后prev和curr各前进一步。当curr为None时prev即为新头。递归法：递归到链表末尾，回溯时将next.next指向当前节点，当前节点.next置None，逐层返回新头。两种方法时间O(n)，迭代空间O(1)，递归空间O(n)。

**优化代码**：

优化重点是 dummy 简化头节点、快慢指针定位、原地反转或哈希映射复杂指针。

**易错点**：快慢指针定位中点、倒数第 N、环入口；归并排序适合链表排序；复杂节点复制可用哈希表或原地穿插。

### 141. 环形链表（Easy）

**题目描述**：给定单链表头节点，判断链表中是否存在环。要求O(1)额外空间。

**为什么属于这个类型**：题目操作对象是链表节点，不能随机访问，核心是改变或利用 `next/random` 指针关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：快慢指针追及：快指针每次走2步，慢指针走1步，若有环必在某处相遇。

**套用统一模板**：用 dummy 统一头节点变化；每次修改前先保存后继；反转/删除/合并后保证链不断。

**基础模板代码**：

```python
def hasCycle(self, head):
    slow = head
    fast = head
    while fast and fast.next:
        slow = slow.next       # 慢指针走1步
        fast = fast.next.next  # 快指针走2步
        if slow == fast:       # 相遇则有环
            return True
    return False
```

**信息流动 / 窗口变化 / 指针移动过程**：

快慢指针法（Floyd判圈）：慢指针每次走一步，快指针每次走两步。若存在环，快指针终将追上慢指针（因为每步距离差缩小1，一定在环内相遇）；若无环，快指针先到尾部None。时间O(n)，空间O(1)。也可用哈希表记录已访问节点，时间O(n)但空间O(n)。面试推荐快慢指针。

**优化代码**：

优化重点是 dummy 简化头节点、快慢指针定位、原地反转或哈希映射复杂指针。

**易错点**：快慢指针定位中点、倒数第 N、环入口；归并排序适合链表排序；复杂节点复制可用哈希表或原地穿插。

### 21. 合并两个有序链表（Easy）

**题目描述**：将两个升序链表合并为一个新的升序链表并返回。新链表由原链表节点拼接而成。

**为什么属于这个类型**：题目操作对象是链表节点，不能随机访问，核心是改变或利用 `next/random` 指针关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：双指针归并：每次取较小头节点接入结果链，递归或迭代实现。

**套用统一模板**：用 dummy 统一头节点变化；每次修改前先保存后继；反转/删除/合并后保证链不断。

**基础模板代码**：

```python
def mergeTwoLists(self, l1, l2):
    dummy = ListNode(0)
    tail = dummy
    while l1 and l2:
        if l1.val <= l2.val:
            tail.next = l1
            l1 = l1.next
        else:
            tail.next = l2
            l2 = l2.next
        tail = tail.next
    # 拼接剩余部分
    tail.next = l1 or l2
    return dummy.next
```

**信息流动 / 窗口变化 / 指针移动过程**：

迭代法：用哨兵节点dummy简化头节点处理，用tail指针追踪合并链表尾部。比较两个链表当前节点值，较小者接入tail，对应指针后移，直到某一链表遍历完，再将剩余部分接上。递归法：比较两链表头节点值，较小者的next指向递归合并剩余部分的结果。两者时间O(n+m)，迭代空间O(1)，递归空间O(n+m)。

**优化代码**：

优化重点是 dummy 简化头节点、快慢指针定位、原地反转或哈希映射复杂指针。

**易错点**：快慢指针定位中点、倒数第 N、环入口；归并排序适合链表排序；复杂节点复制可用哈希表或原地穿插。

### 2. 两数相加（Medium）

**题目描述**：两个非负整数分别以逆序存储在链表中（每个节点存一位数字），将两数相加并以同样形式返回链表。两数均不包含前导零，数字位数可达100位。

**为什么属于这个类型**：题目操作对象是链表节点，不能随机访问，核心是改变或利用 `next/random` 指针关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：链表逐位相加带进位：对应位相加+进位，注意最后可能还有进位。

**套用统一模板**：用 dummy 统一头节点变化；每次修改前先保存后继；反转/删除/合并后保证链不断。

**基础模板代码**：

```python
def addTwoNumbers(self, l1, l2):
    dummy = ListNode(0)
    current = dummy
    carry = 0

    while l1 or l2 or carry:
        if l1:
            val1 = l1.val
        else:
            val1 = 0

        if l2:
            val2 = l2.val
        else:
            val2 = 0

        total = val1 + val2 + carry
        carry = total // 10
        digit = total % 10
        current.next = ListNode(digit)
        current = current.next

        if l1:
            l1 = l1.next
        else:
            l1 = None

        if l2:
            l2 = l2.next
        else:
            l2 = None

    return dummy.next
```

**信息流动 / 窗口变化 / 指针移动过程**：

同时遍历两条链表，逐位相加并维护进位carry。设当前位和为sum=l1.val+l2.val+carry，新节点值为sum%10，carry=sum//10。若某链表已遍历完则视其值为0。遍历结束后若carry>0则额外添加一个节点。注意链表是逆序存储，所以个位在头部，天然对齐无需翻转。时间O(max(m,n))，空间O(max(m,n))（结果链表）。

**优化代码**：

优化重点是 dummy 简化头节点、快慢指针定位、原地反转或哈希映射复杂指针。

**易错点**：快慢指针定位中点、倒数第 N、环入口；归并排序适合链表排序；复杂节点复制可用哈希表或原地穿插。

### 19. 删除链表的倒数第N个结点（Medium）

**题目描述**：给定链表头节点和整数n，删除链表的倒数第n个节点并返回头节点。要求一趟扫描完成，n保证有效。

**为什么属于这个类型**：题目操作对象是链表节点，不能随机访问，核心是改变或利用 `next/random` 指针关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：快慢指针定位：快指针先走N步，然后快慢同速前进，快到末尾时慢恰好在倒数第N+1个位置。

**套用统一模板**：用 dummy 统一头节点变化；每次修改前先保存后继；反转/删除/合并后保证链不断。

**基础模板代码**：

```python
def removeNthFromEnd(self, head, n):
    dummy = ListNode(0, head)
    fast = dummy
    slow = dummy
    # 快指针先走n步
    for _ in range(n):
        fast = fast.next
    # 同速前进，快到末尾时慢在倒数第n+1个
    while fast.next:
        fast = fast.next
        slow = slow.next
    slow.next = slow.next.next  # 删除倒数第n个
    return dummy.next
```

**信息流动 / 窗口变化 / 指针移动过程**：

双指针法：先让快指针前进n步，然后快慢指针同时前进，当快指针到达末尾时慢指针恰好在倒数第n个节点的前一个位置。使用哨兵节点dummy指向head，这样慢指针从dummy出发，停止时slow.next就是要删除的节点，直接slow.next=slow.next.next即可。若不设dummy，当删除的是头节点时需要特殊处理。时间O(n)，空间O(1)。

**优化代码**：

优化重点是 dummy 简化头节点、快慢指针定位、原地反转或哈希映射复杂指针。

**易错点**：快慢指针定位中点、倒数第 N、环入口；归并排序适合链表排序；复杂节点复制可用哈希表或原地穿插。

### 160. 相交链表（Easy）

**题目描述**：给定两个单链表的头节点，找出它们相交的起始节点（引用相等，非值相等）。若不相交返回null。要求O(n)时间O(1)空间，且保持原链表结构。

**为什么属于这个类型**：题目操作对象是链表节点，不能随机访问，核心是改变或利用 `next/random` 指针关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：双指针走完自己的链再走对方的链：两指针走过的总路程相同，必在交点处相遇。

**套用统一模板**：用 dummy 统一头节点变化；每次修改前先保存后继；反转/删除/合并后保证链不断。

**基础模板代码**：

```python
def getIntersectionNode(self, headA, headB):
    pointer_a = headA
    pointer_b = headB

    while pointer_a != pointer_b:
        if pointer_a:
            pointer_a = pointer_a.next
        else:
            pointer_a = headB

        if pointer_b:
            pointer_b = pointer_b.next
        else:
            pointer_b = headA

    return pointer_a
```

**信息流动 / 窗口变化 / 指针移动过程**：

关键洞察：设A独有长度a、B独有长度b、公共长度c，则a+c+b=b+c+a。指针p从A走完走B，q从B走完走A，两指针必在相交点相遇（或同时到达尾部null）。不需要计算长度差，只需一趟遍历即可。若不相交，两指针各走a+b步后同时为null，退出循环返回null。时间O(m+n)，空间O(1)。

**优化代码**：

优化重点是 dummy 简化头节点、快慢指针定位、原地反转或哈希映射复杂指针。

**易错点**：快慢指针定位中点、倒数第 N、环入口；归并排序适合链表排序；复杂节点复制可用哈希表或原地穿插。

### 148. 排序链表（Medium）

**题目描述**：给定链表头节点，在O(nlogn)时间和O(1)空间内对链表进行排序。

**为什么属于这个类型**：题目操作对象是链表节点，不能随机访问，核心是改变或利用 `next/random` 指针关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：归并排序适配链表：用快慢指针找中点，递归排序两半，再合并两个有序链表。

**套用统一模板**：用 dummy 统一头节点变化；每次修改前先保存后继；反转/删除/合并后保证链不断。

**基础模板代码**：

```python
def sortList(self, head):
    if not head or not head.next:
        return head

    slow = head
    fast = head.next
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    mid = slow.next
    slow.next = None

    left = self.sortList(head)
    right = self.sortList(mid)

    dummy = ListNode(0)
    tail = dummy

    while left and right:
        if left.val < right.val:
            tail.next = left
            left = left.next
        else:
            tail.next = right
            right = right.next
        tail = tail.next

    if left:
        tail.next = left
    else:
        tail.next = right

    return dummy.next
```

**信息流动 / 窗口变化 / 指针移动过程**：

链表排序首选归并排序，因为归并的合并操作天然适合链表（无需随机访问，O(1)额外空间）。自底向上归并：先求链表长度n，然后subLen从1开始倍增，每次将相邻两段subLen长度的子链表合并。用哨兵节点简化合并逻辑，每次合并后更新尾指针。subLen倍增直到>=n。时间O(nlogn)，空间O(1)。也可递归归并（快慢指针找中点），但递归栈空间O(logn)不符合严格O(1)要求。

**优化代码**：

优化重点是 dummy 简化头节点、快慢指针定位、原地反转或哈希映射复杂指针。

**易错点**：快慢指针定位中点、倒数第 N、环入口；归并排序适合链表排序；复杂节点复制可用哈希表或原地穿插。

### 234. 回文链表（Easy）

**题目描述**：给定单链表头节点，判断链表是否为回文。要求O(n)时间和O(1)空间。

**为什么属于这个类型**：题目操作对象是链表节点，不能随机访问，核心是改变或利用 `next/random` 指针关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：找中点+反转后半+双指针比较：O(1)空间检测回文，比较后可恢复链表。

**套用统一模板**：用 dummy 统一头节点变化；每次修改前先保存后继；反转/删除/合并后保证链不断。

**基础模板代码**：

```python
def isPalindrome(self, head):
    # 快慢指针找中点
    slow = head
    fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    # 反转后半段
    prev = None
    curr = slow
    while curr:
        next_temp = curr.next
        curr.next = prev
        prev = curr
        curr = next_temp
    # 双指针比较
    left = head
    right = prev
    while right:  # 只需比较后半段
        if left.val != right.val:
            return False
        left = left.next
        right = right.next
    return True
```

**信息流动 / 窗口变化 / 指针移动过程**：

O(1)空间方案：三步走。(1)快慢指针找链表中点（慢指针走一步快指针走两步，快指针到尾时慢指针在中点）；(2)反转后半部分链表；(3)双指针从头部和反转后的后半部头部同时遍历比较，全相等则为回文。最后可选恢复链表（再反转一次恢复原状）。注意处理奇数长度时跳过中点节点。时间O(n)，空间O(1)。若允许O(n)空间，直接复制到数组双指针比较更简单。

**优化代码**：

优化重点是 dummy 简化头节点、快慢指针定位、原地反转或哈希映射复杂指针。

**易错点**：快慢指针定位中点、倒数第 N、环入口；归并排序适合链表排序；复杂节点复制可用哈希表或原地穿插。

### 142. 环形链表II（Medium）

**题目描述**：给定链表头节点，返回链表开始入环的第一个节点。若链表无环返回null。要求O(1)空间。

**为什么属于这个类型**：题目操作对象是链表节点，不能随机访问，核心是改变或利用 `next/random` 指针关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：快慢指针找到相遇点后，从head和相遇点同速出发，必在环入口相遇：a=nb-s的数学结论。

**套用统一模板**：用 dummy 统一头节点变化；每次修改前先保存后继；反转/删除/合并后保证链不断。

**基础模板代码**：

```python
def detectCycle(self, head):
    slow = head
    fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:  # 相遇
            # 从head和相遇点同速出发，必在环入口相遇
            ptr = head
            while ptr != slow:
                ptr = ptr.next
                slow = slow.next
            return ptr
    return None
```

**信息流动 / 窗口变化 / 指针移动过程**：

Floyd算法分两阶段。第一阶段：快慢指针找相遇点，快指针走2步慢指针走1步，相遇时设慢指针走了s步，快指针走了2s步。设head到入环点距离a，入环点到相遇点距离b，环长c，则2s=s+kc（快指针多走k圈），即s=kc。而s=a+b，故a+b=kc，即a=kc-b=(k-1)c+(c-b)。这意味着从相遇点和head同时出发，两指针必在入环点相遇。第二阶段：一个指针从head出发，一个从相遇点出发，都每次走一步，相遇点即为入环点。时间O(n)，空间O(1)。

**优化代码**：

优化重点是 dummy 简化头节点、快慢指针定位、原地反转或哈希映射复杂指针。

**易错点**：快慢指针定位中点、倒数第 N、环入口；归并排序适合链表排序；复杂节点复制可用哈希表或原地穿插。

### 92. 反转链表II（Medium）

**题目描述**：给定单链表头节点和整数left、right（1-indexed），反转从位置left到right的链表部分。一趟扫描完成。

**为什么属于这个类型**：题目操作对象是链表节点，不能随机访问，核心是改变或利用 `next/random` 指针关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：定位反转区间[left,right]，反转中间段后重新连接：关键是记录反转段的前驱和后继。

**套用统一模板**：用 dummy 统一头节点变化；每次修改前先保存后继；反转/删除/合并后保证链不断。

**基础模板代码**：

```python
def reverseBetween(self, head, left, right):
    dummy = ListNode(0, head)
    pre = dummy
    # 走到left的前驱
    for _ in range(left - 1):
        pre = pre.next
    # 反转left到right之间的节点
    curr = pre.next
    prev = None
    for _ in range(right - left + 1):
        next_temp = curr.next
        curr.next = prev
        prev = curr
        curr = next_temp
    # 重新连接：pre.next -> 反转后头，反转后尾 -> curr
    pre.next.next = curr  # 反转段尾接后继
    pre.next = prev       # 前驱接反转段头
    return dummy.next
```

**信息流动 / 窗口变化 / 指针移动过程**：

核心是定位反转区间并在反转后正确重连。用哨兵节点dummy处理left=1的情况。先走到第left-1个节点pre，记录反转区间起点curr=pre.next，然后用头插法将[right-left]个节点依次插入到pre之后：每次保存curr.next为next，将next从原位置摘出插入pre之后（next.next=pre.next, pre.next=next, curr.next=next.next），共操作right-left次。头插法避免了额外反转再重连的复杂度，一趟完成。时间O(n)，空间O(1)。

**优化代码**：

优化重点是 dummy 简化头节点、快慢指针定位、原地反转或哈希映射复杂指针。

**易错点**：快慢指针定位中点、倒数第 N、环入口；归并排序适合链表排序；复杂节点复制可用哈希表或原地穿插。

### 25. K个一组翻转链表（Hard）

**题目描述**：给定链表头节点和整数k，每k个节点一组进行翻转，若剩余节点不足k个则保持原顺序。不能直接修改节点值，只能修改指针。

**为什么属于这个类型**：题目操作对象是链表节点，不能随机访问，核心是改变或利用 `next/random` 指针关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：每K个节点一组反转，不足K个不反转：递归/迭代地处理每组，组间重新连接。

**套用统一模板**：用 dummy 统一头节点变化；每次修改前先保存后继；反转/删除/合并后保证链不断。

**基础模板代码**：

```python
def reverseKGroup(self, head, k):
    # 检查剩余是否够k个
    curr = head
    count = 0
    while curr and count < k:
        curr = curr.next
        count += 1
    if count < k:
        return head  # 不够k个，不反转
    # 反转前k个
    prev = None
    curr = head
    for _ in range(k):
        next_temp = curr.next
        curr.next = prev
        prev = curr
        curr = next_temp
    # head反转后变成组尾，接上后续反转结果
    head.next = self.reverseKGroup(curr, k)
    return prev  # prev是反转后的新头
```

**信息流动 / 窗口变化 / 指针移动过程**：

分步处理：(1)先检测剩余节点是否够k个，不够则不翻转直接返回；(2)翻转当前k个节点，使用经典三指针反转（prev、curr、next），反转后原头变为尾部，原尾变为头部；(3)将反转后的子链表接入结果：prevGroup的next指向新头，原头（现尾）的next指向下一组起点。递归或迭代均可：递归将翻转后的尾部.next指向递归处理下一组的返回值；迭代则维护prevGroup指针。时间O(n)，空间O(1)迭代/O(n/k)递归。

**优化代码**：

优化重点是 dummy 简化头节点、快慢指针定位、原地反转或哈希映射复杂指针。

**易错点**：快慢指针定位中点、倒数第 N、环入口；归并排序适合链表排序；复杂节点复制可用哈希表或原地穿插。

### 138. 随机链表的复制（Medium）

**题目描述**：给定含random指针的链表头节点，深拷贝该链表并返回副本的头节点。每个节点有val、next、random三个域，random可指向null或链表中任意节点。

**为什么属于这个类型**：题目操作对象是链表节点，不能随机访问，核心是改变或利用 `next/random` 指针关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：两次遍历：第一次在原节点后插入副本节点，第二次复制random指针，最后拆分两链表。

**套用统一模板**：用 dummy 统一头节点变化；每次修改前先保存后继；反转/删除/合并后保证链不断。

**基础模板代码**：

```python
def copyRandomList(self, head):
    if not head:
        return None

    current = head
    while current:
        copied_node = Node(current.val)
        copied_node.next = current.next
        current.next = copied_node
        current = copied_node.next

    current = head
    while current:
        copied_node = current.next
        if current.random:
            copied_node.random = current.random.next
        current = copied_node.next

    current = head
    copied_head = head.next
    while current:
        copied_node = current.next
        next_original = copied_node.next

        current.next = next_original
        if next_original:
            copied_node.next = next_original.next
        else:
            copied_node.next = None

        current = next_original

    return copied_head
```

**信息流动 / 窗口变化 / 指针移动过程**：

最优方案为三次遍历原地复制。第一次遍历：在每个原节点后插入拷贝节点（A→A'→B→B'→...），拷贝节点的val已复制，next暂连后续原节点。第二次遍历：处理random指针，若原节点A.random=B，则拷贝A'.random=B.next=B'（因为B'紧跟在B后）。第三次遍历：拆分链表，将原链表和拷贝链表分离恢复。时间O(n)，空间O(1)（不计输出）。也可用哈希表映射原节点→拷贝节点，两次遍历完成，空间O(n)。

**优化代码**：

优化重点是 dummy 简化头节点、快慢指针定位、原地反转或哈希映射复杂指针。

**易错点**：快慢指针定位中点、倒数第 N、环入口；归并排序适合链表排序；复杂节点复制可用哈希表或原地穿插。

### 143. 重排链表（Medium）

**题目描述**：给定单链表L0→L1→…→Ln，重排为L0→Ln→L1→Ln-1→L2→Ln-2→…。要求原地操作，不能修改节点值。

**为什么属于这个类型**：题目操作对象是链表节点，不能随机访问，核心是改变或利用 `next/random` 指针关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：找中点+反转后半+双指针交替合并：L0→Ln→L1→Ln-1→...的三步曲。

**套用统一模板**：用 dummy 统一头节点变化；每次修改前先保存后继；反转/删除/合并后保证链不断。

**基础模板代码**：

```python
def reorderList(self, head):
    if not head or not head.next:
        return
    # 第一步：快慢指针找中点
    slow = head
    fast = head
    while fast.next and fast.next.next:
        slow = slow.next
        fast = fast.next.next
    # 第二步：反转后半段
    prev = None
    curr = slow.next
    slow.next = None  # 切断前后两半
    while curr:
        next_temp = curr.next
        curr.next = prev
        prev = curr
        curr = next_temp
    # 第三步：交替合并
    first = head
    second = prev
    while second:
        tmp1 = first.next
        tmp2 = second.next
        first.next = second
        second.next = tmp1
        first = tmp1
        second = tmp2
```

**信息流动 / 窗口变化 / 指针移动过程**：

三步法：(1)快慢指针找链表中点，将链表分为前后两半，slow停在中间节点；(2)反转后半部分链表，得到第二条链表rev；(3)双指针交替合并两条链表：依次从第一条取一个、第二条取一个拼接。注意前半段长度>=后半段（奇数时中间节点归前半段），合并时以后半段指针非空为循环条件。时间O(n)，空间O(1)。

**优化代码**：

优化重点是 dummy 简化头节点、快慢指针定位、原地反转或哈希映射复杂指针。

**易错点**：快慢指针定位中点、倒数第 N、环入口；归并排序适合链表排序；复杂节点复制可用哈希表或原地穿插。

## 顺序约束（栈）

### 如何识别这是栈题

栈适用于“最近打开的东西必须最先关闭”的顺序约束。识别信号是：

1. 括号、嵌套、编码、路径、递归遍历。
2. 当前 token 要和最近的未完成 token 匹配。
3. 需要保存进入下一层之前的上下文。
4. 处理顺序呈现后进先出。

判断公式：

> 如果最近发生的未完成约束必须最先被解决，就考虑栈。

### 一句话本质

最近未完成的事情最先被处理。

### 第一性原理

栈的第一性原理是后进先出。当问题存在嵌套、最近约束优先闭合、或递归过程需要显式模拟时，栈就是最自然的数据结构。

### 统一模板：先问 5 个问题

1. 什么东西需要等待未来匹配或完成？
2. 遇到开始符号时入栈什么？原值、期待值还是上下文？
3. 遇到结束符号时如何和栈顶合并？
4. 栈空时意味着合法结束还是错误？
5. 是否可以用栈模拟递归遍历？

### 通用代码骨架

```python
stack = []
for token in sequence:
    if token 是开始/待处理对象:
        stack.append(上下文)
    else:
        if not stack or 不匹配:
            return False
        top = stack.pop()
        合并结果
return 栈满足结束条件
```

### 常见状态变量

- 括号期待值
- 字符串解码的上一层字符串和重复次数
- 树遍历中的节点和访问状态
- 辅助最小值

### 常见剪枝 / 优化

- 右括号来时栈空直接失败
- 类型不匹配直接失败
- 遍历结束栈非空代表仍有未完成约束

### 本类题目总览

- 20. 有效的括号（Easy）
- 394. 字符串解码（Medium）
- 94. 二叉树的中序遍历（Easy）
- 155. 最小栈（Medium）

### 20. 有效的括号（Easy）

**题目描述**：给定仅含括号字符的字符串，判断括号是否有效——左括号必须以正确顺序和类型闭合。字符串长度可达10^4。

**为什么属于这个类型**：题目存在最近未完成的约束必须最先被处理的后进先出关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：括号匹配的本质是「最近未闭合的左括号必须最先被右括号闭合」——一个严格的后进先出约束。

**套用统一模板**：开始符号或待处理对象入栈，结束符号弹出并合并；遍历结束检查栈是否清空或是否有最终值。

**基础模板代码**：

```python
def isValid(self, s: str) -> bool:
    # 左右括号的映射
    pairs = {')': '(', ']': '[', '}': '{'}
    stack = []
    for ch in s:
        if ch in pairs.values():  # 左括号：入栈等待匹配
            stack.append(ch)
        elif ch in pairs:         # 右括号：必须匹配栈顶（最近的左括号）
            if not stack or stack[-1] != pairs[ch]:
                return False
            stack.pop()
    return not stack  # 所有左括号都必须被闭合
```

**信息流动 / 窗口变化 / 指针移动过程**：

栈的典型应用。遍历字符串：遇到左括号压栈对应的右括号（方便直接比较），遇到右括号则弹栈比较是否匹配，不匹配或栈空则无效。遍历结束后栈必须为空才有效。关键点：左括号压入对应的右括号（如遇'('压')'），这样遇到右括号时只需比较弹栈值是否等于当前字符，避免额外的映射查找。时间O(n)，空间O(n)。

**优化代码**：

优化重点是压入期待值、保存上下文或用显式栈替代递归。

**易错点**：括号题可压入期待字符；解码题压入上一层上下文；树遍历可用栈模拟递归。

### 394. 字符串解码（Medium）

**题目描述**：给定编码后的字符串，按k[encoded_string]规则解码，方括号内字符串重复k次。可嵌套编码，输入保证合法。

**为什么属于这个类型**：题目存在最近未完成的约束必须最先被处理的后进先出关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：嵌套解码的本质是「外层需要等待内层结果才能继续」——嵌套深度越深，优先级越高，天然对应栈的LIFO。

**套用统一模板**：开始符号或待处理对象入栈，结束符号弹出并合并；遍历结束检查栈是否清空或是否有最终值。

**基础模板代码**：

```python
def decodeString(self, s: str) -> str:
    stack = []  # 每个元素: (前缀字符串, 重复次数)
    cur_str = ''
    cur_num = 0
    for ch in s:
        if ch.isdigit():
            cur_num = cur_num * 10 + int(ch)
        elif ch == '[':
            # 进入新层：保存外层状态，重置当前层
            stack.append((cur_str, cur_num))
            cur_str = ''
            cur_num = 0
        elif ch == ']':
            # 退出当前层：用内层结果拼回外层
            prev_str, num = stack.pop()
            cur_str = prev_str + cur_str * num
        else:
            cur_str += ch
    return cur_str
```

**信息流动 / 窗口变化 / 指针移动过程**：

用栈处理嵌套结构。遍历字符串四种情况：(1)数字：累积倍数k（注意多位数字如12需累积）；(2)字母：累积到当前结果字符串；(3)'['：将当前倍数和结果字符串压栈，重置倍数和结果（保存外层上下文）；(4)']'：弹出栈顶的倍数和前一段字符串，当前结果=前一段+当前结果重复倍数次。栈中保存的是外层还未处理的上下文，遇到']'时回溯合并。时间O(n)（输出长度），空间O(n)。

**优化代码**：

优化重点是压入期待值、保存上下文或用显式栈替代递归。

**易错点**：括号题可压入期待字符；解码题压入上一层上下文；树遍历可用栈模拟递归。

### 94. 二叉树的中序遍历（Easy）

**题目描述**：给定二叉树根节点，返回中序遍历（左→根→右）的节点值序列。要求迭代法也需掌握。

**为什么属于这个类型**：题目存在最近未完成的约束必须最先被处理的后进先出关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：中序遍历的栈实现本质是「沿左链深入到底，回溯时处理节点再转向右子树」——深度优先的递归结构用栈显式管理返回点。

**套用统一模板**：开始符号或待处理对象入栈，结束符号弹出并合并；遍历结束检查栈是否清空或是否有最终值。

**基础模板代码**：

```python
def inorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
    res = []
    stack = []
    cur = root
    while cur or stack:
        # 沿左链深入到底，沿途节点入栈
        while cur:
            stack.append(cur)
            cur = cur.left
        # 回溯：弹出栈顶，处理，转向右子树
        cur = stack.pop()
        res.append(cur.val)
        cur = cur.right
    return res
```

**信息流动 / 窗口变化 / 指针移动过程**：

递归法最直观：先递归左子树、访问根、再递归右子树。迭代法用栈模拟递归：不断沿左子树入栈直到null，弹栈访问节点，然后转向右子树重复过程。Morris遍历可做到O(1)空间：利用线索化，在左子树的最右节点（前驱）的right指向当前节点，遍历完左子树后通过线索回到当前节点，恢复线索后转右子树。递归/迭代时间O(n)，空间O(h)（h为树高）；Morris时间O(n)均摊，空间O(1)。

**优化代码**：

优化重点是压入期待值、保存上下文或用显式栈替代递归。

**易错点**：括号题可压入期待字符；解码题压入上一层上下文；树遍历可用栈模拟递归。

### 155. 最小栈（Medium）

**题目描述**：undefined

**为什么属于这个类型**：题目存在最近未完成的约束必须最先被处理的后进先出关系。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：在O(1)内获取最小值的关键是「每个元素入栈时，同步记录当前时刻的最小值」——空间换时间，让历史最小值随栈状态同步演进。

**套用统一模板**：开始符号或待处理对象入栈，结束符号弹出并合并；遍历结束检查栈是否清空或是否有最终值。

**基础模板代码**：

```python
class MinStack:
    def __init__(self):
        # 每个元素: (值, 以该元素为栈顶时的最小值)
        self.stack = []

    def push(self, val: int) -> None:
        if not self.stack:
            self.stack.append((val, val))
        else:
            # 当前最小值 = min(新值, 之前的最小值)
            self.stack.append((val, min(val, self.stack[-1][1])))

    def pop(self) -> None:
        self.stack.pop()

    def top(self) -> int:
        return self.stack[-1][0]

    def getMin(self) -> int:
        # 栈顶直接携带当前最小值
        return self.stack[-1][1]
```

**信息流动 / 窗口变化 / 指针移动过程**：

普通栈无法O(1)取最小值，因为弹出最小值后需要重新扫描。第一性原理分析：最小值是栈状态的函数——栈在某个状态下的最小值，取决于之前所有入栈元素。关键洞察：如果我们让每个栈元素同时携带「以它为栈顶时的历史最小值」，那么任何时刻的栈顶就直接给出了当前最小值。弹出时无需重新计算，因为新栈顶已经保存了那时的最小值。这是用O(n)额外空间将查询从O(n)降到O(1)。

**优化代码**：

优化重点是压入期待值、保存上下文或用显式栈替代递归。

**易错点**：括号题可压入期待字符；解码题压入上一层上下文；树遍历可用栈模拟递归。

## 单调空间的搜索（二分）

### 如何识别这是二分搜索题

二分不是只能用于“有序数组查找”。它真正适用的条件是搜索空间存在单调性。识别信号是：

1. 题目给有序数组，要求查找值、边界、插入位置。
2. 虽然数组不是整体有序，但局部有序或旋转后仍能排除一半。
3. 答案是一个数值，且“答案 <= x 是否可行”或“答案 >= x 是否可行”具有单调性。
4. 暴力试答案是 O(n) 或 O(range)，但每次判断可以排除半个答案空间。

判断公式：

> 如果可以设计一个单调判断 `check(mid)`：某一侧全都可行，另一侧全都不可行，就考虑二分。

典型例子：

- 在有序数组中找目标：`nums[mid] < target` 时左半边都不可能。
- 找第一个满足条件的位置：`check(mid)` 为 True 后，答案还可能在更左边。
- 搜索答案值：如果容量 `mid` 能完成任务，那么更大的容量也能完成，这就是单调性。

不适用二分的情况：

- `check(mid)` 没有单调性，True/False 交错出现。
- 移动 `mid` 无法排除一整半候选。
- 只是“想快一点”，但没有可证明的有序或单调结构。

### 一句话本质

在单调判断中每次排除半个搜索空间。

### 第一性原理

二分的第一性原理是：如果某个判断函数在搜索空间上具有单调性，那么一次判断就能确定答案不在其中一半。

### 统一模板：先问 5 个问题

1. 搜索空间是什么？下标、数值答案还是分割位置？
2. 判断函数 `check(mid)` 是否单调？
3. 要找第一个满足、最后一个满足，还是精确命中？
4. 边界是左闭右闭还是左闭右开？
5. 更新 `left/right` 后是否会死循环？

### 通用代码骨架

```python
left, right = 搜索边界
while left < right:
    mid = (left + right) // 2
    if check(mid):
        right = mid
    else:
        left = mid + 1
return left
```

### 常见状态变量

- `left/right`：候选答案区间
- `mid`：一次判断点
- `check`：把复杂条件转为布尔单调判断

### 常见剪枝 / 优化

- 旋转数组通过有序半边排除
- 找边界时不要在命中后立即返回
- 数值二分要防止平方溢出或浮点精度问题

### 本类题目总览

- 33. 搜索旋转排序数组（Medium）
- 34. 在排序数组中查找元素的第一个和最后一个位置（Medium）
- 153. 寻找旋转排序数组中的最小值（Medium）
- 4. 寻找两个正序数组的中位数（Hard）
- 69. x的平方根（Easy）
- 162. 寻找峰值（Medium）
- 35. 搜索插入位置（Easy）
- 367. 有效的完全平方数（Easy）

### 33. 搜索旋转排序数组（Medium）

**题目描述**：给定升序排列后旋转的整数数组nums和目标值target，返回target在数组中的下标，不存在返回-1。时间复杂度O(logn)，数组中无重复元素。

**为什么属于这个类型**：题目存在有序空间或单调判断，一次检查 `mid` 可以排除一半候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：旋转数组本质是两段单调子序列的拼接——二分时必有一半是有序的，利用有序半边确定目标在哪一侧，从而持续剪枝。

**套用统一模板**：定义 `check(mid)`，明确找左边界还是右边界，再选择稳定的边界更新规则。

**基础模板代码**：

```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        left = 0
        right = len(nums) - 1

        while left <= right:
            mid = left + (right - left) // 2
            if nums[mid] == target:
                return mid

            left_part_sorted = nums[left] <= nums[mid]
            if left_part_sorted:
                target_in_left = nums[left] <= target < nums[mid]
                if target_in_left:
                    right = mid - 1
                else:
                    left = mid + 1
            else:
                target_in_right = nums[mid] < target <= nums[right]
                if target_in_right:
                    left = mid + 1
                else:
                    right = mid - 1

        return -1
```

**信息流动 / 窗口变化 / 指针移动过程**：

旋转数组的特点是一分为二后必有一半是有序的。二分时先判断mid左右哪半有序：若nums[left]<=nums[mid]则左半有序，否则右半有序。在有序的一半中判断target是否落在其中：若左半有序且nums[left]<=target<nums[mid]，则right=mid-1，否则left=mid+1；若右半有序且nums[mid]<target<=nums[right]，则left=mid+1，否则right=mid-1。每次将搜索范围缩小一半，时间O(logn)。使用前闭后闭[left,right]模板。

**优化代码**：

优化重点是边界模板统一、避免死循环、区分找精确值/左边界/右边界。

**易错点**：旋转数组先判断哪半有序；找区间边界做两次二分；数值答案题用答案域二分。

### 34. 在排序数组中查找元素的第一个和最后一个位置（Medium）

**题目描述**：给定非递减顺序数组nums和目标值target，返回target出现的起始和终止位置，不存在返回[-1,-1]。要求O(logn)时间。

**为什么属于这个类型**：题目存在有序空间或单调判断，一次检查 `mid` 可以排除一半候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：找边界就是找「最后一个<target的位置」和「第一个>target的位置」——将边界问题转化为两次二分判定。

**套用统一模板**：定义 `check(mid)`，明确找左边界还是右边界，再选择稳定的边界更新规则。

**基础模板代码**：

```python
class Solution:
    def searchRange(self, nums: List[int], target: int) -> List[int]:
        def find_left_boundary():
            left = 0
            right = len(nums) - 1
            result = -1

            while left <= right:
                mid = left + (right - left) // 2
                if nums[mid] == target:
                    result = mid
                    right = mid - 1
                elif nums[mid] < target:
                    left = mid + 1
                else:
                    right = mid - 1

            return result

        def find_right_boundary():
            left = 0
            right = len(nums) - 1
            result = -1

            while left <= right:
                mid = left + (right - left) // 2
                if nums[mid] == target:
                    result = mid
                    left = mid + 1
                elif nums[mid] < target:
                    left = mid + 1
                else:
                    right = mid - 1

            return result

        left_boundary = find_left_boundary()
        right_boundary = find_right_boundary()
        return [left_boundary, right_boundary]
```

**信息流动 / 窗口变化 / 指针移动过程**：

需要两次二分分别找左边界和右边界。找左边界：当nums[mid]==target时继续向左收缩right=mid-1，最终left即为第一个>=target的位置，检查是否等于target。找右边界：当nums[mid]==target时继续向右收缩left=mid+1，最终right即为最后一个<=target的位置，检查是否等于target。两次二分均为前闭后闭[left,right]模板。关键是在找到target后不停下而是继续收缩边界，才能定位到最左/最右位置。时间O(logn)。

**优化代码**：

优化重点是边界模板统一、避免死循环、区分找精确值/左边界/右边界。

**易错点**：旋转数组先判断哪半有序；找区间边界做两次二分；数值答案题用答案域二分。

### 153. 寻找旋转排序数组中的最小值（Medium）

**题目描述**：给定升序排列后旋转的数组（无重复元素），找到其中的最小值。要求O(logn)时间。

**为什么属于这个类型**：题目存在有序空间或单调判断，一次检查 `mid` 可以排除一半候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：最小值位于旋转断点的右侧——通过与右端点比较，判断最小值在mid的哪一侧，持续缩小搜索范围。

**套用统一模板**：定义 `check(mid)`，明确找左边界还是右边界，再选择稳定的边界更新规则。

**基础模板代码**：

```python
class Solution:
    def findMin(self, nums: List[int]) -> int:
        left = 0
        right = len(nums) - 1

        while left < right:
            mid = left + (right - left) // 2
            if nums[mid] > nums[right]:
                left = mid + 1
            else:
                right = mid

        return nums[left]
```

**信息流动 / 窗口变化 / 指针移动过程**：

旋转数组的最小值在旋转点。二分时将mid与right比较：若nums[mid]>nums[right]，说明最小值在mid右侧（右半无序，旋转点在其中），left=mid+1；若nums[mid]<nums[right]，说明右半有序，最小值在mid或mid左侧，right=mid；若相等则right-=1（本题无重复可省略）。循环退出时left==right即为最小值位置。关键洞察：与right比较而非left，因为与left比较无法区分整体有序和右半有序的情况。前闭后闭模板，但right=mid而非mid-1因为mid本身可能是答案。

**优化代码**：

优化重点是边界模板统一、避免死循环、区分找精确值/左边界/右边界。

**易错点**：旋转数组先判断哪半有序；找区间边界做两次二分；数值答案题用答案域二分。

### 4. 寻找两个正序数组的中位数（Hard）

**题目描述**：给定两个大小分别为m和n的正序数组nums1和nums2，找出并返回这两个数组的中位数。要求O(log(m+n))时间复杂度。

**为什么属于这个类型**：题目存在有序空间或单调判断，一次检查 `mid` 可以排除一半候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：中位数是「将全体分成等大的两半」的分界点——在两个有序数组上二分搜索这个分界点，利用有序性O(1)验证分割的有效性。

**套用统一模板**：定义 `check(mid)`，明确找左边界还是右边界，再选择稳定的边界更新规则。

**基础模板代码**：

```python
class Solution:
    def findMedianSortedArrays(self, nums1, nums2):
        def find_kth(k):
            index1 = 0
            index2 = 0

            while True:
                if index1 == len(nums1):
                    return nums2[index2 + k - 1]
                if index2 == len(nums2):
                    return nums1[index1 + k - 1]
                if k == 1:
                    return min(nums1[index1], nums2[index2])

                half = k // 2
                new_index1 = index1 + half - 1
                new_index2 = index2 + half - 1

                if new_index1 < len(nums1):
                    pivot1 = nums1[new_index1]
                else:
                    pivot1 = float('inf')

                if new_index2 < len(nums2):
                    pivot2 = nums2[new_index2]
                else:
                    pivot2 = float('inf')

                if pivot1 <= pivot2:
                    removed_count = new_index1 - index1 + 1
                    index1 = new_index1 + 1
                else:
                    removed_count = new_index2 - index2 + 1
                    index2 = new_index2 + 1

                k -= removed_count

        total_len = len(nums1) + len(nums2)
        if total_len % 2 == 1:
            middle = total_len // 2 + 1
            return find_kth(middle)

        left_middle = find_kth(total_len // 2)
        right_middle = find_kth(total_len // 2 + 1)
        return (left_middle + right_middle) / 2
```

**信息流动 / 窗口变化 / 指针移动过程**：

中位数等价于找第k小的元素（奇数k=(m+n+1)/2，偶数取第k小和第k+1小的平均）。二分在第k小上：比较两数组中第k/2个元素，较小的那个其前k/2-1个元素不可能为第k小，排除后在剩余部分递归找第(k-k/2)小。迭代实现：维护双指针i、j表示两数组已排除的位置，每次排除k/2个元素直到k=1。边界处理：某数组已全部排除则直接取另一数组第k个；k=1时取两数组当前元素较小者。时间O(log(m+n))，空间O(1)。使用前闭后闭思想，k从总长度开始逐步缩小。

**优化代码**：

优化重点是边界模板统一、避免死循环、区分找精确值/左边界/右边界。

**易错点**：旋转数组先判断哪半有序；找区间边界做两次二分；数值答案题用答案域二分。

### 69. x的平方根（Easy）

**题目描述**：给定非负整数x，计算并返回x的算术平方根的整数部分。不能使用内置指数函数，返回结果向下取整。

**为什么属于这个类型**：题目存在有序空间或单调判断，一次检查 `mid` 可以排除一半候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：sqrt(x)是最大的使得mid*mid<=x的整数——在有序整数空间上二分搜索这个临界点。

**套用统一模板**：定义 `check(mid)`，明确找左边界还是右边界，再选择稳定的边界更新规则。

**基础模板代码**：

```python
class Solution:
    def mySqrt(self, x: int) -> int:
        left = 0
        right = x

        while left <= right:
            mid = left + (right - left) // 2
            square = mid * mid

            if square <= x:
                left = mid + 1
            else:
                right = mid - 1

        return right
```

**信息流动 / 窗口变化 / 指针移动过程**：

求最大的mid使得mid*mid<=x，等价于找右边界。二分搜索[left,right]=[0,x]，mid*mid<=x时记录mid并继续向右搜索left=mid+1，mid*mid>x时right=mid-1。循环结束right即为答案（因为right=left-1，left是第一个mid*mid>x的位置，right是最后一个mid*mid<=x的位置）。为避免mid*mid溢出，可用mid<=x//mid替代，但需注意整除的精度问题。前闭后闭模板，时间O(logx)。

**优化代码**：

优化重点是边界模板统一、避免死循环、区分找精确值/左边界/右边界。

**易错点**：旋转数组先判断哪半有序；找区间边界做两次二分；数值答案题用答案域二分。

### 162. 寻找峰值（Medium）

**题目描述**：给定整数数组nums（nums[-1]=nums[n]=负无穷），找到任一峰值索引并返回。峰值元素大于其相邻元素。要求O(logn)时间。

**为什么属于这个类型**：题目存在有序空间或单调判断，一次检查 `mid` 可以排除一半候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：沿着上升方向走必然到达峰值——局部单调性提供了方向信息，二分剪枝的依据是「mid的邻居关系告诉你峰值在哪一侧」。

**套用统一模板**：定义 `check(mid)`，明确找左边界还是右边界，再选择稳定的边界更新规则。

**基础模板代码**：

```python
class Solution:
    def findPeakElement(self, nums: List[int]) -> int:
        left = 0
        right = len(nums) - 1

        while left < right:
            mid = left + (right - left) // 2
            if nums[mid] < nums[mid + 1]:
                left = mid + 1
            else:
                right = mid

        return left
```

**信息流动 / 窗口变化 / 指针移动过程**：

由于边界外视为负无穷，数组中必存在峰值。二分的关键是比较nums[mid]和nums[mid+1]：若nums[mid]<nums[mid+1]，说明右侧有上升趋势，峰值在右半部分（因为右边界外是负无穷，上升序列必定会下降到边界），left=mid+1；若nums[mid]>nums[mid+1]，说明mid右侧在下降，mid本身可能是峰值，峰值在mid或左侧，right=mid。循环退出时left==right即为峰值位置。前闭后闭模板变形，时间O(logn)。

**优化代码**：

优化重点是边界模板统一、避免死循环、区分找精确值/左边界/右边界。

**易错点**：旋转数组先判断哪半有序；找区间边界做两次二分；数值答案题用答案域二分。

### 35. 搜索插入位置（Easy）

**题目描述**：给定排序数组和目标值，找到目标值的插入位置使其仍有序。若目标值已存在则返回其索引。时间复杂度O(logn)。

**为什么属于这个类型**：题目存在有序空间或单调判断，一次检查 `mid` 可以排除一半候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：插入位置就是「第一个不小于target的索引」——标准的下界二分搜索，有序性的最直接应用。

**套用统一模板**：定义 `check(mid)`，明确找左边界还是右边界，再选择稳定的边界更新规则。

**基础模板代码**：

```python
class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        left = 0
        right = len(nums) - 1

        while left <= right:
            mid = left + (right - left) // 2
            if nums[mid] < target:
                left = mid + 1
            else:
                right = mid - 1

        return left
```

**信息流动 / 窗口变化 / 指针移动过程**：

本质是找第一个>=target的位置（lower_bound）。二分[left,right]：当nums[mid]<target时left=mid+1，当nums[mid]>=target时right=mid-1。循环退出后left即为插入位置：若target存在，left指向第一个等于target的位置；若不存在，left指向第一个大于target的位置（即应插入的位置）。left范围在[0,n]之间。前闭后闭模板，最终返回left（因为left=right+1，是第一个>=target的位置）。时间O(logn)。

**优化代码**：

优化重点是边界模板统一、避免死循环、区分找精确值/左边界/右边界。

**易错点**：旋转数组先判断哪半有序；找区间边界做两次二分；数值答案题用答案域二分。

### 367. 有效的完全平方数（Easy）

**题目描述**：给定正整数num，判断它是否是完全平方数。不能使用内置平方根函数。

**为什么属于这个类型**：题目存在有序空间或单调判断，一次检查 `mid` 可以排除一半候选。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：完全平方数n存在整数k使得k*k=n——在有序整数空间上二分搜索这个k，与平方根问题的验证版等价。

**套用统一模板**：定义 `check(mid)`，明确找左边界还是右边界，再选择稳定的边界更新规则。

**基础模板代码**：

```python
class Solution:
    def isPerfectSquare(self, num: int) -> bool:
        left = 1
        right = num

        while left <= right:
            mid = left + (right - left) // 2
            square = mid * mid

            if square == num:
                return True
            if square < num:
                left = mid + 1
            else:
                right = mid - 1

        return False
```

**信息流动 / 窗口变化 / 指针移动过程**：

二分搜索[1,num]中是否存在mid使得mid*mid==num。前闭后闭模板：mid*mid==num返回True，mid*mid<num则left=mid+1，mid*mid>num则right=mid-1。循环结束未找到则返回False。优化：right初始可取num//2+1（因为num>1时平方根不超过num//2+1，但num=1时需特判或right从1开始）。注意mid*mid可能溢出，Python整数无溢出问题，其他语言需用除法或长整型。时间O(logn)。

**优化代码**：

优化重点是边界模板统一、避免死循环、区分找精确值/左边界/右边界。

**易错点**：旋转数组先判断哪半有序；找区间边界做两次二分；数值答案题用答案域二分。

## 二值信息的并行处理（位运算）

### 如何识别这是位运算题

位运算适用于信息能按二进制位独立处理的问题。识别信号是：

1. 题目出现只出现一次、成对抵消、汉明距离、bit 计数。
2. 每一位可以独立统计或转移。
3. 需要 O(1) 空间压缩多个布尔状态。
4. 异或、与、移位能直接表达题目关系。

判断公式：

> 如果数字的二进制位可以被看作独立状态通道，就考虑位运算。

### 一句话本质

把整数看成多个并行布尔通道。

### 第一性原理

位运算的第一性原理是：整数的每一位都是一个独立的二值状态。异或、与、移位等操作可以同时处理所有位，从而压缩时间和空间。

### 统一模板：先问 5 个问题

1. 每一位代表什么信息？
2. 异或、与、或、取反分别表达什么集合操作？
3. 是否可以用 `x & (x-1)` 消掉最低位 1？
4. 状态能否按位独立转移？
5. 是否要考虑负数或语言整数位宽？

### 通用代码骨架

```python
ans = 0
while x:
    处理最低位或最低位1
    x = 更新后的 x
return ans
```

### 常见状态变量

- 异或累计值
- 位计数数组
- 最低位 1：`x & -x`
- 消除最低位 1：`x & (x - 1)`

### 常见剪枝 / 优化

- 成对元素异或后抵消
- 只循环 1 的个数而不是所有位
- 汉明距离先异或再计数

### 本类题目总览

- 136. 只出现一次的数字（Easy）
- 191. 位1的个数（Easy）
- 338. 比特位计数（Medium）
- 461. 汉明距离（Easy）

### 136. 只出现一次的数字（Easy）

**题目描述**：给定非空整数数组，其中除一个元素只出现一次外其余每个元素均出现两次，找出只出现一次的元素。要求线性时间且不使用额外空间。

**为什么属于这个类型**：题目信息可以按二进制位独立处理，位操作能并行压缩多个布尔状态。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：XOR的自反性——a⊕a=0消去所有成对元素，a⊕0=a保留孤立项——将「配对消除」编码为位级并行操作。

**套用统一模板**：根据题意选择异或抵消、按位计数、最低位 1 或位 DP 递推。

**基础模板代码**：

```python
def singleNumber(self, nums: List[int]) -> int:
    result = 0
    for num in nums:
        result ^= num  # 成对数异或归零，孤立项保留
    return result
```

**信息流动 / 窗口变化 / 指针移动过程**：

异或运算的核心性质：a^a=0，a^0=a，异或满足交换律和结合律。将数组所有元素异或，出现两次的元素互相抵消为0，最终结果就是只出现一次的元素。遍历一次即可，时间O(n)，空间O(1)。这是位运算最经典的应用之一。

**优化代码**：

优化重点是 `x & (x-1)`、异或抵消、最低位提取和位 DP。

**易错点**：`x & (x-1)` 可快速移除最低位 1；成对数字用异或抵消；汉明距离先异或再计数。

### 191. 位1的个数（Easy）

**题目描述**：给定无符号整数，返回其二进制表示中1的个数（汉明权重）。

**为什么属于这个类型**：题目信息可以按二进制位独立处理，位操作能并行压缩多个布尔状态。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：n&(n-1)每次消除最低位的1——从「数位」降维到「数1的个数」，避免了逐位检查的浪费。

**套用统一模板**：根据题意选择异或抵消、按位计数、最低位 1 或位 DP 递推。

**基础模板代码**：

```python
def hammingWeight(self, n: int) -> int:
    count = 0
    while n:
        n &= n - 1  # 每次消除最低位的1
        count += 1
    return count
```

**信息流动 / 窗口变化 / 指针移动过程**：

方法一：逐位检查，n&1检查最低位，n>>=1右移，循环32次。方法二：Brian Kernighan算法——n&=(n-1)每次消除最低位的1（n-1将最低位的1变为0，其后0变为1，与原n相与后该1及其后位变为0）。循环次数等于1的个数，效率更高。例如n=12(1100)，n-1=11(1011)，n&(n-1)=8(1000)消除了最低位的1。时间O(k)（k为1的个数），空间O(1)。

**优化代码**：

优化重点是 `x & (x-1)`、异或抵消、最低位提取和位 DP。

**易错点**：`x & (x-1)` 可快速移除最低位 1；成对数字用异或抵消；汉明距离先异或再计数。

### 338. 比特位计数（Medium）

**题目描述**：给定整数n，返回长度为n+1的数组ans，ans[i]为i的二进制表示中1的个数。要求O(n)时间。

**为什么属于这个类型**：题目信息可以按二进制位独立处理，位操作能并行压缩多个布尔状态。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：i的二进制1个数 = i去掉最低位1后的1个数 + 1——利用已计算的小值结果推导大值，位运算与动态规划的完美融合。

**套用统一模板**：根据题意选择异或抵消、按位计数、最低位 1 或位 DP 递推。

**基础模板代码**：

```python
def countBits(self, n: int) -> List[int]:
    dp = [0] * (n + 1)
    for i in range(1, n + 1):
        # 去掉最低位1后的值已计算，加1即可
        dp[i] = dp[i & (i - 1)] + 1
    return dp
```

**信息流动 / 窗口变化 / 指针移动过程**：

动态规划+位运算。关键递推：ans[i]=ans[i>>1]+(i&1)，即i的1的个数等于i右移一位后的1的个数加上最低位是否为1。也可用ans[i]=ans[i&(i-1)]+1，i&(i-1)消除最低位1后的数一定比i小，其1的个数已计算过。两种方法都是O(n)时间O(n)空间。后者更巧妙，直接利用Brian Kernighan的核心观察。

**优化代码**：

优化重点是 `x & (x-1)`、异或抵消、最低位提取和位 DP。

**易错点**：`x & (x-1)` 可快速移除最低位 1；成对数字用异或抵消；汉明距离先异或再计数。

### 461. 汉明距离（Easy）

**题目描述**：给定两个整数x和y，计算它们之间的汉明距离（二进制表示中不同位的数目）。

**为什么属于这个类型**：题目信息可以按二进制位独立处理，位操作能并行压缩多个布尔状态。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：汉明距离就是两个数XOR结果中1的个数——差异信息被XOR压缩到一个整数中，再数1即可。

**套用统一模板**：根据题意选择异或抵消、按位计数、最低位 1 或位 DP 递推。

**基础模板代码**：

```python
def hammingDistance(self, x: int, y: int) -> int:
    xor = x ^ y  # 差异编码：不同位为1
    count = 0
    while xor:
        xor &= xor - 1  # 逐个消除1
        count += 1
    return count
```

**信息流动 / 窗口变化 / 指针移动过程**：

汉明距离等于x^y的结果中1的个数。先异或得到两数差异的位掩码，再用Brian Kernighan算法计数1的个数：xor&=(xor-1)每次消除一个最低位的1，计数器+1，直到xor=0。时间O(k)（k为1的个数），空间O(1)。也可用内置bin(xor).count('1')，但面试推荐手写计数。

**优化代码**：

优化重点是 `x & (x-1)`、异或抵消、最低位提取和位 DP。

**易错点**：`x & (x-1)` 可快速移除最低位 1；成对数字用异或抵消；汉明距离先异或再计数。

## 空间变换（矩阵）

### 如何识别这是矩阵题

矩阵题通常是二维坐标规则题。识别信号是：

1. 输入是二维数组。
2. 操作涉及旋转、螺旋、置零、二维搜索、颜色分区。
3. 需要处理坐标映射、边界收缩或原地标记。
4. 行列存在有序性，可以一次排除一行或一列。

判断公式：

> 如果关键难点是二维坐标如何移动、映射或标记，就先写坐标规则。

### 一句话本质

用坐标映射描述二维空间变化。

### 第一性原理

矩阵题的第一性原理是：二维数组的操作本质是坐标变换或状态标记。只要写清楚旧坐标和新坐标的对应关系，旋转、遍历、置零、搜索都会变成规则执行。

### 统一模板：先问 5 个问题

1. 当前位置 `(i,j)` 在变换后去哪里？
2. 遍历边界如何收缩？
3. 是否需要原地修改，如何避免覆盖未读信息？
4. 矩阵是否具有行列单调性？
5. 能否用第一行第一列作为标记空间？

### 通用代码骨架

```python
for i in range(m):
    for j in range(n):
        根据坐标规则读取或写入
        维护边界/标记/答案
```

### 常见状态变量

- 四个边界：`top/bottom/left/right`
- 原地标记：第一行、第一列、特殊值
- 坐标变换：转置、翻转、旋转
- 行列单调搜索位置

### 常见剪枝 / 优化

- 单调矩阵从右上/左下开始每步排除一行或一列
- 螺旋遍历每走完一边收缩边界
- 置零先标记后统一修改，避免污染信息

### 本类题目总览

- 48. 旋转图像（Medium）
- 54. 螺旋矩阵（Medium）
- 73. 矩阵置零（Medium）
- 240. 搜索二维矩阵II（Medium）
- 75. 颜色分类（Medium）

### 48. 旋转图像（Medium）

**题目描述**：给定n×n的二维矩阵表示图像，将图像顺时针旋转90度，必须原地修改。

**为什么属于这个类型**：题目核心是二维坐标移动、映射、边界收缩或原地标记。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：90度旋转等价于「先转置再水平翻转」——将一个复合坐标变换分解为两个简单变换的叠加。

**套用统一模板**：明确 `(i,j)` 的移动规则或新旧坐标映射，再选择合适遍历顺序避免覆盖信息。

**基础模板代码**：

```python
def rotate(self, matrix: List[List[int]]) -> None:
    n = len(matrix)
    # 第一步：转置 (i,j) -> (j,i)
    for i in range(n):
        for j in range(i + 1, n):
            matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
    # 第二步：水平翻转 (i,j) -> (i, n-1-j)
    for i in range(n):
        for j in range(n // 2):
            matrix[i][j], matrix[i][n-1-j] = matrix[i][n-1-j], matrix[i][j]
```

**信息流动 / 窗口变化 / 指针移动过程**：

方法一：两次翻转——先沿主对角线转置（matrix[i][j]↔matrix[j][i]），再水平翻转每行（reverse每行）。转置+水平翻转=顺时针90度。方法二：直接四角轮换——对于每一层，将(i,j)→(j,n-1-i)→(n-1-i,n-1-j)→(n-1-j,i)→(i,j)四角交换。只需遍历i∈[0,n/2),j∈[i,n-1-i)。方法一实现更简洁不易出错，时间O(n²)，空间O(1)。

**优化代码**：

优化重点是原地标记、从角落排除行列、转置翻转或三指针分区。

**易错点**：旋转可转置再翻转；置零用第一行列做标记；行列有序矩阵从角落开始排除。

### 54. 螺旋矩阵（Medium）

**题目描述**：给定m×n矩阵，按顺时针螺旋顺序返回所有元素。

**为什么属于这个类型**：题目核心是二维坐标移动、映射、边界收缩或原地标记。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：螺旋遍历是「沿边界行走，走到尽头就收缩边界」——用四个边界变量刻画当前可走范围，方向切换由边界触碰触发。

**套用统一模板**：明确 `(i,j)` 的移动规则或新旧坐标映射，再选择合适遍历顺序避免覆盖信息。

**基础模板代码**：

```python
def spiralOrder(self, matrix: List[List[int]]) -> List[int]:
    if not matrix:
        return []
    top = 0
    bottom = len(matrix) - 1
    left = 0
    right = len(matrix[0]) - 1
    res = []
    while top <= bottom and left <= right:
        # 向右走
        for j in range(left, right + 1):
            res.append(matrix[top][j])
        top += 1
        # 向下走
        for i in range(top, bottom + 1):
            res.append(matrix[i][right])
        right -= 1
        # 向左走（需检查边界）
        if top <= bottom:
            for j in range(right, left - 1, -1):
                res.append(matrix[bottom][j])
            bottom -= 1
        # 向上走（需检查边界）
        if left <= right:
            for i in range(bottom, top - 1, -1):
                res.append(matrix[i][left])
            left += 1
    return res
```

**信息流动 / 窗口变化 / 指针移动过程**：

按层模拟：用四个变量top、bottom、left、right界定当前未遍历的边界。每层按右→下→左→上四条边遍历：先左到右遍历top行（top++），再上到下遍历right列（right--），若top<=bottom则右到左遍历bottom行（bottom--），若left<=right则下到上遍历left列（left++）。循环条件为top<=bottom且left<=right。注意第三步和第四步需要检查边界避免重复遍历单行或单列的情况。时间O(mn)，空间O(1)（不含输出）。

**优化代码**：

优化重点是原地标记、从角落排除行列、转置翻转或三指针分区。

**易错点**：旋转可转置再翻转；置零用第一行列做标记；行列有序矩阵从角落开始排除。

### 73. 矩阵置零（Medium）

**题目描述**：给定m×n矩阵，若某元素为0则将其所在行和列全部置零。要求原地操作且使用O(1)额外空间。

**为什么属于这个类型**：题目核心是二维坐标移动、映射、边界收缩或原地标记。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：零行的标记和零列的标记可以复用矩阵自身的第一行和第一列——将O(m+n)额外空间压缩到O(1)。

**套用统一模板**：明确 `(i,j)` 的移动规则或新旧坐标映射，再选择合适遍历顺序避免覆盖信息。

**基础模板代码**：

```python
def setZeroes(self, matrix: List[List[int]]) -> None:
    m = len(matrix)
    n = len(matrix[0])
    # 记录第一行和第一列是否原本含0
    first_row_zero = any(matrix[0][j] == 0 for j in range(n))
    first_col_zero = any(matrix[i][0] == 0 for i in range(m))
    # 用第一行/列作为标记空间
    for i in range(1, m):
        for j in range(1, n):
            if matrix[i][j] == 0:
                matrix[i][0] = 0  # 标记第i行
                matrix[0][j] = 0  # 标记第j列
    # 根据标记置零（先处理非首行首列）
    for i in range(1, m):
        if matrix[i][0] == 0:
            for j in range(n):
                matrix[i][j] = 0
    for j in range(1, n):
        if matrix[0][j] == 0:
            for i in range(m):
                matrix[i][j] = 0
    # 处理第一行和第一列
    if first_row_zero:
        for j in range(n):
            matrix[0][j] = 0
    if first_col_zero:
        for i in range(m):
            matrix[i][0] = 0
```

**信息流动 / 窗口变化 / 指针移动过程**：

O(1)空间方案：用矩阵第一行和第一列作为标记数组。第一步先用两个布尔变量记录第一行和第一列本身是否有0。第二步遍历矩阵（除第一行第一列），若matrix[i][j]==0则标记matrix[i][0]=0和matrix[0][j]=0。第三步根据标记将对应行和列置零。第四步根据最初的布尔变量处理第一行和第一列。必须先记录再标记再置零，顺序不能乱，否则标记会被覆盖。时间O(mn)，空间O(1)。

**优化代码**：

优化重点是原地标记、从角落排除行列、转置翻转或三指针分区。

**易错点**：旋转可转置再翻转；置零用第一行列做标记；行列有序矩阵从角落开始排除。

### 240. 搜索二维矩阵II（Medium）

**题目描述**：给定m×n矩阵，每行从左到右升序、每列从上到下升序，判断目标值是否在矩阵中。要求高效。

**为什么属于这个类型**：题目核心是二维坐标移动、映射、边界收缩或原地标记。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：从右上角出发，每次比较可以确定性排除一行或一列——矩阵的行列双单调性提供了方向决策的充分信息。

**套用统一模板**：明确 `(i,j)` 的移动规则或新旧坐标映射，再选择合适遍历顺序避免覆盖信息。

**基础模板代码**：

```python
def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
    if not matrix or not matrix[0]:
        return False
    m = len(matrix)
    n = len(matrix[0])
    # 从右上角出发
    i = 0
    j = n - 1
    while i < m and j >= 0:
        if matrix[i][j] == target:
            return True
        elif matrix[i][j] > target:
            j -= 1  # 排除当前列
        else:
            i += 1  # 排除当前行
    return False
```

**信息流动 / 窗口变化 / 指针移动过程**：

从右上角开始搜索：若matrix[i][j]==target返回true；若matrix[i][j]>target则当前列以下都大于target，排除该列j--；若matrix[i][j]<target则当前行左边都小于target，排除该行i++。每次比较排除一行或一列，最多m+n步。也可从左下角对称搜索（大于target上行i--，小于target右移j++）。时间O(m+n)，空间O(1)。比逐行二分O(mlogn)更优。

**优化代码**：

优化重点是原地标记、从角落排除行列、转置翻转或三指针分区。

**易错点**：旋转可转置再翻转；置零用第一行列做标记；行列有序矩阵从角落开始排除。

### 75. 颜色分类（Medium）

**题目描述**：给定只含0、1、2的数组，原地进行排序使其按0、1、2顺序排列。只能使用常数空间的单趟扫描算法。

**为什么属于这个类型**：题目核心是二维坐标移动、映射、边界收缩或原地标记。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：三路划分的本质是「用两个边界将数组切成三个区间」——一次遍历，每个元素根据值落入对应区间，交换即归类。

**套用统一模板**：明确 `(i,j)` 的移动规则或新旧坐标映射，再选择合适遍历顺序避免覆盖信息。

**基础模板代码**：

```python
def sortColors(self, nums):
    left = 0
    index = 0
    right = len(nums) - 1

    while index <= right:
        if nums[index] == 0:
            temp = nums[left]
            nums[left] = nums[index]
            nums[index] = temp
            left += 1
            index += 1
        elif nums[index] == 2:
            temp = nums[right]
            nums[right] = nums[index]
            nums[index] = temp
            right -= 1
        else:
            index += 1
```

**信息流动 / 窗口变化 / 指针移动过程**：

荷兰国旗问题，三指针法。维护三个区域：[0,p0)全为0，[p0,p2]全为1，(p2,n-1]全为2。指针p0指向0的右边界，p2指向2的左边界，i为当前扫描指针。当nums[i]==0时交换nums[i]和nums[p0]，p0++，i++；当nums[i]==1时i++；当nums[i]==2时交换nums[i]和nums[p2]，p2--，i不动（因为交换过来的元素未扫描需继续判断）。时间O(n)，空间O(1)。一趟扫描完成。

**优化代码**：

优化重点是原地标记、从角落排除行列、转置翻转或三指针分区。

**易错点**：旋转可转置再翻转；置零用第一行列做标记；行列有序矩阵从角落开始排除。

## 堆：动态有序集（优先队列/分治）

### 如何识别这是堆 / 优先队列题

堆适用于动态集合中的极值维护。识别信号是：

1. 题目反复要求当前最大、最小、第 K 大、第 K 小。
2. 集合会动态加入新元素或弹出旧元素。
3. 完整排序成本过高，但每次只关心极值。
4. 多路归并时，每一路当前最小元素竞争全局最小。

判断公式：

> 如果需要在动态候选集合中反复取极值，就考虑堆；如果只静态找第 K 大，也可以考虑快速选择。

### 一句话本质

在动态集合中反复取极值。

### 第一性原理

堆的第一性原理是：当集合不断插入、删除，同时每一步都需要当前最大/最小元素时，完全排序太浪费，堆只维护足够支持极值查询的局部有序性。

### 统一模板：先问 5 个问题

1. 需要动态维护最大值还是最小值？
2. 堆里存值、下标、链表节点还是频次元组？
3. 堆大小是否固定为 `k`？
4. 弹出元素后是否要补入后继元素？
5. 能否用快速选择或 Boyer-Moore 取代堆？

### 通用代码骨架

```python
heap = []
for item in items:
    heappush(heap, key(item))
    if len(heap) > k:
        heappop(heap)
return 堆顶或堆内容
```

### 常见状态变量

- 小顶堆维护前 K 大
- 多路归并堆存每条链/数组当前节点
- 频次堆存 `(freq, value)`
- 快速选择用 partition 维护第 K 位置

### 常见剪枝 / 优化

- 固定大小 K，超过就弹出最小候选
- 多数元素有计数抵消法，不必堆排序
- 链表归并弹出一个节点后只加入它的后继

### 本类题目总览

- 215. 数组中的第K个最大元素（Medium）
- 23. 合并K个升序链表（Hard）
- 169. 多数元素（Easy）
- 287. 寻找重复数（Medium）

### 215. 数组中的第K个最大元素（Medium）

**题目描述**：给定整数数组和整数k，返回数组中第k个最大的元素。要求时间复杂度优于O(nlogn)。

**为什么属于这个类型**：题目需要在动态候选集合中反复取得最大值、最小值或第 K 个元素。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：第K大就是「比它大的只有K-1个」——用小顶堆维护前K大的候选集，堆顶就是答案。

**套用统一模板**：堆里存能比较的 key；每次弹出当前极值；固定 K 时保持堆大小不超过 K。

**基础模板代码**：

```python
import heapq

def findKthLargest(self, nums: List[int], k: int) -> int:
    # 小顶堆维护前K大的元素，堆顶=第K大
    heap = []
    for num in nums:
        heapq.heappush(heap, num)
        if len(heap) > k:
            heapq.heappop(heap)  # 弹出最小的，保留大的
    return heap[0]  # 堆顶即第K大
```

**信息流动 / 窗口变化 / 指针移动过程**：

三种方案：(1)最小堆维护大小为k的堆，遍历后堆顶即为第k大，O(nlogk)；(2)快速选择算法，基于partition，每次将枢轴放到最终位置，若恰为第n-k个位置则返回，否则递归对应一侧，平均O(n)最坏O(n²)；(3)堆排序取第k大。面试推荐快速选择：随机选枢轴避免最坏情况，partition后根据枢轴位置与目标位置的关系决定搜索方向。也可用库函数nlargest，但面试需手写。

**优化代码**：

优化重点是固定大小 K、小顶/大顶选择、多路归并或快速选择替代。

**易错点**：第 K 大可用小顶堆或快速选择；多路归并弹出节点后放入后继；多数元素可用投票法更优。

### 23. 合并K个升序链表（Hard）

**题目描述**：给定k个升序链表的数组，将所有链表合并为一个升序链表并返回。链表总数可达10^4个。

**为什么属于这个类型**：题目需要在动态候选集合中反复取得最大值、最小值或第 K 个元素。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：K路归并的核心是「每次取K个候选中的最小值」——堆天然维护这个动态最小值查询。

**套用统一模板**：堆里存能比较的 key；每次弹出当前极值；固定 K 时保持堆大小不超过 K。

**基础模板代码**：

```python
import heapq

def mergeKLists(self, lists: List[Optional[ListNode]]) -> Optional[ListNode]:
    # 最小堆：(节点值, 链表索引, 节点)
    heap = []
    for i, node in enumerate(lists):
        if node:
            heapq.heappush(heap, (node.val, i, node))
    dummy = ListNode(0)
    cur = dummy
    while heap:
        val, i, node = heapq.heappop(heap)  # 取当前最小
        cur.next = node
        cur = cur.next
        if node.next:  # 该链表的下一个节点入堆
            heapq.heappush(heap, (node.next.val, i, node.next))
    return dummy.next
```

**信息流动 / 窗口变化 / 指针移动过程**：

最优方案：最小堆。将所有链表头节点放入最小堆（按val排序），每次弹出最小节点接入结果，并将该节点的next压入堆。时间O(Nlogk)（N为总节点数，k为链表数），空间O(k)（堆大小）。分治合并也可：两两配对合并，类似归并排序，时间O(Nlogk)，空间O(logk)递归栈。暴力逐个合并O(kN)最差。推荐最小堆方案，实现简洁效率最优。

**优化代码**：

优化重点是固定大小 K、小顶/大顶选择、多路归并或快速选择替代。

**易错点**：第 K 大可用小顶堆或快速选择；多路归并弹出节点后放入后继；多数元素可用投票法更优。

### 169. 多数元素（Easy）

**题目描述**：给定大小为n的数组，找出其中出现次数超过⌊n/2⌋的元素。保证多数元素存在。

**为什么属于这个类型**：题目需要在动态候选集合中反复取得最大值、最小值或第 K 个元素。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：多数元素出现超过半数——用抵消法，不同元素成对消灭，最后剩下的一定是多数。

**套用统一模板**：堆里存能比较的 key；每次弹出当前极值；固定 K 时保持堆大小不超过 K。

**基础模板代码**：

```python
def majorityElement(self, nums: List[int]) -> int:
    candidate = None
    count = 0
    for num in nums:
        if count == 0:
            candidate = num  # 重置候选者
            count = 1
        elif num == candidate:
            count += 1  # 同阵营+1
        else:
            count -= 1  # 异阵营抵消
    return candidate  # 多数元素必然存活
```

**信息流动 / 窗口变化 / 指针移动过程**：

Boyer-Moore投票算法：遍历数组，维护候选者candidate和计数count。count为0时将当前元素设为candidate，当前元素等于candidate则count++，否则count--。核心原理：多数元素出现次数>n/2，即使所有非多数元素抵消多数元素，多数元素仍有剩余。最终candidate即为答案。时间O(n)，空间O(1)。也可用哈希表统计频率O(n)空间，但投票法最优。

**优化代码**：

优化重点是固定大小 K、小顶/大顶选择、多路归并或快速选择替代。

**易错点**：第 K 大可用小顶堆或快速选择；多路归并弹出节点后放入后继；多数元素可用投票法更优。

### 287. 寻找重复数（Medium）

**题目描述**：给定含n+1个整数的数组nums，数字范围在[1,n]内，只有一个重复数字（可能重复多次），找出该重复数。要求不修改数组且O(1)额外空间。

**为什么属于这个类型**：题目需要在动态候选集合中反复取得最大值、最小值或第 K 个元素。

**从暴力思路出发**：如果直接枚举所有可能答案或所有操作顺序，通常会产生重复计算或无效候选。需要利用本类型的结构把搜索空间压缩。

**发现可复用结构 / 单调性 / 选择树**：数组值域[1,n]且含重复——值可以视为指针，数组变成链表，重复值造成环，Floyd找环入口即重复数。

**套用统一模板**：堆里存能比较的 key；每次弹出当前极值；固定 K 时保持堆大小不超过 K。

**基础模板代码**：

```python
def findDuplicate(self, nums: List[int]) -> int:
    # Floyd判圈：快慢指针找相遇点
    slow = 0
    fast = 0
    while True:
        slow = nums[slow]       # 慢指针走一步
        fast = nums[nums[fast]] # 快指针走两步
        if slow == fast:
            break
    # 找环入口=重复数
    slow = 0
    while slow != fast:
        slow = nums[slow]
        fast = nums[fast]
    return slow
```

**信息流动 / 窗口变化 / 指针移动过程**：

将数组视为链表——值nums[i]视为从i指向nums[i]的指针，因范围[1,n]且共n+1个元素，必有环且入环点即为重复数。Floyd判圈：快慢指针从0出发，慢指针走nums[slow]，快指针走nums[nums[fast]]，相遇后一个指针回0，两指针每次各走一步再次相遇即为重复数。原理与#142相同。此法不修改数组、O(1)空间、O(n)时间，满足全部约束。也可二分：对[1,n]二分，计数<=mid的个数判断重复在哪一侧，O(nlogn)。

**优化代码**：

优化重点是固定大小 K、小顶/大顶选择、多路归并或快速选择替代。

**易错点**：第 K 大可用小顶堆或快速选择；多路归并弹出节点后放入后继；多数元素可用投票法更优。

