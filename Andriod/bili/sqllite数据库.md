## 使用步骤
1. 创建helper，对应上相应数据库：
```java
public class DataBaseHelper extends SQLiteOpenHelper {

    private static final String TAG = "database...";

    public DataBaseHelper(@Nullable Context context) {
        super(context, DataBaseConstants.DATABASE_NAME, null, DataBaseConstants.VERSION_CODE);
    }
}
```
2. 在dao层拿到这个databasehelp对象，用来获取数据库连接，crud(一般通过谷歌api进行)
```java
    public long insert(User user) {
        db = helper.getWritableDatabase();
        ContentValues contentValues = getContentValues(user);
        long result = db.insert(DataBaseConstants.TABLE_NAME, null, contentValues);
        db.close();  // 关闭连接
        return result;
    }
```

## 生命周期
- 每次执行：`db = helper.getWritableDatabase();`，都会看看是否存在这个数据库，如果没有就会创建并执行回调`onCreate()`，如果有就会直接拿到相应连接。
- 每次创建helper的时候需要送入版本号`VERSION_CODE`，执行`db = helper.getWritableDatabase();`的时候如果此时数据库存在，但是版本号增加了（只能增加），会执行回调`onUpgrade`，一般用来添加一些字段啥的。

## 事务
在需要使用事务的时候，在事务开始的时候添加`beginTransion`，结束加上`endTransion`。

## 小问题
- 在执行sql的时候要注意api的使用：`int result = db.update(DataBaseConstants.TABLE_NAME, getContentValues(user), "_id = ?", new String[]{user.getId() + ""});`，记得在查询的参数后面加上？