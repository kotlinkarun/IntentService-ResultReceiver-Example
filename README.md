# IntentService-ResultReceiver-Example


package com.demo;

import android.content.Intent;
import android.os.Bundle;
import android.os.Handler;
import android.os.ResultReceiver;
import android.support.annotation.Nullable;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;
import android.view.View;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {


    public static String DOWNLOAD_ACTION = "com.e";
    public static int RESULT_CODE = 1;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

    }


    public void btn(View v) {
        Intent intent = new Intent(MainActivity.this, UploadIntentService.class);
        intent.setAction(DOWNLOAD_ACTION);
        intent.putExtra("receiver", new MyResultReceiver(new Handler()));
        startService(intent);
    }


    class MyResultReceiver extends ResultReceiver {
        MyResultReceiver(Handler handler) {
            super(handler);
        }

        @Override
        protected void onReceiveResult(int resultCode, Bundle resultData) {
            if (resultCode == RESULT_CODE) {
                String message = resultData.getString("res");
                Toast.makeText(getApplicationContext(),message,Toast.LENGTH_SHORT).show();
                Log.d("TAGS",message);
            }
        }
    }
}


//////////////////////////////////////////////////////////////////////

Service class


package com.demo;

import android.app.IntentService;
import android.content.Intent;
import android.os.Bundle;
import android.os.ResultReceiver;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.URL;

public class UploadIntentService extends IntentService {

    ResultReceiver resultReceiver;
    public UploadIntentService() {
        super("UploadIntentService");
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        resultReceiver = intent.getParcelableExtra("receiver");
        if (intent.getAction().equals(MainActivity.DOWNLOAD_ACTION)) {

            try {

            URL  url = new URL("https://raw.githubusercontent.com/kotlinkarun/api/master/.gitignore/emp.json");
            BufferedReader reader=new BufferedReader(new InputStreamReader(url.openStream()));
            String s;
            StringBuilder builder=new StringBuilder();
            while ((s=reader.readLine())!= null){
                builder.append(s);
            }
            Bundle b = new Bundle();
            b.putString("res",builder.toString());
            resultReceiver.send(MainActivity.RESULT_CODE,b);
                reader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

////////////////////////////////////////////////////////////////////////////////////////////
Android only service

package com.demo;

import android.app.Service;
import android.content.Intent;
import android.os.Bundle;
import android.os.IBinder;
import android.os.ResultReceiver;
import android.util.Log;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.URL;

import static com.demo.MainActivity.DOWNLOAD_ACTION;

public class MyService extends Service {
    public MyService() {
    }

    ResultReceiver resultReceiver;

    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        throw new UnsupportedOperationException("Not yet implemented");
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {

        resultReceiver=intent.getParcelableExtra("receiver");

        if(intent.getAction().equals(DOWNLOAD_ACTION)) {

            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {

                        URL url = new URL("https://raw.githubusercontent.com/kotlinkarun/api/master/.gitignore/emp.json");
                        BufferedReader reader = new BufferedReader(new InputStreamReader(url.openStream()));
                        String s;
                        StringBuilder builder = new StringBuilder();
                        while ((s = reader.readLine()) != null) {
                            builder.append(s);
                        }
                      //  Log.d("TAGS", "data : " + builder.toString());
                         Bundle b = new Bundle();
                         b.putString("res",builder.toString());
                         resultReceiver.send(MainActivity.RESULT_CODE,b);
                         reader.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }

                }
            }).start();

        }
        return START_STICKY;
    }
}



