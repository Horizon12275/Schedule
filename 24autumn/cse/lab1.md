## Set Proxy for Git

```bash
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

## Windows Docker

```bash
docker create -t -i --privileged --name chfs -v ${PWD}:/home/stu/chfs chfs_image bash
docker start -a -i chfs
```

## Docker Dev Container

- 在 vscode 界面中、点击左下角的蓝色按钮、选择`Attach to Running Container`、然后选择`chfs`、就可以进入容器了，然后会和挂载在硬盘上的文件夹实时同步。

- 但是没有 vscode 插件、需要手动安装。

- 注意在这个插件里打开 terminal 的时候要用 bash 而不是 sh，不然功能不够。

- 测试脚本会有换行符的问题，可以用下面的命令解决：

```bash
cd ./scripts/lab1/
sed -i 's/\r$//' integration_test.sh
sed -i 's/\r$//' handin.sh
sed -i 's/\r$//' set-env.sh
sed -i 's/\r$//' start_fs.sh
sed -i 's/\r$//' stop_fs.sh
sed -i 's/\r$//' test-lab1-part2-a.pl
sed -i 's/\r$//' test-lab1-part2-b.pl
sed -i 's/\r$//' test-lab1-part2-c.pl
sed -i 's/\r$//' test-lab1-part2-d.sh
sed -i 's/\r$//' test-lab1-part2-e.sh
```

- 去左边的 cmake 编译工具里面设置一下编译器，不然静态检查会报错。

- docker 只要不删除容器、即使不挂载的话、电脑重新启动、然后再 start 容器、里面的数据还是会在的、只有删除容器的时候，不挂载的数据才会丢失。所以不用太担心。

## Lab 1

### Part 1: Block Layer

1. Note that the block manager is **not** thread-safe.

2. explicit 关键字用于修饰构造函数，以防止隐式转换。具体来说，当一个构造函数被声明为 explicit 时，这意味着该构造函数不能被用来进行类型转换或初始化其他类型的对象，而必须明确调用。

3. std::abort() 是 C++ 标准库中的一个函数，定义在 <cstdlib> 头文件中。它用于立即终止程序的执行。

4. this->block_data = new u8[buf_sz]; 是因为要以字节的大小去操作数据，一个 u8 就是一个字节。

5. non-volatile memory 是非易失性内存。

6. std::monostate 是 C++17 引入的一种类型，通常用于表示一个空状态或无效状态。

7. std::variant 是 C++17 引入的一种类型，可以存储几种不同类型中的一种。它类似于联合体（union），但提供了更安全的类型管理和更方便的用法。

8. \_\_FILE\_\_ 和 \_\_LINE\_\_ 是 C/C++ 语言中的预定义宏，分别用于获取当前源文件名和当前源文件行号。

9. ftruncate(fd, total_file_sz)：这是一个系统调用，用于调整与给定文件描述符 fd 关联的文件的大小到 total_file_sz 字节。如果成功，返回 0；如果失败，返回 -1。

10. lab 里面默认的 block_sz 和 block_cnt 是 4096 和 4096，所以总大小是 16MB。

11. Block Manager 通过 mmap 映射直接操作文件，如下所示：

```cpp
    this->block_data =
        static_cast<u8 *>(mmap(nullptr, this->total_storage_sz(),
                               PROT_READ | PROT_WRITE, MAP_SHARED, this->fd, 0));
```

12. std::move 将 buffer 的所有权从原始对象转移到 iter.buffer，而不是复制内容。这意味着不需要重新分配内存或复制数据，能够显著提高性能，尤其是在处理大数据时。

13. std::vector<u8> buffer(bm->block_sz); 即构造一个大小和块大小相同的缓冲区

14. std::shared_ptr<BlockManager> bm; 是 C++ 中的一种智能指针，用于自动管理动态分配内存的生命周期，避免内存泄漏。

15. BlockManager 管理一组 block

16. data()：这是 std::vector 的一个成员函数，返回一个指向数组首元素的指针。具体来说，buffer.data() 返回一个 u8\* 类型的指针，指向 buffer 中存储的数组数据的起始位置。使用 data() 可以方便地与需要原始指针的其他 API 或者 C 风格的函数进行交互。

17. block_id 表示当前处理的块的 ID。block_idx 表示当前位图在其对应块中的具体位置或索引。

18. std::optional 是一个模板类，用于表示一个可能包含值的对象。它可以存储一个值，也可以不存储任何值。

19. std::nullopt 是一个常量，用于初始化或赋值给 std::optional 对象，以表示该对象不包含任何值。

20. find_first_free 和 res 的类型不匹配的问题？？

### Part 2: File Layer (Inode)

1. Note that the Inode Table stores the indirection mapping relationships of inode_id->block_id. The block_id is the block which actually stores the Inode (Do not forget one Inode matches one block exactly)

2. 在 lab 1 中，inode table 里面存储的是 inode id -> block id 的映射。当我们想要根据一个 inode id 去访问一个 inode，我们就需要根据这个 inode id 去查 inode table，得到存储这个 inode 的 block 的 block id，并去该 block 访问这个 inode。

3. set_table() 不需要设置 inode allocation bitmap？When an Inode is allocated or deallocated, the data in this area should be changed. 貌似这里的 set_table() 只是一个设置 inode table 的工具函数，不需要设置 inode allocation bitmap。

### Part 3: Filesystem Layer

1. [[maybe_unused]]：这是一个属性标志，用于告诉编译器该变量可能未被使用。这样做的目的是避免在编译时产生警告信息，尤其是在某些情况下你可能会暂时不使用这个变量，但仍然希望保留它的定义。这种用法有助于在开发过程中保持代码的清晰性，尤其是在需要保留某些结构但尚未完全实现功能时。

2. inode 在哪层写入 block 啊？

3. std::uniform_int_distribution<int> uni(0, 100); 是 C++ 标准库中的一行代码，主要用于创建一个均匀分布的随机整数生成器。

4. byd 这个 setTable 恶心人。内层单元测试的时候从 0 开始、所以我就没用 LOGIC_TO_RAW，但是其实外层调用的时候、需要传进去 RAW 的 block_id。

5. 这一行调用了 reserve 方法，以确保 indirect_block 向量在内部分配足够的内存来存储至少 block_size 个元素。reserve 不会改变向量的大小，只是增加其容量，以减少未来插入元素时可能发生的内存重新分配次数，从而提高性能。

   ```cpp
   std::vector<u8> indirect_block(0);
   indirect_block.reserve(block_size);
   ```

6. 一个 inode 里面存了一个 inode 的元数据，还有 nblock 个 block 的 id。

7. 这个 lab 中的 inode 实现、貌似只有一个 inode 的最后一个 block 里面可以存 indirect inode block id，其他的 block 里面存的都是 direct block id。

8. 那个 bool flag 设置需要每个分支都检查。不然会报错
