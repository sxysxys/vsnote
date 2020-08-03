## 概念
内容提供者，就是向第三方暴露自己的数据库的！目前来说，只有google的短信/电话是这么做的，其他应用基本上不会这么做！所以呢，使用场景也少了！比如说微信/支付宝，在第一次使用的时候会询问你是否同意让它读取你的联系人/通讯录，就是通过内容提供者来读取的。
https://www.sunofbeach.net/a/1186839898213076992

## 提供端
1. 根据安卓通讯录的源码，一般内容提供者的代码模板是这样的：
```java xml
public class UserProvider extends ContentProvider {

    private static UriMatcher mUriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
    private static final int DATABASEDEMO_CODE = 1;

    static {
        mUriMatcher.addURI("com.shen.databasedemo","user",DATABASEDEMO_CODE); // 一般第一个是请求的路径，一般填当前包名，后面一般填想请求的数据表。将这个路径暴露给第三方，第三方直接拿到这俩路径发起请求即可
    }

    private DataBaseHelper dataBaseHelper;

    @Override
    public boolean onCreate() {
        dataBaseHelper = new DataBaseHelper(getContext());
        return false;
    }

    @Nullable
    @Override
    public Cursor query(@NonNull Uri uri, @Nullable String[] strings, @Nullable String s, @Nullable String[] strings1, @Nullable String s1) {
        int match = mUriMatcher.match(uri);
        switch (match) {
            case DATABASEDEMO_CODE:
                SQLiteDatabase readableDatabase = dataBaseHelper.getReadableDatabase();
                return readableDatabase.query(DataBaseConstants.TABLE_NAME,strings, s, strings1, s1, null, null);
        }
        return null;
    }
```
2. 在manifest中注册：
```xml
        <provider
            android:exported="true"
            android:authorities="com.shen.databasedemo"
            android:name=".provider.UserProvider"/>
```
## 第三方请求端
```java
        ContentResolver resolver = this.getContentResolver();
        Uri uri = Uri.parse("content://com.shen.databasedemo/user");
//        Uri uri = Uri.parse(AUTHORITY_URI);
        Cursor cursor = resolver.query(uri, null, null, null, null);
```
## 请求系统功能
第三方应用想获取用户当前通讯录、短信等，可以通过内容提供者获取（因为这些东西本质也是存在系统应用的数据库中的），需要拿到上面说的那俩路径（一般通过百度或者查看源码），然后再manifest中添加`<user-permission>`请求相应权限。

