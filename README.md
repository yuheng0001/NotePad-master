# 期中实验
---
## 增加时间戳

在NotePad应用中，找到显示标题的页面：noteslist_item.xml<br>
将显示标题的TextView放入垂直线性布局中<br>
再添加一个TextView显示时间,如下<br>
![](https://github.com/yuheng0001/NotePad-master/blob/master/image/2.png)<br>
![]()<br>
在NoteList.java中修改PROJECTION<br>
![](https://github.com/yuheng0001/NotePad-master/blob/master/image/3.png)<br>
在dataColumns，viewIDs中添加时间部分：
![](https://github.com/yuheng0001/NotePad-master/blob/master/image/4.png)<br>
最后在NotePadProvider.java中的insert方法和NoteEditor.java中的updateNote方法中添加时间转换<br>
![](https://github.com/yuheng0001/NotePad-master/blob/master/image/5.png)<br>
效果图如下：<br>
![](https://github.com/yuheng0001/NotePad-master/blob/master/image/1.png)<br>

## 增加时间戳
找到ist_options_menu.xml<br>
添加一个搜索的item<br>
搜索图标用安卓自带的图标，设为总是显示<br>
![](https://github.com/yuheng0001/NotePad-master/blob/master/image/6.png)<br>
