# Creating Commands

Commands can be created in one of three ways:

1. Using a :term:`factory method` on a :term:`Mechanism` object
2. Using a static factory method in from the ``Command`` interface
3. Creating a class that implements the ``Command`` interface (this approach is only recommended for very complex logic or for programmers who are uncomfortable with :term:`lambda functions`)

## Where to put Commands

A key idea in the commands framework is that of requirements: every command requires some number of mechanisms, and each mechanism may only be required by a single running command at a time. This prevents conflicting or dangerous control requests being issued: for example, if an "arm up" command is started while an "arm down" command is running, the "arm down" command will stop - if these commands weren't using the requirements system, then the arm would be commanded to go both up *and* down simultaneously.

It is therefore *strongly* recommended to use the following system when writing code that controls physical hardware:

1. Write a class that implements the ``Mechanism`` interface
2. Make all fields in the class ``private`` to prevent external access
3. Make all methods that use those fields to control hardware also ``private``
4. Write ``public`` methods that return ``Commands`` for all control of the mechanism

.. tab-set-code::

  ```java
  import org.wpilib.command3.Command;
  import org.wpilib.command3.Mechanism;

  public class ExampleArm implements Mechanism {
    // This motor controller is declared private to guarantee that it can't be used
    // dangerously, outside of the command requirements system
    private final SmartMotorController pivotMotor = new SmartMotorController(...);

    // Triggers can be and are encouraged to be public. They can't control the mechanism,
    // and make it easier to coordinate complex actions
    public final Trigger isUp = new Trigger(() -> pivotMotor.getAngle() >= 90);
    public final Trigger isDown = new Trigger(() -> pivotMotor.getAngle() <= 0);

    public ExampleArm() {
      setDefaultCommand(stop());
    }

    // This method controls the motor directly.
    // It's private for the same reason the field is - to prevent dangerous usage
    private void stopMotor() {
      pivotMotor.setVoltage(0);
    }

    // This factory method returns a Command.
    // It's public because all hardware control should go through commands
    // instead of unsafe method calls.
    public Command stop() {
      // `run` and `runRepeatedly` will automatically require the mechanism
      // so we don't need to manually spell it out every time
      return runRepeatedly(this::stopMotor).named("Stop Arm");
    }

    public Command up() {
      return run(coroutine -> {
        pivotMotor.setGoalPosition(90);
        coroutine.waitUntil(isUp);
      }).named("Arm Up");
    }

    public Command down() {
      return run(coroutine -> {
        pivotMotor.setGoalPosition(0);
        coroutine.waitUntil(isDown);
      }).named("Arm Down");
    }
  }
  ```


## Looping Commands

## One-Shot Commands

A command that never yields is called a *one-shot* command. It will be mounted and run to completion before the scheduler will pick up the next command to execute. Long-running one-shot commands, such as ones that wait on data to be loaded from disk or run expensive vision or path planning algorithms, will stall the scheduler and the entire robot program.

One-shot commands generally do one very simple thing and immediately exit without taking much time. Good examples of one-shot commands are zeroing a sensor or assigning a new value to a variable.

.. tab-set-code::

  ```java
  Command.noRequirements(_ -> m_gyro.reset());
  Command.noRequirements(_ -> m_field = 0);
  ```
