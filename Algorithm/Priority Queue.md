## 우선순위 큐

- 선입선출 구조인 Queue와 달리 Queue에 들어있는 자료 중 우선순위를 설정하여
  ***우선순위가 높은 순서대로 데이터를 꺼내는 구조이다.***

  - 큐에 들어오는 모든 데이터에 우선순위가 존재하며, 데이터를 꺼낼 때 우선순위가 높은 순서대로 나온다.

  - 우선순위가 같은 경우에는 선입선출 방식을 따른다.

- 자바의 우선순위 큐는 힙(Heap) 방식으로 구현되어 있다.

1. 기본 생성 
매개변수에 아무 값도 넣지 않으면 기본 정렬 방법(오름차순)을 따른다.
```java
PriorityQueue<Integer> queue = new PriorityQueue<>();
```
2. Comparable 구현
임의의 클래스로 구성된 PriorityQueue의 경우, </br>
해당 클래스에 Comparable Interface를 상속받아서 compareTo 메서드를 구현해주면 우선순위를 지정해줄 수 있다.

``` java
PriorityQueue<MyMember> memberQueue = new PriorityQueue<>();

---------------------------------------

class MyMember implements Comparable<MyMember>{
    String name;
    int age;

    public MyMember(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int compareTo(MyMember o) {
        return this.age-o.age;
    }
}
```
```java
PriorityQueue<MyMember> memberQueue = new PriorityQueue<>();
memberQueue.offer(new MyMember("memA",20));
memberQueue.offer(new MyMember("memB",10));
memberQueue.offer(new MyMember("memC",30));
```
이 때, age를 기준으로 compareTo메서드를 구현해주었기 때문에 </br>

우선순위는 memB(age=10) >  memA(age=20) >  memC(age=30) 이 된다.

3. 람다식 이용
PriorityQueue 생성시에 매개변수로 람다식을 작성하여 우선순위를 지정해 줄 수있다.
```java
PriorityQueue<MyMember> memberQueue = new PriorityQueue<>((m1,m2)->m1.age-m2.age);
```

## comparable 혹은 람다식, 우선순위 지정에 대한 공부 더 필요.
유연하게 사용할 수 있도록.












발췌 : https://innovation123.tistory.com/112










