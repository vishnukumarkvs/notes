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
    - example , we have t1 and t2 threads, t1 is using resource A and is tryoing to lock resource B. whereas t2, using resource B and trying to lock resource A now
    - real world examples : Database conenctions , multithreadgn file access, nested synchronization blocks
    - use global ordering for aquiring state lock,  use timeouts etc

6. How do you do thread pooling in java
    - Thread pooling is a collection of reusable threads. We use thread pooling to improve performance by reusing threads instead of frequently creating and destroying them
    - ExecutorService executor = Executors.newFixedThreadPool(10);

7. What is volatile keyword
   - volatile indicates that the variable value will be modified by different threads
   - It prevents threadds from caching value of variable locally and ensures visibliltly acaross threads


# Springboot

1. How so u recognize a springboot project
   - If the main method has @SpringBootApplication annotation

2. What does @SpringbootApplication annotation do
    - It combines 3 annotaions at the start of application
    - @Configuration + @EnabledAutoConfiguration + @ComponentScan
    - ComponentScan scans  for @Component and @Service etc
    - Configuration etc loads dependencies and beans

3. Which annotions are used for REST services
    - @RestController : This annotation is used to define a RESTful web service controller. It is a specialized version of the @Controller annotation that includes the @ResponseBody annotation by default.
    - @RequestMapping: Used to map http requests to specific method in controller

4. Where do you change the port of a tomcat server
    - We can change by specifying bew value to service.port in applications.properties file

5. What are starter dependencies in springboot app
    - These are pre configured maven dependencies that makes it easier to build specific applications
    - spring-boot-starter-web for web applications

6. What ia Springboot actuator
   -