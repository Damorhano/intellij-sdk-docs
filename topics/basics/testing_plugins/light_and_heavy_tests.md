<!-- Copyright 2000-2023 JetBrains s.r.o. and contributors. Use of this source code is governed by the Apache 2.0 license. -->

# Light and Heavy Tests

<link-summary>Introduction to light tests reusing a single project for multiple tests, and heavy tests creating a new project for each test.</link-summary>

Plugin tests run in a real, rather than mocked, IntelliJ Platform environment and use real implementations for most application and project [services](plugin_services.md).

Loading and initializing all the project components and services for a project to run tests is a relatively expensive operation, and we want to avoid doing it for each test.
Dependently on the loading and execution time, we make a difference between *light* tests and *heavy* tests available in the IntelliJ Platform test framework:

* *Light* tests reuse a project from the previous test run when possible.
* *Heavy* tests create a new project for each test.

Light and heavy tests use different base classes or fixture classes, as described below.

> Because of the performance difference, we recommend plugin developers to write *light* tests whenever possible.
>
{style="note"}

## Light Tests

The standard way of writing a light test is to extend one of the following classes:

<tabs>
<tab title="Default">

[`BasePlatformTestCase`](%gh-ic%/platform/testFramework/src/com/intellij/testFramework/fixtures/BasePlatformTestCase.java) for tests that don't have any dependency on Java functionality.

For 2019.2 and earlier, use [`LightPlatformCodeInsightFixtureTestCase`](%gh-ic%/platform/testFramework/src/com/intellij/testFramework/fixtures/LightPlatformCodeInsightFixtureTestCase.java).

</tab>

<tab title="Plugins using Java PSI">

For tests that require the [Java PSI](idea.md#java) or related functionality:
- [`LightJavaCodeInsightFixtureTestCase`](%gh-ic%/java/testFramework/src/com/intellij/testFramework/fixtures/LightJavaCodeInsightFixtureTestCase.java) for JUnit3
- [`LightJavaCodeInsightFixtureTestCase4`](%gh-ic%/java/testFramework/src/com/intellij/testFramework/fixtures/LightJavaCodeInsightFixtureTestCase4.kt) for JUnit4 (2021.1 and later)
- [`LightJavaCodeInsightFixtureTestCase5`](%gh-ic%/java/testFramework/src/com/intellij/testFramework/fixtures/LightJavaCodeInsightFixtureTestCase5.kt) for JUnit5 (2021.1 and later)

For 2019.2 and earlier, use [`LightCodeInsightFixtureTestCase`](%gh-ic%/java/testFramework/src/com/intellij/testFramework/fixtures/LightCodeInsightFixtureTestCase.java).

> See [](testing_faq.md#how-to-test-a-jvm-language) on how to set up your test environment to obtain the required _Mock JDK_ automatically.

</tab>
</tabs>

### LightProjectDescriptor

When writing a light test, you can specify the project's requirements that you need to have in your test, such as the module type, the configured [SDK](sdk.md), [facets](facet.md), [libraries](library.md), etc.
You do so by extending the [`LightProjectDescriptor`](%gh-ic%/platform/testFramework/src/com/intellij/testFramework/LightProjectDescriptor.java) class and returning your project descriptor (usually stored in `static final` field) from `getProjectDescriptor()`.

Before executing each test, the project instance will be reused if the test case returns the same project descriptor as the previous one or recreated if the descriptor is different (`equals() = false`).

## Heavy Tests

> If you need to set up a multi-module project for your tests, you **must** write a heavy test.
>
{style="note"}

> In 2019.3, `PlatformTestCase` has been renamed to [`HeavyPlatformTestCase`](%gh-ic%/platform/testFramework/src/com/intellij/testFramework/HeavyPlatformTestCase.java) reflecting its "heavy test" characteristics.
>
{style="note"}

The setup code for a multi-module Java project looks something like that:

```java
TestFixtureBuilder<IdeaProjectTestFixture> projectBuilder =
        IdeaTestFixtureFactory.getFixtureFactory().createFixtureBuilder(getName());

// Repeat the following line for each module
JavaModuleFixtureBuilder moduleFixtureBuilder =
        projectBuilder.addModule(JavaModuleFixtureBuilder.class);

myFixture = JavaTestFixtureFactory.getFixtureFactory()
        .createCodeInsightFixture(projectBuilder.getFixture());
```
