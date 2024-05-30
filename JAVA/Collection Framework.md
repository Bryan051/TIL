## Collection Framework

![image](https://github.com/Bryan051/TIL/assets/68111122/50813cc3-ba57-4ffa-b250-5a4f0f198576)

컬렉션 프레임워크는 크게 Collection 인터페이스와 Map 인터페이스로 나뉜다.

## Iterable 인터페이스
- 컬렉션 인터페이스들의 가장 최상위 인터페이스
    - Map은 iterable 인터페이스를 상속받지 않기 때문이 iterator()와 spliterator()는 Map 컬렉션에 구현이 되어 있지 않다. </br>
      따라서 직접적으로 Map 컬렉션을 순회할수는 없고 스트림(Stream)을 사용하거나 간접적으로 키를 Collection으로 반환하여 루프문으로 순회하는 식을 이용한다.

## Collection 인터페이스
- List, Set, Queue 에 상속을하는 실질적인 최상위 컬렉션 타입
    - Collection 인터페이스의 메서드를 보면 요소(객체)에 대한 추가, 삭제, 탐색은 다형성 기능으로 사용이 가능하지만, 데이터를 get 하는 메서드는 보이지 않는다. 왜냐하면 각 컬렉션 자료형 마다 구현하는 자료 구조가 제각각 이기 때문에 최상위 타입으로 조회하기 까다롭기 때문이다.

## List 인터페이스
![image](https://github.com/Bryan051/TIL/assets/68111122/b95b0aca-cc62-48cc-a351-115b1ff87200)

- 저장 순서가 유지되는 컬렉션을 구현하는 데 사용
- 같은 요소의 중복 저장을 허용
- 배열과 마찬가지로 index로 요소에 접근
- 리스트와 배열의 가장 큰 차이는 리스트는 자료형 크기가 고정이 아닌 데이터 양에 따라 동적으로 늘어났다 줄어들수 있다는 점이다. (가변)
- 요소 사이에 빈공간을 허용하지 않아 삽입/삭제 할때마다 배열 이동이 일어난다

## ArrayList 클래스
![image](https://github.com/Bryan051/TIL/assets/68111122/0d6b9456-2245-4920-b8dc-829c0b822633)

- 데이터의 저장순서가 유지되고 중복을 허용
- 데이터량에 따라 공간(capacity)가 자동으로 늘어나거나 줄어들음
- 단방향 포인터 구조로 자료에 대한 순차적인 접근에 강점이 있어 조회가 빠르다
- 하지만, 삽입 / 삭제가 느리다는 단점이 있다. 단, 순차적으로 추가/삭제 하는 경우에는 가장 빠르다

## LinkedList 클래스
![image](https://github.com/Bryan051/TIL/assets/68111122/e27adc4c-efc6-4abe-8c2a-85b894ff6a3a)

- 노드(객체)를 연결하여 리스트 처럼 만든 컬렉션 (배열이 아님)
- 데이터의 중간 삽입, 삭제가 빈번할 경우 빠른 성능을 보장한다.
- 하지만  **임의의 요소에 대한 접근 성능은 좋지 않다.**
- 자바의 LinkedList는 Doubly LinkedList(양방향 포인터 구조)로 이루어져 있다.
- LinkedList는 리스트 용도 이외에도, 스택, 큐, 트리 등의 자료구조의 근간이 된다.

## Set 인터페이스
![image](https://github.com/Bryan051/TIL/assets/68111122/148e4d02-4bd2-4b1e-82c5-3e538258908f)

- 데이터의 중복을 허용하지 않고 순서를 유지하지 않는 데이터의 집합 리스트
- 순서 자체가 없으므로 인덱스로 객체를 검색해서 가져오는 get(index) 메서드도 존재하지 않는다
- 중복 저장이 불가능하기 때문에 심지어 null값도 하나만 저장할 수 있다

## HashSet 클래스
![image](https://github.com/Bryan051/TIL/assets/68111122/092dc46f-a1a8-498f-bddc-2e5903033639)

- 배열과 연결 노드를 결합한 자료구조 형태
- 가장 빠른 임의 검색 접근 속도를 가진다
- 추가, 삭제, 검색, 접근성이 모두 뛰어나다
- 대신 순서를 전혀 예측할 수 없다

## LinkedHashSet 클래스

- 순서를 가지는 Set 자료
- 추가된 순서 또는 가장 최근에 접근한 순서대로 접근 가능
- 만일 중복을 제거하는 동시에 저장한 순서를 유지하고 싶다면, HashSet 대신 LinkedHashSet을 사용하면 된다

## TreeSet 클래스
![image](https://github.com/Bryan051/TIL/assets/68111122/84f15168-5406-4e6e-9950-e2da78b31c01)


- 이진 검색 트리(binary search tree) 자료구조의 형태로 데이터를 저장
- 중복을 허용하지 않고, 순서를 가지지 않는다
- 대신 데이터를 정렬하여 저장하고 있다는 특징이다
- 정렬, 검색, 범위 검색에 높은 성능을 뽐낸다. 

---

## Map 인터페이스
![image](https://github.com/Bryan051/TIL/assets/68111122/e1a609e4-b504-4ef6-8b54-77ef4e5461ce)

- 키(Key)와 값(value)의 쌍으로 연관지어 이루어진 데이터의 집합
- 값(value)은 중복되서 저장될수 있지만, 키(key)는 해당 Map에서 고유해야만 한다.
- 만일 기존에 저장된 데이터와 중복된 키와 값을 저장하면 기존의 값은 없어지고 마지막에 저장된 값이 남게 된다.
- 저장 순서가 유지 되지 않는다

## Map.Entry 인터페이스
![image](https://github.com/Bryan051/TIL/assets/68111122/24ea16fb-6bc2-45ff-8a52-5fe3f3f243c0)

- Map.Entry 인터페이스는 Map 인터페이스 안에 있는 내부 인터페이스이다.
- Map 에 저장되는 key - value 쌍의 Node 내부 클래스가 이를 구현하고 있다.
- Map 자료구조를 보다 객체지향적인 설계를 하도록 유도하기 위한 것이다.

## HashMap 클래스
![image](https://github.com/Bryan051/TIL/assets/68111122/c57335d9-b0e1-420a-a08b-cafbe43015a5)

- Hashtable을 보완한 컬렉션
- 배열과 연결이 결합된 Hashing형태로, 키(key)와 값(value)을 묶어 하나의 데이터로 저장한다
- 중복을 허용하지 않고 순서를 보장하지 않는다
- 키와 값으로 null이 허용된다
- 추가, 삭제, 검색, 접근성이 모두 뛰어나다
- HashMap은 비동기로 작동하기 때문에 멀티 쓰레드 환경에서는 어울리지않는다 (대신 ConcurrentHashMap 사용)

## LinkedHashMap 클래스
![image](https://github.com/Bryan051/TIL/assets/68111122/bc14b338-1271-47d6-a7c2-b454b5a543ce)

- HashMap을 상속하기 때문에 흡사하지만, Entry들이 연결 리스트를 구성하여 데이터의 순서를 보장한다.
- 일반적으로 Map 자료구조는 순서를 가지지 않지만, LinkedHashMap은 들어온 순서대로 순서를 가진다.

## TreeMap 클래스
![image](https://github.com/Bryan051/TIL/assets/68111122/4df01f39-c04c-4172-a9d2-7c7d75e8a6c3)

- 이진 검색 트리의 형태로 키와 값의 쌍으로 이루어진 데이터를 저장 (TreeSet 과 같은 원리)
- TreeMap은 SortedMap 인터페이스를 구현하고 있어 Key 값을 기준으로 정렬되는 특징을 가지고 있다.
- 정렬된 순서로 키/값 쌍을 저장하므로 빠른 검색(특히 범위 검색)이 가능하다
- 단, 키와 값을 저장하는 동시에 정렬을 행하기 때문에 저장시간이 다소 오래 걸리다
- 정렬되는 순서는 숫자 → 알파벳 대문자 → 알파벳 소문자 → 한글 순이다.

# 컬렉션 프레임워크 선택 시점

![image](https://github.com/Bryan051/TIL/assets/68111122/a12a0338-e538-44ea-aa8b-272e5067f658)

- ArrayList
  - 리스트 자료구조를 사용한다면 기본 선택
  - 임의의 요소에 대한 접근성이 뛰어남
  - 순차적인 추가/삭제 제일 빠름
  - 요소의 추가/삭제 불리

- LinkedList
  - 요소의 추가/삭제 유리
  - 임의의 요소에 대한 접근성이 좋지 않음


- HashMap / HashSet
  - 해싱을 이용해 임의의 요소에 대한 추가/삭제/검색/접근성 모두 뛰어남
  - 검색에 최고성능 ( get 메서드의 성능이 O(1) )


- TreeMap / TreeSet
  - 요소 정렬이 필요할때
  - 검색(특히 범위검색)에 적합
  - 그래도 검색 성능은 HashMap보다 떨어짐

- LinkedHashMap / LinkedHashSet : HashMap과 HashSet에 저장 순서 유지 기능을 추가
- Queue : 스택(LIFO) / 큐(FIFO) 자료구조가 필요하면 ArrayDeque 사용
- Stack, Hashtable : 가급적 사용 X (deprecated)






</br></br></br></br>

출처: https://inpa.tistory.com/entry/JCF-🧱-Collections-Framework-종류-총정리 [Inpa Dev 👨‍💻:티스토리]

























출처: https://inpa.tistory.com/entry/JCF-🧱-Collections-Framework-종류-총정리 [Inpa Dev 👨‍💻:티스토리]
