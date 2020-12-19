# 期中实验

笔记本应用的修改<br>

---
## 一、两个基本功能
### 1、增加时间戳
![]()<br>
在NotePad应用中，找到显示标题的页面：noteslist_item.xml<br>
将显示标题的TextView放入垂直线性布局中<br>
再添加一个TextView显示时间,如下<br>
```
<LinearLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <TextView
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:layout_marginLeft="5dip"
        android:textColor="#000"
        android:singleLine="true"
        />
    <!--添加显示时间的TextView-->
    <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingLeft="5dip"
        android:textColor="#000"/>
</LinearLayout>
```

在NoteList.java中修改PROJECTION<br>
```
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, //2
            NotePad.Notes.COLUMN_NAME_BACK_COLOR, //3
    };
```
在dataColumns，viewIDs中添加时间部分：<br>
```
        // The names of the cursor columns to display in the view, initialized to the title column
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;

        // The view IDs that will display the cursor columns, initialized to the TextView in
        // noteslist_item.xml
        int[] viewIDs = { android.R.id.text1 , R.id.text2};
```
最后在NotePadProvider.java中的insert方法和NoteEditor.java中的updateNote方法中添加时间转换<br>
```
        Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        SimpleDateFormat format = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");
        format.setTimeZone(TimeZone.getTimeZone("GMT+08:00"));
        String dateTime = format.format(date);
```
效果图如下<br>
![](https://github.com/yuheng0001/NotePad-master/blob/master/image/1.png)<br>

### 2、添加搜索功能
找到list_options_menu.xml<br>
添加一个搜索的item<br>
搜索图标用安卓自带的图标，设为总是显示<br>
```
    <item
        android:id="@+id/menu_search"
        android:title="@string/menu_search"
        android:icon="@android:drawable/ic_search_category_default"
        android:showAsAction="always">
    </item>
```
在NotesList.java中找到onOptionsItemSelected方法，在switch中添加搜索的case语句<br>
```
            case R.id.menu_search:
                Intent intent = new Intent();
                intent.setClass(NotesList.this,NoteSearch.class);
                NotesList.this.startActivity(intent);
                return true;
```
新建NoteSearch.java和note_search.xml文件<br>
note_search.xml的布局如下<br>
```
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
NoteSearch.java要对SearchView文本变化设置监听<br>
```
public class NoteSearch extends ListActivity  implements SearchView.OnQueryTextListener {
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
            NotePad.Notes.COLUMN_NAME_BACK_COLOR  //3
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        SearchView searchview = (SearchView)findViewById(R.id.search_view);
        //为查询文本框注册监听器
        searchview.setOnQueryTextListener(NoteSearch.this);
    }
    @Override
    public boolean onQueryTextSubmit(String query) {
        return false;
    }
    @Override
    public boolean onQueryTextChange(String s) {
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";
        String[] selectionArgs = { "%"+s+"%" };
        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                selection,
                selectionArgs,
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text2 };
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
onListItemClick方法和之前一样<br>
这边的MyCursorAdapter继承SimpleCursorAdapter，因为下面彩色背景要用到。<br>

效果图如下<br>

![](https://github.com/yuheng0001/NotePad-master/blob/master/image/2.png)<br>
![](https://github.com/yuheng0001/NotePad-master/blob/master/image/3.png)<br>


## 二、三个扩展功能
---
### 1、UI美化：添加彩色背景
在NotePad.java中添加背景颜色字段和颜色类型<br>
```
        public static final String COLUMN_NAME_BACK_COLOR = "color";
        
        public static final int DEFAULT_COLOR = 0; //白
        public static final int YELLOW_COLOR = 1; //黄
        public static final int BLUE_COLOR = 2; //蓝
        public static final int GREEN_COLOR = 3; //绿
        public static final int RED_COLOR = 4; //红
```
在创建数据库表地方添加颜色的字段<br>
在NotePadProvider.java中添加相应的设置<br>
```
        //新建时默认颜色
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
        }

```
**添加新字段时要将程序卸载重装，否则会有一些错误**  超级重要！！！<br>
在NotePadProvider.java中的insert方法中设置新建背景为白色<br>

在NotesList.java中的PROJECTION添加颜色项,并且将NotesList.java中用的SimpleCursorAdapter改使用MyCursorAdapter<br>
MyCursorAdapter.java的内容如下<br>
```
public class MyCursorAdapter extends SimpleCursorAdapter {


    public MyCursorAdapter(Context context, int layout, Cursor c,
                           String[] from, int[] to) {
        super(context, layout, c, from, to);
    }

    @Override
    public void bindView(View view, Context context, Cursor cursor){
        super.bindView(view, context, cursor);
        int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
        switch (x){
            case NotePad.Notes.DEFAULT_COLOR:
                view.setBackgroundColor(Color.rgb(255,255,255));
                break;
            case NotePad.Notes.YELLOW_COLOR:
                view.setBackgroundColor(Color.rgb(250,250,210));
                break;
            case NotePad.Notes.BLUE_COLOR:
                view.setBackgroundColor(Color.rgb(135,206,250));
                break;
            case NotePad.Notes.GREEN_COLOR:
                view.setBackgroundColor(Color.rgb(50,205,50));
                break;
            case NotePad.Notes.RED_COLOR:
                view.setBackgroundColor(Color.rgb(240,128,128));
                break;
            default:
                view.setBackgroundColor(Color.rgb(255,255,255));
                break;
        }
    }
}
```
还要在添加一个背景更换的菜单<br>
在编辑页面的菜单文件下添加更换背景选项<br>
在NoteEditor.java中的onOptionsItemSelected方法中添加跳转NoteColor.java<br>
NoteColor.java如下<br>
```
public class NoteColor extends Activity {
    private Cursor mCursor;
    private Uri mUri;
    private int color;
    private static final int COLUMN_INDEX_TITLE = 1;
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,//2
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
}
```
在选择完颜色就会响应对应的函数，将选择的颜色信息保存在color变量中，然后会执行onPause()，在onPause()中将颜色存入数据库，<br>
Activity从NoteColor回到NoteEditor，NoteEditor被唤醒，会调用NoteEditor的onResume()，onResume()会读取数据库背景颜色值并设置背景<br>

效果图如下<br>
![](https://github.com/yuheng0001/NotePad-master/blob/master/image/4.png)<br>
![](https://github.com/yuheng0001/NotePad-master/blob/master/image/5.png)<br>

### 2、笔记排序

在菜单文件list_options_menu.xml中添加选项<br>
```
    <item
        android:id="@+id/menu_sort"
        android:title="@string/menu_sort"
        android:icon="@android:drawable/ic_menu_sort_by_size"
        android:showAsAction="always" >
        <menu>
            <item
                android:id="@+id/menu_sorttime"
                android:title="@string/menu_sort1"/>
            <item
                android:id="@+id/menu_sorttitle"
                android:title="@string/menu_sort2"/>
            <item
                android:id="@+id/menu_sortcolor"
                android:title="@string/menu_sort3"/>
        </menu>
    </item>
```
在NOteList.java的onOptionsItemSelected方法中添加响应<br>
```
            //按时间排序
            case R.id.menu_sorttime:
                cursor = managedQuery(
                        getIntent().getData(),            // Use the default content URI for the provider.
                        PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                        null,                             // No where clause, return all records.
                        null,                             // No where clause, therefore no where column values.
                        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE  // Use the default sort order.
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

            //按标题排序
            case R.id.menu_sorttitle:
                cursor = managedQuery(
                        getIntent().getData(),            // Use the default content URI for the provider.
                        PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                        null,                             // No where clause, return all records.
                        null,                             // No where clause, therefore no where column values.
                        NotePad.Notes.COLUMN_NAME_TITLE // Use the default sort order.
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

            //按颜色排序
            case R.id.menu_sortcolor:
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
按时间排序如下<br>
![](https://github.com/yuheng0001/NotePad-master/blob/master/image/time.png)<br>
按标题排序如下<br>
![](https://github.com/yuheng0001/NotePad-master/blob/master/image/title.png)<br>
按颜色排序如下<br>
![](https://github.com/yuheng0001/NotePad-master/blob/master/image/color.png)<br>

### 3、导出笔记到本地

同上，先在编辑页面菜单（editor_optins_menu.xml）里添加选项<br>
并在NoteEditor.java的onOptionsItemSelected()方法添加响应，跳转到ExportText.java<br>
ExportText.java如下<br>
```
public class ExportText extends Activity {
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_NOTE, // 2
            NotePad.Notes.COLUMN_NAME_CREATE_DATE, // 3
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 4
            NotePad.Notes.COLUMN_NAME_BACK_COLOR, //5
    };

    private String TITLE;
    private String NOTE;
    private String CREATE_DATE;
    private String MODIFICATION_DATE;
    private Cursor mCursor;
    private EditText mName;
    private Uri mUri;
    private boolean flag = false;

    private static final int COLUMN_INDEX_TITLE = 1;

    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.export_text);
        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,        // The URI for the note that is to be retrieved.
                PROJECTION,  // The columns to retrieve
                null,        // No selection criteria are used, so no where columns are needed.
                null,        // No where columns are used, so no where values are needed.
                null         // No sort order is needed.
        );
        mName = (EditText) findViewById(R.id.output_name);
    }
    @Override
    protected void onResume(){
        super.onResume();
        if (mCursor != null) {
            // The Cursor was just retrieved, so its index is set to one record *before* the first
            // record retrieved. This moves it to the first record.
            mCursor.moveToFirst();
            //编辑框默认的文件名为标题，可自行更改
            mName.setText(mCursor.getString(COLUMN_INDEX_TITLE));
        }
    }
    @Override
    protected void onPause() {
        super.onPause();
        if (mCursor != null) {
            TITLE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE));
            NOTE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE));
            CREATE_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_CREATE_DATE));
            MODIFICATION_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE));
            if (flag == true) {
                write();
            }
            flag = false;
        }
    }
    public void OutputOk(View v){
        flag = true;
        finish();
    }
    private void write()
    {
        try
        {
            // 如果手机插入了SD卡，而且应用程序具有访问SD的权限
            if (Environment.getExternalStorageState().equals(
                    Environment.MEDIA_MOUNTED)) {
                // 获取SD卡的目录
                File sdCardDir = Environment.getExternalStorageDirectory();
                //创建文件目录
                File targetFile = new File(sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt");
                //写文件
                PrintWriter ps = new PrintWriter(new OutputStreamWriter(new FileOutputStream(targetFile), "UTF-8"));
                ps.println(TITLE);
                ps.println(NOTE);
                ps.println("创建时间：" + CREATE_DATE);
                ps.println("最后一次修改时间：" + MODIFICATION_DATE);
                ps.close();
                Toast.makeText(this, "保存成功,保存位置：" + sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt", Toast.LENGTH_LONG).show();
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```
在AndroidManifest.xml中将ExportText定义为对话框样式<br>
```
        <!-- 导出activity -->
        <activity android:name=".ExportText"
            android:label="@string/Export_name"
            android:theme="@android:style/Theme.Holo.Dialog"
            android:windowSoftInputMode="stateVisible">
        </activity>
```
存储权限的申请<br>
```
    <!--权限申请-->
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"
        tools:ignore="ProtectedPermissions" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
添加一个动态获取权限，在第一次运行应用时执行<br>
在NoteList.java中添加字段和checkPermission方法<br>
```
    String[] permissions = new String[]{Manifest.permission.MOUNT_UNMOUNT_FILESYSTEMS,
            Manifest.permission.WRITE_EXTERNAL_STORAGE};
    List<String> mPermissionList = new ArrayList<>();

    private static final int PERMISSION_REQUEST = 1;
```

```
    private void checkPermission() {
        mPermissionList.clear();
        //判断哪些权限未授予
        for (int i = 0; i < permissions.length; i++) {
            if (ContextCompat.checkSelfPermission(this, permissions[i]) != PackageManager.PERMISSION_GRANTED) {
                mPermissionList.add(permissions[i]);
            }
        }

        if (mPermissionList.isEmpty()) {//未授予的权限为空，表示都授予了
        } else {//请求权限方法
            String[] permissions = mPermissionList.toArray(new String[mPermissionList.size()]);//将List转为数组
            ActivityCompat.requestPermissions(NotesList.this, permissions, PERMISSION_REQUEST);
        }
    }
    
    //这里不管用户是否拒绝，都进入首页，不再重复申请权限
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode) {
            case PERMISSION_REQUEST:
                break;
            default:
                super.onRequestPermissionsResult(requestCode, permissions, grantResults);
                break;
        }
    }
```
在NoteList.java的onCreate方法的最上方应用checkPermission方法<br>
```
  protected void onCreate(Bundle savedInstanceState) {
        checkPermission();
        super.onCreate(savedInstanceState);

```
权限申请效果图如下<br>

![](https://github.com/yuheng0001/NotePad-master/blob/master/image/9.png)<br>

导出笔记效果图如下<br>
![](https://github.com/yuheng0001/NotePad-master/blob/master/image/6.png)<br>
![](https://github.com/yuheng0001/NotePad-master/blob/master/image/7.png)<br>
![](https://github.com/yuheng0001/NotePad-master/blob/master/image/8.png)<br>
