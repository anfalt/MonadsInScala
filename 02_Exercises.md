## Exercises
Download the Exercise Project and open it in Intellij.
https://github.com/anfalt/MonadsInScala/raw/master/KP_MonadsInScala_Exc.zip

### Scala Basics

#### FlatMap
Navigate to KP_MonadsInScala_Exc\src\Scala_Basics\FlatMap.sc and implement the methods _splitWordsInUpperCaseChars_ and _increaseAndDecreaseListItem_ by using flatMap.

Documentation flatMap of List:
http://www.scala-lang.org/api/2.12.3/scala/collection/immutable/List.html#flatMap[B](f:A=>scala.collection.GenTraversableOnce[B]):List[B]


#### Option
Navigate to KP_MonadsInScala_Exc\src\Scala_Basics\Option.sc and write a Method that tries to parse a String to an Int value.
Return a Option[Int]. If the parsing was successful, return a the type Some, if the string was no valid int return None.
Use the your _toInt_  method in flatMap and calculate the sum by using _sum_.

Documentation of Option:
http://www.scala-lang.org/api/2.12.3/scala/Option.html

Documentation sum method of a List:
http://www.scala-lang.org/api/2.12.3/scala/collection/immutable/List.html#sum:A

#### Future
Navigate to KP_MonadsInScala_Exc\src\Scala_Basics\Future.sc and wrap a HTTP call in a simple Future.
Print the HTTP Response of a GET Rquest to https://my-json-server.typicode.com/anfalt/MonadsInScala/Invoices by using the onComplete Method of Future

Documentation of Future:
http://www.scala-lang.org/api/2.12.3/scala/concurrent/Future.html

### Monad Exercise



