JIT，即Just-in-time,动态(即时)编译，边运行边编译；AOT，Ahead Of Time，指运行前编译，是两种程序的编译方式



dalvik(android 5.0之前) = JIT

ART(Android run time, android 5.0之后) = AOT

android N 后使用JIT和AOT混合： 首次使用JIT, 在空闲的时候AOT编译存好

