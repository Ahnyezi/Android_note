## 0819 연습문제 | 메세지 수신함 (디버깅 완료)

> Sms.java
```java
package com.example.receivertest;

public class Sms {
    private String phone;
    private String msg;

    public Sms() {
    }

    public Sms(String phone, String msg) {
        this.phone = phone;
        this.msg = msg;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    @Override
    public String toString() {
        return "(" + phone + ")         "+
                 msg ;
    }
}

```


> activity_main.xml <br/>
![image](https://user-images.githubusercontent.com/62331803/90599562-0b0e1000-e230-11ea-975f-2bc141936b60.png)

> MainActivity.java
```java
package com.example.receivertest;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.Manifest;
import android.app.AlertDialog;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.DialogInterface;
import android.content.Intent;
import android.content.IntentFilter;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.Bundle;
import android.telephony.SmsMessage;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.Toast;

import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {
    private ArrayList<Sms> msgs = new ArrayList<>();;
    private ArrayAdapter<Sms> aa;
    private ListView listView;
    private Intent intent;
    private String msg;
    private String sender;

    //각 리시버를 깨울 상수값
    private final static String SEND_ACTION = "SENT";
    private final static String DELIVER_ACTION = "DELIVERED";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        listView = findViewById(R.id.lv1);

        aa = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1,msgs);
        listView.setAdapter(aa);

        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int i, long l) {
                // Toast로 sender와 msg 띄우기 *** 수정
                Sms msg = msgs.get(i);
                Toast.makeText(MainActivity.this, msg.toString(), Toast.LENGTH_SHORT).show();
            }
        });

        // 새로운 브로드캐스트 리시버 생성해서 사용하기
        BroadcastReceiver smsRecv = new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                // an Intent broadcast.
                Bundle bundle = intent.getExtras();
//                String msg;
//                String sender;

                int i;
                if (bundle != null) {
                    Object[] rawData = (Object[]) bundle.get("pdus");
                    SmsMessage[] sms = new SmsMessage[rawData.length];

                    for (i = 0; i < rawData.length; i++) {
                        sms[i] = SmsMessage.createFromPdu((byte[]) rawData[i]);
                    }

                    for (i = 0; i < sms.length; i++) {
                        msg = sms[i].getMessageBody();
                        sender = sms[i].getOriginatingAddress();

                        AlertDialog.Builder builder = new AlertDialog.Builder(MainActivity.this);
                        builder.setMessage(msg + "\nfrom: " + sender)
                                .setPositiveButton("OK", new DialogInterface.OnClickListener() {
                                    @Override
                                    public void onClick(DialogInterface dialogInterface, int i) {
                                        msgs.add(new Sms(sender,msg));
                                        aa.notifyDataSetChanged();
                                        dialogInterface.cancel();//다이얼로그 창 닫음
                                        //finish(): 액티비티 종료
                                    }
                                })
                                .setTitle("You've got Message!");
                        AlertDialog alertDialog = builder.create();
                        alertDialog.show();
                    }
                }
            }
        };
        IntentFilter f3 = new IntentFilter();
        f3.addAction("android.provider.Telephony.SMS_RECEIVED");//인텐트 필터 생성.
        registerReceiver(smsRecv,f3);//브로드캐스트 리시버에 해당 필터 등록.

        // intent filter 설정
        SMSSendResult send = new SMSSendResult();
        IntentFilter f1 = new IntentFilter();
        f1.addAction(SEND_ACTION);

        SMSReceiveResult recv = new SMSReceiveResult();
        IntentFilter f2 = new IntentFilter();
        f2.addAction(DELIVER_ACTION);

        registerReceiver(send,f1);
        registerReceiver(recv,f2);

        if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.M)
        {
            // For device above MarshMallow
            boolean permission = getSMSPermission();
            if(permission) {
//                Toast.makeText(this,"sms 권한 획득", Toast.LENGTH_SHORT).show();
            }
        }
        else{
            Toast.makeText(this,"권한 획득이 필수가 아닌 버전", Toast.LENGTH_SHORT).show();
        }
    }

    public boolean getSMSPermission(){
        boolean hasPermission = ((ContextCompat.checkSelfPermission(this,
                Manifest.permission.SEND_SMS) == PackageManager.PERMISSION_GRANTED) &&
                (ContextCompat.checkSelfPermission(this,
                        Manifest.permission.RECEIVE_SMS) == PackageManager.PERMISSION_GRANTED));
        if (!hasPermission) {
            ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.SEND_SMS,
                    Manifest.permission.RECEIVE_SMS}, 1);
        }
        return hasPermission;
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode)
        {
            case 1: {
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED)
                {
//                    Toast.makeText(this, "sms 권한 부여", Toast.LENGTH_SHORT).show();
                }
            }
        }
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        menu.add(0,Menu.FIRST,0,"New Message");
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(@NonNull MenuItem item) {
        switch(item.getItemId()){
            case 1:
                //메세지 입력창으로 이동
                intent = new Intent(this, SMSActivity.class);
                startActivityForResult(intent,1); //startActivity() X
                break;
        }
        return true;
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if(resultCode==RESULT_OK){
            switch(requestCode){
                case 1:
                    Toast.makeText(this,"전송완료",Toast.LENGTH_SHORT).show();
                    break;
            }
        }

    }
}
```

> activity_s_m_s.xml <br/>

![image](https://user-images.githubusercontent.com/62331803/90599668-398beb00-e230-11ea-93c1-cb1cbf964c49.png)


> SMSActivity.java
```java
package com.example.receivertest;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.Manifest;
import android.app.Activity;
import android.app.AlertDialog;
import android.app.PendingIntent;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.DialogInterface;
import android.content.Intent;
import android.content.IntentFilter;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.Bundle;
import android.telephony.SmsManager;
import android.telephony.SmsMessage;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

public class SMSActivity extends AppCompatActivity {
    private EditText msg_et;
    private EditText phone_et;

    //각 리시버를 깨울 상수값
    private final static String SEND_ACTION = "SENT";
    private final static String DELIVER_ACTION = "DELIVERED";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_s_m_s);
        msg_et = findViewById(R.id.msg);
        phone_et = findViewById(R.id.phone);
    }

    public void onSend(View view){
        String msg = msg_et.getText().toString();
        String phone = phone_et.getText().toString();

        // 둘 중 하나라도 내용이 없다면, 내용입력 받게끔 설정
        if(msg.length() <=0 || phone.length() <=0){
            Toast.makeText(getApplicationContext(),
                    "input the message and phone number!",
                    Toast.LENGTH_SHORT).show();
            return;
        }

        //sms 보내는 쪽 상태확인을 위한 객체
        PendingIntent sendStat = PendingIntent.getBroadcast(this,0,
                new Intent("SENT"),0);// context, 확인 코드, 리시버를 깨울 인텐트, flag
        //sms 받는 쪽 상태확인을 위한 객체
        PendingIntent deliveredStat = PendingIntent.getBroadcast(this,0,
                new Intent("DELIVERED"),0);

        SmsManager sms = SmsManager.getDefault();

        //sms 전송
        sms.sendTextMessage(phone, null, msg, sendStat, deliveredStat);
        setResult(RESULT_OK);//보낼 데이터가 없다면 intent 보내지 않음.
        finish();//현재 activity 종료.
    }
}
```

> 결과화면 <br/>

![image](https://user-images.githubusercontent.com/62331803/90704320-237f3880-e2cb-11ea-9c38-abfb2c7f9332.png)

<br/>

> 피드백
#### 1 | New Message로 이동시 startActivity()로 intent 활성화
- startActivityForResult()로 활성화해야함.
- startActivity()는 이동할 때마다 매번 새로운 액티비티를 만드는 것이기 때문에
- 데이터가 매번 갱신된다.
- msgs 배열 안의 요소를 초기화 시키지 않기 위해서는 forResult 사용
#### 2 | startActivityForResult로 이동한 activity에서 데이터를 보낼 필요 없는 경우
- getIntent 필요 없음
- setResult에 intent 넣을 필요 없음
- 예시
```java
        setResult(RESULT_OK);//보낼 데이터가 없다면 intent 보내지 않음.
        finish();//현재 activity 종료.
```
