# Using Lambda Functions

Lambda functions are a way of passing code to a function for *that* function to execute when it needs it. Java allows any object that could be an interface with a single method (a so-called "functional interface") to be written instead using a lambda function to improve readability and performance.

.. tab-set-code::

  ```java-anonymous-class
  List<String> strings = new ArrayList<>();
  strings.add("first");
  strings.add("second");
  strings.add("third");

  strings.forEach(new Consumer<String>() {
    @Override
    public void accept(String string) {
      System.out.println(string);
    }
  });

  // Output
  first
  second
  third
  ```

  ```java-full-lambda
  // The "new Consumer<String>()" and anonymous class can be written with a lambda function instead:
  strings.forEach((String string) -> {
    System.out.println(string);
  });
  ```

  ```java-inferred-type-lambda
  // For lambda functions with a single argument, the type can be omitted entirely because the compiler
  // can figure out what it's supposed to be.
  strings.forEach(string -> {
    System.out.println(string);
  });
  ```

  ```java-single-line-lambda
  // If the lambda function just does a single thing, the curly braces can be removed
  // and the entire lambda function can be written on a single line
  strings.forEach(string -> System.out.println(string));
  ```

  ```java-discarded-lambda-parameter
  // If the lambda function doesn't use a parameter, we can replace its name with a single `_`
  // to tell the compiler we don't use it.
  strings.forEach(_ -> System.out.println("Got another string, but I won't tell you what it is"));
  ```

  ```java-method-reference
  // And if we just want to pass the input to another method, we can tell Java
  // what method to use (in this case, `System.out.println`) instead of manually
  // calling it ourselves.
  strings.forEach(System.out::println);
  ```

## How commands use lambda functions

Command builders are based around providing a lambda function for the logic that the command will run.  ``Command.noRequirements()`` and ``Command.requiring(...).executing()`` both accept lambda functions for the command logic. These lambda functions accept a single ``Coroutine`` object and perform whatever command logic is needed. The optional ``whenCanceled()`` builder method also a accepts a lambda function, but this one doesn't have any arguments.

.. tab-set-code::

  ```java
  Command.noRequirements((Coroutine coroutine) -> {
      System.out.println("I'm in a command!");
    })
    .whenCanceled(() -> {
      System.out.println("Command was canceled");
    });
  ```

  ```java-anonymous-class
  Command.noRequirements(new Consumer<Coroutine>() {
      @Override
      public void accept(Coroutine coroutine) {
        System.out.println("I'm in a command!");
      }
    })
    .whenCanceled(new Runnable() {
      @Override
      public void run() {
        System.out.println("Command was canceled");
      }
    });
  ```

  ```java-shorthand-lambdas
  Command.noRequirements(_ -> System.out.println("I'm in a command!"))
    .whenCanceled(() -> System.out.println("Command was canceled"));
  ```