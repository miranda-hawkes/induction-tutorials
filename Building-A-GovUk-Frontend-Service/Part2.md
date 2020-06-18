# Part Two - A Basic Gov UK Style Page

Here we'll create a basic [Gov UK Design System](https://design-system.service.gov.uk/) style page with support for multiple languages.

## Create A New Controller

1. Delete the following files (so we can start fresh):
    * `app/views/index.scala.html`
    * `app/views/main.scala.html`
    * `test/controllers/HomeControllerSpec`

2. Change the `HomeController` so it extends `FrontendController` instead of `BaseController`. This will give us access to a number of functions for our actions.

    * You'll notice there is now an error stating that MessagesControllerComponents must be implemented - this is a dependency of the FrontendController trait.
    * This class contains values necessary in most frontend controllers, such as multi-language support. To remedy this, change the class signature to the following:
   
      ```scala
      @Singleton
      class HomeController @Inject()(mcc: MessagesControllerComponents) extends FrontendController(mcc) {
      ```
   
    * We won't go much into dependency injection (DI) as part of this, but using DI you inject **objects needed by a class**, typically through a constructor
3. Import `uk.gov.hmrc.play.bootstrap.controller.FrontendController` when prompted

4. Write a Scala method in the controller called index() that simply returns TODO
    
    ```scala
    def index: Action[AnyContent] = TODO
    ```
    * `TODO` is a Play feature that supplies default code for controller actions that haven't been completed. It is a useful way of keeping your app functioning while building features incrementally.
    * `TODO` just returns a HTML page with 'Not implemented' text on
    
5. Before we can see the TODO page in our browser, we need to add an app route that references the new controller and method. In the routes file, update the existing route to the following:
   
    ```
    GET     /     controllers.HomeController.index
    ```

6. Run your application and hit the new route at [localhost:9000](https://localhost:9000). You should see the TODO text.

### Add a Gov UK Template

1. Create a new file in the `views` package
    * Right click 'views'
    * New -> HTML File
    * Call it `GovUkTemplate.scala`
    
2. The new file created should have the extension `GovUkWrapper.scala.html`. These types of files are Play's [Twirl templates](https://www.playframework.com/documentation/2.6.x/ScalaTemplates) and allow you to use HTML and Scala code together to create dynamic web pages.

3. Add the following to the file:

    ```scala
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
Don't worry too much about this as we don't need to change it and it is created by default for every new frontend service on MDTP.

### Create a new view

Now we should create a view that uses the GovUkWrapper and displays a simple heading.

1. Create a new Twirl file (using the same steps as when you created `GovUkWrapper.scala.html`) called `Index` and add the following content:

    ```scala
    @import views.html.GovUkWrapper
    
    @this(template: GovUkWrapper)
    
    @()(implicit messages: Messages)
    
    @template(title = "Hello") {
        <h1>Hello!<h1>
    }
    ```
    * We have a dependency on the `GovUkWrapper` component, so we are injecting that as a dependency with `@this()`. Note - this is used for services on Play version 2.6 onwards.
    * The next line is declaring the parameters required for the view. The only one needed is an implicit `Messages` object. More on this later.
    * We then call our template by doing `@template() { ... }`. The round brackets passes in what the title of the page should be, and the curly brackets passes any HTML (or Scala code) you want to be displayed on the body of the page

2. One last thing is to update our controller to call the view:

    ```scala
    class HomeController @Inject()(mcc: MessagesControllerComponents,
                                   indexView: Index) extends FrontendController(mcc) {
    
      def index: Action[AnyContent] = Action { implicit request =>
        Ok(indexView())
      }
    
    }
    ```
    * We inject our new Index view into the controller
    * We update our `index()` action so that it returns a HTTP 200 / OK response, with the `Index` view in the body

3. Run your service again and navigate to `localhost:9000`. It doesn't look too great just yet but we're getting there.

    That's because we don't have a **locally running instance** of the Assets Frontend service, so the browser can't retrieve the CSS & Javascript assets it needs.
    The reason we use a separate service for this is so that every microservice on MDTP doesn't need to store or maintain its own UI assets, instead we rely on one microservice to host them for us. 

4. Run the following command to start Assets Frontend via Service Manager: `sm --start ASSETS_FRONTEND -f`.
5. Refresh the page and you should see a perfectly formatted Gov UK Style page with a welcome message!

## Messages Files

**What if a user of our service couldn't read English?** At the moment, if we wanted to display a Spanish version of our page for example, we'd have to create a whole new almost identical view, but with the word 'Hola' instead of 'Hello'.

Play Framework has **multi language support**, allowing you to put all the text displayed across the whole service in one place. If you want to add support for another language, you add another messages file containing all the translations.

This is where that `Messages` object comes in.

1. Open up `conf/messages` and add in the following:
   ```
   index.title = This is the nav title
   index.pageHeading = This is the page heading
   ```

2. In `Index.scala.html`, update the **two** places where the word 'Hello' was written:
  ```scala
  @template(title = messages("index.title")) {
      <h1>@messages("index.pageHeading")<h1>
  }
  ```

Now refresh the page in your browser. The text has changed to what you specified in the messages file!

Obviously we're only catering for English still, but if you created a duplicate messages file called `messages.es` (ES = Español) and replaced the text with the Spanish equivalent, and a user hit the page with a certain browser cookie (`PLAY_LANG`) set to Spanish (`es`), it would render the page in Spanish.

* On most MDTP services we implement an English and Welsh toggle button on pages so users can easily change their preference
* For this reason, **we always use messages files for text on views** - don't write the English version directly into views!

## [Part 3](Part3.md) 