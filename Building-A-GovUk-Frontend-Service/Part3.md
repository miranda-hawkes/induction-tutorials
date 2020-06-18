# Part Three - Model, View, Controller

Lets add some more components to our page to make it more interesting.

Add the following to your Index view code:
``` 
<div class="grid-row">
    <div class="column-one-third form-hint">
        @messages("index.date")
    </div>

    <div class="column-one-third form-hint">
        @messages("index.description")
    </div>

    <div class="column-one-third r-align form-hint">
        @messages("index.amount")
    </div>
</div>

<div class="grid-row">
    <div class="column-one-third">
        11 February 2020
    </div>

    <div class="column-one-third">
        Direct debit
    </div>

    <div class="column-one-third r-align">
        £50
    </div>
</div>
```
This simply creates what looks like a table with three headings and one item underneath. 

1. Add entries to the messages file using the keys specified above so the headings read as the following:
    ```
    Date     Reference      Amount
    ```

2. Refresh the page in the browser to see your changes.

3. Add another 'row' to the table with dummy data similar to the above.

    There are a few problems building up here:
    * There's a lot of duplicated code, for example `<div class="column-one-third">` is used twice
    * The values inside the table are hard-coded in the view. What if the values came from an API or Database and needed to be dynamically displayed?

### Models
We can use what are known as 'models' in the Scala world to hold data that can be dynamically passed into our view.

Models are similar to classes and create data structures based off the parameters you pass in. These parameters are immutable (can't be changed).

A simple model:
```
case class Book(name: String
                author: String,
                numSales: Int)
```
To create a new instance of that model, you can do either of the following:

   ```
   val bookOne = Book("Book name", "Author name", 10)
   val bookTwo = Book(name = "Book name", author = "Author name", numSales = 10)
   ```

1. Create a new directory in `app` called `models`

2. Inside there, create a new Scala file called `Payment`

3. Create a model similar to the `Book` example shown above, but with the following attributes:

    | Field Name  | Data Type               |
    |-------------|-------------------------|
    | date        | org.joda.time.LocalDate |
    | reference   | String                  |
    | amount      | BigDecimal              |

4. Back to your view, change the parameters to the following:

    ```
    @(payments: List[Payment])(implicit messages: Messages)
    ```

   **Now the view accepts a list of Payments as a parameter.**

5. Refresh the page - there's now a compilation error. This is because there's a **new parameter** for `Index` that we're not supplying in the controller when we call the view.
We can create a list of dummy (fake) Payments for demonstration purposes.

6. Update the controller action:
    ```
    def index: Action[AnyContent] = Action { implicit request =>
      Ok(indexView(List(
        Payment(LocalDate.parse("01-01-2020"), "abcd1234", 20.00),
        Payment(LocalDate.parse("05-01-2020"), "efgh5678", 50.00)
      )))
    }
    ```
    * `LocalDate.parse()` creates a date based off a String value.
    * e.g. `LocalDate.parse("01-01-2020")` creates a date data type the equivalent of 1st Jan 2020

7. Go back to your Index view and replace the hardcoded rows with the following:
    ```
    @payments.map { payment =>
        <div class="grid-row">
            <div class="column-one-third">
                @payment.date
            </div>
    
            <div class="column-one-third">
                @payment.reference
            </div>
    
            <div class="column-one-third r-align">
                @payment.amount
            </div>
        </div>
    }
    ```

   *  `.map { ... }` **loops** over the list of payments one by one
   * for each payment in the list, it creates a series of `div`s and outputs the date, reference and amount fields from the model.

8. Refresh the page in the browser and you should see the two payments displayed.

## Formatting data

The way that the dates and money values are displayed isn't ideal. A date like `1 January 2020` is more readable than `2020-01-01`, and we should really put a `£` sign and two decimal places for our monetary value.

### Dates
We can use a Play UI helper to format the date.

1. In your view, import:
    ```
    @import uk.gov.hmrc.play.views.formatting.Dates
    ```

2. Then change the line you output the date from:
    ```
    @payment.date
    ```
    to
    ```
    @Dates.formatDate(payment.date)
    ```

3. Refresh the page and the date should be nicely formatted.

### Money

There is another Play UI helper we can use to format the monetary value to display like `£50.00` - i.e. with a currency symbol and two decimal places.

1. Explore in the Play UI library [here](https://github.com/hmrc/play-ui/tree/master/src/main/twirl/uk/gov/hmrc/play/views/formatting) and try and find a suitable method to help format the date
2. Import the object (in a similar way to the Money helper)
3. Use the helper method you found in the same way as with the `Dates.formatDate()` method. You will have to specify the number of decimal places (i.e. 2) as a parameter too.

## Model, View, Controller

So far we've created:

* A **model** called `Payment`
    * This defines the **shape** and **properties** of a Payment, i.e. with a date, reference and amount
* A **view** called `Index.scala.html`
    * This is the design for *how* the `Payment` should be marked up in **HTML** and what it should look like to the user in the browser, AKA the user interface
* A **controller** called `HomeController`
    * This is the **request handler** of the service. It controls the *routing* and the facilitation of creating a list of `Payment`s and passing it into the `Index` view. 

This is a widely-used pattern known as Model, View, Controller or **MVC**.

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/a/a0/MVC-Process.svg/1200px-MVC-Process.svg.png" width="400">

### Why use MVC?
* Faster development and support for parallel programming:
    * Its possible for one person to work on the view, another working on the model and someone else working on the controller, all at the same time
* You can create **multiple** views for **one** model, limiting code duplication as it separates business logic from the display
    * e.g. you could use the same `Payment` model to show the user's *most recent* payment made on an 'Overview' type dashboard
* Modification of the UI does not affect the model
    * You have a nice separation of concerns and can iterate the UI as much as you want without affecting any other components
* Models are supported by many different interfaces, not just HTML
    * You could format your data model into a PDF, JSON API, or even Flash