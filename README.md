# 2020-Fall-DBMS-Project
This the project of DBMS course in 2020 Fall.

## 文档目的
读了介绍文档，你应该知道**需要做什么**和**不能做什么**。

## 注意事项
1. 各小组请自行建立属于小组的Github仓库，所有作业小组成员通过Github或其他代码托管网站协作完成（请自学Git相关操作）。
2. 小组个人评分时根据Github的贡献量，请小组自行平均分工。
3. Github地址对其他小组保密，避免被抄袭。
4. **各小组切忌参考别人的代码，TA将使用软件查重，一抓一个准，高度雷同的小组都按抄袭处理，课程也会大概率挂科**。

## Persistent Memory Linear Hash
这次课程设计的主题是实现一个简单的基于NVM的线性哈希索引，涉及到的知识点有数据库系统的线性哈希，非易失性内存NVM环境的仿真模拟，PMDK库的安装和使用，NVM编程要点。项目已经写好头文件，包括数据结构的基本设计和相应的函数接口，目标是实现这些接口来满足相应的需求

### 线性哈希
索引是数据库系统中重要的数据结构，数据库的数据存储依靠索引来进行组织，可以理解为一种存放协议。有了存放的规则，我们就可以依据存放的规则进行相应的查找，怎么放就怎么找。有了查找功能，我们也可以相应实现修改和删除功能。所以索引具有四大基本操作：**增删改查**，而操作的数据对象都是**键值对**的形式。

常见的索引有B+Tree和哈希索引，线性哈希属于一种哈希索引。线性哈希的核心是会将所有哈希表维护成一种连续的数组的模式，这样我们就可以方便地直接通过下标来访问相应的哈希桶。本次课程设计的线性哈希实现参考课本《数据库管理系统原理与设计》第三版的283页部分对线性哈希的描述来实现其增删改查操作。

### 非易失性内存NVM
Non-Volatile Memory(NVM)也即非易失性内存，是一种新型存储硬件。其不仅具有传统内存的字节寻址特性，同时也具有磁盘的数据持久化的特性，将数据存放在NVM上面可以实现持久化。这次课程设计中的线性哈希的键值数据等就是要存放到NVM上，以达到持久化和可恢复的目的。要实现持久化，需要进行相应的数据存储的设计，即解决如何存放持久的数据然后如何读取这些数据的问题，便于程序依靠这些数据恢复。

### NVM环境模拟
NVM设备目前还不常见，但是Intel已经提供了一份[教程](https://software.intel.com/content/www/us/en/develop/articles/how-to-emulate-persistent-memory-on-an-intel-architecture-server.html)来讲解如何利用自己机器上的普通内存来模拟NVM环境。在Ubuntu 18等具有较新Linux内核的操作系统中进行模拟的话，直接从GRUB Configuration步骤开始即可。

### NVM编程要点
由于NVM属于一种特殊的存储硬件，所以编程上需要按照特定的模式进行以对NVM的持久数据按字节访问而不是IO流的形式。现在主流的NVM编程模型是依据SNIA的推荐的标准，使用mmap内存映射的方式进行NVM文件数据的访问，mmap相关背景请自行百度了解。具体来说，就是在NVM的设备上安装一个NVM-aware的文件系统如EXT4,然后开启DAX模式以供程序能够以直接访问模式访问文件数据，然后程序以mmap的形式打开NVM文件，以字节寻址的方式操作文件数据。而在实际编程中，可以借助开源的PMDK库依据这个编程模型进行NVM编程。

为了实现NVM编程，对于数据操作细节，需要注意几点：
1. 先通过mmap的形式打开文件通过其虚拟地址来操作相应的数据，然后通过unmap的形式关闭文件
2. 每次修改NVM上的数据，都是会先在CPU Cache中进行。为了真正持久化到NVM中，都要调用CLWB/CLFLUSH指令来将数据扫回NVM，然后调用一次sfence来保证先后写入的有序性

上述操作和NVM编程可以借助PMDK来实现。

### PMDK库
PMDK库是开源的NVM编程的工具库，详情请查看官方[github仓库](https://github.com/pmem/pmdk)和[主页](https://pmem.io)。本次课程设计使用的库是最基本的libpmem库，主要用到的函数只有三个，用于协助我们进行NVM编程和解决NVM编程要点的数据操作问题：
1. pmem_map(): 这个函数将目标文件通过内存映射的方式打开并返回文件数据的虚拟地址，这样操作这个虚拟地址的数据即通过字节寻址的方式操作数据
2. pmem_persist(): 这个函数调用CLFLUSH和SFENCE指令来显式持久化相应的数据，每次NVM数据的修改都应调用一次
3. pmem_unmap(): 这个函数用于关闭pmem_map打开的NVM文件

## 具体任务
本次课程设计分几步完成，涉及环境搭建和项目代码实现：
1. 实验环境应是Ubuntu 18等具有较新Linux内核的操作系统，推荐使用Ubuntu 18或20
2. 每个小组应建立自己的Git仓库进行协同编程，没有Git的提交记录当没有贡献处理
3. 根据[Intel的教程](https://software.intel.com/content/www/us/en/develop/articles/how-to-emulate-persistent-memory-on-an-intel-architecture-server.html)，利用普通内存模拟NVM环境并测试是否配置正确
4. 根据PMDK的README安装教程进行库安装
5. 根据项目框架和需求实现代码并运行，测试运行并截图相应结果
6. 撰写report总结任务3-5的实现过程，重点在第5步

## PML-Hash设计细节

### 数据结构设计
PMLHash的主要结构已经在头文件中给出，主要包括元数据，存储的哈希表数组数据以及溢出哈希表数据，默认将所有数据放在一个16MB的文件中存储，结构如下：
```
// PMLHash
| Metadata | Hash Table Array | Overflow Hash Tables |
+---------- 8 MB -------------+------- 8 MB ---------+

// Metadata
| size | level | next | overflow_num |

// Hash Table Array
| hash table[0] | hash table[1] | ... | hash table[n] | 

// Overflow Hash Tables
| hash table[0] | hash table[1] | ... | hash table[m] |

// Hash Table
| key | value | fill_num | next_offset |
```