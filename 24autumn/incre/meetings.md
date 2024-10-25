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

## 10.23

1. 这个 program 和图里面会涉及到、多条 rule 指向一个 head predicate、而且也可能会是一条 rule 里面的 body 中有多个 predicate 才能导出一个 head、这个在图里面怎么表示？(从 iTemporal 生成的示例来看、会通过不同边的类型来进行区分、有些是 union edge, 有些是 intersection edge)（不过如果我们做 DRed 的 stratifaction 的话、可能不会受到这个影响、因为还是按层级来进行推理）（然后这个 stratifaction 也会存在环的问题）（需要看一下 MeTeoR 里面对于 dependency graph 的处理）（或者因为没有 negation、根本不需要做 stratifaction？，因为定义是大于等于就行了）

2. 所以一个实现的小问题在于分层的必要性。是不是只有 negative 的时候需要用到分层？

3. MeTeoR 实现里的 rule body 的 literal 是按照先 binaryliteral，再以 arity 的降序进行排序的

4. MeTeoR 实现里、对于 fact 的管理、其实是在 D[predicate][entity]的 list 中存储了很多个 interval 的序列、然后在每一轮 materialize 的结束之后、通过 coalescing 函数进行合并

5. 然后如果对于朴素的 DRED 迁移到 DatalogMTL 上、其中集合的减去应该是不会根据 interval、而是最下沉到 entity constant。可能需要先实现两种集合相减的函数、一种是下沉到了 interval、而另一种是下沉到 entity 就结束。

6. 对于集合的并集操作、对 dataset 里的每个 entity 的 interval list push 之后 然后 coalesce 一下就行了

7. 对于集合的交集操作、需要额外对每个 interval 进行判定处理、然后用 union 函数进行合并、找出其中的交集区间

8. DatalogMTL 中的一个 head 是不带 sometimes 和 since 和 until 的，所以每次 apply 一个 rule 新得到的结果要么是一段 interval、要么是一个 punctuation

9. 如果我们将 DREDc 直接应用到 DatalogMTL 中、那么猜测它在做 overdelete 的时候、删除这个 fact 是依然成立的，即当一个 fact 的 non-recursive counter 为 0 的时候，就可以删除这个 fact。如果一个 fact 的 non-recursive counter 是 0，而且 recursive counter 是非零、那么就 rederive 它。但是这个 rederive 之后的 interval 具体是哪些？同样地、如果在做 conter 的减少的时候、真实情况下这个 counter 减少的 fact，其中包含的 intervals 肯定是最终得到的内容的子集。（只考虑 delete）

10. 如果没有分层 stratification，那么这步 rederivation 能不能用 magic set 来做？我们如果能保证在 overdelete 的时候、能够把所有的 interval 维护正确的结果、那么在这里可以通过判别它的 recursive counter 是不是大于 0，如果大于 0，那么用 magic set 可以很快地推出它所有的 interval 的结果（只需要加入 interval 为（-inf,+inf）的 magic fact 就可以了）。好像不行。

11. 而且还有这个 E-的问题、它是不是要允许能够减去一段 interval、还是减去一个原始的 fact

12. 还需要考虑的是：因为 MeTeoR 在推理的时候、是先用 naive join 找到 delta_new (a dictionary object): store new materialized results.

    ```python
    delta_new = naive_immediate_consequence_operator(D=CR.D, rules=CR.Program, D_index=CR.D_index)
    ```

    找到了新推出的结果，然后再添加原来的 interval list 里面、然后再合并

    ```python
    CR.D[tmp_predicate][tmp_entity] += delta_new[tmp_predicate][tmp_entity]
    coalescing_d(CR.D)
    ```

    也有下面的对于每个规则进行应用并得到新的结果的过程，可以直接使用

    ```python
    def naive_immediate_consequence_operator(rules, D, D_index):
        delta_new = defaultdict(lambda: defaultdict(list))
        for i, rule in enumerate(rules):
            naive_join(rule, D, delta_new, D_index)
        return delta_new
    ```

13. 貌似不太理解的点：insert 的 seminaive 的过程？还有 B/F 的 check 和 saturate 的过程？如果我们同样能够 deletes precisely those facts that no longer hold after the update, rederivation is not needed.

14. 这里的 DatalogMTL 的 seminaive 过程、在另一篇文章里有提到，待进一步研究一下是否是类似的 ？或许可以将 fact entailment 里面的 materialization 的过程改变为 seminaive 的。

15. 通过 kmax 进行，在 overdeletion 的时候，可能不能用 kmax。可以记录 materialize 的轮次和 overdeletion 的轮次。正向 materialization 的时候记录轮次。可以先聚焦在第一个的条件上、即上一轮里的 fact 是否和原来的一样的终止条件。
