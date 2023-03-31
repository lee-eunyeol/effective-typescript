# 4. 타입 설계

## 아이템 28. 유효한 상태만 표현하는 타입을 지향하기

- 타입을 잘 설계 하면 코드는 직관적으로 작성할 수 있다.
    - 효과적으로 타입을 설계하려면 유효한 상태만 표현할 수 있는 타입을 만들어내는 것이 중요하다!
        
        > 유효한 상태??
        > 
    
    ```tsx
    interface State {
      pageText: string;
      isLoading: boolean;
      error?: string;
    }
    declare let currentPage: string;
    function renderPage(state: State) {
      if (state.error) {
        return `Error! Unable to load ${currentPage}: ${state.error}`;
      } else if (state.isLoading) {
        return `Loading ${currentPage}...`;
      }
      return `<h1>${currentPage}</h1>\n${state.pageText}`;
    }
    ```
    
    위 코드를 보면 renderPage 함수에서 `State` 의 상태에 따라 error이냐 , isLoading이 true이냐에 따라 분기처리 하는 것을 볼 수 있다. 이는 분기조건이 명확히 분리되어 있지 않은것이다.
    
    - 만약 isLoading이 true이고, 동시에 error가 존재한다면 해당 분기를 명확하게 구분할 수가 없다.
        - 즉 State타입은 isLoading이 true 이면서, 동시에 error값이 설정되어 있는 **무효한 상태**
            
            를 허용하는것이다. 무효한 상태가 존재하면 함수를 제대로 구현할 수 없게 된다.
            
    
    **방법!**
    
    ```tsx
    interface RequestPending {
      state: 'pending';
    }
    interface RequestError {
      state: 'error';
      error: string;
    }
    interface RequestSuccess {
      state: 'ok';
      pageText: string;
    }
    type RequestState = RequestPending | RequestError | RequestSuccess;
    
    interface State {
      currentPage: string;
      requests: {[page: string]: RequestState};
    }
    function getUrlForPage(p: string) { return ''; }
    function renderPage(state: State) {
      const {currentPage} = state;
      const requestState = state.requests[currentPage];
      switch (requestState.state) {
        case 'pending':
          return `Loading ${currentPage}...`;
        case 'error':
          return `Error! Unable to load ${currentPage}: ${requestState.error}`;
        case 'ok':
          return `<h1>${currentPage}</h1>\n${requestState.pageText}`;
      }
    }
    ```
    
    위 코드에서는 태그된 유니온이 사용되었는데 State 타입이 명시적으로 모델링 되어 함수를 구현할때 훨씬 쉽게 구현이 될 수 있다.
    
    - 현재 페이지가 무엇인지 명확하며, 모든 요청은 정확이 하나의 상태로 맞아 떨어진다.
        
        > 이게 유효한 상태!
        무효한 상태는 예상할수 없는 분기처리가 생기는 로직을 뜻 한다고 생각
        > 
        
    
    <aside>
    💡 유효한 상태만 허용하는것은 매우 일반적인 원칙이다.
    코드가 길어지더라도 유효한 상태만 표현하는 타입을 지향하자!
    
    </aside>
    
    ---
    

## 아이템 29 사용할 때는 너그럽게,생성할때는 엄격하게

> 멋있는말
> 

당신의 작업은 엄격하게 하고 , 다른사람의 작업은 너그럽게 받아들여야 한다.

함수 시그니처도 비슷하게, 함수의 매개변수는 타입의 범위가 넓어도 되지만 결과를 반환할때는 타입의 범위가 구체적이여야 한다.

- 아래 코드에서  setCamera(카메라 위치 설정) viewportForBounds(경계박스의 뷰 포트를 계산)하는 함수가 있다.
    - viewportForBounds의 리턴값이 setCamera의 매개변수로 바로 들어 갈 수 있다면 편리하게 사용할 수 있을것 같아 아래와 같이 `CameraOptions` 를 리턴할 수 있도록 구현되었다.
    - 또한, CameraOptions 타입과 LngLat 타입은 매개변수의 범위를 넓혀 다양하게 쓰일 수 있도록 설계되었다.
        - ex) { lng: number; lat: number; } 뿐만 아니라 LngLat 의 형태와 비슷한 [number, number] 의 형태로도 데이터를 받아 비슷한 형태로도 매개변수가 들어 갈 수 있도록 되었다
            
            (주로 라이브러리의 타입설계에서도 비슷한 느낌으로 설계)
            

```tsx
interface CameraOptions {
  center?: LngLat;
  zoom?: number;
  bearing?: number;
  pitch?: number;
}
type LngLat =
  { lng: number; lat: number; } |
  { lon: number; lat: number; } |
  [number, number];
type LngLatBounds =
  {northeast: LngLat, southwest: LngLat} |
  [LngLat, LngLat] |
  [number, number, number, number];
declare function setCamera(camera: CameraOptions): void;
declare function viewportForBounds(bounds: LngLatBounds): CameraOptions;

```

### 문제점

- 아래 코드를 보면 viewportForBounds 의 리턴 타입인 camera : `CameraOptions` 가 범위가 너무 넓혀져 `CameraOptions` 의 center :LngLat 객체를 비구조화 할당 할때, 정확한 { lng: number; lat: number; } 을 추론할 수 없게 되었다.
    - 즉 리턴타입의 범위가 구체적이지 않게 되었다.

```tsx
type Feature = any;
declare function calculateBoundingBox(f: Feature): [number, number, number, number];
function focusOnFeature(f: Feature) {
  const bounds = calculateBoundingBox(f);
  const camera = viewportForBounds(bounds);
  setCamera(camera);
  const {center: {lat, lng}, zoom} = camera;
               // ~~~      Property 'lat' does not exist on type ...
               //      ~~~ Property 'lng' does not exist on type ...
  zoom;  // Type is number | undefined
  window.location.search = `?v=@${lat},${lng}z${zoom}`;
}
```

### 해결

- `CameraOptions` 와 `Camera` 로 타입을 분리하고,  매개변수로는`CameraOptions`  , 리턴값으로는 `Camera` 를 반환하여 리턴타입의 범위를 구체적으로 표시하여 에러를 해결하였다.

```tsx
interface LngLat { lng: number; lat: number; };
type LngLatLike = LngLat | { lon: number; lat: number; } | [number, number];

interface Camera {
  center: LngLat;
  zoom: number;
  bearing: number;
  pitch: number;
}
interface CameraOptions extends Omit<Partial<Camera>, 'center'> {
  center?: LngLatLike;
}
type LngLatBounds =
  {northeast: LngLatLike, southwest: LngLatLike} |
  [LngLatLike, LngLatLike] |
  [number, number, number, number];

declare function setCamera(camera: CameraOptions): void;
declare function viewportForBounds(bounds: LngLatBounds): Camera;

declare function setCamera(camera: CameraOptions): void;
declare function viewportForBounds(bounds: LngLatBounds): Camera;
function focusOnFeature(f: Feature) {
  const bounds = calculateBoundingBox(f);
  const camera = viewportForBounds(bounds);
  setCamera(camera);
  const {center: {lat, lng}, zoom} = camera;  // OK
  zoom;  // Type is number
  window.location.search = `?v=@${lat},${lng}z${zoom}`;
}
```

- 따라서 camera 객체는 `Camera` 타입을 반환받아 정확한 비구조화 할당이 가능하게 되었다.

<aside>
💡 1. 선택적속성 (?) 과 유니온타입 LngLat | { lon: number; lat: number; } … 은 매개변수 타입에 일반적이다.
2. 매개변수와 반환타입의 재사용을 위해 기본형태(반환 타입 [LngLat])와 느슨한 형태(매개변수[LngLatLike])를 도입하자!

</aside>

## 아이템 30 문서에 타입 정보 쓰지 않기

<aside>
💡 주석과 변수명에 타입정보를 적어 타입정보에 모순이 생기지 않게 하자 → 타입만으로도 충분이 타입정보에 대한 주석을 대체할 수 있고 변경시 정보가 정확하게 동기화 될수 있다.
대신 , 타입이 명확하지 않은 경우 변수에 단위정보를 포함하자 time → timeMs

</aside>

## 아이템 31 타입 주변에 null 값 배치하기

- 아래 코드에는 설계적 버그와 결함이 있다.
    - 최솟값이나 최대값이 0인경우 값이 덧씌워져 버린다.
        - [0,1,2] 를 매개변수로 넣을때 [0,2] 가 아닌 [1,2]가 반환된다.
    - nums 배열이 비어있다면, [undefined, undefined] 가 반환된다.

```tsx

function extent(nums: number[]) {
  let min, max;
  for (const num of nums) {
    if (!min) {
      min = num;
      max = num;
    } else {
      min = Math.min(min, num);
      max = Math.max(max, num);
    }
  }
  return [min, max];
}
```

### 해결

- min , max를 포함한 단일객체를 사용하고, null을 할당한다.

```tsx
function extent(nums: number[]) {
  let result: [number, number] | null = null;
  for (const num of nums) {
    if (!result) {
      result = [num, num];
    } else {
      result = [Math.min(num, result[0]), Math.max(num, result[1])];
    }
  }
  return result;
	}
```

- 반환 타입이 [number,number]  | null 이 되어 사용하기가 더 수월해졌고, ( ! ) 를 이용하여 쉽게 null을 걸러내어 범위를 구할 수 있게 되었다.
    
    ```tsx
    const range = extent([0, 1, 2]);
    if (range) {
      const [min, max] = range;
      const span = max - min;  // OK
    }
    ```
    
- 클래스를 만들때에도 모든값이 준비되었을때 생성하여 null 이 존재하지 않도록 하는것이 좋다.
    - 아래 코드의 UserPosts의 프로퍼티들은 null 가능성이 있어 버그가 발생할 확률이 높다.

```tsx
interface UserInfo { name: string }
interface Post { post: string }
declare function fetchUser(userId: string): Promise<UserInfo>;
declare function fetchPostsForUser(userId: string): Promise<Post[]>;
class UserPosts {
  user: UserInfo | null;
  posts: Post[] | null;

  constructor() {
    this.user = null;
    this.posts = null;
  }

  async init(userId: string) {
    return Promise.all([
      async () => this.user = await fetchUser(userId),
      async () => this.posts = await fetchPostsForUser(userId)
    ]);
  }

  getUserName() {
    // ...?
  }
}
```

- 아래처럼 프로퍼티가 모두 생성되었을 경우, 객체를 생성하도록 하여 null이 없도록 하여 버그가능성을 줄여야한다.

```tsx
interface UserInfo { name: string }
interface Post { post: string }
declare function fetchUser(userId: string): Promise<UserInfo>;
declare function fetchPostsForUser(userId: string): Promise<Post[]>;
class UserPosts {
  user: UserInfo;
  posts: Post[];

  constructor(user: UserInfo, posts: Post[]) {
    this.user = user;
    this.posts = posts;
  }

  static async init(userId: string): Promise<UserPosts> {
    const [user, posts] = await Promise.all([
      fetchUser(userId),
      fetchPostsForUser(userId)
    ]);
    return new UserPosts(user, posts);
  }

  getUserName() {
    return this.user.name;
  }
}
```

---

## 아이템 32 유니온의 인터페이스 보다는 인터페이스의 유니온을 사용하기

- 유니온의 인터페이스
    - Layer에서 layout이 FillLayout타입이고 , paint가 LinePaint 인 Layer가 존재한다는것은 의도상 맞지 않는다.

```tsx
interface Layer {
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}
```

- 인터페이스의 유니온
    - 아래형태로 레이어를 구분하여 layer와 paint 속성이 잘못된 조합으로 섞이는 경우를 방지할 수 있다.

```tsx
interface FillLayer {
  layout: FillLayout;
  paint: FillPaint;
}
interface LineLayer {
  layout: LineLayout;
  paint: LinePaint;
}
interface PointLayer {
  layout: PointLayout;
  paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;
```

- 위와 같이 구성하게되면 인터페이스에 태그된 유니온을 사용하여, 런타임에 어떤타입의 Layer가 사용되는지 판단하는데 쓰이게 할 수 있다.
    - 태그된 유니온은 타입스크립트 타입체커와 잘 맞기 때문에 이 패턴을 잘 기억해서 필요할때 적용할 수 있도록 하자.
    
    ```tsx
    interface FillLayer {
      type: 'fill';
      layout: FillLayout;
      paint: FillPaint;
    }
    interface LineLayer {
      type: 'line';
      layout: LineLayout;
      paint: LinePaint;
    }
    interface PointLayer {
      type: 'paint';
      layout: PointLayout;
      paint: PointPaint;
    }
    type Layer = FillLayer | LineLayer | PointLayer;
    ```
    

<aside>
💡 유니온의 인터페이스보다 인터페이스의 유니온이 더 정확하다!

</aside>

---

## 아이템 33 string 타입보다 더 구체적인 타입 사용하기

- string 타입은 범위가 매우 넓다. 그러므로, 혹시 string으로 변수를 선언하려 한다면, 그보다 더 좁은 타입은 없는지 고려해보자
    - 아래와 같이 string 타입을 남발하면 recordingType 프로퍼티에 ‘Studio’로 오기가 되어도 오류를 발생시키지 않아 문제를 일으킬 수 있다.
    
    ```tsx
    interface Album {
      artist: string;
      title: string;
      releaseDate: string;  // YYYY-MM-DD
      recordingType: string;  // E.g., "live" or "studio"
    }
    const kindOfBlue: Album = {
      artist: 'Miles Davis',
      title: 'Kind of Blue',
      releaseDate: 'August 17th, 1959',  // Oops!
      recordingType: 'Studio',  // Oops!
    };  // OK
    ```
    
- 타입 유니온을 쓰자
    
    ```tsx
    type RecordingType = 'studio' | 'live';
    
    interface Album {
      artist: string;
      title: string;
      releaseDate: Date;
      recordingType: RecordingType;
    }
    const kindOfBlue: Album = {
      artist: 'Miles Davis',
      title: 'Kind of Blue',
      releaseDate: new Date('1959-08-17'),
      recordingType: 'Studio'
    // ~~~~~~~~~~~~ Type '"Studio"' is not assignable to type 'RecordingType'
    };
    ```
    
    > 솔직히 당연한 소리같음
    > 

- string 은 any와 비슷한 문제 (너무 범위가 넓다)를 갖고 있어 잘못 사용하게 되면 무효한 값(예상치 못한 분기)을 허용하고, 타입간의 관계도 감추어버린다.
    - 이는 타입체커를 방해하고 실제 버그를 찾지 못하게 만든다
- 객체의 속성 이름을 함수 매개변수로 받을 때는 string 보다 key of T를 사용하자.

---

## 아이템 34. 부정확한 타입보다는 미완성 타입을 사용하기

- 타입선언을 작성하다 보면 코드의 동작을 더 구체적으로 또는 덜 구체적으로 모델링하게 되는 상황을 맞닦뜨린다.
    - 그러나, 타입선언의 정밀도를 높이는 일에는 주의를 기울여야한다.
        - 실수가 발생하기 쉽고 잘못된 타입은 차라리 타입이 없는것보다 못할 수 있다.

> 예제 예시의 경우, 부정확한 타입 설계를 했을때의 문제점을 주로 설명하였다. 따로 적지 않음.
> 

[effective-typescript/samples/ch04-design/item-34-incomplete-over-innacurate at master · danvk/effective-typescript](https://github.com/danvk/effective-typescript/tree/master/samples/ch04-design/item-34-incomplete-over-innacurate)

<aside>
💡 어설프게 타입의 완벽을 추구하려가 오히려 역효과가 발생하는것을 주의하자
정확하게 타입을 모델링 할 수 없다면, 부정확하게 모델링 하지 말자.

</aside>

---

## 아이템 35 데이터가 아닌, API와 명세를 보고 타입 만들기

- 명세를 참고해 타입을 생성하면 타입스크립트는 사용자가 실수를 줄일 수 있게 도와준다.
    - 반면에 예시 데이터를 참고하여 타입을 생성하게 되면 눈앞에 있는 데이터들만 고려하게 되므로 예기치 않은 곳에서 오류가 발생할 수 있다.
    

> 아이템 34와같이 Geojson 을 활용할때 명세 ( @types/geojson )와 graphql Apollo를 통해 타입을 구성하고 코드에 활용하면 보다 정확하게 작동할 수 있다는 확신을 가질 수 있다는 예제가 있었고, 정리에 불필요하다고 생각
> 

<aside>
💡 코드의 구석 구석 까지 타입 안정성을 얻기 위해 API 또는 데이터 형식에 대한 타입생성을 고려하게된다. 이때, 데이터에 드러나지 않는 예외적인 경우들이 문제가 될 수 있기 때문에 데이터보다는 명세를 기준으로 코드를 생성하자

</aside>

---

## 아이템 36 해당 분야의 용어로 타입 이름 짓기

- 타입의 이름짓기 또한 타입 설계에서 중요한 부분이다.
- 엄선된 타입, 속성, 변수의 이름은 의도를 명확히 하고 코드와 타입의 수준을 높여준다.
    
    ```tsx
    interface Animal {
      name: string;
      endangered: boolean;
      habitat: string;
    }
    
    const leopard: Animal = {
      name: 'Snow Leopard',
      endangered: false,
      habitat: 'tundra',
    };
    ```
    
    - 문제
        - name은 너무 일반적이다, 동물의 학명인지 일반적인 명칭인지 알 수 없다.
        - endangered 속성이 멸종위기를 표현하기 위해  boolean 타입을 사용한것이 이상하다. 이미 멸종된 동물을 true로 해야하는지 애매해진다.
        - habitat (서식지) 속성이 string타입이여 너무 범위가 넓고, 서식지라는 뜻 자체도 불분명하다.
        - 객체의 변수명은 leopard 지만 name 은 Snow Leopard 이다. name과 객체가 다른의도로 쓰였는지 불분명하다.
    
    - 해결
    
    ```tsx
    interface Animal {
      commonName: string;
      genus: string;
      species: string;
      status: ConservationStatus;
      climates: KoppenClimate[];
    }
    type ConservationStatus = 'EX' | 'EW' | 'CR' | 'EN' | 'VU' | 'NT' | 'LC';
    type KoppenClimate = |
      'Af' | 'Am' | 'As' | 'Aw' |
      'BSh' | 'BSk' | 'BWh' | 'BWk' |
      'Cfa' | 'Cfb' | 'Cfc' | 'Csa' | 'Csb' | 'Csc' | 'Cwa' | 'Cwb' | 'Cwc' |
      'Dfa' | 'Dfb' | 'Dfc' | 'Dfd' |
      'Dsa' | 'Dsb' | 'Dsc' | 'Dwa' | 'Dwb' | 'Dwc' | 'Dwd' |
      'EF' | 'ET';
    const snowLeopard: Animal = {
      commonName: 'Snow Leopard',
      genus: 'Panthera',
      species: 'Uncia',
      status: 'VU',  // vulnerable
      climates: ['ET', 'EF', 'Dfd'],  // alpine or subalpine
    };
    ```
    
    - 더 명확하고 구분할 수 있게 해결함

- 가독성을 높이고, 추상화 수준을 올리기위해 해당 분야의 용어를 사용하자
- 특별한 의미가 있을때만 용어를 붙이고, 같은의미의 용어에 다른 의미의 이름을 붙이지 말자

---

## 아이템 37 공식 명칭에는 상표를 붙이기

```tsx
interface Vector2D {
  x: number;
  y: number;
}
function calculateNorm(p: Vector2D) {
  return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm({x: 3, y: 4});  // OK, result is 5
const vec3D = {x: 3, y: 4, z: 1};
calculateNorm(vec3D);  // OK! result is also 5
```

```tsx
interface Vector2D {
  _brand: '2d';
  x: number;
  y: number;
}
function vec2D(x: number, y: number): Vector2D {
  return {x, y, _brand: '2d'};
}
function calculateNorm(p: Vector2D) {
  return Math.sqrt(p.x * p.x + p.y * p.y);  // Same as before
}

calculateNorm(vec2D(3, 4)); // OK, returns 5
const vec3D = {x: 3, y: 4, z: 1};
calculateNorm(vec3D);
           // ~~~~~ Property '_brand' is missing in type...
```

---