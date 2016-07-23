# ネットワークとWebプログラミング

## HTTP

Haskell has a variety of HTTP request and processing libraries.

```haskell
{-# LANGUAGE OverloadedStrings #-}

import Network.HTTP.Types
import Network.HTTP.Client
import Control.Applicative
import Control.Concurrent.Async

type URL = String

get :: Manager -> URL -> IO Int
get m url = do
  req <- parseUrl url
  statusCode <$> responseStatus <$> httpNoBody req m

single :: IO Int
single = do
  withManager defaultManagerSettings $ \m -> do
    get m "http://haskell.org"

parallel :: IO [Int]
parallel = do
  withManager defaultManagerSettings $ \m -> do
    -- Fetch w3.org 10 times concurrently
    let urls = replicate 10 "http://www.w3.org"
    mapConcurrently (get m) urls

main :: IO ()
main = do
  print =<< single
  print =<< parallel
```

## Warp

Warp is a particularly efficient web server, it's the backed request engine
behind several of popular Haskell web frameworks. The internals have been finely
tuned to utilize Haskell's concurrent runtime and is capable of handling a great
deal of concurrent requests.

```haskell
{-# LANGUAGE OverloadedStrings #-}

import Network.Wai
import Network.Wai.Handler.Warp (run)
import Network.HTTP.Types

app :: Application
app req = return $ responseLBS status200 [] "Engage!"

main :: IO ()
main = run 8000 app
```

See: [Warp](http://aosabook.org/en/posa/warp.html)

## Scotty

Continuing with our trek through web libraries, Scotty is a web microframework
similar in principle to Flask in Python or Sinatra in Ruby.

```haskell
{-# LANGUAGE OverloadedStrings #-}

import Web.Scotty

import qualified Text.Blaze.Html5 as H
import Text.Blaze.Html5 (toHtml, Html)
import Text.Blaze.Html.Renderer.Text (renderHtml)

greet :: String -> Html
greet user = H.html $ do
  H.head $
    H.title "Welcome!"
  H.body $ do
    H.h1 "Greetings!"
    H.p ("Hello " >> toHtml user >> "!")

app = do
  get "/" $
    text "Home Page"

  get "/greet/:name" $ do
    name <- param "name"
    html $ renderHtml (greet name)

main :: IO ()
main = scotty 8000 app
```

Of importance to note is the Blaze library used here overloads do-notation but
is not itself a proper monad so the various laws and invariants that normally
apply for monads may break down or fail with error terms.

See: [Making a Website with Haskell](http://adit.io/posts/2013-04-15-making-a-website-with-haskell.html)
