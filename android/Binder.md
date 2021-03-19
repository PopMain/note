

#### aidl

```aidl
// IMyAidlInterface.aidl
package com.example.myapplication;

// Declare any non-default types here with import statements

interface IMyAidlInterface {
    void myTestFunction(String str);
}
```

生成同名java文件

```java
/*
 * This file is auto-generated.  DO NOT MODIFY.
 */
package com.example.myapplication;
// Declare any non-default types here with import statements

public interface IMyAidlInterface extends android.os.IInterface
{
  /** Default implementation for IMyAidlInterface. */
  public static class Default implements com.example.myapplication.IMyAidlInterface
  {
    @Override public void myTestFunction(java.lang.String str) throws android.os.RemoteException
    {
    }
    @Override
    public android.os.IBinder asBinder() {
      return null;
    }
  }
  /** Local-side IPC implementation stub class. */
  public static abstract class Stub extends android.os.Binder implements com.example.myapplication.IMyAidlInterface
  {
    private static final java.lang.String DESCRIPTOR = "com.example.myapplication.IMyAidlInterface";
    /** Construct the stub at attach it to the interface. */
    public Stub()
    {
      this.attachInterface(this, DESCRIPTOR);
    }
    /**
     * Cast an IBinder object into an com.example.myapplication.IMyAidlInterface interface,
     * generating a proxy if needed.
     */
    public static com.example.myapplication.IMyAidlInterface asInterface(android.os.IBinder obj)
    {
      if ((obj==null)) {
        return null;
      }
      android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
      if (((iin!=null)&&(iin instanceof com.example.myapplication.IMyAidlInterface))) {
        return ((com.example.myapplication.IMyAidlInterface)iin);
      }
      return new com.example.myapplication.IMyAidlInterface.Stub.Proxy(obj);
    }
    @Override public android.os.IBinder asBinder()
    {
      return this;
    }
    @Override public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException
    {
      java.lang.String descriptor = DESCRIPTOR;
      switch (code)
      {
        case INTERFACE_TRANSACTION:
        {
          reply.writeString(descriptor);
          return true;
        }
        case TRANSACTION_myTestFunction:
        {
          data.enforceInterface(descriptor);
          java.lang.String _arg0;
          _arg0 = data.readString();
          this.myTestFunction(_arg0);
          reply.writeNoException();
          return true;
        }
        default:
        {
          return super.onTransact(code, data, reply, flags);
        }
      }
    }
    private static class Proxy implements com.example.myapplication.IMyAidlInterface
    {
      private android.os.IBinder mRemote;
      Proxy(android.os.IBinder remote)
      {
        mRemote = remote;
      }
      @Override public android.os.IBinder asBinder()
      {
        return mRemote;
      }
      public java.lang.String getInterfaceDescriptor()
      {
        return DESCRIPTOR;
      }
      @Override public void myTestFunction(java.lang.String str) throws android.os.RemoteException
      {
        android.os.Parcel _data = android.os.Parcel.obtain();
        android.os.Parcel _reply = android.os.Parcel.obtain();
        try {
          _data.writeInterfaceToken(DESCRIPTOR);
          _data.writeString(str);
          boolean _status = mRemote.transact(Stub.TRANSACTION_myTestFunction, _data, _reply, 0);
          if (!_status && getDefaultImpl() != null) {
            getDefaultImpl().myTestFunction(str);
            return;
          }
          _reply.readException();
        }
        finally {
          _reply.recycle();
          _data.recycle();
        }
      }
      public static com.example.myapplication.IMyAidlInterface sDefaultImpl;
    }
    static final int TRANSACTION_myTestFunction = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
    public static boolean setDefaultImpl(com.example.myapplication.IMyAidlInterface impl) {
      // Only one user of this interface can use this function
      // at a time. This is a heuristic to detect if two different
      // users in the same process use this function.
      if (Stub.Proxy.sDefaultImpl != null) {
        throw new IllegalStateException("setDefaultImpl() called twice");
      }
      if (impl != null) {
        Stub.Proxy.sDefaultImpl = impl;
        return true;
      }
      return false;
    }
    public static com.example.myapplication.IMyAidlInterface getDefaultImpl() {
      return Stub.Proxy.sDefaultImpl;
    }
  }
  public void myTestFunction(java.lang.String str) throws android.os.RemoteException;
}

```

Sub继承自Binder，为server端实例， asInterface返回提供客户端调用的Proxy实例（Proxy最终调用Stub）



```java
// MainActivity-ProcessA:
private val binder  = object : IMyAidlInterface.Stub() {
        override fun myTestFunction(str: String?) {
            Log.e("wzx", "service: $str")
            textView?.text = str
        }
    }
val bundle = Bundle()
            bundle.putBinder("binder", binder.asBinder())
            val intent = Intent(this, SecondActivity::class.java)
            intent.putExtras(bundle)
            startActivity(intent)
  // SecondActivity-ProcessB:
    val iBinder = intent.extras?.getBinder("binder")      		     IMyAidlInterface.Stub.asInterface(iBinder)?.myTestFunction("djfoasdfjoasidofoaisdfjioasiodfioasidofasiodf")
```



### Binder







https://blog.csdn.net/freekiteyu/article/details/70082302

