### Global / Local Class
* T-CODE(SE24)을 이용해 전역 클래스와 전역 인터페이스를 정의 할 수 있음.
  * 이렇게 정의된 클래스/인터페이스는 Class Pools에 저장되며, 모든 ABAP 프로그램에서 해당 클래스를 사용할 수 있다.
* ABAP Object 내부에서 선언한 클래스/인터페이스는 해당 Object(프로그램) 내부에서만 사용할 수 있음.
  * 인스턴스화되어 사용될 때에는 전역 클래스와 지역 클래스 간에 아무런 차이가 없다.
 
### Class 선언 기본 구문
* DEFINITION : 클래스의 구성 항목(Attributes, Method, Events)를 정의
* IMPLEMENTATION : 클래스의 Method를 구현

```
CLASS c1 DEFINITION. "C1이라는 클래스 정의"
 PUBLIC SECTION.    
 METHODS meth.	"meth라는 이름의 METHOD를 정의"
 ENDCLASS.       "C1 DEFITION:"
 
 CLASS c1 IMPLEMENTATION. "c1의 IMPLEMENTATION(구현)"
 METHOD meth. "meth라는 이름의 METHOD의 로직을 선언하는 부분"
 WRITE / 'CLASS TEST'. "meth라는 METHOD는 'CLASS TEST'를 WRITE하는 것."
 ENDMETHOD.
 ENDCLASS.       "C1 IMPLEMENTATION"
 
 DATA : go_cref TYPE REF TO c1. "클래스 c1을 객체 참조 변수하는 go_cref를 선언"
 START-OF-SELECTION.			
 CREATE OBJECT go_cref.			"CREATE OBJECT : 객체로 만들기"
 CALL METHOD go_Cref->meth. 	"CALL METHOD : go_cref를 통해 METHOD : meth부르기"
```

### CLASS 구성요소
* 클래스의 모든 구성요소는 Declaration part(선언부)에서 선언
* 클래스를 정의할 때, 각 항목은 3개의 접근 제한 영역(PRIVATE, PROTECTED, PUBLIC) 중 한 곳에서 선언되어야 한다.
* 클래스의 모든 항목은 클래스 내부에서는 모두 보이지만, **선언 방식에 따라서 다른 클래스에서는 보이지 않을 수(인터페이스 되지 않을 수) 있다.**
---
### 이게 무슨 말일까?
* 클래스 안에서는 모든 구성요소를 다 볼 수 있다.
* 하지만 다른 클래스에서 그 구성요소를 사용하려면, 선언한 접근 영역에 따라 보이는 범위가 달라진다는 의미
```
CLASS my_class DEFINITION.
  PUBLIC SECTION.
    METHODS: public_method.
  PROTECTED SECTION.
    METHODS: protected_method.
  PRIVATE SECTION.
    METHODS: private_method.
ENDCLASS.

CLASS my_class IMPLEMENTATION.
  METHOD public_method.
    WRITE 'Public'.
  ENDMETHOD.
  METHOD protected_method.
    WRITE 'Protected'.
  ENDMETHOD.
  METHOD private_method.
    WRITE 'Private'.
  ENDMETHOD.
ENDCLASS.
````
* public_method → 다른 클래스에서도 호출 가능
* protected_method → 상속받은 클래스만 호출 가능
* private_method → 오직 my_class 내부에서만 호출 가능
즉, '**선언 방식에 따라 인터페이스가 다르게 된다**'는 것은, 외부에서 볼 수 있는지 없는지가 달라진다는 뜻
---

### 클래스의 두 가지 구성요소
* Instance Component : 클래스를 참조하여 객체를 생성할 때마다 메모리에 생성되는 항목
  * Attribute는 `DATA` / Method는 `METHOD` 구문으로 선언
  * Attribute는 그냥 모든 ABAP 데이터 타입을 가질 수 있는 클래스의 내부 데이터 필드
* Static Component : 클래스 생성자(CREATE OBJECT 구문)을 만나면 프로그램이 종료될 때까지 메모리에 저장되는 클래스에 의존적인 항목
  * 클래스의 인스턴스와 상관없이 클래스 자체에 속하는 멤버(변수/메서드)를 의미하는듯 하다.
  * `CLASS-DATA` / `CLASS-METHOD` 구문으로 선언
 
```
DATA l_data TYPE C.
CLASS-DATA c_data TYPE i.
```

### 이벤트(Event)란?

* 한 객체가 “무슨 일이 일어났다”라고 알림을 보내는 것
  * 이벤트를 발생시키는 객체 → 이벤트 소유자(Event Source)
  * 이벤트에 반응하는 객체 → 이벤트 핸들러(Event Handler)
    * 이벤트를 발생시키는 객체와 반응하는 객체는 상속관계가 없어도 상호작용 가능

### 이벤트 사용 흐름
**(1) 이벤트 선언**
```
CLASS cl_example DEFINITION.
  PUBLIC SECTION.
    EVENTS my_event
      EXPORTING value1 TYPE i value2 TYPE string.
ENDCLASS.
```
my_event라는 이름으로 이벤트 선언
이벤트가 발생할 때 전달할 데이터(value1, value2)를 지정 가능


**(2) 이벤트 발생(Raise)**
```
METHOD trigger_event.
  RAISE EVENT my_event
    EXPORTING value1 = 100 value2 = 'Hello'.
ENDMETHOD.
```
이벤트는 소유한 객체 내 메서드에서 RAISE EVENT로 발생시킴(이 예시에서는 c1_example에서만 RAISE EVENT 할 수 있다는 말)
발생시키는 것 자체가 호출이 아님 → 이벤트에 반응하도록 연결된 핸들러들이 호출됨

**(3) 이벤트 핸들러 메서드 선언**
```
CLASS cl_handler DEFINITION.
  PUBLIC SECTION.
    METHODS handle_event
      FOR EVENT my_event OF cl_example
      IMPORTING value1 value2.
ENDCLASS.
```
이벤트가 발생했을 때 실제로 동작할 메서드를 정의
이벤트가 보내는 데이터(value1, value2)를 IMPORTING으로 받음

**(4) 핸들러 등록**
```
DATA: obj_example TYPE REF TO cl_example,
      obj_handler TYPE REF TO cl_handler.

CREATE OBJECT obj_example.
CREATE OBJECT obj_handler.

SET HANDLER obj_handler->handle_event FOR obj_example.
```
누가 이벤트를 받을지(obj_handler) 등록
이제 obj_example에서 이벤트 발생 시 obj_handler->handle_event가 자동으로 호출됨

### 흐름 정리

`[클래스 A] ---RAISE EVENT---> (이벤트 발생) -> [클래스 B] (SET HANDLER) -> handle_event 호출`

**A는 이벤트를 발생시킬 뿐, B가 어떻게 반응할지는 알 필요 없음**

B는 관심 있는 이벤트만 등록해서 처리

* 이벤트 = 알림(어떤 일이 발생했음을 알리는 신호)
* RAISE EVENT = 알림 발생
* Handler = 알림을 받고 처리하는 메서드
* SET HANDLER = 누가 알림을 받을지 등록

**이벤트 사용 목적 -> 객체 간 느슨한 결합(loose coupling), 즉 서로 직접적으로 연결되지 않고 상호작용 가능**
