



20200821 (금) 
### 수업 목차
#### 1. DataStorage 매커니즘 3 | Databases
[예제1. MyDBAdapter](#예제1-MyDBAdapter) <br/>
[예제2. DBActivity](#예제2-DBActivity) <br/>
[예제3. Room을 이용한 Data 저장 (미완) (JPA와 유사)](#예제3-Room을-이용한-Data-저장미완)




<br/>
<br/>


##  1. DataStorage 매커니즘 3 | Databases

[(참고) 안드로이드 developers | # SQLite를 사용하여 데이터 저장](https://developer.android.com/training/data-storage/sqlite?hl=ko)
### 예제1 MyDBAdapter

> MyDBAdapter.java
 - SQLite <br/>
안드로이드에서는 어플리케이션의 효과적인 데이터 관리를 위하여 구조화된 **내부 SQL Database**인 SQLite Database를 지원한다. 
파일 기반의 관계형 데이터 베이스로 **먼저 db파일을 생성하고,  그 안에서 테이블을 생성하여 사용**한다.  

<br/>

- Helper 클래스 MyHelper 생성
```java
private static class MyHelper extends SQLiteOpenHelper {  
  
    private static final String DB_CREATE = "create table " + DB_TABLE + " (" + ID +  
            " integer primary key autoincrement, " + NAME + " text not null );";  
  
  //a. Helper 객체 생성: context 객체, db파일 이름, cursor 관리 팩토리 객체(안쓰면 null), db파일 버전  
  public MyHelper(Context context, String name,  
  SQLiteDatabase.CursorFactory factory, int version) {  
  
        super(context, name, factory, version);  
  }  
  
    // b. 테이블 생성: db파일 생성될 때 한번만 호출된다.  
  @Override  
  public void onCreate(SQLiteDatabase db) {//param으로 db 조작 객체를 받아와서  
         db.execSQL(DB_CREATE);  
  }  
  
    // c. 테이블 업그레이드: 버전이 맞지 않을 때 호출된다.  
  @Override  
  public void onUpgrade(SQLiteDatabase db, int oVers, int nVers) {  
        db.execSQL("DROP TABLE IF EXISTS " + DB_TABLE);//현재 테이블을 삭제  
        onCreate(db);//새로 생성  
  }  
}

```
<br/>



 -  멤버변수 선언
```java
private static final String DB = "MyDB.db"; // DB 파일명  
private static final String DB_TABLE = "MyTable"; // 테이블명  
private static final String ID = "_id"; // 컬럼명  
private static final String NAME = "name"; // 컬럼명  
private static final int DB_VERS = 1; // DB 파일 버전  
  
private SQLiteDatabase mdb;//db 파일 처리 객체(가장 중요)  
private final Context context;  
private MyHelper mHelper; //MyHelper 객체
```
<br/>


 -  생성자
```java
public MyDBAdapter(Context context) {  
    this.context = context;  
    //helper 객체 생성  
    mHelper = new MyHelper(context, DB, null, DB_VERS);  
}
```
<br/>


 - DB OPEN (helper 객체 사용)
```java
//mHelper: helper 객체  
//dao와 동일한 역할  
public void open() throws SQLiteException {  
    try {  
        mdb = mHelper.getWritableDatabase();//읽고 쓰기 모드로 오픈  
  } catch (SQLiteException ex) {  
        mdb = mHelper.getReadableDatabase();//문제 있을 경우, 읽기 전용 모드로 오픈  
  }  
}  
```
<br/>

 - DML  작업

1.  insert
```java
public long insertData(String name) {  
    ContentValues cv = new ContentValues();//ContentValues: 값을 담는 그릇  
    cv.put(NAME, name);//insert할 컬럼명, 값  
    return mdb.insert(DB_TABLE, null, cv);  
}
```
2.  delete
```java
public int removeData(long index) {  
    return mdb.delete(DB_TABLE, ID + "=" + index, null);  
}
```
3.  update
```java
public int updateData(long index, String name) {  
    String where = ID + " = " + index;  
    ContentValues cv = new ContentValues();  
    cv.put("name", name);  
    return mdb.update(DB_TABLE, cv, where, null);  
}
```
4. 전체 검색
```java
   public Cursor getAll() {//Cursor(==resultset)  
  //query(): 검색기능  
  //param: 테이블 명, 검색할 컬럼명...  
  return mdb.query(DB_TABLE, new String[] { ID, NAME}, null, null, null, null, null);  
}
```
<br/>



<details>

<summary> MyDBAdapter.java 전체코드보기</summary>

```java
package com.example.storagetest;  
  
import android.content.ContentValues;  
import android.content.Context;  
import android.database.Cursor;  
import android.database.sqlite.SQLiteDatabase;  
import android.database.sqlite.SQLiteException;  
import android.database.sqlite.SQLiteOpenHelper;  
  
public class MyDBAdapter {  
    /*  
 SQLite는 파일 기반의 관계형 데이터 베이스  
 따라서 먼저 db파일을 생성하고,  
  그 안에서 테이블을 생성하여 사용한다.  
 */  
 //<멤버변수>  
  private static final String DB = "MyDB.db"; // DB 파일명  
  private static final String DB_TABLE = "MyTable"; // 테이블명  
  private static final String ID = "_id"; // 컬럼명  
  private static final String NAME = "name"; // 컬럼명  
  private static final int DB_VERS = 1; // DB 파일 버전  
  
  private SQLiteDatabase mdb;//db 파일 처리 객체(가장 중요)  
  private final Context context;  
  private MyHelper mHelper;  
  
 public MyDBAdapter(Context context) {  
        this.context = context;  
        //helper 객체 생성  
        mHelper = new MyHelper(context, DB, null, DB_VERS);  
        //설정한 버전이 맞지 않으면 onUpgrade 호출된다.  
  }  
  
   //helper 객체로 db파일 오픈  
  //dao와 동일한 역할  
  public void open() throws SQLiteException {  
        try {  
            mdb = mHelper.getWritableDatabase();//읽고 쓰기 모드로 오픈  
  } catch (SQLiteException ex) {  
            mdb = mHelper.getReadableDatabase();//문제 있을 경우, 읽기 전용 모드로 오픈  
  }  
    }  
  
   //db 파일 닫기  
  public void close() {  
        mdb.close();  
  }  
  
  // <DML>  
 // a. insert  
 public long insertData(String name) {  
        ContentValues cv = new ContentValues();//ContentValues: 값을 담는 그릇  
        cv.put(NAME, name);//insert할 컬럼명, 값  
        return mdb.insert(DB_TABLE, null, cv);  
  }  
  
  // b. delete  
  public int removeData(long index) {  
        return mdb.delete(DB_TABLE, ID + "=" + index, null);  
  }  
  
  // c. update  
  public int updateData(long index, String name) {  
        String where = ID + " = " + index;  
        ContentValues cv = new ContentValues();  
        cv.put("name", name);  
        return mdb.update(DB_TABLE, cv, where, null);  
  }  
  
  // d. 전체 검색  
  public Cursor getAll() {//Cursor(==resultset)  
  //query(): 검색기능  
  //param: 테이블 명, 검색할 컬럼명...  
  return mdb.query(DB_TABLE, new String[] { ID, NAME}, null, null, null, null, null);  
  }  
  
  
    // <SQLiteOpenHelper 기능>  
  private static class MyHelper extends SQLiteOpenHelper {  
        private static final String DB_CREATE = "create table " + DB_TABLE + " (" + ID +  
                " integer primary key autoincrement, " + NAME + " text not null );";  
  
  //a. Helper 객체 생성: context 객체, db파일 이름, cursor 관리 팩토리 객체(안쓰면 null), db파일 버전  
  public MyHelper(Context context, String name,  
  SQLiteDatabase.CursorFactory factory, int version) {  
            super(context, name, factory, version);  
  }  
  
   // b. 테이블 생성: db파일 생성될 때 한번만 호출된다.  
  @Override  
  public void onCreate(SQLiteDatabase db) {//param으로 db 조작 객체를 받아와서  
            db.execSQL(DB_CREATE);  
  }  
  
   // c. 테이블 업그레이드: 버전이 맞지 않을 때 호출된다.  
  @Override  
  public void onUpgrade(SQLiteDatabase db, int oVers, int nVers) {  
            db.execSQL("DROP TABLE IF EXISTS " + DB_TABLE);//현재 테이블을 삭제  
            onCreate(db);//새로 생성  
  }  
    }  
  
}
```
</details>

<br/>

### 예제2 DBActivity

> activity_d_b.xml <br/>

![image](https://user-images.githubusercontent.com/62331803/90840505-1b8bcb00-e395-11ea-9e32-a987f7a2bd9e.png)

> DBActivity.java 

<details>
<summary>코드보기</summary>

```java
package com.example.storagetest;  
  
import androidx.appcompat.app.AppCompatActivity;  
  
import android.database.Cursor;  
import android.os.Bundle;  
import android.view.View;  
import android.widget.ArrayAdapter;  
import android.widget.EditText;  
import android.widget.ListView;  
  
import java.util.ArrayList;  
  
class Names{  
    public int _id;  
    public String name;  

    public Names() {  
    }  
  
    public Names(int _id, String name) {  
        this._id = _id;  
        this.name = name;  
  }  
  
    @Override  
  public String toString() {  
        return "Names{" +  
                "_id=" + _id +  
                ", name='" + name + '\'' +  
                '}';  
  }  
}  
  
public class DBActivity extends AppCompatActivity {  
    private MyDBAdapter dbAdapter;  
    private ArrayList<Names> list; //db 내용 저장  
    private ArrayAdapter<Names> adapter;  
    private ListView listView;  
    private Cursor cursor;//검색 결과 담는 그릇  
    private EditText name;  
  
  @Override  
  protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_d_b);  
        listView = findViewById(R.id.lv1);  
        name = findViewById(R.id.name);  

        dbAdapter = new MyDBAdapter(getApplicationContext());//getApplicationContext(): 현재 프로그램의 컨텍스트 get. (==this)  list = new ArrayList<>();  

        adapter = new ArrayAdapter(getApplicationContext(), android.R.layout.simple_list_item_1,list);  
        listView.setAdapter(adapter);  
        DBinit();  
        }  
  
     //데이터 베이스 open  public void DBinit(){ 
     dbAdapter.open();  
     makeList();  
  }  
  
    //데이터 추출하여 list 갱신  
  public void makeList(){  
        list.clear();
        cursor = dbAdapter.getAll();  
        if(cursor.moveToFirst()){//첫째줄로 이동  
        do{ 
         list.add(new Names(cursor.getInt(0),cursor.getString(1)));
         //0번과 1번 컬럼값 setting  	
       }while(cursor.moveToNext());//다음줄로 이동  
    }  
     adapter.notifyDataSetChanged();//뷰 갱신  
 }  
  
  //데이터 추가  
  public void onSave(View view){  
        String n = name.getText().toString();  
        dbAdapter.insertData(n);//insert 호출하여 데이터 저장  
        makeList();  
  }  
  
    //현재 activity 종료  
  @Override  
  protected void onDestroy() {  
        super.onDestroy();  
        dbAdapter.close();//db close  
  }  
}

```

</details>

> 결과화면
- 이름 저장 <br/>
![image](https://user-images.githubusercontent.com/62331803/90840046-ecc12500-e393-11ea-9398-c25d19a93579.png)

- 재실행 후: 이전 시행에서 저장한 id 출력됨<br/>
![image](https://user-images.githubusercontent.com/62331803/90840099-0c584d80-e394-11ea-97a1-34cc5d5a862f.png)

<br/>

### 예제3 Room을 이용한 Data 저장(미완)

[(참고) 안드로이드 developers | Room 사용 가이드](https://developer.android.com/training/data-storage/room)
> Room 설명
[참고 블로그](https://medium.com/@eevee300/android-sqlite-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-94d17f06123d)

Android 에서는 기본적으로 모바일 데이터베이스로 SQLite를 제공한다.<br/>
기존에는 SQLite(SQLiteOpenHelper)를 사용하여 필요한 데이터를 저장하였다. 하지만 2017년 Google I/O 에서 Android Architecture Components(AAC)를 발표하면서 Android Developer 사이트에는 SQLite를 직접 사용하는것은 low level API 이며 구현시 오류발생의 가능성이 높아 **보다 안전하게 모바일 데이터베이스를 사용하고 싶으면 AAC 의 Room을 사용할것을 강력 추천**한다. **Room은 SQLite를 추상화 하여 쉽게 데이터베이스에 대한 인터페이스를 제공해주는 라이브러리**이다.

> 순서 

1.  build.gradle (Module:app) 파일 셋팅
```java
dependencies {  
  def room_version = "2.2.5"  
  
  implementation "androidx.room:room-runtime:$room_version"  
  annotationProcessor "androidx.room:room-compiler:$room_version" // For Kotlin use kapt instead of annotationProcessor  
  
 // optional - Kotlin Extensions and Coroutines support for Room  implementation "androidx.room:room-ktx:$room_version"  
  
  // optional - RxJava support for Room  
  implementation "androidx.room:room-rxjava2:$room_version"  
  
  // optional - Guava support for Room, including Optional and ListenableFuture  
  implementation "androidx.room:room-guava:$room_version"  
  
  // Test helpers  
  testImplementation "androidx.room:room-testing:$room_version"  
  
}
```
2. Member VO (테이블) 생성  
```java
package com.example.storagetest;  
  
  
import androidx.room.Entity;  
import androidx.room.PrimaryKey;  
  
//VO 클래스(테이블)  
@Entity  
public class Member {  
	 @PrimaryKey(autoGenerate = true)
	 public int _id;  
	 public String name;  
	 public String tel;  
	  
 public Member() {  
    }  
  
 public Member(int _id, String name, String tel) {  
	 this._id = _id;  
	 this.name = name;  
	 this.tel = tel;  
  }  
  
    @Override  
  public String toString() {  
        return  
  "_id=" + _id + "     name= " + name + "      tel=" + tel;  
  }  
}
```

3. MemberDao 인터페이스 생성 (JPARepository 역할)
```java
package com.example.storagetest;  
  
import androidx.room.Dao;  
import androidx.room.Delete;  
import androidx.room.Insert;  
import androidx.room.Query;  
import androidx.room.Update;  
  
import java.util.List;  
  
//JPARepository 역할  
@Dao  
public interface MemberDao {  
    @Insert  
  void insert(Member m);  
  
  @Query("select * from member")  
    List<Member> selectAll();  
  
  @Query("select * from member where _id=(:id)")//_id 처럼 언더바 쓰면 오류남  
  Member selectById(int id);  
  
  @Delete  
  void delete(Member m);  
  
  @Update  
  void update(Member m);  
}
```

4. AppDatabase

```java
package com.example.storagetest;  
  
import android.content.Context;  
  
import androidx.room.Database;  
import androidx.room.Room;  
import androidx.room.RoomDatabase;  
  
@Database(entities = {Member.class}, version=1)  
public abstract class AppDatabase extends RoomDatabase {  
    private static AppDatabase appDatabase;//database  
  public abstract MemberDao memberDao();//Repository 객체  
  
  public static AppDatabase getInstance(Context context){  
/*  
 db에는 커넥션 생성 시간이 젤 오래 걸림  
 따라서 매번 불필요하게 생성하지 않고 싱글톤으로 하나만 만들어서 클래스 내에서 공유한다.  
 */  
 if(appDatabase==null){  
            appDatabase = Room.databaseBuilder(context, AppDatabase.class, "app_db").build();  
  }  
        return appDatabase;  
  }  
  
    public static void delDB(){  
        appDatabase = null;  
  }  
}
```

5. MemberDBAdapter (x)
```java
package com.example.storagetest;  
  
import java.util.List;  
  
public class MemberDBAdapter {  
    private AppDatabase db;  
 private MemberDao dao;  
  
 public MemberDBAdapter() {  
        dao = db.memberDao();  
  }  
  
    public void addMember(Member m){  
        dao.insert(m);  
  }  
  
    public List<Member> getAll(){  
        return dao.selectAll();  
  }  
  
    public void editMember(Member m){  
        dao.update(m);  
  }  
  
    public void delMember(Member m){  
        dao.delete(m);  
  }  
}
```
6. RoomActivity 
- activity_room.xml
![image](https://user-images.githubusercontent.com/62331803/90844674-6ad6f900-e39f-11ea-95cc-abcbd92ff7b1.png)

- RoomActivity.java <br/>
**CF. 안드로이드 UI 조작 | AsyncTask <br/>
:: UI thread와 Handler의 처리를 위한 Helper Class**  [참고 블로그](https://mommoo.tistory.com/29) <br/>
 안드로이드에서의 일처리는 메인스레드(UI 스레드)가 담당한다. 특히 UI와 관련된( ex) TextView,ImageView )  
일처리는 메인스레드만 담당 하게끔 설계를 했다. 그래서 메인스레드를 UI스레드라고도 불린다.  
따라서 **복잡한 계산은 백그라운드 스레드( 메인 스레드가 아닌 다른 스레드의 총칭)에 맡긴후**  
계산된 결과값을 UI스레드에게 일을 시켜야 하는 것이다.

<details>
<summary>RoomActivity.java 전체코드보기 (미완) </summary>

```java
package com.example.storagetest;  
  
import androidx.appcompat.app.AppCompatActivity;  
  
import android.os.AsyncTask;  
import android.os.Bundle;  
import android.view.View;  
import android.widget.ArrayAdapter;  
import android.widget.EditText;  
import android.widget.ListView;  
  
import java.util.ArrayList;  
  
public class RoomActivity extends AppCompatActivity {  
    private AppDatabase database;  
 private MemberDao dao;  
 private ArrayList<Member> list;  
 private ArrayAdapter<Member> adapter;  
 private EditText name_et;  
 private EditText tel_et;  
 private ListView listView;  
  
  @Override  
  protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
  setContentView(R.layout.activity_room);  
  name_et = findViewById(R.id.name_et);  
  tel_et = findViewById(R.id.tel_et);  
  listView = findViewById(R.id.lv2);  
  
  database = AppDatabase.getInstance(getApplicationContext());  
  dao = database.memberDao();  
  
  list = new ArrayList<>();  
  adapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1, list);  
  listView.setAdapter(adapter);  
  
  }  
  
    public void onSave(View view){  
        String n = name_et.getText().toString();  
  String t = tel_et.getText().toString();  
//        new InsertAsyncTask(new Member(0,n,t)).execute();  
//  
//        dao.insert(new Member(0,n,t));  
//        list= (ArrayList<Member>) dao.selectAll();  
//        adapter.notifyDataSetChanged();  
  }  
  
    /*  
 <안드로이드 UI 조작>  
 AsyncTask : UI thread와 Handler의 처리를 위한 Helper Class  안드로이드에서의 일처리는 메인스레드(UI 스레드)가 담당한다. 특히 UI와 관련된( ex) TextView,ImageView )  
  일처리는 메인스레드만 담당 하게끔 설계를 했다. 그래서 메인스레드를 UI스레드라고도 불린다.  
  따라서 복잡한 계산은 백그라운드 스레드( 메인 스레드가 아닌 다른 스레드의 총칭)에 맡긴후  
 계산된 결과값을 UI스레드에게 일을 시켜야 하는 것이다.  
 */  

 public static class InsertAsyncTask extends AsyncTask<Member, Void, Void> {  
      private MemberDao dao;  
  
	  public InsertAsyncTask(MemberDao dao){  
         this.dao = dao;  
  }  
  
   // 백그라운드에서 작업할 내용  
  @Override  
  protected Void doInBackground(Member... members) {  
            dao.insert(members[0]);  
			return null;  
		}  
    }  
}
```

</details>

## 2.  큰 제목

### 부제

> 파일명
