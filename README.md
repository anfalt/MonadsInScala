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

## Links
Monad transformers down to earth by Gabriele Petronella:<br>
[https://www.youtube.com/watch?v=jd5e71nFEZM&feature=youtu.be](https://www.youtube.com/watch?v=jd5e71nFEZM&feature=youtu.be)<br>
[https://blog.buildo.io/monad-transformers-for-the-working-programmer-aa7e981190e7](https://blog.buildo.io/monad-transformers-for-the-working-programmer-aa7e981190e7)

Blog Post by Sinisa Louc<br>
[https://medium.com/@sinisalouc/demystifying-the-monad-in-scala-cc716bb6f534](https://medium.com/@sinisalouc/demystifying-the-monad-in-scala-cc716bb6f534)

Monads do not compose<br>
[http://blog.tmorris.net/posts/monads-do-not-compose/](http://blog.tmorris.net/posts/monads-do-not-compose/)
