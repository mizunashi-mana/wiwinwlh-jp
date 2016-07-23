# グラフィックス

## diagrams

Diagrams is a a parser combinator library for generating vector images to SVG and a variety of other formats.

```haskell
import Diagrams.Prelude
import Diagrams.Backend.SVG.CmdLine

sierpinksi :: Int -> Diagram SVG R2
sierpinksi 1 = eqTriangle 1
sierpinksi n =
      s
     ===
  (s ||| s) # centerX
  where
    s = sierpinksi (n - 1)

example :: Diagram SVG R2
example = sierpinksi 5 # fc black

main :: IO ()
main = defaultMain example
```

```bash
$ runhaskell diagram1.hs -w 256 -h 256 -o diagram1.svg
```

![](https://raw.githubusercontent.com/sdiehl/wiwinwlh/master/img/diagram1.png)

See: [Diagrams Quick Start Tutorial](http://projects.haskell.org/diagrams/doc/quickstart.html)

## gloss
