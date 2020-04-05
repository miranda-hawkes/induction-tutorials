# Part Two - A Basic Gov UK Style Page

Here we'll create a basic Gov Uk style page with support for multiple languages.

## Create A New Controller

Delete the following files:
* `app/views/index.scala.html`
* `app/views/main.scala.html`
* `test/controllers/HomeControllerSpec`

1. Change the `HomeController` so it extends `FrontendController` instead of `BaseController`. This will give us access to a number of functions for our actions

    * You'll notice there is now an error stating that MessagesControllerComponents must be implemented - this is a dependency of the FrontendController trait. This class contains values necessary in most frontend controllers, such as multi-language support. To remedy this, change the class signature to the following:
    ```
    @Singleton
    class HomeController @Inject()(mcc: MessagesControllerComponents) extends FrontendController(mcc) {
    ```
    * We won't go much into dependency injection (DI) as part of this, but using DI you inject the objects needed by a class typically through a constructor
    * Import `uk.gov.hmrc.play.bootstrap.controller.FrontendController` when prompted

2. Write a Scala method in the controller called index() that simply returns TODO
    ```
    def index: Action[AnyContent] = TODO
    ```
    * TODO is a Play feature that is essentially a default page for controller actions that haven't been completed. It is a useful way of keeping your app functioning while building functionality incrementally.

3. Before we can see the TODO page, we need to add an app route that references the new controller and method. In the routes file, update the existing route to the following:
    ```
    GET     /index     controllers.HomeController.index
    ```

4. Run your application and hit the new route. You should see the TODO text.

### Add a Gov Uk Template

1. Create a new file in the `views` package
    * Right click 'views'
    * New -> HTML File
    * Call it GovUkTemplate.scala
    
2. The new file created should have the extension `GovUkWrapper.scala.html`. These types of files are Play's [Twirl templates](https://www.playframework.com/documentation/2.6.x/ScalaTemplates) and allow you to use HTML and Scala code together to create dynamic web pages.

3. Add the following to the file:

    ```
    @import views.html.layouts.GovUkTemplate
    @import uk.gov.hmrc.play.views.html.layouts._
    @import play.twirl.api.HtmlFormat
    
    @this(govUkTemplate: GovUkTemplate,
          head: Head,
          headerNav: HeaderNav,
          mainContent: MainContent,
          footerLinks: FooterLinks,
          article: Article)
    
    @(title: String)(mainBodyContent: Html)(implicit messages: Messages)
    
    @headContent = {
      @head(linkElem = None, headScripts = None)
    }
    
    @insideHeader = {
      @headerNav(navTitle = None, navTitleLink = None, showBetaLink = false, navLinks = None)
    }
    
    @footerLinksContent = {
      @footerLinks(
        additionalLinks = None,
        euExitLinks = None,
        accessibilityFooterUrl = None
      )
    }
    
    @content = {
      @mainContent(article(mainBodyContent))
    }
    
    @govUkTemplate(title = Some(title), bodyClasses = None)(headContent,
                                                            HtmlFormat.empty,
                                                            insideHeader,
                                                            HtmlFormat.empty,
                                                            HtmlFormat.empty,
                                                            Some(footerLinksContent),
                                                            true)(content)
    ```

There is a lot going on here, but it is basically dealing with some mandatory UI elements needed by the HMRC Gov UK Template.
Don't worry too much about this as we don't need to change it and it is created by default for every new frontend service.

### Create a new view

Now we should create a view that uses the GovUkWrapper and displays a simple heading.

Create a new Twirl file called `Index` and add the following content:

```
@import views.html.GovUkWrapper

@this(template: GovUkWrapper)

@()(implicit messages: Messages)

@template(title = "Hello") {
    <h1>Hello!<h1>
}
```
* We have a dependency on the `GovUkWrapper` component, so we are injecting that as a dependency with `@this()`. Note - this is used for services on Play version 2.6 onwards.
* The next line is declaring the parameters required for the view. The only one needed is an implicit `Messages` object. More on this later.
* We then call our template by doing `@template() { ... }`. The round brackets passes in what the title of the page should be, and the curly brackets passes any HTML you want to be displayed on the body of the page

One last thing is to update our controller to call the view:

```
class HomeController @Inject()(mcc: MessagesControllerComponents,
                               indexView: Index) extends FrontendController(mcc) {

  def index: Action[AnyContent] = Action { implicit request =>
    Ok(indexView())
  }

}
```
* We inject our new Index view into the controller
* We update our `index()` action so it returns a HTTP 200 / OK response, with the `Index` view in the body

Run your service again and navigate to `localhost:9000/index`. It doesn't look too great just yet.

That's because we don't have a locally running instance of the Assets Frontend service, so the browser can't retrieve the CSS & Javascript assets it needs.
The reason we need a service for this is so that every microservice doesn't need to store or maintain its own UI assets, instead we rely on one microservice to host them for us. 

Run the following command to start Assets Frontend via Service Manager: `sm --start ASSETS_FRONTEND -f`.

Refresh the page and you should see a perfectly formatted Gov UK Style page with a welcome message!

## Messages Files

Play Framework has multi language support, allowing you to put all possible bits of text needed for every page, in multiple different languages in one place.
At the moment, if we wanted to display a Welsh version of our page, we'd have to create a whole new identical view which isn't ideal.

This is where that `Messages` object comes in.

If you open up `conf/messages`, add in the following:
```
index.title = This is the nav title
index.pageHeading = This is the page heading
```

Then in `Index.scala.html`, update where you call the template:
```
@template(title = messages("index.title")) {
    <h1>@messages("index.pageHeading")<h1>
}
```

Now refresh the page in your browser. The text has changed to what you specified in the messages file!

Obviously we're only catering for English still, but if you created a duplicate messages file called `messages.cy` and replaced the text with the Welsh equivalent, and a user hit the page with a certain browser cookie (`PLAY_LANG`) set to Welsh (`cy`), it would render the page in Welsh.
On most MDTP services we implement an English/Welsh toggle button on pages so users can easily change their preference. 
For this reason, we always use messages files for text on views - don't write the English version directly into views.

## [Part 3](Part3.md) 