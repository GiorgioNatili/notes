Getting Started with Robolectric, JBehave and Maven
======
This post is **not** a complete guide about __Robolectric_ or _JBehave_, it is just a quick guide to start to use TDD in a more productive way with Android. In order to complete the steps described in the **Project Setup** section you have to install in your system [**Eclipse Luna**](https://projects.eclipse.org/releases/luna) (you can use also different versions of Eclipse, Luna is the one I actually used for this tutorial) and [**Brew**](http://brew.sh/) in order to easily install [**Maven**](http://maven.apache.org/) on your system.

### Intro
One of the biggest concerns when working with Android is the speed of the emulator. If you are lucky it takes from 40 to 60 seconds to run your app on the emulator or on a physical device, with the technology stack we are going to discuss in this tutorial it's possible to significantly reduce this amount of time because you will be able to run the tests without waiting for the emulator. Last but not least using the Maven plugin [_Emma_](http://emma.sourceforge.net/) you can obtain accurate reports about your tests coverage outside and [inside Eclipse](https://wiki.eclipse.org/Code_Coverage_with_Emma#Setup). 

### TDD
The acronym _TDD_ refers to the term Test-driven development. Test-driven development is a software development process that relies on the repetition of a very short development cycle where the developer  first writes an automated test case that defines a desired feature, then produces the minimum amount of code to pass that test, and finally refactors the new code until the feature is not completely implemented. To successfully implement this process the cycle has to be fast and easy to execute.
Well-written automated tests provide a working specification of your functional code and as a result tests effectively become a significant portion of your technical documentation.
Most of the time developers use the term TDD to indicate _unit testing_. Unit testing is a software testing method by which individual units of source code, sets of one or more computer program modules together with associated control data, usage procedures, and operating procedures, are tested to determine whether they are fit for use (_wikipedia_). 
Writing uint test first is a pattern well recognized in the modern software development industry. 
However apply blindly this pattern, in my opinion, is not the best approach because it can impact negatively your software design. When you focus only on testing then you try to avoid time consuming operations such as the access to the filesystem or to an external API mocking most of them and adding an unneeded layer of complexity. A probably better approach is to decide first what to test focusing on the automated tests that can improve the end user experience. An automated test can improve the end user experience because it can prevent that a new feature breaks an existing _behavior_ of the software.

### BDD and JBehave
The acronym _BDD_ refers to the term Behavior-driven development. It shares the same process of TDD and in fact, also when using BDD, you let the tests to drive you during the development. The main difference between these techniques is what you test, in fact with BDD you test the behavior of a software in a specific _scenario_. Focusing on the behaviors of the software teams concentrate their efforts on identifying, understanding, and building valuable features that matter and that can be tested in a real scenario. When defining an automated test that describe the behavior of a software becomes natural to use a language that is not understandable just by developers, for instance take a look to this written using the [Gherkin syntax](https://github.com/cucumber/cucumber/wiki/Gherkin) 

> Feature: Withdrawal with balance <br>
> Given my bank account is in credit, and I made no withdrawals recently, <br>
> When I attempt to withdraw an amount less than my card's limit, <br>
> Then the withdrawal should complete without errors or warnings

This test is understandable by developers, testers, project managers and product people, Behavior-driven development is one of the easiest way to ensure that all stakeholders are on the same page and understand how to build the right software rather than how to build the software right.

[_JBehave_](http://jbehave.org/) is an open source BDD framework originally written by Dan North, the inventor of BDD. It is strongly integrated into the JVM world, and widely used by Java development teams wanting to implement BDD practices in their projects. The main advantage of using JBehave is that it allows to write a test case using plain text that is then converted to a Java class. JBehave makes the transition from natural language style BDD-tests to Java methods incredibly quick also because it offers integration tools both for [Eclipse](http://jbehave.org/eclipse-integration.html) and for [IntelliJ IDEA](https://github.com/witspirit/IntelliJBehave).

### Robolectric
[_Robolectric_](http://robolectric.org/) is a unit test framework that de-fangs the Android SDK jar so you can test-drive the development of your Android app inside the JVM on your workstation. One of the most important features of Robolectric in fact is that it mocks part of the Android framework contained in the android.jar file, this allows  to run Android tests directly on the JVM with the JUnit 4 framework rather then waiting for the device emulator or for a real device. Robolectric is a highly configurable framework in fact it's possible to specify which output to use, how to handle dependencies, etc. (for more details check the online [guide](http://robolectric.org/configuring/)) and it works with the most common automation tools like [Maven](http://maven.apache.org/) or [_Gradle_](https://www.gradle.org/).
In order to start using Robolectric with Maven you can clone this [template project](https://github.com/robolectric/deckard-maven) from GitHub, if you prefer to use Gradle then clone [this template](https://github.com/robolectric/deckard-gradle) instead.

### Maven
Maven is a build automation tool used primarily for Java based projects. In Maven an XML file describes the software project being built, its dependencies on other external modules and components, the build order, directories, and required plug-ins. Practically speaking the XML files contains:

* A detailed description about how the software is built 
* An exhaustive list of the software dependencies

Installing Maven with Brew it's pretty easy, check first that the system is updated by entering the following command in a terminal window `$ brew doctor` (the worst case scenario is that installed brew _formulas_ will be updated) and then type the command `$ brew install maven`. In order to verify that Maven is installed correctly type the command `$ mvn --version` and check that the installed version matches the most recent [Maven release](http://maven.apache.org/release-notes-all.html). 
In order to create a Maven project is enough to type the command `$ mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false`. The project folder contains an XML file named `pom.xml`, it is the core of a project's configuration in Maven. It is a single configuration file that contains the majority of information required to build a project. 

### Project Setup
In order to start to use Robolectric, JBehave and Maven with Eclipse it's mandatory to install the Android Developer Tools (i.e. ADT) [plugin](http://developer.android.com/sdk/installing/installing-adt.html) for Eclipse and eventually update the Android SDK. Once the plugin is installed Eclipse should be restarted and then it's possible to download the last Android SDK or to add an existing SDK installation to the plugin.
Robolectric currently supports the _Android API Level 18_ (i.e. Android 4.3 aka Jelly Bean), in order to run it correctly double check that the API is part of the current Android SDK installation.
The setup of a Robolectric-JBehave-Maven Android project in Eclipse can be summarized as following:

1. If not yet installed, install the [Android for Maven Eclipse Plugin](http://rgladwell.github.io/m2e-android/)
2. Install the JBehave Eclipse [plugin](http://jbehave.org/reference/eclipse/updates/)
3. Download the Robolectric JAR with all the dependencies available on [sonatype.org](https://oss.sonatype.org/#nexus-search;gav~org.robolectric) and save it in a folder accessible from your Eclipse workspace
4. Download the JBehave core 3.9.5 JAR available on [mvnrepository.com](http://mvnrepository.com/artifact/org.jbehave/jbehave-core/3.9.5) and save it in the same folder that contains the   Robolectric JAR
5. Download the JBehave Maven plugin available on [maven.org](http://search.maven.org/#search|ga|1|jbehave-maven-plugin) and save it with the previous downloaded JARs
6. Clone the repository available on the Robolectric GitHub [web site](https://github.com/robolectric/deckard-maven) and execute in a terminal window the command `$ deckard-maven/setup.sh`
7. Copy the `deckard-maven` folder in the Eclipse workspace executing the following command in a terminal window `$ cp -R deckard-maven "path/to/the/eclipse-workspace"` and eventually rename it into something more appropriate executing the following command `$ mv deckard-maven roboletric-getting-started` in the Eclipse workspace directory
8. Using a terminal window move to the folder that contains the project and execute the command `$ mvn clean install` in order to verify that the configuration is correctly working
9. Open Eclipse and import the existing Maven project ignoring the error messages
10. Select the project, right click and open the `Properties`
11. In the `Properties` panel select `Java Build Path` then the tab `libraries` and click `Add External JARs...`; when prompted select the JARs previously downloaded click `open` and then `OK`
12. A message starting with "Plugin execution not covered by lifecycle configuration:" should be still in place in the `Problems` tab in Eclipse, right click on it and select `Quick Fix`; when prompted select the option `Permanently mark the goal ... as ignored in Eclipse built`
13. Select again the project, right click and from the `Maven` menu item select the option `Update Project...`
14. Right click again on the project and from the `Run As...` menu items select the option `Run Configurations` and create a new `Maven Build` one; specify in the goals `clean test` and run the project

The project will run some tests without using the emulator, even more now it's possible to use Eclipse as development IDE and run the tests from it or from the terminal. With this configuration it's also possible to execute Emma (i.e. `$ mvn emma:emma`)and other Maven plugins in order to get as much information as possible from the tests. 

### What's Next
In the next post I would like to address another common need that is to add Robolectric, JBehave and Maven as a test framework to an existing Android project without applying any significant change to the existing project.
If you want to discuss this post or provide any suggestion for the next post let's talk over twitter [@giorgionatili](https://www.twitter.com/giorgionatili), stay tuned and thanks for reading!

### Resources
There are a lot of resources available on the web to improve your knowledge about Robolectric, JBehave, Maven, TDD and Android. The following links are a good starting point (this list was created in Jan 2015):

* [Testing Android Apps with Robotium and JBehave](http://www.softwaretestingmagazine.com/knowledge/testing-android-apps-with-robotium-and-jbehave/)
* [BDD vs TDD in Android](http://mobilebytes.wordpress.com/2011/04/07/bdd-vs-tdd-in-android)
* [Behavior-Driven Development (BDD) with JBehave, Gradle, and Jenkins
](http://www.javacodegeeks.com/2012/09/behavior-driven-development-bdd-with.html)
* [Robolectric, Quick Start for Eclipse](http://robolectric.org/eclipse-quick-start/)
* [Robolectric Repository](https://github.com/robolectric/robolectric)
* [Emma, the code coverage reporting tool](http://emma.sourceforge.net/maven-emma-plugin/)
* [Maven to Eclipse guide](https://www.eclipse.org/m2e/documentation/m2e-execution-not-covered.html)
* [Unit Testing Android Apps With Robolectric and Eclipse](http://www.peterfriese.de/unit-testing-android-apps-with-robolectric-and-eclipse/)
* [Robolectric Installation for Unit Testing](https://github.com/codepath/android_guides/wiki/Robolectric-Installation-for-Unit-Testing)
* [JBehave Maven Goals](http://jbehave.org/reference/stable/maven-goals.html)
* [Using the M2Eclipse Maven Plugin in Eclipse](https://wiki.openmrs.org/display/docs/Using+the+M2Eclipse+Maven+Plugin+in+Eclipse)
* [The Secret Ninja Cucumber Scrolls](http://collection.openlibra.com.s3.amazonaws.com/pdf/cuke4ninja-2011-03-16.pdf?AWSAccessKeyId=AKIAIGY5Y2YOT7GYM5UQ&Signature=09WGDgslpT0535GSDBxO5WkSBus%3D&Expires=1420323590)
* [The Android manifest explained](https://www.cs.cmu.edu/~srini/15-446/android/android-sdk-linux_x86-1.0_r2/docs/devel/bblocks-manifest.html)
