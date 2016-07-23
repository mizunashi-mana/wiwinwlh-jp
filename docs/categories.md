# 圏

<!-- markdownlint-disable MD033 -->

## はじめに

Alas we come to the topic of category theory. Some might say all discussion of
Haskell eventually leads here at one point or another.

Nevertheless the overall importance of category theory in the context of Haskell
has been somewhat overstated and unfortunately mystified to some extent. The
reality is that amount of category theory which is directly applicable to
Haskell roughly amounts to a subset of the first chapter of any undergraduate
text.

## 代数的関係

Grossly speaking category theory is not terribly important to Haskell
programming, and although some libraries derive some inspiration from the
subject most do not. What is more important is a general understanding of
equational reasoning and a familiarity with various algebraic relations.

Certain relations show up so frequently we typically refer to their properties
by name ( often drawn from an equivalent abstract algebra concept ). Consider a
binary operation ``a `op` b``.

**Associativity**

```haskell
a `op` (b `op` c) = (a `op` b) `op` c
```

**Commutativity**

```haskell
a `op` b = b `op` a
```

**Units**

```haskell
a `op` e = a
e `op` a = a
```

**Inversion**

```haskell
(inv a) `op` a = e
a `op` (inv a) = e
```

**Zeros**

```haskell
a `op` e = e
e `op` a = e
```

**Linearity**

```haskell
f (x `op` y) = f x `op` f y
```

**Idempotency**

```haskell
f (f x) = f x
```

**Distributivity**

```haskell
a `f` (b `g` c) = (a `f` b) `g` (a `f` c)
(b `g` c) `f` a = (b `f` a) `g` (c `f` a)
```

**Anticommutativity**

```haskell
a `op` b = inv (b `op` a)
```

And of course combinations of these properties over multiple functions gives
rise to higher order systems of relations that occur over and over again
throughout functional programming, and once we recognize them we can abstract
over them. For instance a monoid is a combination of a unit and a single
associative operation.

## 圏の基本的な構造

The most basic structure is a category which is an algebraic structure of
objects (``Obj``) and morphisms (``Hom``) with the structure that morphisms
compose associatively and the existence of an identity morphism for each object.

With kind polymorphism enabled we can write down the general category
parameterized by a type variable "c" for category, and the instance ``Hask`` the
category of Haskell types with functions between types as morphisms.

```haskell
{-# LANGUAGE PolyKinds #-}
{-# LANGUAGE TypeOperators #-}
{-# LANGUAGE TypeSynonymInstances #-}

import Prelude hiding ((.), id)

-- Morphisms
type (a ~> b) c = c a b

class Category (c :: k -> k -> *) where
  id :: (a ~> a) c
  (.) :: (y ~> z) c -> (x ~> y) c -> (x ~> z) c

type Hask = (->)

instance Category Hask where
  id x = x
  (f . g) x = f (g x)
```

## 同型

Two objects of a category are said to be isomorphic if we can construct a
morphism with 2-sided inverse that takes the structure of an object to another
form and back to itself when inverted.

```haskell
f  :: a -> b
f' :: b -> a
```

Such that:

```haskell
f . f' = id
f'. f  = id
```

For example the types ``Either () a`` and ``Maybe a`` are isomorphic.

```haskell
{-# LANGUAGE ExplicitForAll #-}

data Iso a b = Iso { to :: a -> b, from :: b -> a }

f :: forall a. Maybe a -> Either () a
f (Just a) = Right a
f Nothing  = Left ()

f' :: forall a. Either () a -> Maybe a
f' (Left _)  = Nothing
f' (Right a) = Just a

iso :: Iso (Maybe a) (Either () a)
iso = Iso f f'

data V = V deriving Eq

ex1 = f  (f' (Right V)) == Right V
ex2 = f' (f  (Just V))  == Just V
```

```haskell
data Iso a b = Iso { to :: a -> b, from :: b -> a }

instance Category Iso where
  id = Iso id id
  (Iso f f') . (Iso g g') = Iso (f . g) (g' . f')
```

## 双対性

One of the central ideas is the notion of duality, that reversing some internal
structure yields a new structure with a "mirror" set of theorems. The dual of a
category reverse the direction of the morphisms forming the category
C<sup>Op</sup>.

```haskell
import Control.Category
import Prelude hiding ((.), id)

newtype Op a b = Op (b -> a)

instance Category Op where
  id = Op id
  (Op f) . (Op g) = Op (g . f)
```

See:

* [Duality for Haskellers](http://blog.ezyang.com/2012/10/duality-for-haskellers/)

## 関手

Functors are mappings between the objects and morphisms of categories that
preserve identities and composition.

```haskell
{-# LANGUAGE MultiParamTypeClasses #-}
{-# LANGUAGE TypeSynonymInstances #-}

import Prelude hiding (Functor, fmap, id)

class (Category c, Category d) => Functor c d t where
  fmap :: c a b -> d (t a) (t b)

type Hask = (->)

instance Category Hask where
  id x = x
  (f . g) x = f (g x)

instance Functor Hask Hask [] where
  fmap f [] = []
  fmap f (x:xs) = f x : (fmap f xs)
```

```haskell
fmap id ≡ id
fmap (a . b) ≡ (fmap a) . (fmap b)
```

## 自然変換

Natural transformations are mappings between functors that are invariant under
interchange of morphism composition order.

```haskell
type Nat f g = forall a. f a -> g a
```

Such that for a natural transformation ``h`` we have:

```haskell
fmap f . h ≡ h . fmap f
```

The simplest example is between (f = List) and (g = Maybe) types.

```haskell
headMay :: forall a. [a] -> Maybe a
headMay []     = Nothing
headMay (x:xs) = Just x
```

Regardless of how we chase ``safeHead``, we end up with the same result.

```haskell
fmap f (headMay xs) ≡ headMay (fmap f xs)
```

```haskell
fmap f (headMay [])
= fmap f Nothing
= Nothing

headMay (fmap f [])
= headMay []
= Nothing
```

```haskell
fmap f (headMay (x:xs))
= fmap f (Just x)
= Just (f x)

headMay (fmap f (x:xs))
= headMay [f x]
= Just (f x)
```

Or consider the Functor ``(->)``.

```haskell
f :: (Functor t)
  => (->) a b
  -> (->) (t a) (t b)
f = fmap

g :: (b -> c)
  -> (->) a b
  -> (->) a c
g = (.)

c :: (Functor t)
  => (b -> c)
  -> (->) (t a) (t b)
  -> (->) (t a) (t c)
c = f . g
```

```haskell
f . g x = c x . g
```

A lot of the expressive power of Haskell types comes from the interesting fact
that, with a few caveats, polymorphic Haskell functions are natural
transformations.

See: [You Could Have Defined Natural Transformations](http://blog.sigfpe.com/2008/05/you-could-have-defined-natural.html)

## 米田の補題

The Yoneda lemma is an elementary, but deep result in Category theory. The
Yoneda lemma states that for any functor ``F``, the types ``F a`` and ``∀ b. (a -> b) -> F b`` are isomorphic.

```haskell
{-# LANGUAGE RankNTypes #-}

embed :: Functor f => f a -> (forall b . (a -> b) -> f b)
embed x f = fmap f x

unembed :: Functor f => (forall b . (a -> b) -> f b) -> f a
unembed f = f id
```

So that we have:

```haskell
embed . unembed ≡ id
unembed . embed ≡ id
```

The most broad hand-wavy statement of the theorem is that an object in a
category can be represented by the set of morphisms into it, and that the
information about these morphisms alone sufficiently determines all properties
of the object itself.

In terms of Haskell types, given a fixed type ``a`` and a functor ``f``, if we
have some a higher order polymorphic function ``g`` that when given a function
of type ``a -> b`` yields ``f b`` then the behavior ``g`` is entirely determined
by ``a -> b`` and the behavior of ``g`` can written purely in terms of ``f a``.

See:

* [Reverse Engineering Machines with the Yoneda Lemma](http://blog.sigfpe.com/2006/11/yoneda-lemma.html)

## クライスリ圏

Kleisli composition (i.e. Kleisli Fish) is defined to be:

```haskell
(>=>) :: Monad m => (a -> m b) -> (b -> m c) -> a -> m c
f >=> g ≡ \x -> f x >>= g

(<=<) :: Monad m => (b -> m c) -> (a -> m b) -> a -> m c
(<=<) = flip (>=>)
```

The monad laws stated in terms of the Kleisli category of a monad ``m`` are
stated much more symmetrically as one associativity law and two identity laws.

```haskell
(f >=> g) >=> h ≡ f >=> (g >=> h)
return >=> f ≡ f
f >=> return ≡  f
```

Stated simply that the monad laws above are just the category laws in the
Kleisli category.

```haskell
{-# LANGUAGE TypeOperators #-}
{-# LANGUAGE ExplicitForAll #-}

import Control.Monad
import Control.Category
import Prelude hiding ((.))

-- Kleisli category
newtype Kleisli m a b = K (a -> m b)

-- Kleisli morphisms ( a -> m b )
type (a :~> b) m = Kleisli m a b

instance Monad m => Category (Kleisli m) where
  id            = K return
  (K f) . (K g) = K (f <=< g)


just :: (a :~> a) Maybe
just = K Just

left :: forall a b. (a :~> b) Maybe -> (a :~> b) Maybe
left f = just . f

right :: forall a b. (a :~> b) Maybe -> (a :~> b) Maybe
right f = f . just
```

For example, ``Just`` is just an identity morphism in the Kleisli category of
the ``Maybe`` monad.

```haskell
Just >=> f ≡ f
f >=> Just ≡ f
```

## 参考文献

* [Category Theory, Awodey](http://www.amazon.com/Category-Theory-Oxford-Logic-Guides/dp/0199237182)
* [Category Theory Foundations](https://www.youtube.com/watch?v=ZKmodCApZwk)
* [The Catsters](http://www.youtube.com/user/TheCatsters)
