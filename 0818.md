# Activity(GridView), Service

### 1. GridView 
> GridView 기본
1. activity_main7
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity7">

    <GridView
        android:layout_width="match_parent" 
        android:layout_height="match_parent"
        android:columnWidth="90dp"  //각 컬럼의 너비
        android:gravity="center"    //칸 가운데 정렬
        android:horizontalSpacing="10dp" //수평방향 띄어쓰기
        android:numColumns="auto_fit" //
        android:stretchMode="columnWidth" //
        android:verticalSpacing="10dp" />
</LinearLayout>
```
![image](https://user-images.githubusercontent.com/62331803/90458865-868d9580-e13a-11ea-8741-ac21d2e4726e.png)

1.2. MainActivity7
```java
package com.example.myapplication;

import androidx.appcompat.app.AppCompatActivity;

import android.content.Context;
import android.os.Bundle;
import android.view.View;
import android.view.ViewGroup;
import android.widget.AdapterView;
import android.widget.BaseAdapter;
import android.widget.GridView;
import android.widget.ImageView;
import android.widget.Toast;

public class MainActivity7 extends AppCompatActivity {
    private GridView gv;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main7);
        gv = findViewById(R.id.grid);

        //어뎁터 생성하여 그리드뷰와 연결
        gv.setAdapter(new ImageAdapter(this));

        //개별 이미지 클릭이벤트 함수 지정
        AdapterView.OnItemClickListener m = new AdapterView.OnItemClickListener(){

            //클릭한 이미지의 위치(인덱스) 출력
            public void onItemClick(AdapterView<?> arg0, View arg1,int arg2, long arg3){//gridview, 개별 view, idx, id(반환 안함)
                Toast.makeText(MainActivity7.this, arg2+"", Toast.LENGTH_SHORT).show();
            }
        };

        // 그리드뷰에 생성한 클릭 리스너 설정
        gv.setOnItemClickListener(m);
    }
}

/*
* BaseAdapter?
모든 어뎁터의 부모 클래스
* Adapter란
데이터와 뷰를 실제적으로 연결. 데이터를 담은 뷰를 반환하는 역할.
* phoneAdapter에서 사용한 arrayAdapter 또한 BaseAdapter를 상속받은 구현 클래스.
하지만, 이미 구현이 되어있기 때문에 BaseAdapter를 직접 상속 받은 것보다 완성도 있음.
BaseAdapter를 바로 상속받을 경우 추상메서드 구현 필요
*/

//BaseAdapter를 상속받는 ImageAdapter 클래스 선언
class ImageAdapter extends BaseAdapter{
    private Context mContext;
    //이미지 뷰에 출력할 리소스 id들
    private Integer[] Idata = {R.drawable.img1,R.drawable.img2,R.drawable.img3,R.drawable.img4,R.drawable.img5,R.drawable.img6};

    public ImageAdapter(Context c){
        mContext = c ;
    }
    //idata의 요소 개수 반환
    @Override
    public int getCount() {
        return Idata.length;
    }
    //특정 위치에 있는 데이터 반환
    @Override
    public Object getItem(int i) {
        return null;
    }
    //특정 위치에 있는 데이터의 id 반환
    @Override
    public long getItemId(int i) {
        return 0;
    }

    //특정 위치의 데이터를 출력할 뷰를 get
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {//처리할 데이터 idx, 데이터를 출력할 view, 부모 뷰 객체(grid view)
        ImageView imageView;
        if(convertView == null){
            imageView = new ImageView(mContext);

            //그리드뷰 세부속성 설정
            imageView.setLayoutParams(new GridView.LayoutParams(200,200));//가로 세로 길이
            imageView.setScaleType(ImageView.ScaleType.CENTER_CROP);//정렬 방법
            imageView.setPadding(8,8,8,8);
        } else{
            imageView = (ImageView) convertView;
        }
        imageView.setImageResource(Idata[position]);
        return imageView;
    }
}
```
* 결과 화면<br/>
![image](https://user-images.githubusercontent.com/62331803/90459815-55629480-e13d-11ea-87c0-952acdd78364.png)
<br/>

> GridView 연습문제: 이미지 위치 변경하기
```java
package com.example.myapplication;

import androidx.appcompat.app.AppCompatActivity;

import android.content.Context;
import android.os.Bundle;
import android.view.View;
import android.view.ViewGroup;
import android.widget.AdapterView;
import android.widget.BaseAdapter;
import android.widget.GridView;
import android.widget.ImageView;
import android.widget.Toast;

public class MainActivity7 extends AppCompatActivity {
    private GridView gv;
    private ImageAdapter imageAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main7);
        gv = findViewById(R.id.grid);

        //어뎁터 생성하여 그리드뷰와 연결
        imageAdapter = new ImageAdapter(this);
        gv.setAdapter(imageAdapter);

        //개별 이미지 클릭이벤트 함수 지정
        AdapterView.OnItemClickListener m = new AdapterView.OnItemClickListener(){

            public void onItemClick(AdapterView<?> arg0, View arg1,int arg2, long arg3){//gridview, 개별 view, idx, id(반환 안함)
                Toast.makeText(MainActivity7.this, arg2+"", Toast.LENGTH_SHORT).show();
                imageAdapter.set_idx(arg2);//클릭한 위치 idx 세팅. 조건 만족하면 changeImg() 호출
                //notify는 changeimg에서 호출
            }

        };

        gv.setOnItemClickListener(m);
    }
}

class ImageAdapter extends BaseAdapter{
    private Context mContext;
    private Integer[] Idata = {R.drawable.img1,R.drawable.img2,R.drawable.img3,R.drawable.img4,R.drawable.img5,R.drawable.img6};
    private int idx1; //첫번째 클릭시 idx값 저장
    private int idx2; //두번째 클릭시 idx값 저장

    public void init_idx(){ //idx 초기화(이미지 변경 이후)
        idx1 = -1;
        idx2 = -1;
    }

    public void set_idx(int idx){//클릭된 idx를 변수에 저장
        if(idx1==-1){idx1=idx;}
        else{idx2=idx;changeImg();}
    }

    public void changeImg(){//이미지 위치 변경(Idata 배열 변경)
        int tmp;
        tmp = Idata[idx1];
        Idata[idx1] = Idata[idx2];
        Idata[idx2] = tmp;
        init_idx();
        notifyDataSetChanged();//화면 갱신
    }

    public ImageAdapter(Context c){
        mContext = c ;
    }

    public Integer[] getIdata() {
        return Idata;
    }

    public void setIdata(Integer[] idata) {
        Idata = idata;
    }

    @Override
    public int getCount() {
        return Idata.length;
    }

    @Override
    public Object getItem(int i) {
        return null;
    }

    @Override
    public long getItemId(int i) {
        return 0;
    }

    //특정 위치의 데이터를 출력할 뷰를 get
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {//처리할 데이터 idx, 데이터를 출력할 view, 부모 뷰 객체(grid view)
        ImageView imageView;
        if(convertView == null){
            imageView = new ImageView(mContext);

            //그리드뷰 세부속성 설정
            imageView.setLayoutParams(new GridView.LayoutParams(200,200));//가로 세로 길이
            imageView.setScaleType(ImageView.ScaleType.CENTER_CROP);//정렬 방법
            imageView.setPadding(8,8,8,8);
        } else{
            imageView = (ImageView) convertView;
        }
        imageView.setImageResource(Idata[position]);
        return imageView;
    }
}
```
* 시작 화면<br/>
![image](https://user-images.githubusercontent.com/62331803/90460122-39abbe00-e13e-11ea-82c9-86393ed5530e.png)

* 조작 이후 화면<br/>
![image](https://user-images.githubusercontent.com/62331803/90460138-47f9da00-e13e-11ea-8021-71999436db43.png)


### 2. Intent 활성화
> 2-a. 명시적 활성화
* 클래스 이름으로 직접 활성화

> 2-b. 묵시적 활성화
* manifest.xml에 묵시적으로 활성화할 intent에 filter 내용을 기재
* intent filter: 해당 intent가 활성화될 조건을 미리 지정
* 활성화 조건: action(필수), data, category 
* action : 동작을 문자열로 나타낸 이름
* data : 실행하는데에 필요한 데이터
* category : 기능별로 카테고리를 지정 

* 안드로이드 내에서 동작하는 파일들 확장자명은 .apk 
* apk명 앞에는 항상 패키지가 쓰임
* 따라서 패키지 이름으로 파일들을 구별

### 3. 메인 컴포넌트2 | Service
* ui없이 동작하는 컴포넌트
* 백그라운드 작업
* ex. 음악 재생기

* 스타트 모드: 서비스 생성완료 후, 타 컴포넌트에서 서비스를 부를 때
<br/>

![image](https://user-images.githubusercontent.com/62331803/90461841-5649f500-e142-11ea-8d5a-e1a5ab8a8ad4.png)

<br/>
* 바인드 모드: 서비스 생성 전, 타 컴포넌트에서 서비스를 부를 때
<br/>

![image](https://user-images.githubusercontent.com/62331803/90462477-f05e6d00-e143-11ea-9056-ac57bc319b82.png)

<br/>
###### start mode와 bind mode 차이
##### start mode
* 서비스가 이미 구현된 상태에서 타 컴포넌트에서 호출
* stopservice 호출해서 서비스 사용 종료
##### bind: 아직 구현되지 않은 상태에서 타 컴포넌트에서 binding 하는 경우
* binding된 컴포넌트 없으면 자동으로 stop됨 
<br/>

![image](https://user-images.githubusercontent.com/62331803/90462571-0d933b80-e144-11ea-92b4-9c71a3fafb39.png) 

<br/>

![image](https://user-images.githubusercontent.com/62331803/90462788-62cf4d00-e144-11ea-8f4f-956529e4e9ea.png)

<br/> 

> service 기능 사용예제

##### - START ->BIND -> UNBIND -> STOP
##### - BIND -> UNBIND : ONCREATE와 ONDESTROY 자동 호출

* a. mainActivity
```java
package com.example.myapp2;

import androidx.appcompat.app.AppCompatActivity;

import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Bundle;
import android.os.IBinder;
import android.view.View;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
    //서비스 시작
    public void onStart(View view){
        Intent intent = new Intent(this,MyService.class);
        startService(intent);
    }
    //추상메서드를 가진 service connection 객체 생성.
    ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
            Toast.makeText(MainActivity.this,"activity에서 service 연결됨", Toast.LENGTH_SHORT).show();
        }

        @Override
        public void onServiceDisconnected(ComponentName componentName) {

        }
    };

    public void onBind(View view){
        Intent intent = new Intent(this, MyService.class);
        bindService(intent,connection, Context.BIND_AUTO_CREATE);//현재 activity와 서비스를 연결
    }

    public void onUnbind(View view){
        unbindService(connection);
    }
    //현재 service 종료
    public void onStop(View view){
        Intent intent = new Intent(this, MyService.class);
        stopService(intent);
    }




}
```
* b. myservice
```java
package com.example.myapp2;

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.widget.Toast;

public class MyService extends Service {

    public MyService() {
    }

    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        Toast.makeText(this,"onBind()",Toast.LENGTH_SHORT).show();
        return null;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        Toast.makeText(this,"onCreate()",Toast.LENGTH_SHORT).show();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Toast.makeText(this,"onStart()",Toast.LENGTH_SHORT).show();
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Toast.makeText(this,"onDestroy()",Toast.LENGTH_SHORT).show();
    }

    @Override
    public boolean onUnbind(Intent intent) {
        Toast.makeText(this,"onUnbind()",Toast.LENGTH_SHORT).show();
        return super.onUnbind(intent);
    }
}

```

> 실습예제 : 볼륨 조절 기능
* activity_main.xml<br/>
![image](https://user-images.githubusercontent.com/62331803/90467887-391c2300-e150-11ea-9b12-f906a6572f50.png)

* MainActivity
```java
package com.example.myapp2;

import androidx.appcompat.app.AppCompatActivity;

import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Bundle;
import android.os.IBinder;
import android.view.View;
import android.widget.ProgressBar;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {
    private ProgressBar progressBar;
    private MyService.MyBinder mm;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        progressBar = findViewById(R.id.progressBar);
//        progressBar.setProgress(50);// progressbar 수치 50으로 설정
//        int x = progressBar.getProgress();// progressbar 수치 읽어옴
    }
    //서비스 시작
    public void onStart(View view){
        Intent intent = new Intent(this,MyService.class);
        startService(intent);
    }
    //추상메서드를 가진 service connection 객체 생성.
    ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {//iBinder 받는 역할
            mm = (MyService.MyBinder) iBinder;
            int v = mm.getVolumn();
            progressBar.setProgress(v);//기본 볼륨값 적용
    }

        @Override
        public void onServiceDisconnected(ComponentName componentName) {

        }
    };

    public void onBind(View view){
        Intent intent = new Intent(this, MyService.class);
        bindService(intent,connection, Context.BIND_AUTO_CREATE);//현재 activity와 서비스를 연결
    }

    public void onUnbind(View view){
        unbindService(connection);
        mm = null;
    }
    //현재 service 종료
    public void onStop(View view){
        Intent intent = new Intent(this, MyService.class);
        stopService(intent);
    }

    public void onVolumnUp(View view){
        if(mm==null){
            Toast.makeText(this,"bind 되지 않았습니다.", Toast.LENGTH_SHORT).show();
            return;
        }
        progressBar.setProgress(mm.volumeUp());
    }

    public void onVolumnDown(View view){
        if(mm==null){
            Toast.makeText(this,"bind 되지 않았습니다.", Toast.LENGTH_SHORT).show();
            return;
        }
        progressBar.setProgress(mm.volumnDown());
    }
}
```

* MyService
```java
package com.example.myapp2;

import android.app.Service;
import android.content.Intent;
import android.os.Binder;
import android.os.IBinder;
import android.widget.Toast;

public class MyService extends Service {
    private int volumn = 50;//0~100. 기본값 50
    // Binder: RMI 기능을 구현한 클래스
    // JAVA RMI란? Java Remote Method Invocation (자바 원격 함수 호출)
    class MyBinder extends Binder{
        int getVolumn(){
            return volumn;
        }
        //volumeUp()//볼륨10증가. 100못넘음. 현재 볼륨값 반환
        int volumeUp(){
            volumn+=10;
            if(volumn>100){volumn=100;}
            return volumn;
        }
        //volumeDown()//볼륨10감소.0못넘음. 현재 볼륨값 반환
        int volumnDown(){
            volumn-=10;
            if(volumn<0){volumn=0;}
            return volumn;
        }
    }

    public MyService() {
    }

    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        Toast.makeText(this,"onBind()",Toast.LENGTH_SHORT).show();
        return new MyBinder();
    }

    @Override
    public void onCreate() {
        super.onCreate();
        Toast.makeText(this,"onCreate()",Toast.LENGTH_SHORT).show();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Toast.makeText(this,"onStart()",Toast.LENGTH_SHORT).show();
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Toast.makeText(this,"onDestroy()",Toast.LENGTH_SHORT).show();
    }

    @Override
    public boolean onUnbind(Intent intent) {
        Toast.makeText(this,"onUnbind()",Toast.LENGTH_SHORT).show();
        return super.onUnbind(intent);
    }
}

```
* 결과 화면<br/>
![image](https://user-images.githubusercontent.com/62331803/90467968-6a94ee80-e150-11ea-9ee4-d9498bce0ae5.png)




