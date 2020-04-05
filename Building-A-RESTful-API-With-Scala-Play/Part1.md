# Part One - Setup

## Pre-requisites
These should all be installed as part of your laptop setup.
* Scala
* Mongo
* git
* IntelliJ IDEA
* SBT

## Generating a Play/Scala application seed 
We'll first generate a template for your Play application that gives you the initial setup you need.

From the [Lightbend Developer Hub](https://developer.lightbend.com/start/):
1. Select **Play - create a reactive web application project**

2. Select **Play Scala Seed**

3. Give it a sensible name and hit **Create project for me**

4. Extract the .zip file and move the folder to your workspace

5. From **IntelliJ**:
    * File → Open → find your project
    * Check the project JDK is setup to Java 1.8
    * Ensure library sources is ticked
    * Hit OK

Take a few minutes to get familiar with what has been created for you. In particular:
* `app/controllers/HomeController`
* `index.scala.html` & `main.scala.html`
* `conf/routes`
* `test/controllers/HomeControllerSpec`
* `build.sbt`
* `.gitignore`

## Running the new Application
1. Bring up the Terminal window in IntelliJ and run `sbt run`

2. Your project will compile and run by default on port `9000`

3. Navigate to `localhost:9000` in your browser and you should see a greeting message from Play

4. Where is this text coming from? Use the files in the `controllers` & `views` packages to investigate. More in-depth frontend development will be covered in a separate tutorial.

5. Try customising both the header and page title text in those files to your own message

6. Use `control + C` in the terminal to stop the server running 

## Running Tests
1. Run the command `sbt test` in the terminal. This command runs all tests that were set up by the Play application seed, i.e. in `test/controllers/HomeControllerSpec`.

2. You should get failed tests after changing the page title text above

3. Alter the tests so that they now pass

**Note: this isn't [Test-Driven Development](https://www.agilealliance.org/glossary/tdd/#q=~(infinite~false~filters~(postType~(~%27page~%27post~%27aa_book~%27aa_event_session~%27aa_experience_report~%27aa_glossary~%27aa_research_paper~%27aa_video)~tags~(~%27tdd))~searchTerm~%27~sort~false~sortDirection~%27asc~page~1))! Consider how you could have done the above tasks using a TDD approach.**

## Set up Git & GitHub Repository
We want to track changes to the project using git.

1. To set it up for your new repo, bring up the Terminal window in IntelliJ and run `git init`

2. Run `git status`, and you can now see there are various files git has picked up but is not yet tracking any changes and only recognises a whole bunch of new folders and files.

3. Run `git add .` to add all files in red to a 'staging area' in preparation for a final commit

4. To make a commit, run `git commit -m "Initial commit"`

5. Running `git status` again will show that there are no changes left to commit.

6. Go to your GitHub account → Your repositories → Create new repository
    * Give it the same name as your local project
    * Ensure it is public
    * Ensure 'Initialise with README' is unticked

7. Back to the command line:
    * We need to tell git where the remote GitHub repository is, so that we can push up changes. Run the following, substituting with your GitHub username and newly created project name
    * `git remote add origin git@github.com:<username>/<repo-name>.git`
    * e.g. `git remote add origin git@github.com:miranda-hawkes/play-scala-seed.git`

8. Run `git remote -v` to check the remote is correct

9. Push your new commit up to master `git push origin master`

10. Go to your new project on GitHub to check everything pushed up okay

11. Hit the button **Add a README**

GitHub supports making direct changes to your repo within the UI itself. This isn't generally recommended as you're unable to run tests to ensure your changes are working okay, but we can use this feature just to create a dummy README.md file:
1. Add a commit comment describing what you're doing

2. Commit the new file

3. Back to the command line, run git pull to pull down the new README

4. You should get an error from git, informing you that you need to give it an upstream origin for the master branch

5. Using the help text git gives you, **try and fix this error**

6. Run git pull again and check you now see the README.md file

**From this point onwards, we'll now use branches to make changes, rather than committing straight to master (big no no!)**

## Checkout A New Branch
To create and checkout a new branch:

```
git checkout -b <branch-name>
```
Keep the branch name short and sweet. If your branch on real microservice relates to a JIRA ticket, use the ID of the ticket as the branch name.

## Installing Dependencies
As mentioned earlier, we're going to use Reactive Mongo in our project to make basic requests to a database.
We need to use a separate library for this, and we'll be using the HMRC version that is used widely across MDTP. We'll also add a useful testing library.

We're going to first change the version of Play framework we're using so that it works with simple-reactivemongo. We'll also need to change a couple other dependency versions.

In your project:

1. Find the version of `sbt-plugin` and change it to **2.6.23**

2. Find the `scala` version and change it to **2.11.11**

3. Find the `scalatestplus-play` version and change it to **3.1.2**

4. Find the `sbt` version and change it to **0.13.18**

5. Run `sbt test` to check everything still works and your app downloads the versions correctly

To add a new library dependency to your project, open up plugins.sbt

1. Add the following resolver to the file. This tells sbt where to find any public HMRC libraries:
```
resolvers += "HMRC Releases" at "https://dl.bintray.com/hmrc/releases"
```

2. Add the new library and version to the `build.sbt` file, similar to how scalatestplus-play is added:
```
libraryDependencies += "uk.gov.hmrc" %% "simple-reactivemongo" % "7.26.0-play-26"
libraryDependencies += "uk.gov.hmrc" %% "hmrctest" % "3.9.0-play-26" % Test
```

3. Update the `lazy val root` in `build.sbt`:
```
lazy val root = (project in file("."))
  .enablePlugins(PlayScala)
  .settings(resolvers ++= Seq(
    Resolver.bintrayRepo("hmrc", "releases"),
    Resolver.jcenterRepo
  ))
```

4. Some extra configuration is needed for the reactive mongo library to know how to connect to your local Mongo database and to give it a name. Add the following to application.conf and adding in the name of your app:
```
mongodb {
  uri = "mongodb://localhost:27017/replace-with-your-app-name-here"
}
```
5. Run tests again and check everything is working okay

6. Add the changed files to git, do a git commit briefly detailing what you have changed, then push the branch up to GitHub (hint: `git push origin <branch-name>`)

7. Navigate to GitHub and find your new branch and view the changes you have made

## [Part 2](Part2.md) 