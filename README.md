# notepad
mid experiment  

##1.基本要求  
###实现时间戳功能  
在定义notes表时，已经定义了COLUMN_NAME_MODIFICATION_DATE（更改时间）  
```
public static final String COLUMN_NAME_MODIFICATION_DATE = "modified";
```  

所以只需要将时间数据从数据库中取出显示即可  
（1）先在layout下找到显示便签的notelist_item.xml，在其中改为线性布局，添加显示时间戳的一个TextView  

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:paddingLeft="6dip"
        android:paddingRight="6dip"
        android:paddingBottom="3dip"
        >
    <TextView 
        android:id="@android:id/text1"
        android:layout_width="wrap_content"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"
    />
    <TextView 
        android:id="@android:id/text2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"
    />
   
</LinearLayout>
```  
(2)到显示notes的noteList类中，将时间戳添加到游标需要的列中，用来查询时间
```
 private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, 
            NotePad.Notes.COLUMN_NAME_TITLE, 
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
    };
```  
(3)在视图中显示的游标列中加入时间：
```
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;
```  
(4)将时间戳的显示，放到(1)中新添加的Textview中(id为text2)  
```
int[] viewIDs = { android.R.id.text1,android.R.id.text2};
```  
(5)通过SimpleCursorAdapter adapter将游标中的数据映射到布局文件中的TextView中  
```
SimpleCursorAdapter adapter
            = new SimpleCursorAdapter(
                      this,                             // The Context for the ListView
                      R.layout.noteslist_item,          // Points to the XML for a list item
                      cursor,                           // The cursor to get items from
                      dataColumns,
                      viewIDs
              );

        // Sets the ListView's adapter to be the cursor adapter that was just created.
        setListAdapter(adapter);
```  
(6)但最终显示的时间为一串int型日期，所以要将int转为Date型，可以在存入数据时转换，也可以在取出数据时转换，我选在存入数据是转换  
(7)在NoteEditor中找到更新时间的函数updateNote（），将存入的当前时间转为Date类型  
 ```
        ContentValues values = new ContentValues();
        //改变时间格式
        long currentTime = System.currentTimeMillis();
        SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd：HH：mm");
        Date date = new Date(currentTime);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, formatter.format(date));
 ```
(8)查看最终显示效果：  
![1](https://github.com/Xiaohui-Song/notepad/blob/main/pictures/notes.png)  
<br>  
###查询功能
