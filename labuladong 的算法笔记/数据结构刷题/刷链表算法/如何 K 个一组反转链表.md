| LeetCode                                                                                | 力扣                                                                      | 难度  |
| --------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- | --- |
| [25. Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/) | [25. K 个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group/) | 🔴  |
```ad-question
title: 25. K 个一组翻转链表
给你链表的头节点 `head` ，每 `k` 个节点一组进行翻转，请你返回修改后的链表。

`k` 是一个正整数，它的值小于或等于链表的长度。如果节点总数不是 `k` 的整数倍，那么请将最后剩余的节点保持原有顺序。

你不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。

**示例 1：**
![[Pasted image 20240621143237.png]]
**输入：**head = [1,2,3,4,5], k = 2
**输出：**[2,1,4,3,5]

**示例 2：**
![[Pasted image 20240621143246.png]]
**输入：**head = [1,2,3,4,5], k = 3
**输出：**[3,2,1,4,5]

**提示：**
- 链表中的节点数目为 `n`
- `1 <= k <= n <= 5000`
- `0 <= Node.val <= 1000`

**进阶：**你可以设计一个只用 `O(1)` 额外内存空间的算法解决此问题吗？

```

### 分析问题
链表是一种兼具递归和迭代性质的数据结构，该**问题具有递归性质**。比如说我们对这个链表调用 `reverseKGroup(head, 2)`，即以 2 个节点为一组反转链表：
![[Pasted image 20240621164454.png|575]]
如果我设法把前 2 个节点反转，那么后面的那些节点怎么处理？后面的这些节点也是一条链表，而且规模（长度）比原来这条链表小，这就叫**子问题**。
![[Pasted image 20240621164513.png|575]]
我们可以把原先的 `head` 指针移动到后面这一段链表的开头，然后继续递归调用 `reverseKGroup(head, 2)`，因为子问题（后面这部分链表）和原问题（整条链表）的结构完全相同，这就是所谓的递归性质。

发现了递归性质，就可以得到大致的算法流程：
**1、先反转以 `head` 开头的 `k` 个元素**。
![[Pasted image 20240621164734.png|575]]
**2、将第 `k + 1` 个元素作为 `head` 递归调用 `reverseKGroup` 函数**。
![[Pasted image 20240621164756.png|575]]
**3、将上述两个过程的结果连接起来**。
![[Pasted image 20240621164818.png|575]]

### 代码实现
首先，我们要实现一个 `reverse` 函数反转一个区间之内的元素。在此之前我们再简化一下，给定链表头结点，如何反转整个链表？
```python
# 反转以 a 为头结点的链表
def reverse(a: ListNode) -> ListNode:
    pre, cur, nxt = None, a, a
    while cur:
        nxt = cur.next
        # 逐个结点反转
        cur.next = pre
        # 更新指针位置
        pre = cur
        cur = nxt
    # 返回反转后的头结点
    return pre
```
算法执行的过程如下 GIF 所示：：
![[8.gif|625]]
这次使用迭代思路来实现的，借助动画理解应该很容易。

「反转以 `a` 为头结点的链表」其实就是「反转 `a` 到 null 之间的结点」，那么如果让你「反转 `a` 到 `b` 之间的结点」，你会不会？

只要更改函数签名，并把上面的代码中 `null` 改成 `b` 即可：
```python
# 反转区间 [a, b) 的元素，注意是左闭右开
def reverse(a:ListNode, b:ListNode) -> ListNode:
    pre, cur, nxt = None, a, a
    # while  终止的条件改一下就行了
    while cur != b:
        nxt = cur.next
        cur.next = pre
        pre = cur
        cur = nxt
    # 返回反转后的头结点
    return pre
```
现在我们迭代实现了反转部分链表的功能，接下来就按照之前的逻辑编写 `reverseKGroup` 函数即可，完整代码如下：
```python
def reverseLR(left, right):  
    pre, cur, nxt = None, left, left  
    while cur != right:  
        nxt = cur.next  
        # 逐个结点反转  
        cur.next = pre  
        # 更新指针位置  
        pre = cur  
        cur = nxt  
    # 返回反转后的头结点  
    return pre
  
class Solution:  
    def reverseKGroup(self, head: Optional[ListNode], k: int) -> Optional[ListNode]:  
        if not head:  
            return None  
  
        # 区间 [left, right) 包含 k 个待反转元素  
        left, right = head, head  
  
        for i in range(k):  
            # 不足 k 个，不需要反转，base case  
            if not right:  
                return head  
            right = right.next  
  
        new_head = reverseLR(left, right)  
        left.next = self.reverseKGroup(right, k)  
  
        return new_head
```
解释一下 `for` 循环之后的几句代码，注意 `reverse` 函数是反转区间 `[a, b)`，所以情形是这样的：
![[Pasted image 20240621165419.png|600]]
递归部分就不展开了，整个函数递归完成之后就是这个结果，完全符合题意：
![[Pasted image 20240621165441.png|600]]
