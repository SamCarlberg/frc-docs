# Making Commands Run

There are three ways to make a command run: using a ``Trigger`` to set up a command to automatically run when some event occurs in the future; immediately running a command via ``Coroutine.fork(Command)``; and configuring a :term:`Mechanism` with a default command to execute when it would otherwise be idle. Each approach is useful in different situations: immediately running a command is useful inside of compositions - when one command runs another - while event-based triggers allow automated behavior without needing to manually check conditions.

## Triggers

Triggers are objects that check the value of a binary condition. Binary signals are either low (``false``) or high (``true``). A signal transition from low to high is called a *rising edge*, and a transition from high to low is called a *falling edge*. Commands can be bound to triggers to run when a rising edge or falling edge is detected. Bindings are created ahead of time to tell the robot how to respond to events when they occur in the future.

Triggers implement the Java ``BooleanSupplier`` interface, and are fully compatible with other areas of the library and WPILib that work with that interface (for example, ``Coroutine.waitUntil(BooleanSupplier)`` will happily accept a ``Trigger`` argument).

.. tab-set-code::

  ```java
  public class Robot extends OpModeRobot {
    private final DigitalInput lowerLimitSwitch = new DigitalInput(1);

    // This trigger will be checked every time the scheduler runs.
    // But it won't do anything until bindings are added.
    public final Trigger atMinLimit = new Trigger(() -> lowerLimitSwitch.get());

    public Robot() {
      // Bind a command to execute whenever the minimum limit is reached.
      atMinLimit.onTrue(Command.print("Min limit reached!").named("Example Command"));
    }
  }
  ```

### Using Triggers to respond to events

There are four standard types of trigger bindings:

``whileTrue(Command)`` - schedules a command on a rising edge, and cancels it on a falling edge. If the command stops while the trigger condition is still true, it will not be restarted; another rising edge will be needed to start the command again.
``retryWhileTrue(Command)`` - schedules a command on a rising edge, and cancels it command on a falling edge. However, unlike ``whileTrue``, if the command stops while the trigger condition is still true, it will automatically be rescheduled.
``onTrue(Command)`` - schedules a command on a rising edge. The command will run until it finishes on its own or is interrupted by another command, even if the trigger condition becomes false in the meanwhile. If the command is still running on the next rising edge
``toggleOnTrue(Command)`` - schedules a command on a rising edge, and cancels it on the next rising edge. If the command stops before the second rising edge, the second rising edge will schedule the command again instead of attempting to cancel a command that's not running.

Falling edge binding variants also exist: ``whileFalse(Command)``, ``retryWhileFalse(Command)``, ``onFalse(Command)``, and ``toggleOnFalse(Command)``.

A fifth binding type, ``multiPress(int, Time)``, allows commands to be bound when a trigger signal has had some minimum number of rising edges within a specific time period. For example, ``Trigger.multiPress(2, Seconds.of(1.5))`` will go high when there have been *at least* two rising edges within the last 1.5 seconds, and will go low when there have been *fewer than* two rising edges within that period; it will remain high as long as the rate of two rising edges per 1.5 seconds is maintained. This allows for commands to be fired when the second press is detected (using ``onTrue``), or to be active as long as there are 2 edges per 1.5 seconds (using ``whileTrue``).

### Modifying Triggers

Triggers may be combined using boolean operators AND and OR. Triggers may also be negated, debounced, and transformed into rising and falling edge detectors. In all cases, the original triggers are unmodified: new ``Trigger`` objects are always created instead.

``Trigger.and(BooleanSupplier)`` creates a new trigger that is only high when *both* signals are high.
``Trigger.or(BooleanSupplier)`` creates a new trigger that is high when *either* signal is high.
``Trigger.negate()`` creates a new trigger that is high when the original signal is low, and is low when the original signal is high.
``Trigger.debounce(Time)`` creates a new trigger that is high when the original signal is high *for at least* the given duration of time.
``Trigger.risingEdge()`` creates a new trigger that is high only for the single loop cycle where the original signal transitions from low to high.
``Trigger.fallingEdge()`` creates a new trigger that is high only for the single loop cycle where the original signal transitions from high to low.

## Manually running a command

TODO