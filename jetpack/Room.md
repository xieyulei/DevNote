Jetpack 之 Room

[TOC]

------

##### **1.依赖添加**

```java
dependencies {
	implementation "andoirdx.room:room-runtime:2.2.2"
	annotationProcessor "androidx.room:room-compiler:2.2.2"
}
```



##### **2.创建Entity, 创建一张学生表**

```java
新建一个名为Student的Java类文件，并在类文件的上方添加@Entity标签
	
	@Entity(tableName = "student")
	public class Student {
		@PrimaryKey(autoGenerate = true)
		@ColumnInfo(name = "id",typeAffinity = ColumnInfo.INTEGER)
		public int id;
		
		@ColumnInfo(name = "name", typeAffinity = ColumnInfo.TEXT)
		public String name;
		
		@ColumnInfo(name = "age", typeAffinity = ColumnInfo.TEXT)
		public String age;
		
		/**
		 * Room默认会使用这个构造器操作数据
		**/ 
		public Student(int id , String name , String age){
			this.id = id;
			this.name = name;
			this.age = age;
		}
		
		/**
		 * 由于Room只能识别和使用一个构造器，如果希望定义多个构造器
		 * 可以使用Ignore标签，让Room忽略这个构造器
		 * 不仅如此，@Ignore标签还可以用于字段
		 * Room不会吃就话被@Ignore标签标记过的字段的数据
		 */
		@Ignore
		public Student(String name,String age){
			this.name = name;
			this.age = age;
		}
	}
```



##### **3.针对Entity定一个对应的Dao接口文件，以便对Entity进行访问**

```java
@Dao
public interface StudentDao {
    @Insert
    void insert(Student student);

    @Delete
    void delete(Student student);

    @Update
    void update(Student student);

    @Query("SELECT * FROM student")
    List<Student> getStudentList();

    @Query("SELECT * FROM student WHERE id= :id")
    Student getStudentById(int id);

}
```



##### **4.创建数据库**

```java
// @Database标签用于告诉系统这是Room数据库对象
// entities属性用于指定该数据库有哪些表若要建立多张表，则表名要用逗号隔开
// version用于指定数据库版本号，后面数据库的升级就是依据此版本号进行

@Database(entities = {Student.class}, version = 1)
public abstract class MyDatabase extends RoomDatabase {

    private static final String DATABASE_NAME = "my_db";

    private static MyDatabase databaseInstance;

    public static synchronized MyDatabase getInstance(Context context) {
        if (databaseInstance == null) {
            databaseInstance = Room.databaseBuilder(context.getApplicationContext(), MyDatabase.class, DATABASE_NAME).build();
        }
        return databaseInstance;
    }

    public abstract StudentDao studentDao();

}
```



##### **5.操作数据库**

```java

	千万注意：这些对于数据库的操作，不能直接在UI线程中执行这些操作，所有操作都需要放在工作线程中进行，例如可以AsyncTask操作;

	5.1 获取数据库对象
        MyDatabase myDatabase = MyDatabase.getInstance(this);
    
	5.2 插入数据
        MyDatabase.studentDao().insert(new Student(name,age));
    
	5.3 更新数据
    	MyDatabase.studentDao().update(new Student(id,name,age));
        
	5.4 删除数据
    	MyDatabase.studentDao().delete(student);
        
	5.5 查询所有学生
    	MyDatabase.studentDao().getStudentList();
        
	5.6 查询某个学生
		MyDatabase.studentDao().getStudnetById(id);

```



##### **6.Room + LiveData + ViewModel的结合使用**

```java
思路：
    按照数据库的基本使用方法，每当数据库中的数据发生变化时，我们都需要开启一个工作线程去重新获取数据库中的数据;
	这给我们带来很大的不便，我们希望在数据库数据发生变化时候自动收到通知;

总结：使用Room + LiveData + ViewModel结合的方式，当数据库数据发生变化时，通过LiveData组件通知View层，实现数据的更新;

// 1.修改Dao文件，将要监听的数据使用LiveData包装起来
@Dao
public interface StudentDao {
    @Insert
    void insert(Student student);

    @Delete
    void delete(Student student);

    @Update
    void update(Student student);

    //使用LiveData将数据包装起来
    @Query("SELECT * FROM student")
    LiveData<List<Student>> getStudentList();

    @Query("SELECT * FROM student WHERE id= :id")
    Student getStudentById(int id);

}


// 2.创建ViewModel类，继承自AndroidViewModel，在构造器中实例化数据库，将使用LiveData包装数据
public class StudentViewModel extends AndroidViewModel{
    private MyDatabase myDatabase;
    private LiveData<List<Student>> liveDataStudent;
    
    public StudentViewModel(@NonNull Application application){
        super(application);
        myDatabase = MyDatabase.getInstance(application);
        liveDataStudent = myDatabase.studentDao().getStudentList();
    }
    
    public LiveData<List<Student>> getLiveDataStudent(){
        return liveDataStudent;
    }
}
    
// 3.实例化ViewModel并引用，通过ViewModel获取其内部方法，并设置observe进行变化监听，数据发生改变后，进行相应的操作
    
private List<Student> studentList = new ArrayList<>();

StudentViewModel studentViewModel = new  ViewModelProvider(this).get(StudentViewModel.class);
studentViewModel.getLiveDataStudent().observe(this,new Observer<List<Student>>()){
    @Override
    public void onChanged(List<Student> students){
        studentList.clear();
        studentList.addAll(students);
        studentAdapter.notifyDataSetChanged();
    }
}
    
```



##### **7.Room数据库升级**

```java
// 1.数据库升级--Migration
当数据库的表需要增加或者删除时，就需要对数据库进行升级，Room数据库升级可以使用Migration;

升级分为两步：
    第一步：修改数据库版本，@Database(entities = {Student.class , version = 1}) 修改version的值即可
    第二步：添加升级方案 并将方案加入数据库的创建中;（添加Migration）


        
数据库升级从1到3时，Room会先判断当前有没有直接从1到3的升级方案，有则直接升级;
没有的话，就按照Migration(1,2),然后Migration(2,3)的顺序完成升级.

Migration类有两个参数== public class Migration(int startVersion, int endVersion)
startVersion表示当前数据库的版本
endVersion表示即将要升级到的版本
    
例如：将数据库版本从1升级到2
static final Migration MIGRATION_1_2 = new Migration(1,2)
    
@Override
public void migrate(@NonNull SupportSQLiteDatabase database){
    //执行与升级相关的操作
}
    
同理，将数据库版本从2升级到3
static final Migration MIGRATION_2_3 = new Migration(2,3)
    
@Override
public void migrate(@NonNull SupportSQLiteDatabase database){
    //执行与升级相关的操作
}    




// 2.升级异常
场景：我们要将数据库升级到版本4,但是没有写相应的Migration,则会出现异常
Cause by:java.lang.IllegalStateException
    A migration from 3 to 4 was required but not found.

解决方法：
    方法一：添加对应的Migration升级方案（建议）    
    方法二：在创建数据库时加入fallbackToDestructiveMigration()方法，该方法能够在出现升级异常时，
    	   重新创建数据库，避免程序崩溃，但是有明显的缺点是数据库表会被重新创建，所有数据都会丢失.
	代码如下：
    Room.databaseBuilder(context.getApplicationContext(),MyDatabase.class,DATABASE_NAME)
    	.fallbackToDestructiveMigration()
    	.addMigrations(MIGRATION_1_2,MIGRATION_2_3,MIGRATION_1_3)
    	.build();


// 3.查看升级结果--Schema升级文件

Room升级之后，会自动生成一个Schema文件，这是一个Json格式的文件，其中包含了数据库的所有信息;
Schema文件默认是导出的，我们可以在build.gradle(app) 文件中指定Schema文件的导出位置;

也可以设置不导出，即：在@Database标签中指定 exportSchema = false;
例如：@Database(@entities = {Student.class},exportSchema = false ,version = 2)

自定义导出路径：
android {
    defaultConfig {
		javaCompileOptions {
            annotationProcessorOptions {
                //指定数据库Schema文件的导出位置,此处设置为：根目录/schema目录下
                arguments = ["room.schemaLocation":
                             "$projectDir/schemas".toString()]
            }
        }
    }
}
 

// 4.销毁与重建策略

案例：将Student表中的age字段类型从INTEGER改为TEXT.
    
思路：采用销毁重建策略
    第一步：创建一张表结构要求的临时表temp_Student
    第二步：将数据从旧表Student复制到临时表temp_Student
    第三步：删除旧表Student
    第四步：将临时表temp_Student重命名为Student;

static final Migration MIGRATION_3_4 = new Migration(3,4)
    
@Override
public void migrate(@NonNull SupportSQLiteDatabase database){
   	database.execSQL("CREATE TABLE temp_Student("+
                     "id INTEGER PRIMARY KEY NOT NULL,"+
                     "name TEXT,"+)
                     "age TEXT)");
    
    database.execSQL("INSERT INTO temp_Student(id,name,age)"+
                    "SELECT id, name ,age FORM Student");
    database.execSQL("DROP TABLE Student");
    database.execSQL("ALTER TABLE temp_Student RENAME TO Student");
}  
    
```













