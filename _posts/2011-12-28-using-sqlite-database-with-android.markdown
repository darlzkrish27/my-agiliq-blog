---
layout: post
title:  "Using SQLite Database with Android"
date:   2011-12-28 11:30:00+05:30
tags: java
author: balu
---
Android embeds an Open Source Database called SQLite, which supports standard relational database features like SQL syntax, transactions and prepared statements. In addition it requires only little memory at runtime (approx. 250 KB). In this post I would like to show how to work with a simple pragmatic example : Baby Names App.

Before writing the code, let me show you some of the screen-shots of Baby Names App, which requires interaction with the database.

1. When ever you launch the App, it will shows a Menu of items as below.
   ![Baby Names App Menu](/dumps/20111229/baby_names_menu.png)

2. When we click on the highlighted button - "Common Names", it will displays a list of all the names present in "common_names" which is a SQLite database table we will create for this app. Here is the screen-shot of the emulator after clicking on the Button.
   ![List of Common Names](/dumps/20111229/baby_names_cn_list.png)

3. Now, if we click on a name, let us be the first one "andrew" in the above screen-shot, we have to get some details about the name. In this case will get details about "andrew" as below.
   ![Description of the Person](/dumps/20111229/baby_names_cn_desc.png)

---------------------------
Let us write the Java Code
---------------------------
Create a SQLiteDatabase with name **baby_names_database**.

To create the database create a class with name **BabyNamesDBHelper** which extends **SQLiteOpenHelper** and then call the super class constructor, passing database name and version as arguments. *SQLiteOpenHelper* is a helper class to manage database creation and version management. Here is the code to implement this.

    public class BabyNamesDBHelper extends SQLiteOpenHelper {

	public static final String DATABASE_NAME = "baby_names_database";
	public static final int DATABASE_VERSION = 1;

	public static final String DATABASE_TABLE_1 = "common_names";

	// Columns present in DATABASE_TABLE
	public static final String COMMON_NAME_ROWID = "_id";
	public static final String COMMON_NAME = "common_name";
	public static final String COMMON_NAME_COUNT = "common_name_count";

	// SQL query string for creating DATABASE_TABLE
	static final String CREATE_DATABASE_TABLE_1 =
			"create table " + DATABASE_TABLE_1 + " (" + COMMON_NAME_ROWID + 
			" integer primary key autoincrement, " + COMMON_NAME_COUNT +
			" text not null, " + COMMON_NAME + " text not null);";

	// Constructor
	public BabyNamesDBHelper(Context context) {
		super(context, DATABASE_NAME, null, DATABASE_VERSION);
		this.context = context;
	}



Now we have to create a table - **common_names** in "baby_names_database". For that we need to execute the SQL command for creating the table using SQLiteDatabase.execSQL() in onCreate(). Here is the code for this.

        static final String CREATE_DATABASE_TABLE_1 =
			"create table " + DATABASE_TABLE_1 + " (" + COMMON_NAME_ROWID + 
			" integer primary key autoincrement, " + COMMON_NAME_COUNT +
			" text not null, " + COMMON_NAME + " text not null);";
        @Override
        public void onCreate(SQLiteDatabase db) {
        	// Creating Table
        	db.execSQL(CREATE_DATABASE_TABLE_1);
        }



We can insert data into database using

        ContentValues initialValues = new ContentValues();
        
        initialValues.put(COMMON_NAME_COUNT, '424516');
        initialValues.put(COMMON_NAME, 'andrew');
        
        db.insert(DATABASE_TABLE_1, null, initialValues);

 

Now create a class with name CommonNamesAdapter where we will define the necessary functions that are required to interact with common_names table in the database. Some important implemented functions are...

    * open
    * close
    * fetchAllCommonNames

To view the code please [click here](https://github.com/Balu-Varanasi/BabyNamesApp/blob/master/src/balu/android/database/CommonNamesAdapter.java)


When ever we click on Common Names button in the first screen-shot, Activity in the second screen-shot will be launched. Here is the code for that.


    public class CommonNames extends ListActivity {

	CommonNamesAdapter cnTable;
	ListView cnListView;
	Cursor c;

	private static final int COMMON_NAME_ACTIVITY_START = 1;

	@Override
	protected void onCreate(Bundle savedInstanceState) {

		super.onCreate(savedInstanceState);
		setContentView(R.layout.common_names_list);

		cnTable = new CommonNamesAdapter();
		cnTable.open(getApplicationContext());

		c = cnTable.fetchAllCommonNames();
		startManagingCursor(c);

		if(c!=null){
			SimpleCursorAdapter adapter = new SimpleCursorAdapter(this,
					R.layout.common_names_row,
					c,
					new String[] {c.getColumnName(1)},
					new int[] {R.id.commonName});
			setListAdapter(adapter);		
		}
	}


Now, we have to implement onListItemClickListner() in CommonNames Activity to handle click events on the menu item. Here is the code...


	@Override
	protected void onListItemClick(ListView l, View v, int position, long id) {
		super.onListItemClick(l, v, position, id);

		c.moveToPosition(position);

		Intent i = new Intent(this, CommonNameDescription.class);
		i.putExtra(CommonNamesAdapter.COMMON_NAME_ROWID, id);
		i.putExtra(CommonNamesAdapter.COMMON_NAME, c.getString(
				c.getColumnIndexOrThrow(CommonNamesAdapter.COMMON_NAME)));
		i.putExtra(CommonNamesAdapter.COMMON_NAME_COUNT, c.getString(
				c.getColumnIndexOrThrow(CommonNamesAdapter.COMMON_NAME_COUNT)));
		startActivityForResult(i, COMMON_NAME_ACTIVITY_START);
	}



After clicking on a List Item, a Activity in the third screen-shot will be displayed. [Click here](https://github.com/Balu-Varanasi/BabyNamesApp/blob/master/src/balu/android/CommonNameDescription.java) to see the code for that.


This is how we can use android SQLite API to create and interact with databases.

* [Click here to view the Complete Source which is in github](https://github.com/Balu-Varanasi/BabyNamesApp/tree/master/src/balu/android)

