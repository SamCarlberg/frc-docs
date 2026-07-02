# Command Triggers

.. tab-set-code::

  ```java
  new Trigger(condition::get).onTrue(run(_ -> piston.set(DoubleSolenoid.Value.kForward)));
  ```

Triggers are objects that check the true/false value of a binary condition. Commands can be bound to triggers to start and stop when the condition turns on or when it turns off.

.. tab-set-code::

  ```java
  boolean condition;
  Trigger trigger = new Trigger(() -> condition);

  trigger.onTrue(run(_ -> IO.println("Condition turned on")));
  trigger.onFalse(run(_ -> IO.println("Condition turned off")));
  trigger.whileTrue(runRepeatedly((() -> IO.println("Condition is still true")));
  trigger.whileFalse(runRepeatedly(() -> IO.println("Condition is still false")));
  ```

## Scopes

Trigger bindings exist in *scopes*. When a scope exits, any trigger binding that was created in that scope will be removed and the commands attached to that binding will be canceled. There are three scopes:

1. The *global* scope. Trigger bindings created in this scope will always be active. Bindings created during program startup in the ``Robot`` class constructor (or code called by it) will exist in the global scope.
2. *Opmode* scope. Trigger bindings created when an opmode is active will only exist within that opmode. This prevents potential dangerous or unexpected behavior if a command were to continue to run when a different opmode is loaded. Any triggers in the global scope will still be active in an opmode scope.
3. *Command* scope. Trigger bindings created in a running command are limited to the lifetime of that command. Any bindings created in a running command are deleted when the command exits. Any triggers in the global or opmode scopes will still be active in a command scope.

.. tab-set-code::

  ```java
  // TODO: Example code demonstrating trigger bindings in global, opmode, and command scopes
  ```
