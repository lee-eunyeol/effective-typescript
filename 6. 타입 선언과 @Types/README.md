# 타입 선언과 @Types

## 아이템 45. devDependencies에 typescript와 @types 추가하기

- npm은 자바스크립트 세상에서 필수적이며 , 자바스크립트 라이브러리저장소 ( npm 레지스트리 ) 와 프로젝트가 의존하고 있는 라이브러리들의 버전을 지정하는 방법(package.json)을 제공한다.
    - dependencies
        - 현재 프로젝트를 실행하는데 필수적인 라이브러리가 포함됨
            - 다른사람이 이 프로젝트를 설치하면 dependencies에 들어 있는 라이브러리들도 함께 설치됨
            
            <aside>
            💡 이를 전이 의존성이라고도 한다.
            
            </aside>
            
    - devDependencies
        - 현재 프로젝트를 개발하고 테스트하는 데 사용되지만 런타임에는 필요없는 라이브러리
            - 테스트 프레임워크 같은 라이브러리
            - 다른 사용자가 이 프로젝트를 설치해도 요건 설치 안됨
            - 타입스크립트와 관련된 라이브러리
            
    - peerDependencies
        - 런타임에 필요하긴 하지만 의존성을 직접 관리하지 않는 라이브러리
            - ex) 플러그인
    

### 모든 타입스크립트 프로젝트에서 공통적으로 고려해야 할 의존성 2가지

1. 타입스크립트 자체 의존성
    - 타입스크립트를 시스템 레벨로 설치할 수 있지만 추천하지 않는다.
        - 팀원들 모두가 항상 동일한 버전을 사용한다는 보장이 없다
        - 프로젝트를 셋업할 때 별도의 단계가 추가된다.
    
    > 웬만하면 devDependencies에 typescript를 추가하자
    > 

1. 타입 의존성(@types)를 고려해야 한다.
    - 사용하는 라이브러리에 타입선언이 포함되어 있지 않더라도, DefinitelyTyped에서 타입정보를 얻을 수 있다.
        - DefinitelyTyped에서 정한 타입들은 @types 스코프에 공개된다.
            - ex) @types/jquery

---

## 아이템 46. 타입 선언과 관련된 세가지 버전 이해하기

- 프로젝트에서 라이브러리를 추가할때, 라이브러리의 전이적 의존성이 호환되는지 깊게 생각하지 않았을 것이다.
    - 실제로 타입스크립트는 알아서 의존성 문제를 해결해주기는 커녕, 의존성 관리를 더 복잡하게 만든다.
        - 왜? 타입스크립트를 사용하게 되는 순간 세가지 사항을 추가로 고려해야한다.
            1. 라이브러리 버전
            2. 타입 선언 버전 (@types)
            3. 타입스크립트 버전
        
        이 중 하나라도 맞지 않으면, 의존성과 상관없어 보이는 곳에서 오류를 발생시킨다.
        

- 타입스크립트에서 일반적으로 의존성을 사용하는 방식
    - 특정 라이브러리 설치
    
    ```bash
    npm install react
    react@16.8.6
    ```
    
    - 타입 정보 설치
    
    ```bash
    npm install --save-dev @types/react
    @types/react@16.8.19
    ```
    
    - 각 라이브러리 설치 버전을 메이저와 마이너 버전을 일치하지만 패치버전은 일치하지않는다.
    - 그래도, react가 시맨틱 버전 규칙을 제대로 지켰다면 둘 다 16.8 공개 API 사양을 준수 하기 때문에 큰 문제가 있진 않을 것이다.
        - 그러나 실제 라이브러리와 타입정보의 버전이 별도로 관리되는 방식은 4가지 문제가 있다.
- 4가지 문제
    1. 라이브러리를 업데이트 했지만 실수로 타입 선언은 업데이트 하지 않은 경우
        - 하휘호환성이 깨지는 변경이 있었다면 타입 체커를 통과하더라도 런타임 오류가 발생할 수 있음
            
            해결
            
            1. 타입 선언 버전 업데이트
            2. 보강기법을 활용하여 사용하려는 새 함수와 메서드 타입정보를 추가
            3. 타입선언의 업데이트 버전을 직접 작성한 후 공개
    2. 라이브러리보다 타입 선언 버전이 최신인 경우
        - 라이브러리 버전 업데이트 또는 타입 선언 버전 업데이트
    
    1. 프로젝트에서 사용하는 타입스크립트 버전보다 라이브러리에서 필요로 하는 타입스크립트 버전이 최신인 경우
        - 

---

## 아이템 47. 공개 API 등장하는 모든 타입을 익스포트하기

<aside>
💡 라이브러리를 만들때 굳이 **공게 메서드에 등장한 타입**을 숨기지 말고 모두 export 하여 라이브러리 사용자가 쉽게 타입을 이용할 수 있도록 하자

</aside>

---

## 아이템 48. API 주석에 tsDoc 사용하기

- 주석을 쓰는것은 함수 이해를 도울 수 있어 좋지만 아래와 같이 쓰면 jsDoc을 활용할 수 없다.

```tsx
// Generate a greeting. Result is formatted for display.
function greet(name: string, title: string) {
  return `Hello ${title} ${name}`;
}
```

- 이렇게 jsDoc 스타일로 주석을 적으면, 함수를 호출할때, jsdoc스타일의 주석이 툴팁으로 표시되기 때문에 용이하다.

```tsx
/** Generate a greeting. Result is formatted for display. */
function greetJSDoc(name: string, title: string) {
  return `Hello ${title} ${name}`;
}
```

- TSDoc 스타일은 마크다운 형식을 지원하기 때문에 굵은글씨 기울임 등을 사용할 수 있다.
    - 타입스크립트에서는 타입 정보가 코드에 있기때문에 타입정보를 명시하지는 말자.

```tsx

/**
 * This _interface_ has **three** properties:
 * 1. x
 * 2. y
 * 3. z
 */
interface Vector3D {
  x: number;
  y: number;
  z: number;
}
```

<aside>
💡 주석은 JsDoc/TSDoc 형태로 작성하되, 주석에 타입정보를 표현하지는 말자

</aside>

---

## 아이템 49. 콜백에서 this에 대한 타입 제공하기

- 자바스크립트에서의 this 기능은 매우 혼란스러운 기능이다
    - let 이나 const 는 [`lexical scope`](https://ljtaek2.tistory.com/145) 인 방면, this는 `dynamic scope` 이다.
        - `dynamic scope` 의 값은 정의된 방식이 아닌, 호출된 방식에 따라 달라진다.
    
- this는 일반적으로 클래스에서 사용된다.
    
    ```tsx
    class C {
      vals = [1, 2, 3];
      logSquares() {
        for (const val of this.vals) {
          console.log(val * val);
        }
      }
    }
    
    const c = new C();
    c.logSquares();
    //1, 4, 9
    ```
    
- 그런데, 아래처럼 외부변수에 메소드를 할당하면 어떻게 될까?
    
    ```tsx
    class C {
      vals = [1, 2, 3];
      logSquares() {
        for (const val of this.vals) {
          console.log(val * val);
        }
      }
    }
    const c = new C();
    const method = c.logSquares;
    method();
    ```
    
    ```tsx
    undefined 의 vals 속성을 읽을 수 없습니다 
    ```
    
    라는 런타임 오류가 발생한다
    
    ### 왜?
    
    - c.logSquares() 함수는 실제로 두가지 일을 수행한다
        1. C.prototype.logSquares() 를 호출한다.
        2. c 에 this를 바인딩 한다.
    
    ### 그런데..
    
    - 앞의 코드에서는 logSquares의 참조변수를 사용함으로써 두가지 일을 분리했고, this의 값은 undefined로 설정된다.
    
    > 추가 내용을 보았는데 this 바인딩에 내용이 너무 어렵게 기술되어있었다
    따라서 책에서 언급하는 내용보다는 블로그가 내용을 이해하는데 더 도움이 되어 참조하고 마친다.
    > 
    
    [[JS] 알쏭달쏭 자바스크립트 this 바인딩](https://seungtaek-overflow.tistory.com/21)
    
    [[Javascript] this와 바인딩](https://iamsjy17.github.io/javascript/2019/06/07/js33_15_this.html)
    
    <aside>
    💡 this 바인딩이 동작하는 원리를 이해하자.
    
    콜백함수에서 this를 사용해야 한다면 타입정보를 명시하자.
    
    </aside>
    

- 이 this 바인딩 문제는 call() 로 해결할 수 있다.
    
    ```tsx
    class C {
      vals = [1, 2, 3];
      logSquares() {
        for (const val of this.vals) {
          console.log(val * val);
        }
      }
    }
    const c = new C();
    const method = c.logSquares;
    method.call(c);  // Logs the squares again
    ```
    

---

## 아이템 50. 오버로딩 타입보다는 조건부 타입을 사용하기

- 아래의 double 함수에는 string 또는 number 타입의 매개변수가 들어올 수 있다.

```tsx
function double(x: number|string): number|string;
```

- 그러나 이는 모호한 부분이 있다.
    - 아래처럼 12라는 number를 넣었을때, 의도했던 number를 반환 하는것이 아닌 string | number union을 반환한다.
    
    ```tsx
    const num = double(12);  // string | number
    const str = double('x');  // string | number
    ```
    

### 잘못된 방법들..

- 제너릭을 사용해볼 수 있다.
    
    ```tsx
    function double<T extends number|string>(x: T): T;
    function double(x: any) { return x + x; }
    
    const num = double(12);  // Type is 12
    const str = double('x');  // Type is "x"
    ```
    
    - 그런데, 이는 너무 과해서 의도했던 number 또는 string이 아닌 리터럴 문자열이 반환되었다.
    
- 여러가지 타입선언으로 분리해볼 수도 있다.
    
    ```tsx
    function double(x: number): number;
    function double(x: string): string;
    function double(x: any) { return x + x; }
    
    const num = double(12);  // Type is number
    const str = double('x');  // Type is string
    ```
    
    - 함수타입은 명확해졌지만, 유니온 타입을 인자로 받을때 문제가 생긴다
        
        ```tsx
        function f(x: number|string) {
          return double(x);
                     // ~ Argument of type 'string | number' is not assignable
                     //   to parameter of type 'string'
        }
        ```
        

### 해결책

- 조건부 타입을 이용해보자

```tsx
function double<T extends number | string>(
  x: T
): T extends string ? string : number;
function double(x: any) { return x + x; }
```

- 조건부타입은 자바스크립트의 삼항 연산자(?:) 처럼 사용하면 된다
    - 조건부 타입은 앞선 모든 예제가 동작한다
        - 타입이 더 정확해졌다.

<aside>
💡 오버로딩 타입보다 조건부 타입을 사용하는것이 문제를 해결하기 더 좋다.

</aside>

---

## 아이템 51. 의존성 분리를 위해 미러 타입 사용하기

- 만약 작성중인 라이브러리가 의존하는 라이브러리의 구현과 무관하게 타입에만 의존한다면, 필요한 선언부만 추출하여 작성중인 라이브러리에 넣는것(미러링)을 고려해보는것도 좋다.
    - 특정사용자 ( 웹 기반이나 자바스크립트만 사용하는 사람 ) 에게 불필요하거나 사용할 수 없는 라이브러리(ex) types/node의 Buffer 타입)를 의존하도록 제공할 순 없기 떄문에
        
        ```tsx
        function parseCSV(contents: string | Buffer): {[column: string]: string}[]  {
          if (typeof contents === 'object') {
            // It's a buffer
            return parseCSV(contents.toString('utf8'));
          }
          // COMPRESS
          return [];
          // END
        }
        
        -> Buffer 타입을 미러링한 CsvBuffer //구조적 타이핑을 이용하여 Buffer 처럼 쓸 수 있게 되었다.
        interface CsvBuffer {
          toString(encoding: string): string;
        }
        function parseCSV(contents: string | CsvBuffer): {[column: string]: string}[]  {
          // COMPRESS
          return [];
          // END
        }
        ```
        

- 그러나, 프로젝트가 다른 라이브러리 타입선언의 대부분을 추출해야 한다면 차라리 명시적으로 @types 의존성을 추가하는게 낫다

<aside>
💡 필수가 아닌 의존성을 분리할대는 구조적 타이핑(::미러링)을 이용하자

공개한 라이브러리를 사용하는 자바스크립트 사용자가 @types의존성을 가지거나

웹 개발자가 Nodejs 관련 의존성을 가지지 않도록 하자

</aside>

---

## 아이템 52. 테스팅 타입의 함정에 주의하기

- 프로젝트를 공개할때, 테스트 코드를 작성하는것은 필수이며, 타입선언도 테스트를 거쳐야 한다.
    - 그러나, 타입선언을 테스트하기에는 매우 어렵다.
        - 궁극적으로는 dtslint 또는 타입 시스템 외부에서 타입을 검사하는 유사한 도구를 사용하는것이 안전하고, 간단하다

> 여기까지 읽으면서, 책의 후반부는 뭔가 주로 라이브러리 작성에 관해서 나와서 실무에 잘 이용될지는 모르겠다..
> 

- 유틸리티 라이브러리에서 제공하는 map 함수의 타입선언을 작성한다고 가정해보자
    
    ```tsx
    declare function map<U, V>(array: U[], fn: (u: U) => V): V[];
    ```
    
    - 타입선언이 예상한 타입으로 결과를 내는지 체크할 수 있는 한가지 방법은 테스트 파일을 작성 하는것이다.
    
    ```tsx
    map(['2017', '2018', '2019'], v => Number(v));
    ```
    
    - 그러나 이 코드는 허점이 존재하는데, map의 첫번째 매개변수에 배열이 아닌 단일 값이 있엇다면, 매개변수 타입에 대한 오류는 잡을 수 있다.
        - 그러나 반환갑에 대한 체크가 누락되어 완전한 테스트라고 볼 수 없다.
        
        ```tsx
        const square = (x: number) => x * x;
        test('square a number', () => {
          square(1);
          square(2);
        });
        ```
        
        - 비슷한 예로 square 함수를 테스트 한다 했을때, 위 테스트 코드는 `함수의 실행`에서 문제가 되지는 않지만, 반환값에 대해서는 체크하지 않기 때문에 `함수의 실행에 대한 결과` 는 테스트 되지 않는것이다.
- 반환값을 특정 타입의 변수에 할당하여 반환타입을 체크할 수 있는 방법
    - 테스팅을 위해 할당을 사용하는 방법
    
    ```tsx
    declare function map<U, V>(array: U[], fn: (u: U) => V): V[];
    const lengths: number[] = map(['john', 'paul'], name => name.length);
    ```
    
    - 이는 두가지의 문제가 있다.
        1. 불필요한 변수를 만들어야 한다.
            1. 이는 일부 린팅규칙(미사용 변수 경고)를 비활성화 해야한다.
            - 해결책 : 헬퍼함수를 정의
                
                ```tsx
                const square = (x: number) => x * x;
                declare function map<U, V>(array: U[], fn: (u: U) => V): V[];
                function assertType<T>(x: T) {}
                
                assertType<number[]>(map(['john', 'paul'], name => name.length));
                ```
                
            - 불필요한 변수를 해결했지만 또 다른 문제가 있다. (2번 참고)
        2. 두 타입이 동일한지 체크하는것이 아닌, 할당가능성을 체크하기 때문에 문제가 된다
            
            예시
            
            ```tsx
            function assertType<T>(x: T) {}
            const add = (a: number, b: number) => a + b;
            assertType<(a: number, b: number) => number>(add);  // OK
            
            const double = (x: number) => 2 * x;
            assertType<(a: number, b: number) => number>(double);  // OK!?
            ```
            
            - 위 함수 체크가 성공하는 이유는 타입스크립트의 함수는 매개변수가 더 적은 함수타입에 할당 가능하기 때문이다.
                
                ```tsx
                const g: (x: string) => any = () => 12;  // OK
                ```
                
            - 해결책: Parameters 와 ReturnType 제너릭 타입을 이용해 함수의 매개변수 타입과 반환 타입을 분리하여 테스트
            
            ```tsx
            declare function map<U, V>(array: U[], fn: (u: U) => V): V[];
            function assertType<T>(x: T) {}
            const double = (x: number) => 2 * x;
            let p: Parameters<typeof double> = null!;
            assertType<[number, number]>(p);
            //                           ~ Argument of type '[number]' is not
            //                             assignable to parameter of type [number, number]
            let r: ReturnType<typeof double> = null!;
            assertType<number>(r);  // OK
            ```
            

- 돌아와서 앞서 등장한 테스트는 세부사항을 테스트하는 whitebox 테스트가 아닌, 매개 변수와 반환타입만 테스트하는 blackbox 테스트였다.
    - 세부사항을 테스트 해보자
    
    ```tsx
    const beatles = ['john', 'paul', 'george', 'ringo'];
    assertType<number[]>(map(
      beatles,
      function(name, i, array) {
    // ~~~~~~~ Argument of type '(name: any, i: any, array: any) => any' is
    //         not assignable to parameter of type '(u: string) => any'
        assertType<string>(name);
        assertType<number>(i);
        assertType<string[]>(array);
        assertType<string[]>(this);
                          // ~~~~ 'this' implicitly has type 'any'
        return name.length;
      }
    ));
    ```
    
    - 위 코드는 콜백함수에서 몇가지 문제가 발생했다.
    
    <aside>
    💡 보충 필요!
    
    </aside>
    
- 타입 시스템 내에서 any타입을 발견하는것은 매우 어렵기 때문에 타입체커와 독립적으로 동작하는 도구를 이용해서 타입선언을 테스트 하는방법이 권장된다.
    - DefinitelyType의 타입선언을 위한 도구 → dtslint
    
    ```tsx
    declare function map<U, V>(
      array: U[],
      fn: (this: U[], u: U, i: number, array: U[]) => V
    ): V[];
    const beatles = ['john', 'paul', 'george', 'ringo'];
    map(beatles, function(
      name,  // $ExpectType string
      i,     // $ExpectType number
      array  // $ExpectType string[]
    ) {
      this   // $ExpectType string[]
      return name.length;
    });  // $ExpectType number[]
    ```
    
    - dtslint는 할당 가능성을 체크하는 대신, 각 심벌의 타입을 추출하여 글자 자체가 같은지 비교한다.
        - 이 비교과정은 편집기에서 타입선언을 눈으로 보고 확인하는것과 같은데, dtslint는 이를 자동화한다
            - 대신, 이는 단점이 있는데 , string|number와 number|string 유니온 타입은 같은 타입이지만 글자 자체를 비교하는 dtslint는 이를 다르다고 인식하기 때문에 유니온타입의 타입 나열순서를 맞춰줘야한다.

<aside>
💡 타입을 테스트할때, 함수타입의 동일성과 할당 가능성의 차이를 알고 있어야 한다.

콜백이 있는 함수를 테스트 할 때, 매개변수의 추론된 타입을 체크해야한다. this가 API 의 일부라면, 역시 테스트해야 한다.

타입관련된 테스트에서 any를 주의해야 하며, 더 엄격한 테스트를위해 dtslint를 사용하자.

</aside>

---