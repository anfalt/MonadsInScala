## Exercises
Download the Exercise Project and open it in Intellij.
https://github.com/anfalt/MonadsInScala/raw/master/KP_MonadsInScala_Exc.zip

### Scala Basics




#### Option
Navigate to KP_MonadsInScala_Exc\src\Scala_Basics\1_Option.sc and write a Method that tries to parse a String to an Int value.
Return a Option[Int]. If the parsing was successful, return a the type Some, if the string was no valid int return None.
Use the your _toInt_  method in flatMap and calculate the sum by using _sum_.

Documentation of Option:
http://www.scala-lang.org/api/2.12.3/scala/Option.html

Documentation sum method of a List:
http://www.scala-lang.org/api/2.12.3/scala/collection/immutable/List.html#sum:A

#### Future

Open KP_MonadsInScala_Exc\src\Scala_Basics\2_FutureVSThread.sc and implement the given thread by using a Future

Documentation of Future:
http://www.scala-lang.org/api/2.12.3/scala/concurrent/Future.html


Open KP_MonadsInScala_Exc\src\Scala_Basics\3_ExceptionHandlingFutureVsThread.sc. Implement a thread, that will throw an exception while its execution. Handle the Exception by using Thread.UncaughtExceptionHandler.

Now Implement the same logic with a Future.


Navigate to KP_MonadsInScala_Exc\src\Scala_Basics\4_FutureHTTP.sc and wrap a HTTP call in a simple Future.
Print the HTTP Response of a GET Rquest to https://my-json-server.typicode.com/anfalt/MonadsInScala/Invoices by using the onComplete Method of Future

Documentation of Future:
http://www.scala-lang.org/api/2.12.3/scala/concurrent/Future.html

### Monad Exercise
KP_MonadsInScala_Exc\src\main\scala.
The project strucutre contains a data model with the classes _Customer_, _Invoice_ and _InvoiceItem_. Those classes are already implemented and you just can use them. Furthermore there are two Service classes for requesting Invoices and Customers from a JSON Webservice. The first task is to implement the methods _requestInvoiceByID_ in _requestCustomerByID_. The implemented mehtod _requestAllInvoices_ in InvoiceService.sc can be used as an example.
After that implement the _requestInvoiceCustomerDataWithoutMonad_ method in Program.sc with flatMap and without using a Monad.
The last exercise is to write the same method by using the _OptionT_ class of the Cats library.

Documentation OptionT:
https://typelevel.org/cats/datatypes/optiont.html




