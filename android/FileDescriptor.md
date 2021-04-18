```kotlin
//读取当前进程的fd列表  
 @RequiresApi(Build.VERSION_CODES.LOLLIPOP)
    private suspend fun listFd(): String? = withContext(Dispatchers.IO) {
        val fdFile = File("/proc/" + android.os.Process.myPid() + "/fd")
        val files = fdFile.listFiles() // 列出当前目录下所有的文件
        val length = files.size - 1 // 进程中的fd数量
        val stb = StringBuilder()
        for (i in 0..length) {
            try {
                Log.e("wzx", "absolutePath ${files[i].absolutePath}") // /proc/14424/fd/113
                val strFile = Os.readlink(files[i].absolutePath) // 得到软链接实际指向的文件 /dev/ashmem
                stb.append(strFile + "\n")
            } catch (x: Exception) {
                Log.e("wzx", "listFd error=" + x)
            }
        }
        Log.d("wzx", "listFd=$stb")
        stb.toString()
    }
```

