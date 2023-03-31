# 1. 타입스크립트 알아보기

## 아이템 1 . 타입스크립트와 자바스크립트의 관계 이해하기

- 타입스크립트의 타입 시스템의 목표중 하나는 런타임에 발생할 오류를 미리 찾아 내는것이다.
    - 타입스크립트가 `정적 타입 시스템` 이라는 것은 바로 이런 특징을 말한다.
        - 그러나 타입 체커가 모든 오류를 찾아내지는 않는다.
        
- 타입 스크립트는 자바스크립트 런타임 동작으로 모델링 된다. 그러나, 타입스크립트가 모든 런타임 오류를 잡을것이라 생각하면 안된다.
    
    ```tsx
    const x = 2 + '3' //정상 --> javascript 런타임에선 '23'문자열이 되는 변수로 인식됨
    ```
    
    - 다른 정적언어에서는 에러를 발생시키는 코드이지만, 자바스크립트 런타임에서는 문제가 되지 않는 부분은 타입스크립트도 에러를 발생시키지 않는다.
    - 반대로 , 런타임에서는 정상이지만 타입스크립트에서 오류가 발생하는경우가 있다.
        
        ```tsx
        const a = null + 7 //에러 --> javascript에선 a가 7이지만 타입스크립트는 숫자형에 null
        									 //을 더할 수 없다는 에러가 뜸
        ```
        

---

## 아이템 2 타입스크립트 설정 이해하기

- 타입스크립트 컴파일러는 100개에 이르는(어쩌면 그 이상?) 매우 많은 설정을 가지고 있다.
    - `noImplicit Any`  :: Implicit(암시적) any를 허용할지 안할지
        - 변수들이 미리 정의된 타입을 가져야하는지 체크
        
        ```tsx
        function(a,b){
        	return a + b;	
        }
        //noImplicit Any
        function(a : number,b : number){
        	return a + b;	
        }
        ```
        
        되도록이면 설정하자
        
    - `strictNullCheck`
        - null 과 undefined가 모든 타입에서 허용되는지
        
        ```tsx
        const x : number = null
        
        //strictNullCheck 
        const x : number = null --> number 타입은 null을 할당할 수 없음.
        ```
        
    

---

## 아이템 3. 코드 생성과 타입이 관계 없음을 이해하기

- 타입스크립트 컴파일러는 크게 두가지 역할을 수행한다.
    - 최신 타입스크립트 / 자바스크립트를 브라우저에서 동작할 수 있도록 구 버전의 자바스크립트로 트랜스파일
    - 코드타입의 오류 체크

### 타입 오류가 있는 코드도 컴파일이 가능하다.

- 컴파일은 타입체크와 독립적으로 동작하기 때문에 타입오류가 있는 코드도 컴파일이 가능하다.
    - 자바와 C 같이 타입체크와 컴파일이 동시에 이루어지는게 아니라.
    - 타입스크립트 오류는 Warning과 비슷하게 문제가 될 만한 부분을 알려주지만 빌드를 멈추지는 않는다.

- 오류가 있을때 컴파일 하지 않으려면 tsconfig.json에 `noEmitOnError`를 설정하자

### 런타임에는 타입체크가 불가능하다.

- 타입스크립트의 타입은 제거가능하다.
    - 자바스크립트로 컴파일 되는 과정에서 **모든 인터페이스 / 타입 / 타입 구문은 제거된다.**

- 그렇다면, 런타입에 타입을 유지하는 방법?
    - 태그 기법
        - 타입을 구분지을수 있는 `kind` 를 추가하여 타입을 구분한다.
            - 별로..
        
        ```tsx
        interface Square {
          kind: 'square';
          width: number;
        }
        interface Rectangle {
          kind: 'rectangle';
          height: number;
          width: number;
        }
        type Shape = Square | Rectangle;
        
        function calculateArea(shape: Shape) {
          if (shape.kind === 'rectangle') {
            shape;  // Type is Rectangle
            return shape.width * shape.height;
          } else {
            shape;  // Type is Square
            return shape.width * shape.width;
          }
        }
        ```
        
    - 클래스를 사용
        
        ```tsx
        class Square {
          constructor(public width: number) {}
        }
        class Rectangle extends Square {
          constructor(public width: number, public height: number) {
            super(width);
          }
        }
        type Shape = Square | Rectangle;
        
        function calculateArea(shape: Shape) {
          if (shape instanceof Rectangle) {
            shape;  // Type is Rectangle
            return shape.width * shape.height;
          } else {
            shape;  // Type is Square
            return shape.width * shape.width;  // OK
          }
        }
        ```
        
        - 인터페이스는 타입으로만 사용하지만, 클래스는 타입 , 값으로 모두 사용할 수 있다.
        

### 런타임 타입은 타입스크립트에서 선언된 타입과 다를 수 있다.

타입스크립트에서는 런타임 타입과 선언된 타입이 맞지 않을 수 있다.

선언된 타입이 언제든지 달라질 수 있다는 것을 명심하자.

---

## 아이템 4. 구조적 타이핑에 익숙해지기

- 자바스크립트는 기본적으로 덕타이핑 기반이기 때문에 타입스크립트는 이를 모델링 하기 위해 구조적 타이핑을 사용한다 . 타입간 호환만 된다면 같은 메소드를 사용할 수 있다.
    
    ```tsx
    interface Vector2D {
    	x : number;
    	y : number;
    }
    
    interface NamedVector {
    	name : string;
    	x : number;
    	y : number;
    }
    
    function calculateLength(v : Vector2D){
    	return Math.sqrt(v.x*v.x + v.y*v.y)
    }
    
    calculateLength(NamedVector) // NamedVector타입 사용가능 -> Vector2D타입과 호환가능하기 때문
    
    ```
    
- 구조적 타이핑 규칙들은 어떠한 값이 다른속성도 가질 수 있음을 의미한다.
- 구조적 타이핑을 사용하면 유닛테스팅을 손쉽게 할 수 있다.
    
    ```tsx
    interface PostgresDB { 
    //실제 PostgresDB의 세부내역을 가진 모킹객체가 없어도, 인터페이스로 runQuery 메소드만 정의하여 테스트함
    //이렇게 일부만 구현(구조적 타이핑)해도 실제로도 같은 메소드가 있어 실제 동작엔 문제가 없음.
      runQuery: (sql: string) => any[];
    } 
    interface Author {
      first: string;
      last: string;
    }
    function getAuthors(database: PostgresDB): Author[] {
      const authorRows = database.runQuery(`SELECT FIRST, LAST FROM AUTHORS`);
      return authorRows.map(row => ({first: row[0], last: row[1]}));
    }
    ```
    

---

## 아이템 5. Any타입 지양하기

- Any 타입에는 타입 안정성이 없다. , 함수 시그니처를 무시할 수 있다.
    - 인자를 any로 선언해버리면 number를 받아야하는 메서드에 string을 넣을수도 있다.
        - 이는 혼돈의 씨앗이 된다.
        
        ```tsx
        function add(x : number) {
            
        }
        
        add('1' as any)
        ```
        

- Any 타입은 언어 서비스가 지원이 되지않는다.
    - 타입에 따라 지원할 수 있는 자동완성기능이 적용되지 않는다.
        - 변수명 심볼 동시에 바꾸기 같은 기능이 제공안된다.
        
        ```tsx
        interface Person {
            first: string;
            last: string;
          }
          const formatName = (p: Person) => `${p.first} ${p.last}`;
          const formatNameAny = (p: any) => `${p.first} ${p.last}`; 
        // --> p는 any라서 first 심볼을 firstName으로 바꾸어도 formatNameAny의 first는 바뀌지 않는다.
        ```
        

- Any 타입은 리팩토링시 버그를 감춘다.
- Any는 타입설계를 감춘다
    - 클래스 설계시 정확한 타입을 사용해서 정확하고 명료한 코드작성이 되는데 타입설계가 되지않은채 any타입을 설정하면 , 설계가 잘 되었는지 ,설계가 어떻게 되어있는지 전혀 알 수 없다.
- Any는 타입시스템의 신뢰도를 떨어뜨린다.