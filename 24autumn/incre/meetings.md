## 10.11

0. naive join 函数貌似就是一轮 program 的应用，得到的 delta new 是在这轮应用中新推出来的所有 data。然后通过下面的语句进行合并、其中 delta_new[tmp_predicate][tmp_entity]是一个 list、存储了新推出来的这个 fact 的所有 intervals，所以会涉及到后续的合并操作 coalescing_d(CR.D)，即把这个 dataset 里所有的 fact 的 interval 进行合并。

```python
CR.D[tmp_predicate][tmp_entity] += delta_new[tmp_predicate][tmp_entity]
```

1. 类比 AAAI-18 的一种思路：尝试将 counter 加入到 DatalogMTL 语言里的 fact 中

   - 可以在合并的时候加入 counter。原来代码中的合并单条 fact 的所有 intervals 的实现如下，发现涉及到了一个 union 操作、即找出两个 interval 之间的交集。

   ```python
   def coalescing(old_intervals):
       if len(old_intervals) == 0:
           return old_intervals
       new_intervals = []
       old_intervals = sorted(old_intervals, key=lambda t: (t.left_value, t.left_open))
       i = 1
       mover = old_intervals[0]
       while i <= len(old_intervals)-1:
           tmp_interval = Interval.union(mover, old_intervals[i])
           if tmp_interval is None:
               # no intersection
               new_intervals.append(mover)
               mover = old_intervals[i]
           else:
               mover = tmp_interval
           i += 1
       new_intervals.append(mover)
       return new_intervals
   ```

   - 然后进一步地、可以设计 recursive counter 和 non-recursive counter 两种类型的 counter，分别对应于递归和非递归的情况。

   - counter 里面存储的信息：如果是合并了[1,4]和[2,8]，那么应该存储 counter[[1,4]] = 1，counter[[2,8]] = 1。还是 counter[[1,2]] = 1，counter[[2,4]] = 2，counter[[4,8]] = 1？或者说其他形式

2. 感觉可以先考虑非递归的情况。然而非递归的情况 counter 的含义似乎也不能直接被搬过来用？

3. 或者说、我们的目的是、对 dataset 更改之后、减少它算 cr 的时间？就是 find periods 的时间？

## 10.18

1. 现在的问题是 MeTeoR 不支持 stratified negation，但是在 MeTeoR 的实现中，" It would be interesting to extend MeTeoR to support stratified negation, which will require a non trivial revision of our algorithm since materialisation within each separate stratum may be non-terminating."，所以在实现方面只能先不考虑 negation 的情况。

2. 考虑实现 DRed 算法在 DatalogMTL 上的应用，直接照搬先看看会有什么问题和结果，需要实现以下几个部分（由于没有 negation body atom、所以可以简化一些部分）：
   - 检查 dataset 的赋值是不是 deepcopy
   - dataset 差集，并集，交集，检查是否是空集的实现函数
   - 分层化 stratum （需结合图的遍历，拓扑排序）
   - 得到 Os 即第 s 层的所有 predicate
   - OVERDELETE
   - INSERT
   - instr 函数：即挑选出能且实例化后的规则实例 rσ
   - Πs 和 Πsr ：基于 instr 函数的实现，对 rσ 的规则头进行筛选，即挑选出能推出的结果
