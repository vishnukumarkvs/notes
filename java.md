java
====

# Collections

Collection framework provides a set of interfaces and classes thatt help in managing group s of objects

Top interfaces

Collection: The root interface for all collection types

List: An ordered collection that can contain duplicate elements (ArrrayList , LinkedList)

Set: A collection taht cannot contain duplicate  elements (Hashset, Treeset)

Queue : A collection designed for holding elements prior to processing (PriorityQueue)

Deque: A souble ended queue that allows insertion and removal from both ends (AraayDeque)

Map: an interface taht represents a collection of key value pairs (HashMap , TreeMap)

This is called collection hirerarchy

Collection interface extends Iterable interface. It has List, Set, Queue
Map is a seperate interface frim collection

# List interface

- its part of java.util package
- can contain duplicate elements
- Order preservation
- index based

### ArrayList - class
- its a resizable array

```
import java.util.ArrayList;
import java.util.List;

//TIP To <b>Run</b> code, press <shortcut actionId="Run"/> or
// click the <icon src="AllIcons.Actions.Execute"/> icon in the gutter.
public class Main {
    public static void main(String[] args) {
        int[] arr= new int[5];
        System.out.println(arr[1]);

        int[] arr2 = {1,2,3};

        // polymordphism : code to an interface not a implementation
        // List on right side is more flexible and interchangable
        List<Integer> al = new ArrayList<>();
        al.add(10);
        al.get(0); // 0 based indexing
        System.out.println("size: " + al.size());

        for (int x: al){
            System.out.println(x);
        }

        // existence
        System.out.println(al.contains(7));

        al.remove(0);
        System.out.println(al.size());
        al.add(1);
        al.set(0,10);

        System.out.println(al.get(0));
    }
}
```

- ArrayLIst can grow and shriink
- Intial capacity is 10
- capacity that list has without needing resize
- Add element : check capaity - resize if needed - add element
- resize 1.5 times the current capacity 
- when resizing , all elements from old array are copied to new array - O(n)


```
List<Integer> arrk = new ArrayList<>(10);
        System.out.println(arrk.size());
        System.out.println(arrk.get(0));
```
 
 - size is till 0, capcidty is 10, get(0)  will retunrn error
 - there is no method to print capacity but it can be done as jugaad using refelction
 
```
elementData = (Object[]) field.get(list);
sout(elementData.length);
```

- capacity : al.TrimToSize() - makes capacity to size - needs excepdtion

```
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

//TIP To <b>Run</b> code, press <shortcut actionId="Run"/> or
// click the <icon src="AllIcons.Actions.Execute"/> icon in the gutter.
public class Main {
    public static void main(String[] args) {
        List<String> arr = Arrays.asList("hi", "bye");
        System.out.println(arr.getClass().getName()); //  java.util.Arrays$ArrayList
//        arr.add("ba"); // throws exception. we can replace elements

        List<String> a2 = new ArrayList<>();
        System.out.println(a2.getClass().getName()); // java.util.ArrayList

        List<Integer> l = List.of(1,3,4); // immutable, unmodifiable
        List<Integer> ll = new ArrayList<>(l);
        ll.remove(Integer.valueOf(3));
        System.out.println(ll); // removes 3 value instead of value on index 3
        
        List<String> k = new ArrayList<>(arr);
        k.add("pa");
        k.remove("pa");

        System.out.println(k);
        
        Collections.sort(k);
        l.sort(null);
    }
}
```

- asList - is a fixed array
- List.of() - unmodifiable lilst

https://www.youtube.com/watch?v=XgF7XNLNf38&list=PLA3GkZPtsafZZsLj0Tybu3y0HVl-hp1ea&index=3

