# 연습문제 | 전화번호부

### manifest.xml
- 전화 동작: manifest 파일에 CALL_PHONE permission 추가<br/>
```html
    <uses-permission android:name="android.permission.CALL_PHONE" />
```
### 0. activity_main.xml
![image](https://user-images.githubusercontent.com/62331803/90359334-9f8f3b80-e093-11ea-9d4f-344d478e318a.png)

### 0.1. MainActivity
```java
package com.example.phonebook;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

import android.content.Intent;
import android.os.Bundle;
import android.view.ContextMenu;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.Toast;

import com.example.phonebook.model.Member;

import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {
    private ArrayList<Member> datas;
    private phoneAdapter pa;//여러개의 view를 담을 수 있는 컨테이너 **읽어와서 view 객체 생성해주는 중간단계 역할
    private ListView listView;
    private Intent intent;
    private int idx;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        listView = findViewById(R.id.lv1);

        datas = new ArrayList<>();
        pa = new phoneAdapter(this, R.layout.item_layout, datas);//this: activity(application을 상속-context를 상속)
        listView.setAdapter(pa);//어댑터 연결: listView는 화면을 출력할 때 어댑터로부터 데이터를 받아서 해당 데이터를 화면에 출력
        //이벤트 리스너: 이벤트가 발생했는가 기다렸다가 발생하면 핸들러를 호출
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {//리스터를 listview에 부착
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int i, long l) {//i: 인덱스
                Member m  = datas.get(i);
                String s = m.getName() + " / " + m.getTel();
                Toast.makeText(MainActivity.this, s ,Toast.LENGTH_SHORT).show();
            }
        });
        registerForContextMenu(listView);//리스트 뷰에 콘텍스트 메뉴 붙이기
    }

    //options menu 생성 *context menu와 별개의 개념
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        menu.add(0,Menu.FIRST,0,"add"); //Menu.FIRST==1
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(@NonNull MenuItem item) {
        switch(item.getItemId()){
            case 1:
                System.out.println("add 메뉴로 이동");
                Intent intent = new Intent(this, AddActivity.class);//명시적으로 activity 활성화
                startActivityForResult(intent,1);
                break;
        }
        return true;
    }

    @Override
    public void onCreateContextMenu(ContextMenu menu, View v, ContextMenu.ContextMenuInfo menuInfo) {
        super.onCreateContextMenu(menu, v, menuInfo);
        super.onCreateContextMenu(menu, v, menuInfo);
        menu.setHeaderTitle("UPDATE");//MENU TITLE 설정
        menu.add(0,Menu.FIRST,0,"edit");
        menu.add(0,Menu.FIRST+1,0,"delete");
    }

    @Override
    public boolean onContextItemSelected(@NonNull MenuItem item) {
        AdapterView.AdapterContextMenuInfo info = (AdapterView.AdapterContextMenuInfo) item.getMenuInfo();
        idx = info.position;
        switch(item.getItemId()){
            case 1://수정
                Intent intent = new Intent(this, EditActivity.class);//명시적으로 activity 활성화
                intent.putExtra("m",datas.get(idx));
                startActivityForResult(intent,2);
                break;
            case 2://삭제
                datas.remove(idx);
                pa.notifyDataSetChanged();//어댑터에 갱신 명령
                break;
        }
        return true;
    }

    public void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (resultCode == RESULT_OK) {
            switch (requestCode) {
                case 1:
                    Member m = (Member) data.getSerializableExtra("m");
                    if(m!=null){
                        System.out.println(m);
                        datas.add(m);
                        pa.notifyDataSetChanged();//뷰 갱신
                    }else{
                        Toast.makeText(MainActivity.this,"추가정보없음",Toast.LENGTH_SHORT).show();
                    }
                    break;
                case 2:
                    Member m2 = (Member) data.getSerializableExtra("m");
                    if(m2!=null){
                        datas.set(idx, m2);
                        pa.notifyDataSetChanged();//뷰 갱신
                    }else{
                        Toast.makeText(MainActivity.this,"수정정보없음",Toast.LENGTH_SHORT).show();
                    }
                    break;
            }
        }
    }

}
```
- 버튼 동작: descendant focusable => block설정 <br/>

### 0.2. item_layout.xml
![image](https://user-images.githubusercontent.com/62331803/90359382-c51c4500-e093-11ea-913f-5efd085a570f.png)

### 0.3. phoneAdapter
```java
package com.example.phonebook;

import android.content.Context;
import android.content.Intent;
import android.net.Uri;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;

import com.example.phonebook.model.Member;

import java.util.ArrayList;
import java.util.List;

public class phoneAdapter extends ArrayAdapter<Member> {
    private Context context;
    private ArrayList<Member> list;
    private int resId;

    public phoneAdapter(@NonNull Context context, int resource, @NonNull List<Member> objects) {
        super(context, resource, objects);
        this.context = context;
        this.list = (ArrayList<Member>) objects;
        resId = resource;
    }

    @NonNull
    @Override
    //getview: 어댑터에서 가장 중요한 작업! 데이터 위치(position)를 읽어와서 리소스로 지정한 뷰로 생성하여 반환
    //list의 요소개수만큼 호출되는 함수
    public View getView(int position, @Nullable View convertView, @NonNull ViewGroup parent) {
        View itemView = convertView;
        if(itemView==null){            //뷰가 없다면 생성
            LayoutInflater vi = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);//LayoutInflater: xml등의 리소스를 실제 뷰로 부풀리는 역할을 하는 객체
            itemView = vi.inflate(resId,null);//설계한 자료의 id를 받아서 해당 id로 실제 뷰를 생성.(item_layout.xml)
        }
        final Member m = list.get(position);//현재 위치의 멤버 객체 추출
        if(m != null) {
            TextView t1 = itemView.findViewById(R.id.textView);//이름 tv
            TextView t2 = itemView.findViewById(R.id.textView2);//전화번호 tv
            ImageView imgv = itemView.findViewById(R.id.imageView);//이미지
//            imgv.setImageResource(R.drawable.ic_launcher_foreground);//임시로 고정값 사용.

            Button sms = itemView.findViewById(R.id.button17);
            Button call = itemView.findViewById(R.id.button19);

            //뷰에 텍스트 세팅
            if(t1!=null){
                t1.setText(m.getName());
            }
            if(t2!=null){
                t2.setText(m.getTel());
            }
            if(imgv!=null){
                imgv.setImageResource(m.getImgRes());
            }

            call.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View view) {
                    //action_call: 시스템에서 제공하는 것이기 때문에 permission 설정 필요 => manifestAndroid
                    Intent intent = new Intent(Intent.ACTION_DIAL, Uri.parse("tel:"+m.getTel()));//묵시적으로 activity 활성화
                    context.startActivity(intent);
                }
            });
        }
        return itemView;
    }
}
```

### 1. activity_add.xml
![image](https://user-images.githubusercontent.com/62331803/90350020-bffccd00-e076-11ea-94e5-1712eaf6b322.png)
- 가운데 정렬 (gravity 속성 변경)<br/>
![image](https://user-images.githubusercontent.com/62331803/90350094-eae72100-e076-11ea-978c-99890e956d95.png)

### 1.1. AddActivity
```java
package com.example.phonebook;

import androidx.appcompat.app.AppCompatActivity;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.EditText;
import android.widget.ImageView;
import android.widget.RadioGroup;
import android.widget.Spinner;
import android.widget.Toast;

import com.example.phonebook.model.Member;

import java.util.ArrayList;

public class AddActivity extends AppCompatActivity {
    private Intent intent;
    private EditText name;
    private EditText tel;
    private Spinner spinner;
    private ImageView imageview;
    private ArrayList<String> img_names;
    private ArrayList<Integer> imgs;
    private ArrayAdapter<String> aa;
    private RadioGroup rg;
//    private int phoneType;
    private Member member;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_add);
        intent = getIntent();
        name = findViewById(R.id.name);//이름
        tel = findViewById(R.id.tel);//전화번호
        spinner = findViewById(R.id.spinner);
        imageview = findViewById(R.id.imageView);

        member = new Member();
        member.setImgRes(R.drawable.img1);
        member.setPhoneType(1);

        imgs = new ArrayList<>();
        imgs.add(R.drawable.img1);
        imgs.add(R.drawable.img2);
        imgs.add(R.drawable.img3);
        imgs.add(R.drawable.img4);
        imgs.add(R.drawable.img5);
        imgs.add(R.drawable.img6);

        img_names = new ArrayList<>();
        img_names.add("img1");
        img_names.add("img2");
        img_names.add("img3");
        img_names.add("img4");
        img_names.add("img5");
        img_names.add("img6");

        aa = new ArrayAdapter<>(this,android.R.layout.simple_spinner_item,img_names);
        spinner.setAdapter(aa);
        spinner.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> adapterView, View view, int i, long l) {
                imageview.setImageResource(imgs.get(i));
                member.setImgRes(imgs.get(i));
            }

            @Override
            public void onNothingSelected(AdapterView<?> adapterView) {
                Toast.makeText(AddActivity.this,"선택된 항목 없음", Toast.LENGTH_SHORT).show();
            }
        });

        rg = findViewById(R.id.rg);
        rg.setOnCheckedChangeListener(new RadioGroup.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(RadioGroup radioGroup, int i) {
                int phoneType = 1;
                switch(i){
                    case R.id.rg://휴대폰
                        phoneType = 1;
                        break;
                    case R.id.rb2://집
                        phoneType = 2;
                        break;
                    case R.id.rb3://직장
                        phoneType = 3;
                        break;
                }
            }
        });
    }
            public void onSave(View view) {
                member.setName(name.getText().toString());
                member.setTel(tel.getText().toString());
                intent.putExtra("m",member);
                setResult(RESULT_OK,intent);
                finish();
            }

}
```
### 2. activity_edit.xml
![image](https://user-images.githubusercontent.com/62331803/90359231-4e7f4780-e093-11ea-8044-f53eba4ff851.png)

### 2.1. EditActivity
```java
package com.example.phonebook;

import androidx.appcompat.app.AppCompatActivity;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.EditText;
import android.widget.ImageView;
import android.widget.RadioGroup;
import android.widget.Spinner;
import android.widget.Toast;

import com.example.phonebook.model.Member;

import java.util.ArrayList;

public class EditActivity extends AppCompatActivity {
    private EditText name_et;
    private EditText tel_et;
    private Intent intent;
    private Spinner spinner;
    private ImageView imageview;
    private ArrayList<String> img_names;
    private ArrayList<Integer> imgs;
    private ArrayAdapter<String> aa;
    private RadioGroup rg;
    private Member member;
    private int phonetype = 1;
    private boolean flag = true;
    private boolean flag2 = true;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_edit);

        intent = getIntent();
        member = (Member) intent.getSerializableExtra("m");

        name_et = findViewById(R.id.name);
        name_et.setText(member.getName());
        tel_et = findViewById(R.id.tel);
        tel_et.setText(member.getTel());
        spinner = findViewById(R.id.spinner);

        imageview = findViewById(R.id.imageView);
        imageview.setImageResource(member.getImgRes());

        imgs = new ArrayList<>();
        imgs.add(R.drawable.img1);
        imgs.add(R.drawable.img2);
        imgs.add(R.drawable.img3);
        imgs.add(R.drawable.img4);
        imgs.add(R.drawable.img5);
        imgs.add(R.drawable.img6);

        img_names = new ArrayList<>();
        img_names.add("img1");
        img_names.add("img2");
        img_names.add("img3");
        img_names.add("img4");
        img_names.add("img5");
        img_names.add("img6");

        aa = new ArrayAdapter<>(this,android.R.layout.simple_spinner_item,img_names);
        spinner.setAdapter(aa);
        spinner.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> adapterView, View view, int i, long l) {
                if(flag){
                    flag = false;
                }
                else{
                    imageview.setImageResource(imgs.get(i));
                    member.setImgRes(imgs.get(i));
                }
            }

            @Override
            public void onNothingSelected(AdapterView<?> adapterView) {
                Toast.makeText(EditActivity.this,"선택된 항목 없음", Toast.LENGTH_SHORT).show();
            }
        });

        rg = findViewById(R.id.rg);
        rg.setOnCheckedChangeListener(new RadioGroup.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(RadioGroup radioGroup, int i) {
                    switch(i){
                        case 1://휴대폰
                            phonetype = 1;
                            break;
                        case 2://집
                            phonetype = 2;
                            break;
                        case 3://직장
                            phonetype = 3;
                            break;
                    }
                }
        });
    }

    public void onGoBack(View view){
        member.setName(name_et.getText().toString());
        member.setTel(tel_et.getText().toString());
        member.setPhoneType(phonetype);
        intent.putExtra("m",member);
        setResult(RESULT_OK,intent);//resultCode, data
        finish();
    }
}
```

### 결과
![image](https://user-images.githubusercontent.com/62331803/90359512-11678500-e094-11ea-8f64-aa7b1ce5f9b8.png)
<br/>
![image](https://user-images.githubusercontent.com/62331803/90359535-1fb5a100-e094-11ea-836c-403447aef223.png)
<br/>
![image](https://user-images.githubusercontent.com/62331803/90359547-280ddc00-e094-11ea-8d19-28926da56e9f.png)
<br/>
![image](https://user-images.githubusercontent.com/62331803/90359556-2e9c5380-e094-11ea-9b3d-2772c10c8a01.png)
<br/>
![image](https://user-images.githubusercontent.com/62331803/90359570-3b20ac00-e094-11ea-84a4-5c9108dc03c9.png)
<br/>
![image](https://user-images.githubusercontent.com/62331803/90359585-4247ba00-e094-11ea-9743-4645b290626f.png)
<br/>
![image](https://user-images.githubusercontent.com/62331803/90359596-48d63180-e094-11ea-949f-c92c881f9dc8.png)
<br/>
![image](https://user-images.githubusercontent.com/62331803/90359600-4d024f00-e094-11ea-9ff5-b9b6fa999ad9.png)
<br/>
