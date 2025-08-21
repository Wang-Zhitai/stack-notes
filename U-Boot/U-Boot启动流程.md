# U-Boot 启动流程详解

## 一、连接脚本 `u-boot.lds` 详解

1. **入口地址**：
   - U-Boot 的入口地址为 `_start`。
   - `__image_copy_start`：`0x87800000`。
   - `.vectors`：`0x87800000`，存放中断向量表。
   - `arch/arm/cpu/armv7/start.o`：`start.c` 文件中的代码。
   - `__image_copy_end`：`0x8785dc6c`。
2. **重定位段**：
   - `__rel_dyn_start`：`0x8785dc6c`，重定位段开始。
   - `__rel_dyn_end`：`0x878668a4`，重定位段结束。
   - `__end`：`0x878668a4`。
3. **BSS 段**：
   - `__bss_start`：`0x8785dc6c`，BSS 段开始。
   - `__bss_end`：`0x878a8d74`，BSS 段结束。
4. **其他**：
   - `_image_binary_end`：`0x878668a4`。

------

## 二、U-Boot 启动流程

### 1. **reset 函数**

- **目的**：
  - 将处理器设置为 SVC 模式，并关闭 FIQ 和 IRQ。
  - 设置中断向量。
  - 初始化 CP15 协处理器。

### 2. **lowlevel_init 函数**

- **SP 指针设置**：

  - `CONFIG_SYS_INIT_SP_ADDR` 的计算：

    ```c
    #define CONFIG_SYS_INIT_SP_ADDR \
        (CONFIG_SYS_INIT_RAM_ADDR + CONFIG_SYS_INIT_SP_OFFSET)
    #define CONFIG_SYS_INIT_SP_OFFSET \
        (CONFIG_SYS_INIT_RAM_SIZE - GENERATED_GBL_DATA_SIZE)
    #define CONFIG_SYS_INIT_RAM_SIZE  IRAM_SIZE
    #define IRAM_SIZE  0x00020000
    #define CONFIG_SYS_INIT_RAM_ADDR  IRAM_BASE_ADDR
    #define IRAM_BASE_ADDR  0x00900000  // 6ULL 内部 OCRAM
    #define GENERATED_GBL_DATA_SIZE  256
    ```

  - 最终计算得到 `SP` 指针为 `0x0091ff00`。

- **设置 SP 指针和 R9 寄存器**。

### 3. **s_init 函数**

- 空函数，未实现。

### 4. **_main 函数**

- 负责 U-Boot 的主要初始化流程。

### 5. **board_init_f 函数**

- **initcall_run_list**：
  - 调用 `init_sequence_f` 数组中的一系列初始化函数。
- **环境变量初始化**：
  - `version_string[] = U_BOOT_VERSION_STRING`。
  - `U_BOOT_VERSION = U-Boot 2016.03`。
- **内存分配**：
  - `TOTAL_MALLOC_LEN = (CONFIG_SYS_MALLOC_LEN + CONFIG_ENV_SIZE)`。
  - `CONFIG_SYS_MALLOC_LEN = 16 * SZ_1M`。
  - `CONFIG_ENV_SIZE = SZ_8K`。

### 6. **relocate_code 函数**

- **重定位过程**：
  - `r0 = gd->relocaddr = 0x9FF47000`（重定位后的首地址）。
  - `r1 = 0x87800000`（源地址起始地址）。
  - `r4 = 0x9FF47000 - 0x87800000 = 0x18747000`（偏移量）。
  - `r2 = 0x8785dc6c`。
- **重定位问题**：
  - 重定位后，函数调用和全局变量引用会出问题。
  - U-Boot 通过 `.rel.dyn` 段和位置无关码解决此问题。

### 7. **relocate_vectors 函数**

- 设置 `VBAR` 寄存器为重定位后的中断向量表起始地址。

### 8. **board_init_r 函数**

- 类似于 `board_init_f`，执行 `init_sequence_r` 初始化序列。

### 9. **run_main_loop 函数**

- **流程**：
  - `run_main_loop` -> `main_loop` -> `bootdelay_process`。
  - `autoboot_command` -> `abortboot` -> `abortboot_normal`。
  - `cli_loop` -> `parse_file_outer` -> `parse_stream_outer` -> `parse_stream` -> `run_list` -> `run_list_real` -> `run_pipe_real` -> `cmd_process`。

### 10. **cli_loop 函数**

- 处理 U-Boot 命令行输入。

### 11. **cmd_process 函数**

- **命令处理**：
  - U-Boot 使用 `U_BOOT_CMD` 定义命令。
  - 所有命令存储在 `.u_boot_list` 段中。
  - `cmd_tbl_t` 的 `cmd` 成员指向具体的命令执行函数（如 `do_xxx`）。
- **流程**：
  - `cmd_process` -> `find_cmd` -> `cmd_call` -> `cmdtp->cmd`。

------

## 三、bootz 启动 Linux 内核过程

### 1. **images 全局变量**

- `bootm_headers_t images`：用于存储内核镜像、设备树等信息。

### 2. **do_bootz 函数**

- **流程**：
  - `do_bootz` -> `bootz_start` -> `do_bootm_states`（`BOOTM_STATE_START`）。
  - `images->ep = 0x80800000`（内核镜像地址）。
  - `bootz_setup`：检查 `zImage` 是否正确。
  - `bootm_find_images` -> `boot_get_fdt`：找到设备树并设置 `images->ft_addr` 和 `images->ft_len`。
  - `bootm_disable_interrupts`：关闭中断。
  - `images.os.os = IH_OS_LINUX`：设置启动目标为 Linux。
  - `do_bootm_states`（`BOOTM_STATE_OS_PREP`、`BOOTM_STATE_OS_FAKE_GO`、`BOOTM_STATE_OS_GO`）。
    - `bootm_os_get_boot_func`：查找 Linux 内核启动函数 `do_bootm_linux`。
    - `boot_fn(BOOTM_STATE_OS_PREP, argc, argv, images)`：执行 `do_bootm_linux`。
    - `boot_prep_linux`：启动前的准备工作（如传递 `bootargs`）。
    - `boot_selected_os` -> `do_bootm_linux`（`BOOTM_STATE_OS_GO`）。
      - `boot_jump_linux`：启动 Linux 内核。
        - `machid = gd->bd->bi_arch_number`。
        - `kernel_entry = (void (*)(int, int, uint))images->ep`（`0x80800000`）。
        - `announce_and_cleanup`：输出 `Starting kernel...`。
        - `kernel_entry(0, machid, r2)`：跳转到 Linux 内核入口。

### 3. **do_bootm_states 函数**

- 根据不同的 BOOT 状态执行相应的操作。

### 4. **bootm_os_get_boot_func 函数**

- 查找并返回 Linux 内核启动函数 `do_bootm_linux`。

### 5. **do_bootm_linux 函数**

- **流程**：
  - `boot_prep_linux`：准备启动参数。
  - `boot_jump_linux`：跳转到 Linux 内核入口。

------

## 四、注意事项

1. **命令名称冲突**
   - 确保自定义命令的名称不与 U-Boot 内置命令冲突。
   - 如果冲突，U-Boot 会优先执行内置命令。
2. **命令参数限制**
   - 命令的最大参数个数由 `U_BOOT_CMD` 宏的 `maxargs` 参数决定。
   - 如果传入的参数超过 `maxargs`，U-Boot 会忽略多余的参数。
3. **命令可重复性**
   - 如果 `U_BOOT_CMD` 宏的 `repeatable` 参数为 1，则该命令可以重复执行（按回车键即可）。
   - 如果为 0，则每次执行后需要重新输入命令。
4. **命令返回值**
   - 命令处理函数应返回 0 表示成功，非 0 表示失败。
   - 返回值会影响 U-Boot 的行为（例如脚本执行）。

------

## 五、总结

在 U-Boot 命令行中，可以直接执行自定义命令，前提是该命令已经通过代码实现并编译到 U-Boot 中。通过 `U_BOOT_CMD` 宏，可以轻松地定义和注册自定义命令。在命令行中执行自定义命令时，需注意命令名称、参数和逻辑的正确性。

如果自定义命令无法正常工作，可以通过调试信息、检查注册情况和重新编译等方法排查问题。