# Week 3

## Get Return Type

내장 제네릭 `ReturnType<T>`을 이를 사용하지 않고 구현하세요.

예시:

```ts
const fn = (v: boolean) => {
  if (v)
    return 1
  else
    return 2
}

type a = MyReturnType<typeof fn> // should be "1 | 2"
```

```ts
type cases = [
  Expect<Equal<string, MyReturnType<() => string>>>,
  Expect<Equal<123, MyReturnType<() => 123>>>,
  Expect<Equal<ComplexObject, MyReturnType<() => ComplexObject>>>,
  Expect<Equal<Promise<boolean>, MyReturnType<() => Promise<boolean>>>>,
  Expect<Equal<() => 'foo', MyReturnType<() => () => 'foo'>>>,
  Expect<Equal<1 | 2, MyReturnType<typeof fn>>>,
  Expect<Equal<1 | 2, MyReturnType<typeof fn1>>>,
]

type ComplexObject = {
  a: [12, 'foo']
  bar: 'hello'
  prev(): number
}

const fn = (v: boolean) => v ? 1 : 2
const fn1 = (v: boolean, w: any) => v ? 1 : 2
```

### 문제 분석

해당 타입 제네릭에 함수 타입이 들어갈 경우 그 return 타입을 반환하는 유틸리티 타입으로 보인다.

조건문으로 반환하는 경우 그 조건문에 양쪽에서 반환가능한 답을 유니온 타입으로 반환한다.



### 첫번째 접근

```ts
type MyReturnType<T> = T extends (...arg:any[]) => (infer P) ? MyReturnType<P> : T
```

Awaited 유틸리티 타입의 형태를 참고해서 infer를 재귀하는 방식으로 구성했다.

연쇄적으로 작동하는 함수의 리턴타입을 반환하지는 못했다

오히려 조건부로 반환하는 타입을 유니온타입으로 반환하는 동작은 안될 줄 알았는데 반환했다.



### 두번째 접근

```ts
type MyReturnType<T> = T extends (...args: any[]) => (infer P) ? P : never
```

문제 이해를 잘못한 것이었다

마지막 반환 타입을 반환하는걸로 이해했는데 그냥 첫번째 반환값을 가져오면 되는 것이었다.



## Omit

`T`에서 `K` 프로퍼티만 제거해 새로운 오브젝트 타입을 만드는 내장 제네릭 `Omit<T, K>`를 이를 사용하지 않고 구현하세요.

예시:

```ts
interface Todo {
  title: string
  description: string
  completed: boolean
}

type TodoPreview = MyOmit<Todo, 'description' | 'title'>

const todo: TodoPreview = {
  completed: false,
}
```

```ts
type MyOmit<T, K> = any

/* _____________ 테스트 케이스 _____________ */
import type { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<Expected1, MyOmit<Todo, 'description'>>>,
  Expect<Equal<Expected2, MyOmit<Todo, 'description' | 'completed'>>>,
  Expect<Equal<Expected3, MyOmit<Todo1, 'description' | 'completed'>>>,
]

```

### 문제 분석

객체 형식의 타입에서 두번째 제네릭 값에 포함되는 키를 제외하고 다시 객체 형식의 타입으로 반환한다



### 첫번째 접근

```ts
type FirstMyOmit<T, K> = {
  [P in (keyof T)] : P extends K ? never : T[P]
}
```

T의 키값을 순회하면서 그 키값이 K에 포함되지 않으면 반환하지 않도록 설정했다

하지만 다시 보니 이런경우 그 키값이 빠지는게 아니라 K에 속하는 경우 value값만 never로 반환할 뿐이었다

### 두번째 접근

```ts
type MyOmit<T, K> = {
  [P in ((keyof T) extends K ? never : (keyof T))] : T[P]
}
```

그래서 애초에 순회하는 타입을 제한해서 순회하는 것으로 생각했다

T의 키값에 분배 법칙으로 K에 포함되는지 테스트 해서 유니온 타입으로 만든 다음에 순회를 돌리려 했다

### 세번째 접근

```ts
type MyOmit<T, K> = {
  [P in keyof T as P extends K ? never : P] : T[P]
}
```

as를 해야 위에 의도한 대로 작동을 했다.



### as

as는 여러방식으로 쓰이는데 일단 여기서는 조건부 필터링

로 쓰인다

순회를 돌때 그 P값이 뒤에 조건에 따라 필터링이 되는 것이다

모양은 두번째 접근이랑 비슷한데 동작은 첫번째 접근에서 의도한 대로 동작한다



두번째 접근이 T를 먼저 한정 -> P 순회라면

첫번째 접근은 P순회한 이후 -> P를 한정

인데 as를 사용할 경우 순회해서 P를 만들어 낸 후 그 P를 조건부로 필터링할 수 있게 된다



## Readonly 2

`T`에서 `K` 프로퍼티만 읽기 전용으로 설정해 새로운 오브젝트 타입을 만드는 제네릭 `MyReadonly2<T, K>`를 구현하세요. `K`가 주어지지 않으면 단순히 `Readonly<T>`처럼 모든 프로퍼티를 읽기 전용으로 설정해야 합니다.

예시:

```ts
interface Todo {
  title: string
  description: string
  completed: boolean
}

const todo: MyReadonly2<Todo, 'title' | 'description'> = {
  title: "Hey",
  description: "foobar",
  completed: false,
}

todo.title = "Hello" // Error: cannot reassign a readonly property
todo.description = "barFoo" // Error: cannot reassign a readonly property
todo.completed = true // OK
```



```ts
import type { Alike, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Alike<MyReadonly2<Todo1>, Readonly<Todo1>>>,
  Expect<Alike<MyReadonly2<Todo1, 'title' | 'description'>, Expected>>,
  Expect<Alike<MyReadonly2<Todo2, 'title' | 'description'>, Expected>>,
  Expect<Alike<MyReadonly2<Todo2, 'description' >, Expected>>,
]

```



### 첫번째 접근

```ts
type MyReadonly2<T, K> = {
  [P in keyof T as P extends K ? readonly P : P]: T[P]
}
```

readonly는 개체의 속성에만 붙일 수 있는 거여서 저렇게는 안붙는다

### 두번째 접근

```ts
type MyReadonly2<T, K> = {
  readonly [P in keyof T as P extends K ? P : never]: T[P]
} & {
  [P in keyof T as P extends K ? never : P]: T[P]
}
```

이렇게 했는데

이 경우 

```ts
  Expect<Alike<MyReadonly2<Todo1>, Readonly<Todo1>>>,
```

가 통과가 안되고

에러도 안된다

### 세번째 접근

```ts
type MyReadonly2<T, K extends keyof T> = {
  readonly [P in keyof T as P extends K ? P : never]: T[P]
} & {
  [P in keyof T as P extends K ? never : P]: T[P]
}
```

일단 에러를 발생시켰다

### 네번째 접근

```ts
type MyReadonly2<T, K extends keyof T = keyof T> = {
  readonly [P in keyof T as P extends K ? P : never]: T[P]
} & {
  [P in keyof T as P extends K ? never : P]: T[P]
}
```

두번째 인자가 없을 경우 그냥 전체를 readonly를 해야 했다

#### 제네릭 기본값

제네릭에 = 를 붙이면 기본값을 넣을 수 있었다



## Deep Readonly

객체의 프로퍼티와 모든 하위 객체를 재귀적으로 읽기 전용으로 설정하는 제네릭 `DeepReadonly<T>`를 구현하세요.

이 챌린지에서는 타입 파라미터 `T`를 객체 타입으로 제한하고 있습니다. 객체뿐만 아니라 배열, 함수, 클래스 등 가능한 다양한 형태의 타입 파라미터를 사용하도록 도전해 보세요.

예시:

```ts
type X = { 
  x: { 
    a: 1
    b: 'hi'
  }
  y: 'hey'
}

type Expected = { 
  readonly x: { 
    readonly a: 1
    readonly b: 'hi'
  }
  readonly y: 'hey' 
}

type Todo = DeepReadonly<X> // should be same as `Expected`
```



### 첫번째 접근

```ts
type DeepReadonly<T> = {
  readonly [P in keyof T as P extends DeepReadonly<P> ? DeepReadonly<P> : P]: T[P]
}
```

as 필터링을 이용했다

다만 객체가 있는 상태인지 필터링을 제한하는 방법이 혼란스러웠다



### 답

```ts
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object
    ? T[P] extends Function
      ? T[P]
      : DeepReadonly<T[P]>
    : T[P];
};
```

이렇게 하면 된다 한다

value값에 재귀를 걸어주면 된다



## 10 Tuple To Union

튜플 값으로 유니온 타입을 생성하는 제네릭 `TupleToUnion<T>`를 구현하세요.

예시:

```ts
type Arr = ['1', '2', '3']

type Test = TupleToUnion<Arr> // expected to be '1' | '2' | '3'
```

```ts
type cases = [
  Expect<Equal<TupleToUnion<[123, '456', true]>, 123 | '456' | true>>,
  Expect<Equal<TupleToUnion<[123]>, 123>>,
]
```

답

```ts
type TupleToUnion<T extends unknown[]> = T[number]
```



## Chainable Option

체인 가능 옵션은 일반적으로 Javascript에서 사용됩니다. 하지만 TypeScript로 전환하면 제대로 구현할 수 있나요?

이 챌린지에서는 `option(key, value)`과 `get()` 두가지 함수를 제공하는 객체(또는 클래스) 타입을 구현해야 합니다. 현재 타입을 `option`으로 지정된 키와 값으로 확장할 수 있고 `get`으로 최종 결과를 가져올 수 있어야 합니다.

예시

```ts
declare const config: Chainable

const result = config
  .option('foo', 123)
  .option('name', 'type-challenges')
  .option('bar', { value: 'Hello World' })
  .get()

// 결과는 다음과 같습니다:
interface Result {
  foo: number
  name: string
  bar: {
    value: string
  }
}
```



문제를 해결하기 위해 js/ts 로직을 작성할 필요는 없습니다. 단지 타입 수준입니다.

`key`는 `string`만 허용하고 `value`는 무엇이든 될 수 있다고 가정합니다. 같은 `key`는 두 번 전달되지 않습니다.



```ts
type Chainable = {
  option(key: string, value: any): any
  get(): any
}

/* _____________ 테스트 케이스 _____________ */
import type { Alike, Expect } from '@type-challenges/utils'

declare const a: Chainable

const result1 = a
  .option('foo', 123)
  .option('bar', { value: 'Hello World' })
  .option('name', 'type-challenges')
  .get()

const result2 = a
  .option('name', 'another name')
  // @ts-expect-error
  .option('name', 'last name')
  .get()

const result3 = a
  .option('name', 'another name')
  // @ts-expect-error
  .option('name', 123)
  .get()

type cases = [
  Expect<Alike<typeof result1, Expected1>>,
  Expect<Alike<typeof result2, Expected2>>,
  Expect<Alike<typeof result3, Expected3>>,
]

type Expected1 = {
  foo: number
  bar: {
    value: string
  }
  name: string
}

type Expected2 = {
  name: string
}

type Expected3 = {
  name: number
}
```



### 답

```ts
type Chainable<T = {}> = {
  option: <K extends string, V>(key: K extends keyof T ?
    V extends T[K] ? never : K
    : K, value: V) => Chainable<Omit<T, K> & Record<K, V>>
  get: () => T
}
```

그렇다고 합니다.