# 一、VSCode+MDK 开发环境简介

> 1. 用MDK编译调试，功能强大
> 2. VScode 编辑+下载，编辑舒适

# 二、插件安装

![image-20250806234202434](./assets/image-20250806234202434.png)

**如图：**

1. C / C++ ：用于提供C语言代码提示和补全的工具，也可以用更好的 clangd 代替
2. Catppuccin for VSCode（可选） ：VSCode 皮肤
3. Chinese（可选）：简体中文包
4. Doxygen Document Genertor（可选）：代码/文件注释自动生成
5. Keil uVisin Assistant（必选）：可以在VScode里打开Keil工程，实现编译和下载

# 三、注意事项

1. 配置和调试工程需要到Keil里，VSCode只能编译和下载

2. 打开Keil所在的文件夹，会自动识别出来，如下图，然后就能编译下载了，注意不要选择打开Keil工程，直接用VSCode打开所在的文件夹就好了，如果是CUBEMX生成的文件夹，你就选 CUBEMX配置文件（.ioc）所在的文件夹直接用VSCode打开

   ![image-20250806235235688](./assets/image-20250806235235688.png)