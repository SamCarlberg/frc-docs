# Scopes

Commands are only allowed to run in the scope that schedules them; when a scope exits, all commands still running in that scope are canceled. This is for safety: if, for example, a command that applies a lot of power to a particular mechanism is scheduled in an opmode, that command should not continue to run when the opmode exits - otherwise, if the next opmode doesn't use that mechanism, then the dangerous command will resume and potentially injure somebody or damage the mechanism.

Scopes aren't limited to only command lifetimes. A trigger binding will only be active in the scope where the binding is created, and will be automatically disabled and deleted when that scope exits; default commands set in a narrower scope take precedence over default commands from wider scopes; and sideloaded scheduler functions will also be removed from the scheduler when the scope that set up those functions exits.

The framework has three scopes, which may all coexist, in order of narrowest to widest:

## The Command Scope

If a command is scheduled inside another command (often referred to as a "child" and a "parent" command), the child command will be canceled when the parent command stops. This applies recursively: when a parent command stops, its immediate children are canceled, which then go on to cancel *their* immediate children, and so on.

This scope is particularly useful for automating event-based behavior within a specific command.

.. tab-set-code::

  ```java
  /**
   * Creates a command that drives the robot in a path that sweeps through the
   * neutral zone to gather fuel, returns to the alliance zone, and then shoots
   * at the hub until the command is canceled or interrupted.
   *
   * @param robot The robot instance to control
   * @return A sweep-and-score command
   */
  private Command sweepAndScore(Robot robot) {
    return Command.noRequirements(
            coroutine -> {
              // Set up a trigger to automatically extend the intake once the robot enters the
              // neutral zone, then leave it extended for the rest of the command.
              robot.inNeutralZone.onTrue(robot.intake.intake());

              // Follow a path into the neutral zone to pick up fuel,
              // then return to the alliance zone to score.
              coroutine.await(robot.swerveDrive.followPath("nz-sweep-left-trench"));

              // Continually shoot at the hub when the robot returns from the neutral zone sweep.
              coroutine.await(robot.shootAt(FieldConstants.targetHub()));
            })
        .named("Sweep and Score");
  }
  ```

## The OpMode Scope

Any commands scheduled while an opmode is selected on the driver station will be canceled when a different opmode is selected. Use this scope for commands and bindings that are specific to a particular opmode.

.. tab-set-code::

  ```java
  @Autonomous
  public class SweepAuto implements OpMode {
    public SweepAuto(Robot robot) {
      // Start the intake stowed
      robot.intake.setDefaultCommand(robot.intake.stow());

      // Once the robot is enabled, start following a sweep path through the
      // left trench, into the neutral zone, then back over the bump.
      // When we return to the alliance zone, aim at the hub and start shooting.
      RobotModeTriggers.enabled().onTrue(sweepAndScore(robot));
    }
  }
  ```

## The Global Scope

The global scope. Any commands that are scheduled in this scope or bound to triggers in this scope will continue to run when opmodes change. Use this scope for commands and bindings that need to be active regardless of opmode or robot state - typically, this would be for default commands that de-energize mechanisms or commands that control non-physical mechanisms like LED strips or gamepad rumble settings.
