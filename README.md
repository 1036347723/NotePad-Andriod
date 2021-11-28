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
![时间戳](/img/时间戳.png)




# 2.搜索功能

## 2.1在list_options_menu.xml中添加一个搜索的item

```
<!--添加一个搜索的按钮-->
    <item
        android:id="@+id/menu_search"
        android:title="@string/menu_search"
        android:icon="@android:drawable/ic_search_category_default"
        android:showAsAction="always">
    </item>
```

## 2.2在NoteList的onOptionsItemSelected方法中增加查找按钮的case语句:
```
case R.id.menu_search:
//           查找按钮的事件
            Intent intent = new Intent();
            intent.setClass(NotesList.this,NoteSearch.class);
            NotesList.this.startActivity(intent);;
            return true;
```


## 2.3在layout中新建布局文件note_search_list.xml：
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="请输入要查询的内容..."
        android:layout_alignParentTop="true">
    </SearchView>
    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>
</LinearLayout>
```

## 2.4创建NoteSearch类
```
public class NoteSearch extends ListActivity  implements SearchView.OnQueryTextListener {
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
      
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search_list);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        SearchView searchview = (SearchView)findViewById(R.id.search_view);
        searchview.setOnQueryTextListener(NoteSearch.this);  
    }
    @Override
    public boolean onQueryTextSubmit(String query) {
        return false;
    }
    @Override
    public boolean onQueryTextChange(String newText) {
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";
        String[] selectionArgs = { "%"+newText+"%" };
        Cursor cursor = managedQuery(
                getIntent().getData(),         
                PROJECTION,                      
                selection,                      
                selectionArgs,                    
                NotePad.Notes.DEFAULT_SORT_ORDER 
        );
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text2 };
        SimpleCursorAdapter adapter = new SimpleCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;
    }
    @Override
    protected void onListItemClick(ListView l, View v, int position, long id) {

        Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);
        String action = getIntent().getAction();
        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {
            setResult(RESULT_OK, new Intent().setData(uri));
        } else {
            startActivity(new Intent(Intent.ACTION_EDIT, uri));
        }
    }
}
```
## 2.5最后在AndroidManifest.xml注册NoteSearch：
```
    <activity
        android:name="NoteSearch"
        android:label="search">
    </activity>
```
