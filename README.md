# STM32 FreeRTOS Counting Semaphore

## Overview
This project demonstrates the usage of counting semaphores for resource management in an STM32 microcontroller using FreeRTOS. Counting semaphores are used to manage a pool of resources where multiple tasks can access them simultaneously up to a maximum count.

> This project uses **native FreeRTOS APIs** directly, not the CMSIS-RTOS wrapper.

## Project Description
The application creates four tasks with different priorities:
- HPT Task (Highest Priority: 3)
- MPT Task (Medium-High Priority: 2)
- LPT Task (Medium-Low Priority: 1)
- VLPT Task (Lowest Priority: 0)

A counting semaphore with maximum count of 3 is used to manage access to a shared resource pool containing three data values {111, 222, 333}. This demonstrates how counting semaphores can protect a pool of identical resources.

## Key Concept: Producer-Consumer Pattern
- ISR (Producer) : The UART interrupt service routine produces semaphore tokens when user sends 'r'
- Tasks (Consumers) : All tasks consume tokens to access shared resources
- Tokens are never returned by tasks - they are "consumed" and must be replenished by the ISR

## Hardware Requirements
- STM32 development board (STM32F103C8T6 "Blue Pill")
- USB-to-UART converter (for sending commands and viewing debug messages)
- UART connection (PA9/PA10 for USART1 on STM32F103)

## Pin Configuration
- USART1 TX: PA9 (debug output)
- USART1 RX: PA10 (receive 'r' command to give semaphore tokens)

## Task Priorities
| Task | Priority | Behavior |
|------|----------|----------|
| HPT_Task | 3 (Highest) | 3000ms delay after accessing resource |
| MPT_Task | 2 | 2000ms delay after accessing resource |
| LPT_Task | 1 | 1000ms delay after accessing resource |
| VLPT_Task | 0 (Lowest) | 500ms delay after accessing resource |

## UART Output

https://github.com/user-attachments/assets/a0287168-f4be-496a-bd99-11d74c2c87cb

## Counting Semaphore Behavior
1. Resource Pool Management
    - Semaphore created with xSemaphoreCreateCounting(3, 0) - max 3 tokens, initially 0
    - HPT Task gives 3 initial tokens at startup
    - Maximum 3 tasks can access resources simultaneously

2. Multiple Task Access
    - Unlike binary semaphores, counting semaphores allow multiple tasks to hold the semaphore simultaneously:
    - First 3 tasks can acquire tokens and run in parallel
    - 4th task (VLPT) blocks until tokens become available

3. Priority-Based Scheduling
    - Higher priority tasks run first when multiple tasks are ready
    - HPT (prio 3) → MPT (prio 2) → LPT (prio 1) → VLPT (prio 0)

4. ISR as Exclusive Producer
    - Tokens are ONLY given in the UART ISR when 'r' is received
    - Tasks ONLY take tokens, never give them back
    - This demonstrates a pure producer-consumer pattern

5. Task Blocking
    - Tasks block when no tokens are available
    - All 4 tasks eventually block after consuming the initial 3 tokens
    - System waits for user to send 'r' via UART to replenish tokens
    - 
## Semaphore Release Strategy

In this project, semaphore tokens are only released from the UART ISR, not from tasks. This demonstrates an important embedded systems concept:
- ISR-based release is more challenging - Requires special FromISR functions and context switching considerations
- Tasks can easily release semaphores using xSemaphoreGive() - This is straightforward
- ISR release demonstrates:
    - Proper use of xSemaphoreGiveFromISR()
    - Context switching with portYIELD_FROM_ISR()
    - Safe communication from interrupt context to tasks

> **Note:** Semaphores can be released from anywhere - tasks, interrupts, timers, etc. However, releasing from an ISR is more complex and requires special handling. This project intentionally uses the more challenging ISR-based release to demonstrate proper interrupt-safe semaphore operations.

### Comparison: Task vs ISR Release
| Aspect | Task Release | ISR Release (This Project) |
|--------|--------------|----------------------------|
| Function | `xSemaphoreGive()` | `xSemaphoreGiveFromISR()` |
| Context switching | Automatic | Manual with `portYIELD_FROM_ISR()` |
| Complexity | Simple | More complex |
| Use case | Normal task synchronization | Interrupt-driven events |

## Binary vs Counting Semaphore Comparison

| Feature | Binary Semaphore | Counting Semaphore |
|---------|------------------|-------------------|
| Maximum value | 1 | Configurable (3 in this project) |
| Simultaneous holders | 1 task | Multiple tasks (up to max) |
| Use case | Mutual exclusion | Resource counting/pooling |
| Task blocking | Only 1 task can wait | Multiple tasks can wait |
| Example | Protect single resource | Protect 3 identical resources |

## Contact
**Rubin Khadka Chhetri**  
📧 rubinkhadka84@gmail.com <br>
🐙 GitHub: https://github.com/rubin-khadka
