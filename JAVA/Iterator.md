## Iterator VS for

- Iterator의 장점
  - 컬렉션에서 요소를 제어하는 기능
  - Iterator \<T> iterator = Collection.iterator();
  - next() 및 previous()를 써서 앞뒤로 이동하는 기능
  - hasNext()를 써서 더 많은 요소가 있는지 확인하는 기능
  - List와 Set 인터페이스에서 iterator() 메소드를 사용
    - Set의 경우 인덱스가 없기 때문에 일반 for문을 사용할 수 없지만, for-each문은 사용할 수 있다.</br>
      그러나 for-each문으로 자료구조를 돌다가 값을 수정, 삭제해야 해서 수정, 삭제를 수행하는 코드를 넣으면 ConcurrentModificationException이 발생한다.

- iterator를 이용해 값을 수정하는 예제
    - ListIterator 인터페이스는 List 인터페이스를 구현한 List 컬렉션 클래스에서만 listIterator() 메소드를 통해 사용할 수 있다.
```java

import java.util.ArrayList;
import java.util.ListIterator;

public class Main
{
    public static void main(String[] args)
    {
        // 컬렉션 생성
        ArrayList<String> list = new ArrayList<>();
        list.add("A");
        list.add("B");
        list.add("C");
        list.add("D");
        list.add("E");
        list.add("F");
        System.out.println("while문 지나기 전 리스트에 들어있던 값 : " + list);

        // 리스트에 들어있는 값에 각각 '+' 붙이기
        ListIterator<String> listIterator = list.listIterator();
        while(listIterator.hasNext())
        {
            Object element = listIterator.next();
            listIterator.set(element + "+");
        }
        System.out.println("while문 지난 후 수정된 결과 : " + list);

        // 리스트에 들어있는 값을 역순으로 표시
        System.out.print("역순 출력 결과 : ");
        while(listIterator.hasPrevious())
        {
            Object element = listIterator.previous();
            System.out.print(element + " ");
        }
        System.out.println();
    }

}
// >> while문 지나기 전 리스트에 들어있던 값 : [A, B, C, D, E, F]
// >> while문 지난 후 수정된 결과 : [A+, B+, C+, D+, E+, F+]
// >> 역순 출력 결과 : F+ E+ D+ C+ B+ A+


```



