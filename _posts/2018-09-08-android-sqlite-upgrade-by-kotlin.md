---
layout: post
title:  Android Sqlite数据库升级——kotlin
date:   2018-09-08 20:56:14 +0800
categories: Sqlite
tag: [Sqlite, kotlin]
---

* content
{:toc}



### Android Sqlite数据库升级——kotlin
对于android开发同学来说，数据库相关操作是我们日常操作之一，相应的数据库的升级操作就必不可少了。这里我总结下数据库升级时需要注意的事项。

Android中数据库操作的核心类是SqliteOpenHelper，这个类有两个方法，onCreate和onUpgrade。onCreate方法只会调用一次，onUpgrade方法会在版本号增加之后触发。

数据库升级的操作过程中，我们需要处理的情况有两种，第一种是数据库从低版本升级上来的，第二种是新安装app的用户。

每次数据库需要升级时，我们需要将新版本的修改同步到两个地方：
一个是onCreate方法中，这里确保新安装app的用户可以使用到最新的数据库，所以这里创建数据库的语句应该是可以直接创建最新版本数据库的语句。
另一个是onUpgrade方法中，在这里我们需要确保低版本用户在升级到最新版时，能够将数据库更新到最新，这里我们就需要将各个版本间的差异用代码体现出来。

代码如下：

```javascript
// 表的名字
const val sqlite_name = "MySqliteHelper.db"

internal class MySqliteHelper(context: Context, version: Int) : SQLiteOpenHelper(context, sqlite_name, null, version) {
    val TAG = "MySqliteHelper"

    //  创建语句
    val sqlCreate = "create table Test (" +
            "id integer primary key autoincrement, " +
            "author text, " +
            "price real, " +
            "pages integer, " +
            "name text, " +
            "ver2 text, " +
            "ver3 text, " +
            "ver4 text, " +
            "ver5 text)"

    override fun onCreate(db: SQLiteDatabase?) {
        Log.e(TAG, "onCreate")
        db?.execSQL(sqlCreate)
    }

    override fun onUpgrade(db: SQLiteDatabase?, oldVersion: Int, newVersion: Int) {
        Log.e(TAG, "onUpgrade oldVersion:$oldVersion newVersion:$newVersion")
        // 1 升级到 2
        if (oldVersion < 2) {
            Log.e(TAG, "onUpgrade 1~2")
        }

        // 2 升级到 3
        if (oldVersion < 3) {
            Log.e(TAG, "onUpgrade 2~3")
        }

        // 3 升级到 4
        if (oldVersion < 4) {
            Log.e(TAG, "onUpgrade 3~4")
        }

        // 4 升级到 5
        if (oldVersion < 5) {
            Log.e(TAG, "onUpgrade 4~5")
        }
    }


}
```

2、使用SqliteOpenHelper的子类创建数据库：

```javascript
class SqliteUpgradeActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_sqlite_upgrade)

        var sqliteHelper = MySqliteHelper(this, 1);
        var db = sqliteHelper.writableDatabase

        db.execSQL("insert into Test (name, author, pages, price) values(?, ?, ?, ?)",
                arrayOf("tiny's book", "tiny", "600", "20.9"))
        db.execSQL("insert into Test (name, author, pages, price) values(?, ?, ?, ?)",
                arrayOf("tongtong", "tong", "250", "9.99"))


    }
}
```

3、这里我们先设置数据库的版本为1，然后看下执行结果：
log中有如下输出：表明我们的数据库创建好了。

```javascript
E/MySqliteHelper: onCreate
```

然后我们查看下创建好的数据库，如下所示，可以看到数据库中的数据字段确实如我们期望的那样。

![onCreate方法执行之后的数据库](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/WX20180908-203553%402x.png)

5、修改版本号，模拟从低版本升级到高版本：
这里我们直接将version字段的值修改为5，代表着用户是从第1版数据库的app直接升级到第5版的数据库。这种情况下，onUpgrade方法中的所有代码都会顺序执行，我们运行代码验证下：

```javascript
E/MySqliteHelper: onUpgrade oldVersion:1 newVersion:5
    onUpgrade 1~2
    onUpgrade 2~3
    onUpgrade 3~4
    onUpgrade 4~5
```

可以看到，输出的Log跟我们期望的完全相同。

总结：
使用SqliteOpenHelper进行数据库升级操作时，onCreate方法中需要时刻保持最新的业务语句，因为这个方法只有新安装的用户才会执行。另外，在onUpgrade方法中，需要保留每次升级时新版本与上版本的差异。

