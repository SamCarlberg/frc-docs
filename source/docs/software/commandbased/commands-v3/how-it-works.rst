# How the Scheduler Works

.. mermaid::

  sequenceDiagram
      participant Robot as User Robot Class
      participant Scheduler as Scheduler
      participant Sideloads as Sideloads & Periodics
      participant Triggers as Trigger Bindings (EventLoop)
      participant Commands as Running Commands

      Note over Robot: robotPeriodic()
      Robot->>Scheduler: run()

      rect rgb(240, 240, 240)
          Note right of Scheduler: Phase 1: Cleanup
          Scheduler->>Scheduler: cancelStaleBindings()
          Scheduler->>Scheduler: unbindStaleTriggers()
      end

      rect rgb(230, 240, 255)
          Note right of Scheduler: Phase 2: Sideloads
          Scheduler->>Sideloads: runPeriodicSideloads()
      end

      rect rgb(240, 255, 240)
          Note right of Scheduler: Phase 3: Scheduling
          Scheduler->>Triggers: m_eventLoop.poll()
          Note over Triggers: Triggers may schedule or cancel commands
          Scheduler->>Scheduler: scheduleDefaultCommands()
          Scheduler->>Scheduler: promoteScheduledCommands()
          Note over Scheduler: Evicts conflicting commands & moves to running set
      end

      rect rgb(255, 245, 230)
          Note right of Scheduler: Phase 4: Execution
          Scheduler->>Commands: runCommands()
          loop for each CommandState (reversed)
              Scheduler->>Commands: runCommand(state)
              Commands->>Commands: coroutine.mount()
              Commands->>Commands: coroutine.runToYieldPoint()
              Note over Commands: Command logic executes until yield or completion
              alt is coroutine done?
                  Commands->>Scheduler: emitCompletedEvent()
                  Commands->>Scheduler: remove from running set
              else yielded
                  Commands->>Scheduler: emitYieldedEvent()
              end
          end
      end


