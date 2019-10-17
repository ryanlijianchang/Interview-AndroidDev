### 一、View绘制过程 ###

onMeasure->onLayout->onDraw

详看：[《面试题之View的绘制流程》](https://www.jianshu.com/p/8a71cbf7622d)

### 二、onMeasure ###

从上往下，遍历子容器，后去子容器的大小。

- ViewGroup.LayoutParams：设置宽高
- MeasureSpec：测量规格。前两位是测量模式（不确定的、exactly确定大小，atmost限定子控件最大大小），后30位是测量大小


### 三、draw两个容易混淆的方法 ###

1. invalidate()
2. requestLayout()

### 四、scrollTo和scrollBy的区别 ###

scrollTo()：移动到具体的坐标
scrollBy()：在原有的位置移动一定的距离