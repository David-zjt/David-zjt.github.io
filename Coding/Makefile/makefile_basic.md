## 附1：Makefile 基础

### 附1-1：核心结构

Makefile 由 规则构成，每条规则定义目标文件的生成逻辑： 

```makefile
目标文件: 依赖文件 
[TAB]命令1 
[TAB]命令2 
```

关键元素解析： 

1. 目标
- 要生成的文件名（如 `main.o`）或伪目标（如 `clean`） 
- 第一个目标是默认目标（运行 `make` 时自动执行）
2. 依赖 
- 目标所需的源文件或其他目标（如 `main.c` 或 `utils.o`） 
- 依赖更新时，目标会重新编译
3. 命令 
- 以 Tab 开头 的 Shell 命令（如 `gcc -c main.c`） 
- 用于从依赖生成目标

### 附1-2：基础语法

#### 附1-2-1：变量定义与使用

```makefile
CC = gcc # 定义编译器 
CFLAGS = -Wall # 定义编译选项 
OBJS = main.o utils.o # 定义目标文件列表 
app: $(OBJS) 
$(CC) $(CFLAGS) -o app $(OBJS) # 使用变量 $(VAR)
```

- 赋值类型： 
- `=`：递归赋值（使用时解析） 
- `:=`：立即赋值（定义时解析）[[2]()][[5]()]  

#### 附1-2-2：自动变量（简化命令）

| 变量   | 含义       | 示例                |
| ---- | -------- | ----------------- |
| `$@` | 当前目标文件名  | `gcc -c $@ -o $@` |
| `$<` | 第一个依赖文件名 | `gcc -c $< -o $@` |
| `$^` | 所有依赖文件   | `gcc $^ -o $@`    |

```makefile
%.o: %.c 
$(CC) $(CFLAGS) -c $< -o $@ # 编译所有 .c 到 .o 
```

#### 附1-2-3：通配符与模式规则

- `%`：通用匹配符（如 `%.o` 匹配所有 `.o` 文件） 

- `*`：匹配任意字符（如 `*.c` 匹配所有 C 文件） 
  
  ```makefile
  SOURCES = $(wildcard *.c) # 获取所有 .c 文件 
  OBJS = $(patsubst %.c, %.o, $(SOURCES)) # 将 .c 替换为 .o 
  ```

#### 附1-2-4 伪目标

用于执行非文件生成操作（如清理）： 

```makefile
.PHONY: clean # 声明伪目标 
clean: 
rm -f *.o app # 删除编译产物
```

运行：`make clean`

#### 附1-2-5条件判断

```makefile
DEBUG = 1 
ifeq ($(DEBUG), 1) 
CFLAGS += -g # 调试模式加 -g 
else 
CFLAGS += -O2 # 发布模式优化 
endif 
```

### 附1-3：进阶技巧

#### 附1-3-1：函数应用：

- `$(subst from,to,text)`：替换字符串 
- `$(filter %.c, $(FILES))`：过滤 C 文件 

#### 附1-3-2：多目录管理：

- 子目录 Makefile 通过 `include` 集成

#### 附1-3-3：自动生成依赖：

- 用 `gcc -M` 自动追踪头文件依赖

## 附2：Makefile 关键字

### 附2-1：变量定义与操作类

| 关键字        | 功能           | 语法示例                                       | 核心特性                      |
| ---------- | ------------ | ------------------------------------------ | ------------------------- |
| `=`        | 递归赋值（延迟展开）   | `VAR = $(OTHER)`                           | 变量引用在使用时解析                |
| `:=`       | 立即赋值（定义时展开）  | `VAR := value`                             | 右侧表达式定义时立即计算              |
| `?=`       | 条件赋值（未定义时生效） | `VAR ?= default_value`                     | 仅当VAR未定义时赋值               |
| `+=`       | 追加赋值         | `CFLAGS += -O2`                            | 向已存在变量追加值                 |
| `override` | 强制覆盖命令行参数    | `override CFLAGS = -Wall`                  | 防止命令行输入覆盖变量定义[[5]()]      |
| `define`   | 定义多行变量（宏）    | `define FUNC`<br>`echo "Hello"`<br>`endef` | 需用`$(call FUNC)`调用[[4]()] |

### 附2-2：流程控制类

| 关键字            | 功能          | 语法示例                                                 |
| -------------- | ----------- | ---------------------------------------------------- |
| `ifeq/ifneq`   | 条件判断（相等/不等） | `ifeq ($(ARCH), x86)`<br>`CFLAGS += -m32`<br>`endif` |
| `ifdef/ifndef` | 判断变量是否定义    | `ifdef DEBUG`<br>`CFLAGS += -g`<br>`endif`           |
| `else`         | 条件分支        | 与`ifeq`/`ifdef`联用                                    |
| `endif`        | 结束条件块       | 标记条件语句结束                                             |

### 附2-3：文件包含与规则声明

| 关键字         | 功能           | 语法示例                | 关键机制                            |
| ----------- | ------------ | ------------------- | ------------------------------- |
| `include`   | 包含其他Makefile | `include config.mk` | 支持路径搜索，失败可忽略（`-include`）[[7]()] |
| `.PHONY`    | 声明伪目标（非文件目标） | `.PHONY: clean all` | 避免与同名文件冲突[[5]()]                |
| `.SUFFIXES` | 声明后缀规则支持     | `.SUFFIXES: .c .o`  | 定义隐式规则扩展名[[2]()]                |
| `vpath`     | 设置文件搜索路径     | `vpath %.c src`     | 按模式指定依赖文件搜索目录                   |

### 附2-4：函数与通配符

| 关键字/函数     | 功能       | 语法示例                                 |
| ---------- | -------- | ------------------------------------ |
| `wildcard` | 通配符展开文件名 | `SRCS = $(wildcard *.c)`             |
| `patsubst` | 模式替换     | `OBJS = $(patsubst %.c,%.o,$(SRCS))` |
| `subst`    | 字符串替换    | `STR = $(subst from,to,$(VAR))`      |
| `filter`   | 按模式过滤列表  | `C_FILES = $(filter %.c,$(FILES))`   |

### 附2-5：特殊目标（以`.`开头）

| 目标         | 功能         | 典型用法                                 |
| ---------- | ---------- | ------------------------------------ |
| `.DEFAULT` | 未定义目标的默认规则 | `.DEFAULT: @echo "Target not found"` |
| `.IGNORE`  | 忽略命令执行错误   | `.IGNORE: clean`                     |
| `.SILENT`  | 禁止回显命令     | `.SILENT:`                           |

### 附2-6：高级依赖控制

| 关键字         | 功能                | 语法示例                                       |
| ----------- | ----------------- | ------------------------------------------ |
| `export`    | 导出变量到子Makefile    | `export PATH`                              |
| `unexport`  | 取消导出变量            | `unexport CFLAGS`                          |
| `select`    | 反向依赖声明（Kconfig专用） | `config FEATURE_A`<br>`select LIB_X`       |
| `dependson` | 正向依赖声明            | `config FEATURE_B`<br>`dependson ARCH_X86` |

> 最佳实践提示： 
> 
> 1. 变量选择：优先使用`:=`避免递归展开性能损耗，动态生成路径时用`=` 
> 2. 错误处理：关键命令前加`-`（如`-rm file`）忽略可能的失败[[7]()] 
> 3. 模块化：大型项目用`include`拆分Makefile，结合`export`传递配置[[1]()][[7]()] 
>    附录：代码示例片段

```makefile
多目录构建示例
include build/config.mk
export PROJECT_ROOT := $(CURDIR)
.PHONY: all
all: app
app: $(OBJS)
 $(CC) -o $@ $^ $(LIBS)
条件编译调试版
ifeq ($(DEBUG),1)
 CFLAGS += -O0 -g
else
 CFLAGS += -O3
endif
自动生成依赖
%.o: %.c
 $(CC) $(CFLAGS) -MMD -c $< -o $@
-include $(OBJS:.o=.d)
```
