# functional_programming_library_nodejs
A single file Javascript Functional programming module based on the mostly adequate functional programming book

## Introduction

In recent times, the functional programming paradigm has attracted a lot of attention. As commonly understood, it is a paradigm that stresses on composing pure functions in order to achieve desired results. This is not to say that one behaves like a purist and completely avoids non-functional code like if blocks and assignments. Rather, and this is an opinion, the attempt is to restrict such non-functional code to the very bottom of the functional hierarchy, the bricks of the mansion, as it were, and reveal the grand design of the program as far as possible via function composition.


## Acknowledgements and Disclaimers

 All that I know about Functional Programming that I have elaborated in this post, is courtesy "Professor Frisby's Mostly Adequate Guide to Functional Programming". Here is the link to this masterpiece - 


https://github.com/MostlyAdequate/mostly-adequate-guide. 


This post unfortunately might also include what I have misunderstood from my readings, which are all my sole responsibility.


All examples in this post use the infrastructure library that is given in the appendices of the aforementioned book.

## Side-effects and purity

We used the term 'pure functions' above. Pure functions are those that do not have any side effects, including ones that change (mutate) input arguments. This means that pure functions are much like mathematical functions. You pass 2 and 3 to an add function, and it will always return 5, no matter when or where. But as you can see, this restricts what can be done to largely CPU processing strictly on input arguments. No printing to files or terminals, no reading from files, no changing global state, no REST calls, no database operations, no DOM manipulation. In short, no doing anything useful ! Then what is it that goes for purity? You can test once and be sure it will work the same always. Since there is no state dependency, these functions can run correctly even in parallel. They are thus highly suited for use in code that is meant to horizontally scale on commodity machines in a cloud. But what use is all this if they don't do anything useful? Yes, this needs to be addressed, and the way that it is done is by tucking away this impure part inside of standard constructs that are part of the code infrastructure libraries (comprising Functors, Monads and the like, that we shall encounter soon), and which the application developer never writes, or, when this is not always possible, the application developer is provided convenient mechanisms to localize, so that 'this necessary evil' is contained and controlled!


## Functions as first class objects

A big pillar of functional programming is the 'functions are first class objects' feature of the coding language. You need to be able to do with functions what you can do with objects - assign them to variables, pass them as arguments to other functions , return them as return values from other functions, and

I might add, call methods on function objects themselves. This feature makes possible a whole lot of others, like currying, partial application of functions, function generation, function composition etc. This truly is a core requirement.

 Functions that take or return other functions are also called Higher order functions. So 'map' or 'filter' or 'sort' are higher order functions since they take as an input argument another function. The map takes a transform function, filter takes a filtering function, sort takes a comparator function. Curry and Bind are higher order functions since they take and return a function. Although we can't strictly generalize, these functions are usually part of the 'infrastructure'. They are the base on which our applications will be built. We need to understand how they work. We will rarely need to write them ourselves. Elaborating a little bit before time, the map function is a method of the Functor, that we shall see presently (but do see the point-free style below that expresses the same thing as a non-method). That leads me to a side story. When we say 'Functor', do we mean a class or an object? Remember this is not OOP. But if we are forced talk in terms of OOP, then (and with due apologies to those who disagree) I would compare the structures, like Functors, that we elaborate below, to generics in Java/C++.

## Composition

We talked about function composition above. What we mean by that is generating the following  function, fg:

which is a function such that fg(x) =  f(g(x))


fg is called a composition of f and g.

The input of f is obviously the output of g, and we are talking single input argument functions here.

 Let's look at an example. Suppose we have a sentence of words, that we first want to extract the words out from, then concatenate them back with a comma delimiter, and finally convert the alphabets to uppercase. The composed function that does this is:

`const finalFunction = compose(toUpperCase,toString,split(' '))
 const result = finalFunction("Just as a Bilwa fruit may be fully well known on close examination when it is placed on the palm of hand, so it is possible for man to acquire a thorough knowledge of this world as he is in direct contact with it.")`

The value  in result will be 'JUST,AS,A,BILWA,FRUIT,MAY,BE,FULLY,WELL,KNOWN,ON,CLOSE,

EXAMINATION,WHEN,IT,IS,PLACED,ON,THE,PALM,OF,HAND,SO,IT,IS,

POSSIBLE,FOR,MAN,TO,ACQUIRE,A,THOROUGH,KNOWLEDGE,OF,THIS,

WORLD,AS,HE,IS,IN,DIRECT,CONTACT,WITH,IT.'


We use a toString() instead of a join() here since the latter means something else in relation to Monads as we'll see soon, and I don't want to confuse the reader.

## Function generation

Suppose we want to define several functions that make a HTTP GET request to different servers. We can define as many functions to do our job, or more sanely, use the concept of function generation as in the rather contrived example that follows:

`const httpGetGenerator = (baseurl)=>{

   return (path)=>http(baseurl+path)

}
const callAmazon = httpGetGenerator('https://www.amazon.com')
const callGoogle = httpGetGenerator('https://www.google.com');
callAmazon('/getBook/?name=Abhignyana+Shakuntalam") 
callGoogle('/search/?query=Kalidasa')`

callGoogle and callAmazon are functions that we have generated by calling the generator function httpGetGenerator

## Currying and Partial Function execution

 Suppose I define a function

`
const add = (x,y)=>x+y
`

And call it as follows

`
add("Kalidasa")
`

The value returned is "Kalidasaundefined". What has happened is that the function has been invoked with the undefined second variable initialized to undefined, and string addition results in the final value that we see.

 But suppose what we expected out of the invocation

`
add("Kalidasa")
`

is simply that it returned another function that we can later invoke with the remaining second argument? This is made possible by currying.

`
const add = curry((x,y)=>x+y)
`

Invoking curry with a function passed in as argument, results in a function being returned that can later be invoked with one or more of the arguments. If fewer than all arguments are passed in an invocation, then a function that takes rest of the arguments as input, is returned. So, if I say

`
add("Kalidasa")
`

it returns a function that can take one argument, the second one for the original add function. But if I say,

`
add ("Kalidasa", " from Ujjain")
`

the original function is completely invoked and the result is not a function but the string


"Kalidasa from Ujjain"


Similarly, if we invoke it like this we get the same result

`
 add ("Kalidasa")(" from Ujjain")
`

Currying is at the core of all the higher order functions that we will use. We will see that map, filter etc. that we use from the infrastructure are all curried functions.

## Functor

A functor is a data structure, actually a higher data type, that defines a map function. That's it. That's the working definition. A functor can be thought of as a box which wraps the real data. The Javascript array is a common example of a functor. The map method on the functor takes a function that can take this real data as argument and the result of the function call wrapped in the same functor is what is returned by the map method. That is

`
F.of(x).map(f) `   which is the same as F.of(f(x))  



where F is the functor enclosing the data x and f is the function passed to the map method.

## Point-free style

 Some people prefer the point-free style. In the point-free notation, the same thing looks like:

`map(f, F.of(x))` which is the same as  F.of(f(x)) again


What good are functors anyway? They help us write code that comprise only  function calls, avoiding the clutter caused by other syntactic constructs such as if conditionals to handle errors, async processing etc..


## Pointed Functors

They are functors with the 'of' method ( which is pretty much all the functors that we will deal with). This method helps construct functors initialized with minimal default values. So you can construct a pointed functor in the following manner:

`var X = F.of(x)`

## What does it mean to create a box around something?

 When we create a functor F x from a 'value', (actually a Type) x, we are making x part of some concept. A very visual example of a concept is a structure, such as when we put x in a List functor (the structure here is an array)


[x]


or in a Map functor


{"key1": x}


The 'contents' of a functor can be 'values' (actually Types) or functions. In order not to mix them up with infrastructure functions dealing with functors, like 'map', 'of' etc., let us refer to these functions as morphisms (from the book referred to above). Also, it may be mentioned that the generic term for the collection of data types that our morphisms act on and return, is 'Category'. The data types and morphisms that we mentioned before are called 'objects' of the Category. There is a whole branch of Math devoted to this subject which is a grand generalization of several branches. It is called Category Theory. In this scheme of things, the Functor itself can be see as a higher data type, that can be applied to other Types.


## The identity morphism

'id' is a special morphism called the identity morphism. It returns whatever we 

pass to it as an argument. It is useful in proving theorems of functional programming. This theorem proving is useful when we have functional code and we want to prove certain things about the code behaviour. For example look at this trivial theorem, where f is a morphism. This is true irrespective of the value of f. It is an identity. This is a very trivial one, granted. But more complicated theorems like this, that hold for any morphism or data type can be used to prove code behaviour and such an exercise naturally gives us confidence about the  correctness of our program.

`compose(id,    f)    ===    compose(f,    id)    ===    f;`


## Monad
Monad is a functor with the join and chain (also called flatmap) method. Examples of Monad are 'Either', 'Maybe', 'Task', 'IO' etc. The join method simply strips off the outermost Functor layer. The chain method is like map and takes an argument,f, which is the morphism that is applied on the Functor-enclosed value. Additionally, the join method is also called on the  resulting functor. For example

`Task.of(IO.of(x)).join() = IO.of(x)
Task.of(x).chain(f) = Task.of(f(x)).join() = f(x)`
 
## Functor examples

### Maybe

One of the simpler examples of a Functor (Monad) is the Maybe functor. Often times, we need to process values that may be null, and our code then needs to put in additional processing for null value handling. This often leads to the 'forest' of the design intent of your code getting lost amid the 'trees' of conditionals meant for  null and valid value processing. An example follows:


  `const getName=(obj)=>{
  if (obj)
     return obj.name;
    else
     throw new Error("undefined obj");
  }
 const uppercase = str=>str.toUpperCase();
 const capitalize = (str)=>{
    if (typeof str =='undefined')
      throw new Error("undefined");
    return str.toUpperCase();
 }
var a={};
console.log(getName(a));`

This can quickly get complicated.


 `const getProperty=(obj,property)=>{
      if (obj) {
         if (property) {
            return obj[property];
         } else {
            return new Error("undefined property");
          }
      } else {
            return new Error("undefined obj");
      }
 }
 try {
    getProperty(obj, "name");
 } catch(e) {
    console.log(e);
  }`


 See how the important task of getting out the property value from an object is being obfuscated by rest of the code which checks for null/undefined etc. Now let us use  the Maybe functor, which leads to a much neater implementation avoiding clutter due to null checks.


`const getName =(obj)=>{
    return obj.name;
 }
const uppercase = str=>str.toUpperCase();
const capitalize = (str)=>{
    return str.map(upperCase);
 }
var a = {};
compose(capitalize, map(getName),Maybe.of)(a);
//Alternately, if we define a different function 
// const capitalize1 = str=>{
//    return uppercase(str)
//  }
//Then we can call it like this 
// 
// Maybe.of(a).map(getName).map(capitalize1) 
//   or using the point free way (See the section on Either functor
//   for a definition of the point-free map function)
// map(capitalize1, map(getName, Maybe.of(a)))
//  or even
// compose(map(capitalize1), map(getName), Maybe.of) (a)
//  or even
// compose(map(compose(capitalize1, getName)),Maybe.of) (a)`


This returns Maybe(undefined) if a or a.name is undefined or Maybe(<value>) if a is an object with a field 'name' and value <value>. The trick that the Maybe implementation employs is that whenever it contains (it's a 'box' after all) an undefined or null, subsequent processing on it, typically by calling map on it, will do nothing. Otherwise the map function works as expected, applying the morphism that it is passed, to the non-null value. To elaborate, in the last line of the above code, if a or a.name were undefined, then getName would return a Maybe(undefined), and this would go as an argument to function capitalize, whose line str.map(upperCase) would not call upperCase at all, because str is a Maybe containing an undefined.If we were to avoid wrapping 'a' with a Maybe functor, and directly called getName followed by the simpler function capitalize1 (coded in the comment above) on it, then the call to upperCase(str) would throw an exception, since str is undefined, and this condition would need to be handled by cluttering code in function capitalize1.


### Either

While the Maybe functor (Monad) is good for handling null/undefined values gracefully, it only obviates the  need for null checks. Many times the need is for returning a 'good' and a 'bad' value, the latter when there is an error condition, so that different things can be done downstream based on what type of value is returned. In these cases it is often better to have separate functors for good and bad (errorneous)  values so that their conditional handling is automatically taken care of outside the main application code and inside the plumbing code that these functors provide out of the box. Here is where the Either functor comes in. It is a functor class with two sub classes - Right (for a good value) and Left (for a bad or error value).


Let's see how it is used. The example code below has a function getData making a REST request to a remote http server using the url passed as a function parameter. The url itself is generated by generateURL() which takes an argument, endpoint, that can potentially be undefined. In case it is undefined, we want getData to behave properly, yet the code should not seem too cluttered either.


`//getData::string-> boolean

const makeServerCall = (url)=>{

  //... make http call

}


//generateURL:: string->Either

const generateURL = (endpoint)=>{

   if (validate(endpoint)) {

     return new Right(BASEURL+endpoint);

  } else {

     return new Left("endpoint error");

  }

}


//Based on the third argument which is an Either, the either function call below will call the  function

// passed in the first argument if the Either object is a Left, or the function passed in the second 

// argument if the Either object is a Right   


either(err=>console.log("ERROR", err),  makeServerCall, generateURL("/api/changesomething"))


//Alternately you can use composition and call ...

compose(map(makeServerCall), generateURL)) ("/api/changesomething")`


The alternative using compose is interesting. You might wonder what happens if the generateURL call returns a Left object, which then goes as input to the makeServerCall function, but does it really? No. Don't forget the map. It actually goes as the second argument to the map function call ( map(makeServerCall) is a partial function invocation of map, which returns a function that then takes the Either object as input argument). The map function is itself defined as 

 

   `const map =    curry((f,F)=>F.map(f))`


In our example F is Left. And we know that Left.map() quietly returns the Left object on which it is invoked ( 'this' object), i.e. the same Left object. It's a bad object and map knows it should not do further bad things by applying the input function to it.

### Identity

This is the simplest functor there is. It can be thought of as a transparent box around the real value. I have never used this in practice but it is used to state and prove theorems about functors and monads, which in turn can be used to reason about the behaviour of a functional program. Here are some examples of theorems (the accurate word here is identities )

These are some identities for functors


`//    identity

map(id)    ===    id;


//    composition

compose(map(f),    map(g))    ===    map(compose(f,    g));


These are some identities for Monads


//    associativity

compose(join,    map(join))    ===    compose(join,    join);


//    identity    for    all    (M    a)

compose(join,    of)    ===    compose(join,    map(of))    ===    id;`

### List (Array)

  This is the functor equivalent of a Javascript array. It is traversible (sequence and traverse methods are defined) but not monadic (no join and chain methods)


Examples of its use are :


`let l = new F.List([ "Jack Sparrow", "Black Pearl"])

l.map(F.Either.of)`


result will be the functor

 List([Right('Jack Sparrow'), Right('Black Pearl')])

### Map

   This is the functor equivalent of a Javascript Map. It is traversible (sequence and traverse methods are defined) but not monadic (no join and chain methods) and not pointed (no of method). An example of its use follows.


`let m = new Map({name: "Jack Sparrow", location: "Black Pearl"})

let  mapOfList = m.map(str=>str.split(' '));`


mapOfList will contain 

Map({name: ['Jack', 'Sparrow'], location: ['Black', 'Pearl']})


`let result = m.map(F.Either.of)`


result will be the functor

 Map({name: Right('Jack Sparrow'), location: Right('Black Pearl')})


 ### IO

The IO monad is typically employed to enclose synchronous IO tasks, such as accessing or changing an in-memory global cache.

### Task

The Task monad is typically used to enclose results from asynchronous function calls such as results from a database query call.An example follows


`let task = new Task((reject, resolve)=>{
                 db.any('select * from employee')
                      .then((results)=>resolve(results))
                      .then(null, err=>reject(err))
             });`


The argument to a Task constructor is called the fork function. Note how in creating a Task we have only passed its constructor a definition of the fork function. The fork function code gets executed only when the fork method is actually called on the Task functor. This creates possibilities of deferring evaluation until the very end, and till then our functors and morphisms are simply getting composed. This also allows the fork function to be invoked on the same Task several times. The fork method is called like this:


`task.fork(err=>console.log("Ouch"), results=>console.log(results));`


It is only during the fork call that you are defining the resolve and reject functions. This pattern cleanly separates execution of the task from processing of the results of that execution. This is different from how a Promise behaves.



### Promise

The Promise monad,  while very similar to a Task, is different in when the Promise task gets executed.


`let promise = new Promise ((resolve, reject) =>{
                      db.any('select * from employee')
                         .then((results)=>resolve(results))
                         .then(null, err=>reject(err));
                      });`


Here there is no deferring. The function passed to the Promise constructor executes right away. So it is not convenient for composition and deferred execution.

Note that the then function of a Promise behaves like a chain method in that if the function passed in the then call returns a Promise, 'then' still returns an object couched in only one Promise box.

### Stream

### Compose

The Compose functor marks its contained value as a Functor composition - that its value is actually a nested functor - a functor within a functor. For example, we can create a Compose functor by passing F.of(G.of(x)) to its constructor. This creates a functor

   Compose F G x

What good is this? The map method on the Compose functor is defined to ensure that the morphism passed to the map function will execute on the value x deep inside. How is this? The map method of Compose is defined as:


  `class Compose {

     ...

     map: (f)=>{ return map(map(f), this.$value)}

  }`


which ensures that f is finally executed with x as argument.

So if we want to create a Compose out of a functor F G H x, then this would do it


`let c = Compose.of(F.of(Compose.of(G.of(H.of(x)))))`


See how c.map(f) will actually call f on x?

### Applicative Functor

 An applicative functor is a pointed functor with an ap method. Why do we require it when we already have map and chain (the latter for monads)? Let us discuss this with an example. Suppose our requirement is to call a function passing two values one obtained after an (asynchronous) http call, and another obtained after an asynchronous database call. The function (morphism) signatures are:


`//getNameFromServer::url->Task Either n

//getDetailsFromDatabase::place->Task Either d

//printPersonInfo::name->details->IO


getNameFromServer(url).map(map(n=>{getDetailsFromDatabase(place).map(map(d=>{printPersonInfo(n,d)}))})`


This returns a deeply nested Task Either Task Either IO functor. Moreover all three functions are called sequentially. As against this, if we used the ap method, this is what we could do


`Task.of(printPersonInfo)

.ap(getNameFromServer(url).chain(eitherToTask))

.ap(getDetailsFromDatabase(place).chain(eitherToTask))`


This returns a Task IO  functor. Moreover, getNameFromServer and getDetailsFromDatabase can run concurrently.

### Natural Functor transformation 

A natural transformation is a morphism between functors. It is a function that operates on containers themselves. Only those natural transformations are allowed that do not lose information. So, 


ioToTask  is allowed but not


taskToIO 


since converting a potentially asynhronous task abstraction to a deterministic and synchronous one results in loss of information. Other examples of principled transformations are 


maybeToArray, idToMaybe, eitherToTask etc.


### Lenses


### Traversible Functor

A traversible functor B is one that has the sequence and traverse methods defined on it.

In the attempt to get rid of complicated control structures and make our code flow with only function calls and function composition, we return functors and monads from functions, which then have to be consumed by other code, leading to deeply nested functors, much like the Russian dolls, one inside another, with the real value of interest couched deep inside the innermost functor. An example will make this clear.


  `// f:: x-> A a

  // g::x ->B b

  // h::x ->C c


  let fn = compose(map(map(h)),map(g),f)

  fn(x) // Returns  A B C c, and the value c is now 3 levels deep !`


 In case B and C are the same or A and B are the same, then some of the functor layers can be merged if they are Monads and therefore have the chain (also called flatmap) or join methods defined on them. For example if A and B were actually both same, say A then we could say


   `let fn = compose(map(h), join, map(g), f) // returns A C c  `


   or you could go


   `let fn = compose(map(h), chain(g), f) //returns A C c`


Another mechanism to curb the nesting hell, which sometimes works, is to use natural transformations from one functor to another. Several such exist like:


    eitherTotask, ioToTask, promiseToTask, maybeToTask, idToIO, idToMaybe


So, in the above example, if there was a natural transformation from C to A (named cToA), then the return value would reduce to just A a, by doing:


`let fn = compose(chain(cToA),map(h), chain(g), f)  //returns A a`


//Here we are still assuming A and B are the same


But a natural transformation may not exist where we want it. For example there is none from Task to IO (although the reverse transformation is available). So what do we do if we have a 'IO Task a'? It is simple, just use


     `ioToTask(Ix).join()  where Ix is the 'IO Task a' object.`
     
This gives 'Task a'


There are other situations that are more tricky. Suppose we have an applicative functor case where the applicative functor is

   A f

and this is to be applied on another nested functor like

  B A a


 We can't use the ap method of A directly, since the argument to ap requires a functor A, not 'B A'. But all is not lost, provided B is what is called a traversible functor.


Suppose  x is a nested functor, B A a , where B is traversible and A is an applicative functor


then

 `F.sequence(A.of, x) is a functor A B b  // A and B flip`

and

if y is the functor B b, and suppose func is a function that takes b and returns  a functor like A a, then


 F.traverse(A.of, func, y) = A B b


Calling traverse is much like first calling map ( with a function func that returns a functor)


    `map(func,y) = B func(b) //which is functor B A a`

and then applying sequence to the result

   F.sequence(A.of, B A a)  //which is functor A B b

So, assuming x is functor B b, and func is a function that takes b and returns functor A a,

 `F.traverse(A.of, func, x) = x.map(func).sequence(A.of)`


### Examples of traverse follow:


Assume that fromPredicate is a function that takes a function f as first argument and a functor as second argument. f is a function  that  returns a F.Left or a F.Right functor object based on the argument to it. Finally, arr is an array (functor)


`//fromPredicate::(b-> boolean)->( b -> A a)  or

//fromPredicate::(b-> boolean)-> b -> A a  (we use currying after all!)


traverse(Either.of, fromPredicate(f), arr)  = arr.map(fromPredicate(f)).sequence(Either.of) or which is the same thing


sequence(Either.of, map(fromPredicate(f), arr))`


Another example shows that we have an all or nothing behaviour with traverse when the .applicative functor is an Either. The traversible functor in this case is a List. Calling traverse returns an Either err [x,x,x,...]


`var frompred = curry((f,b)=>{if (f(b)) return new Right(b); else return new Left(b);})

arr = new List([9,10,8,90,76,1,-1])

var f = a=>a>10

var g = a=>a >-2

arr.traverse(Either.of, frompred(g))

//outputs Right(List([9, 10, 8, 90, 76, 1, -1]))

arr.traverse(Either.of, frompred(f))

//outputs Left(-1)`


Let's discuss some differences and similarities between chain and traverse. Assuming that the traversible functor is also a monad ( like Identity, Maybe, Either), both chain and traverse called on such a functor, involve applying a function to the contents of the functor. This application returns another functor, so now we have the real value couched in the innermost of two functors, one in another. The chain method can additionally merge the two functors, which need to be the same for this merger to be valid.  The traverse method interchanges the two functors instead, and this is what is required to address the 'nesting hell' in some cases. The following link gives a more elaborate example.






