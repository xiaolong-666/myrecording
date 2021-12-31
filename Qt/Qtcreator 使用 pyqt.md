# Qtcreator 使用 pyqt

## 准备环境

1. 首先安装pyqt5

```bash
pip3 install pyqt5
```

如果提示`安装失败`

则执行以下命令，升级pip

```bash
pip3 install --upgrade pip
```

2. 将`$HOME/.local/bin`添加到环境变量中



## 创建UI界面

使用qtcreator创建ui界面



## 将UI 转换为 PY 文件

使用 `pyuic5` 命令转换

```bash
pyuic5 mainwindow.ui -o mainwindow.py
```



## PY 文件加载界面

1. 开启 `pyqt` 调试模式: `export QT_DEBUG_PLUGINS=1`

2. 如果提示，找不到`libxcb-util.so.1  `， 创建软链接。[参考](https://blog.csdn.net/Fozei/article/details/116160454)

```bash
sudo ln -s /usr/lib/x86_64-linux-gnu/libxcb-util.so.0 /usr/lib/x86_64-linux-gnu/libxcb-util.so.1  
```



加载界面的简单demo

```pytho
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import sys

from PyQt5 import QtWidgets
from mainwindow import Ui_MainWindow
from PyQt5.QtWidgets import QFileDialog

class MyMainWindow(QtWidgets.QMainWindow):
    """
    继承 UI 界面属性
    """
    def __init__(self):

        # 初始化 UI 对象
        super(MyMainWindow,self).__init__()
        self.ui = Ui_MainWindow()
        self.ui.setupUi(self)

        # 链接选择多个文件槽函数
        self.ui.pbtn_select_files.clicked.connect(self.slot_select_files)
       
    def slot_select_files(self):
        """
        选择多个execl文件
        :return:
        """
        files, filetype= QFileDialog.getOpenFileNames(self,"选取多个文件", "./", "Excel files (*.xls *.xlsx )")
        if len(files) == 0:
            print("取消选择")
            return
        for file in files:
            print(file)
            text = "选择的文件："+ file
            self.ui.plainTextEdit.appendPlainText(text)

if __name__ == '__main__':
    app = QtWidgets.QApplication(sys.argv)

    mywindow = MyMainWindow()
    mywindow.show()

    sys.exit(app.exec_())

```



