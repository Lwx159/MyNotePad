# MyNotePad
## 目录  
* [笔记实现的功能](#笔记实现的功能)    
  * [原有功能](#原有功能)
  * [要求实现的功能](#要求实现的功能)
  * [拓展的功能](#拓展的功能)
* [笔记时间戳](#笔记时间戳)  
* [笔记按标题搜索](#笔记按标题搜索)
* [笔记界面美化](#笔记界面美化)
* [笔记更换背景](#笔记更换背景)
* [笔记排序](#笔记排序)
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
5. 至此，时间戳功能完成,截图展示：
![image](
https://github.com/Lwx159/MyNotePad/blob/master/img/2%E3%80%81time.png)

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
6. 至此，笔记搜索功能完成，截图展示：
* 搜索页面：

![image](https://github.com/Lwx159/MyNotePad/blob/master/img/3%E3%80%81search.png)

* 搜索沁园春：

![image](https://github.com/Lwx159/MyNotePad/blob/master/img/4%E3%80%81search2.png)

* 搜索沁园春结果：

![image](https://github.com/Lwx159/MyNotePad/blob/master/img/5%E3%80%81search3.png)


<a name="笔记界面美化"></a>  
 ## 笔记界面美化
 1. 修改**AndroidManifest.xml**的**NotesList**:
 ```
 <activity android:name="NotesList" android:label="@string/title_notes_list"
            android:theme="@android:style/Theme.Holo.Light">
```
4. 在**NotePad**中添加：
```
public static final String COLUMN_NAME_BACK_COLOR = "color";
        public static final int DEFAULT_COLOR = 0; //颜色对应的数值
        public static final int YELLOW_COLOR = 1;
        public static final int BLUE_COLOR = 2;
        public static final int GREEN_COLOR = 3;
        public static final int RED_COLOR = 4;
        public static final int PURPLE_COLOR=5;
```
5. 在**NotePadProvider**中：
* 数据库定义里添加一行颜色字段 **NotePad.Notes.COLUMN_NAME_BACK_COLOR**：
```
@Override
       public void onCreate(SQLiteDatabase db) {
           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                   + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER"
                   + ");");
       }
```
* 在static{}中添加：
```
sNotesProjectionMap.put(
        NotePad.Notes.COLUMN_NAME_BACK_COLOR,
        NotePad.Notes.COLUMN_NAME_BACK_COLOR);
```
* 在insert中添加：
```
 // 新建笔记，背景默认为白色
if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
        values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
        }
```
6. 新建**MyCursorAdapter.java**:
```
package com.example.android.notepad;

import android.content.Context;
import android.database.Cursor;
import android.graphics.Color;
import android.view.View;
import android.widget.SimpleCursorAdapter;

/**
 * Created by Administrator on 2017/5/18.
 */

public class MyCursorAdapter extends SimpleCursorAdapter {


    public MyCursorAdapter(Context context, int layout, Cursor c,
                           String[] from, int[] to) {
        super(context, layout, c, from, to);
    }


    @Override
    public void bindView(View view, Context context, Cursor cursor){
        super.bindView(view, context, cursor);
        //从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色
        int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
        
        switch (x){
            case NotePad.Notes.DEFAULT_COLOR:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
            case NotePad.Notes.YELLOW_COLOR:
                view.setBackgroundColor(Color.rgb(255, 255, 153));
                break;
            case NotePad.Notes.BLUE_COLOR:
                view.setBackgroundColor(Color.rgb(204, 204, 255));
                break;
            case NotePad.Notes.GREEN_COLOR:
                view.setBackgroundColor(Color.rgb(204, 255, 153));
                break;
            case NotePad.Notes.RED_COLOR:
                view.setBackgroundColor(Color.rgb(255, 204, 204));
                break;
            case NotePad.Notes.PURPLE_COLOR:
                view.setBackgroundColor(Color.rgb(226,178,242));
                break;
            default:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
        }
    }
}
```
7. 修改**NotesList**：
*
 ```
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };
```
*
```
 //修改为可以填充颜色的自定义的adapter，自定义的代码在MyCursorAdapter.java中
adapter = new MyCursorAdapter(
        this,
        R.layout.noteslist_item,
        cursor,
        dataColumns,
        viewIDs
    );
```
8. 至此，笔记界面从黑色变成白色，为接下来的背景改变做准备
* 编辑界面：
![image](https://github.com/Lwx159/MyNotePad/blob/master/img/1%E3%80%81editor.png)
* 首页面：
![image](https://github.com/Lwx159/MyNotePad/blob/master/img/2%E3%80%81time.png)
<a name="笔记更换背景"></a>  
## 笔记更换背景
1. 在**NoteEditor**中:
* **PROJECTION** 添加颜色项BACK_COLOR：
```
  private static final String[] PROJECTION =
        new String[] {
            NotePad.Notes._ID,
            NotePad.Notes.COLUMN_NAME_TITLE,
            NotePad.Notes.COLUMN_NAME_NOTE,
            NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };
```
*  protected void onResume() 方法添加：
```
protected void onResume() {
//省略部分代码
...
 //读取颜色数据做准备
            int x = mCursor.getInt(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));

            switch (x){
                case NotePad.Notes.DEFAULT_COLOR:
                    mText.setBackgroundColor(Color.rgb(255, 255, 255));
                    break;
                case NotePad.Notes.YELLOW_COLOR:
                    mText.setBackgroundColor(Color.rgb(255, 255, 153));
                    break;
                case NotePad.Notes.BLUE_COLOR:
                    mText.setBackgroundColor(Color.rgb(204, 204, 255));
                    break;
                case NotePad.Notes.GREEN_COLOR:
                    mText.setBackgroundColor(Color.rgb(204, 255, 153));
                    break;
                case NotePad.Notes.RED_COLOR:
                    mText.setBackgroundColor(Color.rgb(255, 204, 204));
                    break;
                case NotePad.Notes.PURPLE_COLOR:
                    mText.setBackgroundColor(Color.rgb(226,178,242));
                    break;
                default:
                    mText.setBackgroundColor(Color.rgb(255, 255, 255));
                    break;
            }
            ...
            //省略部分代码
   ｝
```
2. 在**editor_options_menu.xml**中添加一个更改背景颜色的项
```
  <!--更换背景-->
    <item android:id="@+id/menu_color"
        android:title="@string/menu_color"
        android:icon="@drawable/ic_menu_color"
        android:showAsAction="always"/>
```
3. 新建布局**note_color.xml**
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">
    <ImageButton
        android:id="@+id/color_white"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorWhite"
        android:onClick="white"/>

    <ImageButton
        android:id="@+id/color_yellow"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorYellow"
        android:onClick="yellow"/>

    <ImageButton
        android:id="@+id/color_blue"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorBlue"
        android:onClick="blue"/>

    <ImageButton
        android:id="@+id/color_green"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorGreen"
        android:onClick="green"/>

    <ImageButton
        android:id="@+id/color_red"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorRed"
        android:onClick="red"/>
    <ImageButton
        android:id="@+id/color_purple"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorPurple"
        android:onClick="purple"/>
</LinearLayout>
</LinearLayout>
```
4. 新建**NoteColor.java**
```
package com.example.android.notepad;

import android.app.Activity;
import android.content.ContentValues;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;

/**
 * Created by Administrator on 2017/5/18.
 */

public class NoteColor extends Activity {

    private Cursor mCursor;
    private Uri mUri;
    private int color;
    private static final int COLUMN_INDEX_TITLE = 1;

    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };

    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_color);
        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,        // The URI for the note that is to be retrieved.
                PROJECTION,  // The columns to retrieve
                null,        // No selection criteria are used, so no where columns are needed.
                null,        // No where columns are used, so no where values are needed.
                null         // No sort order is needed.
        );

    }

    @Override
    protected void onResume(){
        if (mCursor != null) {
            mCursor.moveToFirst();
            color = mCursor.getInt(COLUMN_INDEX_TITLE);
        }
        super.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, color);
        getContentResolver().update(mUri, values, null, null);

    }

    public void white(View view){
        color = NotePad.Notes.DEFAULT_COLOR;
        finish();
    }

    public void yellow(View view){
        color = NotePad.Notes.YELLOW_COLOR;
        finish();
    }

    public void blue(View view){
        color = NotePad.Notes.BLUE_COLOR;
        finish();
    }

    public void green(View view){
        color = NotePad.Notes.GREEN_COLOR;
        finish();
    }

    public void red(View view){
        color = NotePad.Notes.RED_COLOR;
        finish();
    }
    public void purple(View view){
        color = NotePad.Notes.PURPLE_COLOR;
        finish();
    }
}
```
5. 在**AndroidManifest.xml**中定义**NoteColor**
```
       <!--换背景色-->
        <activity android:name="NoteColor"
            android:theme="@android:style/Theme.Holo.Light.Dialog"
            android:label="更换背景颜色"
            android:windowSoftInputMode="stateVisible"/>
```
6. 在**NoteEditor.java**中：
* onOptionsItemSelected（）**方法中添加：
```
 public boolean onOptionsItemSelected(MenuItem item) {
  // Handle all of the possible menu actions.
        switch (item.getItemId()) {
        ...
        //省略部分代码
           //换背景颜色选项
            case R.id.menu_color:
                changeColor();
                break;
            ...
            //省略部分代码
  }
```
* 在NoteEditor中添加函数changeColor()：<br>
```
//跳转改变颜色的activity，将uri信息传到新的activity
    private final void changeColor() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,NoteColor.class);
        NoteEditor.this.startActivity(intent);
    }
```
7. 至此，笔记更换背景颜色功能完成。
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
