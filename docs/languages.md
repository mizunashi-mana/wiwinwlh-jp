# 言語

## unbound

Several libraries exist to mechanize the process of writing name capture and
substitution, since it is largely mechanical. Probably the most robust is the
``unbound`` library.  For example we can implement the infer function for a
small Hindley-Milner system over a simple typed lambda calculus without having
to write the name capture and substitution mechanics ourselves.

```haskell
{-# LANGUAGE TemplateHaskell #-}
{-# LANGUAGE FlexibleInstances #-}
{-# LANGUAGE UndecidableInstances #-}
{-# LANGUAGE MultiParamTypeClasses #-}
{-# LANGUAGE OverloadedStrings #-}

module Infer where

import Data.String
import Data.Map (Map)
import Control.Monad.Error
import qualified Data.Map as Map

import qualified Unbound.LocallyNameless as NL
import Unbound.LocallyNameless hiding (Subst, compose)

data Type
  = TVar (Name Type)
  | TArr Type Type
  deriving (Show)

data Expr
  = Var (Name Expr)
  | Lam (Bind (Name Expr) Expr)
  | App Expr Expr
  | Let (Bind (Name Expr) Expr)
  deriving (Show)

$(derive [''Type, ''Expr])


instance IsString Expr where
    fromString = Var . fromString
instance IsString Type where
    fromString = TVar . fromString
instance IsString (Name Expr) where
    fromString = string2Name
instance IsString (Name Type) where
    fromString = string2Name

instance Eq Type where
    (==) = eqType

eqType :: Type -> Type -> Bool
eqType (TVar v1) (TVar v2) = v1 == v2
eqType _ _ = False

uvar :: String -> Expr
uvar x = Var (s2n x)

tvar :: String -> Type
tvar x = TVar (s2n x)

instance Alpha Type
instance Alpha Expr

instance NL.Subst Type Type where
  isvar (TVar v) = Just (SubstName v)
  isvar _ = Nothing

instance NL.Subst Expr Expr where
  isvar (Var v) = Just (SubstName v)
  isvar _ = Nothing

instance NL.Subst Expr Type where


data TypeError
  = UnboundVariable (Name Expr)
  | GenericTypeError
  deriving (Show)

instance Error TypeError where
  noMsg = GenericTypeError


type Env = Map (Name Expr) Type
type Constraint = (Type, Type)
type Infer = ErrorT TypeError FreshM

empty :: Env
empty = Map.empty

freshtv :: Infer Type
freshtv = do
  x <- fresh "_t"
  return $ TVar x

infer :: Env -> Expr -> Infer (Type, [Constraint])
infer env expr = case expr  of

  Lam b -> do
    (n,e) <- unbind b
    tv <- freshtv
    let env' = Map.insert n tv env
    (t, cs) <- infer env' e
    return (TArr tv t, cs)

  App e1 e2 -> do
     (t1, cs1) <- infer env e1
     (t2, cs2) <- infer env e2
     tv <- freshtv
     return (tv, (t1, TArr t2 tv) : cs1 ++ cs2)

  Var n -> do
     case Map.lookup n env of
        Nothing -> throwError $ UnboundVariable n
        Just t  -> return (t, [])

  Let b -> do
     (n, e) <- unbind b
     (tBody, csBody) <- infer env e
     let env' = Map.insert n tBody env
     (t, cs) <- infer env' e
     return (t, cs ++ csBody)
```

## unboundジェネリクス

Recently unbound was ported to use GHC.Generics instead of Template Haskell. The
API is effectively the same, so for example a simple lambda calculus could be
written as:

```haskell
{-# LANGUAGE DeriveGeneric #-}
{-# LANGUAGE DeriveDataTypeable #-}
{-# LANGUAGE FlexibleInstances #-}
{-# LANGUAGE FlexibleContexts #-}
{-# LANGUAGE MultiParamTypeClasses #-}
{-# LANGUAGE ScopedTypeVariables #-}

module LC where

import Unbound.Generics.LocallyNameless
import Unbound.Generics.LocallyNameless.Internal.Fold (toListOf)

import GHC.Generics

import Data.Typeable (Typeable)
import Data.Set as S

import Control.Monad.Reader (Reader, runReader)

data Exp
  = Var (Name Exp)
  | Lam (Bind (Name Exp) Exp)
  | App Exp Exp
  deriving (Show, Generic, Typeable)

instance Alpha Exp

instance Subst Exp Exp where
  isvar (Var x) = Just (SubstName x)
  isvar _       = Nothing

fvSet :: (Alpha a, Typeable b) => a -> S.Set (Name b)
fvSet = S.fromList . toListOf fv

type M a = FreshM a

(=~) :: Exp -> Exp -> M Bool
e1 =~ e2 | e1 `aeq` e2 = return True
e1 =~ e2 = do
    e1' <- red e1
    e2' <- red e2
    if e1' `aeq` e1 && e2' `aeq` e2
      then return False
      else e1' =~ e2'

-- Reduction
red :: Exp -> M Exp
red (App e1 e2) = do
  e1' <- red e1
  e2' <- red e2
  case e1' of
    Lam bnd -> do
        (x, e1'') <- unbind bnd
        return $ subst x e2' e1''
    otherwise -> return $ App e1' e2'
red (Lam bnd) = do
   (x, e) <- unbind bnd
   e' <- red e
   case e of
     App e1 (Var y) | y == x && x `S.notMember` fvSet e1 -> return e1
     otherwise -> return (Lam (bind x e'))
red (Var x) = return $ (Var x)


x :: Name Exp
x = string2Name "x"

y :: Name Exp
y = string2Name "y"

z :: Name Exp
z = string2Name "z"

s :: Name Exp
s = string2Name "s"

lam :: Name Exp -> Exp -> Exp
lam x y = Lam (bind x y)

zero  = lam s (lam z (Var z))
one   = lam s (lam z (App (Var s) (Var z)))
two   = lam s (lam z (App (Var s) (App (Var s) (Var z))))
three = lam s (lam z (App (Var s) (App (Var s) (App (Var s) (Var z)))))

plus = lam x (lam y (lam s (lam z (App (App (Var x) (Var s)) (App (App (Var y) (Var s)) (Var z))))))

true = lam x (lam y (Var x))
false = lam x (lam y (Var y))
if_ x y z = (App (App x y) z)

main :: IO ()
main = do
  print $ lam x (Var x) `aeq` lam y (Var y)
  print $ not (lam x (Var y) `aeq` lam x (Var x))
  print $ lam x (App (lam y (Var x)) (lam y (Var y))) =~ (lam y (Var y))
  print $ lam x (App (Var y) (Var x)) =~ Var y
  print $ if_ true (Var x) (Var y) =~ Var x
  print $ if_ false (Var x) (Var y) =~ Var y
  print $ App (App plus one) two =~ three
```

See:

* [unbound-generics](https://github.com/lambdageek/unbound-generics)

## LLVM

LLVM is a library for generating machine code. The llvm-general bindings provide a way to model, compile and
execute LLVM bytecode from within the Haskell runtime.

See:

* [Implementing a JIT Compiled Language with Haskell and LLVM](http://www.stephendiehl.com/llvm/)

## 整形表示器のコンビネータ

Pretty printer combinators compose logic to print strings.

              Combinators
-----------   ------------
``<>``        Concatenation
``<+>``       Spaced concatenation
``char``      Renders a character as a ``Doc``
``text``      Renders a string as a ``Doc``

```haskell
{-# LANGUAGE FlexibleInstances #-}

import Text.PrettyPrint
import Text.Show.Pretty (ppShow)

parensIf ::  Bool -> Doc -> Doc
parensIf True = parens
parensIf False = id

type Name = String

data Expr
  = Var String
  | Lit Ground
  | App Expr Expr
  | Lam Name Expr
  deriving (Eq, Show)

data Ground
  = LInt Int
  | LBool Bool
  deriving (Show, Eq, Ord)


class Pretty p where
  ppr :: Int -> p -> Doc

instance Pretty String where
  ppr _ x = text x

instance Pretty Expr where
  ppr _ (Var x)         = text x
  ppr _ (Lit (LInt a))  = text (show a)
  ppr _ (Lit (LBool b)) = text (show b)

  ppr p e@(App _ _) =
    let (f, xs) = viewApp e in
    let args = sep $ map (ppr (p+1)) xs in
    parensIf (p>0) $ ppr p f <+> args

  ppr p e@(Lam _ _) =
    let body = ppr (p+1) (viewBody e) in
    let vars = map (ppr 0) (viewVars e) in
    parensIf (p>0) $ char '\\' <> hsep vars <+> text "." <+> body

viewVars :: Expr -> [Name]
viewVars (Lam n a) = n : viewVars a
viewVars _ = []

viewBody :: Expr -> Expr
viewBody (Lam _ a) = viewBody a
viewBody x = x

viewApp :: Expr -> (Expr, [Expr])
viewApp (App e1 e2) = go e1 [e2]
  where
    go (App a b) xs = go a (b : xs)
    go f xs = (f, xs)

ppexpr :: Expr -> String
ppexpr = render . ppr 0


s, k, example :: Expr
s = Lam "f" (Lam "g" (Lam "x" (App (Var "f") (App (Var "g") (Var "x")))))
k = Lam "x" (Lam "y" (Var "x"))
example = App s k

main :: IO ()
main = do
  putStrLn $ ppexpr s
  putStrLn $ ppShow example
```

The pretty printed form of the ``k`` combinator:

```haskell
\f g x . (f (g x))
```

The ``Text.Show.Pretty`` library can be used to pretty print nested data structures in a more human readable
form for any type that implements ``Show``.  For example a dump of the structure for the AST of SK combinator
with ``ppShow``.

```haskell
App
  (Lam
     "f" (Lam "g" (Lam "x" (App (Var "f") (App (Var "g") (Var "x"))))))
  (Lam "x" (Lam "y" (Var "x")))
```

Adding the following to your ghci.conf can be useful for working with deeply nested structures interactively.

```haskell
import Text.Show.Pretty (ppShow)
let pprint x = putStrLn $ ppShow x
```

See: [The Design of a Pretty-printing Library](http://belle.sourceforge.net/doc/hughes95design.pdf)

## haskeline

Haskeline is cross-platform readline support which plays nice with GHCi as well.

```haskell
runInputT :: Settings IO -> InputT IO a -> IO a
getInputLine :: String -> InputT IO (Maybe String)
```

```haskell
import Control.Monad.Trans
import System.Console.Haskeline

type Repl a = InputT IO a

process :: String -> IO ()
process = putStrLn

repl :: Repl ()
repl = do
  minput <- getInputLine "Repl> "
  case minput of
    Nothing -> outputStrLn "Goodbye."
    Just input -> (liftIO $ process input) >> repl

main :: IO ()
main = runInputT defaultSettings repl
```
