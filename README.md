# MyNotePad
## 目录  
* [笔记实现的功能](#笔记实现的功能)    
  * [原有功能](#原有功能)
  * [要求实现的功能](#要求实现的功能)
  * [拓展的功能](#拓展的功能)
* [笔记时间戳](#笔记时间戳)  
* [笔记按标题搜索](#笔记按标题搜索)
* [笔记更换背景](#笔记更换背景)
* [笔记排序](#笔记排序)
* [导出笔记功能](#导出笔记功能)
<a name="笔记实现的功能"></a>  
## 笔记实现的功能
<a name="原有功能"></a>  
##### 原有功能：
* 新建笔记
* 编辑笔记标题
* 保存笔记
* 复制、粘贴、删除笔记
<a name="要求实现的功能"></a>  
#### 要求实现的功能：
* 笔记时间戳
* 笔记按标题搜索
<a name="拓展的功能"></a>  
#### 拓展的功能：
* 更换笔记背景颜色
* 笔记内容排序：按照创建笔记时间、修改笔记时间、笔记背景颜色排序
* 导出笔记
<a name="笔记时间戳"></a>  
## 笔记时间戳
1. 修改**layout**中**noteslist_item.xml**布局文件:
* 在LinearLayout里设置垂直显示布局：
```
<LinearLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
```
* 添加一个显示时间戳的TextView，并设置其字体大小和颜色
```
 <TextView
        android:id="@+id/text1_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingLeft="5dip"
        android:textColor="@color/colorBlack"/>
 ```
2. 在**NotePadProvider.java**中定义了数据库，其中**CREATE_DATE**是创建笔记的时间，**MODIFICATION_DATE**是修改笔记的时间，这就是我们之后要用到的时间数据。
```
 @Override
        public void onCreate(SQLiteDatabase db) {
           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                   + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
                   + ");");
       }
 ```
 3. 在**NoteList.java**中添加一行修改时间**NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE**
 ```
 private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            // 修改时间
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
    };
```
4.  把时间戳改为以时间格式存入:
     改动地方有两处：
* 第一处为**NotePadProvider.java** ，创建时间
```
     public Uri insert(Uri uri, ContentValues initialValues) {
        //省略部分代码
         // 将milliseconds转化为一定的时间格式
        Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
        String dateTime = format.format(date);
       //省略部分代码
       }
```
* 第二处：**NoteEditor.java**，修改时间
```
 private final void updateNote(String text, String title) {

     //省略代码
     ...
        //修改时间
        Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
        String dateTime = format.format(date);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateTime);
        ...
       //省略代码
       }
```
5. 至此，时间戳功能完成。

<a name="笔记按标题搜索"></a>  
## 笔记按标题搜索
1. 在**list_options_menu.xml**中添加搜索项:
```
    <item
        android:id="@+id/menu_search"
        android:title="@string/menu_search"
        android:icon="@android:drawable/ic_search_category_default"
        android:showAsAction="always">
    </item>
```
2. 新建搜索的布局文件**note_search_list.xml**：
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
        android:queryHint="输入搜索内容..."
        android:layout_alignParentTop="true">
    </SearchView>

    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>

</LinearLayout>
```
3. 新建一个名为**NoteSearch**的**activity**:
```
package com.example.android.notepad;

import android.app.ListActivity;
import android.content.ContentUris;
import android.content.Intent;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;
import android.widget.ListView;
import android.widget.SearchView;
import android.widget.SimpleCursorAdapter;


/**
 * Created by Administrator on 2017/5/10.
 */

public class NoteSearch extends ListActivity  implements SearchView.OnQueryTextListener {

    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
            NotePad.Notes.COLUMN_NAME_BACK_COLOR
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
        searchview.setOnQueryTextListener(NoteSearch.this);  //为查询文本框注册监听器
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
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                selection,                        // 条件左边
                selectionArgs,                    // 条件右边
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );

        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text1_time };

        MyCursorAdapter adapter = new MyCursorAdapter(
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

        // Constructs a new URI from the incoming URI and the row ID
        Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);

        // Gets the action from the incoming Intent
        String action = getIntent().getAction();

        // Handles requests for note data
        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {

            // Sets the result to return to the component that called this Activity. The
            // result contains the new URI
            setResult(RESULT_OK, new Intent().setData(uri));
        } else {

            // Sends out an Intent to start an Activity that can handle ACTION_EDIT. The
            // Intent's data is the note ID URI. The effect is to call NoteEdit.
            startActivity(new Intent(Intent.ACTION_EDIT, uri));
        }
    }
}
```
4. 修改**NotesList.java**的**onOptionsItemSelected方法**，在switch中添加搜索的case语句:
```
 //添加搜素
            case R.id.menu_search:
                Intent intent = new Intent();
                intent.setClass(NotesList.this,NoteSearch.class);
                NotesList.this.startActivity(intent);
                return true;
```
5. 在**AndroidManifest.xml**注册**NoteSearch**：
```
<!--添加搜索activity-->
    <activity
        android:name="NoteSearch"
        android:label="@string/title_notes_search">
    </activity>
```
6. 至此，笔记搜索功能完成。
<a name="笔记更换背景"></a>  
## 笔记更换背景
<a name="笔记排序"></a>  
## 笔记排序
1.在 **list_options_menu.xml** 中添加排序项
```
<!--排序 按照创建时间、修改时间、颜色-->
    <item
        android:id="@+id/menu_sort"
        android:title="@string/menu_sort"
        android:icon="@android:drawable/ic_menu_sort_by_size"
        android:showAsAction="always" >
        <menu>
            <item
                android:id="@+id/menu_sort1"
                android:title="@string/menu_sort1"/>
            <item
                android:id="@+id/menu_sort2"
                android:title="@string/menu_sort2"/>
            <item
                android:id="@+id/menu_sort3"
                android:title="@string/menu_sort3"/>
        </menu>
    </item>
```
2. 在**NotesList** 的switch下添加三种排序方法的case：
```
            //创建时间排序
            case R.id.menu_sort1:
                cursor = managedQuery(
                        getIntent().getData(),            // Use the default content URI for the provider.
                        PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                        null,                             // No where clause, return all records.
                        null,                             // No where clause, therefore no where column values.
                        NotePad.Notes._ID  // Use the default sort order.
                );
                adapter = new MyCursorAdapter(
                        this,
                        R.layout.noteslist_item,
                        cursor,
                        dataColumns,
                        viewIDs
                );
                setListAdapter(adapter);
                return true;

            //修改时间排序
            case R.id.menu_sort2:
                cursor = managedQuery(
                        getIntent().getData(),            // Use the default content URI for the provider.
                        PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                        null,                             // No where clause, return all records.
                        null,                             // No where clause, therefore no where column values.
                        NotePad.Notes.DEFAULT_SORT_ORDER // Use the default sort order.
                );

                adapter = new MyCursorAdapter(
                        this,
                        R.layout.noteslist_item,
                        cursor,
                        dataColumns,
                        viewIDs
                );
                setListAdapter(adapter);
                return true;

            //颜色排序
            case R.id.menu_sort3:
                cursor = managedQuery(
                        getIntent().getData(),            // Use the default content URI for the provider.
                        PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                        null,                             // No where clause, return all records.
                        null,                             // No where clause, therefore no where column values.
                        NotePad.Notes.COLUMN_NAME_BACK_COLOR // Use the default sort order.
                );
                adapter = new MyCursorAdapter(
                        this,
                        R.layout.noteslist_item,
                        cursor,
                        dataColumns,
                        viewIDs
                );
                setListAdapter(adapter);
                return true;
```
3.  至此，笔记排序功能完成。
<a name="导出笔记功能"></a>  
## 导出笔记功能
