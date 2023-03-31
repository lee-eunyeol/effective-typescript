# 3. 타입 추론

## 아이템 19. 추론 가능한 타입을 사용해 장황한 코드 방지하기

- 추론이 가능하다면 굳이 타입을 명시할 필요가 없다.

```tsx
let x : number = 12;
let x = 12; //알아서 타입스크립트가 number 타입을 추론함
```

예시2

![Screen Shot 2022-11-18 at 3.51.18 PM.png](3%20%E1%84%90%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%B8%20%E1%84%8E%E1%85%AE%E1%84%85%E1%85%A9%E1%86%AB%201440d7faaaa2478b837f8e2c5301b820/Screen_Shot_2022-11-18_at_3.51.18_PM.png)

리턴 타입을 굳이 명시하지 않고, 타입스크립트에서 추론한 타입을 이용하는게 더 정확하고 이해하기 쉽다.

- 비 구조화 할당문은 모든 지역번수의 타입이 추론되도록 한다.
    - 따라서 명시적 타입 구문을 넣는것은 불필요하고 장황해진다.

![Screen Shot 2022-11-18 at 3.54.02 PM.png](3%20%E1%84%90%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%B8%20%E1%84%8E%E1%85%AE%E1%84%85%E1%85%A9%E1%86%AB%201440d7faaaa2478b837f8e2c5301b820/Screen_Shot_2022-11-18_at_3.54.02_PM.png)

- 정보가 부족하여 타입스크립트가 타입을 판단하기 어려운 상황에서는 명시적 타입 구문이 필요하다.
    - 당연한 이야기라 생각함..
- 이상적인 타입스크립트 코드는 함수/메서드 시그니처에 타입구문을 포함하지만, 함수 내에서 생성된 지역변수에는 타입 구문을 넣지 않는다.
    - 코드를 읽는 사람이 구현 로직에 집중할 수 있다.

```tsx
async function translateEnglishText({
  translateClient,
  textFormat,
  originEnglishText,
  targetLanguageCode,
}: {
  translateClient: TranslateClient
  textFormat: string | null
  originEnglishText: any
  targetLanguageCode: string
}) {
  if (textFormat === 'RTF') {
    const formatedText = originEnglishText
      ?.replace(/\\par[d]?/g, '')
      .replace(/\{\*?\\[^{}]+}|[{}]|\\\n?[A-Za-z]+\n?(?:-?\d+)?[ ]?/g, '')
      .trim()
    const cmd = new TranslateTextCommand({
      SourceLanguageCode: 'en',
      TargetLanguageCode: targetLanguageCode,
      Text: formatedText,
    })
    return {
      englishText: formatedText,
      ...(await translateClient.send(cmd).catch(() => null)),
    }
  }

  return null
}
```

- 타입이 추론될 수 있음에도 여전히 타입을 명시하고 싶은 경우
    - 객체 리터럴을 정의할때
        
        ```tsx
        const elmo:Product = {
        name: 'Tickle Me Elmo',
        id : '1234',
        price : 28.99}
        ```
        
        - 잉여 속성 체크가 들어가 선택적 속성이 있는 타입의 오타 같은 오류를 잡을 수 있다.
        - 할당되는 시점에 오류가 표시되도록 한다
        - 위의 타입구문을 제거하면, 잉여속성 체크가 동작하지 않고, 객체가 선언된곳이아닌 객체가 사용되는 곳에서 오류가 발생한다
        
        ```tsx
        const elmo = {
        name: 'Tickle Me Elmo',
        id : 1234, //원래 Product 타입은 id : string
        price : 28.99}
        
        logProduct(elmo);
        //...형식의 인수는 'Product' 형식의 매개변수에 할당될 수 없습니다.
        //'id'속성의 형식이 호환되지 않습니다.
        ```
        

## 아이템 20. 다른 타입에는 다른 변수 사용하기

- 자바스크립트에서는 한 변수를 다른목적을 가지는 다른 타입으로 재사용할 수 있다 → 동적언어 특징
    
    ```tsx
    let id = "12-34-56"
    id = 123456;
    ```
    
    - 그러나 타입스크립트는 불가능하다.
    - 타입 스크립트는 변수의 값을 바뀔수 있지만 그 타입은 보통 바뀌지않는다 라는 중요한 관점을 알 수있다.
        - 타입스크립트에서 위처럼 사용하기 위에선 string | number의 유니온 타입을 이용하면 되지만 매번 id를 사용할때마다 어떤 타입인지 확인해야 하기 때문에 다루기 어렵다.
        → 그래서 차라리 별도의 변수를 도입하는게 낫다.
        
        ```tsx
        let id: string|number = '12-34-56'
        id = 123456;
        
        -->
        const id = "12-34-56"
        const serial = 123456
        
        ```
        
    
    > 당연한 이야기가 많아 정리하지 않음
    > 
    

## 아이템 21. 타입 넓히기

- 런타임에 모든 변수는 유일한 값을 가지지만, 타입스크립트가 작성된 코드를 체크하는 정적 분석 시점에, 변수는 가능한 값들의 집한인 `타입` 을 가진다.
    - 따라서 상수를 사용해서 변수를 초기화 할때 타입이 명시되지않으면 타입체커는 지정된 단일 값을 가지고 할당 가능한 값들의 집합을 유추한다.
        - 이러한 과정을 넓히기라고 한다.
    
    ```tsx
    
    //런타임에서는 잘동작함
    interface Vector3 { x: number; y: number; z: number; }
    function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
      return vector[axis];
    }
    
    // 편집기에서는 오류 발생
    
    interface Vector3 { x: number; y: number; z: number; }
    function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
      return vector[axis];
    }
    let x = 'x';
    let vec = {x: 10, y: 20, z: 30};
    getComponent(vec, x);
                   // ~ Argument of type 'string' is not assignable to
                   //   parameter of type '"x" | "y" | "z"'
    ```
    
    - 위 의getComponent 함수는 x에 타입 넓히기 가 동작해서 ‘x’ 리터럴이 아닌 string 타입으로 추론되었다. 따라서 getComponet 함수에 x를 할당할 수 없는 버그가 발생했다.
    
- 타입스크립트 넓히기를 제어하는 방법
    1. const 사용하기
        1. let 대신 const를 사용하면 좁은타입이 되어 ‘x’ 리터럴로 인식한다
            1. 왜? x는 재 할당될 수 없기때문에 더 좁은 타입으로 추론 가능
        
        ```tsx
        const x = 'x'; // 타입이 'x'
        let vec = {x: 10, y: 20, z: 30};
        getComponent(vec, x); //정상
        ```
        
    2. 명시적 타입 제공하기
    
    ```tsx
    const v: {x: 1|3|5} = {
    x : 1,} //타입이 {x :1|3|5 }
    ```
    
    1. const 단언문 사용하기
        
        > 나는 되게 많이 이용하는 요소중 하나.
        > 
        
        ```tsx
        interface Vector3 { x: number; y: number; z: number; }
        function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
          return vector[axis];
        }
        const v1 = {
          x: 1,
          y: 2,
        };  // Type is { x: number; y: number; }
        
        const v2 = {
          x: 1 as const,
          y: 2,
        };  // Type is { x: 1; y: number; }
        
        const v3 = {
          x: 1,
          y: 2,
        } as const;  // Type is { readonly x: 1; readonly y: 2; }
        ```
        
        - 값 뒤에 as const 를 작성하면, 타입스크립트는 최대한 좁은 타입으로 추론한다.
            - 각 속성을 let으로 추론하지 않는 부분이 좋은것 같다.
            
        
        ```tsx
        const emailSubject = (
          orderNumber: string | null | undefined
        ): { [key in G.EmailTemplateType]: string } => {
          return {
            M_SUPLLEMENT_RETURN_REJECTED: `[웰코치샵] ${orderNumber} 주문 반품 요청이 거절됐습니다.`,
            M_EREQ_CREATE_REQUEST: `[웰코치샵] ${orderNumber} 주문 사전 준비사항 안내 드립니다.`,
            M_TEST_GUIDE: `[웰코치샵] ${orderNumber} 주문 검진 준비가 완료 됐습니다.`,
            M_REPORT_REGISTERED: `[웰코치샵] ${orderNumber} 주문 검진 결과가 도착했습니다.`,
            M_TEST_CANCEL_ORDER_REJECTED: `[웰코치샵] ${orderNumber} 주문 취소 요청이 거절됐습니다.`,
            M_FIND_ID: `[웰코치샵] 계정이 확인됐습니다.`,
            M_INQUIRY_ANSWER: `[웰코치샵] 1대1 문의주신 내용에 대한 답변입니다.`,
            M_DEFAULT: `[웰코치샵] 안녕하세요 웰코치 샵입니다.`,
            M_SUPLLEMENT_SHIPPING_FINISHED: `[웰코치샵]${orderNumber} 배송이 완료되었습니다.`,
          } as const
        }
        ```
        
        - 배열을 튜플타입으로 추론할때에도 as const를 사용할 수 있다.
            
            ```tsx
            interface Vector3 { x: number; y: number; z: number; }
            function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
              return vector[axis];
            }
            const a1 = [1, 2, 3];  // Type is number[]
            const a2 = [1, 2, 3] as const;  // Type is readonly [1, 2, 3]
            ```
            
    

## 아이템 22 . 타입 좁히기

> 요것도 많이씀
> 
- 타입 좁히기의 가장 일반적인 예시

```tsx
//null 체크
const el = document.getElementById('foo'); // Type is HTMLElement | null
if (el) {
  el // Type is HTMLElement
  el.innerHTML = 'Party Time'.blink();
} else {
  el // Type is null
  alert('No element #foo');
}

```

- 타입을 좁히는 다른방법 : 명시적 태그 붙이기

```tsx
interface UploadEvent { type: 'upload'; filename: string; contents: string }
interface DownloadEvent { type: 'download'; filename: string; }
type AppEvent = UploadEvent | DownloadEvent;

function handleEvent(e: AppEvent) {
  switch (e.type) {
    case 'download':
      e  // Type is DownloadEvent
      break;
    case 'upload':
      e;  // Type is UploadEvent
      break;
  }
}

```

- 이 패턴은 태그된 유니온 또는 구별된 유니온이라고 불린다.

- 타입을 좁힐때 타입을 섣불리 판단하는 실수를 하지 않도록 유의해야한다.
    - 자바스크립트에서는 typeof null 이 object이므로 좁히기가 안됨
    
    ```tsx
    const el = document.getElementById('foo'); // type is HTMLElement | null
    if (typeof el === 'object') {
      el;  // Type is HTMLElement | null
    }
    ```
    
    - string의 “”와 number의 0은 falsy 이므로 좁히기가 안됨
    
    ```tsx
    function foo(x?: number|string|null) {
      if (!x) {
        x;  // Type is string | number | null | undefined
      }
    }
    ```
    
- 타입스크립트가 타입을 식별하지 못한다면, 식별을 돕기위해 커스텀 함수를 도입할수도 있다.

```tsx
const jackson5 = ['Jackie', 'Tito', 'Jermaine', 'Marlon', 'Michael'];
const members = ['Janet', 'Michael'].map(
  who => jackson5.find(n => n === who)
).filter(who => who !== undefined);  // Type is (string | undefined)[]

const jackson5 = ['Jackie', 'Tito', 'Jermaine', 'Marlon', 'Michael'];
function isDefined<T>(x: T | undefined): x is T {
  return x !== undefined;
}
const members = ['Janet', 'Michael'].map(
  who => jackson5.find(n => n === who)
).filter(isDefined);  // Type is string[]
```

```tsx
export type Falsy = false | null | undefined | 0 | -0 | 0n | ''

export type Truthy<X> = Exclude<X, Falsy>

type T<X> = Truthy<X> | Falsy

export function isTruthy<X>(condition: X): condition is Truthy<X> {
  return !!condition
}

export function isFalsy<X>(condition: T<X>): condition is Falsy {
  return !condition
}

export type Result<T, E extends Error> = { v: T } | { e: E }
```

## 아이템 24 일관성 있는 별칭 사용하기

- 별칭을 남발해서 사용하면 제어 흐름을 분석하기 어렵다.

아래와 같이 다각형을 표현하는 타입을 보자

```tsx
interface Coordinate {
  x: number;
  y: number;
}

interface BoundingBox {
  x: [number, number];
  y: [number, number];
}

interface Polygon {
  exterior: Coordinate[];
  holes: Coordinate[][];
  bbox?: BoundingBox;
}
```

Polygon의 bbox변수는 optional이다.

- 아래와 같이 polygon.bbox를 별칭으로 재 할당해 사용했을경우, 편집기에서 오류가 발생하는 것을 볼 수 있다.
    
    ```tsx
    interface Coordinate {
      x: number;
      y: number;
    }
    
    interface BoundingBox {
      x: [number, number];
      y: [number, number];
    }
    
    interface Polygon {
      exterior: Coordinate[];
      holes: Coordinate[][];
      bbox?: BoundingBox;
    }
    function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
      const box = polygon.bbox;
      if (polygon.bbox) {
        if (pt.x < box.x[0] || pt.x > box.x[1] ||
            //     ~~~                ~~~  Object is possibly 'undefined'
            pt.y < box.y[1] || pt.y > box.y[1]) {
            //     ~~~                ~~~  Object is possibly 'undefined'
          return false;
        }
      }
      // ...
    }
    ```
    
    - 이는 별칭이 제어흐름 분석을 발생 했기 때문.
        - 이러한 오류는 `별칭은 일관성 있게 사용한다` 라는 기본 원칙을 지키면 방지할 수 있다.

- `별칭은 일관성 있게 사용한다` 를 적용한 예시
    - bbox의 타입 추론이 올바르게 적용(if문에서 undefined 제거)되었다.

```tsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const box = polygon.bbox;
  if (box) {
    if (pt.x < box.x[0] || pt.x > box.x[1] ||
        pt.y < box.y[1] || pt.y > box.y[1]) {  // OK
      return false;
    }
  }
  // ...
}
```

- 그러나 코드를 읽는 사람에게는 box와 bbox는 같은 값인데 이를 혼용하여 쓸 수 있는 문제가있다.
    - 따라서 객체 비구조화를 이용하여 간결한 문법으로 일관된 이름을 사용한다.
        
        ```tsx
        function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
          const {bbox} = polygon;
          if (bbox) {
            const {x, y} = bbox;
            if (pt.x < x[0] || pt.x > x[1] ||
                pt.y < x[0] || pt.y > y[1]) {
              return false;
            }
          }
          // ...
        }
        ```
        
        - 객체 비구조화를 이용할떄는 두가지를 주의해야 한다.
            - 위 코드의 bbox객체→x,y속성이 Optional이라면, 속성체크가 필요하게 된다, 따라서 타입의 경계에 null값을 추가하는것이 좋다.
            - 빈 배열은 ~가 없음을 나타내는 좋은 방법이다. 굳이 ~가 없음을 이름으로 구별할 필요가 없다.
                
                ```tsx
                const holes:number[] = [1,2,3]
                const notholes = [] //---X
                const holes = [] // --- holes가 없는것이나 마찬가지
                ```
                

---

## 아이템 25. 비동기 코드에는 콜백 대신 async함수 사용하기

- 이전 자바스크립트는 비동기 동작을 모델링 하기위해 콜백을 사용했고, 콜백지옥이 펼쳐졌었고, 이는 직관적으로 이해하기 어려웠다.
    
    ```tsx
    function fetchURL(url: string, cb: (response: string) => void) {
      cb(url);
    }
    const url1 = '1';
    const url2 = '2';
    const url3 = '3';
    // END
    fetchURL(url1, function(response1) {
      fetchURL(url2, function(response2) {
        fetchURL(url3, function(response3) {
          // ...
          console.log(1);
        });
        console.log(2);
      });
      console.log(3);
    });
    console.log(4);
    
    // Logs:
    // 4
    // 3
    // 2
    // 1
    ```
    
    - 그러다가 ES2015에서 Promise가 도입 되었고 이는 콜백지옥을 해결하고, 실행순서가 코드순서와 동일해져 직관적으로 이해하기 쉬워졌다.
        
        ```tsx
        const page1Promise = fetch(url1);
        page1Promise.then(response1 => {
          return fetch(url2);
        }).then(response2 => {
          return fetch(url3);
        }).then(response3 => {
          // ...
        }).catch(error => {
          // ...
        });
        ```
        
    - 나아가 ES2017에는 async/await가 도입되어 더 간단하게 처리가 되었다.
        
        ```tsx
        async function fetchPages() {
          const response1 = await fetch(url1);
          const response2 = await fetch(url2);
          const response3 = await fetch(url3);
          // ...
        }
        ```
        
- ES5 또는 더이전버전의 경우 타입스크립트 컴파일러는 async/await가 동작하도록 정교한 변환을 수행한다
    - 이는 타입스크립트는 런타임에 관계없이 async/await를 사용할 수 있다.

- 콜백보다는 Promise를 사용해야 하는 이유
    - Promise가 코드를 작성하기 쉽고, 타입을 추론하기 쉽다.
    
    ```tsx
    async function fetchPages() {
      const [response1, response2, response3] = await Promise.all([
        fetch(url1), fetch(url2), fetch(url3)
      ]);
      // ...
    }
    ```
    
    위는 Promise.all을 사용해서 병렬로 페이지를 로드하고, 구조분해할당을 한 예시이다.
    
    - 위 방식을 콜백으로 동일하게 적용하려면 매우 복잡해진다.

- 프로미스를 직접 생성해야할때, 선택의 여지가 있다면 일반적으로 Promise보다는 async/await함수를 생성하자
    - 일반적으로 더 간결하고 직관적인 코드가 된다.
    - async함수는 항상 프로미스를 반환하도록 강제된다.
    
    ```tsx
    // function getNumber(): **Promise<number>**
    async function getNumber() {
      return 42;
    }
    ```
    
- 떄에 따라 즉시 사용 가능한 값에도 프로미스를 반환하는것이 이상하게 보일 수 있지만, 실제로는 비동기 함수로 통일하도록 강제하는데 도움이 된다.
    - 함수는 항상 동기 또는 비동기로 실행되어야하고, 절대 혼용해서는 안된다.
    
    잘못된 예시
    
    ```tsx
    const _cache: {[url: string]: string} = {};
    function fetchWithCache(url: string, callback: (text: string) => void) {
      if (url in _cache) {
        callback(_cache[url]); //동기
      } else {
        fetchURL(url, text => { // 비동기
          _cache[url] = text;
          callback(text);
        });
      }
    }
    ```
    
    위를 asnyc함수를 이용하여 올바르게 표현한 예시
    
    async가 항상 promise를 래핑하여 무조건 비동기 코드를 작성하는것 처럼 된다.
    
    ```tsx
    const _cache: {[url: string]: string} = {};
    async function fetchWithCache(url: string) {
      if (url in _cache) {
        return _cache[url];
      }
      const response = await fetch(url);
      const text = await response.text();
      _cache[url] = text;
      return text;
    }
    
    let requestStatus: 'loading' | 'success' | 'error';
    async function getUser(userId: string) {
      requestStatus = 'loading';
      const profile = await fetchWithCache(`/user/${userId}`);
      requestStatus = 'success';
    }
    ```
    
    ---
    
    ## 아이템 26 타입 추론에 문맥이 어떻게 사용되는지 이해하기
    
- 타입스크립트는 타입을 추론할때 단순히 값만 고려하지않고, 값의 존재하는곳의 문맥까지 살핀다.
    - 그런데 가끔 문맥을 고려해 타입을 추론하면 이상한 결과가 나올때가 있다
        - 아래의 코드를 보면 같은 “JavaScript” 라는 문자열을 입력했음에도 language 변수는 string으로 추론되어 오류가 발생하는 것을 볼 수 있다.
    
    ```tsx
    type Language = 'JavaScript' | 'TypeScript' | 'Python';
    function setLanguage(language: Language) { /* ... */ }
    
    setLanguage('JavaScript');  // OK
    
    let language = 'JavaScript'; //type is string
    setLanguage(language);
             // ~~~~~~~~ Argument of type 'string' is not assignable
             //          to parameter of type 'Language'
    ```
    
    - 이를 해결하는 방법
        - language 변수에 Language 로 타입 선언해주기
            - 오타 같은 문제도 해결해 줄 수 있음
        
        ```tsx
        type Language = 'JavaScript' | 'TypeScript' | 'Python';
        function setLanguage(language: Language) { /* ... */ }
        let language: Language = 'JavaScript';
        setLanguage(language);  // OK
        ```
        
        - language 변수를 상수로 만들어 리터럴로 추론하기
        
        ```tsx
        type Language = 'JavaScript' | 'TypeScript' | 'Python';
        function setLanguage(language: Language) { /* ... */ }
        const language  = 'JavaScript';
        setLanguage(language);  // OK
        ```
        
        - 문제는 해결되었지만 Language 타입이라는**문맥으로부터 “**JavaScript” 스트링 리터럴 **값을 분리** 하였기 떄문에 몇가지 문제를 발생시킬수 있다.

- **문맥으로부터 값을 분리하면 생길 수 있는 문제점**
    
    
    - 튜플 사용시 문제점
        
        ```tsx
        
        function panTo(where: [number, number]) { /* ... */ }
        
        panTo([10, 20]);  // OK
        
        const loc = [10, 20];
        panTo(loc);
        //    ~~~ Argument of type 'number[]' is not assignable to
        //        parameter of type '[number, number]'
        ```
        
        - 두번째의 경우 loc가 number[]로 추론되었음
        
        **해결법**
        
        1. 타입 선언해주기
        
        ```tsx
        type Language = 'JavaScript' | 'TypeScript' | 'Python';
        function setLanguage(language: Language) { /* ... */ }
        // Parameter is a (latitude, longitude) pair.
        function panTo(where: [number, number]) { /* ... */ }
        const loc**: [number, number]** = [10, 20];
        panTo(loc);  // OK
        ```
        
        1. const 단언
            
            const 단언을 사용하게 될시 , 해당 함수의 인자는 readonly를 붙여줘야 한다.
            
        
        ```tsx
        type Language = 'JavaScript' | 'TypeScript' | 'Python';
        function setLanguage(language: Language) { /* ... */ }
        // Parameter is a (latitude, longitude) pair.
        
        function panTo(where: readonly [number, number]) { /* ... */ }
        const loc = [10, 20] as const;
        panTo(loc);  // OK
        ```
        
        const 단언은 문맥손실과 관련한 문제를 깔끔하게 해결할 수 있지만 단점이 있다.
        
        만약 타입정의에 실수가 있다면 오류는 타입정의가 아닌 호출되는 곳에서 발생한다.
        
        이는 근본적인 원인 파악에 문제가 될 수 있다.
        
        ```tsx
        function panTo(where: readonly [number, number]) { /* ... */ }
        const loc = [10, 20, 30] as const;  // error is really here.
        panTo(loc);
        //    ~~~ Argument of type 'readonly [10, 20, 30]' is not assignable to
        //        parameter of type 'readonly [number, number]'
        //          Types of property 'length' are incompatible
        //            Type '3' is not assignable to type '2'
        ```
        
- 객체 사용시 주의점
    
    
    ```tsx
    type Language = 'JavaScript' | 'TypeScript' | 'Python';
    interface GovernedLanguage {
      language: Language;
      organization: string;
    }
    
    function complain(language: GovernedLanguage) { /* ... */ }
    
    complain({ language: 'TypeScript', organization: 'Microsoft' });  // OK
    
    const ts = {
      language: 'TypeScript',
      organization: 'Microsoft',
    };
    complain(ts);
    //       ~~ Argument of type '{ language: string; organization: string; }'
    //            is not assignable to parameter of type 'GovernedLanguage'
    //          Types of property 'language' are incompatible
    //            Type 'string' is not assignable to type 'Language'
    ```
    
    **해결법**
    
    튜플과 같다.
    

---

## 아이템 27 함수형 기법과 라이브러리로 타입 흐름 유지하기

- 표준 라이브러리를 하는 수많은 자바스크립트 라이브러리 (ex 제이쿼리, 로대시)를 사용하지 않고 직접 관련 함수를 수현하면 , 타입체크에 대한 관리가 필요하다.
    - 그러나 이는 복잡함을 야기할 수 있다.
    
    ```tsx
    const csvData = "...";
    const rawRows = csvData.split('\n');
    const headers = rawRows[0].split(',');
    
    const rows = rawRows.slice(1).map(rowStr => {
      const row = {};
      rowStr.split(',').forEach((val, j) => {
        row[headers[j]] = val;
      });
      return row;
    });
    ```
    
    - 이를 로대시를 이용하여 쉽게 표현할 수 있다.
    
    ```tsx
    // requires node modules: @types/lodash
    const csvData = "...";
    const rawRows = csvData.split('\n');
    const headers = rawRows[0].split(',');
    
    import _ from 'lodash';
    const rows = rawRows.slice(1)
        .map(rowStr => _.zipObject(headers, rowStr.split(',')));
    ```
    
    - 서드파티 라이브러리를 사용하는데 시간이 오래 걸린다면 사용안하는게 나을 수 있지만 타입스크립트를 사용한다면 무조건 서드파티 라이브러리 기반으로 바꾸는것이 시간이 훨씬 단축된다.
    
    <aside>
    💡 잘 알려진 서드파티 유틸리티 라이브러리를 사용하는것이 중요하다는것을 보여주는 내용이 주됨
    
    </aside>
    

---