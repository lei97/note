#  Qml 整理

##  编码规范

###  定位

D:\Documents\我的文档\Tencent Files\1022958177\FileRecv



####  相对坐标

####  锚定位

####  Layout

##  Python下使用 Qml

###  简单 Demo

####  Python 文件

```python 
from PySide2.QtWidgets import QApplication
from PySide2.QtQuick import QQuickView
from PySide2.QtCore import QUrl
# from PySide2 import QQmlApplicationEngine 

class Ui (object):
    def __init__(self,qml_url):
        self.app = QApplication([])
        self.view = QQuickView()
        self.url = QUrl(qml_url)
        self.view.setSource(self.url)

    def run(self) :
        self.view.show()
        self.app.exec_()

if __name__ == '__main__':
    ui = Ui("Qml/main.qml")
    ui.run()
```

####  Qml 文件

```javascript
import QtQuick 2.6
import QtQuick.Controls 1.4
import QtQuick.Window 2.2

Rectangle {
    width: 800
    height: 800
    id: window1
    
    Rectangle {
        width: 50
        height: 50
        color: "red"
    }
    
    Text {
        text: "hello world"
        color: "black"
        font: 40
    }
}
```

####  .pylintrc 文件

```
extension-pkg-whitelist=PySide2
```

##  组件化编程

###  C++ 目录

###  导入其他目录下 Qml 文件

> 注意事项：除了 **main.qml** 文件之外的其他 **qml** 文件命名时首字母大写。

* 文件目录
  * **component** 存放功能模块。如：**button**。
  * **page** 存放各个图层的具体实现。
  * **test** 存放测试文件目录。
  * **main.qml** 初始调用文件。
  * **Img** 文件夹为图片 **UI** 存放位置。

![image-20191119163100398](img/image-20191119163100398.png)

* 代码调用

```javascript
import "./component/button" as QBTN  

// 使用 QBIN 
QBTN.TextBtn {
    id: button_1
    x: 200
    y: 200 
    height: 200
    width: 200
    text: "is a button text"
    textColor: "#171717"
}
```

## [Qt Quick 性能优化](https://doc.qt.io/qt-5/qtquick-performance.html) （ 60 fps )

###  调试手段

TODO :  等待开始。

TODO：使用Qt Profiling 进行性能分析。

URL：  http://doc.qt.io/qtcreator/creator-qml-performance-monitor.html   

### 事件驱动

* 避免定时轮询
* 使用信号槽

###  使用多线程

* C++ 多线程
* Qml WorkerScript 元件

###  使用 Qt Quick Compiler

在 pro 文件添加 `CONFIG += qtquickcompiler`。

```shell 
#!/bin/sh
rm *.rcc
rcc -binary qml.qrc -o qml.rcc
```

**补充（ 待确认）**：

* Qt 5.12 会有新的 Qml 代码生成器 ，性能比 QuickCompiler 。

###  避免使用 CPU 渲染的元件

* Canvas

* Qt Charts

###  使用异步加载

补充： Qml 的 Image 元件有 异步加载属性 asynchronous 默认为 false 设置 为 true 就是异步加载

* 图片异步加载

* C++ 处理大数据加载

###  JavaScript Code 关于 JavaScript 优化使用

####  属性绑定

1. QML 优化了引擎，简单的表达式不会启动 JavaScript 。

```JavaScript
x : 10 + 2 * 5 		//qml 直接计算
```

2. 避免申明 JavaScript 中间变量

```javascript
for (var i = 0; i < 100; ++i) 
{
	var obj = xxx;		// 中间变量 obj 频繁创建，会有性能问题
	obj.xxx;
}
```

3. 避免在即时求值 范围外访问属性 （绑定表达式所在对象的属性 ，组件中的 id ，组件中的根部元素 id 需要用到属性进行运算时避免直接写操作。

```javascript
// bad
for (var i = 0; i < 100; ++1)
{
    Item.x += doSomething(i);
}

// good
var pos = 0;
for (var i = 0; i < 100; ++i)
{
    pos += doSomething(i);
}
Item.x = pos;		// 局部变量存储结果， 执行一次读写操作
```

#### 属性解析

避免频繁访问属性 ( QML Profiler 看到的调用频率比较高的部分，要尤其注意不要进行属性访问 ）

###  Qt Quick 图片和布局优化

####  降低图片加载时间和内存开销

* 异步加载
* 设置图片尺寸 （ 待确认 ）

#### 锚定布局

坐标 > 锚定 > 绑定 > JavaScript 函数

### 元素周期设计

使用 Loader 动态加载和卸载组件

* 使用 active 属性，可以延迟实例化。
* 使用 source 属性，可以提供初始属性值。
* asynchronous 异步属性设置为 true，在组件实例时可以提高流畅性

###  渲染注意事项

* 避免使用 Clip 属性，剪切损失性能。
* 被覆盖的元素设置 visible 为 false 。
* 透明 与 不透明：不透明效率更高，全透明设置不可见。
* 半透明组件设置为 `visible ：opacity >  0`

### 使用 Animation 不使用 Timer

Qt 优化了动画的实现，性能高于定时器触发属性的改变。Timer 触发动画的方式性能低下、更耗电。