# leetcode题集--链表


<!--more-->


## 题目
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201208184555981.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)

## 思路

#### 1、用List集合
先将一个链表headA的节点都存入集合list中，然后遍历另一个链表headB时，比较headB的每个节点是否存在集合list中，如果存在，就返回这个节点；如果不存在，返回空节点
```java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    	//链表为空的情况
       if(headA==null||headB==null){
           return null;
       }
       //定义集合list
       ArrayList list=new ArrayList();
       ListNode p=headA;
       //遍历链表headA,将链表节点存入集合list
       while(p!=null){
           list.add(p);
           p=p.next;
       }
       //遍历链表headB
       ListNode q=headB;
       while(q!=null){
       	   //比较每个节点是否存在集合list中
           if(list.contains(q)){
               return q;
           }
           q=q.next;
       }
        return null;
        
    }
}
```
![效率比较低](https://img-blog.csdnimg.cn/2020120909264085.png#pic_center)
集合的比较消耗比较多，整体效率比较低

####  2 用链表长度
先计算两个链表的长度，让长的链表走多出来的距离，使得链表headA和headB等长。然后headA和headB同步走，如果这时两个节点headA等于headB，返回这个节点；如果遍历完节点也不满足这时的两个节点headA等于headB，则返回空节点。
```java
public class Solution {

    public int length(ListNode p){
        int count=0;
        while(p!=null){
            count+=1;
            p=p.next;
        }
        return count;
    }
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
       if(headA==null||headB==null){
           return null;
       }
       //统计链表A和链表B的长度
    int lenA = length(headA), lenB = length(headB);
    while(lenA!=lenB){
        if(lenA>lenB){
            headA=headA.next;
            lenA--;
        }
        if(lenA<lenB){
            headB=headB.next;
            lenB--;
        }
    }
    while(headA!=null&& headB!=null){
        if(headA==headB){
            return headA;
        }
        headA=headA.next;
        headB=headB.next;
    }
    return null;   
    }
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/202012091826043.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
#### 3、双指针
假设公共链表长度为x,链表headA为lenA,链表headB为lenB，
两个指针跑完链表A和链表B，
	p 节点：lenA+lenB-X
	q 节点：lenA+lenB-X
	则 p、q指针相遇的那个节点就是公共链表的头节点
	分为三个阶段：（假设链表A更长，指针p、q分别从链表A、B出发）
		阶段一：指针q 走完了链表B，指针p还在走链表B
		阶段二：指针q 从链表A开始走，指针p 走完了链表A
		阶段三：指针q正在走链表A，指针p开始走链表B
		阶段四：指针p、q相遇在公共节点，返回公共节点
					  或者 都走完了各自的链表，返回空节点
		
		

```java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        ListNode p=headA;
        ListNode q=headB;
        while(p!=q){
           if(p==null){
               p=headB;
           }else{
               p=p.next;
           }
            if(q==null){
               q=headA;
           }else{
               q=q.next;
           }
        }
        return p;      
    }
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201209184800693.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)

**（参考leetcode题解，本篇文章内容只是个人解题思路总结）**

