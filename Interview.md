Interview
=========

# Java

1. What is JIT
   - JIT is a part of jvm which converts part of byte code to machine code during runtime
   - Compiler converts source .java code to byte code

2. why main method is static in java
    - main method is entry point in java
    - jvm needs to call this function
    - static methods no need to be instatiated and we can  directly call by classname, no object required

3. Primitive vs non primitive datatypes
    - Primitive data types are single value variables which can be stored directly in memory
    - no extra capabilities
    - int, boolean, byte, long, short, float, double, char
    - total 8
    - Non Primitive data types will need a memory address for variable
    - Ex: String, Array, Class, Object, Interface

4. Whats a thread in java, how do u create a thread
    - A thread in java is a lilghtweight subprocess which can run independently but uses same resources and memory space
    - you can create a thread by extending Thread class or implementing a Runnable interface

5. What is a deadllock
    - A deadlock is a situation where ttwo or more threads are stuck forever
    - example , we have t1 and t2 threads, t1 is using resource A and is tryoing to lock resource B. whereas t2, usi