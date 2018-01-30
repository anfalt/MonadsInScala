## Monads in Scala

[Exercise](https://github.com/anfalt/MonadsInScala/blob/master/02_Exercises.md)

In this blog we get to know more about Monads. 

Monads are derived from category theory in mathematics. They are commonly used in programming to mean higher order functions. 
For this post it isn't necessary to have knowledge in category theory.

Having a programming language like Scala where everything has to be immutable is often hard to intercat. Because in the real world everything changes the whole time. 
They found something in monad which allows to explain it in the function contract. Everytime you call a function with the same argument you have to get same result in cause no side effects. But if you pass a function as parameter to a function you can get different results.

Before we get to know more about Monads we have to explain a Options, Futures and also defining a Functor because Monad is a type of it.


## Scala Basics 

### Option
The Option type in scala is used to wrap a value that could be null. You can compare
this type with _Nullables_ in Java. To check the value of a nullable type, you often see
nested ifs like the following:
```Java
String housenumber = null;
User user = userService.getUserByID(1);
if(user != null){
  Address address = user.getAddress();
  if(address != null){
    housenumber = address.getHousenumber();
  }
}
```
One needs to null-check every object and property of the object. You could to this
in the same way in scala. Or you could use the generic _Option_ type. An Option can
either be an instance of the _Some_ class or the _None_. As the following example shows,
the Option type provides a factory method, that creates None if the given parameter
is null, otherwise it creates an instance of Some.

```scala
val greeting: Option[String] = Some("Hello world")
val greeting2: Option[String] = None

val factoryGreeting: Option[String] = Option("Hello world")
val factoryGreeting2: Option[String] = Option(null)
```

As you can see it´s really simple to create a new Option type. By wrapping the
value that may be null with the Option, it is possible to work with it without any
null-checking.This can be achieved by using flatMap and Options.

```scala
class User(var firstName:String, var lastName:String,var address:Option[Address]){
  def getAddress():Option[Address] = return this.address
}

class Address(var city:String,var street:String,var houseNumber:Option[String]){
  def getHouseNumber():Option[String] = return this.houseNumber
}

def address1 = new Address("Rosenheim","Hochschulstr.",Option("1b"))
def address2 = new Address("Rosenheim","Hochschulstr.",Option(null))

def user1 = new User("Max","Mustermann",Option(address1))
def user2 = new User("Max","Mustermann",Option(address2))
def user3 = new User("Max","Mustermann",Option(null))

user1.getAddress().flatMap(_.getHouseNumber()) //Some(1b)
user2.getAddress().flatMap(_.getHouseNumber()) //None
user3.getAddress().flatMap(_.getHouseNumber()) //None 

```

One disadvatage of the Option type is that you get no feedback why the wrapped value is None. In some cases you need to know why there is no value inside the Option. An example could  be  you try to request a user from a user service. If the returned Option contains None, you don´t know if the user doesn´t exist, or if a server or network error occurred. For this cases the _Either_ type exists.
Either provides the same funtionality like Option, None and Some are called Left and Right. The Difference is, that Left can contain a value, that can describes the reason why not the expected value is inside. The expected value would be inside Right. 

### Future
The _Future_ type is used to run programm steps concurrently. By creating a Future it
starts running asynchronous and will return a value in some time. You may known asynchronous programming with threads, so we start with a little comparison Threads vs Futures.
Asynchronous programming with threads:
```scala
 val thread = new Thread {

    var resultValue = 0;

    override def run = {
    resultValue =  func(0)
      println("result: " + resultValue)
    }
    def func(a:Int): Int ={
      if(a<20){
        println("Value of a: " + a);
        func(a+1)
      }else{
        a
      }
    }

    def getValue() = {
         resultValue
    }
  }
  
thread.start()
println("This gets printed before the values of a")
thread.join()
println("This gets printed after the values of a")
val finalA = thread.getValue();
println("Final value of a: " + finalA)
```

Basically the same logic implemented with a future:

```scala
val f = Future {
    def func(a:Int): Int ={
      println("Value of a: " + a);
      if(a<20){
        println("Value of a: " + a);
        func(a+1)
      }else{
        a
      }
    }
    println("This gets printed before the values of a")
    func(0)
    Thread.sleep(3000)
  }

  f.onComplete {
    case Success(value) => println(value)
    case Failure(e) => e.printStackTrace()
  }

  println("This gets exectuded befor the values of a")
```

The _onComplete_ function accepts a callback, that gets executed, when the asynchronous function has
finished. The result value can be an instance of _Success_ or _Failure_ class, depending on whether the request was successful or not. 
As you can see the Error and Exception Handling in Futures is pretty straight forward. Exception Handling in Threads is not as easy:



Exception Handling in asynchronous executed threads:

```scala
 val excHandler = new Thread.UncaughtExceptionHandler() {
    override def uncaughtException(th: Thread, ex: Throwable): Unit = {
      System.out.println("Exception Handler: Uncaught exception: " + ex)
    }
  }

  val thread = new Thread {



    override def run = {
      val a =  1/0
      println("value of a: " + a)

    }

  }
  thread.setUncaughtExceptionHandler(excHandler)
  thread.start()
```


A simple HTTP GET Request could be implemented like this.

```scala
val f = Future {
  Http("https://jsonplaceholder.typicode.com/posts/22").asString
}
f.onComplete {
  case Success(value) => println(value)
  case Failure(e) => e.printStackTrace()
}
println("This gets printed before the HTTP Response value");
```
To test this Snippet e.g in an scala worksheet, you may have to add the following line to wait until the Futere has returned a value. 
Otherwise the executing thread has stopped, before the async request has finished and now output is visible.
```scala
Await.result(f, 10 seconds)
```
Since the the future is not blocking the programm execution, the println command
gets executed before the onComplete callback. The Future type also provides a flatMap
method, that allows us to control the execution order of multiple Futures.

```scala
def getUserID() : Future[String] = {...}
def getUserAddress(userID: String) : Future[Address] = {...}
def createInvoice(address:Address) : Future[Invoice] = {...}

val futResult: Future[Invoice] =
  getUserID().flatMap( userID =>
    getUserAddress(userID).flatMap( address =>
      createInvioce(address)
    )
)
```



### Functor

Functors are objects that can implement the map function. Examples for objects are collections, container or single values.
Overall you can say if you can map the object, it is a Functor.

A map is a higher order function. This means that it's a function that either takes a function as an argument or returns a function. For example you define a List in Scala:

```Scala
val l1 = List(1,2,3)
//l1: List[Int] = List(1,2,3)
```

in the next step you create a new list by adding one to each element of the list by using the map function:

```Scala
val l2 = l1.map(x=>x+1)
// l2: List[Int] = List(2, 3, 4)
```

this makes it clearer that map is a higher order function on lists. Therefore is List[] a Functor because you can map it.

In Scala you can think of a Functor as a trait. In the following pseudo-code represents F the Functor:

```Scala
trait F[A] {
  def map(f: A => B): F[B]
}
```

What functors do is to transform the context of a value and you will get back the same type of structure. 

You have a value which is wrapped into a context ``` F[A]```. 

After you apply map to it, the value will be unwrapped and the function will be applied ```(f: A => B)```. 

At least the value will be rewrapped back into the context ```F[B]```. 

If you look at the first example, you have List l1 and after you map it you get l2 which is also a List and again a functor.


### Monads

Monads are like wrappers where you can wrap objects. It's like wrapping a present to make it look prettier and that's the same with Monads. They are also allow us to handle asynchronous code which is very powerful.

Recalling that a Functor is a type which has a map method, a Monad is a type of Functor which implements the flatMap method. This can be written in a pseudo-code:

```Scala
trait M[A] {
  def map(f: A => B): M[B]
  def flatMap(f: A => M[B]): M[B]
}
```

You have a class M with the type A, this would be written in M[A]. If you want to flatten it you need to provide a function A -> M[B]. FlatMap will transform A into M[B] and the result is M[M[B]]. After mapping, flatMap will flatten it and you get M[B].


The following example shows how to map a function returning a list over the elements of a List:

```Scala
val l3 = l1.map { x => List(x-1, x+1) }
//l3: List[List[Int]] = List(List(0, 2), List(1, 3), List(2, 4))
```

This example returns a list of lists but this result looks a bit confusing. If you want to flatten it down to have only the list with the values in it you can use the operation flatMap:

```Scala
val l4 = l1.flatMap { x => List(x-1, x+1) }
//l4: List[Int] = List(0, 2, 1, 3, 2, 4)
```

Notice that not all functors can have a flattening operation, so not all functors are Monads, but all Monads are Functors. Monads are therefore more powerful than Functors.

The flatMap function doesn't require a function with A -> M[A]. If M is a List and A is a Int, Int -> List[String], this would also work, so it doesn’t have to be Int -> List[Int].

```Scala

val l5 = l1.flatMap(x=>List(x + "a"))
//l5: List[String] = List(1a, 2a, 3a)
```

Therefore flatMap is more powerful then map because it will chain operations together. The map function is just a subset of flatMap.

The syntactic sugar of a flatMap method is to write it as a for-comprehension with the symbol ```<-```. Notice that it is not comparable with a for-loop.

```Scala
l1.flatMap(x => l2.map(y => x + y ) )
//res0: List[Int] = List(3, 4, 5, 4, 5, 6, 5, 6, 7)

for {
    first <- l1
    second <- l2
} yield first + second
//res1: List[Int] = List(3, 4, 5, 4, 5, 6, 5, 6, 7)
```

As you can see, the for-comprehension is much more readable then using the flatMap method.


## When do we need Monads?

To get a better understanding why Monads are helpful, here is an example.

Lets say you want to get the address from a user. These two methods will return a user and an address with the type Future. 

```scala
def getUser(name: String): Future[User] = ...

def getAddress(user: User): Future[Address] = ...
```

As we mentioned before, Future is an asynchron computation and will return a value in some time later. It has also the flatMap method which you can use to transform the content and also write it as a for-comprehension.

```scala
val getCity: Future[String] =
      getUser("Andreas").flatMap(user => getAddress(user).map(address => address.city))
```


```scala
val getCity: Future[String] = 
    for {
        user        <- getUser("Andreas")
        address     <- getAddress(user)
    } yield address.city
```

The code is now much more easier to read. First you get the user and pass it to the getAddress method. Once you have the address you will get the city of it.

There are several problems which we didn't mention yet. It is possible that not every name corresponds to an existing user or the user has no address. For this reason you can use Option which serves this purpose. You just have to change the returntype of the two methods and the value ```getCity```.

```scala
def getUser(name: String): Future[Option[User]] = ... 

def getAddress(user: User): Future[Option[Address]] = ...
```

```scala
val getCity: Future[Option[String]] = 
    for {
        user        <- getUser("Andreas")
        address     <- getAddress(user)
    } yield address.city
    
```

There is another problem because it doesn't compile any more.

```scala
error: type mismatch;
 found   : Option[User]
 required: User
           address <- getAddress(user)
```

The getAddress method needs the type User but getUser hands over an Option[User]. In the follwing example you can see the procedure of unwrapping the values. The first extraction works easily but second where you hand over the value user is failing. 

```scala
user    <- getUser("Andreas")   <=>   Option[User] <- Future[Option[User]] 

address <- getAddress(user)     <=>   address <- getAddress(Option[User]) //ERROR            
```

Now you try to solve the problem by writing something like this:

```scala
val getCity: Future[Option[String]] = 
    for {
        maybeUser  <- getUser("Andreas")
        maybeCity  <- maybeUser match{
            case Some(user) => getAddress(user).map(_.map(_.city))
            case None => Future.successful(None)
        }
    } yield maybeCity
```

This code doesn't look good so you ideally want to write it more readable:

```scala
def city: Future[Option[String]] =

  for {
    user    <- maybeUser    <- getUser("Andreas")
    address <- maybeAddress <- getAddress(user)
  } yield address.city
```    

You double flatMap the two methods. First extracting from the Future and then from the Option that you get just the user.

```
        user        <-       maybeUser       <-       getUser("Andreas")

"Andreas" = User <------ Option["Andreas"] <------ Future[Option["Andreas"]]
```

This code example doesn't work in real live so you want something like a method which works on Future[Option[X]]. For this case you need a wrapper class.


### Wrapper class

Let's go back to Functors and Monads. If you have two Functors A and B and you know how to map over A[X] and B[X] you also know how to map over A[B[X]].
This is not possible for Monads. Knowing how to flatMap over A[X] and over B[X] doesn't grant you to flatMap over A[B[X]].

Therefore you have to write a wrapper class that works for Future[Option[A]] with its own map and flatMap methods.

The first snippet shows again the definition of a Monad class and the second the wrapper class.

```scala
trait M[A] {
	def map(f: A => B): M[B] 
	def flatMap(f: A => M[B]): M[B] 
}
```

```scala
case class FutOpt[A](value: Future[Option[A]]) {
 
    def map[B](f: A => B): FutOpt[B] = FutOpt(value.map(optA => optA.map(f)))
  
    def flatMap[B](f: A => FutOpt[B]): FutOpt[B] = 
        FutOpt(value.flatMap(opt => opt match {
            case Some(a) => f(a).value
            case None => Future.successful(None)
        }))
}
```

Now we can try again to get the city from a user and running the code with the wrapper class. You just have to use the FutOpt in the for-comprehension and use .value to get the result.

```scala
val f: FutOpt[String] =
    for {
        user    <- FutOpt(getUser("Andreas"))
        address <- FutOpt(getAddress(user))
    } yield address.city
    
    
val getCity: Future[Option[String]] = f.value
```

The wrapper class also works for List[Option[A]]:

```scala
case class ListOpt[A](value: List[Option[A]]) {
 
    def map[B](f: A => B): ListOpt[B] = ListOpt(value.map(optA => optA.map(f)))
 
    def flatMap[B](f: A => ListOpt[B]): ListOpt[B] =
        ListOpt(value.flatMap(opt => opt match {
            case Some(a) => f(a).value
            case None => List(None)
        }))
}
```

As you can see there is no knowledge about the 'outer' Monad (Future or List) required. The 'outer' Monad can be any type of structure as long as you can map or flatMap over it. You just need a knowledge about the 'inner' Monad, which is Option in this case. In general it's a generic data structure that wraps any Monad around an Option.

The library cats already provides a class OptionT which solves this problem.


### OptionT

OptionT has two type parameters F and A: **OptionT[F, A].**
F is the wrapping Monad and A is the type inside Option. OptionT[F, A] is a flat Version of F[Option[A]] which we had in our examples before. It is also Monad and has its own map and flatMap methods so you can use the for-comprehension.

In general you can say: **Foo[Bar[X]] becomes BarT[Foo, X]**

To use the OptionT you have to import the cats library. The value ```f``` need to have the return type OptionT and in the for-comprehension OptionT it will extract the user and the address. With ```.value``` you can get the result.

```scala
import cats.data.OptionT
import cats.std.future._


val f: OptionT[Future, String] =
    for {
        user    <- OptionT(getUser("Andreas"))
        address <- OptionT(getAddress(user))
    } yield address.city


val getCity: Future[Option[String]] = f.value
```

For OptionT the return type has to be Future[Option[X]]. If you have methods with another return type like Option[String] or Future[Int] it doesn't work.

The code could also be written a bit nicer from the beginning when you know you wrapping more often. The methods could return OptionT directly.

```scala
def getUser(name: String): OptionT[Future, User] = OptionT { ... }
def getAddress(user: User): OptionT[Future, Address] = OptionT{ ... }


val f: OptionT[Future, String] =
    for {
        user <- getUser("Andreas")
        address <- getAddress(user)
    } yield address.city


val getCity: OptionT[Future, String] = f.value
```

Hint: OptionT provides several methods to transform types into the expected type:

```scala
def getUser(id: String): Future[Option[User]] = ... 
def getAge(user: User): Future[Int] = ... 
def getNickname(user: User): Option[String] = ...


val getNickname: OptionT[Future, String] =
    for {
        user <- OptionT(getUser("123"))
        age  <- OptionT.liftF(getAge(user))   
        name <- OptionT.fromOption(getName(user))
    } yield name.nickname
```


### Monad Best Practise
- Don´t stack more than two monads 
- Don´t expose Monads in a public API
- Monads influence can reduce the performance of your code
- Use Monads only for local optimization, not as a global/architectural solution


## Links
Monad transformers down to earth by Gabriele Petronella:<br>
[https://www.youtube.com/watch?v=jd5e71nFEZM&feature=youtu.be](https://www.youtube.com/watch?v=jd5e71nFEZM&feature=youtu.be)<br>
[https://blog.buildo.io/monad-transformers-for-the-working-programmer-aa7e981190e7](https://blog.buildo.io/monad-transformers-for-the-working-programmer-aa7e981190e7)<br>

Blog Post by Sinisa Louc<br>
[https://medium.com/@sinisalouc/demystifying-the-monad-in-scala-cc716bb6f534](https://medium.com/@sinisalouc/demystifying-the-monad-in-scala-cc716bb6f534)<br>


Blog Posts by Darren Wilkinson<br>
[https://darrenjw.wordpress.com/2016/04/15/first-steps-with-monads-in-scala/](https://darrenjw.wordpress.com/2016/04/15/first-steps-with-monads-in-scala/)<br>
[https://darrenjw.wordpress.com/2015/11/16/hofs-closures-partial-application-and-currying-to-solve-the-function-environment-problem-in-scala/](https://darrenjw.wordpress.com/2015/11/16/hofs-closures-partial-application-and-currying-to-solve-the-function-environment-problem-in-scala/)<br>

Monads do not compose<br>
[http://blog.tmorris.net/posts/monads-do-not-compose/](http://blog.tmorris.net/posts/monads-do-not-compose/)<br>

Functors, Applicatives, And Monads In Pictures<br>
[http://adit.io/posts/2013-04-17-functors,\_applicatives,\_and\_monads_in_pictures.html](http://adit.io/posts/2013-04-17-functors,\_applicatives,\_and_monads_in_pictures.html)<br>

Monads do not compose<br>
[http://blog.tmorris.net/posts/monads-do-not-compose/](http://blog.tmorris.net/posts/monads-do-not-compose/)

