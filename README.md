# notepad
mid experiment  

## 1.基本要求  
### 实现时间戳功能  
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
(3)在视图中显示的游标列中加入更新时间：
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
(6)但最终显示的时间为一串int型日期，所以要将数据转型，可以在存入数据时转换，也可以在取出数据时转换，我选在存入数据时转换  
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
![1](https://github.com/Xiaohui-Song/notepad/blob/main/pictures/notes_before.png)  
<br>  
### 查询功能  
(1)在layout下创建note_search.xml，添加一个搜索图标  
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="搜索">
    </SearchView>
    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        >
    </ListView>
</LinearLayout>
```  
(2)在NoteEditor中找到onCreatOptionMenu()函数，将新写的xml布局，映射到menu中  
```
MenuInflater inflater = getMenuInflater();
inflater.inflate(R.menu.editor_options_menu, menu);
```  
(3)在NoteList找到onOptionsItemSelected()函数，添加选择搜索事件的处理  
```
case R.id.menu_search:
                Intent intent = new Intent();
                intent.setClass(this, NoteSearch.class);
                this.startActivity(intent);
                return true;
```  
(4)添加一个搜索的Activity事件，Activity中获取到note_search中的ListView和SearchView  
```
listview=findViewById(R.id.list_view);
SearchView search=findViewById(R.id.search_view);
```  
(5)为SearchView添加监听，使得搜索内容改变后，重新执行一次查询，在query的selection参数采用模糊查询，可以通过COLUMN_NAME_NOTE或者COLUMN_NAME_TITLE查询内容
最后通过SimpleCursorAdapter映射查询到的内容
```
 sqLiteDatabase=new NotePadProvider.DatabaseHelper(this).getReadableDatabase();
 @Override
    public boolean onQueryTextChange(String newText) {
        Cursor cursor=sqLiteDatabase.query(
                NotePad.Notes.TABLE_NAME,
                PROJECTION,
                NotePad.Notes.COLUMN_NAME_TITLE+" like ? or "+NotePad.Notes.COLUMN_NAME_NOTE+" like ?",
                new String[]{"%"+newText+"%","%"+newText+"%"},//根据标题或内容模糊查询
                null,
                null,
                NotePad.Notes.DEFAULT_SORT_ORDER);
        int[] viewIDs = { android.R.id.text1,android.R.id.text2};
        SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                NoteSearch.this,
                R.layout.noteslist_item,
                cursor,
                new String[]{NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE},
                viewIDs
        );
        listview.setAdapter(adapter);
        return true;
    }
```  
(6)通过setOnItemClickListener为ListView添加监听，点击可以进入编辑页面，查看到notes的详细内容
```
   listview.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int i, long l) {
                Uri uri = ContentUris.withAppendedId(getIntent().getData(), l);
                String action = getIntent().getAction();
                if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {
                    setResult(RESULT_OK, new Intent().setData(uri));
                } else {
                    startActivity(new Intent(Intent.ACTION_EDIT, uri));
                }
            }
        });
```  
(7)查看效果 （按内容/标题查询）
![2](https://github.com/Xiaohui-Song/notepad/blob/main/pictures/search_before.png)
![3](https://github.com/Xiaohui-Song/notepad/blob/main/pictures/search2_before.png)  
<br>  
## 附加功能  
### 3.UI美化  
(1)为其添加便签图标，但在观察drawable下的app_notes已经有了便签图标，所以直接将其显示即可  
(2)在noteslist_item.xml中再添加一个线性布局，形成嵌套，外部线性布局添加便签的图片显示,并使其相对于父容器居中显示  
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip"
    >

    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:background="@drawable/app_notes"
     />
    <LinearLayout>
    ······
    </LinearLayout>
</LinearLayout>
```  
(3)为notes添加背景，向drawable下放置background.png图片，在noteslist_item.xml的外层线性布局添加图片
```
android:background="@drawable/background"
```  
(4)查看效果  
![4](https://github.com/Xiaohui-Song/notepad/blob/main/pictures/notes.png)  
![5](https://github.com/Xiaohui-Song/notepad/blob/main/pictures/search.png)  
### 4.更改记事本背景  
(1)观察到记事本已经有menu选项：Edit title，所以他已经有onCreateOptionsMenu()函数，只需要在该函数内添加改变背景菜单的选项，并在onOptionsItemSelected()函数的case中添加对应的点击相应即可  
(2)找到NoteEditor下的onCreateOptionsMenu()函数，向其中添加“backgroun color”“size”标题，并向期中加入子标题  
```
        SubMenu colorMenu = menu.addSubMenu("background color");
        colorMenu.setHeaderTitle("select color");
        colorMenu.add(0,COLOR_1,0,"杏色");//#fab27b
        colorMenu.add(0,COLOR_2,0,"浅绿");//#84bf96
        colorMenu.add(0,COLOR_3,0,"藤色");//#afb4db

        SubMenu fontMenu = menu.addSubMenu("size");
        fontMenu.setHeaderTitle("select font");
        fontMenu.add(0,FONT_12,0,"12号字体");
        fontMenu.add(0,FONT_16,0,"16号字体");
        fontMenu.add(0,FONT_20,0,"20号字体");
```
(3)在下面的onOptionsItemSelected()函数中，对相应的点击事件添加case，做出响应（改变颜色、字体后，弹出提示信息）
```
            case COLOR_1:
                mText.setBackgroundColor(Color.parseColor("#fab27b"));
                Toast.makeText(this,"颜色变为杏色",Toast.LENGTH_SHORT).show();
            break;
            case COLOR_2:
                mText.setBackgroundColor(Color.parseColor("#84bf96"));
                Toast.makeText(this,"颜色变为浅绿",Toast.LENGTH_SHORT).show();
                break;
            case COLOR_3:
                mText.setBackgroundColor(Color.parseColor("#afb4db"));
                Toast.makeText(this,"颜色变为藤色",Toast.LENGTH_SHORT).show();
                break;
            case FONT_12:
                mText.setTextSize(12*2);
                Toast.makeText(this,"12号字体",Toast.LENGTH_SHORT).show();
                break;
            case FONT_16:
                mText.setTextSize(16*2);
                Toast.makeText(this,"16号字体",Toast.LENGTH_SHORT).show();
                break;
            case FONT_20:
                mText.setTextSize(20*2);
                Toast.makeText(this,"20号字体",Toast.LENGTH_SHORT).show();
                break;
```  
(4)查看效果
![6](https://github.com/Xiaohui-Song/notepad/blob/main/pictures/edit_menu.png)
![7](https://github.com/Xiaohui-Song/notepad/blob/main/pictures/changebg.png)
![8](https://github.com/Xiaohui-Song/notepad/blob/main/pictures/color1.png)
![9](https://github.com/Xiaohui-Song/notepad/blob/main/pictures/changesize.png)
![10](https://github.com/Xiaohui-Song/notepad/blob/main/pictures/size20.png)
