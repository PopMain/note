延期的值提供了一种便捷的方法使单个值在多个协程之间进行相互传输。 通道提供了一种在流中传输值的方法。

一个 [Channel](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-channel/index.html) 是一个和 `BlockingQueue` 非常相似的概念。其中一个不同是它代替了阻塞的 `put` 操作并提供了挂起的 [send](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-send-channel/send.html)，还替代了阻塞的 `take` 操作并提供了挂起的 [receive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-receive-channel/receive.html)。

```kotlin
 
fun channelFun() {
        launch {
            val channel = Channel<Int>()
            launch(Dispatchers.IO) {
                for (x in 1..5) {
                    delay(1000)
                    channel.send(x * x)
                }
            }
            repeat(5) {
                Log.e("wzx", "${channel.receive()}")
            }
//            val channel2 = numbersFrom()
//            while (true) {
//                Log.e("wzx", "${channel2.receive()}")
//                textReadWriteFile.text = "${channel2.receive()}"
//            }
            val producer = numbersFrom()
            repeat(5) {
                launchProcessor(it, producer)
            }
            delay(950)
            producer.cancel()
            val sender0 = Channel<String>(4)
            launch {
                sendString(sender0, "foo", 200)
            }
            launch {
                sendString(sender0, "bar", 500)
            }
            repeat(6) {
                Log.e("wzx", sender0.receive())
            }
            val sender = Channel<Int>()
            launch {
                repeat(10) {
                    Log.e("wzx", "Sending $it")
                    sender.send(it)
                }
            }
            delay(1000)
            sender.cancel()
            val tickerChannel = ticker(delayMillis = 100, initialDelayMillis =  100)
            var nextElement = withTimeoutOrNull(1) { tickerChannel.receive() }
            Log.e("wzx", "Initial element is available immediately: $nextElement") // no initial delay

            nextElement = withTimeoutOrNull(50) { tickerChannel.receive() } // all subsequent elements have 100ms delay
            Log.e("wzx", "Next element is not ready in 50 ms: $nextElement")

            nextElement = withTimeoutOrNull(60) { tickerChannel.receive() }
            Log.e("wzx", "Next element is ready in 100 ms: $nextElement")

            // 模拟大量消费延迟
            Log.e("wzx", "Consumer pauses for 150ms")
            delay(150)
            // 下一个元素立即可用
            nextElement = withTimeoutOrNull(1) { tickerChannel.receive() }
            Log.e("wzx", "Next element is available immediately after large consumer delay: $nextElement")
            // 请注意，`receive` 调用之间的暂停被考虑在内，下一个元素的到达速度更快
            nextElement = withTimeoutOrNull(60) { tickerChannel.receive() }
            Log.e("wzx", "Next element is ready in 50ms after consumer pause in 150ms: $nextElement")

            tickerChannel.cancel() // 表明不再需要更多的元素
        }
    }
```

