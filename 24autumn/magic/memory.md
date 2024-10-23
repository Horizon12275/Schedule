### 内存测试

1. 发现内存测试可以是测当前进程所占用的最大内存、也可以是测当前进程所占用的实际内存。

2. 如果连续做这个内存测试、会出现前几个内存小、后面内存就一样的问题、比如 iTemporal magic not entailed。所以对于对比测试、可以通过简单的每个进程测一次 predicate 进行测试。（使用 sh 脚本）

3. 测试：先试试看对比实验，即 baseline 和 magic 对于几个固定 predicate 的情况对比一下，然后想想 scability 是否需要做然后如何做。

4. 细节回顾：当初这个 weather 是一个一个测的、是因为如果不是一个个测的话、他测出来的时间会有问题

5. 第一次测下来、这里面的 magic 和非 magic 的内存对比有大有小。分析原因是因为测 magic 的时候、保留了复制了一份 dataset，所以有时候、这个内存消耗数是非 magic 的两倍。

6. 然后也要考虑到其中 load dataset for program 的影响、即 magic 会因为 program 的过滤而减少读入内存的数据量、所以内存消耗会减少。（不过最后一版的实验里面貌似把这个函数删掉了）（最终决定不使用这个函数、因为之前的论文没有用到）

7. 接着尝试记录这个 dataset 所占用的内存大小、而不是当前进程所占用的内存大小。

8. 根据 Seminaïve Materialisation in DatalogMTL 文章里的实验、他这里也将用了 dataset 里面的 fact 条数来代替内存的影响的。最后测试中也采用这种方法了。

```python
# 记录 dataset 所占用的内存大小（实际上是记录了 dataset 里面的 fact 条数）
# 下面是MeTeoR里面原始的operate_dataset中的一段代码，类似于记录了dataset里的fact条数
# 其实可以观察到、他这里的fact条数指的是每个gound fact的所有intervals的个数，因此我们采用了类似的方法进行计算
    for predicate in D:
        number_of_facts = 0
        for entity, intervals in D[predicate].items():
            number_of_facts = number_of_facts + len(intervals)

# 在operate_dataset.py中加入了如下函数、用于计算dataset里面的fact条数
def count_facts(D):
    """
    Count the number of facts in D
    Args:
        D (a dictionary object):

    Returns:
        number_of_facts (int): the number of facts in D
    """
    number_of_facts = 0
    for predicate in D:
        for entity, intervals in D[predicate].items():
            number_of_facts = number_of_facts + len(intervals)
    return number_of_facts
```

### To Do List

- 那我现在计划进行如下的测试：

  0. 暂时先使用并行的测试、即不将进程绑定到固定的 cpu 上并设置进程最高优先级。
  1. 不使用 load_dataset_for_program，因为先前的论文里貌似没有用到这个函数
  2. 通过记录 dataset 里面的 fact 条数进行内存记录（方法如下图代码、和原来 MeTeoR 中的工具函数类似，即计算对应每个 gound 的所有 intervals 个数，先前的论文里使用）
  3. 进行 magic vs baseline 对比实验，在对于每一个 query 的测试中记录两个数据：推理前的 dataset fact 总数和推理后的 dataset fact 总数（就也可以得到 delta 了）
  4. 进行 magic scalability 实验，在每一个数据规模里面、分成 entailed 和 not entailed 两种情况、对于每一个 predicate 的情况下、记录两个数据：推理前的 dataset fact 总数和推理后的 dataset fact 总数
  5. 对于 baseline 方法、仅记录一次推出 cr 后的 fact 总数即可，填入对比实验的表格中

- 根据 excel 表格中的内容进行实验、并记录下来

- 发现其中 baseline 使用了 load program for dataset，会在原始的 program 下就少 load 1w 条 fact 左右

- 发现了一个小问题、就是 iTemporal scalability 里面我们当时忘记把先前的加入 magic fact 的代码删掉了，导致多了一条 fact

```python
#用了
Fact Number Before: 505875
#没用
Fact Number Before: 630580
```

#### Comparison

- 貌似不需要写下面的脚本了，因为现在只需要记录 fact 的个数即可，而不需要记录内存的大小。

- 不过做 weather 的时候最后还是用到了脚本，因为其中的代码当时没有重构，懒得再改了。

- 可以进行记录的是：原有 dataset 中的 fact 条数、magic 之后的 dataset 中的 fact 条数、baseline 之后的 dataset 中的 fact 条数。

- ~~把这个对比实验、全部写成参数加命令行的形式启动、然后写一个脚本、可以一次性启动所有的对比实验。~~

- ~~其中 baseline.py 不需要修改参数、其他的 magic 都需要参数。~~

1. iTemporal

   - iTemporal_baseline.py
   - iTemporal_magic_fixed_query_entailed.py
   - iTemporal_magic_fixed_query_not_entailed.py

2. LUBM

   - LUBM_baseline.py
   - LUBM_magic_fixed_query_entailed.py
   - LUBM_magic_fixed_query_not_entailed.py

3. weather

   - weather_baseline.py
   - weather_magic_fixed_entailed.py
   - weather_magic_fixed_not_entailed.py

```sh
#!/bin/bash

# weather magic entailed has 12 queries
for i in {0..11}
do
    python weather_magic_fixed_entailed.py --query $i
done
```

```sh
#!/bin/bash

# weather magic not entailed has 16 queries
for i in {0..15}
do
    python weather_magic_fixed_not_entailed.py --query $i
done
```

#### Scalability

- 用类似的方法、记录在 scalability 的情况下、fact 的个数，包括原有 dataset 中的 fact 条数、magic 之后的 dataset 中的 fact 条数、baseline 之后的 dataset 中的 fact 条数。（这样也就能得出 delta 的增加的条数）

### 报告生成脚本

- 根据先前的经验、如果手动来将一个个整理好的 excel 进行复制黏贴、或者用指定 python 脚本来生成小报告后再复制黏贴、还是会存在因为数据集很多、导致每次这个操作都很繁琐。

- 因此需要分别给对比实验和 scalability 实验写一个脚本、可以直接生成最终的报告格式的结果，而不需要编辑代码中的内容后再复制黏贴。

#### 对比实验报告生成脚本

- 仔细斟酌了一下、因为当时这个命名规范都有点乱、所以准备先从 lubm 的测试开始编写专门的脚本、然后看一下能不能从中提取出特定函数、然后再进行其他数据集的自动报告生成。

- lubm 有 3 个需要处理的文件夹、lubm_baseline_results，lubm_new_program_results_entailed 和 lubm_new_program_results_not_entailed

- 其中在 lubm_baseline_results 下、txt 的命名格式都是 Chair(ID2002)@[8,8]\_inside.txt 这样的

- 在 lubm_new_program_results_entailed 下、txt 的命名格式是这样的 Chair(ID2002)@8_magic_new_inside.txt，同时也存在 outside 的情况 Faculty(ID26)@149_magic_new_outside.txt

- 在 lubm_new_program_results_not_entailed 下、txt 也只存在 outside 的情况 Course(ID2)@69_magic_new_outside.txt

- 但对于每一个 txt 文档，都是要解析出 txt 文档中 Fact Number Before: 630580 Fact Number After: 628320 后面的数字这样的数据

- 现在需要对于每一个文件夹、生成一个 excel 表格，每个表格包含 3 列：query、Fact Number Before、Fact Number After，其中 query 是文件名中 Chair(ID2002)@[8,8] 的样子，即除去了后续的\_inside，\_magic_new_outside 或者\_magic_new_inside，Fact Number Before 是 Fact Number Before: 630580 中的数字，Fact Number After 是 Fact Number After: 628320 中的数字

- 实现在了 cmp_script.py 中

#### Scalability 实验报告生成脚本

- 所有的 scalability 的数据日志文件都生成在了 scalability_results 文件夹下，在这个文件夹下、我们需要手机并整理 lubm 和 iTeporal 的 scalability 结果数据。

- 在这个 scalability_results 文件夹中、我们首先需要找到文件名类似于 iTemporal_10^1_statistics_10_23.xlsx，iTemporal_10^2_statistics_10_23.xlsx， lubm_10^2_statistics_10_23.xlsx， lubm_10^3_statistics_10_23.xlsx 这样的 excel 文件，然后将这些文件中的数据提取出来，

- 在这样的\_statistics 的 excel 文件中、他们内容的格式都是一样的，第一列是 Predicate，第二列是 Average Facts Number Before(Entailed)，第三列是 Average Facts Number After(Entailed)，第四列是 Max Facts Number After(Entailed)，第五列是 Average Facts Number Before(Not Entailed)，第六列是 Average Facts Number After(Not Entailed)，第七列是 Max Facts Number After(Not Entailed)

- 对于每一种数据集、比如 lubm，希望将每一个数据规模，即 10^2 到 10^6 的 statistics 数据提取出来，汇总到一个 lubm_scalability_results.xlsx 的 excel 文件中。在这个文件中、有四个 sheet，分别用于存储 Average Facts Number After(Entailed)，Max Facts Number After(Entailed)，Average Facts Number After(Not Entailed)，Max Facts Number After(Not Entailed) 这四种数据，其中每一列对应一个数据规模，每一行对应一个 Predicate。

- 同样地、对于 iTemporal，希望将每一个数据规模，即 10^1 到 10^5 的 statistics 数据提取出来，汇总到一个 iTemporal_scalability_results.xlsx 的 excel 文件中。在这个文件中、有四个 sheet，分别用于存储 Average Facts Number After(Entailed)，Max Facts Number After(Entailed)，Average Facts Number After(Not Entailed)，Max Facts Number After(Not Entailed) 这四种数据，其中每一列对应一个数据规模，每一行对应一个 Predicate。

- 每个 Sheet 最终的格式如下：

| Predicate | 10^1 | 10^2 | 10^3 | 10^4 | 10^5 |
| --------- | ---- | ---- | ---- | ---- | ---- |
| P1        |      |      |      |      |      |
| P2        |      |      |      |      |      |
| P3        |      |      |      |      |      |

- 实现在了 scala_script.py 中

- excel 的一个 trick，可以让公式里面的=B7-$B$15，就可以做到在拖动公式的时候，$B$15 这个值不变，而 B7 这个值会变化。
