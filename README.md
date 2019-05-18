# MyNotePad
## 目录  
* [笔记实现的功能](#笔记实现的功能)    
  * [原有功能](#原有功能)
  * [要求实现的功能](#要求实现的功能)
  * [拓展的功能](#拓展的功能)
* [笔记时间戳](#笔记时间戳)  
* [笔记按标题搜索](#笔记按标题搜索)
* [笔记排序](#笔记排序)
* [笔记更换背景颜色](#笔记更换背景颜色代码分)
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
* UI界面美化
* 笔记内容排序：按照创建笔记时间、修改笔记时间、笔记背景颜色排序
* 更换笔记背景颜色
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
<a name="笔记排序"></a>  
## 笔记排序
<a name="笔记更换背景颜色"></a>  
## 笔记更换背景颜色
<a name="导出笔记功能"></a>  
## 导出笔记功能
