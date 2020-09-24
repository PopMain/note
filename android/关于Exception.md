1. Java控制台程序

   a. java子线程异常（没有捕获）是不会导致主线程退出

   

2. Android程序

   a. android不管是子线程还是主线程异常，如果是没有捕获的话都会导致APP Crash

   ​       /***  https://juejin.im/post/5dd52e156fb9a05a7523778e ***/

​       b. android主线程的异常没有捕获，即使设置了UnCaughtExceptionHandler,  处理了这个exception，都会导致APP卡住（原因：崩 溃导致了Main线程退出loop）

       ```java
package com.popmain.test;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;

import android.animation.ObjectAnimator;
import android.animation.ValueAnimator;
import android.os.Bundle;
import android.view.View;

import static android.view.animation.Animation.INFINITE;

public class MainActivity extends AppCompatActivity implements Thread.UncaughtExceptionHandler {
    private Thread.UncaughtExceptionHandler defaultUnCaughtHandler;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        defaultUnCaughtHandler = Thread.getDefaultUncaughtExceptionHandler();
        Thread.setDefaultUncaughtExceptionHandler(this);
        setContentView(R.layout.activity_main);
        ObjectAnimator valueAnimator = ObjectAnimator.ofFloat(findViewById(R.id.tv), "alpha", 0, 1, 0);
        valueAnimator.setDuration(500);
        valueAnimator.setRepeatCount(INFINITE);
        valueAnimator.setRepeatMode(ValueAnimator.REVERSE);
        valueAnimator.start();
        /**
        **java.lang.RuntimeException: Unable to start activity ComponentInfo{com.popmain.test/com.popmain.test.MainActivity}: java.lang.NullPointerException: Attempt to invoke virtual method 'char java.lang.String.charAt(int)' on a null object reference
        */
//        String s = null;
//        s.charAt(99);
        new Thread(new Runnable() {
            @Override
            public void run() {
              	// uncaughtException了空指针异常了（没有调用defaultUnCaughtHandler.uncaughtException不会导致APP崩溃）
                String s = null;
                s.charAt(99);
            }
        }).start();
    }

    public void helloWorldClick(View view) {
        System.out.println("wzx, helloWorldClick");
    }

    @Override
    public void uncaughtException(@NonNull Thread t, @NonNull Throwable e) {
      /**
      ** 主线程崩溃最终会被处理，最总抛出来的Exception类型是 ava.lang.RuntimeException
      ** 这里既是return了没有调用defaultUnCaughtHandler.uncaughtException，APP也会卡住（loop已经退出）
      **/
//        if (e instanceof RuntimeException) {
//            System.out.println("wzx, runtime exception " + e.getMessage());
//            return;
//        }
        if (e instanceof NullPointerException) {
            System.out.println("wzx, NullPointerException Occur");
            return;
        }
        defaultUnCaughtHandler.uncaughtException(t, e);
    }
}
       ```



​    如果是Error类型的错误，那么如果导致了虚拟机退出，不管是主线程还是子线程java、android都会导致JVM退出

