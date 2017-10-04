---
title: "Haskell für Fortgeschrittene - Workshop"
author: Carsten König
date: Oktober 2017
geometry: margin=2cm
output: pdf_document
papersize: a4
---

# Functoren

## Intuition:

- *Kontainer* mit einem oder mehr Werte
- *Berechnung* die einen Wert produziert
- `fmap` bringt eine Funktion, die Werte transformiert in den Funktor

## Typklasse

```haskell
:i Functor
class Functor (f :: * -> *) where
    fmap :: (a -> b) -> f a -> f b
    (<$) :: a -> f b -> f a
    {-# MINIMAL fmap #-}
```

`<$>` ist Operator für `fmap`

## Beispiele

- Listen
- `Maybe`
- `IO`
- `Either a`
- `((->) r)`
- `((,) a)`

```haskel
> fmap succ [1..5]
[2,3,4,5,6]

> fmap succ (Just 5)
Just 6

> fmap succ Nothing
Nothing

> fmap (++ "!") getLine
Hello
"Hello!"

> fmap succ (Right 5)
Right 6

> fmap succ (Left "argh")
Left "argh"

> fmap succ (*5) 2
11

> fmap succ ("Hallo", 41)
("Hallo",42)
```

## Extension
mit `{-# LANGUAGE DeriveFunctor #-}` kann fast jeder ADT zu einem Funktor gemacht werden (mittels `deriving Functor`)

## Übungen

- Functor Instanz für `data Pair a = Pair a a`
- Functor Instanz für `data ITree a = Leaf (Int -> a) | Node [ITree a]`
- Gesucht Typ mit Kind `* -> *`, der kein Functor ist

## Functor sind sind *komposierbar*

```haskell
newtype Compose f g a = Compose (f (g a))
  deriving Show


instance (Functor f, Functor g) => Functor (Compose f g) where
  fmap f (Compose x) = Compose $ fmap (fmap f) x


type MaybeList a = Compose [] Maybe a

example :: MaybeList String
example = fmap show $ Compose [Just 5, Nothing, Just 4]
```

## Laws

- `fmap id = id`
- `fmap (g . h) = (fmap g) . (fmap h)`


## heftiges Beispiel Recursion Schemes

```haskell
{-# LANGUAGE DeriveFunctor #-}

module RecSchemes where


data ExprF a
  = ValF Int
  | AddF a a
  | MulF a a
  deriving (Show, Eq, Functor)


type Expr = Fix ExprF


data Fix f = Fix { unFix :: f (Fix f) }


val :: Int -> Expr
val n = Fix $ ValF n

add :: Expr -> Expr -> Expr
add a b = Fix $ AddF a b

mul :: Expr -> Expr -> Expr
mul a b = Fix $ MulF a b


type Algebra f a = f a -> a


evalAlg :: Algebra ExprF Int
evalAlg (ValF n) = n
evalAlg (AddF a b) = a + b
evalAlg (MulF a b) = a * b


eval :: Expr -> Int
eval = cata evalAlg


cata :: Functor f => Algebra f a -> Fix f -> a
cata alg =
  alg . fmap (cata alg) . unFix


showAlg :: Algebra ExprF String
showAlg (ValF n) = show n
showAlg (AddF a b) = "(" ++ a ++ " + " ++ b ++ ")"
showAlg (MulF a b) = a ++ " * " ++ b


showExpr :: Expr -> String
showExpr = cata showAlg
```