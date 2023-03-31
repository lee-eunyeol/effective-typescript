# 5. any 다루기

## 아이템 38 any 타입은 가능한 한 좁은 범위에서만 사용하기

```tsx
interface Foo { foo: string; }
interface Bar { bar: string; }
declare function expressionReturningFoo(): Foo;
function processBar(b: Bar) { /* ... */ }

function f() {
  const x = expressionReturningFoo();
  processBar(x);
  //         ~ Argument of type 'Foo' is not assignable to
  //           parameter of type 'Bar'
}
```

- 위 코드 에러를 해결하기 위해 any 타입을 써보자.

```tsx
interface Foo { foo: string; }
interface Bar { bar: string; }
declare function expressionReturningFoo(): Foo;
function processBar(b: Bar) { /* ... */ }
function f1() {
  const x: any = expressionReturningFoo();  // Don't do this
  processBar(x);
}

function f2() {
  const x = expressionReturningFoo();
  processBar(x as any);  // Prefer this
}
```

- f1()과 같이 x를 any 타입으로 선언하지말고 f2() 와같이 가능한한 좁은 범위에서 any타입을 쓰자
    - 왜?
    
    ```tsx
    interface Foo { foo: string; }
    interface Bar { bar: string; }
    declare function expressionReturningFoo(): Foo;
    function processBar(b: Bar) { /* ... */ }
    function f1() {
      const x: any = expressionReturningFoo();
      processBar(x);
      return x;
    }
    
    function g() {
      const foo = f1();  // Type is any
      foo.fooMethod();  // This call is unchecked!
    }
    ```
    
    - 위처럼 f1()의 리턴값을 사용하다 했을때 x가 any타입으로 반환되어 정확했던 타입이 불분명해지는 결과를 낳는다.

```tsx
interface Foo { foo: string; }
interface Bar { bar: string; }
declare function expressionReturningFoo(): Foo;
function processBar(b: Bar) { /* ... */ }
function f1() {
  const x = expressionReturningFoo();
  // @ts-ignore
  processBar(x);
  return x;
}
```

- 다른 방식으로 @ts-ignore를 추가하여 에러를 해결할 수 있지만 근본적인 원인을 해결하는것이 아니기때문에 추천하지 않는다고 한다.

예시2

```tsx
interface Foo { foo: string; }
interface Bar { bar: string; }
declare function expressionReturningFoo(): Foo;
function processBar(b: Bar) { /* ... */ }
interface Config {
  a: number;
  b: number;
  c: {
    key: Foo;
  };
}
declare const value: Bar;
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value
  }
} as any;  // Don't do this! 이렇게 말고

//요렇게 쓰자
const config: Config = {
  a: 1,
  b: 2,  // These properties are still checked
  c: {
    key: value as any
  }
};

```

<aside>
💡 결론  : 의도치 않은 타입 안전성의 손실을 피하기 위해 any의 사용 범위를 최소한으로 좁히자

</aside>

---

## 아이템 39 any를 구체적으로 변형해서 사용하기

- any는 자바스크립트에서 표현할 수 있는 모든값을 아우르는 매우 큰 범위의 타입이다.
    - 반대로 말하면 일반적으로는 any를 대체할 수 있는 구체적인 타입이 존재하니 , 앵간해선 any쓰지말자.

- 아래와 같이 any에도 [] 형태의 타입을 추가하면 매개변수를 좀 더 구체화 하여 일부 타입을 추론 할 수 있다.
    - array.length 가 타입체크됨.
    - 제네릭에서 애용할것 같음

```tsx
function getLengthBad(array: any) {  // Don't do this!
  return array.length;
}

function getLength(array: any[]) {
  return array.length;
}
```

- 함수의 매개변수가 객체이지만 값을 알 수 없다면 , {[key: string]: any} 처럼 사용하여 표현할 수도 있다.

```tsx
function getLengthBad(array: any) {  // Don't do this!
  return array.length;
}

function getLength(array: any[]) {
  return array.length;
}
function hasTwelveLetterKey(o: {[key: string]: any}) {
  for (const key in o) {
    if (key.length === 12) {
      return true;
    }
  }
  return false;
}
```

- object타입을 사용할 수도 있다
    - object 타입은 객체의 키를 열거할 수는 있지만 속성에 접근할 수 없다는 점에서 , {[key: string]: any} 와 약간 다르다.

```tsx
function hasTwelveLetterKey(o: object) {
  for (const key in o) {
    if (key.length === 12) {
      console.log(key, o[key]);
                   //  ~~~~~~ Element implicitly has an 'any' type
                   //         because type '{}' has no index signature
      return true;
    }
  }
  return false;
}

```

<aside>
💡 결론 : 그냥 any보다는 적어도 조금더 구체적으로 추론할 수 있는 형태 ex) any[] , …args : any[] 등 을 사용할 수 있도록 하자

</aside>

---

## 아이템 40. 함수안으로 타입 단언문 감추기

- 함수를 작성하다 보면, 보통 외부로 드러난 타입 정의는 간단하지만 내부로직이 복잡해서 안전한 타입으로 구현하기 어려운 경우가 많다.
    - 그렇다고 불필요한 예외처리를 고려하며 타입정보를 힘들게 구성하기보단, 함수내부에서 타입 단언을 사용하고, 외부로 드러나는 타입 정의를 정확히 명시하는 정보로 끝내는게 낫다.
    
- 예시

```tsx
declare function shallowEqual(a: any, b: any): boolean;
function cacheLast<T extends Function>(fn: T): T {
  let lastArgs: any[]|null = null;
  let lastResult: any;
  return function(...args: any[]) {
      // ~~~~~~~~~~~~~~~~~~~~~~~~~~
      //          Type '(...args: any[]) => any' is not assignable to type 'T'
    if (!lastArgs || !shallowEqual(lastArgs, args)) {
      lastResult = fn(...args);
      lastArgs = args;
    }
    return lastResult;
  };
}
```

- 위 코드는  반환문에 있는 함수와 원본함수 T타입이 어떤관련이 있는지 알지 못하기 때문에 오류가 발생했다.

```tsx
declare function shallowEqual(a: any, b: any): boolean;
function cacheLast<T extends Function>(fn: T): T {
  let lastArgs: any[]|null = null;
  let lastResult: any;
  return function(...args: any[]) {
    if (!lastArgs || !shallowEqual(lastArgs, args)) {
      lastResult = fn(...args);
      lastArgs = args;
    }
    return lastResult;
  } as unknown as T;
}
```

- 위처럼 타입 단언문을 통해 오류를 제거해도 어차피 예상한 결과가 나오기 때문에 위 방식으로 오류를 해결해도 큰 문제가 되지않는다.
    - 또한, 함수 내부에서만 any를 사용했고 , 함수 외부로 보내는 타입은 any가 아니기 때문에 cacheLast함수를 사용하는쪽에서는 any가 사용됐는지 알지 못한다.

- shallowEqual 예시
    - (k in b)로 k 속성이 b에 있다는 것을 확인했지만, aVal !== b[k]부분에서 에러가 뜨는게 이상하다. (책에선 타입스크립트의 문맥활용능력이 부족한것 같다고 말한다)

```tsx
declare function shallowEqual(a: any, b: any): boolean;
function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
  for (const [k, aVal] of Object.entries(a)) {
    if (!(k in b) || aVal !== b[k]) {
                           // ~~~~ Element implicitly has an 'any' type
                           //      because type '{}' has no index signature
      return false;
    }
  }
  return Object.keys(a).length === Object.keys(b).length;
}
```

- 어차피 속성이 있다는것이 보장되기 때문에 이를 any 단언을 통해 해결할 수 있다

```tsx
declare function shallowEqual(a: any, b: any): boolean;
function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
  for (const [k, aVal] of Object.entries(a)) {
    if (!(k in b) || aVal !== (b as any)[k]) {
      return false;
    }
  }
  return Object.keys(a).length === Object.keys(b).length;
}
```

<aside>
💡 결론 : 불가피하게 any를 사용해야 한다면, 정확한 정의를 가지는 함수안으로 숨기자.

</aside>

---

## 아이템 41. any의 진화를 이해하기

- 타입스크립트에서 일반적으로 변수의 타입은 변수를 선언할때 결정되지만 ,any 타입과 관련해서 예외인 경우가 존재한다.
    
    ```tsx
    function range(start: number, limit: number) {
      const out = [];  // Type is any[]
      for (let i = start; i < limit; i++) {
        out.push(i);  // Type of out is any[]
      }
      return out;  // Type is number[]
    }
    ```
    
    - 위 코드를 보면 out 변수가 처음엔 any[]에서 number[] 로 추론되었음을 볼 수 있다.
        - any 타입은 다양한 타입이 들어감에 따라 타입이 진화(추론) 될 수 있다.
    
    ```tsx
    const result = [];  // Type is any[]
    result.push('a');
    result  // Type is string[]
    result.push(1);
    result  // Type is (string | number)[]
    ```
    
    - 분기문에서도 타입이 진화됨
    
    ```tsx
    let val;  // Type is any
    if (Math.random() < 0.5) {
      val = /hello/;
      val  // Type is RegExp
    } else {
      val = 12;
      val  // Type is number
    }
    val  // Type is number | RegExp
    ```
    
- any타입의 진화는 noImplicitAny가 설정된 상태에서 변수의 타입이 **암시적 any인 경우에만** 일어난다.
    - 따라서 암시적이 아닌, 명시적으로 any를 선언하면 타입이 그대로 유지된다.
    
    ```tsx
    let val: any;  // Type is any
    if (Math.random() < 0.5) {
      val = /hello/;
      val  // Type is any
    } else {
      val = 12;
      val  // Type is any
    }
    val  // Type is any
    ```
    
    - any 타입의 진화는 암시적 any 타입에 어떤값을 할당할때만 일어나고, 암시적 any 타입일때 값을 읽으려고 하면 오류가 발생한다.
    - 암시적 any 타입은 함수 호출을 거쳐도 진화하지 않는다.
    
    ```tsx
    function range(start: number, limit: number) {
      const out = [];  // Type is any[]
      for (let i = start; i < limit; i++) {
        out.push(i);  // Type of out is any[]
      }
      return out;  // Type is number[]
    }
    function makeSquares(start: number, limit: number) {
      const out = [];
         // ~~~ Variable 'out' implicitly has type 'any[]' in some locations
      range(start, limit).forEach(i => {
        out.push(i * i);
      });
      return out;
          // ~~~ Variable 'out' implicitly has an 'any[]' type
    }
    
    ```
    
    <aside>
    💡 any과 진화하는 방식은 일반적인 변수가 타입추론되는 원리와 비슷하다, 그러나 암시적 any를 진화시키는 방식보다 , 명시적 타입구문을 사용하는것을 추천한다.
    
    </aside>
    
    ---
    

## 아이템 43. 몽키 패치보다는 안전한 타입을 사용하기

> 몽키 패치란?
> 

몽키패치는 **원래 소스코드를 변경하지 않고 실행 시 코드 기본 동작을 추가, 변경 또는 억제하는 기술**
이다. 쉽게 말해 어떤 기능을 위해 이미 있던 코드에 삽입하는 것이다

- 자바스크립트의 가장 큰 특징중 하나는, 객체와 클래스에 임의의 속성을 추가할 수 있을 만큼 유연하다.
    - 이런기능은 window나 document에 값을 할당하여 전역변수를 만드는데 쓰거나
    
    ```tsx
    window.monkey = 'Tamarin';
    document.monkey = 'Howler';
    ```
    
    - DOM 엘리먼트에 데이터를 추가하기 위해서도 사용한다.
    
    ```tsx
    const el = document.getElmentById('colobus')
    el.home = 'tree';
    ```
    
- 심지어 내장 기능의 프로토타입에도 속성을 추가할 수 있다. 그런데 이상한 결과를 보일때가 있다.
    
    ```tsx
    RegExp.prototype.monkey = 'Capuchin'
    >'Capuchin'
    /123/.monkey
    >'Capuchin'
    ```
    
    - 정규식(/123/)에 monkey를 추가한적이 없는데 `Capuchin` 이 나오는 것이다.
    
- 사실 객체의 임의의 속성을 추가하는것은 일반적으로 좋은 설계가 아니다
    - ex) window나 DOM 노드에 데이터를 추가한다고 가정했을때, 그 데이터는 기본적으로 전역변수가 된다.
        - 전역변수를 사용하면 은연중에 프로그램 내에서 서로 멀리 떨어진 부분들 간에 의존성을 만들게 된다. (내가 모르는 부분에서 이 변수를 쓰고 있을 수 있다.)
    - 심지어 Typescript는 임의로 추가한 속성이 타입 체킹 되지 않기 때문에 오류를 발생시킨다.
    
    ```tsx
    document.monkey = 'Tamarin';
          // ~~~~~~ Property 'monkey' does not exist on type 'Document'
    ```
    
    - 차선책
        1. **해당 전역변수에서 데이터를 분리할 수 있는 경우 분리하자!**
        2. **interface의 특수 기능중 하나인 보강을 이용하자**
            
            ```tsx
            interface Document {
              /** Genus or species of monkey patch */
              monkey: string;
            }
            
            document.monkey = 'Tamarin';  // OK
            ```
            
            **장점**
            
            - 보강을 이용하면 타입이 안전해진다.
            - 인터페이스 속성에 주석을 붙일 수 있다.
            - 자동완성 기능이 제공된다.
            - 몽키패치가 어떤 부분에 적용되었는지 정확한 기록이 남는다.
            
            모듈 관점에서 제대로 동작하게 하려면 global 선언을 추가해야 한다
            
            [Global에 대해](https://velog.io/@hosickk/Typescript-declare)
            
            ```tsx
            export {};
            declare global {
              interface Document {
                /** Genus or species of monkey patch */
                monkey: string;
              }
            }
            document.monkey = 'Tamarin';  // OK
            ```
            
            **주의할 점**
            
            1. 보강은 전역적으로 적용되기 때문에, 코드의 다른 부분이나 라이브러리로부터 분리할 수 없다.
            2. 애플리케이션이 실행되는동안 속성을 할당하면 해당 실행시점에서 보강을 적용할 수 없다.
                1. 특정웹페이지의 HTML엘리먼트를 조작할때, 어떤 엘리먼트는 속성이 있고 어떤 엘리먼트는 없을 수 있다. → 타입을 string | undefined 로 선언하게 되는데 이는 다루기 불편해진다.
        
        1. **더 구체적인 타입 단언문을 사용하자.**
            
            ```tsx
            interface MonkeyDocument extends Document {
              /** Genus or species of monkey patch */
              monkey: string;
            }
            
            (document as MonkeyDocument).monkey = 'Macaque';
            ```
            
            MonkeyDocument는 Document를 확장하기 때문에 타입 단언문은 정상이며 타입이 안전하게 된다.
            

<aside>
💡 전역변수나 DOM에 데이터를 저장하지 말고 데이터를 분리하자

</aside>

---

## 아이템 44. 타입 커버리지를 추적하여 타입 안전성 유지하기

- noImplicitAny를 설정하고 모든 암시적 any 대신 명시적 타입 구문을 추가 해도  타입과 관련된 문제들로부터 안전하다고 할 수 없다.
    - 명시적 any 타입
        - [아이템 38,39](https://www.notion.so/5-any-55c383b6ac6a453aaf4b9fea351915de) 에 따라 any 타입의 범위를 좁혀도 any 타입이다.
    
    - 서드파티 타입 선언
        - 이 경우는 @types로부터 any 타입이 전파되기 때문에 조심해야 한다.
        

type-cover-age 패키지를 통해 any타입을 추적할 수 있다.

```bash
npx type-coverage --detail
```

- 이걸로 any타입을 얼마나 적절히 사용하여 설계했는지 알 수 있다.
    
    ![Screen Shot 2023-01-08 at 10.04.46 AM.png](5%20any%20%E1%84%83%E1%85%A1%E1%84%85%E1%85%AE%E1%84%80%E1%85%B5%2055c383b6ac6a453aaf4b9fea351915de/Screen_Shot_2023-01-08_at_10.04.46_AM.png)
    
    ![Screen Shot 2023-01-08 at 10.05.45 AM.png](5%20any%20%E1%84%83%E1%85%A1%E1%84%85%E1%85%AE%E1%84%80%E1%85%B5%2055c383b6ac6a453aaf4b9fea351915de/Screen_Shot_2023-01-08_at_10.05.45_AM.png)
    

> 이외 서드파티 라이브러리로부터 비롯 되는 any타입의 발생에 대한 예시를 설명
> 

<aside>
💡 any타입은 다양한 이유로 발생할 수 있다 . type-coverage 같은 라이브러리로 any 타입을 줄여보자

작성한 프로그램의 타입이 얼마나 잘 선언되었는지 추적해야 한다.

</aside>

---