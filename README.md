# Haskell UX
Let's make haskells error messages helpful.

Often GHC Haskell error messages are not helpful or confusing. This is a huge barrier to entry for people just adopting haskell.

Let's collect these "bad" error messages here and suggest and improved error message.

## Error Message Design Goals

Good error messages should be **clear**, **actionable**, and **practical**. A great error message tells the user what went wrong and what the user needs to do to fix the problem. It should direct the user to the solution most likely needed.

## Contribute

Please make a PR and add your own suggestions to this file :)

--- 

# Haskell UX Improvement: 1

**Current:** 
```haskell
Expected a type, but "email"' has kind Symbol
```
**Better:**
```haskell
Type level lists with only a single element need a ' in front of the list. Prepend a ' like '["email]' to get it working.
```

## Details

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


**Reported to GHC via https://gitlab.haskell.org/ghc/ghc/-/issues/19096**



# Haskell UX Improvement: 2

**Current:** 
```haskell
The last statement in a 'do' block must be an expression
```
**Better:**
```haskell
The let-expression is only indented 4 spaces from the do-statement, but it needs to be indented 8 spaces
```

## Details
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

**Reported to GHC via https://gitlab.haskell.org/ghc/ghc/-/issues/19097**



# Haskell UX Improvement: 3

**Current:** 
```haskell
Could not deduce: ?context::ControllerContext arising from a use of putContext' from the context: ?modelContext::ModelContext bound by the type signature for:
```
**Better:**
```haskell
The call to `putContext` requires an implicit parameter `?context::ControllerContext` to be available. Change the type signature to this: fetchCategories :: (?modelContext::ModelContext, ?context :: ControllerContext) => IO ()
```

## Details
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

**Reported to GHC via https://gitlab.haskell.org/ghc/ghc/-/issues/19098**
