# 2. 타입스크립트의 타입 시스템

<aside>
💡 타입스크립트의 가장 중요한 역할은 타입시스템에 있다.

</aside>

## 아이템 7. 타입의 값들의 집합이라고 생각하기

- 타입은 할당가능한 값 들의 집합이다.
    - 집합의 관점에서, 타입 체커의 주요 역할은 하나의 집합이 다른 집합의 부분집합인지를 검사하는 것이라 볼 수 있다.
    
    > [여기](https://www.notion.so/1-765a59055609494aba47fe037eb3ab38) 에서 설명했던 구조적 타이핑 개념과 같은 맥락으로 집합에 포함 되느냐의 관건으로 타입 체킹을 한다 라고 이해했다.
    > 
    

---

## 아이템 8. 타입 공간과 값 공간의 심벌 구분하기

- 타입스크립트의 심벌은 타입 공간이나 값 공간 중의 한 곳에 존재한다.

> 심볼이 정확히 뭘까??
> 

- Class 와 Enum은 상황에 따라 타입과 값 두가지 모두 가능한 예약어이다.
    - 클래스가 타입으로 쓰일때는 형태(속성과 메소드)가 사용되는 반면,
    값으로 쓰일때에는 생성자가 사용된다.

- 연산자중에서도 타입에서 쓰일때와 값에서 쓰일때 다른 기능을 하는 것들이 있다
    
    ```tsx
    interface Person {
      first: string;
      last: string;
    }
    const p: Person = { first: 'Jane', last: 'Jacobs' };
    //    -           --------------------------------- Values
    //       ------ Type
    function email(p: Person, subject: string, body: string): Response {
      //     ----- -          -------          ----  Values
      //              ------           ------        ------   -------- Types
      // COMPRESS
      return new Response();
      // END
    }
    
    class Cylinder {
      radius=1;
      height=1;
    }
    
    function calculateVolume(shape: unknown) {
      if (shape instanceof Cylinder) {
        shape  // OK, type is Cylinder
        shape.radius  // OK, type is number
      }
    }
    type T1 = typeof p;  // Type is Person
    type T2 = typeof email;
        // Type is (p: Person, subject: string, body: string) => Response
    
    const v1 = typeof p;  // Value is "object"
    const v2 = typeof email;  // Value is "function"
    ```
    
    - 타입(type T1)의 관점에서 typeof는 값을 읽어 타입스크립트 타입을 반환한다.
    - 값의(const v1) 관점에서 typeof는 자바스크립트 런타임의 typeof 연산자가 되어 6개의 런타임 타입중 하나를 반환한다 (string, number, boolean, undefined, object, function)
    
    > [https://engineering.linecorp.com/ko/blog/typescript-enum-tree-shaking/](https://engineering.linecorp.com/ko/blog/typescript-enum-tree-shaking/)
    > 
    
    ```tsx
    const v = typeof Cylinder;
    type v = typeof Cylinder;
    ```
    
    - 클래스의 경우 값으로 이용했을때(자바스크립트 런타임 타입) 클래스가 자바스크립트에서는 실제로 함수로 구현되기 때문에 첫번째 기준의 값은 function이 된다.
    
- 타입 스크립트 코드가 잘 동작하지 않는다면, 타입 공간과 값 공간을 혼동해서 잘못 작성했을 가능성이 크다.
    
    ```tsx
    interface Person {
      first: string;
      last: string;
    }
    function email({
      person: Person,
           // ~~~~~~ Binding element 'Person' implicitly has an 'any' type
      subject: string,
            // ~~~~~~ Duplicate identifier 'string'
            //        Binding element 'string' implicitly has an 'any' type
      body: string}
         // ~~~~~~ Duplicate identifier 'string'
         //        Binding element 'string' implicitly has an 'any' type
    ) { /* ... */ }
    ```
    
    - 위는 값으로 해석될 부분에 타입이 들어가서 문제를 일으킨다.
    - 이를 해결하려면 아래처럼 값 관점과, 타입 관점을 나누어주어야 한다.
    
    ```tsx
    interface Person {
      first: string;
      last: string;
    }
    function email(
      {person, subject, body}: {person: Person, subject: string, body: string}
    ) {
      // ...
    }
    ```
    

<aside>
💡 타입스크립트 코드를 읽을때 , 타입인지 값인지 구분하는 방법을 터득해야 한다

</aside>

- 모든 값은 타입을 가지지만 타입은 값을 가지지 않는다.
    - type과 interface는 타입 공간에만 존재한다

---

## 아이템9. 타입 단언보다는 타입 선언을 사용하기

```tsx
interface Person { name: string };
const alice: Person = {}; //타입 선언
   // ~~~~~ Property 'name' is missing in type '{}'
   //       but required in type 'Person'
const bob = {} as Person;  // No error //타입 단언
```

- 위처럼, 타입 단언을 사용하게 될 경우, 해당 인터페이스를 만족하는지 타입스크립트가 컴파일 타임에서 잡지 못하기 때문에 되도록 이면 타입 선언을 쓰자

```tsx
interface Person { name: string };

--------- 타입 단언 ---------
const people = ['alice', 'bob', 'jan'].map(name => ({} as Person));
// No error

--------- 타입 선언 ---------
const people: Person[] = ['alice', 'bob', 'jan'].map(
  (name): Person => ({name})
);
```

- 위처럼 map 함수에 타입 단언으로 리턴하게 되면 런타임 버그가 발생할 수 있다.
- 따라서 map 함수의 콜백에 리턴형식을 지정함으로 타입선언을 통해 컴파일 타임에서 버그를 방지하도록 하자.

- 모든 타입은 unknown타입의 서브타입이다.
    
    ```tsx
    const el = document.body as unknown as Person
    ```
    
    - unknown 타입은 임의의 타입간에 변환을 가능케 하지만 , unknown을 사용하여 단언을 한다는 것 자체가 위험한 동작을 하고있다는것일수 있다.
        - document.body 가 Person 의 서브타입이 아닐 수 있기 때문 → 버그발생의 우려
    

---

## 아이템 10. 객체 래퍼타입 피하기

```tsx
'string'.charAt(3)
```

- 생각해보면 , 기본형(string)은 메소드를 가질수 없는데 charAt메소드가 어떻게 동작하는것일까?
    - 자바스크립트는 메서드를 가지는 String 객체 타입이 정의되어있다. 이때, 자바스크립트는 string 기본형을 String 객체로 자유롭게 변환하여 사용한다.
        - string을 String 객체로 래핑(wrapping) → 메서드 호출 → **래핑한 객체 버리기**

```tsx
x = 'hello'
x.language = 'English'
> 'English'
> x.language 
// x 라는 String 객체에 language속성으로 English가 추가된후 래핑한 객체가 버려졌기 때문에 undefind
undefind
```

---

## 아이템 11. 잉여 속성 체크의 한계 인지하기

- 타입이 명시된 변수에 객체 리터럴을 할당할때 타입스크립트는 해당 타입속성을 다 포함하는지와 `그 외에 속성은 없는지` 확인한다. 이를 `잉여 속성 체크`라 한다
    
    ```tsx
    interface Room {
      numDoors: number;
      ceilingHeightFt: number;
    }
    const r: Room = {
      numDoors: 1,
      ceilingHeightFt: 10,
      elephant: 'present',
    // ~~~~~~~~~~~~~~~~~~ Object literal may only specify known properties,
    //                    and 'elephant' does not exist in type 'Room'
    };
    ```
    
    위의 오류는 구조적 타이핑 관점([아이템 4](https://www.notion.so/1-765a59055609494aba47fe037eb3ab38))에서 보면 오류가 발생하지 않아야 된다.
    
    ```tsx
    interface Room {
      numDoors: number;
      ceilingHeightFt: number;
    }
    const obj = {
      numDoors: 1,
      ceilingHeightFt: 10,
      elephant: 'present',
    };
    const r: Room = obj;  // OK
    ```
    
    - 위 두 예제의 차이는,  첫번째 예제만 구조적 타입 시스템에서 발생할 수 있는 중요한 종류의 오류를 잡을 수 있도록 `잉여 속성 체크` 라는 과정이 수행되었다.
        - 잉여 속성 체크와 할당 가능검사(구조적 타이핑)는 **별도의 과정**이다
        
- 잉여 속성 체크를 이용하면 기본적으로 타입 시스템의 구조적 본질을 해치지 않으면서도 객체 리터럴에 알 수 없는속성(알려지지 않은 속성)을 허용하지 않을 수 있다.
    
    ```tsx
    interface Room {
      numDoors: number;
      ceilingHeightFt: number;
    }
    function setDarkMode() {}
    interface Options {
      title: string;
      darkMode?: boolean;
    }
    
    //구조적 할당의 문제점 --> Options의 범위가 너무 넓어져서 title속성만 있는 객체는 전부 Options로 할당 가능
    const o1: Options = document;  // OK
    const o2: Options = new HTMLAnchorElement;  // OK
    
    //잉여 속성 체크
    const o: Options = { darkMode: true, title: 'Ski Free' };
    ```
    
    - 그래서 `엄격한 객체 리터럴 체크` 라고도 불린다.
    - 잉여 속성 체크는 타입 단언문을 사용할때도 적용되지 않는다.
    
    ```tsx
    const o =  { darkMode: true, title: 'Ski Free' } as Options
    ```
    
    - 이는 타입 단언문을 사용하지 말아야 하는 단적인 이유이기도 하다.
    - 잉여 속성체크는 구조적할당에서 잡지 못하는 (오타) 부분도 확실히 검사할 수 있다.
    
    ```tsx
    interface LineChartOptions {
      logscale?: boolean;
      invertedYAxis?: boolean;
      areaChart?: boolean;
    }
    const o: LineChartOptions = { logScale: true };  
     // ~ Type '{ logScale: boolean; }' has no properties in common --> S가 소문자임
       //   with type 'LineChartOptions'
    ```
    
    - 잉여 속성체크는 객체 리터럴에만 적용된다 (변수 X)
    

---

## 아이템 12. 함수 표현식에 타입 적용하기

- 자바스크립트(타입스크립트)는 `함수 문장`과 `함수 표현식`을 다르게 인식한다.

```tsx
function rollDice1(sides: number): number { /* COMPRESS */ return 0; /* END */ }  // 함수 문장
const rollDice2 = function(sides: number): number { /* COMPRESS */ return 0; /* END */ };  //함수 표현식
const rollDice3 = (sides: number): number => { /* COMPRESS */ return 0; /* END */ };  // 함수 표현식
```

- 타입스크립트에서는 함수표현식을 사용하는게 좋다.
    - 함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 함수표현식에 재사용할 수 있다.
    
    ```tsx
    type DiceRollFn = (sides: number) => number;
    const rollDice: DiceRollFn = sides => { /* COMPRESS */ return 0; /* END */ };
    ```
    

- 함수 타입의 선언을 불필요한 코드의 반복을 줄인다.
    
    ```tsx
    // 함수 타입선언으로 함수 시그니처를 통일
    type BinaryFn = (a: number, b: number) => number; 
    
    const add: BinaryFn = (a, b) => a + b;
    const sub: BinaryFn = (a, b) => a - b;
    const mul: BinaryFn = (a, b) => a * b;
    const div: BinaryFn = (a, b) => a / b;
    ```
    

- 다른 함수의 시그니처로 동일한 타입을 가지는 새 함수를 작성할때도 유용하다
    
    ```tsx
    async function checkedFetch(input: RequestInfo, init?: RequestInit) {
      const response = await fetch(input, init);
      if (!response.ok) {
        // Converted to a rejected Promise in an async function
        throw new Error('Request failed: ' + response.status);
      }
      return response;
    }
    
    //---typeof fetch를 이용하여 RequestInfo , RequestInit를 fetch메소드에서 추론함
    const checkedFetch: typeof fetch = async (input, init) => {
      const response = await fetch(input, init);
      if (!response.ok) {
        throw new Error('Request failed: ' + response.status);
      }
      return response;
    }
    ```
    

<aside>
💡 typeof Fn을 잘 활용해봅시다.

</aside>

---

## 아이템 13. 타입과 인터페이스의 차이점 알기

- 타입스크립트에서 명명된 타입을 정의하는 방법엔 type , interface가 있다.
    
    ```tsx
    type TState = {
      name: string;
      capital: string;
    }
    interface IState {
      name: string;
      capital: string;
    }
    ```
    

### 타입과 인터페이스의 비슷한점

- 명명된 타입은 인터페이스든, 타입이든 상태에는 차이가 없다.
    - 동일 오류 발생
    
    ```tsx
    type TState = {
      name: string;
      capital: string;
    }
    interface IState {
      name: string;
      capital: string;
    }
    const wyoming: TState = {
      name: 'Wyoming',
      capital: 'Cheyenne',
      population: 500_000
    // ~~~~~~~~~~~~~~~~~~ Type ... is not assignable to type 'TState'
    //                    Object literal may only specify known properties, and
    //                    'population' does not exist in type 'TState'
    };
    ```
    
    - 인덱스 시그니처 사용가능
    
    ```tsx
    type TState = {
      name: string;
      capital: string;
    }
    interface IState {
      name: string;
      capital: string;
    }
    type TDict = { [key: string]: string };
    interface IDict {
      [key: string]: string;
    }
    ```
    
    - 함수 타입 정의가능
    
    ```tsx
    type TState = {
      name: string;
      capital: string;
    }
    interface IState {
      name: string;
      capital: string;
    }
    type TFn = (x: number) => string;
    interface IFn {
      (x: number): string;
    }
    
    const toStrT: TFn = x => '' + x;  // OK
    const toStrI: IFn = x => '' + x;  // OK
    ```
    
    - 제너릭 쌉가능
    
    ```tsx
    type TState = {
      name: string;
      capital: string;
    }
    interface IState {
      name: string;
      capital: string;
    }
    type TPair<T> = {
      first: T;
      second: T;
    }
    interface IPair<T> {
      first: T;
      second: T;
    }
    ```
    
    등등.. 클래스에서도 동일하게 implements 할 수 있음
    

### 타입과 인터페이스의 차이점

- 인터페이스는 유니온 개념이 없다.
    - 아래 타입 표현식은 인터페이스로 표현 불가능하다.
    
    ```tsx
    type NamedVariable = (InPut | OutPut) & { name : string }
    ```
    
- 튜플은 type키워드로 구현하는게 낫다.
    - 인터페이스로 튜플과 비슷하게 타입을 구현하면 튜플에서 사용할 수 있는 concat메서드를 사용할 수 없다.
- 보통 타입이 인터페이스보다 쓰임새가 많다고 한다..

그러나 인터페이스만의 기능이 있다! 바로 `선언 병합` :: 보강 기법

- 아래처럼 같은 이름의 인터페이스를 통해 타입을 병합할 수 있다.

```tsx
interface IState {
  name: string;
  capital: string;
}
interface IState {
  population: number;
}
const wyoming: IState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
  population: 500_000
};  // OK
```

따라서…

1. 복잡한 타입은 고민할것도 없이 Type을 사용한다.
2. 해당 코드베이스에 일관된 형식을 따라간다.
    1. interface를 주로 쓰는 베이스면 interface만
    2. type을 주로 쓰는 베이스면 type만
3. 클라이언트를 위한 API는 인터페이스의 병합 기능을 이용하자
    1. 버전이 바뀌어 필드가 추가되어야할때 병합기능을 이용하면 버전관리 용이!

---

## 아이템 14. 타입 연산과 제너릭 사용으로 반복 줄이기

- 같은 코드를 반복하지 말라 DRY (don’t repeat yourself) 원칙
    - 타입중복은 코드 중복만큼 많은 문제를 일으킨다.
    
    ```tsx
    interface Person {
      firstName: string;
      lastName: string;
    }
    
    interface PersonWithBirthDate {
      firstName: string;
      lastName: string;
      birth: Date;
    }
    ```
    
    - 여기서 Person에 middleName이라는 필드가 추가된다면 , Person과 PersonWithBirthDate는 목적과 다른 타입이 될 수 있다.
    - 아래와 같이 extends를 이용해서 중복을 피하자.
    
    ```tsx
    interface Person {
      firstName: string;
      lastName: string;
    }
    
    interface PersonWithBirthDate extends Person {
      birth: Date;
    }
    ```
    

### 제너릭 타입

### PICK

```tsx
interface SaveAction {
  type: 'save';
  // ...
}
interface LoadAction {
  type: 'load';
  // ...
}
type Action = SaveAction | LoadAction;
type ActionType = 'save' | 'load';  // Repeated types!
//---
type ActionType = Action['type'];  // Type is "save" | "load"
//---
type ActionRec = Pick<Action, 'type'>;  // {type: "save" | "load"}
```

### Partial

```tsx
interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}
type OptionsUpdate = {[k in keyof Options]?: Options[k]};

class UIWidget {
  constructor(init: Options) { /* ... */ }
  update(options: Partial<Options>) { /* ... */ }
```

### ReturnType

```tsx
const INIT_OPTIONS = {
  width: 640,
  height: 480,
  color: '#00FF00',
  label: 'VGA',
};
function getUserInfo(userId: string) {
  // COMPRESS
  const name = 'Bob';
  const age = 12;
  const height = 48;
  const weight = 70;
  const favoriteColor = 'blue';
  // END
  return {
    userId,
    name,
    age,
    height,
    weight,
    favoriteColor,
  };
}
// Return type inferred as { userId: string; name: string; age: number, ... }

//-->
type UserInfo = ReturnType<typeof getUserInfo>;
```

- 제너릭 타입은 타입을 위한 함수와 같다.
    - 어찌보면 DRY원칙의 핵심은 제너릭 이라는 것은 당연하다
    - 그런데 제너릭 타입에서 매개변수를 제한하는 방법이 필요하다
        - extends를 사용하자
        
        ```tsx
        interface Name {
          first: string;
          last: string;
        }
        type DancingDuo<T extends Name> = [T, T];
        const dancingDuo = <T extends Name>(x: DancingDuo<T>) => x;
        const couple1 = dancingDuo([
          {first: 'Fred', last: 'Astaire'},
          {first: 'Ginger', last: 'Rogers'}
        ]);
        const couple2 = dancingDuo([
          {first: 'Bono'},
        // ~~~~~~~~~~~~~~
          {first: 'Prince'}
        // ~~~~~~~~~~~~~~~~
        //     Property 'last' is missing in type
        //     '{ first: string; }' but required in type 'Name'
        ]);
        Footer
        ```
        
    
    ---
    
    ## 아이템 15. 동적 데이터에 인덱스 시그니처 사용하기
    
- 자바스크립트의 장점중하나는 객체를 생성하는 문법이 간단하다.

- 인덱스 시그니쳐
    
    ```tsx
    type Rocket = {[property : string] : string}; //<-- 인덱스 시그니처
    
    const rocket : Rocket = {
    	name : 'eunyeolRocket',
    	variant : 'v1.0'
    	thrust : '4,940 kN'
    }
    ```
    
    - 키의 이름 (property): 키의 위치만 표시하는 용도 , 타입 체커에선 사용하지 않음
    - 키의 타입 (string) : string 이나 number 또는 symbol중 하나여야만함. 보통 string을 사용
    - 값의 타입 (string) : 어떤 타입이든 가능

- 인덱스 시그니쳐의 단점
    - 모든 키를 허용한다
        - 위 예제에서 name에 Name이 되어도 오류가 발생하지 않는다.
    - 특정 키가 필요하지 않다
        - { } 도 허용된다.
    - 키마다 다른 타입을 가질 수 없다.
        - 위 상태에서 키값으로 number를 넣을 수 없다.

> 위 단점을 고려했을때는 사실 Rocket은 interface나 type으로 구현되어야 한다.
> 

```tsx
interface Rocket {
  name: string;
  variant: string;
  thrust_kN: number;
}
const falconHeavy: Rocket = {
  name: 'Falcon Heavy',
  variant: 'v1',
  thrust_kN: 15_200
};
```

### 그렇다면 인덱스 시그니쳐는 언제 쓰이는가?

- 인덱스 시그니처는 동적 데이터를 표현할때 사용한다.
    - csv파일처럼 열 이름이 무엇인지 모르고 동적으로 받아올때
        - 열 이름을 알면 인덱스 시그니처를 사용할 필요가 없다.
    
    ```tsx
    function parseCSV(input: string): {[columnName: string]: string}[] {
      const lines = input.split('\n');
      const [header, ...rows] = lines;
      return rows.map(rowStr => {
        const row: {[columnName: string]: string} = {};
        rowStr.split(',').forEach((cell, i) => {
          row[header[i]] = cell;
        });
        return row;
      });
    }
    ```
    
- 가능한 필드가 제한되어있는 경우라면 인덱스 시그니처를 제한해야 한다.
    
    ```tsx
    declare let csvData: string;
    const products = parseCSV(csvData) as unknown as ProductRow[];
    function safeParseCSV(
      input: string
    ): {[columnName: string]: string | undefined}[] {
      return parseCSV(input);
    }
     // 너무 범위가 넓다
    interface Row1 { [column: string]: number } 
    // Better
    interface Row2 { a: number; b?: number; c?: number; d?: number }  
    type Row3 =
    //정확한 타입이지만 너무 길다
        | { a: number; }
        | { a: number; b: number; }
        | { a: number; b: number; c: number;  }
        | { a: number; b: number; c: number; d: number }; 
    ```
    
    - 이럴때 해결법
        - Record를 사용한다.
        
        ```tsx
        declare let csvData: string;
        const products = parseCSV(csvData) as unknown as ProductRow[];
        function safeParseCSV(
          input: string
        ): {[columnName: string]: string | undefined}[] {
          return parseCSV(input);
        }
        type Vec3D = Record<'x' | 'y' | 'z', number>;
        // Type Vec3D = {
        //   x: number;
        //   y: number;
        //   z: number;
        // }
        Footer
        ```
        
        - 매핑된 타입을 사용한다.
        
        ```tsx
        declare let csvData: string;
        const products = parseCSV(csvData) as unknown as ProductRow[];
        function safeParseCSV(
          input: string
        ): {[columnName: string]: string | undefined}[] {
          return parseCSV(input);
        }
        type Vec3D = {[k in 'x' | 'y' | 'z']: number};
        // Same as above
        type ABC = {[k in 'a' | 'b' | 'c']: k extends 'b' ? string : number};
        // Type ABC = {
        //   a: number;
        //   b: string;
        //   c: number;
        // }
        ```
        

---

## 아이템 16. number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기

- 자바스크립트에서 객체란 키/값 쌍의 모음이다.
    
    파이썬이나 자바와 다르게 자바스크립트는 **해시가능 객체 라는 표현 이 없다.**
    
    따라서 객체를 키를 사용하려고 하면, 객체.toStirng() 을통해 키로 표현된다.
    
    ```yaml
    x = {}
    {}
    
    x[[1,2,3]] = 2
    2
    
    x
    {'1,2,3' : 1}
    ```
    
    리스트 또한 객체이므로 숫자 인덱스도 문자열로 변환되어 사용된다.
    
    ```tsx
    
    x = [1, 2, 3[
    Object.keys(x)
    
    ['0','1','2']
    ```
    

- 타입 스크립트는 이런 혼란을 바로잡기 위해 숫자키를 허용하고 , 문자열 키와 다른것으로 인식한다.
    
    ```tsx
    interface Array<T> {
    // ...
    [n : number] :T
    }
    ```
    
    이는 결국 런타임에는 결국 문자열 키로 인식 하므로 완전히 가상이라고 할 수 있지만, 타입 체크시점에 오류를 잡을 수 있어 유용하다.
    
    > 완전 이해 불가..
    > 
- 인덱스 시그니처에 number를 사용하기보다 Array나 튜플, 또는 ArrayLike타입을 사용하는게 좋다.
    - 인덱스 시그니처가 nymber로 표현되어 있다면 입력한 값이 number여야 한다는 것을 의미하지만 실제 런타입에 사용되는 string 타입이다.
    - 그래도 만약 숫자를 사용하여 인덱스할 항목을 지정한다면 Array또는 튜플 타입을 대신 사용하게 될거다.

---

## 아이템 17. 변경 관련된 오류 방지를 위해 readonly 사용하기

- readonly [] 특징
    - 요소를 읽을수는 있지만, 쓸수는 없다.
    - length를 읽을 수 있지만, 바꿀수(배열 변경)는 없다.
    - 배열을 변경하는 메소드를 사용할 수 없다.

- 매개변수를 readonly로 선언하면 다음과 같은 일이 생긴다
    - 타입스크립트는 매개변수가 함수 내에서 변경이 일어나는지 체크한다.
    - 호출하는 쪽에서는 함수가 매개변수를 변경하지 않는다는 보장을 받게 된다.
    - 호출하는 쪽에서 함수에 readonly 배열을 매개변수로 넣을수도 있다.

- 자바스크립트(타입스크립트)에서는 명시적으로 언급하지 않는 한, 함수가 매개변수를 변경하지 않는다고 가정한다.
    - 그러나, 이런 암묵적인 방법은 타입체크에 문제를 일으킬 수 있다. → 변경을 해버리면 암묵적인 룰이 깨지니까
        - 그래서 명시적으로 readonly를 붙여주는 게 컴파일러 , 사람 모두에게 좋다.
        - readonly를 사용하면, 지역변수와 관련된 모든 종류의 변경오류를 방지할 수 있다
        

 

- readonly를 사용하면 의도치 않은 변경을 방지할 수 있다.
- const와 readonly의 차이를 이해해야 한다.
    
    ### Const
    
    - 변수 참조를 위한 것이다.
    - 변수에 다른 값을 할당할 수 없다.
    
    ```
    const ex = '123';
    ex = '456' // ⛔️ 변경 불가
    ```
    
    ### readonly
    
    - 속성을 위한 것이다.
    
    ```
    type readonlyA = {
      readonly barA: string
    };
    
    const x: readonlyA = {barA: 'baz'};
    x.barA = 'quux'; // ⛔️ 변경 불가
    ```
    
- readonly는 얕게 동작한다.
    - 위 예제처럼 속성 `barA`는 변경이 불가능하지만 아래 예제처럼`barB`에 할당된 object값은 변경이 가능하다.
    
    ```
    type readonlyB = {
      readonly barB: { baz: string }
    };
    
    const y: readonlyB = {barB: {baz: 'quux'}};
    y.barB.baz = 'zebranky'; // 👌 변경 가능
    ```
    

---

## 아이템 18. 매핑된 타입을 사용하여 값을 동기화하기