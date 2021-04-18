

协程构建器有两种形式：自动传播异常（[launch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html) 与 [actor](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/actor.html)）或向用户暴露异常（[async](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html) 与 [produce](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/produce.html)）。 前者这类构建器将异常视为**未捕获**异常，类似 Java 的 `Thread.uncaughtExceptionHandler`， 而后者则依赖用户来最终消费异常，例如通过 [await](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/await.html) 或 [receive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-receive-channel/receive.html)



```kotlin
 fun exceptionFun() {
        launch {
            val handler = CoroutineExceptionHandler { _, exception ->
                Log.e("wzx", "CoroutineExceptionHandler got $exception")
            }
            val job = GlobalScope.launch(handler) {
                Log.e("wzx", "Throwing exception from launch")
                throw IndexOutOfBoundsException()
            }
            // 自传播异常，这里try-catch无效,使用 CoroutineExceptionHandler捕获异常
//            try {
//                job.join()
//                Log.e("wzx", "launch Unreached")
//            } catch (e: java.lang.Exception) {
//                Log.e("wzx", "Caught IndexOutOfBoundsException")
//            }
            val deferred = GlobalScope.async(handler) {
                Log.e("wzx", "Throwing exception from async") // // 同样是根协程，但使用 async 代替了 launch
                throw ArithmeticException() // 没有打印任何东西，依赖用户去调用 deferred.await()
            }
//            joinAll(job, deferred)
//            deferred.await()
            //用户暴露异常, try-catch哟i笑傲
//            try {
//                deferred.await()
//                Log.e("wzx", "Unreached")
//            } catch (e: ArithmeticException) {
//                Log.e("wzx", "Caught ArithmeticException")
//            }
        }
```

