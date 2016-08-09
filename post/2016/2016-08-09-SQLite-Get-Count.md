SQLite 查询总数
====


* Api 1

```java
Cursor mCount= db.rawQuery("select count(*) from users where uname='" + loginname + "' and pwd='" + loginpass +"'", null);
mCount.moveToFirst();
int count= mCount.getInt(0);
mCount.close();
```

* Api 11

```java
SQLiteDatabase db = getReadableDatabase();
DatabaseUtils.queryNumEntries(db, "users",
                "uname=? AND pwd=?", new String[] {loginname,loginpass});
```

* SQLiteStatement

```java
 SQLiteStatement s = mDb.compileStatement( "select count(*) from users where uname='" + loginname + "' and pwd='" + loginpass + "'; " );

 long count = s.simpleQueryForLong();
```


[来源](http://stackoverflow.com/questions/5202269/sqlite-query-in-android-to-count-rows)