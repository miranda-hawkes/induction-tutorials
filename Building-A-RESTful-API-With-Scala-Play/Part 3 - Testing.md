# Part Three - Testing

## Mocking With Mockito
Go back to your `ApplicationControllerSpec`. There should be an error, as our controller now has two extra injected dependencies it didn't have before.

We need to add another library dependency to help with unit testing our controller. Add the following library dependency with the rest:
```
"org.mockito" % "mockito-core" % "2.28.2" % Test
```

In `ApplicationControllerSpec`, extend it with `MockitoSugar`. See if you can figure out what import you need.

Then, add the following below the creation of `controllerComponents`:

```
implicit val executionContext: ExecutionContext = app.injector.instanceOf[ExecutionContext]
val mockDataRepository: DataRepository = mock[DataRepository]
```

We've created an `ExecutionContext` class in the same way as the `ControllerComponents`, but our `DataRepository` is different.

### Why?
* We explicitly call methods on the `DataRepository` class in the controller
* These methods can return different responses based off what you call it with
* We don't want to test our `DataRepository` functionality as part of this spec, we only care about the `ApplicationController`

The solution to these problems is a concept known as mocking, where you explicitly set what any method on `DataRepository` should return, so that you can test how your controller responds in certain scenarios.

**Now fix the error in the test spec, by passing these new values into the test controller.**

### Mocking the find() function in the DataRepository

Try running the tests. You should get a `java.lang.NullPointerException`.
This is because, although we've now created a `mock[DataRepository]`, we haven't told it what to do yet.

Add these imports:
```
import org.mockito.ArgumentMatchers._
import org.mockito.Mockito._
```

Add the following to your test, just **above** where you create the `val result = ...`.

```
val dataModel: DataModel = DataModel(
    "abcd",
    "test name",
    "test description",
    100
)
  
when(mockDataRepository.find(any())(any()))
  .thenReturn(Future(List(dataModel)))
``` 
* The `when` keyword is a Mockito function used for stubbing methods
* Inside of this, you'll see `mockDataRepository.find` passed in
* The signature of the `find()` method is like so:
    ```
    def find(query: (String, JsValueWrapper)*)(implicit ec: ExecutionContext): Future[List[A]]
    ```
    It expects two parameters - one inside the first brackets and one inside the second
    It returns a `Future[List[A]]` - `A` is the same data type as what your repository stores, in this case `DataModel` 
* `any()` is a type of Argument Matcher. By writing `mockDataRepository.find(any())(any())` you are loosely saying "when this method is called with any two parameters..." 
* All of this equates to: "when I call the .find function with any parameters, eventually return the `dataModel` I created as a single entry in a `List`"

Now run the tests again and they should pass.

### Testing the create() function

To test this function, we need to use a different HTTP request method with a body, detailing what we want to add to Mongo.

 
