<!--markdown-->1. 关键字不分大小写
2. 变量先定义再使用

`Range("A1")`：选择A1单元格
`MsgBox("消息")`：显示一个弹窗
`Sub`：关键字，定义函数使用
`Dim`：关键字，定义变量使用

## 弹窗显示表格有多少行内容
```
Dim rows as Interger
rows = Sheet2.UsedRange.rows.Count
MsgBox(rows)
```

## if 语句
```
'A1单元格的内容加1，等于10后不在增加
Sub CommandButton1_Click()
    If Range("a1").Value = 10 Then
        MsgBox ("已经封顶了，别再点了")
    Else
        Range("a1") = Range("a1") + 1
    End If
End Sub
```
## 确保单元格A2的最大值不能超过已使用的最大行数
```
sub worksheet_selectionChange(byval Target As Range)
    if Target.count > 1 then
    '全选表格时上面的Targt.Count会提示数值溢出，可以改写为下面的语句
    'if target.rows.count >1 or target.columns.count>1 then
        exit sub
    end if

    dim max as integer
    max = sheet2.usedrange.rows.count

    if range("a2").value > max then
        range("a2") = max
        msgbox "不能大于" & max，vbOKOnly, "提示"
        '单引号是注释，上面的msgbox也可以写成下面这样
        't=msgbox("不能大于" & max，vbOKOnly, "提示")
        'msgbox("不能大于" & max)
    end if
end sub
```	