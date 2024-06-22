| LeetCode                                                                                                   | 力扣                                                                                      | 难度  |
| ---------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | --- |
| [1. Two Sum](https://leetcode.com/problems/two-sum/)                                                       | [1. 两数之和](https://leetcode.cn/problems/two-sum/)                                        | 🟢  |
| [15. 3Sum](https://leetcode.com/problems/3sum/)                                                            | [15. 三数之和](https://leetcode.cn/problems/3sum/)                                          | 🟠  |
| [167. Two Sum II - Input Array Is Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) | [167. 两数之和 II - 输入有序数组](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/) | 🟠  |
| [18. 4Sum](https://leetcode.com/problems/4sum/)                                                            | [18. 四数之和](https://leetcode.cn/problems/4sum/)                                          | 🟠  |
| -                                                                                                          | [剑指 Offer II 007. 数组中和为 0 的三个数](https://leetcode.cn/problems/1fGaJU/)                   | 🟠  |
### 一、twoSum 问题
1 题「[两数之和](https://leetcode.cn/problems/two-sum/)」和 167 题「[两数之和 II - 输入有序数组](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/)」

很简单，1 题「 [两数之和](https://leetcode.cn/problems/two-sum/) 」由于需要返回下标，不能进行排序操作，直接使用哈希表解决；167 题「 [两数之和 II - 输入有序数组](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/) 」使用双指针处理。双指针代码：
```python
def twoSum(nums: List[int], target: int) -> List[int]:
    # 先对数组排序
    nums.sort()
    # 左右指针
    lo, hi = 0, len(nums) - 1
    while lo < hi:
        sum = nums[lo] + nums[hi]
        # 根据 sum 和 target 的比较，移动左右指针
        if sum < target:
            lo += 1
        elif sum > target:
            hi -= 1
        elif sum == target:
            return [nums[lo], nums[hi]]
    return []
```

魔改题目，把这个题目变得更泛化，更困难一点：
**`nums` 中可能有多对儿元素之和都等于 `target`，请你的算法返回所有和为 `target` 的元素对儿，其中不能出现重复**。
比如说输入为 `nums = [1,3,1,2,2,3], target = 4`，那么算法返回的结果就是：`[[1,3],[2,2]]`（注意，我要求返回元素，而不是索引）。对于修改后的问题，关键难点是现在可能有多个和为 `target` 的数对儿，还不能重复，比如上述例子中 `[1,3]` 和 `[3,1]` 就算重复，只能算一次。

两个关键点：
**1）找到和为 target 的数值后还需要继续寻找，直到走完 nums**
**2）对于重复元素，需要跳过**

代码如下：
```python
from typing import List

def twoSumTarget(nums: List[int], target: int) -> List[List[int]]:
    # nums必须有序
    nums.sort()
    lo, hi = 0, len(nums) - 1
    res = []
    while lo < hi:
        s = nums[lo] + nums[hi]
        left, right = nums[lo], nums[hi]
        if s < target:
            while lo < hi and nums[lo] == left:
                lo += 1
        elif s > target:
            while lo < hi and nums[hi] == right:
                hi -= 1
        else:
            res.append([left, right])
            while lo < hi and nums[lo] == left:
                lo += 1
            while lo < hi and nums[hi] == right:
                hi -= 1
    return res
```

### 二、3Sum 问题
15 题「 [三数之和](https://leetcode.cn/problems/3sum/) 」：
```ad-question
title: 15. 三数之和
给你一个整数数组 `nums` ，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k` ，同时还满足 `nums[i] + nums[j] + nums[k] == 0` 。请

你返回所有和为 `0` 且不重复的三元组。

**注意：**答案中不可以包含重复的三元组。

**示例 1：**
**输入：**nums = [-1,0,1,2,-1,-4]
**输出：**[[-1,-1,2],[-1,0,1]]
**解释：**
nums[0] + nums[1] + nums[2] = (-1) + 0 + 1 = 0 。
nums[1] + nums[2] + nums[4] = 0 + 1 + (-1) = 0 。
nums[0] + nums[3] + nums[4] = (-1) + 2 + (-1) = 0 。
不同的三元组是 [-1,0,1] 和 [-1,-1,2] 。
注意，输出的顺序和三元组的顺序并不重要。

**示例 2：**
**输入：**nums = [0,1,1]
**输出：**[]
**解释：**唯一可能的三元组和不为 0 。

**示例 3：**
**输入：**nums = [0,0,0]
**输出：**[[0,0,0]]
**解释：**唯一可能的三元组和为 0 。

**提示：**
- `3 <= nums.length <= 3000`
- `-105 <= nums[i] <= 105`
```
再泛化一下题目，不要光和为 0 的三元组了，计算和为 `target` 的三元组吧，同上面的 `twoSum` 一样，也不允许重复的结果：
```python
from typing import List

def threeSum(nums: List[int]) -> List[List[int]]:
    # 求和为 0 的三元组
    return threeSumTarget(nums, 0)

def threeSumTarget(nums: List[int], target: int) -> List[List[int]]:
    # 输入数组 nums，返回所有和为 target 的三元组
    pass
```
找和为 `target` 的三个数字，那么对于第一个数字，可能是什么？`nums` 中的每一个元素 `nums[i]` 都有可能！
那么，确定了第一个数字之后，剩下的两个数字可以是什么呢？其实就是和为 `target - nums[i]` 的两个数字呗，那不就是 `twoSum` 函数解决的问题么？

完整代码如下：
```python
class Solution:  
    def twoSumTarget(self, nums: List[int], target: int, start: int) -> List[List[int]]:  
        lo, hi = start, len(nums) - 1  
        res = []  
        while lo < hi:  
            s = nums[lo] + nums[hi]  
            left, right = nums[lo], nums[hi]  
            if s < target:  
                while lo < hi and nums[lo] == left:  
                    lo += 1  
            elif s > target:  
                while lo < hi and nums[hi] == right:  
                    hi -= 1  
            elif s == target:  
                res.append([left, right])  
                while lo < hi and nums[lo] == left:  
                    lo += 1  
                while lo < hi and nums[hi] == right:  
                    hi -= 1  
        return res  
  
    def threeSumTarget(self, nums: List[int], target: int) -> List[List[int]]:  
        n = len(nums)  
        res = []  
        i = 0  
        while i < n:  
            tuples = self.twoSumTarget(nums, target - nums[i], i + 1)  
            for t in tuples:  
                t.append(nums[i])  
                res.append(t)  
            while i < n - 1 and nums[i] == nums[i + 1]:  
                i += 1  
            else:  
                i += 1  
        return res  
  
    def threeSum(self, nums: List[int]) -> List[List[int]]:  
        # nums 必须有序  
        nums.sort()  
        return self.threeSumTarget(nums, 0)
```
**关键点在于，不能让第一个数重复，至于后面的两个数，我们复用的 `twoSum` 函数会保证它们不重复**。所以代码中必须用一个 while 循环来保证 `3Sum` 中第一个元素不重复。

### 三、4Sum 问题
18 题「 [四数之和](https://leetcode.cn/problems/4sum/) 」：
```ad-question
title: 18. 四数之和
给你一个由 `n` 个整数组成的数组 `nums` ，和一个目标值 `target` 。请你找出并返回满足下述全部条件且**不重复**的四元组 `[nums[a], nums[b], nums[c], nums[d]]` （若两个四元组元素一一对应，则认为两个四元组重复）：

- `0 <= a, b, c, d < n`
- `a`、`b`、`c` 和 `d` **互不相同**
- `nums[a] + nums[b] + nums[c] + nums[d] == target`

你可以按 **任意顺序** 返回答案 。

**示例 1：**
**输入：**nums = [1,0,-1,0,-2,2], target = 0
**输出：**[[-2,-1,1,2],[-2,0,0,2],[-1,0,0,1]]

**示例 2：**
**输入：**nums = [2,2,2,2,2], target = 8
**输出：**[[2,2,2,2]]

**提示：**
- `1 <= nums.length <= 200`
- `-109 <= nums[i] <= 109`
- `-109 <= target <= 109`

```
`4Sum` 完全就可以用相同的思路：穷举第一个数字，然后调用 `3Sum` 函数计算剩下三个数，最后组合出和为 `target` 的四元组。

完整代码如下：
```python
class Solution:  
    def twoSumTarget(self, nums: List[int], target: int, start: int) -> List[List[int]]:  
        lo, hi = start, len(nums) - 1  
        res = []  
        while lo < hi:  
            s = nums[lo] + nums[hi]  
            left, right = nums[lo], nums[hi]  
            if s < target:  
                while lo < hi and nums[lo] == left:  
                    lo += 1  
            elif s > target:  
                while lo < hi and nums[hi] == right:  
                    hi -= 1  
            elif s == target:  
                res.append([left, right])  
                while lo < hi and nums[lo] == left:  
                    lo += 1  
                while lo < hi and nums[hi] == right:  
                    hi -= 1  
        return res  
  
    def threeSumTarget(self, nums: List[int], target: int, start: int) -> List[List[int]]:  
        n = len(nums)  
        res = []  
        i = start  
        while i < n:  
            tuples = self.twoSumTarget(nums, target - nums[i], i + 1)  
            for t in tuples:  
                t.append(nums[i])  
                res.append(t)  
            while i < n - 1 and nums[i] == nums[i + 1]:  
                i += 1  
            else:  
                i += 1  
        return res  
  
    def fourSum(self, nums: List[int], target: int) -> List[List[int]]:  
        nums.sort()  
        n = len(nums)  
        res = []  
        i = 0  
        while i < n:  
            triples = self.threeSumTarget(nums, target - nums[i], i + 1)  
            for triple in triples:  
                triple.append(nums[i])  
                res.append(triple)  
            while i < n - 1 and nums[i] == nums[i + 1]:  
                i += 1  
            else:  
                i += 1  
  
        return res
```

### 四、100Sum 问题？
**`3Sum` 和 `4Sum` 的过程，实际上是遵循相同的模式的**。只要稍微修改一下 `4Sum` 的函数就可以复用并解决 `5Sum` 问题，然后解决 `6Sum` 问题……
统一出一个 `nSum` 函数：
```python
from typing import List

# 注意：调用这个函数之前一定要先给 nums 排序。n 填写想求的是几数之和，start 从哪个索引开始计算（一般填 0），target 填想凑出的目标和。
def nSumTarget(nums: List[int], n: int, start: int, target: int) -> List[List[int]]:
    sz = len(nums)
    res = []
    # 至少是 2Sum，且数组大小不应该小于 n
    if n < 2 or sz < n:
        return res
    # 2Sum 是 base case
    if n == 2:
        # 双指针那一套操作
        lo, hi = start, sz - 1
        while lo < hi:
            sum = nums[lo] + nums[hi]
            left, right = nums[lo], nums[hi]
            if sum < target:
                while lo < hi and nums[lo] == left:
                    lo += 1
            elif sum > target:
                while lo < hi and nums[hi] == right:
                    hi -= 1
            else:
                res.append([left, right])
                while lo < hi and nums[lo] == left:
                    lo += 1
                while lo < hi and nums[hi] == right:
                    hi -= 1
    else:
        # n > 2 时，递归计算 (n-1)Sum 的结果
        i = start
        while i < sz:
            sub = nSumTarget(nums, n - 1, i + 1, target - nums[i])
            for arr in sub:
                # (n-1)Sum 加上 nums[i] 就是 nSum
                arr.append(nums[i])
                res.append(arr)
            while i < sz - 1 and nums[i] == nums[i + 1]:
                i += 1
            else:
                i += 1
    return res
```
**需要注意的是，调用这个 `nSumTarget` 函数之前一定要先给 `nums` 数组排序。**

上面的 `4Sum` 问题：
```python
def fourSum(nums: List[int], target: int) -> List[List[int]]:
    nums.sort()
    # n 为 4，从 nums[0] 开始计算和为 target 的四元组
    return nSumTarget(nums, 4, 0, target)
```

`3Sum` 问题，找 `target == 0` 的三元组：
```python
def threeSum(nums: List[int]) -> List[List[int]]:  
    # nums 必须有序  
    nums.sort()  
    return nSumTarget(nums, 3, 0, 0)
```
