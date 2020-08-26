




20200824 (월) 
### 수업 목차
#### 1. DataStorage 매커니즘 3 | Databases
예제1) [Room을 이용한 Data 저장(완료) ](#예제1-Room을-이용한-Data-저장완료)  <br/>

#### 2. RoomDBTest 프로젝트 | Databases 심화
예제1)[ Handler를 이용한 Data 처리](#예제1-MainActivity) <br/> 

예제2)  [ LiveData를 이용한 Data 처리](#예제2--LiveDataTestActivity)  <br/>

예제3) [ LiveData와 ViewModel을 이용한 Data 처리](#예제3-ViewModelTestActivity)<br/>





<br/>
<br/>


##  1. DataStorage 매커니즘 3 | Databases

### 예제1 Room을 이용한 Data 저장(완료)

[(참고) 안드로이드 developers | Room 사용 가이드](https://developer.android.com/training/data-storage/room)

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
2. Member VO (테이블) 생성 (autoGenerate 추가)
```java
package com.example.storagetest;


import androidx.room.Entity;
import androidx.room.PrimaryKey;

//VO 클래스(테이블)
@Entity
public class Member {
    @PrimaryKey(autoGenerate = true) //시퀀스 자동할당
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
        return "_id=" + _id + "     name= " + name + "      tel=" + tel;
    }
}


```

3. MemberDao interface (JPARepository 역할)
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

4. AppDatabase (수정) <br/>
allowMainThreadQueries로 UI 쓰레드(메인쓰레드)에서 DB작업 허용시키기
```java
package com.example.storagetest;  
  
import android.content.Context;  
  
import androidx.room.Database;  
import androidx.room.Room;  
import androidx.room.RoomDatabase;  
  
@Database(entities = {Member.class}, version=1)  
  // 테이블 추가하려면 entities에 추가
  public abstract class AppDatabase extends RoomDatabase {  
  private static AppDatabase appDatabase;//database  
  public abstract MemberDao memberDao();//Repository 객체
  //public abstract MemberDao blahblahDao();//Repository 객체 추가
  public static AppDatabase getInstance(Context context){  
 
 /*  
 db에는 커넥션 생성 시간이 젤 오래 걸림  
 따라서 매번 불필요하게 생성하지 않고 싱글톤으로 하나만 만들어서 클래스 내에서 공유한다.  
 */  
 
  if(appDatabase==null){  
    appDatabase = Room.databaseBuilder(context, AppDatabase.class, "app_db")  
    .allowMainThreadQueries()//원래 room db 작업은 ui thread에서 실행할 수 없으나, 이를 허용하는 코드  
    .build();  
}  
    return appDatabase; 
  }  
  
    public static void delDB(){  
        appDatabase = null;  
  }  
}
```

5. RoomActivity 
- activity_room.xml <br/>
![image](https://user-images.githubusercontent.com/62331803/90844674-6ad6f900-e39f-11ea-95cc-abcbd92ff7b1.png)

- RoomActivity.java <br/>

<details>
<summary>RoomActivity.java 전체코드보기 (완료) </summary>

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
		
		getAll();
    }

    //room은 기본적으로 ui쓰레드(메인쓰레드)에서 db작업하는 것을 허용하지 않음.
    //따라서 AppDatabase에서 allowmainthreadquery를 설정해줌으로써 억지로 db작업을 가능케 함.
    public void getAll(){
        list.clear();
        ArrayList<Member> tmp = (ArrayList<Member>) dao.selectAll();//allowmainthreadquery로 허용
        if(tmp==null){
            return;
        }
        list.addAll(tmp);
        System.out.println(list);
        adapter.notifyDataSetChanged();
    }
    public void onSave(View view){
        String n = name_et.getText().toString();
        String t = tel_et.getText().toString();
        dao.insert(new Member(0,n,t));//allowmainthreadquery로 허용
        getAll();
    }
}
```

</details>

<br/>

> 오류 해결 

databases의 app_db관련 파일3개 삭제하고 다시 run <br/>
![image](https://user-images.githubusercontent.com/62331803/90993036-373dde00-e5ee-11ea-9b42-70e0515f82f9.png)

>결과화면 <br/> 

![image](https://user-images.githubusercontent.com/62331803/90993296-48d3b580-e5ef-11ea-952f-e52630f2fc53.png)

<br/>

## 2. RoomDBTest 프로젝트 | Databases 심화

### 예제1 MainActivity
####  :: DBAdapter와 Handler를 사용한 data 처리

1. Member.java (예제1과 동일)
```java
package com.example.roomdbtest;


import androidx.room.Entity;
import androidx.room.PrimaryKey;

//VO 클래스(테이블)
@Entity
public class Member {
    @PrimaryKey(autoGenerate = true)
    public int _id;
    public String name;
    public String tel;

    public Member() {}

    public Member(int _id, String name, String tel) {
        this._id = _id;
        this.name = name;
        this.tel = tel;
    }

    @Override
    public String toString() {
        return "_id=" + _id + "     name= " + name + "      tel=" + tel;
    }
}
```

<br/>

2. MemberDao.java 인터페이스 (예제1과 동일)
```java
package com.example.roomdbtest;

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
    List < Member > selectAll();

    @Query("select * from member where _id=(:id)") //_id 처럼 언더바 쓰면 오류남
    Member selectById(int id);

    @Delete
    void delete(Member m);

    @Update
    void update(Member m);
}
```
<br/>

3.  AppDatabase.java  (예제1과 동일)
```java
package com.example.roomdbtest;

import android.content.Context;

import androidx.room.Database;
import androidx.room.Room;
import androidx.room.RoomDatabase;

@Database(entities = {
    Member.class
}, version = 1)
public abstract class AppDatabase extends RoomDatabase {
    private static AppDatabase appDatabase; //database
    public abstract MemberDao memberDao(); //Repository 객체

    public static AppDatabase getInstance(Context context) {
        //싱글톤으로 생성

        if (appDatabase == null) {
            appDatabase = Room.databaseBuilder(context, AppDatabase.class, "app_db").build();
        }
        return appDatabase;
    }

    public static void delDB() {
        appDatabase = null;
    }
}
```
<br/> 
4. MemberDBAdapter.java

 <details>
<summary> 코드보기 </summary>

 ```java
package com.example.roomdbtest;

import android.content.Context;
import android.os.Handler;
import android.os.Message;

import java.util.ArrayList;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class MemberDBAdapter {
    private AppDatabase database;
    private MemberDao dao;
    private ExecutorService executorService; //db 백그라운드 작업을 위한 도구(쓰레드)
    private Handler handler; //UI thread와 db작업 thread 사이에 메시지 교환을 돕는 역할.

    public MemberDBAdapter(Context context, Handler handler) {
        this.handler = handler; //한쪽에서 만들어서 두 측에서 함께 쓴다.
        database = AppDatabase.getInstance(context);
        dao = database.memberDao();
        executorService = Executors.newSingleThreadExecutor();
    }

    //1. 전체 검색
    public void getAll() {
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                Message msg = handler.obtainMessage(); //메세지를 담을 객체 생성
                msg.what = 0; //메세지 종류 설정.(반대편에 반환할 값을 알려주는 역할. list<Member> or Member...)
                //msg.arg1 : 메세지 값이 int 타입인 경우
                //msg.obj : 메세지 값이 클래스 타입인 경우
                msg.obj = (ArrayList < Member > ) dao.selectAll();
                handler.sendMessage(msg); //handler를 통해 main activity에 전달.
            }
        });
    }

    //2. 멤버 1명 검색
    public void getMember(final int _id) {
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                Message msg = handler.obtainMessage();
                msg.what = 1;
                msg.obj = dao.selectById(_id);
                handler.sendMessage(msg);
            }
        });
    }

    public void addMember(final Member m) {
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                dao.insert(m);
            }
        });
    }


    public void editMember(final Member m) {
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                dao.update(m);
            }
        });
    }

    public void delMember(final Member m) {
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                dao.delete(m);
            }
        });
    }
}
 ```

 </details>
 
 <br/> 
 
 5. Main
> activity_main.xml <br/>

![image](https://user-images.githubusercontent.com/62331803/90995120-f2b64080-e5f5-11ea-9d7a-49018b3ae5b8.png)


> MainActivity.java
<details>
<summary> 코드보기 </summary>

```java
package com.example.roomdbtest;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.os.Handler;
import android.os.Looper;
import android.os.Message;
import android.view.ContextMenu;
import android.view.MenuItem;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.EditText;
import android.widget.ListView;

import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {
    private EditText name_et;
    private EditText tel_et;
    private ListView listView;
    private ArrayAdapter < Member > adapter;
    private ArrayList < Member > list;
    private Handler handler;
    private MemberDBAdapter db;
    private Member m;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        name_et = findViewById(R.id.name_et);
        tel_et = findViewById(R.id.tel_et);
        listView = findViewById(R.id.lv1);
        list = new ArrayList < > ();

        adapter = new ArrayAdapter < > (this, android.R.layout.simple_list_item_1, list);
        listView.setAdapter(adapter);

        //handler == msg queue(쓰레드 간에 통신을 할 수 있게)
        handler = new Handler(Looper.getMainLooper()) { //looper: 내부 loop (내부에 메세지가 존재하면 계속 반복)
            //메세지가 도착하면 자동으로 호출됨
            @Override
            public void handleMessage(@NonNull Message msg) {
                if (msg.what == 0) { //전체 검색
                    ArrayList < Member > tmp = (ArrayList < Member > ) msg.obj;
                    list.clear();
                    list.addAll(tmp);
                    adapter.notifyDataSetChanged();
                } else { //msg.what==1: 멤버 1명 검색
                    Member x = (Member) msg.obj;
                    name_et.setText(x.name);
                    tel_et.setText(x.tel);
                }
            }
        };
        db = new MemberDBAdapter(this, handler);
        registerForContextMenu(listView);

        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView < ? > adapterView, View view, int i, long l) {
                db.getMember(list.get(i)._id); //list i번째 방의 id값으로 검색
            }
        });
    }


    public void onSave(View view) {
        String n = name_et.getText().toString();
        String t = tel_et.getText().toString();
        db.addMember(new Member(0, n, t));
        name_et.setText("");
        tel_et.setText("");
        db.getAll();
    }
    public void onEdit(View view) {
        if (m != null) {
            m.name = name_et.getText().toString();
            m.tel = tel_et.getText().toString();
            db.editMember(m);
            db.getAll();
            m = null;
            name_et.setText("");
            tel_et.setText("");
        }
    }

    //화면 시작되자마자 실행되는 코드
    @Override
    protected void onPostResume() {
        super.onPostResume();
        db.getAll(); //onCreate에 넣으면 싱크 안맞음
    }

    @Override
    public void onCreateContextMenu(ContextMenu menu, View v, ContextMenu.ContextMenuInfo menuInfo) {
        super.onCreateContextMenu(menu, v, menuInfo);
        menu.add(0, 1, 0, "edit");
        menu.add(0, 2, 0, "delete");
    }

    @Override
    public boolean onContextItemSelected(@NonNull MenuItem item) {
        AdapterView.AdapterContextMenuInfo info = (AdapterView.AdapterContextMenuInfo) item.getMenuInfo();
        int idx = info.position; //길게 누른 위치값
        m = list.get(idx);
        switch (item.getItemId()) {
            case 1:
                name_et.setText(m.name);
                tel_et.setText(m.tel);
                break;
            case 2:
                db.delMember(m);
                db.getAll();
                m = null;
                break;
        }
        return true;
    }
}

```


</details>


> 결과화면 <br/>
- data 저장 <br/> 
![image](https://user-images.githubusercontent.com/62331803/90995876-38740880-e5f8-11ea-8973-14ad3cf834de.png)
- context menu로 입력창에 정보 출력<br/> 
![image](https://user-images.githubusercontent.com/62331803/90996812-cc46d400-e5fa-11ea-9b02-dd1289a99e8e.png)

- 삭제<br/>
![image](https://user-images.githubusercontent.com/62331803/90997585-eb466580-e5fc-11ea-8a46-42aaba743116.png)


- 수정 <br/>
![image](https://user-images.githubusercontent.com/62331803/90997622-01542600-e5fd-11ea-8b10-db3cfcab383d.png)

<br/>




### 예제2  LiveDataTestActivity

핸들러와 동일한 역할을 하는 LiveData를 사용 <br/> 
*파생스레드에서 작업할 때 핸들러를 사용하지 않는 경우 오류 발생. 

> activity_live_data_test.xml <br/>

![image](https://user-images.githubusercontent.com/62331803/90999320-4417fd00-e601-11ea-9b1b-3f076304a2dc.png)


> LiveDataTestActivity.java

```java
package com.example.roomdbtest;

import androidx.appcompat.app.AppCompatActivity;
import androidx.lifecycle.MutableLiveData;
import androidx.lifecycle.Observer;

import android.os.Bundle;
import android.widget.TextView;

import java.util.ArrayList;

import static java.lang.Thread.sleep;

public class LiveDataTestActivity extends AppCompatActivity {
    private TextView tv;
    private MutableLiveData < String > liveData; //liveData == handler

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_live_data_test);
        tv = findViewById(R.id.textView);
        liveData = new MutableLiveData < > ();

        liveData.observe(this, new Observer < String > () { //observer가 데이터 변경을 계속 관찰
            @Override
            public void onChanged(String s) { //감지가 되면 자동으로 호출을 하여 view에 적용시킴
                tv.setText(s);
            }
        });

        new Thread() {
            @Override
            public void run() {
                ArrayList < String > list = new ArrayList < > ();
                list.add("aaa");
                list.add("bbb");
                list.add("ccc");
                for (int i = 0; i < 100; i++) {
                    //liveData.setValue("asdf");  // setValue(): main 쓰레드에서 값을 저장할 때 사용. 그런데 파생 쓰레드에선 쓰지 못한다.
                    liveData.postValue(list.get(i % 3)); // => 파생 쓰레드에서 넣어준 값을 저장하려면 postValue() 사용!
                    try {
                        sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }.start();
    }
}
```

> 실행화면 <br/>

텍스트 계속 변경

![image](https://user-images.githubusercontent.com/62331803/90998392-fd290800-e5fe-11ea-80f1-2b10bad66913.png)
![image](https://user-images.githubusercontent.com/62331803/90998323-d074f080-e5fe-11ea-897a-eaaaf8e49d87.png)

<br/>

### 예제3 ViewModelTestActivity
####  :: Livedata와 ViewModel을 사용한 data 처리

> ViewModel <br/>

 액티비티의 생명주기와 연결되어 액티비티 실행 중에는 데이터 유지, 액티비티 소멸 시 함께 소멸
	
> MemberViewModel.java
<details>
<summary>코드보기</summary>

```java
package com.example.roomdbtest;

import android.content.Context;

import androidx.lifecycle.MutableLiveData;
import androidx.lifecycle.ViewModel;

import java.util.ArrayList;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class MemberViewModel extends ViewModel {
    private AppDatabase database;
    private MemberDao dao;
    private ExecutorService executorService; //db 백그라운드 작업을 위한 도구(쓰레드)
    private MutableLiveData < ArrayList < Member >> members; //전체 검색용
    private MutableLiveData < Member > member; //1명 검색용

    //getter: activity에 전달
    public MutableLiveData < ArrayList < Member >> getMembers() {
        return members;
    }
    public MutableLiveData < Member > getMember() {
        return member;
    }

    public void create(Context context) {
        database = AppDatabase.getInstance(context);
        dao = database.memberDao();
        executorService = Executors.newSingleThreadExecutor();
        members = new MutableLiveData < > ();
        member = new MutableLiveData < > ();
    }

    // 전체 검색
    public void getAll() {
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                members.postValue((ArrayList < Member > ) dao.selectAll()); //결과를 livedata에 삽입
            }
        });
    }

    // 멤버 1명 검색
    public void getMember(final int _id) {
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                member.postValue(dao.selectById(_id)); //결과를 livedata에 삽입
            }
        });
    }

    public void addMember(final Member m) {
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                dao.insert(m);
            }
        });
    }

    public void editMember(final Member m) {
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                dao.update(m);
            }
        });
    }

    public void delMember(final Member m) {
        executorService.execute(new Runnable() {
            @Override
            public void run() {
                dao.delete(m);
            }
        });
    }

    //viewModel이 제거될 때 호출
    //view model은 액티비티와 생명주기를 같이 하기 때문에 액티비티가 종료될 때 호출된다.
    @Override
    protected void onCleared() {
        super.onCleared();
        database.close(); //사용한 db 닫기
        System.out.println("db close");
    }
}
```

</details>

> ViewModelTestActivity.java

<details>
<summary>코드보기</summary>

```java
package com.example.roomdbtest;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.lifecycle.MutableLiveData;
import androidx.lifecycle.Observer;
import androidx.lifecycle.ViewModelProvider;

import android.os.Bundle;
import android.os.Handler;
import android.os.Looper;
import android.os.Message;
import android.view.ContextMenu;
import android.view.MenuItem;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.EditText;
import android.widget.ListView;

import java.util.ArrayList;

public class ViewModelTestActivity extends AppCompatActivity {
    private EditText name_et;
    private EditText tel_et;
    private ListView listView;
    private ArrayAdapter < Member > adapter;
    private ArrayList < Member > list;
    private MemberViewModel db; //MemberDBAdapter 역할
    private Member m;
    private MutableLiveData < ArrayList < Member >> members; //전체 검색용
    private MutableLiveData < Member > member; //1명 검색용

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        name_et = findViewById(R.id.name_et);
        tel_et = findViewById(R.id.tel_et);
        listView = findViewById(R.id.lv1);
        list = new ArrayList < > ();

        db = new ViewModelProvider(this).get((MemberViewModel.class));
        db.create(this);
        members = db.getMembers();
        member = db.getMember();

        // 2개의 LiveData에 옵저버 부착
        members.observe(this, new Observer < ArrayList < Member >> () {
            @Override
            public void onChanged(ArrayList < Member > members) { //db의 getall을 호출하면, 결과를 postvalue로 받아옴
                list.clear();
                list.addAll(members);
                adapter.notifyDataSetChanged();
            }
        });
        member.observe(this, new Observer < Member > () { //db의 getmember를 호출하면, 결과를 postvalue로 받아옴
            @Override
            public void onChanged(Member member) {
                name_et.setText(member.name);
                tel_et.setText(member.tel);
                m = member;
            }
        });
        adapter = new ArrayAdapter(this, android.R.layout.simple_list_item_1, list);
        listView.setAdapter(adapter);
        registerForContextMenu(listView);
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView < ? > adapterView, View view, int i, long l) {
                db.getMember(list.get(i)._id);
            }
        });
    }

    public void onSave(View view) {
        String n = name_et.getText().toString();
        String t = tel_et.getText().toString();
        db.addMember(new Member(0, n, t));
        name_et.setText("");
        tel_et.setText("");
        db.getAll();
    }
    public void onEdit(View view) {
        if (m != null) {
            m.name = name_et.getText().toString();
            m.tel = tel_et.getText().toString();
            db.editMember(m);
            db.getAll();
            m = null;
            name_et.setText("");
            tel_et.setText("");
        }
    }

    //화면 시작되자마자 실행되는 코드
    @Override
    protected void onPostResume() {
        super.onPostResume();
        db.getAll(); //onCreate에 넣으면 싱크 안맞음
    }

    @Override
    public void onCreateContextMenu(ContextMenu menu, View v, ContextMenu.ContextMenuInfo menuInfo) {
        super.onCreateContextMenu(menu, v, menuInfo);
        menu.add(0, 1, 0, "edit");
        menu.add(0, 2, 0, "delete");
    }

    @Override
    public boolean onContextItemSelected(@NonNull MenuItem item) {
        AdapterView.AdapterContextMenuInfo info = (AdapterView.AdapterContextMenuInfo) item.getMenuInfo();
        int idx = info.position; //길게 누른 위치값
        m = list.get(idx);
        switch (item.getItemId()) {
            case 1:
                name_et.setText(m.name);
                tel_et.setText(m.tel);
                break;
            case 2:
                db.delMember(m);
                db.getAll();
                m = null;
                break;
        }
        return true;
    }
}
```
</details>

> build.gradle 수정
```gradle
//viewModel 의존성  
implementation "androidx.lifecycle:lifecycle-extensions:2.2.0"
```

> 실행화면 <br/>

![image](https://user-images.githubusercontent.com/62331803/91001712-70cf1300-e607-11ea-9890-e2d7a945ac6a.png)

