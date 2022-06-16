## 泛型

```typescript
function firstElement<Type>(arr: Array<Type>): Type {
  return arr[0];
}
```

`<Type>` 相当于声明一个类型变量，`Type[]`  是对变量的使用。类型变量的取值在使用这个函数时确定。

如何确定在 `Array<string>` 中哪一个是泛型 `Type` ？ 根据泛型定义的 **结构**



### 对泛型变量进行限定 constraints（泛型约束）

```typescript
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}
```

`<Type extends { length: number }>` 限定了这个泛型需要有 length 属性且 length 的类型是 number



### 指定泛型的类型

在使用时：`firstElement<number>`







### Index Signatures（索引类型）

当不知道属性名时

```typescript
interface StringArray {
  [index: number]: string;
}
```

`index signature` 的属性类型只能是 `string` 或者  `number`





### 类型组合的两种方式（interface、type）

#### 类型扩展 extends

```typescript
interface Colorful {
  color: string;
}
 
interface Circle {
  radius: number;
}
 
interface ColorfulCircle extends Colorful, Circle {}
```





#### 类型相交 &

```typescript
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}
 
type ColorfulCircle = Colorful & Circle;
```





#### 区别

interface 默认合并相同的类型，type 不能声明相同的类型





### 元祖类型

元祖类型是知道包含了哪些类型的数组类型





### Index Access Types

```typescript
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"];
```

或者

使用 `number` 获取数据的元素的类型

```typescript
const arr = ['c', 'd']
type type = typeof arr;
type t = type[number] // t --> string
```





### infer

声明式的引入了一个类型变量（泛型）

```typescript
type GetReturnType<Type> = Type extends (...args: never[]) => infer Return
  ? Return
  : never;
```

`Type`    `extends`    `(...args: never[]) => infer Return`    `? Return : never`

`infer`  声明了一个类型变量 `Return`

`extends`  和 `?` 之间的是条件类型的 主体

