---
layout: post
title:  Sqlite升级时向已有表中增加字段
date:   2018-09-10 10:01:51 +0800
categories: Sqlite
tag: [Sqlite, Sqlite升级, alter]
---

* content
{:toc}



### Sqlite升级时向已有表中增加字段

Sqlite数据库升级时，我们经常会遇到给已有表中增加字段的操作。一般来说，**给已有表中增加字段**是数据库操作中的基操，没必要再专门写篇blog记录的，但是sqlite对SQL语句支持的不够彻底，比方说这次我们用到的"ALTER TABLE"命令。官方介绍的第一句如下所示：

```javascript
SQLite supports a limited subset of ALTER TABLE. The ALTER TABLE command in SQLite allows the user to rename a table or to add a new column to an existing table.
```

Sqlite支持“ALTER TABLE”的有限子集，在Sqlite中这个命令允许用户重命名表、向已有表中添加新列。

那么如果我们想在一行SQL语句中同时添加多个字段，这个是不能直接实现的，具体原因看末尾的参考链接[2. sqlite alter table add MULTIPLE columns in a single statemen there](https://stackoverflow.com/questions/6172815/sqlite-alter-table-add-multiple-columns-in-a-single-statement)

既然不能一次添加多个，那么我们一次添加一个总行吧，这里给出一个实现方式，针对添加多个column的需求，这里对应的创建多条sql语句，在升级时依次执行：

```javascript
// sqlite 不支持一次增加多列，只能一次增加一列。
    val sqlsV2 = arrayOf("alter table ${TABLE_NAME} add column age VARCHAR(255)",
            "alter table ${TABLE_NAME} add column gender VARCHAR(255)")
```

依次执行数组中的sql语句：

```javascript
override fun onUpgrade(db: SQLiteDatabase?, oldVersion: Int, newVersion: Int) {
        Log.e(TAG, "onUpgrade oldVersion:$oldVersion newVersion:$newVersion")
        // 1 升级到 2
        if (oldVersion < 2) {
            Log.e(TAG, "onUpgrade 1~2")
            sqlsV2.forEach {
                db?.execSQL(it)
            }
        }
    }
```


参考：

[1. 菜鸟教程](http://www.runoob.com/sqlite/sqlite-syntax.html)

[2. sqlite alter table add MULTIPLE columns in a single statemen there](https://stackoverflow.com/questions/6172815/sqlite-alter-table-add-multiple-columns-in-a-single-statement)

[3. sqlite官网——Alter Table](https://www.sqlite.org/lang_altertable.html)

