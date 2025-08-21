# U-Boot 顶层 Makefile 分析

## 一、U-Boot 源码 VSCode 工程创建

- 使用 VSCode 打开 U-Boot 源码目录。
- 配置 `.vscode` 文件夹中的 `c_cpp_properties.json`，设置 `includePath` 和 `defines`，以便代码跳转和智能提示。

------

## 二、顶层 Makefile 分析

### 1. **变量定义**

- **AS**：汇编器

  ```Makefile
  AS = $(CROSS_COMPILE)as
  AS = arm-linux-gnueabihf-as
  ```

- **ARCH**：架构

  ```Makefile
  ARCH = arm
  ```

- **CPU**：处理器

  ```Makefile
  CPU = armv7
  ```

- **BOARD**：开发板

  ```Makefile
  BOARD = mx6ullevk
  ```

- **VENDOR**：厂商

  ```Makefile
  VENDOR = freescale
  ```

- **SOC**：芯片

  ```Makefile
  SOC = mx6
  ```

- **CPUDIR**：CPU 相关代码目录

  ```Makefile
  CPUDIR = arch/arm/cpu/armv7
  ```

- **BOARDDIR**：开发板相关代码目录

  ```Makefile
  BOARDDIR = freescale/mx6ullevk
  ```

------

## 三、编译处理过程

### 1. **清理编译环境**

```shell
make distclean
```

### 2. **配置编译选项**

```shell
make mx6ull_14x14_ddr512_emmc_defconfig
```

- **规则**：

  ```Makefile
  %config: scripts_basic outputmakefile FORCE
      $(Q)$(MAKE) $(build)=scripts/kconfig $@
  ```

- **目标 `scripts_basic`**：

  ```makefile
  scripts_basic:
      $(Q)$(MAKE) $(build)=scripts/basic
  ```

  - 展开后：

    ```shell
    make -f ./scripts/Makefile.build obj=scripts/basic
    ```

- **`build` 变量定义**：

  ```makefile
  build := -f $(srctree)/scripts/Makefile.build obj
  build := -f ./scripts/Makefile.build obj
  ```

- **`outputmakefile`**：

  - 由于 `KBUILD_SRC` 未定义，`outputmakefile` 无效。

- **最终执行的命令**：

  ```shell
  make -f ./scripts/Makefile.build obj=scripts/basic
  make -f ./scripts/Makefile.build obj=scripts/kconfig xxx_defconfig
  ```

- **第一条命令**：

  - 编译 `scripts/basic/fixdep`，对应的源文件是 `scripts/basic/fixdep.c`。

- **第二条命令**：

  - 执行 `scripts/kconfig/conf`，生成配置文件：

    ```shell
    scripts/kconfig/conf --defconfig=arch/../configs/xxx_defconfig Kconfig
    ```

### 3. **编译 U-Boot**

```shell
make V=1 -j12
```

- **默认目标**：

  ```Makefile
  PHONY := _all
  _all:
  _all: all
  all: $(ALL-y)
  ALL-y += u-boot.srec u-boot.bin u-boot.sym System.map u-boot.cfg binary_size_check
  ```

- **生成 `u-boot.bin`**：

  ```Makefile
  u-boot.bin: u-boot-nodtb.bin FORCE
      $(call if_changed,copy)
  ```

- **生成 `u-boot-nodtb.bin`**：

  ```Makefile
  u-boot-nodtb.bin: u-boot FORCE
      $(call if_changed,objcopy)
      $(call DO_STATIC_RELA,$<,$@,$(CONFIG_SYS_TEXT_BASE))
      $(BOARD_SIZE_CHECK)
  ```

- **生成 `u-boot`**：

  ```Makefile
  u-boot: $(u-boot-init) $(u-boot-main) u-boot.lds FORCE
      $(call if_changed,u-boot__)
  ```

- **`u-boot-init` 和 `u-boot-main`**：

  ```Makefile
  u-boot-init := $(head-y)
  u-boot-main := $(libs-y)
  head-y = arch/arm/cpu/armv7/start.o
  libs-y = lib/built-in.o
  ```

  - `libs-y` 保存了大量的 `built-in.o` 文件。

- **`u-boot` 的生成**：

  - 将 `start.o` 和大量的 `built-in.o` 链接在一起。

- **条件编译**：

  - 如果定义了 `CONFIG_ONENAND_U_BOOT`，则生成 `u-boot-onenand.bin`：

    ```Makefile
    ALL-$(CONFIG_ONENAND_U_BOOT) += u-boot-onenand.bin
    ```

------

## 四、链接

### 1. **链接脚本**

- 链接脚本为 `u-boot.lds`。
- U-Boot 的链接首地址为 `0x87800000`。

### 2. **链接地址设置**

- 在 `mx6_common.h` 文件中设置：

  ```c
  CONFIG_SYS_TEXT_BASE = 0x87800000
  ```

------

## 五、总结

- U-Boot 的编译过程包括清理环境、配置选项、编译和链接等步骤。
- 顶层 Makefile 通过变量和规则控制整个编译流程。
- 链接脚本 `u-boot.lds` 和链接地址 `CONFIG_SYS_TEXT_BASE` 决定了 U-Boot 的内存布局。