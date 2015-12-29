# DaggerMock
A JUnit rule to easily override Dagger 2 objects

Override an object managed by Dagger 2 is not easy, you need to define a TestModule and, if you want
to inject your test object, a TestComponent.

Using a `DaggerMockRule` it's possible to override easily the objects defined in a Dagger module:

```java
public class MainServiceTest {

    @Rule public DaggerMockRule<MyComponent> mockitoRule = new DaggerMockRule<>(MyComponent.class, new MyModule())
            .set(new DaggerMockRule.ComponentSetter<MyComponent>() {
                @Override public void setComponent(MyComponent component) {
                    mainService = component.mainService();
                }
            });

    @Mock RestService restService;

    @Mock MyPrinter myPrinter;

    MainService mainService;

    @Test
    public void testDoSomething() {
        when(restService.doSomething()).thenReturn("abc");

        mainService.doSomething();

        verify(myPrinter).print("ABC");
    }
}
```

In this example
[MyModule](https://github.com/fabioCollini/DaggerMock/blob/master/app/src/main/java/it/cosenonjaviste/daggermock/demo/MyModule.java)
contains three methods to provide `RestService`, `MyPrinter` and `MainService` objects.
The `DaggerMockRule` rule dynamically creates a new module that override `MyModule`, it returns the mocks
for `restService` and `myPrinter` defined in test instead of the real objects.

## Espresso support

A `DaggerMockRule` can be also used in an Espresso test:

```java
public class MainActivityTest {

    @Rule public DaggerMockRule<MyComponent> daggerRule = new DaggerMockRule<>(MyComponent.class, new MyModule())
            .set(new DaggerMockRule.ComponentSetter<MyComponent>() {
                @Override public void setComponent(MyComponent component) {
                    App app = (App) InstrumentationRegistry.getInstrumentation().getTargetContext().getApplicationContext();
                    app.setComponent(component);
                }
            });

    @Rule public ActivityTestRule<MainActivity> activityRule = new ActivityTestRule<>(MainActivity.class, false, false);

    @Mock RestService restService;

    @Mock MyPrinter myPrinter;

    @Test
    public void testCreateActivity() {
        when(restService.doSomething()).thenReturn("abc");

        activityRule.launchActivity(null);

        verify(myPrinter).print("ABC");
    }
}
```

## Custom rules

It's easy to create a `DaggerMockRule` subclass to avoid copy and paste and simplify the test code:

```java
public class MyRule extends DaggerMockRule<MyComponent> {
    public MyRule() {
        super(MyComponent.class, new MyModule());
        set(new DaggerMockRule.ComponentSetter<MyComponent>() {
            @Override public void setComponent(MyComponent component) {
                App app = (App) InstrumentationRegistry.getInstrumentation().getTargetContext().getApplicationContext();
                app.setComponent(component);
            }
        });
    }
}
```

The final test uses the rule subclass:

```java
public class MainActivityTest {

    @Rule public MyRule daggerRule = new MyRule();

    @Rule public ActivityTestRule<MainActivity> activityRule = new ActivityTestRule<>(MainActivity.class, false, false);

    @Mock RestService restService;

    @Mock MyPrinter myPrinter;

    @Test
    public void testCreateActivity() {
        when(restService.doSomething()).thenReturn("abc");

        activityRule.launchActivity(null);

        verify(myPrinter).print("ABC");
    }
}
```

## License

    Copyright 2016 Fabio Collini

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
