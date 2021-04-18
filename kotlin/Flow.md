流： 返回多个值。

```kotlin
 fun flowTest() = launch {
        simple().forEach { value ->
            Log.e("wzx", "value $value")
            textSelect.text = "value=$value"
        }
        Log.e("wzx", "launch end")
        val flow = flowFun()
        Log.e("wzx", "followFun called")
        Log.e("wzx", "collecting")
        withTimeoutOrNull(2000) {
            flow.collect(object : FlowCollector<Int> {
                override suspend fun emit(value: Int) {
                    Log.e("wzx", "value $value")
                    textSelect.text = "value=$value"
                }
            })
        }
        (1..3).asFlow()
            .map { request ->
                Log.e("wzx", performRequest(request))
                request * request
            }
            .collect(object : FlowCollector<Int> {
                override suspend fun emit(value: Int) {
                    Log.e("wzx", "value $value")
                    textSelect.text = "value=$value"
                }
            })
        (1..3).asFlow()
            .transform { request ->
                emit("Making request $request")
                delay(2000)
                loop()
                emit(performRequest(request))
            }
            .collect(object : FlowCollector<String> {
                override suspend fun emit(value: String) {
                    Log.e("wzx", "value $value")
                    delay(2000)
                    textSelect.text = "value=$value"
                }
            })
        flowFun3()
            .map { request ->
                request * request
            }
            .transform { transform ->
                emit("transform $transform")
            }
            .collect(object : FlowCollector<String> {
                override suspend fun emit(value: String) {
                    Log.e("wzx", "value $value")
                    textSelect.text = "value=$value"
                }
            })
        loop()
        getData().collect(object : FlowCollector<Data> {
            override suspend fun emit(value: Data) {
                textSelect.text = "value=${value.data}"
            }
        })
        flowFun3().flowOn(Dispatchers.Default).buffer().collect { value ->
            delay(2000)
            Log.e("wzx", "value $value currentThread ${Thread.currentThread().name}")
            textSelect.text = "value=$value"
        }
        val flow1 = flowOf(1, 2, 3, 4)
        val flow2 = flowOf("one", "two", "three")
        flow1.zip(flow2) { a, b ->
            delay(2000)
            "$a -> $b"
        }.collect {
            textSelect.text = it
        }
        val startTime = System.currentTimeMillis()
        (1..3).asFlow().onEach { delay(100) }
            .flatMapConcat {
                requestFlow(it)
            }
            .collect {
                Log.e("wzx", "$it spend ${System.currentTimeMillis() - startTime} ms")
            }

        Log.e("wzx", "===================================================================")
        val startTime2 = System.currentTimeMillis()
        (1..3).asFlow().onEach { delay(100) }
            .flatMapMerge {
                requestFlow(it)
            }
            .collect {
                Log.e("wzx", "$it spend ${System.currentTimeMillis() - startTime2} ms")
            }

        Log.e("wzx", "===================================================================")
        val startTime3 = System.currentTimeMillis()
        (1..3).asFlow().onEach { delay(100) }
            .flatMapLatest {
                requestFlow(it)
            }
            .collect {
                Log.e("wzx", "$it spend ${System.currentTimeMillis() - startTime3} ms")
            }
        (1..3).asFlow()
            .catch { e -> Log.e("wzx", "catch $e") }
            .collect { value ->
                check(value <= 1)
                Log.e("wzx", "$value")
            }
        Log.e("wzx", "collecting again")
        flow.collect(object : FlowCollector<Int> {
            override suspend fun emit(value: Int) {
                Log.e("wzx", "value $value")
                textSelect.text = "value=$value"
            }
        })
    }

  fun requestFlow(value: Int) = flow {
        emit("$value : First")
        delay(500)
        emit("$value : Second")
    }

    fun flowFun3(): Flow<Int> = flow {
        Log.e("wzx", "flow ${Thread.currentThread().name}")
        emit(1)
        Thread.sleep(200)
//        delay(1000)
        emit(2)
        Thread.sleep(200)
//        delay(1000)
        emit(3)
    }

    fun getData(): Flow<Data> = flow {
        val cache = getCache()
        emit(cache)
        val onlineData = requestData()
        emit(onlineData)
    }

    suspend fun getCache(): Data {
        loop()
        return Data(1, "cache")
    }

    suspend fun requestData(): Data {
        loop()
        return Data(2, "online data")
    }


    fun flowFun(): Flow<Int> = flow {
        for (i in 1..3) {
            delay(1000)
            emit(i)
        }
    }

    fun flowFun2(): Flow<Int> = flow {
        (1..3).asFlow()
    }

    suspend fun performRequest(request: Int): String {
        delay(1000)
        return "$request response"
    }

    suspend fun simple2(): List<Int> {
        delay(1000)
        return listOf(1, 2, 3)
    }

    suspend fun simple(): Sequence<Int> =
        sequence {
            for (i in 1..3) {
                Thread.sleep(1000)
                Log.e("wzx", "yield $i")
                yield(i)
            }
        }


    private suspend fun getUser(): String = withContext(Dispatchers.IO) {
        Log.e("wzx", "getUser Thread ${Thread.currentThread().name}")
        delay(2000)
        "PopMain"
    }

    private suspend fun getId(): String = withContext(Dispatchers.IO) {
        Log.e("wzx", "getId Thread ${Thread.currentThread().name}")
        delay(2000)
        "Eiijf039fjj"
    }

 suspend fun loop() {
        var i = 0
        while (true) {
            if (i == 2) {
                break
            }
            delay(1000)
            Log.e("wzx", "looping")
            i++
        }
    }

```

