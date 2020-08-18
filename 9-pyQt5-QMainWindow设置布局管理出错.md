<!--markdown-->在QMainWindow设置布局管理器时会报错，提示它已有了layout

### 错误提示：

>QWidget::setLayout: Attempting to set QLayout "" on MainWindow "", which already has a layout

### 原因：

You can't set a QLayout directly on the QMainWindow. You need to create a QWidget and set it as the central widget on the QMainWindow and assign the QLayout to that.
```
wid = QtGui.QWidget(self)
self.setCentralWidget(wid)
layout = QtGui.QVBoxLayout()
wid.setLayout(layout)
```

参考链接：[stackoverflow](https://stackoverflow.com/questions/37304684/qwidgetsetlayout-attempting-to-set-qlayout-on-mainwindow-which-already)
	