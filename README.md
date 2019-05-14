期中notepad实验介绍

实现的功能如下：

- NotesList中显示条目增加时间显示
- 笔记查询
- UI界面美化
- 背景颜色更换
- 笔记导出
- 笔记创建时间，修改时间，背景颜色排序

功能实现解析

- NotesList中时间戳显示

在NotePad原应用中，笔记列表只显示了笔记的标题。要对它做时间扩展，可以把时间放在标题的下方。<br>

首先，找到列表中item的布局：noteslist_item.xml。<br>

在这个布局文件中，能看到一个TextView，这个便是笔记列表的标题item了。<br>

    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"
    />

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout  xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
        <TextView xmlns:android="http://schemas.android.com/apk/res/android"
            android:id="@android:id/text1"
            android:layout_width="match_parent"
            android:layout_height="?android:attr/listPreferredItemHeight"
            android:textAppearance="?android:attr/textAppearanceLarge"
            android:gravity="center_vertical"
            android:paddingLeft="5dip"
            android:textColor="@color/colorBlack"
            android:singleLine="true"
            />
        <TextView
            android:id="@+id/text1_time"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textAppearance="?android:attr/textAppearanceSmall"
            android:paddingLeft="5dip"
            android:textColor="@color/colorBlack"/>
    </LinearLayout>

数据库结构如下:<br>

    @Override
        public void onCreate(SQLiteDatabase db) {
            db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER"
                + ");");
        }

可以看出，NotePad数据库已经存在时间信息。<br>

如何将数据装填到列表中：NotesList.java 。<br>

可以发现，当前Activity所用到的数据被定义在PROJECTION中：<br>

    private static final String[] PROJECTION = new String[] {
                NotePad.Notes._ID, // 0
                NotePad.Notes.COLUMN_NAME_TITLE, // 1
        };

通过Cursor从数据库中读取出：<br>

    Cursor cursor = managedQuery(
                getIntent().getData(),           
                PROJECTION,                       
                null,                             
                null,                             
                NotePad.Notes.DEFAULT_SORT_ORDER  
            );

通过SimpleCursorAdapter装填：<br>

    String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE } ;
    int[] viewIDs = { android.R.id.text1 };
    SimpleCursorAdapter adapter
        = new SimpleCursorAdapter(
                this, // The Context for the ListView
                R.layout.noteslist_item, // Points to the XML for a list item
                cursor,   // The cursor to get items from
                dataColumns,
                viewIDs
        );
    // Sets the ListView's adapter to be the cursor adapter that was just created.
    setListAdapter(adapter);

     private static final String[] PROJECTION = new String[] {
                NotePad.Notes._ID,
                NotePad.Notes.COLUMN_NAME_TITLE,
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
                NotePad.Notes.COLUMN_NAME_BACK_COLOR, 
        };

Cursor不变，在dataColumns，viewIDs中补充时间部分：<br>

    String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
    int[] viewIDs = { android.R.id.text1 , R.id.text1_time };

    Long now = Long.valueOf(System.currentTimeMillis());
    Date date = new Date(now);
    SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
    String dateTime = format.format(date);

- 笔记查询（按标题查询）

要添加笔记查询功能，就要在应用中增加一个搜索的入口。找到菜单的xml文件，list_options_menu.xml，添加一个搜索的item，搜索图标用安卓自带的图标，设为总是显示：<br>

    <item
        android:id="@+id/menu_search"
        android:title="@string/menu_search"
        android:icon="@android:drawable/ic_search_category_default"
        android:showAsAction="always">
    </item>

在NotesList中找到onOptionsItemSelected方法，在switch中添加搜索的case语句:<br>

     //添加搜素
        case R.id.menu_search:
        Intent intent = new Intent();
        intent.setClass(NotesList.this,NoteSearch.class);
        NotesList.this.startActivity(intent);
        return true;

菜单：<br>

在case语句中写跳转activity代码之前要先写好搜索的activity，新建一个名为NoteSearch的activity。由于搜索出来的也是笔记列表，所以可以模仿NotesList的activity继承ListActivity。在安卓中有个用于搜索控件：SearchView，可以把SearchView跟ListView相结合，动态地显示搜索结果。先布局搜索页面，在layout中新建布局文件note_search_list.xml：<br>

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

要动态地显示搜索结果，就要对SearchView文本变化设置监听，NoteSearch除了要继承ListView外还要实现SearchView.OnQueryTextListener接口：<br>

    public class NoteSearch extends ListActivity  implements SearchView.OnQueryTextListener {
        private static final String[] PROJECTION = new String[] {
                NotePad.Notes._ID, // 0
                NotePad.Notes.COLUMN_NAME_TITLE,
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, 
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

即使我不需要使用onQueryTextSubmit方法，onQueryTextSubmit和onQueryTextChange两个方法也是实现接口必须写的方法。onListItemClick方法是点击NoteList的item跳转到对应笔记编辑界面的方法，NoteList中有这个方法，搜索出来的笔记跳转原理与NotesList中笔记一样，所以可以直接从NotesList中复制过来直接使用。<br>

使用PROJECTION，Cursor，adapter方法与时间显示的原理一样，这里不多描述，但是可以注意到adapter是用MyCursorAdapter声明的，MyCursorAdapter是继承SimpleCursorAdapter，对其中个别方法进行重写，下文UI部分会提到。<br>

动态搜索的实现最主要的部分在onQueryTextChange方法中，在使用这个方法，要先为SearchView注册监听：<br>

    SearchView searchview = (SearchView)findViewById(R.id.search_view);
    searchview.setOnQueryTextListener(NoteSearch.this);  

而onQueryTextChange方法作用是，当SearchView中文本发生变化时，执行其中代码，搜索还有一个重要的部分就是要做到模糊匹配而不是严格匹配，可以使用数据库查询语句中的LIKE和%结合来实现，newText为输入搜索的内容：<br>

    String[] selectionArgs = { "%"+newText+"%" };

最后要在AndroidManifest.xml注册NoteSearch：<br>

    <!--添加搜索activity-->
        <activity
            android:name="NoteSearch"
            android:label="@string/title_notes_search">
        </activity>

- UI美化

先给NotesList换个主题，把黑色换成白色，在AndroidManifest.xml中NotesList的Activity中添加：<br>

    android:theme="@android:style/Theme.Holo.Light"

UI美化主要是让NotesList和NoteSearch每条笔记都有背景色，并且能保存。要做到保存颜色的数据，最直接的办法就是在数据库中添加一个颜色的字段，在这之前在NotePad契约类中添加：<br>

    public static final String COLUMN_NAME_BACK_COLOR = "color";

创建数据库表地方添加颜色的字段：<br>

     @Override
        public void onCreate(SQLiteDatabase db) {
            db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + "   ("
            + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
            + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
            + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
            + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER" //颜色
            + ");");
           }

把颜色定义为INTEGER的主要原因是，在系统中预定于好五种颜色，根据颜色对应不int值选择要显示的颜色，契约类中的定义：<br>

    

由于数据库中多了一个字段，所以要在NotePadProvider中添加对其相应的处理，static{}中：<br>

    sNotesProjectionMap.put(
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
            NotePad.Notes.COLUMN_NAME_BACK_COLOR);

insert中：<br>

     // 新建笔记，背景默认为白色
    if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
            }

将颜色填充到ListView，可以用SimpleCursorAdapter中的getView，bindView，newView方法来实现，我选择了bindView。自定义一个CursorAdapter继承SimpleCursorAdapter，既能完成cursor读取的数据库内容填充到item，又能将颜色填充：<br>

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
                    view.setBackgroundColor(Color.rgb(255, 255, 255));
                    break;
                case NotePad.Notes.YELLOW_COLOR:
                    view.setBackgroundColor(Color.rgb(247, 216, 133));
                    break;
                case NotePad.Notes.BLUE_COLOR:
                    view.setBackgroundColor(Color.rgb(165, 202, 237));
                    break;
                case NotePad.Notes.GREEN_COLOR:
                    view.setBackgroundColor(Color.rgb(161, 214, 174));
                    break;
                case NotePad.Notes.RED_COLOR:
                    view.setBackgroundColor(Color.rgb(244, 149, 133));
                    break;
                default:
                    view.setBackgroundColor(Color.rgb(255, 255, 255));
                    break;
            }
        }
    }

NotesList中的PROJECTION添加颜色项：<br>

    private static final String[] PROJECTION = new String[] {
                NotePad.Notes._ID, 
                NotePad.Notes.COLUMN_NAME_TITLE, 
                //扩展 显示时间 颜色
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
                NotePad.Notes.COLUMN_NAME_BACK_COLOR,
        };

并且将NotesList中用的SimpleCursorAdapter改使用MyCursorAdapter：<br>

    adapter = new MyCursorAdapter(
            this,
            R.layout.noteslist_item,
            cursor,
            dataColumns,
            viewIDs
        );

由于目前为止并没有设置颜色的选项，所以创建的笔记都是白色背景的，但是结合下一个功能，背景更换，让编辑笔记时的背景色跟笔记列表的该笔记背景色同为一种颜色。<br>

白色：<br>

- 背景更换

背景更换指的是编辑笔记时的背景色更换。编辑笔记的Activity为NoteEditor。同样的，在PROJECTION中添加颜色项：<br>

      private static final String[] PROJECTION =
            new String[] {
                NotePad.Notes._ID,
                NotePad.Notes.COLUMN_NAME_TITLE,
                NotePad.Notes.COLUMN_NAME_NOTE,
                NotePad.Notes.COLUMN_NAME_BACK_COLOR
        };

可以注意到，在NoteEditor类中有onResume()方法，onResume()方法在正常启动时会被调用，一般是onStart()后会执行onResume()，在Acitivity从Pause状态转化到Active状态也会被调用，利用这个特点，将从数据库读取颜色并设置编辑界面背景色操作放入其中，好处除了从笔记列表点进来时可以被执行到，跳到改变颜色Activity（接下来会提到）回来时也会被执行到。<br>

    int x = mCursor.getInt(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
        switch (x){
            case NotePad.Notes.DEFAULT_COLOR:
                mText.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
            case NotePad.Notes.YELLOW_COLOR:
                mText.setBackgroundColor(Color.rgb(247, 216, 133));
                break;
            case NotePad.Notes.BLUE_COLOR:
                mText.setBackgroundColor(Color.rgb(165, 202, 237));
                break;
            case NotePad.Notes.GREEN_COLOR:
                mText.setBackgroundColor(Color.rgb(161, 214, 174));
                break;
            case NotePad.Notes.RED_COLOR:
                mText.setBackgroundColor(Color.rgb(244, 149, 133));
                break;
            default:
                mText.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
        }

先在菜单文件中添加一个更改背景的选项，editor_options_menu.xml，图标自己添加，item总是显示：<br>

    <item android:id="@+id/menu_color"
            android:title="@string/menu_color"
            android:icon="@drawable/ic_menu_color"
            android:showAsAction="always"/>

在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加：<br>

    //换背景颜色选项
        case R.id.menu_color:
            changeColor();
            break;

在NoteEditor中添加函数changeColor()：<br>

    //跳转改变颜色的activity，将uri信息传到新的activity
        private final void changeColor() {
            Intent intent = new Intent(null,mUri);
            intent.setClass(NoteEditor.this,NoteColor.class);
            NoteEditor.this.startActivity(intent);
        }

在此之前，要对选择颜色界面进行布局，新建布局note_color.xml，垂直线性布局放置5个ImageButton，并创建NoteColor的Acitvity，用来选择颜色。在AndroidManifest.xml中将这个Acitvity主题定义为对话框样式：<br>

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="horizontal" android:layout_width="match_parent"
        android:layout_height="match_parent">
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
    </LinearLayout>
    

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
            //从NoteEditor传入的uri
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
        //执行顺序在onCreate之后
            if (mCursor != null) {
                mCursor.moveToFirst();
                color = mCursor.getInt(COLUMN_INDEX_TITLE);
            }
            super.onResume();
        }
        @Override
        protected void onPause() {
        //执行顺序在finish()之后，将选择的颜色存入数据库
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
    

    <!--换背景色-->
    <activity android:name="NoteColor"
        android:theme="@android:style/Theme.Holo.Light.Dialog"
        android:label="ChangeColor"
        android:windowSoftInputMode="stateVisible"/>
    

因为选择颜色就会响应对应的函数，而函数先将选择的颜色信息保存在color变量中，调用finish()后会执行onPause()，在onPause()中将颜色存入数据库，Activity从NoteColor回到NoteEditor，NoteEditor被唤醒，会调用NoteEditor的onResume()，onResume()中有读取数据库颜色信息将设置背景的操作，就达到了换背景色的作用，并且也达到了NoteList中笔记颜色更改与编辑背景一致的效果。

修改笔记一的背景色：<br>

- 导出笔记

先在菜单文件中添加一个导出笔记的选项，editor_options_menu.xml：<br>

    <item android:id="@+id/menu_output"
        android:title="@string/menu_output" />
    

在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加：<br>

    //导出笔记选项
       case R.id.menu_output:
            outputNote();
            break;
    

在NoteEditor中添加函数outputNote()：<br>

    //跳转导出笔记的activity，将uri信息传到新的activity
        private final void outputNote() {
            Intent intent = new Intent(null,mUri);
            intent.setClass(NoteEditor.this,OutputText.class);
            NoteEditor.this.startActivity(intent);
        }
    

在此之前，要对选择导出文件界面进行布局，新建布局output_text.xml，垂直线性布局放置EditText和Button，并创建OutputText的Acitvity，用来选择颜色。：<br>

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:paddingLeft="6dip"
        android:paddingRight="6dip"
        android:paddingBottom="3dip">
        <EditText android:id="@+id/output_name"
            android:maxLines="1"
            android:layout_marginTop="2dp"
            android:layout_marginBottom="15dp"
            android:layout_width="wrap_content"
            android:ems="25"
            android:layout_height="wrap_content"
            android:autoText="true"
            android:capitalize="sentences"
            android:scrollHorizontally="true" />
        <Button android:id="@+id/output_ok"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="right"
            android:text="@string/output_ok"
            android:onClick="OutputOk" />
    </LinearLayout>
    

    public class OutputText extends Activity {
       //要使用的数据库中笔记的信息
        private static final String[] PROJECTION = new String[] {
                NotePad.Notes._ID, // 0
                NotePad.Notes.COLUMN_NAME_TITLE, // 1
                NotePad.Notes.COLUMN_NAME_NOTE, // 2
                NotePad.Notes.COLUMN_NAME_CREATE_DATE, // 3
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 4
        };
        //读取出的值放入这些变量
        private String TITLE;
        private String NOTE;
        private String CREATE_DATE;
        private String MODIFICATION_DATE;
        //读取该笔记信息
        private Cursor mCursor;
        //导出文件的名字
        private EditText mName;
        //NoteEditor传入的uri，用于从数据库查出该笔记
        private Uri mUri;
        //关于返回与保存按钮的一个特殊标记，返回的话不执行导出，点击按钮才导出
        private boolean flag = false;
        private static final int COLUMN_INDEX_TITLE = 1;
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.output_text);
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
            //从mCursor读取对应值
                TITLE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE));
                NOTE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE));
                CREATE_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_CREATE_DATE));
                MODIFICATION_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE));
                //flag在点击导出按钮时会设置为true，执行写文件
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
    

在AndroidManifest.xml中将这个Acitvity主题定义为对话框样式，并且加入权限：<br>

    <!--添加导出activity-->
            <activity android:name="OutputText"
                android:label="@string/output_name"
                android:theme="@android:style/Theme.Holo.Dialog"
                android:windowSoftInputMode="stateVisible">
            </activity>
    

     <!-- 在SD卡中创建与删除文件权限 -->
        <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
        <!-- 向SD卡写入数据权限 -->
        <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    

- 笔记排序

笔记排序相对来说简单，只要把Cursor的排序参数变换下就可以了。在菜单文件list_options_menu.xml中添加：<br>

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
    

在NotesList菜单switch下添加case：<br>

    //创建时间排序
        case R.id.menu_sort1:
            cursor = managedQuery(
                    getIntent().getData(),            
                    PROJECTION,                      
                    null,                          
                    null,                          
                    NotePad.Notes._ID 
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
                    getIntent().getData(),          
                    PROJECTION,                      
                    null,                            
                    null,                       
                    NotePad.Notes.DEFAULT_SORT_ORDER 
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
                    getIntent().getData(),
                    PROJECTION,      
                    null,       
                    null,       
                    NotePad.Notes.COLUMN_NAME_BACK_COLOR
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
    

因为排序会多次使用到cursor，adapter，所以将adapter,cursor,dataColumns,viewIDs定义在函数外类内：<br>、

具体的实验效果可以查看实验截图文件夹中的截图<br>  
