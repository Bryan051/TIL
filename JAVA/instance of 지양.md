# instance of 연산자 사용은 지양한다.
- 다형성으로 해결하는것을 지향
 1. 추상화, 캡슐화가 깨진다
 2. OCP 원칙 위반. (Open Closed Princible) 객체가 확장에는 열려있고 변화에는 닫혀있어야 한다.
 3. SRP 원칙 위반. (Single Responsibility Principle) 객체는 한 가지(본인의) 책임만 지어야 한다.

ex) 
if ( A instance of B ){
  }
보다
isB() 등의 객체를 선언하여 확인한다.
