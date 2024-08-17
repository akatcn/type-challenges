# Solution

## **4 - Pick**

### **❌ 시도1**

```typescript
type MyPick<T, K> = K extends string ? {[key in K] : T[key]} : never  // ❌Error: Type 'k' cannot be used to index type 'T'.
```

좌항을 수정하면 안되는줄 알았다. 그래서 다음과 같이 사고하였다:

1. `K`가 union type임을 TypeScript에게 알려주자. 그러면 `[key in K]`를 통해 집합 `K`의 원소들을 key로 추출할 수 있게될 것이다
2. 추출해낸 `key`를 `T[key]`로 인덱싱하여 value로 사용한다
3. 1에서 생성한 key와 2에서 생성한 value를 `{}`로 묶어 object type으로 정의하자

하지만 TypeScript는 `T`가 추출해낸 `key`라는 프로퍼티를 가지고 있는지 알 수가 없기 때문에 에러를 발생시킨다

### **✅ 시도2**

```typescript
type MyPick<T, K extends keyof T> = { [key in K]: T[key] }
```

집합 `K`가 `T`의 key들로 이루어진 집합이라 정의해버리면 더 간단히 해결될 문제였다. 이를 위해 좌항의 `K`를 `T`의 key집합들로 구속(constrain)시켰다(참조: [Handbook - Generic Constraints](https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-constraints))

---

## 7 - Readonly

### ✅ 시도 1

쉽게 접근하였다. 기존의 객체 타입에서 key, value를 추출하여 readonly 키워드만을 붙여주는 방식을 시도하였다:

```typescript
type MyReadonly<T> = {
  readonly [k in keyof T]: T[k]
}
```

여기서 약간의 애를 먹었던 부분은 `[k in keyof T]`를 통해 객체의 key에 접근하는 것이었다. 여전히 `[...]`와 `in`의 문법이 정확하게 무엇인지 파악하지 못하고있다(자세한 내용은 [Handbook - Mapped Types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)를 참고하면 된다)

---

## 11 - Tuple to Object

### ❌ 시도 1

```typescript
type TupleToObject<T extends readonly any[]> = {
  [item in T[number]]: item
}
```

generic인 `T`는 Array type으로 constraints 되어있다. Array type은 `number`로 색인 되어있으므로, `item in T[number]`를 통해 아이템에 접근이 가능하다(자세한 내용은 [Handbook - Indexed Access Types](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html)를 참조)

좋은 시도였다. 하지만 Array type의 원소 타입이 `any`로 지정되어있었기 때문에 다음과 같은 타입 정의에서 에러가 발생하지 않았다:

```typescript
// @ts-expect-error
type error = TupleToObject<[[1, 2], {}]>  // 에러 발생해야함
```

`item`을 index로 사용하기 때문에, item은 index signature 타입으로 사용되어야만 한다

에러가 발생해야 하는 이유는: index signature의 타입은 오직 `string`, `number`, `symbol`, template literal만이 가능하다는 규칙 때문이다(참조: [Handbook - Object Types / Index Signature](https://www.typescriptlang.org/docs/handbook/2/objects.html#index-signatures))

만일 `item`이 객체의 key 타입으로 사용하지 않았다면 에러를 발생할 필요가 없었을 것이다

### ✅ 시도 2

```typescript
type TupleToObject<T extends readonly (string | number | symbol)[]> = {
  [item in T[number]]: item
}

// @ts-expect-error
type error = TupleToObject<[[1, 2], {}]>  // ❌이제 에러가 발생한다
```

위와 같이 generic T의 타입을 허용 가능한 index signature 타입들(`string`, `number`, `symbol`)로만 구속시켜 에러가 발생하게끔 만들어주었다

---

## 14 - First of Array

### ❌ 시도 1

```typescript
type First<T extends any[]> = T[0]
```

단순하게 생각하였다. `T`는 `any[]`로 constraints 되어있으므로, indexed access type `T[0]`을 통해 첫 번째 원소의 타입을 가져오는 것이었다.

대부분의 케이스에선 문제가 없었지만 빈 배열 `[]`에선 `undefined`를 반환하게 되어 타입 체크 에러가 발생하였다

### ✅ 시도 2

```typescript
type First<T extends any[]> = T extends [] ? never : T[0]
```

결국 답을 봤다. 답을 보는 순간 아차 싶었다. 조건부 타입([Handbook - Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html))을 사용할 생각은 있었지만 `T extends []`를 “`T`가 배열 타입을 extends하는가??” 로만 생각하였다. 하지만 `T extends []`가 의미하는 바는 “`T`가 **빈 배열**을 extends하는가??” 였다. **배열 타입**을 extends 하는지 점검하려면 `T extends any[]`를 사용해야만 한다. 정리하자면:

- `T extends []`: 타입 T가 정확히 빈 배열인지 확인하는 조건
- `T extends any[]`: 타입 T가 배열 타입인지 확인하는 조건

이 문제는 배열 타입이 아닌, 빈 배열인지를 점검해야 하므로 `T extends []`를 사용하면 된다

---

## 18 - Length of Tuple

### ✅ 시도 1

```typescript
type Length<T extends readonly any[]> = T['length']
```

튜플에 대한 이해만 있었다면 간단히 풀 수 있는 문제였다

Tuple이란? 튜플 타입은 배열 타입의 또 다른 종류로, 포함된 요소의 수와 특정 위치에 포함된 유형을 정확히 알 수 있는 타입을 의미한다(출처: [Object Types - Tuple Types](https://www.typescriptlang.org/docs/handbook/2/objects.html#tuple-types))

문제에서 주어진 generic의 타입이 readonly 배열이었기 때문에 generic `T`의 타입을 `readonly any[]`로 constraint하였다. 그런다음 배열 타입에 존재하는 `length` 프로퍼티를 Indexed Access Types 문법인 `T['length']`로 읽어와 길이를 반환하였다

---

## 43 - Exclude

### ❌ 시도 1

삽질에 가까운 시도만 하였다. 생각했던 로직으로는:

- (좌항)generic `T`, `U`를 각각 유니언 타입으로 constraint하기
- (우항)`T`로부터 유니언 타입의 멤버를 각각 조회할 수 있는 연산자를 찾아내고, 마찬가지로 `U`로부터 유니언 타입의 멤버를 각각 조회한다음 서로 비교하여 일치하지 않는 `T`의 타입만 남겨두어 반환

하지만 이런 구현 방법에 대한 충분한 지식이 없어 제대로 시도하지 못하고 실패하였다. 돌이켜보니 애초에 불가능한 동작을 구현하려 했던 것 아닌가 하는 생각이 든다

### ✅ 시도 2

결국 힌트를 보았다. 바로 [Conditional Types - Distributive Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types)라는 내용을 참고해야만 한다는 것을 말이다. Distributive Conditional Types을 간단히 설명하자면, 조건부 타입을 적용할 때 generic 타입이 유니언 타입일 경우, 멤버 각각에 조건부 타입이 적용됨을 의미한다

예를들어:

```typescript
type Dist<T> = T extends any ? [T] : never
```

위의 식에서 generic `T`가 유니언 타입 `"A" | "B"`라면, 조건부 타입이 멤버마다 각각 적용되어 다음과 같이 반환된다:

```typescript
type Res = Dist<"A" | "B">  // Res의 타입: ["A"] | ["B"]
```

이러한 문법을 적용하여 다음과 같이 풀이하였다:

```typescript
type MyExclude<T, U> = T extends U ? never : T
```

코드를 설명하자면 다음과 같다:

- `T extends U ?` → `T`가 유니언 타입일때, `T`의 멤버 각각 마다 `U`의 부분집합인지 점검한다:
    - `T`의 멤버가 `U`의 부분집합이라면: `never`를 반환한다
    - `T`의 멤버가 `U`의 부분집합이 아니라면: 멤버를 그대로 반환한다

---

## 189 - Awaited

### ❌ 시도 1

```typescript
type MyAwaited<T extends Promise<any>> = T extends Promise<infer x> ? (x extends Promise<any> ? MyAwaited<x> : x) : T
```

조건부 타이핑 과정에서 `infer` 키워드를 사용하여 수동으로 직접 `Promise`에 넘겨진 타입을 추출하였다. 이때 조건부 식은 삼항 연산자를 두 개 연달아 쓴 것처럼 사용하였다. 각각 설명하자면:

- `type MyAwaited<T extends Promise<any>>` → generic `T`를 `Promise` 타입으로 constraint 한다
- `T extends Promise<infer x> ? (...) : T` → generic `T`가 `Promise` 타입인지 점검하는 동시에 `infer` 키워드를 통해 `x`를 추출한다:
    - `T`가 `Promise` 타입이면 → 조건부 타이핑을 한 번 더 실행한다(바로 아래에 설명되는 조건부 타이핑 식)
    - `T`가 `Promise` 타입이 아니라면 → `T`를 그대로 반환한다
- `(x extends Promise<any> ? MyAwaited<x> : x)` → 추출해낸 `x`가 `Promise` 타입인지 다시한번 점검한다:
    - `x`가 `Promise` 타입이면 → `MyAwaited<x>`를 한 번 더 호출하여 재귀적으로 타입을 추출해낸다. Promi
    - `x`가 `Promise` 타입이 아니라면 → `x`를 그대로 반환한다

하지만 다음의 테스트 케이스를 통과할 수 없었다:

```typescript
type T = { then: (onfulfilled: (arg: number) => any) => any }

Expect<Equal<MyAwaited<T>, number>>  // ❌ Error
```

뿐만 아니라, 조건부 타이핑을 두 개 연달아 사용하는 것이 매우 복잡해보이는 문제점도 있었다. 조금 더 간결한 방법을 찾아야만 한다

### ✅ 시도 2

도저히 해결할 수 없어 결국 솔루션을 보았다. 해결 방법은 매우 간단하였다: 바로 `Promise` 타입 대신에 `PromiseLike`를 사용하는 것이었다

`Promise`와 `PromiseLike`의 차이점은 다음과 같다:

- **`Promise`**:
    - 이 타입은 자바스크립트의 표준 `Promise`를 정확하게 나타낸다
    - `Promise` 객체는 `.then`, `.catch`, `.finally` 메서드를 가지고 있으며, 이러한 메서드들이 항상 제공되어야만 한다
- **`PromiseLike`**:
    - `PromiseLike`는 표준 `Promise`와 동일한 인터페이스를 가지지만, 반드시 `Promise` 객체일 필요는 없다
    - `PromiseLike` 타입은 `then` 메서드만 가지는 객체를 의미한다. 즉, `thenable` 객체를 나타낼 수 있다
    - 이 타입은 비표준 `Promise` 구현체 또는 커스텀 `Promise`와 유사한 객체를 지원하기 위해 사용된다

다른 솔루션을 찾아봐도 이보다 더 간결하게 표현하는 솔루션은 찾을 수 없었다. 나중에라도 찾게되면 업데이트 해야겠다

---

## Recap

- 객체 타입에 `keyof` 연산자를 사용하면 객체의 프로퍼티명을 추출하여 문자열 유니언 타입으로 반환한다
- `T extends []`와 `T extends any[]`는 다르다:
    - `T extends []`: 타입 T가 **정확히 빈 배열**인지 확인하는 조건
    - `T extends any[]`: 타입 T가 **배열 타입**인지 확인하는 조건
- 제네릭이 유니언 타입일때, 해당 제네릭에 대하여 조건식 `extends`를 적용하면, 유니언 타입의 멤버 각각에 대해 조건식이 적용된다(자세한 내용 참조: [Conditional Types - Distributive Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types))