# ARM Task Scheduler using SysTick Timer

This project implements a **task scheduler** for ARM microcontrollers using the **SysTick timer** and **SysTick_Handler**. It handles task switching, processor stack pointer (PSP) management, and context saving/restoration using ARM-inline-GCC assembly. The scheduler is designed for embedded systems, particularly for STM32-based microcontrollers.

## Features
- **Task Switching**: Utilizes the SysTick timer to trigger periodic interrupts, and the PendSV handler to switch between tasks.
- **Processor Stack Pointer (PSP) Management**: The scheduler switches between the main stack pointer (MSP) and processor stack pointer (PSP) for each task.
- **Task Control Block (TCB)**: Stores task-specific information including PSP value, task state, and task handler function.
- **Task Scheduling**: Round-robin scheduling with tasks switched based on a fixed time quantum.
- **Task Management Functions**: Includes task delay, task block, and task unblock functionality.
- **Fault Handling**: Enables memory, bus, and usage faults to ensure stable task switching.
- **SysTick Timer Management**: Configures and manages the SysTick timer for regular task switching.

## Files
- **main.c**: Main program body, which contains task initialization, SysTick timer configuration, and task management functions.
- **main.h**: Header file defining macros, stack memory configuration, and clock settings.
- **led.h**: Header file for LED control functions.
- **led.c**: Contains the LED control functions.

## Task Control Block (TCB)

The **TCB** is a structure used to manage tasks in the scheduler. Each task has the following attributes:
- **pspVal**: The processor stack pointer (PSP) value for the task.
- **blockCount**: The time at which the task should be unblocked after a delay.
- **currentState**: The state of the task (`TASK_READY_STATE` or `TASK_BLOCKED_STATE`).
- **task_handler**: A pointer to the function that represents the task.

```c
typedef struct {
    uint32_t pspVal;
    uint32_t blockCount;
    uint8_t  currentState;
    void     (*task_handler)(void);
} TCB_t;
```

## Task Initialization

Tasks are initialized with unique stack locations and are assigned handlers that control their execution. The following task handler functions are defined:

- `task1_handler()`: Controls the green LED.
- `task2_handler()`: Controls the orange LED.
- `task3_handler()`: Controls the blue LED.
- `task4_handler()`: Controls the red LED.

Each task runs in a continuous loop, alternating between turning an LED on and off with a delay.

## System Configuration

### Stack Memory Configuration

The system uses separate stack regions for each task and a scheduler stack. The stack memory layout is configured as follows:

```c
#define T1_STACK_START       SRAM_END
#define T2_STACK_START       (SRAM_END - SIZE_TASK_STACK)
#define T3_STACK_START       (SRAM_END - 2 * SIZE_TASK_STACK)
#define T4_STACK_START       (SRAM_END - 3 * SIZE_TASK_STACK)
#define IDLE_STACK_START     (SRAM_END - 4 * SIZE_TASK_STACK)
#define SCHED_STACK_START    (SRAM_END - 5 * SIZE_TASK_STACK)
```

### Clock Configuration

The system uses an HSI (High-Speed Internal) clock at 16 MHz for the SysTick timer. The `SYSTICK_TIM_CLK` macro is defined as:

```c
#define SYSTICK_TIM_CLK      16000000U
```

### Task Scheduling

Tasks are scheduled to run based on the SysTick interrupt. The SysTick timer generates an interrupt at a frequency defined by `TICK_HZ`, and tasks are switched in the PendSV handler.

## Key Functions

- **`init_systick_timer(uint32_t tick_hz)`**: Configures the SysTick timer to generate interrupts at the specified frequency.
- **`init_scheduler_stack(uint32_t sched_top_of_stack)`**: Initializes the scheduler stack with the top of the stack address.
- **`init_tasks_stack()`**: Initializes the stacks for each task and sets up the initial context (PSP, XPSR, PC, LR, etc.).
- **`task_delay(uint32_t tickCount)`**: Delays the current task for a given number of ticks, causing it to be blocked temporarily.
- **`save_psp_value(uint32_t current_psp_val)`**: Saves the current PSP value for the current task.
- **`switch_sp_to_psp()`**: Switches the stack pointer to the PSP.
- **`schedule()`**: Triggers a PendSV exception to switch to the next task.
- **`PendSV_Handler()`**: The context switching handler that saves the state of the current task and loads the state of the next task.
- **`unblock_tasks()`**: Checks and unblocks tasks that have completed their delay.

## Fault Handling

The following processor faults are enabled:

- **Memory Manage Fault**
- **Bus Fault**
- **Usage Fault**

These faults ensure that the system remains stable during task switching and execution.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
