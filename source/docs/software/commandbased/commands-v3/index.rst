# Commands v3 Programming

Command-based programming is a way of writing a program where actions can be defined and configured to execute in response to some event. We call these actions "commands" and the events "triggers". Commands may run other commands to perform more complex actions; these are called "compositions" and are a powerful tool for building sophisticated behavior from simple building blocks. Just like commands outside of compositions, commands inside of compositions can still be configured to run in response to a trigger, but can also be manually scheduled when direct control is desired.

Because multiple commands need to be able to run simultaneously, commands use a :term:`coroutine` to manage concurrency. Coroutines allow commands to say when they have reached a pause point in their work by calling ``Coroutine.yield()``. This pauses the command and will allow another command to run until *it* reaches a pause point, and so on until every running command has been processed. Most importantly, coroutines let us write our command logic using standard Java with ``while`` loops, ``if`` statements, and so on, just with the addition of ``coroutine.yield()`` in our loops to allow other commands to run.

.. warning:: Calling ``coroutine.yield()`` is required to prevent commands from being greedy - if a command never yields, no other commands will be able to run and driver inputs will not be read until the command exits. WPILib will check ``while`` loops at compile-time to ensure that loops inside of command code will yield, and issue a compiler error if any greedy loops are found.

.. note:: Because the Java programming language does not have a concept of coroutines, the ``Coroutine`` class used in the commands library is a custom type created specifically for the library. It can only be used with commands and the command scheduler; it is *not* general-purpose.

All commands have three required attributes:

1. **Requirements**. Commands must declare what mechanisms they control in order to avoid conflicting hardware requests.
2. **Logic**. Commands must *do* something.
3. **Name**. Names appear in telemetry and are crucial for debugging.

.. tab-set-code::

  ```java
  Command command =
    Command.noRequirements(coroutine -> {                       // The first required attribute: stating what mechanisms are required
      System.out.println("The example command is running!");    // The second required attribute: the logic that the command will perform when it runs
    }).named("Example Command");                                // The third required attribute: the name of the command
  ```

Commands prevent conflicting hardware requests from being made by using a requirements system. Every command requires some number of mechanisms, and only one running command may require a particular mechanism at a time. For example, if a running command requires an ``Arm`` mechanism, then no other commands may use the arm at the same time. If another command starts that needs the arm, then the existing command will be canceled to allow the new command to run.

We recommend users define commands using the builders provided by the ``Command`` and ``Mechanism`` classes. The builders keep command code dense, which typically helps with readability, and allows user code to hide direct hardware access from other classes, which helps avoid bugs and dangerous robot behavior. However, users who are more comfortable with object-oriented programming may choose to write class-based commands and implement the ``Command`` interface directly, but will need to handle requirements with care.

.. tab-set-code::

  ```java
  Arm arm = new Arm();
  Command armUp = arm.up();
  driverController.faceLeft().whileTrue(armUp);

  class Arm implements Mechanism {
    public Command up() {
      return run(coroutine -> {
        // ... logic to move the arm up to a predetermined angle ...
      }).named("Arm Up");
    }
  }
  ```


