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