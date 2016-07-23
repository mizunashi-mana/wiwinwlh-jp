# 言語拡張

## はじめに

言語拡張の分類を識別するのは大事なことです。

言語拡張を**一般**と**特殊**の部門に分類すること自体に潜んでいる問題は、これは主観的な分類であるということです。型の宇宙遊泳を行う Haskeller は、データベースプログラミングを行う人とは Haskell の解釈が大きく異なっています。ここで提示するもの自体は保守的な評価です。恣意的な基準として、``FlexibleInstances`` と ``OverloadedStrings`` は”日常的”、``GADTs`` と ``TypeFamilies`` は”特殊”としましょう。

### 要点

* **無難**は、その拡張を導入しても使用しない限りモジュールの意味論は変わらない、ということを表します。
* **歴史的**は、その拡張は使うべきでなく、単に後方互換性だけのために GHC にある、ということを表します。

言語拡張 | 無難 | 歴史的 | 構文拡張 | 用途 | 用途 | 手引き
--- | :---: | :---: | :---: | :---: | :---: | :---:
AllowAmbiguousTypes |  |  |  | 特殊 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/other-type-extensions.html#ambiguity)
Arrows |  |  | ○ | 特殊 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/arrow-notation.html)
AutoDeriveTypeable |  |  |  | 特殊 | メタ | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/deriving.html#auto-derive-typeable)
BangPatterns | ○ |  | ○ | 一般 | 正格性注釈 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/bang-patterns.html)
CApiFFI |  |  |  | 特殊 | FFI | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/ffi.html#ffi-capi)
ConstrainedClassMethods |  |  |  | 特殊 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/type-class-extensions.html#class-method-types)
ConstraintKinds |  |  |  | 特殊 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/constraint-kind.html)
CPP | ○ |  | ○ | 一般 | プリプロセッサ | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/options-phases.html#c-pre-processor)
DataKinds |  |  |  | 特殊 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/promotion.html)
DatatypeContexts |  | ○ | ○ | 非推奨 | 非推奨 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/data-type-extensions.html#datatype-contexts)
DefaultSignatures | ○ |  |  | 特殊 | ジェネリック | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/type-class-extensions.html#class-default-signatures)
DeriveDataTypeable | ○ |  |  | 一般 | ジェネリック | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/deriving.html#deriving-typeable)
DeriveFoldable | ○ |  |  | 一般 | ジェネリック | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/deriving.html#deriving-typeable)
DeriveFunctor | ○ |  |  | 一般 | ジェネリック | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/deriving.html#deriving-typeable)
DeriveGeneric | ○ |  |  | 一般 | ジェネリック | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/deriving.html#deriving-typeable)
DeriveTraversable | ○ |  |  | 一般 | ジェネリック | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/deriving.html#deriving-typeable)
DisambiguateRecordFields | ○ |  | ○ | 特殊 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#disambiguate-fields)
DoRec |  | ○ | ○ | 特殊 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#recursive-do-notation)
EmptyCase |  |  |  | 特殊 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#empty-case)
EmptyDataDecls | ○ |  |  | 一般 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/data-type-extensions.html#nullary-types)
ExistentialQuantification |  |  |  | 特殊 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/data-type-extensions.html#existential-quantification)
ExplicitForAll |  |  | ○ | 特殊 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/other-type-extensions.html#explicit-foralls)
ExplicitNamespaces | ○ |  | ○ | 特殊 | 構文の明確化 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#explicit-namespaces)
ExtendedDefaultRules | ○ |  |  | 特殊 | ジェネリック | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/interactive-evaluation.html#extended-default-rules)
FlexibleContexts |  |  |  | 一般 | 型クラスの拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/other-type-extensions.html#flexible-contexts)
FlexibleInstances |  |  |  | 一般 | 型クラスの拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/type-class-extensions.html#instance-rules)
ForeignFunctionInterface |  |  | ○ | 一般 | FFI | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/ffi.html)
FunctionalDependencies |  |  |  | 一般 | 型クラスの拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/type-class-extensions.html#functional-dependencies)
GADTs |  |  |  | 一般 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/data-type-extensions.html#gadt)
GADTSyntax |  |  | ○ | 一般 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/data-type-extensions.html#gadt-style)
GeneralizedNewtypeDeriving |  |  |  | 一般 | 型クラスの拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/deriving.html#newtype-deriving)
GHCForeignImportPrim |  |  |  | 特殊 | FFI | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/ffi.html#ffi-prim)
ImplicitParams |  |  |  | 特殊 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/other-type-extensions.html#implicit-parameters)
ImpredicativeTypes |  |  |  | 特殊 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/other-type-extensions.html#impredicative-polymorphism)
IncoherentInstances |  |  |  | 特殊 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/type-class-extensions.html#instance-overlap)
InstanceSigs |  |  |  | 特殊 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/type-class-extensions.html#instance-sigs)
InterruptibleFFI |  |  |  | 特殊 | FFI | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/ffi.html#ffi-interruptible)
KindSignatures |  |  |  | 特殊 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/other-type-extensions.html#kinding)
LambdaCase | ○ |  | ○ | 一般 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#lambda-case)
LiberalTypeSynonyms |  |  |  | 特殊 | 型クラスの拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/data-type-extensions.html#type-synonyms)
MagicHash |  |  |  | 特殊 | GHC 内部 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#magic-hash)
MonadComprehensions |  |  | ○ | 特殊 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#monad-comprehensions)
MonoPatBinds |  |  |  | 特殊 | 型の明確化 | [参照](http://www.haskell.org/ghc/docs/7.6.3/html/users_guide/monomorphism.html)
MultiParamTypeClasses | ○ |  |  | 一般 | 型クラスの拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/type-class-extensions.html#multi-param-type-classes)
MultiWayIf |  |  | ○ | 特殊 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#multi-way-if)
NamedFieldPuns |  |  | ○ | 特殊 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#record-puns)
NegativeLiterals |  |  |  | 一般 | 型の明確化 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#negative-literals)
NoImplicitPrelude |  |  |  | 特殊 | インポートの明確化 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#rebindable-syntax)
NoMonoLocalBinds |  |  |  | 一般 | 型の明確化 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/other-type-extensions.html#mono-local-binds)
NoMonomorphismRestriction |  |  |  | 一般 | 型の明確化 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/other-type-extensions.html#monomorphism)
NPlusKPatterns |  | ○ | ○ | 非推奨 | 非推奨 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#n-k-patterns)
NullaryTypeClasses |  |  |  | 特殊 | 型クラスの拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/type-class-extensions.html#nullary-type-classes)
NumDecimals |  |  |  | 一般 | 型の明確化 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#num-decimals)
OverlappingInstances |  |  |  | 特殊 | 型クラスの拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/type-class-extensions.html#instance-overlap)
OverloadedLists |  |  | ○ | 一般 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/type-class-extensions.html#overloaded-lists)
OverloadedStrings |  |  |  | 一般 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/type-class-extensions.html#overloaded-strings)
PackageImports |  |  | ○ | 一般 | インポートの明確化 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#package-imports)
ParallelArrays |  |  |  | 特殊 | DPH | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/lang-parallel.html)
ParallelListComp |  |  | ○ | 一般 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#parallel-list-comprehensions)
PatternGuards |  |  | ○ | 一般 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#pattern-guards)
PatternSynonyms | ○ |  | ○ | 一般 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#pattern-synonyms)
PolyKinds |  |  |  | 特殊 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/kind-polymorphism.html)
PolymorphicComponents |  | ○ |  | 特殊 | 非推奨 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/other-type-extensions.html#universal-quantification)
PostfixOperators | ○ |  | ○ | 特殊 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#postfix-operators)
QuasiQuotes |  |  |  | 特殊 | メタ | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/template-haskell.html#th-quasiquotation)
Rank2Types |  | ○ |  | 特殊 | 歴史の産物 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/other-type-extensions.html#universal-quantification)
RankNTypes |  |  |  | 特殊 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/other-type-extensions.html#universal-quantification)
RebindableSyntax |  |  | ○ | 特殊 | メタ | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#rebindable-syntax)
RecordWildCards | ○ |  | ○ | 一般 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#record-wildcards)
RecursiveDo |  |  |  | 特殊 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#recursive-do-notation)
RelaxedPolyRec |  |  |  | 特殊 | 型の明確化 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/other-type-extensions.html#typing-binds)
RoleAnnotations |  |  |  | 特殊 | 型の明確化 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/roles.html)
Safe |  |  |  | 特殊 | 安全性の検査 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/safe-haskell.html)
SafeImports |  |  |  | 特殊 | 安全性の検査 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/safe-haskell.html)
ScopedTypeVariables |  |  |  | 特殊 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/other-type-extensions.html#scoped-type-variables)
StandaloneDeriving | ○ |  | ○ | 一般 | 型クラスの拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/deriving.html#stand-alone-deriving)
TemplateHaskell | ○ |  | ○ | 特殊 | メタ | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/template-haskell.html)
TraditionalRecordSyntax |  | ○ | ○ | 特殊 | 歴史の産物 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#traditional-record-syntax)
TransformListComp |  |  | ○ | 特殊 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#generalised-list-comprehensions)
Trustworthy |  |  |  | 特殊 | 安全性の検査 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/safe-haskell.html)
TupleSections | ○ |  |  | 一般 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#tuple-sections)
TypeFamilies |  |  |  | 特殊 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/type-families.html)
TypedHoles | ○ |  |  | 一般 | 対話的型付け | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/typed-holes.html)
TypeOperators |  |  |  | 特殊 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/data-type-extensions.html#type-operators)
TypeSynonymInstances | ○ |  |  | 一般 | 型クラスの拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/type-class-extensions.html#flexible-instance-head)
UnboxedTuples |  |  |  | 特殊 | FFI | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/primitives.html#unboxed-tuples)
UndecidableInstances |  |  |  | 特殊 | 型レベル | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/type-class-extensions.html#undecidable-instances)
UnicodeSyntax |  |  | ○ | 特殊 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#unicode-syntax)
UnliftedFFITypes |  |  |  | 特殊 | FFI | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/primitives.html)
Unsafe |  |  |  | 特殊 | 安全性の検査 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/safe-haskell.html)
ViewPatterns | ○ |  | ○ | 一般 | 構文の拡張 | [参照](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/syntax-extns.html#view-patterns)

参照：

* [GHC Extension Reference](http://www.haskell.org/ghc/docs/7.8.4/html/users_guide/flag-reference.html#idp14710736)

## 安全なもの

どの言語拡張が一般に良く使われているかはあまりはっきりとは分かりませんが、これらの言語拡張は無難であり広範囲で使用しても安全である、と言っても差し支えありません。

* NoMonomorphismRestriction
* FlexibleContexts
* FlexibleInstances
* GeneralizedNewtypeDeriving
* GADTs
* FunctionalDependencies
* OverloadedStrings
* TypeSynonymInstances
* BangPatterns
* DeriveGeneric
* DeriveDataTypeable
* ScopedTypeVariables

## 危険なもの

GHC の型検査器は時々、ある種の問題を解決できない時に、言語拡張を有効にするように何気なく教えてくれます。それらの中にはこれが含まれます。

* DatatypeContexts
* OverlappingInstances
* IncoherentInstances
* ImpredicativeTypes

これらはたいてい設計上の欠陥を暗に示しています。GHC が使うように提案しているとしても、エラーをその場で解決するために頼るのはやめましょう！

## 型推論

Haskell での型推論は一般にはかなり正確ですが、問題を引き起こしがちな境界的なケースがいくつかあります。以下の 2 つの関数を考えてみましょう。

### 相互再帰の束縛グループ

```haskell
f x = const x g
g y = f 'A'
```

推論された型シグニチャは使用上は正しいですが、最も一般性のあるシグニチャにはなりません。GHC がモジュールを分析するとき、式の相互の依存関係を分析し、それらをグループ化し、相互に定義されているグループ全体に単一化をかけて、置換を行います。推論される型は、それ自体は可能な限りで最も一般性のある型にはならないかもしれないので、明示的にシグニチャを付けるのが良いでしょう。

```haskell
-- Inferred types
f :: Char -> Char
g :: t -> Char

-- Most general types
f :: a -> a
g :: a -> Char
```

### 多相再帰

```haskell
data Tree a = Leaf | Bin a (Tree (a, a))

size Leaf = 0
size (Bin _ t) = 1 + 2 * size t
```

この式の問題は、``size`` の推論される型変数 ``a`` が 2 つの型の可能性（``a`` と ``(a,a)``）を含むということです。これら 2 つの型は型検査器の出現検査を通過せず、誤った型が推論されてしまいます。

```haskell
    Occurs check: cannot construct the infinite type: t0 = (t0, t0)
    Expected type: Tree t0
      Actual type: Tree (t0, t0)
    In the first argument of `size', namely `t'
    In the second argument of `(*)', namely `size t'
    In the second argument of `(+)', namely `2 * size t'
```

型シグニチャを単に明示するだけで解決できます。多相再帰を使う型推論は、一般には決定不能です。

```haskell
size :: Tree a -> Int
size Leaf = 0
size (Bin _ t) = 1 + 2 * size t
```

参照：

* [Static Semantics of Function and Pattern Bindings](https://www.haskell.org/onlinereport/haskell2010/haskellch4.html#x10-880004.5)

## 単相性制限

型推論の境界的なケースで最も良く見かけるものは、世にも恐ろしい**単相性制限**として知られています。

モジュールのトップレベルの宣言が型の一般性を持つとき、単相性の制限を施すと、Prelude の ``Num`` のサブクラスを含む型を含むトップレベルの値（即ち、ラムダに入っていない式）は一般化されず、代わりに ``default`` で指定されたリスト（通常、`Integer` の次に `Double`）により順々に検査されていった単相型により実体化されます。

```haskell
-- Double は型推論器が推論した型です。
example1 :: Double
example1 = 3.14

-- ラムダがあるだけで、違う型が推論されるのです！
example2 :: Fractional a => t -> a
example2 _ = 3.14

default (Integer, Double)
```

GHC 7.8 の時点で、GHCi ではデフォルトで単相性制限は無効にされています。

```haskell
λ: set +t

λ: 3
3
it :: Num a => a

λ: default (Double)

λ: 3
3.0
it :: Num a => a
```

## セーフハスケル

誰もが結局は気づくことですが、GHC（Haskell 言語ではない）の実装の中には、型システムを破壊するのに使える関数がいくつかあり、それらは ``unsafe``［危険］という接頭辞が付いています。これらの関数は、式の健全性を手で証明できるが、型システムではその性質を表現できない時だけのために存在します。証明の義務を果たさずこれらの関数を使うと、あらゆる尺度で未定義の、想像を超えた痛みと苦しみを与える振る舞いが起こるので、**そうすることを避けることを強く推奨します**。最初に Haskell を学んでいる場合、これらの関数を使う正当な理由は全く持ってありません。以上。

```haskell
unsafeCoerce :: a -> b
unsafePerformIO :: IO a -> a
```

セーフハスケル言語拡張を使えば、危険な言語機能の使用を ``-XSafe`` を使って制限することができます。このフラグは、Safe とされたモジュールのみインポートするよう制限します。また、危険なコードを生み出すのに使える特定の言語拡張も（``-XTemplateHaskell`` など）禁止します。これらの拡張の直接のユースケースはセキュリティー監査の下にあります。

```haskell
{-# LANGUAGE Safe #-}
{-# LANGUAGE Trustworthy #-}
```

```haskell
{-# LANGUAGE Safe #-}

import Unsafe.Coerce
import System.IO.Unsafe

bad1 :: String
bad1 = unsafePerformIO getLine

bad2 :: a
bad2 = unsafeCoerce 3.14 ()
```

```haskell
Unsafe.Coerce: Can't be safely imported!
The module itself isn't safe.
```

参照：

* [Safe Haskell](https://ghc.haskell.org/trac/ghc/wiki/SafeHaskell)

## パターンガード

```haskell
{-# LANGUAGE PatternGuards #-}

combine env x y
   | Just a <- lookup x env
   , Just b <- lookup y env
   = Just $ a + b

   | otherwise = Nothing
```

## ビューパターン

```haskell
{-# LANGUAGE ViewPatterns #-}
{-# LANGUAGE NoMonomorphismRestriction #-}

import Safe

lookupDefault :: Eq a => a -> b -> [(a,b)] -> b
lookupDefault k _ (lookup k -> Just s) = s
lookupDefault _ d _ = d

headTup :: (a, [t]) -> [t]
headTup (headMay . snd -> Just n) = [n]
headTup _ = []

headNil :: [a] -> [a]
headNil (headMay -> Just x) = [x]
headNil _ = []
```

## その他の構文上の拡張

### タプルセクション

```haskell
{-# LANGUAGE TupleSections #-}

first :: a -> (a, Bool)
first = (,True)

second :: a -> (Bool, a)
second = (True,)
```

### 多分岐 if 式

```haskell
{-# LANGUAGE MultiWayIf #-}

operation x =
  if | x > 100   = 3
     | x > 10    = 2
     | x > 1     = 1
     | otherwise = 0
```

### ラムダケース

```haskell
{-# LANGUAGE LambdaCase #-}

data Exp a
  = Lam a (Exp a)
  | Var a
  | App (Exp a) (Exp a)

example :: Exp a -> a
example = \case
  Lam a b -> a
  Var a   -> a
  App a b -> example a
```

### パッケージからのインポート

```haskell
import qualified "mtl" Control.Monad.Error as Error
import qualified "mtl" Control.Monad.State as State
import qualified "mtl" Control.Monad.Reader as Reader
```

### レコードのワイルドカード

レコードのワイルドカードを使えば、レコードの名前を、暗黙のうちにレコードの対応するラベルを調べる変数として展開することができます。

```haskell
{-# LANGUAGE RecordWildCards #-}

data T = T { a :: Int , b :: Int }

f :: T -> Int
f (T {..} ) = a + b
```

## パターンシノニム

型検査器を書いているとすると、関数のシグニチャを入れ子にするのを簡単にするために、特別に ``TArr`` を入れることはよくあることでしょう。GHC は実際コア言語でそうしています。しかし技術的には ``(->)`` コンストラクタのより基本的な運用によって書くこともできます。

```haskell
data Type
  = TVar TVar
  | TCon TyCon
  | TApp Type Type
  | TArr Type Type
  deriving (Show, Eq, Ord)
```

パターンシノニムを使えば、矢印型に対するパターンマッチングを不便にしてしまうことなく、余計なコンストラクタを無くすことができます。

```haskell
{-# LANGUAGE PatternSynonyms #-}

pattern TArr t1 t2 = TApp (TApp (TCon "(->)") t1) t2
```

すると、矢印型に対するエリミネータ（除去子）とコンストラクタはとても自然に書けます。

```haskell
{-# LANGUAGE PatternSynonyms #-}

import Data.List (foldl1')

type Name  = String
type TVar  = String
type TyCon = String

data Type
  = TVar TVar
  | TCon TyCon
  | TApp Type Type
  deriving (Show, Eq, Ord)


pattern TArr t1 t2 = TApp (TApp (TCon "(->)") t1) t2

tapp :: TyCon -> [Type] -> Type
tapp tcon args = foldl TApp (TCon tcon) args

arr :: [Type] -> Type
arr ts = foldl1' (\t1 t2 -> tapp "(->)" [t1, t2]) ts

elimTArr :: Type -> [Type]
elimTArr (TArr (TArr t1 t2) t3) = t1 : t2 : elimTArr t3
elimTArr (TArr t1 t2) = t1 : elimTArr t2
elimTArr t = [t]

-- (->) a ((->) b a)
-- a -> b -> a
to :: Type
to = arr [TVar "a", TVar "b", TVar "a"]

from :: [Type]
from = elimTArr to
```
