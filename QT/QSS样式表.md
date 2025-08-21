# 一、QSS介绍

> QSS，全称为Qt StyleSheet，是Qt框架中用于定义控件外观的一种样式表语言。

QSS可以应用于各种Qt控件（Widgets）上，包括按钮(Button)、标签(Label)、文本框(LineEdit)等，用来改变它们各项外观属性：

- **颜色与背景**：设置部件的前景色、背景色。
- **字体**：调整字体大小、类型、加粗、斜体等。
- **边框**：定义边框的宽度、样式、颜色。
- **渐变与图案**：应用渐变色或背景图案。
- **状态伪类**：针对不同状态（如：`:hover`, `:pressed`, `:disabled`）设置不同的样式。
- **布局与间距**：调整内边距、外边距等。

# 二、QSS的基本使用

## 1、样式表语法

一个基本的QSS规则由选择器和声明块组成：

```css
selector 
{ 
	attribute: value 
}
```

- **selector：选择器**：用来选择部件，指定哪些部件将受到规则的影响。
- **attribute：声明块**：包含一系列属性和值，定义了部件的外观。

例如，要将所有QPushButton的背景色设置为蓝色，文字颜色设置为白色，可以编写如下QSS规则：

```css
QPushButton
{
	background-color:blue;
	color:white;
}
```

## 2、QSS的使用方式

### 直接在代码中设置

直接在Qt的代码中使用`setStyleSheet()`函数为单个控件或整个应用程序设置样式。例如，为一个QPushButton设置背景颜色可以直接在构造函数或其他适当位置写入样式：

```css
// 设置按钮的样式表，将背景颜色设置为蓝色，文字颜色设置为白色
button->setStyleSheet("background-color: blue; color: white;");
```

### 使用外部QSS文件

将样式规则写入一个或多个外部`.qss`文件中，然后在程序运行时加载并应用这些样式。这种方式便于样式管理，特别是当样式复杂或者需要跨多个窗体共享样式时更为有用。

# 三、QSS文件的使用

## 1、手动创建Qss文件

在项目目录下，创建一个qss文件夹，在qss目录下创建一个qss文件。

## 2、添加qss文件

首先在项目中创建资源文件。

项目右键，点击添加新文件，弹出窗口中，选择`Qt Resource File`添加资源文件`src`

在项目中会出现`src.qrc`文件，点击文件右键，选中添加现有文件， 将手动创建的qss文件添加到项目中。

## 3、编辑QSS文件

```cpp
QPushButton
{
    background-color: blue;
    color: white;
}
```

## 4、加载QSS文件

头文件`QFile`中有相关函数用于读取QSS内容，在需要应用样式的 窗口 或 部件 的初始化代码中，使用以下代码读取：

```cpp
#include <QFile>
#include <QApplication>
#include <QDebug>
#include <QWidget>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);
	
    QFile file("qss文件路径");// 创建一个QFile对象，用于操作指定路径的.qss文件
    
    if (file.open(QFile::ReadOnly | QFile::Text))// 以只读和文本模式打开文件
    {
        QByteArray qss = file.readAll();//读取文件全部内容到QByteArray型变量qss中
        
        qApp->setStyleSheet(qss);//将qss数据应用到整个应用程序，qApp是指向整个应用程序实例的全局指针，通过它可以设置全局的样式表
        
        file.close();// 操作完成后关闭文件，释放相关资源
    } 
    else
    {
        qDebug() << "打开失败.";// 如果文件打开失败，则在控制台输出错误提示信息
    }
    
	/*
	这里可以继续创建窗口、添加部件等构建应用程序界面相关操作，此处省略相关代码示例
	*/
    
    return app.exec();
}
```

可以封装一个用来加载QSS文件数据的类，一行代码即可完成读取和设置样式：

```cpp
//qssfileload.h

#ifndef QSSFILELOAD_H
#define QSSFILELOAD_H
#include <QFile>
#include <QApplication>
#include <QWidget>

class QssFileLoad
{
public:
    QssFileLoad();

    //声明一个用来给QWidget加载QSS文件的函数，使用static是因为QSS要在整个程序生命周期内都使用的
    static void loadQssForQWidget(QString PATH, QWidget* target);

    //声明一个用来给QApplication加载QSS文件的函数，使用static是因为QSS要在整个程序生命周期内都使用的
    static void loadQssForQApplication(QString PATH, QApplication *target);
};

#endif
```

```cpp
//qssfileload.cpp

#include "qssfileload.h"
#include <QFile>
#include <QWidget>
#include <QApplication>

QssFileLoad::QssFileLoad()
{

}

void QssFileLoad::loadQssForQWidget(QString PATH, QWidget *target)
{
    QFile file(PATH);//打开文件的路径
    if(file.open(QFile::ReadOnly))//打开方式只读
    {
        QByteArray qss = file.readAll();//读取文件中所有数据并放在qss对象中
        target->setStyleSheet(qss);//给目标控件绑定qss
        file.close();
    }
    else
    {
        qDebug()<<"文件加载失败！";
    }
}

void QssFileLoad::loadQssForQApplication(QString PATH, QApplication *target)
{
    QFile file(PATH);//打开文件的路径
    if(file.open(QFile::ReadOnly))//打开方式只读
    {
        QByteArray qss = file.readAll();//读取文件中所有数据并放在qss对象中
        target->setStyleSheet(qss);//给目标应用绑定qss
        file.close();
    }
    else
    {
        qDebug()<<"文件加载失败！";
    }
}
```

# 四、常用样式

## 1、字体样式

| 样式属性            | 说明                          |
| --------------- | --------------------------- |
| **font-family** | 字体类型                        |
| **font-size**   | 字体大小，单位一般使用 px 像素           |
| **font-weight** | 字体加粗，bold 为加粗， normal 为不加粗  |
| **font-style**  | 字体样式，italic 为斜体， normal普通样式 |
| **font**        | 简写方式，可以同时设置前4种样式            |

```css
font-family: "Microsoft YaHei";
font-size: 14px;
fomt-style: italic;
font-weight: bold;
color: #ccc;
```

简写方式：

```css
font: bold italic 18px "Microsoft YaHei";
```

同时设置字体 style weight size family 的样式时，style(是否斜体) 和 weight (是否加粗)必须出现在开头，size 和 family 在后面，而且 size 必须在 family 之前，否则样式将不生效。

## 2、背景样式

| 样式属性                | 说明                                                   |
| ------------------- | ---------------------------------------------------- |
| background-color    | 背景颜色，可以使用十六进制数表示颜色，或字体颜色：red, green, blue 等          |
| background-image    | 背景图片，图片路径为 url(image-path)                           |
| background-repeat   | 设置背景平铺方式，no-repeat 不重复，repeat-x 在x轴重复，repeat-y 在y轴重复 |
| background-position | 背景定位，只支持 left right top bottom center                |
| background          | 设置背景的所有属性                                            |

```css
background-color: rgb(54,54,54);
background-image: url(:/imgs/picture/0.png);
background-repeat: no-repeat;"
background-position: left center;
```

简写方式：

```css
background: url(":/imgs/picture/0.png") no-repeat left center #363636;
```

background 为设置背景的所有属性，color image repeat position 这些属性值出现的顺序可以任意。

## 3、文本样式

| 样式属性            | 说明                                        |
| --------------- | ----------------------------------------- |
| text-decoration | 文本修饰：none、underline、overline、line-through |
| text-align      | 文本水平方向对齐方式left、right、center、justify       |
| color           | 文本颜色                                      |

```css
text-decoration: underline;
text-align: center;
color: #fff;
```

## 4、边框样式

| 样式属性          | 说明                              |
| ------------- | ------------------------------- |
| border-width  | 边框粗细                            |
| border-color  | 边框颜色                            |
| border-style  | 边框风格                            |
| border-radius | 设置边框圆角                          |
| border        | 设置所有边框属性 例：border:1px solid red |

```css
border-style: solid;
border-width: 2px;
border-color: red;
border-radius: 5px;
```

简写方式：

```css
border: 2px solid red;
```

1. **solid 为实线， dashed 为虚线， dotted 为点线， none 为不显示（如果不设置 border-style 的话，默认带边框）**

2. 边框也支持单独在各个方向上面的属性设置：

```css
border-top-style: solid;
border-top-width: 2px;
border-top-color: red;
border-top: 2px solid red;

border-right-style: solid;
border-right-width: 3px;
border-right-color: green;
border-right: 3px solid green;

border-bottom-style: solid;
border-bottom-width: 2px;
border-bottom-color: blue;
border-bottom: 2px solid blue;

border-left-style: solid;
border-left-width: 3px;
border-left-color: aqua;
border-left: 3px solid aqua;

border-top-left-radius: 5px;
border-top-right-radius: 10px;
border-bottom-left-radius: 15px;
border-bottom-right-radius: 20px;
border-radius: 20px;
```

# 五、选择器

> QSS选择器，用于指定哪些UI元素将受到特定样式的控制。通过选择器可以精确地定位到应用程序中的控件，进而应用样式。

## 1、基础选择器

### （1）类型选择器

**直接指定控件类型。**

```css
QWidget
{
	background-color:blue;
}
```

### （2）ID选择器

**需要在程序代码中，需要为对应的控件设置ID：**

```css
button->setObjectName("myButton");
```

**使用 `#` 前缀，为特定控件的唯一ID进行样式控制：**

```css
#myButton
{
	font-weight: bold;
}
```

### （3）类选择器

**需要在代码中设置控件属性，指定类样式名称：**

```css
setProperty("class","highlighted");
```

**使用`.`前缀，为自定义的一类控件进行QSS样式控制：**

```css
.heighlighted
{
	color:red;
}
```

## 2、层次选择器

### （1）后代选择器

**使用空格分隔，选择某个控件中的所有后代元素（儿子、孙子....）：**

```css
#widget QPushButton /* 选择所有在#widget内的QPushButton */
{
	color:red;
}
```

### （2）子选择器

**使用`>`，只选择直接子控件（只能选中子类，孙子......不行）：**

```css
#widget>QPushButton/* 只选择#widget的直接QPushButton子元素 */
{
	color:red;
}
```

## 3、组合选择器

**使用`,`分隔，可以同时定义多个选择器，应用相同的样式：**

```css
QPushButton,QLineEdit
{
	border:1px solid gray;
}
```

## 4、动态属性选择器

**Qt中使用动态属性，通过`setProperty()`函数设置属性；QSS中属性选择器使用`[]`语法：**

```css
button->setProperty("myStyle", true);
```

**然后在QSS中：**

```css
QPushButton[myStyle="true"]
{
	background-color: yellow;
}
```

# 六、QSS伪类选择器

> QSS伪类用于定义元素在特定状态下的样式。这些状态可以是用户交互导致的（如鼠标悬停、按下、获得焦点等），也可以是元素自身的特定条件（如禁用状态）。

## 1、**`:hover`**

**当鼠标悬停在控件上时的状态。**

```css
QPushButton:hover
{
	background-color: #ddd;
}
```

## 2、**`:pressed`**

**控件被按下时的状态。**

```css
QPushButton:pressed {
    background-color: #ccc;
}
```

## 3、**`:focus`**

**控件获取焦点时的状态。**

```css
QLineEdit:focus {
    border-color: blue;
    color: red;
}
```

## 4、**`:enabled`** 和 `:disabled`

**分别表示元素启用和禁用的状态。**

```css
QPushButton:enabled {
    color: black;
}

QPushButton:disabled {
    color: gray;
}
```

## 5、**`:checked`**

**用于复选框、单选按钮等，当被选中时的状态。**

```css
QRadioButton:checked {
    background-color: lightgreen;
}
```

## 6、**`:selected`**

**在列表、表格中，当项被选中时的状态。**

```css
QListWidget::item:selected {
    background-color: #8080ff;
}
```

**伪类名前的冒号 `:` 是必不可少的，而且伪类的顺序在QSS中有时也会影响最终的样式效果，特别是在存在多个伪类定义相冲突的情况下。合理利用伪类能够极大地丰富界面的动态表现和交互体验。**
