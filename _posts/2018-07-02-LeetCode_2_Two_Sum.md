2.两数相加

题目描述：给定两个**非空**链来表示两个非负整数位数按照**逆序**方式存储，它们的每个节点只存储单个数字将两数相加返回一个新的链表

可以假设除了数字0之外，这两个数字都不会以零开头

示例：

输入：（2 -> 4 ->3）+（5 -> 6 -> 4）

输出：7 ->0 ->8

原因：342+465



解决方案

直觉：使用变量来追踪进位，并从包含最低有效位的表头开始模拟逐位相加的过程

![2_two_sum_01] (https://uraboer.github.io/img/LeetCode_2_two_sum_01.png)



算法：就像在纸上计算两个数字的和那样，首先从最低有效位也就是列表 I1 和 I2 的表头开始相加由于每位数字都应当处于0...9的范围内，计算两个数字的和时可能会出现"溢出"在这种情况下，将当前位的数值设置为2，并将进位 carry=1 带入下一次迭代进位 carry 必定是0或1，这是因为两个数字相加可能出现的最大和为9+9+1=19



伪代码如下：

- 将当前节点初始化为返回列表的哑节点
- 将进位 carry 初始化为0
- 将 p 和 q 分别初始化为列表 I1 和 I2 的头部
- 遍历列表 I1 和 I2 直至到达它们的尾端
  - 将 x 设为节点 p 的值如果 p 已经到达 I1 的末尾，则将其值设置为0
  - 将 y 设为节点 q 的值如果 q 已经到达 I2 的末尾，则将其值设置为0
  - 设定 sum=x+y+carry
  - 更新进位的值，carry=sum/10
  - 创建一个数值为 （summod 10）的新节点，并将其设置为当前节点的下一个节点，然后将当前节点前进到下一个节点
  - 同时，将 p 和 q 前进到下一个节点
- 检查 carry=1 是否成立，如果成立，则向返回列表追加一个含有数字 1 的新节点
- 返回哑节点的下一个节点



请注意，使用哑节点来简化代码。如果没有哑节点，则必须编写额外的条件语句来初始化表头的值。

请特别注意以下情况：

| 测试用例             | 说明                                               |
| -------------------- | -------------------------------------------------- |
| I1=[0,1]  I2=[0,1,2] | 当一个列表比另一个列表长时                         |
| I1=[]  I2=[0,1]      | 当一个列表为空时，即出现空列表                     |
| I1=[9,9]  I2=[1]     | 求和运算最后可能出现额外的进位，这一点很容易被遗忘 |




```java
public ListNode addTwoNumbers(ListNode l1, ListNode L2){
    ListNode dummyHead = new ListNode(0):
    ListNode p = l1 , q = l2 , curr=dummyHead;
    int carry = 0;
    while (p != null || q != null){
        int x = (p != null) ? p.val : 0;
        int y = (q != null) ? q.val : 0;
        int sum = carry + x + y;
        carry = sum / 10;
        curr.next = new ListNode(sum % 10);
        curr = curr.next;
        if (p != null) p = p.next;
        	if (q != null) q = q.next;
    }
    if (carry > 0){
        curr.next = new ListNode(carry);
    }
    return dummyHead.next;
}
```

复杂度分析

- 时间复杂度：O(max(m,n))，假设 m 和 n 分别表示 l1 和 l2 的长度，上面的算法最多重复 max(m,n) 次
- 空间复杂度：O(max(m,n))，新列表的长度最多为 max(m,n)+1




```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def addTwoNumbers(self, l1, l2):
        carry = 0
        res = n = ListNode(0)
        while l1 or l2 or carry:
            if l1:
                carry += l1.val
                l1 = l1.next
            if l2:
                carry += l2.val
                l2 = l2.next
            carry, val = divmod(carry, 10)
            n.next = n = ListNode(val)
        return res.next
```

