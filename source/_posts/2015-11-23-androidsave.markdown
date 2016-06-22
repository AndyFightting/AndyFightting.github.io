---
layout: post
title: "Android基本存储"
date: 2015-11-23 19:48:52 +0800
comments: true
categories: android
---
Android数据保存方式有： 文件存储, SharedPreference, Sqlite, SD卡存储。

### * 文件存储

不对存储数据做任何格式化处理，原封不动的保存到文件中。使用于保存简单的文本数据或者二进制数据。

Context有个`openFileOutput()`方法接受两个参数，第一个：文件名，**不要包含路径**，第二个：操作方式，有两种，`MODE_PRIVATE`(覆盖原文件内容),`MODE_APPEND`(在原文件尾部追加内容)，默认第一种。文件默认保存在：`/data/data/packageName/files/`这个目录里。

	 public static void saveText(Context context,String fileName,String text){
        FileOutputStream outputStream ;
        BufferedWriter bufferedWriter = null;
        try {
           outputStream = context.openFileOutput(fileName,Context.MODE_PRIVATE);
            bufferedWriter = new BufferedWriter(new OutputStreamWriter(outputStream));
            bufferedWriter.write(text);
        }catch (IOException e){
            e.printStackTrace();
        }finally {
            try {
                if (bufferedWriter != null){
                    bufferedWriter.close();
                }
            }catch (IOException e){
                e.printStackTrace();
            }
        }
    }
<!--more-->
    
在我调用 `MyTool.saveText(this,"myFile","Hello World!");`后出现了如下文件，点画圈圈的地方可以导出到电脑上打开查看。
![img](/myimg/android/helloSave.png)

读文件

	public static String getText(Context context,String fileName){
        FileInputStream inputStream;
        BufferedReader bufferReader = null;
        StringBuilder contentBuilder = new StringBuilder();
        try {
            inputStream = context.openFileInput(fileName);
            bufferReader = new BufferedReader(new InputStreamReader(inputStream));
            String lineText = "";
            while ((lineText = bufferReader.readLine()) != null){
                contentBuilder.append(lineText);
            }
        }catch (IOException e){
            e.printStackTrace();
        }finally {
           if (bufferReader != null){
               try {
                   bufferReader.close();
               }catch (IOException e){
                   e.printStackTrace();
               }
           }
        }
        return contentBuilder.toString();
    }

删除文件

	public static void deleteFile(Context context, String fileName) {
        String filePath = context.getFilesDir()+"/"+fileName;
        File file = new File(filePath);
        if (file.exists()){
            file.delete();
        }
    }


### * SharedPreference

用key-value键值对的方式保存数据，适用于保存应用配置之类的简单信息。有以下三种方式得到`SharedPreferences`对象，得到实例后调用实例的`edit()`方法得到`SharedPreferences.Editor`对象，用该`Editor`对象`putBoolean("key",value)``putString("key",value)`等方法添加数据，然后调用`apply()`即可。获取数据就调用对应的`getBoolean("key")``getString("key")`。文件保存在`/data/data/packageName/shared_prefs/`目录下。

1. Context中的`getSharedPreferences()`方法接收两个参数，一个文件名(不要包含路径)，一个操作模式，有`MODE_PRIVATE`(覆盖原文件),`MODE_MULTI_PROCESS`(多个进程对文件进行读写情况)默认第一种。

2. Activity中也有个一个类似的方法`getSharedPreferences()`，它只接收一个操作模式作为参数，**自动将类名作为文件名**。

3. PreferenceManager类中的`getDefaultSharedPreferences()`，这是一个静态方法，接收一个参数`Context`,自动将应用程序的`packageName`为前缀的文件名,如我的包名是`com.suguiming.myandroid`,生成的文件名就是`com.suguiming.myandroid_preferences.xml`。

```
	SharedPreferences sharedPreferences = 	PreferenceManager.getDefaultSharedPreferences(context);
      SharedPreferences.Editor editor = sharedPreferences.edit();
        editor.putString("name","guimingsu");
        editor.putInt("age", 18);
        editor.apply();
```
![img](/myimg/android/shareP.png)
![img](/myimg/android/shareV.png)

一般都把它做成一个工具类

    public class SharedPreUtil {

    public static final String FILE_NAME = "shared_preference";

    public static void put(Context context, String key, Object value) {
        SharedPreferences sharedPreferences = context.getSharedPreferences(FILE_NAME, Context.MODE_PRIVATE);
        SharedPreferences.Editor editor = sharedPreferences.edit();

        if (value instanceof String) {
            editor.putString(key, (String) value);
        } else if (value instanceof Integer) {
            editor.putInt(key, (Integer) value);
        } else if (value instanceof Boolean) {
            editor.putBoolean(key, (Boolean) value);
        } else if (value instanceof Float) {
            editor.putFloat(key, (Float) value);
        } else if (value instanceof Long) {
            editor.putLong(key, (Long) value);
        } else {
            editor.putString(key, value.toString());
        }
        editor.apply();
    }

    public static Object get(Context context, String key, Object defaultValue) {
        SharedPreferences sharedPreferences = context.getSharedPreferences(FILE_NAME, Context.MODE_PRIVATE);

        if (defaultValue instanceof String) {
            return sharedPreferences.getString(key, (String) defaultValue);
        } else if (defaultValue instanceof Integer) {
            return sharedPreferences.getInt(key, (Integer) defaultValue);
        } else if (defaultValue instanceof Boolean) {
            return sharedPreferences.getBoolean(key, (Boolean) defaultValue);
        } else if (defaultValue instanceof Float) {
            return sharedPreferences.getFloat(key, (Float) defaultValue);
        } else if (defaultValue instanceof Long) {
            return sharedPreferences.getLong(key, (Long) defaultValue);
        }
        return null;
    }

    public static void remove(Context context, String key) {
        SharedPreferences sharedPreferences = context.getSharedPreferences(FILE_NAME, Context.MODE_PRIVATE);
        SharedPreferences.Editor editor = sharedPreferences.edit();
        editor.remove(key);
        editor.apply();
    }

    public static void removeAll(Context context) {
        SharedPreferences sharedPreferences = context.getSharedPreferences(FILE_NAME, Context.MODE_PRIVATE);
        SharedPreferences.Editor editor = sharedPreferences.edit();
        editor.clear();
        editor.apply();
    }

    }


### * SQLite

关系型的轻量级数据库，保存复杂或者大数据用这个，支持标准的SQL语法。

创建数据库：Android提供了一个抽象类`SQLiteOpenHelper`，我们继承它重写里面的两个方法`onCreate()`,`onUpdate()`来创建和更新数据库表结构。创建的数据库会保存在 `/data/data/packageName/databases/`目录下。通过`getReadableDatabase`或者`getWritableDatabase`都可得到可读写的数据库，但如果数据库不可写时(如磁盘已满)，`getWritableDatabase`会报错，`getReadableDatabase`不会。

调用`MySqliteHelper.getHelper(mainActivity).getReadableDatabase();`一下数据库就建好了，下次再调用就不会再创建了。

	
    public class MyDatabaseHelper extends SQLiteOpenHelper {

    private static MyDatabaseHelper helper;
    private static final String DB_NAME = "myAndroid.db";

    public MyDatabaseHelper(Context context){
        super(context, DB_NAME, null, 2);//2 数据库version,修改数据库用
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(create_dog);
        db.execSQL(create_cat);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        //更新表才用,这里假设在 版本2 中要添加 cat 表,注意：switch 不要 break !!
        switch (newVersion){
            case 2:
                db.execSQL(create_cat);
            default:
        }
    }
    //获取 helper 单例
    public static synchronized MyDatabaseHelper getHelper(Context context) {
        context = context.getApplicationContext();
        if (helper == null) {
            synchronized (MyDatabaseHelper.class) {
                if (helper == null)
                    helper = new MyDatabaseHelper(context);
            }
        }
        return helper;
    }

    //-------------sql----------------------------
        public static final String create_dog = "create table dog ("
                +"id integer primary key autoincrement,"
                +"name text,"
                +"age integer,"
                +"weight real)";

        public static final String create_cat = "create table cat ("
                +"id integer primary key autoincrement,"
                +"name text,"
                +"age integer,"
                +"weight real)";
    }

![img](/myimg/android/mydb.png)    
把数据库导出到桌面，用[SQLiteManager](http://www.sqlitemanager.org/)打开查看
![img](/myimg/android/mytable.png) 
  
数据增，删，改，查的最简单的例子

	
    public class DogDao {
    private static final String TABLE_NAME = "dog";

    public static void add(Context context,Dog dog){
        SQLiteDatabase db =  MyDatabaseHelper.getHelper(context).getWritableDatabase();
        ContentValues values = new ContentValues();
        values.put("name",dog.getName());
        values.put("age",dog.getAge());
        values.put("weight",dog.getWeight());
        db.insert(TABLE_NAME, null, values);
    }

    public static void update(Context context,Dog dog){
        SQLiteDatabase db =  MyDatabaseHelper.getHelper(context).getWritableDatabase();
        ContentValues values = new ContentValues();
        values.put("name",dog.getName());
        values.put("age",dog.getAge());
        values.put("weight",dog.getWeight());
        db.update(TABLE_NAME, values, null, null);
    }

    public static void deleteAll(Context context){
        SQLiteDatabase db =  MyDatabaseHelper.getHelper(context).getWritableDatabase();
        db.delete(TABLE_NAME, null, null);
    }

    public static List<Dog> getAll(Context context){
        SQLiteDatabase db =  MyDatabaseHelper.getHelper(context).getWritableDatabase();
        Cursor cursor = db.query(TABLE_NAME, null, null, null, null, null, null);
        List<Dog> dogList = new ArrayList<>();
        if (cursor.moveToFirst()){
            do {
                Dog dog = new Dog();
                dog.setName(cursor.getString(cursor.getColumnIndex("name")));
                dog.setAge(cursor.getInt(cursor.getColumnIndex("age")));
                dog.setWeight(cursor.getFloat(cursor.getColumnIndex("weight")));
                dogList.add(dog);
            }while (cursor.moveToNext());
        }
        cursor.close();
        return dogList;
    }

    //事务，要么都成功，要么都失败。
    public static void transactionTest(Context context){
        SQLiteDatabase db =  MyDatabaseHelper.getHelper(context).getWritableDatabase();
        db.beginTransaction();
        try {
            //数据处理....

            db.setTransactionSuccessful();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            db.endTransaction();
        }
    }
    }
也可以用 Android SDK `/platform-tools`文件夹中自带的`adb`工具来操作链接在电脑的手机或者模拟器里的数据库。

但要先配置一下，Mac的配置如下，在终端依次输入 `cd ~` `touch .bash_profile` `open -e .bash_profile`这时会打开一个文本文件，在里面添加`export PATH=${PATH}:这里是platform-tools文件的路径`保存关闭，
然后`source .bash-profile`刷新，然后输入`adb shell`就可以开始用了。
![img](/myimg/android/pzadb.png)
![img](/myimg/android/pzadb2.png)
如查看数据库`cd /data/data/packageName/databases` `l`列出所有数据库
![img](/myimg/android/adbtable.png)

### * SD卡存储

SD卡存储和文件存储差不多，只不过是手机外部的存储空间，容量更大，不过在使用的时候要判断SD卡可不可以用。

	public class SDCardUtil {

    // 默认都保存在这个文件夹下
    private static final String DEFAULT_DIR = "ProData/";

    public static boolean isEnable() {
        return Environment.getExternalStorageState().equals(android.os.Environment.MEDIA_MOUNTED);
    }

    public static long getTotalMB() {
        if (!isEnable())
            return 0;
        File path = Environment.getExternalStorageDirectory();
        StatFs stat = new StatFs(path.getPath());
        long blockSize = stat.getBlockSizeLong();
        long availableBlocks = stat.getBlockCountLong();
        return (availableBlocks * blockSize) / 1024 / 1024;
    }

    public static long getAvailableMB() {
        if (!isEnable())
            return 0;
        File path = Environment.getExternalStorageDirectory();
        StatFs stat = new StatFs(path.getPath());
        long blockSize = stat.getBlockSizeLong();
        long availableBlocks = stat.getAvailableBlocksLong();
        return (availableBlocks * blockSize) / 1024 / 1024;
    }

    public static long getAvailableByte() {
        if (!isEnable())
            return 0;
        File path = Environment.getExternalStorageDirectory();
        StatFs stat = new StatFs(path.getPath());
        long blockSize = stat.getBlockSizeLong();
        long availableBlocks = stat.getAvailableBlocksLong();
        return availableBlocks * blockSize;
    }

    // 路径：/storage/sdcard0/
    public static String getSDCardPath() {
        if (!isEnable())
            return "";
        return Environment.getExternalStorageDirectory().getAbsolutePath() + File.separator;
    }


    public static File createDir(String directoryName) {
        //建默认文件夹
        String defaultDir = getSDCardPath() + DEFAULT_DIR;
        File defaultFile = new File(defaultDir);
        if (!defaultFile.exists()) {
            defaultFile.mkdir();
        }

        String dirPath = getFilePath(directoryName);
        File dirFile = new File(dirPath);
        if (!dirFile.exists()) {
            dirFile.mkdir();
        }
        return dirFile;
    }

    public static File createFile(String fileName) {
        try {
            //建默认文件夹
            String dirPath = getSDCardPath() + DEFAULT_DIR;
            File directory = new File(dirPath);
            if (!directory.exists()) {
                directory.mkdir();
            }

            //再在默认文件夹里建文件
            File file = new File(getFilePath(fileName));
            if (!file.exists()) {
                file.createNewFile();
            }
            return file;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public static String getFilePath(String fileName) {
        return getSDCardPath() + DEFAULT_DIR + fileName;
    }

    public static boolean isFileExist(String fileName) {
        File file = new File(getFilePath(fileName));
        return file.exists();
    }

    public static boolean saveByte(String fileName, byte[] bytes) {
        if (bytes == null) {
            return false;
        }

        OutputStream output = null;
        try {
            if (bytes.length < getAvailableByte()) {
                File file = createFile(fileName);//这里面已经建了默认文件夹
                output = new BufferedOutputStream(new FileOutputStream(file));
                output.write(bytes);
                output.flush();
                return true;
            }
        } catch (IOException e1) {
            e1.printStackTrace();
        } finally {
            try {
                if (output != null) {
                    output.close();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return false;
    }

    public static byte[] getByte(String fileName) {
        File file = new File(getFilePath(fileName));
        if (!file.exists()) {
            return null;
        }
        InputStream inputStream = null;
        try {
            inputStream = new BufferedInputStream(new FileInputStream(file));
            byte[] data = new byte[inputStream.available()];
            inputStream.read(data);
            return data;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (inputStream != null) {
                    inputStream.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return null;
    }

    public static boolean saveBitmap(Bitmap bitmap, String bitmapName) {
        if (bitmap == null) {
            return false;
        }
        //建默认文件夹
        String dirPath = getSDCardPath() + DEFAULT_DIR;
        File directory = new File(dirPath);
        if (!directory.exists()) {
            directory.mkdir();
        }

        File file = new File(getFilePath(bitmapName));
        BufferedOutputStream output = null;
        try {
            output = new BufferedOutputStream(new FileOutputStream(file));
            bitmap.compress(Bitmap.CompressFormat.JPEG, 80, output);
            output.flush();
            output.close();
            return true;
        } catch (IOException e1) {
            e1.printStackTrace();
        } finally {
            try {
                if (output != null) {
                    output.close();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return false;
    }

    public static Bitmap getBitmap(String bitmapName) {
        String myJpgPath = getFilePath(bitmapName);
        BitmapFactory.Options options = new BitmapFactory.Options();
        return BitmapFactory.decodeFile(myJpgPath, options);
    }

    public static void removeFile(String fileName) {
        String filePath = getFilePath(fileName);
        File file = new File(filePath);
        if (file.exists()) {
            file.delete();
        }
    }

    public static void removeFile(File file) {
        if (file.isFile()) {
            file.delete();
            return;
        }
        if (file.isDirectory()) {
            File[] childFiles = file.listFiles();
            if (childFiles == null || childFiles.length == 0) {
                file.delete();
                return;
            }

            for (int i = 0; i < childFiles.length; i++) {
                removeFile(childFiles[i]);
            }
            file.delete();
        }
    }

    public static void removeAll() {
        String filePath = getSDCardPath() + DEFAULT_DIR;
        File file = new File(filePath);
        if (file.exists()) {
            removeFile(file);
        }

        //再建一个空文件夹
        File directory = new File(filePath);
        if (!directory.exists()) {
            directory.mkdir();
        }
      }
    }














