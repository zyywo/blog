<!--markdown-->1. 关键字不分大小写
2. 变量先定义再使用

`Range("A1")`：选择A1单元格
`MsgBox("消息")`：显示一个弹窗
`Sub`：关键字，定义函数使用
`Dim`：关键字，定义变量使用

```
# 弹窗显示表格有多少行内容
Dim rows as Interger
rows = Sheet2.UsedRange.rows.Count
MsgBox(rows)
```

## if 语句
```
# A1单元格的内容加1，等于10后不在增加
Sub CommandButton1_Click()
    If Range("a1").Value = 10 Then
        MsgBox ("已经封顶了，别再点了")
    Else
        Range("a1") = Range("a1") + 1
    End If
End Sub
```	