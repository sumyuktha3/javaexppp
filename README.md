# javaexppp

activity_main.xml
```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/button8"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="delete"
        tools:layout_editor_absoluteX="273dp"
        tools:layout_editor_absoluteY="487dp" />

    <Button
        android:id="@+id/button7"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="update"
        tools:layout_editor_absoluteX="60dp"
        tools:layout_editor_absoluteY="490dp" />

    <Button
        android:id="@+id/button5"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="insert"
        tools:layout_editor_absoluteX="57dp"
        tools:layout_editor_absoluteY="415dp" />

    <EditText
        android:id="@+id/editUserName"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:ems="10"
        android:hint="user name"
        android:inputType="textPersonName"
        tools:layout_editor_absoluteX="112dp"
        tools:layout_editor_absoluteY="82dp" />

    <Button
        android:id="@+id/button6"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Button"
        tools:layout_editor_absoluteX="276dp"
        tools:layout_editor_absoluteY="420dp" />

    <EditText
        android:id="@+id/editUserPassword"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:ems="10"
        android:hint="password"
        android:inputType="textPersonName"
        tools:layout_editor_absoluteX="110dp"
        tools:layout_editor_absoluteY="195dp" />

    <EditText
        android:id="@+id/editUserID"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:ems="10"
        android:hint="user id"
        android:inputType="textPersonName"
        tools:layout_editor_absoluteX="110dp"
        tools:layout_editor_absoluteY="138dp" />
```

MainActivity.java
```
package com.example.java;

import androidx.appcompat.app.AppCompatActivity;

import android.annotation.SuppressLint;
import android.database.Cursor;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.EditText;

public class MainActivity extends AppCompatActivity {
    EditText editUserID;
    EditText editUserName;
    EditText editUserPassword;

    DatabaseManager dbManager;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        editUserID= findViewById(R.id.editUserID);
        editUserName=findViewById(R.id.editUserName);
        editUserPassword=findViewById(R.id.editUserPassword);

        dbManager=new DatabaseManager(this);
        try{
            dbManager.open();
        }
        catch(Exception e){
            e.printStackTrace();
        }
    }
    public void btnInsertPressed(View v){
        dbManager.insert(editUserName.getText().toString(),editUserPassword.getText().toString());
    }

    public void btnFetchPressed(View v){
        Cursor cursor=dbManager.fetch();
        if (cursor.moveToFirst())
        {
            do{
                @SuppressLint("Range") String ID=cursor.getString(cursor.getColumnIndex(DatabaseHelper.USER_ID));
                @SuppressLint("Range") String username=cursor.getString(cursor.getColumnIndex(DatabaseHelper.USER_NAME));
                @SuppressLint("Range") String password=cursor.getString(cursor.getColumnIndex(DatabaseHelper.USER_PASSWORD));
                Log.i("DATABASE_TAG","I have read ID : "+ID+" Username : "+username+" password : "+password);
            }while(cursor.moveToNext());
        }
    }

    public void btnUpdatePressed(View v){
        dbManager.update(Long.parseLong(editUserID.getText().toString()),editUserName.getText().toString(),editUserPassword.getText().toString());
    }

    public void btnDeletePressed(View v){
        dbManager.delete(Long.parseLong(editUserID.getText().toString()));
    }
}
```
DatabaseHelper.java
```
package com.example.java;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;


public class DatabaseHelper extends SQLiteOpenHelper {
    static final String DATABASE_NAME="DataBASE.DB";
    static final int DATABASE_VERSION=1;

    static final String DATABASE_TABLE= "USERS";
    static final String USER_ID= "_ID";
    static final String USER_NAME="user_name";
    static final String USER_PASSWORD="password";

    private static final String CREATE_DB_QUERY = "CREATE TABLE "+DATABASE_TABLE +" ( "+ USER_ID+ " INTEGER PRIMARY KEY AUTOINCREMENT, "+USER_NAME+ " TEXT NOT NULL,   "+ USER_PASSWORD+ " TEXT NOT NULL );";
    public DatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(CREATE_DB_QUERY);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int i, int i1) {
        db.execSQL("DROP TABLE IF EXISTS "+ DATABASE_TABLE);
    }
}
```
DatabaseManager.java
```
package com.example.java;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;

import java.sql.SQLDataException;

public class DatabaseManager {
    private DatabaseHelper dbHelper;
    private Context context;
    private SQLiteDatabase database;

    public DatabaseManager(Context ctx){
        context=ctx;
    }
    public DatabaseManager open() throws SQLDataException {
        dbHelper = new DatabaseHelper(context);
        database = dbHelper.getWritableDatabase();
        return this;
    }
    public void close(){
        dbHelper.close();
    }
    public void insert (String username, String password){
        ContentValues contentValues=new ContentValues();
        contentValues.put(DatabaseHelper.USER_NAME,username);
        contentValues.put(DatabaseHelper.USER_PASSWORD,password);
        database.insert(DatabaseHelper.DATABASE_TABLE,null,contentValues);
    }
    public Cursor fetch(){
        String [] columns= new String[]{DatabaseHelper.USER_ID,DatabaseHelper.USER_NAME,DatabaseHelper.USER_PASSWORD};
        Cursor cursor=database.query(DatabaseHelper.DATABASE_TABLE,columns,null,null,null,null,null);
        if(cursor!=null){
            cursor.moveToFirst();
        }
        return cursor;
    }
    public int update(long _id,String username,String password){
        ContentValues contentValues=new ContentValues();
        contentValues.put(DatabaseHelper.USER_NAME,username);
        contentValues.put(DatabaseHelper.USER_PASSWORD,password);
        int ret = database.update(DatabaseHelper.DATABASE_TABLE,contentValues,DatabaseHelper.USER_ID+"="+_id,null);
        return ret;
    }
    public void delete(long id){
        database.delete(DatabaseHelper.DATABASE_TABLE,DatabaseHelper.USER_ID,null);

    }

}
```
