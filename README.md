=Tackling the Awkward Squad

== 21 Sept 2011

Moderator: Ben
Notetaker: Nick Canzoneri

===Opening discussion up with questions about the paper

Q: why use monads when you can use Haskell for pure functional stuff and interface for other stuff
Sub-question: Difficulty of interfacing?

A: Still not that hard to write imperative code and foreign function interface (FFI) is still monadic

Values from IO are still tainted with IO. Anything returned from the IO Monad is still typed with IO 

Imperative programming in Haskell: actions are first class values. Building actions up (using them as a first class function) doesn't have side effects, can build them up and only on execution get  side effects

The programmer can use actions to make it easier to constrain scope of side effects. For example,  writing compiler - transform one thing to another, need to choose fresh names without reuse, choosing that name is a side effect.

Q: Difference between action and a first class function (like in Scala or other language)?
A: Action is a particular typed function - returns a value in a monad

You can tell a lot about the program from the types, even monadic types. IOMonad and STM Monad provides information to the programmer. (STM == software transactional memory)

Could (probably) write monads in any language. Haskell gives you permanence so you can't cheat when writing monadic code and also the primitives to make it easier, such as with IO.

>>= is pronounced bind

The do notation is much easier to read, but just uses bind under the scenes. That is why in Haskell, opposed to other languages, it's easer to accomplish monadic tasks. Other languages would have to use straight up bind or the user would have to re-implement the do notation again.

Intuition of monad = way of enforcing order in a lazy language - not only way to think about it, but a very good way

return in Haskell - not like return in other languages: 
return in Haskell means "rewrap this output in the monad"

Going to the Haskell interpreter:

:t return 1 # :t in the interpreter prints out the type signature
return 1 :: (Num t, Monad m) => m t

:t return 'c'
return 'c' :: (Monad m) => m Char
#Equivalent Java to above == m<Char> where m extends Monad

Q: Interfacing Haskell with impure program?
A: One of first examples, with Haskell calling C wrapper to read files. The naive implementation uses a request/response model when calling out to the C wrapper. Calling it, with lazy language, those calls to the C wrapper can came back out of order. That's why monads were invented.

Bind  (>>=) - Guarantees machine state happens in the order you expect

Haskell runtime written in C, functions exposed to Haskell programmer that sleep the thread, this is what enforces ordering in some cases.

Trying to decide how the Haskell runtime runs a Haskell program is causing some confusion, it is easier to say that the Haskell code describes the program and we don't need to worry so much about how the program is run. When we use constructs like monads, that is what allows us to be assured that the runtime is going to do the right thing.

Haskell accept - takes  socket to listen on, and returns an tuple of (handle, peer, port) in a monad

==Section 3, Semantics

===Evaluation Contexts

Labelled transitions an attempt to formalize the system. Transitions that are labeled are ones that interact with the outside world, transitions are the things that are changing state either into or out of the world.

Defining these transitions allows evaluation of the program as simplifying the equation.

For your language to interact with the world need to expand your picture, to things in the world. Any nondeterministic function you can't predict, but if you specify at each point the transitions that the function has to go through, you can reason enough about it.

moggi paper - written about monads in terms of semantics, not in any practical terms

Insight - can use operational semantics to perform optimizations and PROVE that the end results are the same, so you need to prove the IO and unsafe things in such a way to perform those optimizations. Need formal rules to say when two different looking programs are the same.

==Section 4.1

Defines the main loop of a web server

acceptConnections :: Config -> socket -> IO () - io unit

Will run forever, accepts a socket and forks

  forever = a >> forever a

Where a is an action and >> is similar to bind but doesn't return a value, just guarantees that you call a and then forever a in order.

Monads are not limited to IO, you could say that monads might be equivalent to interfaces in Java

bind for the IO monad is a primitive

You can define bind for other types that don't have to a primitive and can be lazy. bind only enforces sequential behavior in the IO monad

==Implementing non IO monads in Haskell

  instance monad [] where
    return a = [a]
    xs >>= f = concat (map f xs)

  #type signatures:
  xs :: [X] - xs is list of X
  f :: (X -> [S]) - function f goes from X to list of S
  xs >>= f :: [S] - 
  map f xs :: [[S]]


=== Maybe monads:


  data Maybe a = Nothing | Just a

Maybe a is either value nothing or value a

  instance Monad Maybe where
    return a = Just a
    (>>=) :: Maybe a ->
             (a -> Maybe b) ->
             Maybe b
    ## bind has type function from maybe a to function from a to maybe b to maybe b

    Nothing  >>= f = Nothing
    (Just a) >>= f = (f a)


Using the maybe monad

  #standard implementation
  maybeGetInt :: Maybe Int
  getTwoInts :: Maybe (Int, Int)
  getTwoInts = maybeGetInt >>= \i1 ->
               maybeGetInt >>= \i2 ->
               return (i1, i2)

  #getTwoInts formatted showing nested lambdas
  getTwoInts = maybeGetInt >>=
    \i1 -> maybeGetInt >>=
      \i2 -> return (i1, i2)

  #using explicit bind function and code reformatted not in a Haskell style
  maybeGetInt :: Maybe Int
  getTwoInts :: Maybe (Int, Int)
  getTwoInts = 
    let bind = (>>=) in
    let g = (\i2 -> return (i1, i2)) in
    let f = (\i1 -> bind maybeGetInt g) in
    bind maybeGetInt f

  #using the maybe monad in a do block
  #much easier
  maybeGetInt :: Maybe Int
  getTwoInts :: Maybe (Int, Int)
  getTwoInts = do { i1 <- maybeGetInt
                    i2 <- mabyeGetInt
                    return (i1, i2) }



==Talking about "World"

  type IO a = World -> (a, World)

It is nonsense, something real that really shouldn't be exposed to the person. It's a (false) abstraction to formalize writing functional code

Anything could really cause side effects in there real world i.e., temperature of computer

For us "side effect free", means that programmer can't directly influence the world, programmer can't assign variables

Comment that Haskell has similar problem as Perl in that syntax can change a lot and mean the same thing

Next paper is The Zippers, http://www.st.cs.uni-saarland.de/edu/seminare/2005/advanced-fp/docs/huet-zipper.pdf
