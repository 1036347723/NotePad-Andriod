# NotePad-Andriod
The Andriod Homework   
   
这是一个记事本应用，已完成时间戳与查找笔记的两个基本功能，同时增加了给每一条笔记增加背景色、切换夜间模式、设置字体大小的拓展功能。
# 1.时间戳功能
## 1.1在noteslist_item.xml中增加TextView
这个textview是为了显示时间戳
```
<TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingLeft="5dip"
        android:singleLine="true"
        android:gravity="center_vertical"
    />
```
## 1.2修改时间戳类型，将其变为 年-月-日 时：分：秒 格式
涉及到时间戳的修改的地方为新建笔记和修改笔记时，对应在NotePadProvider中的insert方法与NoteEditor中的updateNote方法。
NotePadProvider中的insert方法:
```
Long now = Long.valueOf(System.currentTimeMillis());
Date date = new Date(now);
//在这里将时间戳进行转换，变为所需格式
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
String dateFormat = simpleDateFormat.format(date);
if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, dateFormat);
}
if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);
}
```
NoteEditor中的updateNote方法:
```
long now = System.currentTimeMillis();
Date date = new Date(now);
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
String dateFormat = simpleDateFormat.format(date);
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);
```
## 1.3 修改NoteList
在原本的PROJECTION中增加修改时间
```
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//添加修改时间
    };
```
显示列dataColumns和他们的viewIDs加入修改时间
```
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE }//加入修改时间;
int[] viewIDs = { android.R.id.text1, R.id.text2 }//加入修改时间;
```
## 1.4 运行截图：
![时间戳]（img/时间戳.png）
