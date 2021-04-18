Actors： 个人理解，解决同步访问**共享的可变状态**

一个 [actor](https://en.wikipedia.org/wiki/Actor_model) 是由协程、 被限制并封装到该协程中的状态以及一个与其它协程通信的 *通道* 组合而成的一个实体。一个简单的 actor 可以简单的写成一个函数， 但是一个拥有复杂状态的 actor 更适合由类来表示。

有一个 [actor](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/actor.html) 协程构建器，它可以方便地将 actor 的邮箱通道组合到其作用域中（用来接收消息）、组合发送 channel 与结果集对象，这样对 actor 的单个引用就可以作为其句柄持有。

使用 actor 的第一步是定义一个 actor 要处理的消息类。 Kotlin 的[密封类](https://kotlinlang.org/docs/reference/sealed-classes.html)很适合这种场景。 我们使用 `IncCounter` 消息（用来递增计数器）和 `GetCounter` 消息（用来获取值）来定义 `CounterMsg` 密封类。 后者需要发送回复。[CompletableDeferred](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-completable-deferred/index.html) 通信原语表示未来可知（可传达）的单个值， 这里被用于此目的



```kotlin
sealed class CounterMsg {
        object IncCounter : CounterMsg()
        class GetCounter(val response: CompletableDeferred<Int>) : CounterMsg()
    }

    fun CoroutineScope.counterActor() = actor<CounterMsg> {
        var counter = 0
        for (msg in channel) {
            when(msg) {
                is CounterMsg.IncCounter -> counter++
                is CounterMsg.GetCounter -> msg.response.complete(counter)
            }
        }
    }

    suspend fun massiveRun(action: suspend () -> Unit) {
        val n = 100  // 启动的协程数量
        val k = 1000 // 每个协程重复执行同一动作的次数
        val time = measureTimeMillis {
            coroutineScope { // 协程的作用域
                repeat(n) {
                    launch {
                        repeat(k) { action() }
                    }
                }
            }
        }
        println("Completed ${n * k} actions in $time ms")
    }
    
    
    fun main() {
     val counter = counterActor()
                withContext(Dispatchers.IO) {
                    massiveRun { counter.send(CounterMsg.IncCounter) }
                }
                val response = CompletableDeferred<Int>()
                counter.send(CounterMsg.GetCounter(response))
                Log.e("wzx", "Counter = ${response.await()}")
    }
```

