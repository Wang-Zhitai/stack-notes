# Framebuffer 和 DRM 的区别

**在学习 `Framebuffer` 和 `DRM` 框架之前需要先学习 `Linux` 的 `Linux graphic subsystem`（图形子系统）**：

[Linux graphic subsytem(1)_概述](http://www.wowotech.net/graphic_subsystem/graphic_subsystem_overview.html)

[Linux graphic subsystem(2)_DRI介绍](http://www.wowotech.net/graphic_subsystem/dri_overview.html)



Framebuffer 和 DRM 都是 Linux Kernel 中的显示子系统，他们有不同的作用和定位。

Framebuffer是一种基础的图形子系统，他为用户空间提供了一种在显示器上面绘制像素的方式，通过一个简单的缓冲区来实现帧的绘制和显示

DRM是一个高级的图形子系统，它提供了许多高级功能，如硬件加速、3D图形渲染、视频解码等。支持多个用户空间客户端同时访问图形硬件。DRM还提供了复杂的内存管理和DMA机制，可以更好的管理系统中的显存。DRM能适应日益更新的显示硬件，DRM原生支持多层合成，支持VSYNC，支持DMA-BUF，支持一部更新，支持fence机制等



**`DRM` 原生支持多图层合成，而 `Framebuffer` 原生不支持，`DRM` 统一管理 `GPU` 和 `DISPLAY` 驱动，使得软件架构更为统一，方便管理和维护**：

| 特性         | DRM  | Framebuffer |
| ------------ | ---- | ----------- |
| 多图层合成   | 支持 | N/A         |
| VSYNC        | 支持 | N/A         |
| DMA-BUF      | 支持 | N/A         |
| 异步更新     | 支持 | N/A         |
| Fence机制    | 支持 | N/A         |
| 统一管理驱动 | 支持 | N/A         |

```mermaid
graph TB
    subgraph "用户空间"
        App[应用程序<br/>直接读写 Framebuffer]
    end

    subgraph "内核空间"
        SysCall[系统调用接口<br/>read/write/ioctl/mmap]
        
        subgraph "Framebuffer 驱动层"
            FB_Core[fbdev 核心框架<br/>统一设备管理 /dev/fbX]
            HW_Driver[具体硬件驱动<br/>设置显示模式/内存映射]
        end
        
        HAL[硬件抽象层]
        HW[显示硬件控制器<br/>CRTC/Timing Controller]
    end

    App --> SysCall
    SysCall --> FB_Core
    FB_Core --> HW_Driver
    HW_Driver --> HAL
    HAL --> HW
    
    style App fill:#e1f5fe
    style FB_Core fill:#f3e5f5
    style HW_Driver fill:#e8f5e8
    style HW fill:#ffecb3
```



```mermaid
graph TB
    subgraph "用户空间图形栈"
        direction LR
        OpenGL[Mesa OpenGL]
        Vulkan[Vulkan 驱动]
        X11[X11 驱动]
        Wayland[Wayland 合成器]
        libdrm[libdrm 库]
        
        OpenGL --> libdrm
        Vulkan --> libdrm
        X11 --> libdrm
        Wayland --> libdrm
    end
    
    subgraph "DRM 内核子系统"
        subgraph "GEM/TTM 内存管理器"
            GEM[显存分配与管理]
            DMA[DMA-BUF 共享]
            Swap[内存压缩/交换]
        end
        
        subgraph "调度器"
            Scheduler[作业调度 drm_sched]
            Queue[GPU 命令队列管理]
            Fence[Fence 同步机制]
        end
        
        subgraph "KMS 内核模式设置"
            CRTC[CRTC/Encoder/Connector 管理]
            Mode[显示模式设置]
            Plane[平面合成 Plane]
            Atomic[原子提交 Atomic Commit]
        end
        
        subgraph "具体硬件驱动"
            GPU[GPU驱动<br/>AMD/Intel/NVIDIA]
            Display[DisplayPort/HDMI 驱动]
        end
    end
    
    subgraph "硬件层"
        GPU_HW[GPU<br/>渲染/计算]
        Display_HW[显示控制器<br/>CRTC/TCON]
    end
    
    libdrm --> GEM
    libdrm --> Scheduler
    libdrm --> CRTC
    
    GEM --> GPU
    Scheduler --> GPU
    CRTC --> Display
    
    GPU --> GPU_HW
    Display --> Display_HW
    GPU_HW --> Display_HW
    
    style libdrm fill:#bbdefb
    style GEM fill:#c8e6c9
    style Scheduler fill:#fff3cd
    style CRTC fill:#f8bbd0
    style GPU fill:#d1c4e9
    style GPU_HW fill:#ffccbc
    style Display_HW fill:#ffecb3
```

