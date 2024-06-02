# Solution
## 4 - Pick
### ❌ 시도1
```typescript
type MyPick<T, K> = K extends string ? {[key in K] : T[key]} : never  // ❌Error: Type 'k' cannot be used to index type 'T'.
```
좌항을 수정하면 안되는줄 알았다. 그래서 다음과 같이 사고하였다:
1. `K`가 union type임을 TypeScript에게 알려주자. 그러면 `[key in K]`를 통해 집합 `K`의 원소들을 key로 추출할 수 있게될 것이다
2. 추출해낸 `key`를 사용하여 `T[key]`로 인덱싱하여 value로 사용한다
3. 1에서 생성한 key와 2에서 생성한 value를 `{}`로 묶어 object type으로 정의하자  

하지만 TypeScript는 `T`가 추출해낸 `key`라는 프로퍼티를 가지고 있는지 알 수가 없기 때문에 에러를 발생시킨다

### ✅ 시도2
```typescript
type MyPick<T, K extends keyof T> = { [key in K]: T[key] }
```
집합 `K`가 `T`의 key들로 이루어진 집합이라 정의해버리면 더 간단히 해결될 문제였다. 이를 위해 좌항의 `K`를 `T`의 key집합들로 구속(constrain)시켰다(참조: [HandBook - Using Type Parameters in Generic Constraints](https://www.typescriptlang.org/docs/handbook/2/generics.html#using-type-parameters-in-generic-constraints))

## 7 - Readonly

쉽게 접근하였다. 기존의 객체 타입에서 key, value를 추출하여 readonly 키워드만을 붙여주는 방식을 시도하였다:

```tsx
type MyReadonly<T> = {
  readonly [k in keyof T]: T[k]
}
```

여기서 약간의 애를 먹었던 부분은 `[k in keyof T]`를 통해 객체의 key에 접근하는 것이었다. 여전히 `[...]`와 `in`의 문법이 정확하게 무엇인지 파악하지 못하였다(자세한 내용은 [Handbook - Mapped Types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)를 참고하면 된다)

## 11 - Tuple to Object

### ❌ 시도 1

```tsx
type TupleToObject<T extends readonly any[]> = {
  [item in T[number]]: item
}
```

generic인 `T`는 Array type으로 constraints 되어있다. Array type은 `number`로 색인 되어있으므로, `item in T[number]`를 통해 아이템에 접근이 가능하다(자세한 내용은 [Handbook - Indexed Access Types](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html)를 참조)

좋은 시도였다. 하지만 Array type의 원소 타입이 `any`로 지정되어있었기 때문에 다음과 같은 타입 정의에서 에러가 발생하지 않았다:

```tsx
// @ts-expect-error
type error = TupleToObject<[[1, 2], {}]>  // 에러 발생해야함
```

`item`을 index로 사용하기 때문에, item은 index signature 타입으로 사용되어야만 한다

에러가 발생해야 하는 이유는: index signature의 타입은 오직 `string`, `number`, `symbol`, template literal만이 가능하다는 규칙 때문이다(참조: [Handbook - Object Types / Index Signature](https://www.typescriptlang.org/docs/handbook/2/objects.html#index-signatures))

만일 `item`이 객체의 key 타입으로 사용하지 않았다면 에러를 발생할 필요가 없었을 것이다

### ✅ 시도 2

```tsx
type TupleToObject<T extends readonly (string | number | symbol)[]> = {
  [item in T[number]]: item
}

// @ts-expect-error
type error = TupleToObject<[[1, 2], {}]>  // ❌이제 에러가 발생한다
```

위와 같이 generic T의 타입을 허용 가능한 index signature 타입들(`string`, `number`, `symbol`)로만 구속시켜 에러가 발생하게끔 만들어주었다