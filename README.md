
# 6.828 Lab2 Memory Management
## Part 1: Physical Page Management
首先分配物理页，它利用一个PageInfo结构体指针来存储内存中空闲的空间。先来看看是怎么一步一步把page初始化的。
第一步：通过i386_detect_memory（）探测内存总共有多少空间，然后规定一页4KB，计算出总共有多少页。  
第二步：通过boot_alloc()函数分配一块内存给页目录，注意：该目录在内存的位置在内核代码上面。  
    boot_alloc()函数功能是从内核代码上面开始分配内存，返回分配空间的起始地址作为结果。其中，它的实现有一个巧妙的地方，用了一个静态局部变量nextfree，这样在初始化的时候会默认为0，并且每次初始化后会更新。
第三步：继续通过boot_alloc()函数分配空间给PageInfo结构体，具体空间为npages*sizeof(PageInfo),其中npages代表总共物理页的数量。  
此时的内存空间应该是这样：  
&emsp;&emsp;       +-----------------------------------------------------------  
&emsp;&emsp;      |&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;| &emsp; kernel  &emsp;  | &emsp;pgdir&emsp;|&emsp;&emsp;&emsp;PageInfo &emsp;&emsp;&emsp;              |  
&emsp;&emsp;      | &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|  &emsp; code  &emsp;   | &emsp;  4K&emsp; |&emsp;npages*sizeof(PageInfo)  |  
&emsp;&emsp;     +--------------|------------|--------|-------------------------|        
虚拟地址         0xf0100000      end  
PageInfo通过一个page[]的数组，数组元素为PageInfo结构,每一个结构占4B。  
物理内存可以用page[]数组去映射,具体结构如下：  
&emsp;&emsp;  page[]  +---------+&emsp;&emsp;&emsp;physical memory+-----------+  
&emsp;&emsp;&emsp;&emsp;          +  &emsp;  6B &emsp;  +--------------------------->|  &emsp;  4KB  &emsp;  |  
&emsp;&emsp; &emsp;&emsp;       + --------+ &emsp;&emsp;&emsp;&emsp;&emsp; &emsp;&emsp; &emsp;&emsp; &emsp; &emsp; +-----------+  
&emsp;&emsp; &emsp;&emsp;         + &emsp;   6B &emsp;  +--------------------------->|  &emsp;  4KB  &emsp;  |  
&emsp;&emsp; &emsp;&emsp;         +---------+   &emsp;&emsp;&emsp; &emsp;&emsp; &emsp;&emsp;&emsp;&emsp; &emsp; &emsp; +-----------+  
第四步：标记内存使用情况，通过PageInfo的映射，把已经使用的页的PageInfo的pp_ref设为1，代表已经被一个引用一次。然后用头插法把空闲的PageInfo里的pp_link插入，空闲链表为page_free_list具体实现代码pmap.c里面的page_init()实现。  
至此，page的初始化基本完成，之后进行的就是对page的分配和使用。  
然后page_walk函数很关键，之前一直没搞清楚二级页表为什么用page_alloc这个分配物理页的函数去分配，其实它是分配了一个4KB物理页作为二级页表，通过page2pa转换成物理地址存储在页目录项里面。









