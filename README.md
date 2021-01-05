<p align="center">
  <a href="https://ihp.digitallyinduced.com/" target="_blank">
          <img src="https://www.haskell.org/img/haskell-logo.svg" width="420"/>
  </a>
  <h1 align="center">Haskell UX: Let's Make Haskell's Error Messages Helpful</h1>
</p>



Often GHC Haskell error messages are not helpful or confusing. This is a huge barrier to entry for people just adopting haskell.

Let's collect these "bad" error messages here and suggest and improved error message.

## Error Message Design Goals

Good error messages should be **clear**, **actionable**, and **practical**. A great error message tells the user what went wrong and what the user needs to do to fix the problem. It should direct the user to the solution most likely needed.

Inspiration:
- https://mobile.twitter.com/jyothsnasrin/status/1037703436043603968

## Contribute

Please make a PR and add your own suggestions to this file :) It's best to add actual real world error messages from a real world use case.

--- 

# List of UX Improvements

<details>
  <summary>
        <strong>HUX1: Type level lists with only a single element need a ' in front of the list. Prepend a ' like '["email]' to get it working.</strong>
  </summary>
  
**Details:**

Given this code:

```haskell
action CreateUserAction = do
        let user = newRecord @User
        let password = param @Text "password"
        user
            |> set #passwordHash password
            |> fill @["email"]
            |> validateField #email isEmail
            |> validateField #passwordHash nonEmpty
            |> debug
            |> ifValid \case
            Left user ->
                render NewView {..}
            Right user -> do
                hashed <- hashPassword (get #passwordHash user)
                user
                    |> set #passwordHash hashed
                    |> createRecord

                setSuccessMessage "You have successfully registered"
```

GHC errors with:

```haskell
Web/Controller/Users.hs:16:23
    * Expected a type, but `"email"' has kind `Symbol'
    * In the type `["email"]'
      In the second argument of `(|>)', namely `fill @["email"]'
      In the first argument of `(|>)', namely
        `user |> set #passwordHash password |> fill @["email"]'
   |
16 |             |> fill @["email"]
   |                       ^^^^^^^
```

A better error message would be:

```haskell
Web/Controller/Users.hs:16:23
    * Type level lists with only a single element need a ' in front of the list. Prepend a ' like `'["email]' to get it working.
    * In the type `["email"]'
      In the second argument of `(|>)', namely `fill @["email"]'
      In the first argument of `(|>)', namely
        `user |> set #passwordHash password |> fill @["email"]'
   |
16 |             |> fill @["email"]
   |                       ^^^^^^^
```
  
</details>


Reported to GHC via https://gitlab.haskell.org/ghc/ghc/-/issues/19096

<details>
  <summary>
        <strong>HUX2: The let-expression is only indented 4 spaces from the do-statement, but it needs to be indented 8 spaces</strong>
  </summary>

**Details:**
Given this code:

```haskell
    action NewEventAction = do
        now <- getCurrentTime
        let event = newRecord @Event
            |> set #createdAt now -- THIS LINE NEEDS MORE INDENTATION
        render NewView { .. }
```

GHC errors with:

```haskell
Admin/Controller/Events.hs:26:9: error:
    The last statement in a 'do' block must be an expression
      let event = newRecord @Event
   |
26 |         let event = newRecord @Event
   |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^...
```

A better error message would be:

```haskell
Admin/Controller/Events.hs:26:9: error:
    The let-expression is only indented 4 spaces from the do-statement, but it needs to be indented 8 spaces
      '|> set #createdAt now'
   |
26 |         let event = newRecord @Event
27 |             |> set #createdAt now
   |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^...
```

</details>

Reported to GHC via https://gitlab.haskell.org/ghc/ghc/-/issues/19097


<details>
  <summary>
        <strong>HUX3: The call to `putContext` requires an implicit parameter `?context::ControllerContext` to be available. Change the type signature to this: fetchCategories :: (?modelContext::ModelContext, ?context :: ControllerContext) => IO ()</strong>
  </summary>

**Details:**
Given this code:
```haskell
fetchCategories :: (?modelContext :: ModelContext) => IO ()
fetchCategories = do
  categories :: [Category] <- query @Category |> fetch
  putContext categories
```

GHC errors with:
```haskell
Web/FrontController.hs:18:3: error:
    * Could not deduce: ?context::ControllerContext
        arising from a use of `putContext'
      from the context: ?modelContext::ModelContext
        bound by the type signature for:
                   fetchCategories :: (?modelContext::ModelContext) => IO ()
        at Web/FrontController.hs:15:1-59
    * In a stmt of a 'do' block: putContext categories
      In the expression:
        do categories :: [Category] <- query @Category |> fetch
           putContext categories
      In an equation for `fetchCategories':
          fetchCategories
            = do categories :: [Category] <- query @Category |> fetch
                 putContext categories
   |
18 |   putContext categories
```

A better error message would be:

```haskell
Web/FrontController.hs:18:3: error:
    * The call to `putContext` requires an implicit parameter `?context::ControllerContext` to be available. Change the type signature to this: fetchCategories :: (?modelContext::ModelContext, ?context :: ControllerContext) => IO ()
        at Web/FrontController.hs:15:1-59
    * In a stmt of a 'do' block: putContext categories
      In the expression:
        do categories :: [Category] <- query @Category |> fetch
           putContext categories
      In an equation for `fetchCategories':
          fetchCategories
            = do categories :: [Category] <- query @Category |> fetch
                 putContext categories
   |
18 |   putContext categories
```
</details>

Reported to GHC via https://gitlab.haskell.org/ghc/ghc/-/issues/19098



<details>
  <summary>
        <strong>HUX4: Perhaps you meant a :: Maybe Int ?</strong>
  </summary>

**Details:**
Given this code:

```haskell
a :: Just Int
a = Just 5
```

GHC errors with:

```haskell
Not in scope: type constructor or class 'Just'A data constructor of that name is in scope; did you mean DataKinds?
```

A better error message would be:

```haskell
Perhaps you meant a :: Maybe Int?
Not in scope: type constructor or class 'Just'
A data constructor of that name is in scope; did you mean DataKinds?
```

</details>

Suggested on reddit: https://www.reddit.com/r/haskell/comments/kgvdon/improving_haskell_ghc_error_messages/gghjajf/?utm_source=reddit&utm_medium=web2x&context=3




<details>
  <summary>
        <strong>HUX5: Perhaps you want to use `pure`?</strong>
  </summary>

**Details:**
Given this code:

```haskell
initModelContext :: FrameworkConfig -> IO ModelContext
initModelContext FrameworkConfig { environment, dbPoolIdleTime, dbPoolMaxConnections, databaseUrl } = do
    let isDevelopment = environment == Env.Development
    modelContext <- (\modelContext -> modelContext { queryDebuggingEnabled = isDevelopment }) <$> createModelContext dbPoolIdleTime dbPoolMaxConnections databaseUrl
    modelContext
```

GHC errors with:

```haskell
IHP/Server.hs:133:5: error:
    • Couldn't match expected type ‘IO ModelContext’
                  with actual type ‘ModelContext’
    • In a stmt of a 'do' block: modelContext
      In the expression:
        do let isDevelopment = environment == Env.Development
           modelContext <- (\ modelContext
                              -> modelContext {queryDebuggingEnabled = isDevelopment})
                             <$>
                               createModelContext dbPoolIdleTime dbPoolMaxConnections databaseUrl
           modelContext
      In an equation for ‘initModelContext’:
          initModelContext
            FrameworkConfig {environment, dbPoolIdleTime, dbPoolMaxConnections,
                             databaseUrl}
            = do let isDevelopment = ...
                 modelContext <- (\ modelContext
                                    -> modelContext {queryDebuggingEnabled = isDevelopment})
                                   <$>
                                     createModelContext
                                       dbPoolIdleTime dbPoolMaxConnections databaseUrl
                 modelContext
    |
133 |     modelContext
```

A better error message would be:

```haskell
IHP/Server.hs:133:5: error:
    • Perhaps you meant `pure modelContext`?
    
    Couldn't match expected type ‘IO ModelContext’
                  with actual type ‘ModelContext’
    • In a stmt of a 'do' block: modelContext
      In the expression:
        do let isDevelopment = environment == Env.Development
           modelContext <- (\ modelContext
                              -> modelContext {queryDebuggingEnabled = isDevelopment})
                             <$>
                               createModelContext dbPoolIdleTime dbPoolMaxConnections databaseUrl
           modelContext
      In an equation for ‘initModelContext’:
          initModelContext
            FrameworkConfig {environment, dbPoolIdleTime, dbPoolMaxConnections,
                             databaseUrl}
            = do let isDevelopment = ...
                 modelContext <- (\ modelContext
                                    -> modelContext {queryDebuggingEnabled = isDevelopment})
                                   <$>
                                     createModelContext
                                       dbPoolIdleTime dbPoolMaxConnections databaseUrl
                 modelContext
    |
133 |     modelContext
```

</details>


# Goodies that are being worked on at GHC


## Better parsing error

Great work by @guibou
PR: https://gitlab.haskell.org/ghc/ghc/-/merge_requests/4711


> Was given the following error message when parsed with current GHC:

```
[1 of 1] Compiling Main             ( ParseErrorTest.hs, ParseErrorTest.o )

ParseErrorTest.hs:6:1: error:
    parse error (possibly incorrect indentation or mismatched brackets)
  |
6 | data Chicken = CotCotCot
  | ^
```

> With this pull request, the error message is as following:

```
[1 of 1] Compiling Main             ( ParseErrorTest.hs, ParseErrorTest.o )

ParseErrorTest.hs:1:17: error:
    Parse error with context: Parsing error: please close this list.
  |
1 | foo x y z = bar [(x, y, z),
  |                 ^

ParseErrorTest.hs:2:13: error:
    Parse error with context: Parsing error: please close this tuple brace.
  |
2 |             (y, x, z,
  |             ^

ParseErrorTest.hs:8:21: error:
    Parse error with context: Parsing error: please close this brace.
  |
8 | bar a b c d = print ((a, b, c, d) + "hello"
  |                     ^

ParseErrorTest.hs:14:3: error:
    Parse error with context: Parsing error: let/in clause without body. Please add a body.
   |
14 |   in
   |   ^^
```

