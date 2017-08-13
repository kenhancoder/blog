---
title: Python基础复习-Set(一)
date: 2017-08-13 11:03:06
tags: Set
category: Python
---

### Set（ 集合）
花大括号或 set() 函数可以用于创建集合。注意： 若要创建一个空集必须使用set()，而不能用 {}    
set() -> new empty set object    
set(iterable) -> new set object    
集合中的元素不会重复且没有顺序。    
集合的基本用途包括成员测试和消除重复条目。    
集合对象还支持并集（union）、 交集（intersection）、 差（difference）和对称差（symmetric_difference）等数学运算。    
查询时用到了Hash，时间在O(1)级别    
set是无序unique值的集合，常用来去重，检验membership等。
set类似一个词典，但只有键key，没有值value，好多操作也类似，但不支持索引，切片等操作。


### 特点：
1. 无序
2. 元素不重复
3. 成员检测时效率快


### Methods
>set.add(x)    
添加一个元素x到集合中。如果元素已存在, 该操作是无效的。因为集合里的元素是不重复的。    
<br>    
>set.clear()    
清空集合内的元素。集合变为空集    
<br>    
>set.copy()    
返回集合的一个浅拷贝。    
<br>    
>set.difference(...)    
返回一个新集合，新集合为两个或者多个集合的差集。例 a.difference(b,c)，返回元素在a中，但不在b、c中    
<br>    
>set.difference_update(...)    
将别的集合中的元素从本集合中删除。例  a.difference_update(b, c), 将b、c中的元素从a中删除    
<br>    
>set.discard(x)    
删除集合中的一个元素x。如果x不是集合中的成员，该操作是无效的。
<br>    
>set.intersection(...)    
返回一个新集合，新集合为两个或者多个集合的交集。    
<br>    
>set.intersection_update(...)    
将集合中的元素更新为与其它集合的交集。例 a.intersection_update(b,c)，a中的元素为a、b、c的交集    
<br>    
>set.isdisjoint(s)    
如果两个集合的交集为空集，则返回True，反之为False    
<br>    
>set.issubset(s)    
如果集合s包含本集合set，则返回True，反之为False    
<br>    
>set.issuperset(s)    
如果本集合set包含集合s，则返回True，反之为False    
<br>    
>set.pop()    
删除并且返回集合set中的一个不确定的元素, 如果集合set为空，将会抛出异常KeyError    
<br>    
>set.remove(e)    
删除集合set中的一个元素e，如果元素e不是集合set的成员，将会抛出异常KeyError    
<br>    
>set.symmetric_difference(s)    
返回一个新的对称差集合，即该集合包含集合set和集合s中不重复的元素，同并集减交集    
<br>    
>set.symmetric_difference_update(s)    
将集合set更新为集合set和集合s的对称差集合    
<br>    
>set.union(...)    
返回一个新的集合，该集合为两个或者多个集合的并集。    
<br>    
>set.update(...)    
将集合set更新为多个集合的并集    


### 集合操作符号
>差集：-    
>交集：&    
>并集：|    
>等于，不等于：==，！=    
>属于，不属于：in, not in    
