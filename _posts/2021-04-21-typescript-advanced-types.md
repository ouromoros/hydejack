---
layout: post
title: 'TypeScript高级类型实践'
tags: [CS]
categories: [Other]
comments: true
---

写此文的背景是`protobufjs`库生成的`d.ts`文件中解码结果的类型不符合预期，proto3协议中规定了解码结果的标量应该都具有默认值，但解码的类型定义中却依然会带有`undefined`和`null`，这导致用户代码为了使编译通过不得不对`null`和`undefined`做额外的检查（`strict`为`true`时），十分影响正常使用。最理想的情况应该是由`protobufjs`库直接在生成的文件中提供正确的解码结果，而在搜索之后只找到了一个未解决的[远古issue](https://github.com/protobufjs/protobuf.js/issues/1171)，应该是因为历史原因和兼容性考虑一直没有解决，于是就考虑自己想办法解决这个问题。调研一番之后发现TypeScript的类型操作方法异常强大，将一个类型递归地去掉`null`和`undefined`生成新的类型也是可以做到的——既然TypeScript的类型系统这么强大，谁还需要Haskell呢（误）。

## Mapped Types

要去掉一个类型所有成员字段的`undefined`和`null`，首先我们需要一个表达这种行为的方式，Mapped Types刚好就是做这件事的。

```ts
type NoUndefinedMember<Type> = {
  [Property in keyof Type]-?: NonNullable<Type[Property]>;
};
```

上面主要有两个新的语法：
1. `type Name<T> = { [K in keyof T]-?: T[K] }`表示对`T`类型的所有成员类型做一个映射得到新的类型并去掉optional的定义，`T[K]`是成员的类型，具体语法可以见[文档](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)。
2. `NonNullable<T>`表示从`T`的类型中去除`undefined`和`null`类型。

上面是非递归的版本，如果类型里面还有嵌套的结构体的话就行不通了，而幸运的是递归的实现是十分自然的，并不需要更多改动，对于基本类型（etc. `number`、`string`）自动做终止处理，而不会陷入无限递归的情况。

```ts
type NoUndefinedMember<Type> = {
  [Property in keyof Type]-?: NoUndefinedMember<NonNullable<Type[Property]>>;
};
```

如果我们的需求只是去除`null`和`undefined`的话，这个时候已经万事大吉了，但在`protobufjs`中虽然基本类型总有默认值，但`message`结构体就没有默认值了，当数据中没有相应字段时返回的便是`null`，因此需要特殊处理，此时只用Mapped Type就不能满足需求了，需要结合Conditional Type才行。

## Conditional Type

顾名思义，类型的操作符可以根据条件的真假返回不同的值，类似于`a === 1 ? true : false `的三元操作符，只不过这里操纵的值是类型而不是一般意义上的值，它的语法是这样的：

```ts
type OtherType<T extends number | string> = T extends number ? string : number;
```

这里给了一个`OtherType`的模版类型定义，它的特点是当`T`是`number`时返回`string`，当`T`是`string`时返回`number`。和三元操作符一样，我们可以通过嵌套的使用Conditional Type来进行多次条件判断，以实现多分支的结果。

那么有了Conditional Type之后，最终可以将我们的方法改写成如下形式：

```ts
type NoUndefinedField<T> = {
    [P in keyof T]-?:
    T[P] extends (infer I)[] | null | undefined ? NoUndefinedField<I>[] :
        T[P] extends number | Long | string | boolean | Buffer | Record<string, number> | null | undefined ? NonNullable<T[P]> :
            NoUndefinedField<T[P]> | null
};
```

这里面主要有以下几点：
1. 首先我们对数组类型做了特殊判断，这里使用了`infer`来推断数组的成员类型，`infer`主要就是一个方便的语法，专用于在Conditional语句中匹配数组并得到成员类型。
2. 然后我们判断`T[P]`是否是一个标量类型，这里利用了Union Type和类型继承的特性，需要注意的是由于`T[P]`中有`undefined`与`null`，因此union中也要将它们包含进去，`enum`类型可以使用`Record<string, number>`匹配。
3. 最后对于不匹配上面任何一种情况的，我们判断它为`message`结构，递归地调用该方法并加上`null`类型。

## 最后...

优化解码返回类型的工作已经大功告成了，上面给出的就是最终的结果，TypeScript的类型操作符是极其强大的，大部分变换都可以通过各种高级类型操作符做到，利用这个特性可以做很多hack，具体就自行想象了。

完成编码工作之后，免不了的是还需要对我们写的类型进行测试，遗憾的是TypeScript本身并没有可以对类型直接进行比较的表达式，不过stackoverflow上的[这篇回答](https://stackoverflow.com/a/54160691/3500362)给出了一个可行的测试方法。

```ts
type Exact<A, B> = (<T>() => T extends A ? 1 : 0) extends (<T>() => T extends B ? 1 : 0)
    ? (A extends B ? (B extends A ? unknown : never) : never)
    : never

/** Fails when `actual` and `expected` have different types. */
declare const exactType: <Actual, Expected>(
    actual: Actual & Exact<Actual, Expected>,
    expected: Expected & Exact<Actual, Expected>
) => Expected
```

它主要利用了两个trick： 如果`A extends B`且`B extends A`的话那么可以认为类型`A`和`B`相同，于是我们就可以定义一个当`A`和`B`不同时返回`never`的模版类型，再利用TypeScript的类型推断，使得只有传入两个参数的类型相同的时候`exactType`才能通过编译，否则会因为入参类型为`never`而导致编译错误（`never`类型不能作为入参）。

有了这样一个函数之后我们就可以放心地利用它构造单元测试了，有一个小小的不足在于当同一文件中任何一个用例有问题时，运行单元测试的时候会触发编译错误（类型检查错误）而直接失败，无法运行其它的测试。不过在TypeScript缺乏直接检查类型相同的能力的背景下，这已经是best effort了。
