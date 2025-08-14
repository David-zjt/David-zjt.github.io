## 附1：Linux内核简介

### 附1.1：主要功能代码

#### 附1.1.1：进程管理

负责进程生命周期控制、调度及通信。 
位置：`kernel/`（核心逻辑） + `include/linux/sched.h`（关键数据结构） 
关键事实： 

- 调度器：完全公平调度器（CFS）动态分配CPU时间（`kernel/sched/`目录） 
- 进程通信：信号、管道等机制代码位于`ipc/` 
- 实时性：`CONFIG_PREEMPT_RT`补丁支持微秒级响应（工业控制场景关键） 
  争议点：实时性优化（如PREEMPT_RT）牺牲吞吐量 vs. 通用场景性能平衡 

#### 附1.1.2：内存管理

虚拟/物理内存分配、回收及地址空间管理。 
位置：`mm/`（通用逻辑） + `arch/*/mm/`（架构相关优化） 
核心机制： 

- 伙伴系统：物理页分配（`mm/page_alloc.c`） 
- Slab分配器：内核对象缓存（`mm/slab.c`） 
- Swap机制：`mm/swap_state.c`（磁盘换页） 
  最新趋势： 
- 5.x内核引入`MGLRU`（多级LRU）提升内存回收效率 
- 异步内存压缩（`zswap`）降低延迟 

#### 附1.1.3：系统调用

定义：用户态程序访问内核功能的唯一安全接口。 
位置：`kernel/sys.c`（通用调用） + `arch/*/kernel/syscall_table`（架构相关表） 
关键流程： 

3. 用户触发`int 0x80`/`syscall`指令 
4. 通过`sys_call_table`跳转目标函数（定义于`include/uapi/asm-generic/unistd.h`） 
5. 结果经`pt_regs`结构返回用户态 
   性能热点：`vdso`（虚拟动态共享库）机制避免模式切换开销 

### 附1-2：推荐资源

1. 内核源码导航：[Elixir Cross Reference](https://elixir.bootlin.com/)（带版本追溯的源码浏览器） 
2. 权威文档：[Linux Kernel Documentation](https://www.kernel.org/doc/html/latest/)（`Documentation/`目录官方在线版） 
3. 实战书籍：《Linux Kernel Development, 3rd Ed》（Robert Love著，详解进程/内存模型） 
4. 深度调试：`ftrace`工具（`Documentation/trace/ftrace.rst`）实时跟踪系统调用链 

### 附1-3：内核编译形态

#### 附1-3-1：内核编译的核心输出文件

1. `vmlinux`：原始ELF可执行文件
- 格式：标准的ELF（Executable and Linkable Format）格式
  - 包含完整的符号表和调试信息，体积较大（通常数十MB）
  - 可直接被调试工具（如GDB）分析，但不能直接引导启动（未压缩且无引导头）。 
  - 生成路径：位于内核源码根目录。
2. 压缩后的可引导镜像
- vmlinuz`：通用名称，实际分为两类： 
- `zImage`： 
  - 适用场景：内核体积较小时（≤640KB）。 
  - 生成方式：对`vmlinux`进行`gzip`压缩，并嵌入解压代码。 
- `bzImage`（Big zImage）： 
  - 适用场景：内核体积较大时（>640KB）。 
  - 特点：解压代码与内核分离，支持更大尺寸内核。 
  - 本质：仍是ELF格式，但通过压缩减小体积，并添加自解压引导程序。
3. 嵌入式场景专用格式
- `uImage`：
  - 生成方式：在`zImage`或`bzImage`前添加 64字节的U-Boot头。 
  - 用途：供U-Boot引导加载器识别和加载（常见于ARM嵌入式设备）。 

#### 附1-3-2：其他辅助文件与格式

1. `Image`： 
- 通过`objcopy`处理`vmlinux`生成的纯二进制文件，不含ELF元信息[[2]()]。 
- 用途：部分引导程序（如QEMU）直接加载运行。 
2. `xipImage`（eXecute In Place）： 
- 特点：无需复制到内存，直接在Nor Flash中执行

#### 附1-3-3：关键区别总结

| 文件         | 格式            | 是否可引导 | 主要用途            |
| ---------- | ------------- | ----- | --------------- |
| `vmlinux`  | ELF（未压缩）      | 否     | 调试、符号分析         |
| `zImage`   | ELF（压缩）       | 是     | 小内核设备的引导（如旧x86） |
| `bzImage`  | ELF（压缩）       | 是     | 现代PC/服务器的标准引导镜像 |
| `uImage`   | ELF + U-Boot头 | 是     | U-Boot引导的嵌入式设备  |
| `xipImage` | 二进制           | 是     | Nor Flash嵌入式设备  |

#### 附1-3-4：加载与执行流程

1. 引导阶段： 
- BIOS/UEFI或引导程序（如GRUB）加载`bzImage`或`uImage`到内存。 
2. 解压与跳转： 
- 压缩镜像中的自解压代码将内核解压到指定内存地址。 
3. 内核启动： 
- CPU跳转到解压后的内核入口点（定义在ELF头中），执行`start_kernel()。 
