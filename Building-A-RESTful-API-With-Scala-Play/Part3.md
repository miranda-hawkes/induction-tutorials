# Part Three - Testing
Working on Scala & Play Framework microservices you'll find yourself needing mocks and stubs a great deal for unit testing.
This section aims to introduce you to the flexibility of the Mockito mocking framework.

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
* We explicitly call methods on the `DataRepository` class in the controller and these methods can return **different** responses based off what you call it with
* We **don't** want to test our `DataRepository` functionality as part of this spec, we only care about the `ApplicationController`

The solution to these problems is a concept known as mocking, where you **explicitly** tell the methods in `DataRepository` what to return, so that you can test how your controller responds independently.

**Fix the error in the test spec, by passing these new values into the test controller as parameters.**

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
* We've created a simple test `DataModel` to use later
* The `when` keyword is a Mockito function used for stubbing methods. Inside of this, you'll see the `mockDataRepository.find` method passed in - this is the `DataRepository` method that is used in our controller that needs to be stubbed.
* The signature of the `find()` method in the library we're using is like so:
    ```
    def find(query: (String, JsValueWrapper)*)(implicit ec: ExecutionContext): Future[List[A]]
    ```
    * It can take **two** parameters - a *mongo query* and an *execution context*
    * It returns a `Future[List[A]]` - `A` is the **same data type** as what your repository stores, in this case `DataModel` 
* `any()` is a type of argument matcher used for defining the parameters of a stubbed method. There are many other more specific argument matches, for example `anyString()`, `anyBoolean()` etc.
* All of this equates to: 
    1. *when the `.find()` function is called*
    2. *eventually (in a Future)*
    3. *return the `dataModel` I created as the single entry in a `List` object*

Now run the tests again and they should pass.

### Mocking the create() function

To test this function, we need to supply a body in the request, detailing what we want to add to Mongo.
Update your .create test:

```
  "ApplicationController .create" when {
    
    "the json body is valid" should {
      
      val jsonBody: JsObject = Json.obj(
        "_id" -> "abcd",
        "name" -> "test name",
        "description" -> "test description",
        "numSales" -> 100
      )

      val writeResult: WriteResult = LastError(ok = true, None, None, None, 0, None, updatedExisting = false, None, None, wtimeout = false, None, None)

      when(mockDataRepository.create(any()))
        .thenReturn(Future(writeResult))

      val result = TestApplicationController.create()(FakeRequest().withBody(jsonBody))

      "return ???" in {
        status(result) shouldBe ???
      }
    }
  }
```

* A valid JSON body is created, matching fields required in a `DataModel`
* A `WriteResult` is created - this is the data type returned by the `DataRepository.create()` function. The one created here shows the addition to mongo was a success. Don't worry about the many parameters!
* `DataRepository.create()` is mocked - it returns a Future with the write result created earlier
* `.withBody(jsonBody)` passes the JSON that was created to the body of the `FakeRequest`

#### Tasks
1. Update the test assertion with the correct HTTP status we expect to be returned

2. Create a test scenario for the `create()` function where the supplied JSON is **not** valid. Put it inside the `"ApplicationController .create" when {` brackets.
    Do you need the mocking here?

3. Create two test scenarios for the `.update()` function:
    1. The supplied JSON is valid - we also want to check the body of the response contains the JSON you passed in. You can use:
     ``` 
     "return the correct JSON" in {
         await(jsonBodyOf(result)) shouldBe jsonBody
     }
     ```
     You will need the following created at the top of the test spec to use the `jsonBodyOf` function:
     ```
     implicit val system: ActorSystem = ActorSystem("Sys")
     implicit val materializer: ActorMaterializer = ActorMaterializer()
     ``` 
    2. The supplied JSON is invalid
    
4. Update the test for `index()` to check the body of the result    

5. Create a test scenario for the `delete()` and `read()` functions

## Catering for failed Mongo queries

One thing we haven't catered for is if the Mongo query fails. This could be due to a number of reasons.

Add a scenario to the `ApplicationController .create` test:

``` 
"the mongo data creation failed" should {

  val jsonBody: JsObject = Json.obj(
    "_id" -> "abcd",
    "name" -> "test name",
    "description" -> "test description",
    "numSales" -> 100
  )

  when(mockDataRepository.create(any()))
    .thenReturn(Future.failed(GenericDriverException("Error")))

  "return an error" in {

    val result = TestApplicationController.create()(FakeRequest().withBody(jsonBody))

    status(result) shouldBe Status.INTERNAL_SERVER_ERROR
    await(bodyOf(result)) shouldBe Json.obj("message" -> "Error adding item to Mongo").toString()
  }
}
```

Some key differences:
* We stub a `Future.failed(GenericDriverException("Error"))` - a type of exception reactive mongo may throw if the query failed
* We expect an INTERNAL_SERVER_ERROR
* The JSON returned should have a suitable error message

1. Run the tests - this scenario should fail.

2. Update the controller method to catch any generic `ReactiveMongoException` using a `recover` block.

```
dataRepository.create(dataModel).map(_ => Created) recover {
  case _: ReactiveMongoException => InternalServerError(Json.obj(
    "message" -> "Error adding item to Mongo"
  ))
}
```

3. Run the tests again and they should pass

4. Implement the same `recover` block for the other methods and create tests for these.
     
## Manually Testing your API

What if you want to see your API in action adding data to Mongo?

You can use [CURL](https://curl.haxx.se/) to make HTTP requests for each of the new endpoints you've created.

1. Start your service with `sbt run`

2. In the command line, run the following command:

```
curl "localhost:9000/api/index" -i
```

You should see something like the following:
``` 
HTTP/1.1 200 OK
Referrer-Policy: origin-when-cross-origin, strict-origin-when-cross-origin
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Content-Security-Policy: default-src 'self'
X-Permitted-Cross-Domain-Policies: master-only
Date: Sun, 05 Apr 2020 11:51:03 GMT
Content-Type: application/json
Content-Length: 2

[]
```
* The first line shows the response was a HTTP 200 OK
* The next few lines show response headers and meta data
* The last line `[]` is the response body. Because there's nothing in Mongo yet, this is empty

### Populating the database

To populate the database with some data, run the following CURL command:
```
curl -H "Content-Type: application/json" -d '{ "_id" : "1", "name" : "testName", "description" : "testDescription", "numSales" : 1 }' "localhost:9000/api" -i
```
* `-H "Content-Type: application/json"` sets a header so we can pass JSON into the body
* `-d '{ "_id" : "1", "name" : "testName", "description" : "testDescription", "numSales" : 1 }'` defines the JSON body
 
You should get the following response:
``` 
HTTP/1.1 201 Created
Referrer-Policy: origin-when-cross-origin, strict-origin-when-cross-origin
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Content-Security-Policy: default-src 'self'
X-Permitted-Cross-Domain-Policies: master-only
Date: Sun, 05 Apr 2020 12:22:04 GMT
Content-Length: 0
```

Try running that same command again. You should get the following:
```
HTTP/1.1 500 Internal Server Error
Referrer-Policy: origin-when-cross-origin, strict-origin-when-cross-origin
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Content-Security-Policy: default-src 'self'
X-Permitted-Cross-Domain-Policies: master-only
Date: Sun, 05 Apr 2020 12:22:56 GMT
Content-Type: application/json
Content-Length: 40

[{"message": "Error adding item to Mongo"}]
```

This is because you've tried to add a document to mongo with the same `_id`.
A DatabaseException was thrown and it was caught in the controller by the code you added.

### Viewing your database
There are a few ways you can view your newly created data in Mongo. One way is to download an app that provides a useful UI.

1. Download and install [MongoDB Compass](https://www.mongodb.com/products/compass)

2. Once installed, open the app and you should be able to connect with the default settings.

3. You should see a database with the same name as your project (set in `application.conf`). Click this

4. There should be a collection called `data` (set in `DataRepository`). Click this

5. You should see an entry for the data you created!

### Manually testing the rest of the API

1. Add another document to the database with a **different** `_id`

2. Run the CURL command from earlier to test the `GET /api/index` route to check it returns your data items

3. Try hitting the `GET /api/:id` route to retrieve **only** the second document you added

4. Hit the `PUT /api/:id` route with a CURL command. To specify the PUT HTTP method, use `-X PUT` in your CURL command

5. Hit the `DELETE /api/:id` route with a CURL command to delete a specific item. How can you specify the DELETE HTTP method?

## Conclusion

Excellent, you've now created a RESTful API with CRUD functions, unit tested it with Mockito and learnt some useful CURL commands.

1. When you're happy your API is working, do another commit and push it up to GitHub.

2. Create a new Pull Request on GitHub, and get a colleague to review your work. Once they're happy, feel free to merge it into your master branch
    
    **Important** - only merge your own pull requests if its your own repo, like this one. For everything else, a member of your team should review and merge the branch into master.