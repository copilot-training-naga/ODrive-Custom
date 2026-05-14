# Lesson 5: C++ Best Practices with GitHub Copilot

**Session Duration:** 50 minutes  
**Audience:** Embedded/C++ Developers (Intermediate to Advanced)  
**Environment:** Windows, VS Code  
**Extensions:** GitHub Copilot  
**Source Control:** GitHub/Bitbucket

---

## Overview

This lesson teaches you how to leverage GitHub Copilot for modern C++ development in embedded systems. You'll learn to apply RAII patterns, enforce embedded constraints (no heap, no exceptions), implement RTOS patterns, and generate production-quality code using specialized prompts.

**What You'll Learn:**
- Modern C++ patterns: RAII, templates, const correctness, move semantics
- Embedded constraints: static allocation, error codes, volatile, ISR safety
- RTOS patterns: state machines, HAL abstractions, task synchronization
- Generating embedded-safe code with Copilot using specialized prompts

**Key Concepts:**

| Concept | Description |
|---------|-------------|
| **RAII** | Resource Acquisition Is Initialization - automatic cleanup |
| **Static Allocation** | No heap (malloc/new) - fixed memory footprint |
| **Error Codes** | No exceptions - return enum class error codes |
| **Volatile** | Required for hardware registers and ISR-shared variables |
| **State Machines** | Task chain pattern for complex control flow |
| **HAL Abstractions** | Hide vendor HAL details behind clean interfaces |

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Why C++ Best Practices Matter](#why-c-best-practices-matter)
- [Learning Path](#learning-path)
- [Modern C++ Patterns](#1-modern-c-patterns-12-min)
- [Embedded C++ Specifics](#2-embedded-c-specifics-12-min)
- [RTOS & Hardware Patterns](#3-rtos--hardware-patterns-10-min)
- [Hands-On: Generate Components](#4-hands-on-generate-components-16-min)
- [Unit Testing with /tests](#5-unit-testing-with-tests)
- [C++ Architecture Documentation](cpp-architecture-documentation.md)
- [Creating Embedded C++ Personas](#creating-embedded-c-personas)
- [Using Personas in Copilot](#using-personas-in-copilot)
- [Practice Exercises](#practice-exercises)
- [Quick Reference](#quick-reference-mode-selection-guide)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)
- [Frequently Asked Questions](#frequently-asked-questions)
- [Summary: Key Takeaways](#summary-key-takeaways)

---

## Prerequisites

Before starting this session, ensure you have:

- ✅ **Completed Planning & Steering Documents** - Understanding of custom instructions and agents
- ✅ **Completed Agentic Development** - Understanding of context engineering
- ✅ **Visual Studio Code** with GitHub Copilot extensions installed and enabled
- ✅ **C++ development experience** - Familiar with classes, templates, STL
- ✅ **ODrive workspace** - Access to the ODrive firmware codebase

### Verify Your Setup

1. **Check ODrive Firmware access:**
   - Ensure `src-ODrive/Firmware/` folder is accessible
   - Verify `MotorControl/`, `Drivers/`, and `fibre-cpp/` subdirectories exist

2. **Verify custom agents are available:**
   - Open Chat view (Ctrl+Alt+I)
   - Check agents dropdown for `@ODrive-Engineer`, `@ODrive-QA`

3. **Test code navigation:**
   - Open `src-ODrive/Firmware/MotorControl/axis.cpp`
   - Verify Go to Definition (F12) works

---

## Why C++ Best Practices Matter

Embedded C++ has unique constraints that distinguish it from desktop/server development. Copilot can help enforce these constraints when given proper context.

### Embedded vs Desktop C++

| Aspect | Desktop C++ | Embedded C++ |
|--------|-------------|--------------|
| Memory | Dynamic allocation OK | Static allocation only |
| Exceptions | Standard practice | Forbidden (code size, determinism) |
| RTTI | Available | Usually disabled |
| Stack | Generous | Very limited (KB) |
| Timing | Best effort | Hard real-time deadlines |

### Benefits of Copilot for Embedded C++

1. **Consistent Patterns**
   - Generates RAII wrappers automatically
   - Follows project coding standards
   - Applies embedded constraints consistently

2. **Reduced Boilerplate**
   - HAL abstractions from templates
   - State machine scaffolding
   - Error handling patterns

3. **Learning Accelerator**
   - Explains unfamiliar patterns in codebase
   - Suggests best practices for new features
   - Validates designs against constraints

---

## Learning Path

This lesson covers four key areas. Work through them sequentially for the best learning experience.

| Topic | What You'll Learn | Estimated Time |
|-------|-------------------|----------------|
| Modern C++ Patterns | RAII, templates, const correctness | 12 min |
| Embedded C++ Specifics | Static allocation, error codes, volatile | 12 min |
| RTOS & Hardware Patterns | State machines, HAL, task sync | 10 min |
| Hands-On: Generate Components | LED driver, SPI sensor, ring buffer | 16 min |

---

## 1. Modern C++ Patterns (12 min)

### RAII & Resource Management
**🎯 Copilot Modes: Agent + Chat**

**Files to demonstrate:**
- [src-ODrive/Firmware/fibre-cpp/include/fibre/cpp_utils.hpp](../../src-ODrive/Firmware/fibre-cpp/include/fibre/cpp_utils.hpp#L380-L400) - Custom `variant` with destructor
- [src-ODrive/Firmware/doctest/doctest.h](../../src-ODrive/Firmware/doctest/doctest.h#L1483-L1510) - `ContextScope` RAII pattern

**💬 Chat Mode Prompt (Analysis):**
```
Review this GPIO class and suggest RAII improvements:
#file:src-ODrive/Firmware/Drivers/STM32/stm32_gpio.hpp

Analyze:
- Current resource lifecycle management
- Missing RAII patterns
- Copy/move semantics issues
- Potential double-free or leak scenarios

Provide specific recommendations with code examples.
```

**🤖 Agent Mode Prompt (Implementation):**
```
Implement RAII improvements for GPIO class

Context: #file:src-ODrive/Firmware/Drivers/STM32/stm32_gpio.hpp
         #file:src-ODrive/Firmware/Drivers/STM32/stm32_gpio.cpp

Requirements:
- Add destructor that calls unsubscribe() if owns_subscription_
- Delete copy constructor and copy assignment
- Implement move constructor and move assignment
- Add owns_subscription_ member to track cleanup responsibility
- Update subscribe() to set ownership flag
- Update unsubscribe() to clear ownership flag
- Add const correctness to read() and get_pin_number()

Acceptance Criteria:
- Compiles without warnings
- No manual unsubscribe() needed in typical usage
- Safe to store in containers with move semantics
- Thread-safe as documented
- Maintains existing public API compatibility
```

### Templates & Type Traits
**🎯 Copilot Modes: Chat + Agent**

**Files to demonstrate:**
- [src-ODrive/Firmware/fibre-cpp/include/fibre/cpp_utils.hpp](../../src-ODrive/Firmware/fibre-cpp/include/fibre/cpp_utils.hpp#L113-L140) - `integer_sequence`, type traits
- [src-ODrive/Firmware/fibre-cpp/include/fibre/cpp_utils.hpp](../../src-ODrive/Firmware/fibre-cpp/include/fibre/cpp_utils.hpp#L980-L1010) - Template iterators

**💬 Chat Mode Prompt (Explain Pattern):**
```
Explain the template metaprogramming patterns in:
#file:src-ODrive/Firmware/fibre-cpp/include/fibre/cpp_utils.hpp

Focus on:
- integer_sequence implementation and use cases
- Type trait techniques for compile-time checks
- SFINAE patterns used
- Benefits for embedded systems (zero runtime cost)
```

**🤖 Agent Mode Prompt (Convert to Generic):**
```
Make buffer class generic using templates

Context: #file:src-ODrive/Firmware/fibre-cpp/include/fibre/bufptr.hpp

Task:
- Convert to template class with type parameter T
- Add static_assert to ensure T is POD (std::is_trivial)
- Preserve const correctness for all accessors
- Maintain zero-overhead abstraction
- Update all methods to work with generic T
- Add Doxygen documentation for template parameter

Acceptance Criteria:
- Works with uint8_t, uint16_t, uint32_t, and custom POD structs
- Compile error for non-POD types with clear message
- No runtime overhead vs original implementation
- All existing usage patterns still work
```

### Const Correctness
**🎯 Copilot Modes: Chat + Agent**

**Files to demonstrate:**
- [src-ODrive/Firmware/fibre-cpp/include/fibre/bufptr.hpp](../../src-ODrive/Firmware/fibre-cpp/include/fibre/bufptr.hpp#L70-L80) - Const accessors

**💬 Chat Mode Prompt (Audit):**
```
Audit const correctness in this class:
#file:src-ODrive/Firmware/Drivers/STM32/stm32_gpio.hpp

Check:
1. Which methods should be marked const but aren't?
2. Which parameters should be const references?
3. Are there any const methods that modify mutable state?
4. Suggest const overloads where appropriate (e.g., begin()/begin() const)

Provide a prioritized list with rationale for each change.
```

**🤖 Agent Mode Prompt (Apply Fixes):**
```
Apply const correctness improvements to GPIO class

Context: #file:src-ODrive/Firmware/Drivers/STM32/stm32_gpio.hpp
         #file:src-ODrive/Firmware/Drivers/STM32/stm32_gpio.cpp

Changes to apply:
- Mark read() as const (doesn't modify logical state)
- Mark get_pin_number() as const
- Mark operator bool() as const
- Review all methods for const qualification
- Ensure member variables aren't mutable unless necessary

Acceptance Criteria:
- Can call read-only methods on const Stm32Gpio objects
- Compiles without warnings
- No behavioral changes
- Thread-safety properties preserved
```

---

## 2. Embedded C++ Specifics (12 min)

### Static Allocation (No Heap)
**🎯 Copilot Modes: Agent + Chat**

**Files to demonstrate:**
- [src-ODrive/Firmware/fibre-cpp/fibre.cpp](../../src-ODrive/Firmware/fibre-cpp/fibre.cpp#L56-L75) - `TheInstance<T>` singleton

**Pattern: TheInstance<T> Singleton**
```cpp
template<typename T>
struct TheInstance {
    static T instance;
    static bool in_use;
};
```

**💬 Chat Mode Prompt (Identify Issues):**
```
Analyze dynamic memory usage in this driver:
#file:src-ODrive/Firmware/Drivers/STM32/stm32_spi_arbiter.cpp

Identify:
1. All uses of new, delete, malloc, free
2. std::vector or other heap-based containers
3. Unbounded queues or buffers
4. Memory allocation in hot paths

For each issue, suggest static allocation alternatives:
- Singleton pattern vs new
- std::array with compile-time size
- Fixed-size ring buffers
- Placement new with static storage
```

**🤖 Agent Mode Prompt (Refactor to Static):**
```
Refactor SPI arbiter to use static allocation only

Context: #file:src-ODrive/Firmware/Drivers/STM32/stm32_spi_arbiter.cpp
         #file:src-ODrive/Firmware/Drivers/STM32/stm32_spi_arbiter.hpp

Refactoring steps:
1. Replace std::vector with std::array<SpiTask, 8>
2. Use TheInstance<> pattern for singleton instead of new
3. Convert dynamic queue to fixed-size circular buffer
4. Add compile-time size constants (MAX_TASKS, QUEUE_SIZE)
5. Add overflow handling (drop oldest or return error)
6. Remove all new/delete/malloc/free

Acceptance Criteria:
- Zero calls to heap allocator (verify with nm/objdump)
- Fixed RAM footprint (no growth at runtime)
- MISRA C++ 2008 compliant
- Performance equal or better than original
- All existing tests pass
```

### No Exceptions - Error Code Pattern
**🎯 Copilot Modes: Chat + Agent**

**Files to demonstrate:**
- [src-ODrive/Firmware/MotorControl/encoder.cpp](../../src-ODrive/Firmware/MotorControl/encoder.cpp#L329-L450) - `set_error()`, returns `bool`
- [src-ODrive/Firmware/MotorControl/axis.cpp](../../src-ODrive/Firmware/MotorControl/axis.cpp#L443-L450) - State machine error handling

**💬 Chat Mode Prompt (Design Error System):**
```
Analyze error handling patterns across ODrive firmware:
#file:src-ODrive/Firmware/MotorControl/encoder.cpp
#file:src-ODrive/Firmware/MotorControl/axis.cpp
#file:src-ODrive/Firmware/MotorControl/motor.cpp

Design unified error handling system:
1. Catalog all current error conditions by category
2. Propose enum class ErrorCode with severity levels:
   - Info (0x0000-0x0FFF): Informational
   - Warning (0x1000-0x1FFF): Degraded operation
   - Error (0x2000-0x2FFF): Operation failed
   - Critical (0x3000-0x3FFF): System halt required
3. Define error propagation strategy (return codes vs callbacks)
4. Suggest error recovery patterns per category

Output: Complete error code enum with documentation
```

**🤖 Agent Mode Prompt (Convert Function):**
```
Convert calibration function to use error codes

Context: #file:src-ODrive/Firmware/MotorControl/encoder.cpp:run_offset_calibration

Conversion steps:
1. Change return type from void to enum class ErrorCode
2. Replace set_error() calls with error code returns
3. Add error checking after each operation:
   - Timeout checks → ErrorCode::TIMEOUT
   - Hardware faults → ErrorCode::HARDWARE_ERROR
   - Invalid state → ErrorCode::INVALID_STATE
4. Update all callers to check return value
5. Add Doxygen @return documentation for each error

Acceptance Criteria:
- No exceptions or throw statements
- Every error path returns specific error code
- All callers updated to check result
- Maintains original behavior for success case
- Errors are actionable (clear what went wrong)
```

### Volatile for Hardware Registers
**🎯 Copilot Mode: Inline + Chat**

**Files to demonstrate:**
- [src-ODrive/Firmware/Board/v3/board.cpp](../../src-ODrive/Firmware/Board/v3/board.cpp#L463-L480) - `volatile` timestamps
- [src-ODrive/Firmware/Board/v3/Src/stm32f4xx_it.c](../../src-ODrive/Firmware/Board/v3/Src/stm32f4xx_it.c#L114-L125) - Direct register access

**Inline Completion Example:**
```cpp
// Define ADC register structure
struct ADC_Regs {
    volatile uint32_t CR1;    // Copilot completes register map
```

**Chat Validation:**
```
Review volatile keyword usage in:
#file:src-ODrive/Firmware/Board/v3/board.cpp

Check:
- All shared ISR variables are volatile
- Hardware register pointers use volatile
- No missing volatile qualifiers
```

### ISR Handlers
**🎯 Copilot Mode: Inline + Edit**

**Files to demonstrate:**
- [src-ODrive/Firmware/Drivers/STM32/stm32_gpio.cpp](../../src-ODrive/Firmware/Drivers/STM32/stm32_gpio.cpp#L188-L230) - EXTI interrupt handlers
- [src-ODrive/Firmware/Drivers/STM32/stm32_system.h](../../src-ODrive/Firmware/Drivers/STM32/stm32_system.h#L23-L40) - Critical sections

**Inline Prompt:**
```cpp
// Generate TIM13 overflow ISR with critical section
extern "C" void TIM13_IRQHandler(void) {
    // Copilot completes handler with proper pattern
```

**Edit Mode Example:**
```
Add critical section protection to this ISR:
#file:src-ODrive/Firmware/Board/v3/board.cpp:TIM8_UP_TIM13_IRQHandler

Requirements:
- Use cpu_enter_critical/cpu_exit_critical
- Minimize critical section duration
- Clear interrupt flags correctly
- No blocking calls (osDelay, malloc, etc.)
```

---

## 3. RTOS & Hardware Patterns (10 min)

### State Machines
**🎯 Copilot Modes: Chat + Agent**

**The Crown Jewel:** [src-ODrive/Firmware/MotorControl/axis.cpp](../../src-ODrive/Firmware/MotorControl/axis.cpp#L454-L600)

**Key Pattern:**
```cpp
void Axis::run_state_machine_loop() {
    for (;;) {
        // Task chain pattern
        switch (current_state_) {
            case AXIS_STATE_MOTOR_CALIBRATION:
                status = motor_.run_calibration();
                break;
            // ... more states
        }
    }
}
```

**💬 Chat Mode Prompt (Understand Pattern):**
```
Explain the state machine architecture in Axis class:
#file:src-ODrive/Firmware/MotorControl/axis.cpp:run_state_machine_loop

Analyze:
1. How does the task chain pattern work?
2. How are states sequenced automatically?
3. How does error handling interrupt state flow?
4. How are resources acquired/released per state?
5. How is this different from a traditional FSM?

Then create a state transition diagram in Mermaid format showing:
- All states and transitions
- Error recovery paths
- Timeout transitions
- Entry/exit actions
```

**🤖 Agent Mode Prompt (Generate State Machine):**
```
Create LED controller with state machine pattern

Reference: #file:src-ODrive/Firmware/MotorControl/axis.cpp
           #file:src-ODrive/Firmware/Drivers/STM32/stm32_gpio.hpp

State Machine Design:

States:
- LED_STATE_OFF: GPIO low, no power
- LED_STATE_ON: GPIO high, steady
- LED_STATE_BLINKING: Toggle at configured rate
- LED_STATE_FADING: PWM fade in/out
- LED_STATE_ERROR: Fast blink pattern

Features:
1. Task chain for automatic sequences (e.g., fade→on→off)
2. State entry/exit actions (enable PWM, configure GPIO)
3. Timeout handling (auto-off after duration)
4. Error recovery (fault→error state→retry→off)
5. RAII guard for GPIO lifecycle

Files to create:
- led_controller.hpp (class definition, state enum)
- led_controller.cpp (state machine loop, state handlers)

Acceptance Criteria:
- Follows Axis pattern (switch/case with task chain)
- FreeRTOS task with osDelay timing
- Static allocation only (std::array for task chain)
- Error codes, no exceptions
- Configurable blink/fade rates
- Doxygen documentation for all states
```

### HAL Abstractions
**🎯 Copilot Modes: Chat + Agent**

**Files to demonstrate:**
- [src-ODrive/Firmware/Drivers/STM32/stm32_gpio.hpp](../../src-ODrive/Firmware/Drivers/STM32/stm32_gpio.hpp#L7-L60) - GPIO HAL
- [src-ODrive/Firmware/Drivers/STM32/stm32_spi_arbiter.hpp](../../src-ODrive/Firmware/Drivers/STM32/stm32_spi_arbiter.hpp#L9-L90) - SPI HAL

**💬 Chat Mode Prompt (Learn Pattern):**
```
Analyze HAL abstraction patterns in ODrive:
#file:src-ODrive/Firmware/Drivers/STM32/stm32_gpio.hpp
#file:src-ODrive/Firmware/Drivers/STM32/stm32_spi_arbiter.hpp

Extract common design principles:
1. How do they hide STM32 HAL implementation details?
2. How is resource ownership managed (RAII)?
3. How are async operations handled (callbacks/futures)?
4. How is thread-safety achieved?
5. How are errors propagated?

Create a HAL Design Checklist for new abstractions based on these patterns.
```

**🤖 Agent Mode Prompt (Create New HAL):**
```
Create UART HAL abstraction following ODrive patterns

References: #file:src-ODrive/Firmware/Drivers/STM32/stm32_gpio.hpp
            #file:src-ODrive/Firmware/Drivers/STM32/stm32_spi_arbiter.hpp

UART HAL Requirements:

Class Structure:
- Stm32UartHal class wrapping UART_HandleTypeDef
- RAII: constructor acquires UART, destructor releases
- Delete copy, implement move semantics
- Hide UART_HandleTypeDef* as private member

Public API:
- ErrorCode init(uint32_t baudrate, uint32_t config)
- ErrorCode write(const uint8_t* data, size_t len, uint32_t timeout_ms)
- ErrorCode write_dma(const uint8_t* data, size_t len, Callback cb)
- std::pair<size_t, ErrorCode> read(uint8_t* buf, size_t max_len)
- void set_rx_callback(Callback cb, void* ctx)

Internal Features:
- Static RX/TX ring buffers (std::array<uint8_t, 256>)
- DMA completion callbacks
- Critical sections for buffer access
- Error recovery (overrun, framing errors)

Files to create:
- Drivers/STM32/stm32_uart_hal.hpp
- Drivers/STM32/stm32_uart_hal.cpp

Acceptance Criteria:
- No direct access to UART_HandleTypeDef by users
- Thread-safe buffer operations
- Zero dynamic allocation
- Comprehensive Doxygen docs
- Example usage in comments
```

### Task Synchronization (FreeRTOS)
**🎯 Copilot Mode: Inline + Chat**

**Files to demonstrate:**
- [src-ODrive/Firmware/MotorControl/main.cpp](../../src-ODrive/Firmware/MotorControl/main.cpp#L826-L850) - Thread creation
- [src-ODrive/Firmware/MotorControl/low_level.cpp](../../src-ODrive/Firmware/MotorControl/low_level.cpp#L388-L410) - Polling thread

**Inline Example:**
```cpp
// Create producer-consumer with semaphore
osSemaphoreDef(adc_ready);
// Copilot completes initialization and usage
```

**Chat Mode Pattern:**
```
Generate ADC sampling architecture using FreeRTOS:
#file:src-ODrive/Firmware/MotorControl/low_level.cpp:analog_polling_thread

Design:
- ADC ISR writes to ring buffer (producer)
- Processing task reads from buffer (consumer)
- Binary semaphore for synchronization
- Queue for timestamped samples
- Proper stack size calculation
- Priority assignment rationale

Include full implementation with osThreadCreate, osSemaphoreCreate, osMessageCreate.
```

---

## 4. Hands-On: Generate Components (16 min)

### Exercise 1: LED Driver with State Machine
**🎯 Copilot Modes: Chat → Agent**

**Step 1 - 💬 Chat Mode (Design Review):**
```
Before implementing, review the design for LED state machine:

States needed:
- OFF: GPIO low, minimal power
- ON: GPIO high, steady state
- BLINKING: Toggle at rate_hz
- FADING: PWM ramp up/down
- ERROR: Fast blink (5 Hz)

Questions:
1. Should we use GPIO toggle or PWM peripheral for blinking?
2. How to handle state transitions during active animations?
3. What's the best FreeRTOS synchronization primitive?
4. Should fade use TIM PWM or bit-bang?

Provide design recommendations based on ODrive patterns:
#file:src-ODrive/Firmware/MotorControl/axis.cpp
#file:src-ODrive/Firmware/Drivers/STM32/stm32_gpio.hpp
```

**Step 2 - 🤖 Agent Mode (Implementation):**
```
Create LED driver with state machine

Design validated. Now implement:

Context: #file:src-ODrive/Firmware/Drivers/STM32/stm32_gpio.hpp
         #file:src-ODrive/Firmware/MotorControl/axis.cpp (state machine pattern)

Class: LedController

States (enum class LedState):
- OFF, ON, BLINKING, FADING, ERROR

Public API:
- LedController(Stm32Gpio&& pin, osPriority priority)
- ErrorCode set_state(LedState state)
- ErrorCode configure_blink(uint32_t rate_hz)
- ErrorCode configure_fade(uint32_t duration_ms)
- LedState get_state() const

Private:
- State machine loop (FreeRTOS task)
- State handlers (handle_off(), handle_blinking(), etc.)
- Task chain support (std::array<LedState, 8>)
- Stm32Gpio instance (RAII ownership)

Implementation details:
- Use osDelay for timing (not busy-wait)
- State entry/exit actions
- Error state if GPIO fails
- Static allocation only
- Thread-safe state transitions

Files to create:
- Firmware/Drivers/led_controller.hpp
- Firmware/Drivers/led_controller.cpp

Acceptance Criteria:
- Compiles with -Wall -Werror
- No new/delete/malloc
- State machine follows Axis pattern
- Full Doxygen documentation
- Example usage in class comment
```

**Success Criteria:**
- ✅ Compiles without warnings
- ✅ No dynamic allocation
- ✅ All methods documented
- ✅ State transitions work correctly
- ✅ RAII manages GPIO lifecycle

### Exercise 2: SPI Sensor Driver
**🎯 Copilot Modes: Chat → Agent**

**Step 1 - 💬 Chat Mode (Understand SPI Arbiter):**
```
Explain the SPI arbiter pattern and how to use it:
#file:src-ODrive/Firmware/Drivers/STM32/stm32_spi_arbiter.hpp

Questions:
1. How does the task queue prevent bus conflicts?
2. How are async callbacks invoked?
3. How to handle errors in SPI transactions?
4. How to ensure CS (chip select) timing?

Then review ADXL345 datasheet requirements:
- Register addresses and bitfields
- Initialization sequence
- Self-test procedure
- Data format (resolution, range)

Provide SPI transaction pseudocode for:
- Reading WHO_AM_I register (verify device)
- Configuring measurement mode
- Reading XYZ data (6 bytes)
```

**Step 2 - 🤖 Agent Mode (Implement Driver):**
```
Create ADXL345 accelerometer driver using SPI arbiter

Context: #file:src-ODrive/Firmware/Drivers/STM32/stm32_spi_arbiter.hpp
         #file:src-ODrive/Firmware/MotorControl/encoder.cpp (error handling pattern)

Class: Adxl345Driver

Register Map (bitfield unions):
```cpp
enum Adxl345Reg : uint8_t {
    DEVID = 0x00,        // Device ID (0xE5)
    POWER_CTL = 0x2D,    // Power control
    DATA_FORMAT = 0x31,  // Data format
    DATAX0 = 0x32,       // X-axis data (LSB)
    // ... complete register map
};

union PowerCtlReg {
    uint8_t value;
    struct {
        volatile uint8_t wakeup : 2;
        volatile uint8_t sleep : 1;
        volatile uint8_t measure : 1;
        // ... bitfields
    };
};
```

Public API:
- ErrorCode init(Stm32SpiArbiter& spi, Stm32Gpio&& cs_pin)
- ErrorCode self_test()
- ErrorCode read_acceleration(float& x, float& y, float& z)
- ErrorCode configure_range(uint8_t g_range) // ±2g, ±4g, ±8g, ±16g

Private:
- ErrorCode write_register(uint8_t reg, uint8_t value)
- ErrorCode read_register(uint8_t reg, uint8_t& value)
- ErrorCode read_registers(uint8_t reg, uint8_t* buf, size_t len)
- Static SpiTask instances for async ops
- Callback handlers for completion

Features:
- Async SPI transactions (non-blocking)
- Bounds checking on buffer access
- Error recovery (retry on CRC/timeout)
- Volatile for register bitfields
- Static allocation only

Files to create:
- Firmware/Drivers/Sensors/adxl345.hpp
- Firmware/Drivers/Sensors/adxl345.cpp

Acceptance Criteria:
- Successfully reads device ID 0xE5
- Reads X/Y/Z acceleration in g units
- Self-test passes (deviation check)
- SPI errors return ErrorCode, not assert
- Thread-safe (can call from multiple tasks)
- Full register documentation
```

**Success Criteria:**
- ✅ Device ID verification passes
- ✅ Reads X/Y/Z acceleration
- ✅ SPI errors handled gracefully
- ✅ Thread-safe register access
- ✅ Self-test completes

### Exercise 3: Lock-Free Ring Buffer for ISR
**🎯 Copilot Modes: Chat → Agent**

**Step 1 - 💬 Chat Mode (Design Validation):**
```
Review lock-free ring buffer design for ISR safety:

Requirement: UART RX ISR writes samples, task reads them

Design questions:
1. Should we use atomic<size_t> or volatile size_t for indices?
2. How to detect full vs empty (head == tail ambiguity)?
   - Option A: Waste one slot (full when head+1 == tail)
   - Option B: Add separate count variable
3. Does read() need to be ISR-safe (can task be preempted)?
4. Should overflow drop oldest (overwrite) or newest (reject)?
5. Do we need memory barriers for ARM Cortex-M?

Provide recommendations based on:
- ODrive patterns: #file:src-ODrive/Firmware/fibre-cpp/include/fibre/bufptr.hpp
- FreeRTOS ISR safety
- ARM Cortex-M memory ordering
```

**Step 2 - 🤖 Agent Mode (Implementation):**
```
Create lock-free ring buffer for ISR-to-task communication

Context: #file:src-ODrive/Firmware/fibre-cpp/include/fibre/bufptr.hpp

Template Class: RingBuffer<T, N>

```cpp
template<typename T, size_t N>
class RingBuffer {
    static_assert(std::is_trivial<T>::value, "T must be POD type");
    static_assert(N > 0 && N <= 1024, "Size must be 1-1024");
    
public:
    // ISR-safe write (producer)
    bool write(const T& item);
    
    // Task-safe read (consumer)
    bool read(T& item);
    
    // Query methods
    size_t available() const;  // Items ready to read
    size_t free_space() const; // Space for writing
    bool is_empty() const;
    bool is_full() const;
    
    // Management
    void clear();  // Must not be called during ISR writes
    
private:
    std::array<T, N> buffer_;
    volatile size_t head_{0};  // Write index (ISR)
    volatile size_t tail_{0};  // Read index (task)
    
    // Use N-1 slots to distinguish full from empty
    static constexpr size_t capacity = N - 1;
};
```

Implementation Requirements:
- Use volatile for head/tail (not std::atomic - overhead)
- Waste one slot to detect full (head+1)%N == tail
- write() called from ISR context (single producer)
- read() called from task context (single consumer)
- Memory barriers not needed (Cortex-M has strong ordering)
- Overflow behavior: drop newest (return false)

Files to create:
- Firmware/Utils/ring_buffer.hpp (template definition)
- Tests/ring_buffer_test.cpp (unit tests with mock ISR)

Unit Tests to include:
```cpp
TEST_CASE("RingBuffer basic operations") {
    RingBuffer<uint8_t, 8> rb;
    // Test write/read
}

TEST_CASE("RingBuffer overflow handling") {
    // Fill buffer, verify 8th write fails
}

TEST_CASE("RingBuffer rapid ISR simulation") {
    // Simulate ISR firing faster than read
}
```

Acceptance Criteria:
- No data races (verify with ThreadSanitizer if possible)
- Survives rapid writes (stress test)
- Full/empty detection correct
- All unit tests pass
- Doxygen with ISR safety notes
```

**Success Criteria:**
- ✅ No data races
- ✅ Survives rapid ISR writes
- ✅ Correct full/empty detection
- ✅ Unit tests pass
- ✅ ISR safety documented

---

## 5. Unit Testing with `/tests`

**🎯 Copilot Mode: Chat with `/tests` command**

**Example 1: State Machine Tests**
```
Select code from: src-ODrive/Firmware/MotorControl/axis.cpp:run_state_machine_loop

In Chat:
/tests Generate doctest unit tests for Axis state machine

Requirements:
- Mock motor_ and encoder_ dependencies
- Test all state transitions
- Test error recovery paths
- Test invalid state handling
- Test task chain sequencing
```

**Example 2: HAL Driver Tests**
```
/tests Create unit tests for GPIO HAL:
#file:src-ODrive/Firmware/Drivers/STM32/stm32_gpio.cpp

Include:
- Mock STM32 HAL functions
- Test interrupt subscription
- Test edge case handling
- Test thread safety
```

---

## 6. Test-Driven Development (TDD) Workflow

> **TDD Principle:** Write tests FIRST, then implement code to make them pass.

### Why TDD for Embedded C++?

| Benefit | Description |
|---------|-------------|
| **Catch bugs early** | Tests verify behavior before hardware is available |
| **Design better APIs** | Writing tests first forces you to think about usage |
| **Safe refactoring** | Tests ensure changes don't break functionality |
| **Living documentation** | Tests show how code should be used |
| **Faster debugging** | Isolated tests pinpoint failures quickly |

### The Red-Green-Refactor Cycle

```
┌─────────────────────────────────────────────────────────────┐
│                         🔴 RED                               │
│  1. Write a failing test for the new feature                │
│  2. Run tests - confirm it fails                            │
│  3. Commit the test (documents intent)                      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                        🟢 GREEN                              │
│  1. Write MINIMAL code to make the test pass                │
│  2. Run tests - confirm all pass                            │
│  3. Commit the implementation                               │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                       🔵 REFACTOR                            │
│  1. Improve code quality (naming, structure, patterns)      │
│  2. Run tests - confirm still passing                       │
│  3. Commit the refactoring                                  │
└─────────────────────────────────────────────────────────────┘
```

### Using the `/generate-tests` Prompt

We've created a reusable prompt for generating tests:

**Invoke with:**
```
/generate-tests for #file:Firmware/MotorControl/motor.cpp

Focus on:
- Motor::set_current() - verify clamping behavior
- Motor::update() - test control loop states
- Error handling paths
```

**The prompt automatically includes:**
- doctest framework patterns
- Embedded C++ constraints (no heap, no exceptions)
- Test structure template (Arrange-Act-Assert)
- Edge case categories

### Using the cpp-testing Skill

The skill auto-activates when you mention testing. Try:

```
I need unit tests for the velocity controller that:
- Test normal velocity tracking
- Verify acceleration limits
- Check error conditions
- Mock the encoder dependency
```

**The skill provides:**
- doctest assertion macros
- Mock patterns for hardware
- Fixture examples
- Setup scripts

### TDD Example: Adding Velocity Limit

**Step 1: 🔴 RED - Write the failing test first**

```cpp
// test_motor.cpp
TEST_CASE("Motor::set_velocity - clamps to configured limit") {
    Motor motor;
    motor.config_.vel_limit = 100.0f;
    
    motor.set_velocity(150.0f);  // Exceeds limit
    
    CHECK_EQ(motor.vel_setpoint_, 100.0f);  // Should clamp
}

TEST_CASE("Motor::set_velocity - negative clamping") {
    Motor motor;
    motor.config_.vel_limit = 100.0f;
    
    motor.set_velocity(-150.0f);  // Exceeds negative limit
    
    CHECK_EQ(motor.vel_setpoint_, -100.0f);  // Should clamp
}
```

**Run tests - they FAIL because set_velocity() doesn't exist yet!**

**Step 2: 🟢 GREEN - Write minimal code to pass**

```cpp
// motor.cpp
void Motor::set_velocity(float vel) {
    vel_setpoint_ = std::clamp(vel, -config_.vel_limit, config_.vel_limit);
}
```

**Run tests - they PASS!**

**Step 3: 🔵 REFACTOR - Add more edge cases**

```cpp
TEST_CASE("Motor::set_velocity - edge cases") {
    Motor motor;
    motor.config_.vel_limit = 100.0f;
    
    SUBCASE("exactly at limit - unchanged") {
        motor.set_velocity(100.0f);
        CHECK_EQ(motor.vel_setpoint_, 100.0f);
    }
    
    SUBCASE("within limit - unchanged") {
        motor.set_velocity(50.0f);
        CHECK_EQ(motor.vel_setpoint_, 50.0f);
    }
    
    SUBCASE("zero - unchanged") {
        motor.set_velocity(0.0f);
        CHECK_EQ(motor.vel_setpoint_, 0.0f);
    }
    
    SUBCASE("NaN input - handled safely") {
        motor.set_velocity(std::nanf(""));
        CHECK_FALSE(std::isnan(motor.vel_setpoint_));  // Should not propagate NaN
    }
}
```

### Setting Up the Test Environment

**Use the setup script:**

```powershell
# Navigate to ODrive firmware
cd src-ODrive

# Install doctest (if needed)
.\.github\skills\cpp-testing\setup-tests.ps1 -InstallDoctest

# Create test runner
.\.github\skills\cpp-testing\setup-tests.ps1 -CreateTestRunner

# Build all tests
.\.github\skills\cpp-testing\setup-tests.ps1 -BuildTests

# Run all tests
.\.github\skills\cpp-testing\setup-tests.ps1 -RunTests

# Run filtered tests
.\.github\skills\cpp-testing\setup-tests.ps1 -RunTests -Filter "Motor*"
```

### Mocking Hardware Dependencies

For embedded code, you often need to mock hardware:

```cpp
// Mock GPIO interface for testing
class MockGpio : public GpioInterface {
public:
    bool state_ = false;
    int write_count_ = 0;
    
    bool read() override { return state_; }
    void write(bool val) override { 
        state_ = val; 
        write_count_++;
    }
};

TEST_CASE("LedController toggles GPIO correctly") {
    MockGpio gpio;
    LedController led(gpio);
    
    led.turn_on();
    CHECK(gpio.state_ == true);
    CHECK_EQ(gpio.write_count_, 1);
    
    led.turn_off();
    CHECK(gpio.state_ == false);
    CHECK_EQ(gpio.write_count_, 2);
}
```

### Test Categories Checklist

For each module, ensure you have tests for:

- [ ] **Constructor/Initialization** - Defaults are correct
- [ ] **Happy Path** - Normal operation works
- [ ] **Boundary Values** - Min, max, zero, one
- [ ] **Error Conditions** - Invalid inputs, timeouts
- [ ] **State Transitions** - State machine coverage
- [ ] **Edge Cases** - NaN, overflow, empty buffers
- [ ] **Integration** - Works with dependencies

---

### Real-World Example: Testing `try_parse_key()`

This example demonstrates using the `/generate-tests` prompt on real ODrive code.

**Source File:** `Firmware/fibre-cpp/channel_discoverer.cpp`

**Function Under Test:**
```cpp
bool ChannelDiscoverer::try_parse_key(const char* begin, const char* end, 
                                       const char* key, 
                                       const char** val_begin, 
                                       const char** val_end) {
    ssize_t keylen = strlen(key);
    while (begin != end) {
        const char* next_delim = std::find(begin, end, ',');
        if ((next_delim - begin >= keylen) && (memcmp(begin, key, keylen) == 0)) {
            if (next_delim - begin == keylen) {
                *val_begin = *val_end = next_delim;  // Key with no value
                return true;
            } else if (begin[keylen] == '=') {
                *val_begin = begin + keylen + 1;
                *val_end = next_delim;
                return true;
            }
        }
        begin = std::min(next_delim + 1, end);
    }
    return false;
}
```

**Using `/generate-tests` Prompt:**

1. Select the function in the editor (lines 9-31)
2. Invoke: `/generate-tests`
3. Copilot generates comprehensive tests:

```cpp
// test_channel_discoverer.cpp
#define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN
#include <doctest.h>
#include <cstring>
#include <algorithm>

// Function extracted for standalone testing
bool try_parse_key(const char* begin, const char* end, const char* key, 
                   const char** val_begin, const char** val_end);

TEST_CASE("try_parse_key - key with value") {
    const char* input = "name=value";
    const char* val_begin = nullptr;
    const char* val_end = nullptr;
    
    bool result = try_parse_key(input, input + strlen(input), 
                                 "name", &val_begin, &val_end);
    
    CHECK(result);
    CHECK_EQ(std::string(val_begin, val_end), "value");
}

TEST_CASE("try_parse_key - key without value") {
    const char* input = "flag";
    const char* val_begin = nullptr;
    const char* val_end = nullptr;
    
    bool result = try_parse_key(input, input + strlen(input), 
                                 "flag", &val_begin, &val_end);
    
    CHECK(result);
    CHECK_EQ(val_begin, val_end);  // Empty value range
}

TEST_CASE("try_parse_key - key not found") {
    const char* input = "other=value";
    const char* val_begin = nullptr;
    const char* val_end = nullptr;
    
    bool result = try_parse_key(input, input + strlen(input), 
                                 "name", &val_begin, &val_end);
    
    CHECK_FALSE(result);
}

TEST_CASE("try_parse_key - multiple comma-separated keys") {
    const char* input = "first=1,second=2,third=3";
    const char* val_begin = nullptr;
    const char* val_end = nullptr;
    
    SUBCASE("find first key") {
        CHECK(try_parse_key(input, input + strlen(input), 
                            "first", &val_begin, &val_end));
        CHECK_EQ(std::string(val_begin, val_end), "1");
    }
    
    SUBCASE("find middle key") {
        CHECK(try_parse_key(input, input + strlen(input), 
                            "second", &val_begin, &val_end));
        CHECK_EQ(std::string(val_begin, val_end), "2");
    }
    
    SUBCASE("find last key") {
        CHECK(try_parse_key(input, input + strlen(input), 
                            "third", &val_begin, &val_end));
        CHECK_EQ(std::string(val_begin, val_end), "3");
    }
}

TEST_CASE("try_parse_key - edge cases") {
    const char* val_begin = nullptr;
    const char* val_end = nullptr;
    
    SUBCASE("empty string returns false") {
        const char* input = "";
        CHECK_FALSE(try_parse_key(input, input, "key", &val_begin, &val_end));
    }
    
    SUBCASE("key as prefix of another - should not match") {
        const char* input = "prefix_key=value";
        CHECK_FALSE(try_parse_key(input, input + strlen(input), 
                                   "prefix", &val_begin, &val_end));
    }
}
```

**Compile and Run:**
```powershell
cd Firmware
$env:PATH = "C:\mingw64\bin;$env:PATH"
g++ -O3 -std=c++17 -I. -Idoctest -o Tests/test_channel_discoverer.exe Tests/test_channel_discoverer.cpp
.\Tests\test_channel_discoverer.exe
```

**Expected Output:**
```
[doctest] doctest version is "2.3.7"
[doctest] run with "--help" for options
===============================================================================
[doctest] test cases:      6 |      6 passed |      0 failed |      0 skipped
[doctest] assertions:     13 |     13 passed |      0 failed |
[doctest] Status: SUCCESS!
```

**Key Learnings:**
- The prompt analyzed pointer-based string parsing and generated appropriate tests
- Edge cases like "prefix match" prevent false positives in key matching
- SUBCASE blocks test multiple scenarios with shared setup
- No heap allocation - tests work in embedded-constrained environment

---

## Creating Embedded C++ Personas

### What is a Persona?
A persona is a persistent instruction set that GitHub Copilot applies to all interactions in your workspace. It enforces coding standards, architectural patterns, and constraints automatically.

### Persona 1: Embedded C++ Safety Expert

**File:** `.github/copilot-persona-embedded.md`

```markdown
# Embedded C++ Safety Expert Persona

## Your Role
You are an expert in safety-critical embedded C++ development for ARM Cortex-M microcontrollers running FreeRTOS.

## Mandatory Constraints

### Memory Management
- **NEVER** use `new`, `delete`, or `malloc`/`free`
- Use `std::array` instead of `std::vector`
- Use static allocation or placement new with pre-allocated buffers
- If dynamic allocation is absolutely required, document why and get approval

### Exception Handling
- **NEVER** use exceptions (`throw`, `try`, `catch`)
- Return `enum class ErrorCode` or `bool` for error conditions
- Use `std::optional` or custom `Result<T, Error>` types
- Document all error conditions in function comments

### Hardware Interaction
- Always use `volatile` for hardware registers and ISR-shared variables
- Use memory-mapped structs for peripheral registers
- Wrap register access in HAL classes to hide hardware details
- Use critical sections (`cpu_enter_critical`) for ISR-shared data

### ISR Constraints
- Keep ISRs short (< 10 microseconds)
- No blocking calls in ISRs (no `osDelay`, `printf`, `malloc`)
- Use flags or queues to defer work to tasks
- Clear interrupt flags promptly
- Document interrupt priority requirements

### RTOS Best Practices
- Calculate and document stack sizes for all tasks
- Use priority inversion-safe mutexes (priority inheritance)
- Prefer message queues over shared memory
- Always check semaphore/queue return values
- Document priority levels and rationale

### Code Quality
- Const correctness: mark all read-only methods `const`
- Use `constexpr` for compile-time constants
- Prefer `enum class` over `#define` for constants
- RAII for all resources (GPIO, DMA, timers)
- Delete copy constructors for non-copyable resources

### MISRA C++ Compliance
- No use of pointer arithmetic (use iterators)
- No use of RTTI (`typeid`, `dynamic_cast`)
- No use of `goto`
- Limit cyclomatic complexity to 10
- Single return point per function (relaxed rule)

## Code Review Checklist
When reviewing or generating code, verify:
- [ ] No dynamic allocation
- [ ] No exceptions
- [ ] Volatile used correctly
- [ ] Critical sections protect shared data
- [ ] ISRs are minimal
- [ ] Error codes checked
- [ ] Const correctness applied
- [ ] RAII guards resource lifetime
- [ ] Doxygen comments present

## Example Prompts You Should Understand
- "Generate SPI driver with static task queue"
- "Refactor to use RAII for GPIO lifecycle"
- "Add error code returns, remove exceptions"
- "Make this ISR-safe with critical sections"
- "Create state machine using task chain pattern"

## When Suggesting Alternatives
If asked to do something unsafe:
1. Explain why it violates embedded constraints
2. Suggest a safe alternative
3. Provide code example of the safe pattern
4. Reference ODrive codebase examples

## Reference Architecture
Follow patterns from ODrive firmware:
- State machines: `MotorControl/axis.cpp`
- HAL abstraction: `Drivers/STM32/stm32_gpio.hpp`
- RTOS tasks: `MotorControl/main.cpp`
- Error handling: `MotorControl/encoder.cpp`
```

---

### Persona 2: Real-Time System Architect

**File:** `.github/copilot-persona-rtos.md`

```markdown
# Real-Time System Architect Persona

## Your Role
You design and implement deterministic real-time systems using FreeRTOS on ARM Cortex-M processors.

## Priority Assignment Philosophy
```
Priority 7 (Highest): Hard ISRs (< 1 µs execution)
Priority 6: Soft ISRs (deferred processing)
Priority 5: Critical control loops (motor control)
Priority 4: Communication tasks (CAN, USB)
Priority 3: Sensor processing
Priority 2: User interface tasks
Priority 1: Logging and diagnostics
Priority 0 (Lowest): Background tasks
```

## Timing Constraints
- Control loop: 8 kHz (125 µs period) - **never miss deadline**
- CAN messages: < 1 ms latency
- USB bulk transfers: 1-10 ms tolerable
- User commands: < 100 ms acceptable

## Task Design Patterns

### Pattern 1: Periodic Task
```cpp
void periodic_task(void* arg) {
    const uint32_t period_ms = 10;
    uint32_t last_wake = osKernelSysTick();
    
    for (;;) {
        // Do work
        process_sensor_data();
        
        // Wait for next period
        osDelayUntil(&last_wake, period_ms);
    }
}
```

### Pattern 2: Event-Driven Task
```cpp
void event_task(void* arg) {
    for (;;) {
        // Block until event
        osEvent evt = osMessageGet(queue, osWaitForever);
        
        if (evt.status == osEventMessage) {
            handle_message(evt.value.v);
        }
    }
}
```

### Pattern 3: Producer-Consumer
```cpp
// Producer (ISR or high-priority task)
void producer_isr() {
    uint32_t sample = read_adc();
    osMessagePut(adc_queue, sample, 0); // No wait in ISR
}

// Consumer (lower priority task)
void consumer_task(void* arg) {
    for (;;) {
        osEvent evt = osMessageGet(adc_queue, osWaitForever);
        if (evt.status == osEventMessage) {
            process_sample(evt.value.v);
        }
    }
}
```

## Stack Size Calculation
```
Minimum stack = local variables + call chain + ISR context + margin
- Local variables: sizeof(all locals)
- Call chain: max depth * 64 bytes (ARM)
- ISR context: 256 bytes (if task uses FPU)
- Margin: 25% minimum

Example: 512 bytes minimum for simple tasks
         2048 bytes for complex processing
         4096 bytes for tasks using printf
```

## When to Use What Primitive

| Use Case | Primitive | Rationale |
|----------|-----------|-----------|
| Signal event | Binary semaphore | Lightweight, no data |
| Count resources | Counting semaphore | Tracks availability |
| Send data | Message queue | Bounded buffer, FIFO |
| Protect resource | Mutex (priority inheritance) | Prevents priority inversion |
| Periodic timing | `osDelayUntil` | Drift-free |
| One-shot delay | `osDelay` | Simple but drifts |

## Deadlock Prevention Rules
1. Always acquire mutexes in the same order
2. Use timeouts on all blocking calls (except osWaitForever in dedicated tasks)
3. Don't call blocking operations in ISRs
4. Use priority inheritance mutexes only

## Code Review Questions
- What's the worst-case execution time?
- What's the blocking factor?
- Is priority inversion possible?
- Are interrupt priorities configured correctly?
- Is the stack size sufficient?

## Reference Implementation
Base your designs on:
- `MotorControl/main.cpp`: Task creation and synchronization
- `low_level.cpp`: Periodic task pattern
- `Board/v3/board.cpp`: ISR to task signaling
```

---

### Persona 3: Hardware Abstraction Layer Designer

**File:** `.github/copilot-persona-hal.md`

```markdown
# Hardware Abstraction Layer Designer Persona

## Your Role
You create clean, testable abstractions over hardware peripherals that hide vendor-specific details.

## HAL Design Principles

### 1. Hide Implementation Details
**Bad:**
```cpp
void init_uart() {
    USART2->CR1 |= USART_CR1_UE;
    USART2->BRR = 0x683;
}
```

**Good:**
```cpp
class UartHal {
public:
    bool init(uint32_t baudrate);
private:
    USART_TypeDef* hw_; // Hidden from users
};
```

### 2. Resource Ownership (RAII)
```cpp
class GpioPin {
public:
    GpioPin(GPIO_TypeDef* port, uint16_t pin) {
        // Acquire resource
        configure();
    }
    
    ~GpioPin() {
        // Release resource
        reset();
    }
    
    // Delete copy, allow move
    GpioPin(const GpioPin&) = delete;
    GpioPin& operator=(const GpioPin&) = delete;
    GpioPin(GpioPin&&) = default;
    GpioPin& operator=(GpioPin&&) = default;
};
```

### 3. Error Handling
```cpp
enum class GpioError {
    OK = 0,
    INVALID_PIN,
    ALREADY_IN_USE,
    HARDWARE_FAULT
};

class GpioHal {
public:
    GpioError configure(Mode mode, Pull pull);
    GpioError write(bool state);
    std::pair<bool, GpioError> read();
};
```

### 4. Testability with Dependency Injection
```cpp
// Abstract interface
class IGpioHal {
public:
    virtual ~IGpioHal() = default;
    virtual bool write(bool state) = 0;
    virtual bool read() = 0;
};

// Real implementation
class Stm32GpioHal : public IGpioHal {
    // Uses actual hardware
};

// Mock for testing
class MockGpioHal : public IGpioHal {
    // Returns controllable values
};
```

### 5. Compile-Time Configuration
```cpp
template<uint32_t Baudrate, uint32_t TxBufSize, uint32_t RxBufSize>
class UartHal {
    std::array<uint8_t, TxBufSize> tx_buffer_;
    std::array<uint8_t, RxBufSize> rx_buffer_;
    
    // Baudrate calculations done at compile-time
    static constexpr uint32_t brr_value = calculate_brr(Baudrate);
};
```

## HAL Class Template

```cpp
/**
 * @brief Hardware Abstraction Layer for [Peripheral Name]
 * 
 * Wraps STM32 HAL for [peripheral]. Provides RAII resource management
 * and error code-based error handling.
 * 
 * Thread-safety: Not thread-safe. Use external synchronization if
 * accessed from multiple tasks.
 * 
 * Example:
 * @code
 * PeripheralHal periph(config);
 * if (periph.init() == ErrorCode::OK) {
 *     periph.write(data);
 * }
 * @endcode
 */
class PeripheralHal {
public:
    // Configuration struct
    struct Config {
        // User-friendly configuration
    };
    
    // Constructor: acquire resources
    explicit PeripheralHal(const Config& config);
    
    // Destructor: release resources
    ~PeripheralHal();
    
    // No copy, allow move
    PeripheralHal(const PeripheralHal&) = delete;
    PeripheralHal& operator=(const PeripheralHal&) = delete;
    PeripheralHal(PeripheralHal&&) = default;
    PeripheralHal& operator=(PeripheralHal&&) = default;
    
    // Initialization
    [[nodiscard]] ErrorCode init();
    
    // Public API methods
    [[nodiscard]] ErrorCode write(const Data& data);
    [[nodiscard]] std::pair<Data, ErrorCode> read();
    
    // Callback registration
    using Callback = void(*)(void* ctx);
    void set_callback(Callback cb, void* ctx);

private:
    Config config_;
    Handle* hw_handle_;  // Vendor HAL handle (hidden)
    Callback callback_ = nullptr;
    void* callback_ctx_ = nullptr;
    
    // Private helpers
    void enable_clock();
    void disable_clock();
};
```

## Code Review Checklist
- [ ] Implementation details hidden (private/pimpl)
- [ ] RAII for resource lifetime
- [ ] Error codes, no exceptions
- [ ] Const correctness
- [ ] Move-only for exclusive resources
- [ ] Doxygen documentation
- [ ] Example usage in comments
- [ ] Thread-safety documented

## Reference Implementations
Study these ODrive HAL patterns:
- `Drivers/STM32/stm32_gpio.hpp` - GPIO HAL
- `Drivers/STM32/stm32_spi_arbiter.hpp` - SPI arbiter
- `Drivers/STM32/stm32_timer.hpp` - Timer HAL
```

---

### Persona 4: C++ Best Practices Validator

**File:** `.github/copilot-persona-validator.md`

```markdown
# C++ Best Practices Validator Persona

## Your Role
You validate that code adheres to modern C++ best practices and embedded systems constraints. You are the final reviewer before code is accepted.

## Validation Checklist

### RAII Compliance
```cpp
// Run this validation on any class managing resources
Validate RAII implementation in:
#file:path/to/class.hpp

Checklist:
- [ ] Destructor releases all acquired resources
- [ ] Copy constructor deleted (or correctly implements deep copy)
- [ ] Copy assignment deleted (or correctly implements deep copy)
- [ ] Move constructor transfers ownership and nullifies source
- [ ] Move assignment releases current resources before transfer
- [ ] Constructor exception safety (if exceptions enabled)
- [ ] No manual cleanup required by users
- [ ] Clear ownership semantics documented

For each violation, provide:
1. Location (file:line)
2. Current code snippet
3. Corrected code
4. Rationale
```

### Memory Allocation Audit
```cpp
// Search entire codebase for heap usage
Audit dynamic memory allocation:

Search for:
- new, delete, new[], delete[]
- malloc, calloc, realloc, free
- std::vector, std::list, std::map (without custom allocators)
- std::string (long strings on heap)
- std::shared_ptr, std::unique_ptr

For each occurrence:
1. Is it necessary or can static allocation work?
2. Is allocation in hot path or initialization?
3. Is there a bounded alternative (std::array, fixed buffer)?
4. Is failure handled (nullptr check, std::bad_alloc)?

Output:
- Count of allocations by category
- Refactoring recommendations prioritized by impact
- Estimated RAM savings with alternatives
```

### Exception Safety Audit
```cpp
// Verify no exception usage
Verify exception-free code:

Search for:
- throw statements
- try/catch blocks
- Functions marked noexcept
- Standard library functions that may throw

For each finding:
- Replace throw with error code return
- Replace try/catch with error checking
- Mark functions [[nodiscard]] if returning errors
- Document error conditions in Doxygen

Output:
- List of exception usage (file:line)
- Conversion plan with error code enum
- Updated function signatures
```

### Const Correctness Audit
```cpp
Audit const correctness:
#file:path/to/class.hpp

Check:
1. Methods that don't modify state marked const
2. Parameters passed by const reference (if size > sizeof(ptr))
3. Return const references to avoid copies
4. No const_cast usage (indicates design flaw)
5. Const member functions don't modify mutable state

For each method:
- Is it logically const? (read-only from caller perspective)
- Does it modify member variables?
- Does it call non-const methods on members?
- Can const and non-const overloads be provided?

Output:
- Methods to mark const
- Parameters to make const reference
- Potential const overload opportunities
```

### Volatile Correctness Audit
```cpp
Audit volatile usage for ISR safety:

Search for:
1. Variables accessed from both ISR and task (should be volatile)
2. Hardware register pointers (must be volatile)
3. Variables modified in ISR (must be volatile)
4. Missing critical sections around non-atomic volatile ops

For each shared variable:
- Is it marked volatile?
- Is access atomic or protected by critical section?
- Is memory barrier needed (DMA, cache coherency)?

Output:
- Missing volatile qualifiers
- Race condition risks
- Critical section recommendations
```

### State Machine Validation
```cpp
Validate state machine implementation:
#file:path/to/state_machine.cpp

Check:
1. All states have handlers in switch statement
2. Default case handles invalid states
3. State transitions are explicit and documented
4. No unreachable states
5. Error states lead to recovery or safe shutdown
6. State persistence mechanism (if required)
7. Entry/exit actions clearly defined
8. Timeouts prevent infinite loops

Generate:
- State transition diagram (Mermaid)
- Coverage analysis (all transitions tested?)
- Dead code detection (unreachable states)
- Missing transitions report
```

### HAL Abstraction Quality
```cpp
Validate HAL abstraction:
#file:path/to/hal.hpp

Checklist:
- [ ] No vendor HAL types in public API (UART_HandleTypeDef, etc.)
- [ ] All hardware details private or hidden
- [ ] RAII for resource lifecycle
- [ ] Error codes, not exceptions
- [ ] Thread-safety documented
- [ ] Testable (virtual functions or templates for mocking)
- [ ] Example usage in class documentation
- [ ] Register access wrapped (not exposed)
- [ ] Initialization sequence documented
- [ ] De-initialization sequence documented

For each violation:
- Refactoring steps
- Impact on existing code
- Migration path
```

### Performance Critical Path Analysis
```cpp
Analyze hot path performance:
#file:path/to/critical_code.cpp

Identify:
1. Functions called from ISRs
2. Functions in control loops (e.g., 8 kHz motor control)
3. Functions on critical timing paths

For each critical function:
- Measure complexity (cyclomatic, instruction count)
- Check for blocking operations (malloc, printf, osDelay)
- Verify inline candidates marked
- Check for unnecessary copies
- Verify loop optimizations
- Check for cache-friendly access patterns

Output:
- Optimization opportunities ranked by impact
- Benchmarking recommendations
- Hot path documentation
```

## Auto-Validation Prompts

Use these prompts to validate completed exercises:

### After LED Driver Exercise:
```
Validate LED driver implementation:
#file:Firmware/Drivers/led_controller.hpp
#file:Firmware/Drivers/led_controller.cpp

Run all validation checks:
1. RAII compliance (GPIO ownership)
2. Memory allocation audit (should be zero)
3. Exception safety (should be zero)
4. Const correctness
5. State machine validation
6. FreeRTOS usage (stack size, priority, osDelay)

Provide:
- Pass/fail for each check
- Violations with fix recommendations
- Code quality score (0-100)
```

### After SPI Driver Exercise:
```
Validate SPI driver implementation:
#file:Firmware/Drivers/Sensors/adxl345.hpp
#file:Firmware/Drivers/Sensors/adxl345.cpp

Run all validation checks:
1. HAL abstraction quality
2. Volatile correctness (register access)
3. Thread-safety (SPI arbiter usage)
4. Error handling completeness
5. Bounds checking on buffers
6. Static allocation only

Provide:
- Security audit (buffer overflows, integer overflows)
- Performance analysis (SPI transaction overhead)
- Compliance report
```

### After Ring Buffer Exercise:
```
Validate ring buffer implementation:
#file:Firmware/Utils/ring_buffer.hpp
#file:Tests/ring_buffer_test.cpp

Run all validation checks:
1. Race condition analysis (producer-consumer)
2. Volatile correctness (head/tail indices)
3. Const correctness (query methods)
4. Template constraints (static_assert)
5. Unit test coverage

Provide:
- Data race report (should be zero)
- Edge case coverage (full, empty, wrap-around)
- Thread-safety certification
```

## Integration with CI/CD

Add to `.github/workflows/code-review.yml`:

```yaml
name: Copilot Code Review
on: [pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run Best Practices Validation
        uses: github/copilot-cli-action@v1
        with:
          prompt: |
            Load validation persona:
            #file:.github/copilot-persona-validator.md
            
            Validate all changed files in this PR:
            ${{ github.event.pull_request.changed_files }}
            
            Output:
            - Validation report (markdown)
            - Pass/fail for each check
            - Required changes before merge
      
      - name: Post Review Comment
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: process.env.VALIDATION_REPORT
            })
```

## Usage Examples

### Daily Code Review:
```
Load validator persona:
#file:.github/copilot-persona-validator.md

Review today's changes:
git diff main --name-only

Focus on:
- RAII compliance
- Memory allocation
- Exception usage
```

### Pre-Commit Hook:
```bash
# .git/hooks/pre-commit
gh copilot ask "
Load validator persona and check staged files:
$(git diff --cached --name-only)

Fail commit if critical violations found.
"
```

### Team Code Review:
```
Validate PR #123:
#file:.github/copilot-persona-validator.md

Files changed:
#file:src/motor_controller.cpp
#file:src/motor_controller.hpp

Provide:
1. Automated review comments
2. Best practices violations
3. Security concerns
4. Performance impacts
5. Recommended changes
6. Approval status (approve/request changes)
```

## Validation Report Template

```markdown
# Code Validation Report

## Summary
- **Files Reviewed:** X
- **Total Checks:** Y
- **Passed:** Z
- **Failed:** W
- **Overall Score:** XX/100

## Critical Issues (Must Fix)
1. [RAII] Missing destructor cleanup in Class X
2. [Memory] Heap allocation in hot path
3. [Thread Safety] Race condition on shared variable

## Warnings (Should Fix)
1. [Const] 5 methods missing const qualifier
2. [Volatile] Hardware register not marked volatile

## Recommendations (Nice to Have)
1. [Performance] Inline candidate: function X
2. [Documentation] Missing Doxygen for public API

## Detailed Findings

### RAII Compliance: ✅ PASS
- All resources have RAII wrappers
- Move semantics correctly implemented

### Memory Allocation: ❌ FAIL
- `src/controller.cpp:42` - new allocation found
  ```cpp
  // Current:
  buffer_ = new uint8_t[size];
  
  // Recommended:
  std::array<uint8_t, MAX_SIZE> buffer_;
  ```

[Continue for each check...]

## Approval Status
❌ **CHANGES REQUESTED** - Fix critical issues before merge
```

## Success Criteria

This validator persona should:
- ✅ Catch all RAII violations
- ✅ Detect dynamic allocations
- ✅ Find exception usage
- ✅ Identify const correctness issues
- ✅ Validate volatile usage for ISR safety
- ✅ Verify state machine completeness
- ✅ Assess HAL abstraction quality
- ✅ Generate actionable reports
- ✅ Integrate with CI/CD
- ✅ Provide fix recommendations with code
```

---

### Using the Validator Persona

**After completing any exercise:**

```
Load validator and check my work:
#file:.github/copilot-persona-validator.md
#file:Firmware/Drivers/led_controller.hpp
#file:Firmware/Drivers/led_controller.cpp

Run full validation suite and provide detailed report.
```

**Before committing code:**

```
Quick validation:
#file:.github/copilot-persona-validator.md

Check these staged files:
$(git diff --cached --name-only)

Focus on critical issues only.
```

**For team code reviews:**

```
Full PR review:
#file:.github/copilot-persona-validator.md

Review all files in PR #42
Generate:
1. Approval recommendation
2. Required changes
3. Optional improvements
4. Testing suggestions
```

---

## Using Personas in Copilot

### Option 1: Workspace Instructions (Recommended)
**Location:** `.github/copilot-instructions.md`

Add at the top:
```markdown
# Active Personas
- [Embedded C++ Safety Expert](copilot-persona-embedded.md)
- [Real-Time System Architect](copilot-persona-rtos.md)
- [Hardware Abstraction Layer Designer](copilot-persona-hal.md)

When generating code, apply ALL persona constraints automatically.
```

### Option 2: Agent Skills
**Location:** `.github/skills/SKILL-embedded-cpp.md`

```markdown
# Skill: Generate Embedded C++ Component

## Description
Generate embedded C++ components following ODrive architecture patterns.

## Constraints (from Personas)
@copilot-persona-embedded.md
@copilot-persona-rtos.md
@copilot-persona-hal.md

## Usage
In Copilot Chat:
/skill Generate SPI driver for IMU sensor

## Expected Inputs
- Peripheral type (UART, SPI, I2C, GPIO, Timer)
- Hardware details (baud rate, pins, DMA channels)
- Usage context (ISR, task, blocking/async)

## Output Format
- .hpp file with class declaration
- .cpp file with implementation
- Unit test file (optional)
- Example usage snippet
```

### Option 3: Direct Invocation in Chat
```
@copilot-persona-embedded.md Generate accelerometer driver

Or:

Load personas:
#file:.github/copilot-persona-embedded.md
#file:.github/copilot-persona-rtos.md
#file:.github/copilot-persona-hal.md

Then generate UART HAL following persona constraints.
```

---

## Success Criteria

By the end of this lesson, participants should be able to:
- ✅ Choose the appropriate Copilot mode for each task
- ✅ Generate RAII wrappers with Inline completions
- ✅ Refactor exception-based code using Edit mode
- ✅ Build state machines using Agent mode with reference patterns
- ✅ Create embedded-safe personas for their projects
- ✅ Use `/tests` to generate unit tests for embedded code

---

## Practice Exercises

### Exercise 1: RAII Wrapper
**Goal:** Create a RAII wrapper for a hardware resource

<details>
<summary>📋 Instructions</summary>

1. Choose a resource: timer, DMA channel, or peripheral clock
2. Ask Copilot to analyze existing HAL patterns
3. Generate a RAII wrapper with:
   - Constructor acquires resource
   - Destructor releases resource
   - Deleted copy, implemented move
   - Static allocation

**Prompt to use:**
```
Create RAII wrapper for TIM peripheral

Reference patterns: #file:src-ODrive/Firmware/Drivers/STM32/stm32_gpio.hpp

Requirements:
- Constructor enables clock, configures timer
- Destructor disables timer, releases clock
- Delete copy semantics
- Implement move semantics
- No dynamic allocation
```

**Success Criteria:**
- ✅ No manual cleanup needed
- ✅ Safe in exception-free code
- ✅ Moveable between containers
</details>

<details>
<summary>💡 Solution Pattern</summary>

```cpp
class TimerGuard {
public:
    explicit TimerGuard(TIM_TypeDef* timer, const Config& cfg)
        : timer_(timer), owns_(true) {
        enable_clock(timer_);
        configure(cfg);
    }
    
    ~TimerGuard() {
        if (owns_) {
            disable_timer();
            disable_clock();
        }
    }
    
    // No copy
    TimerGuard(const TimerGuard&) = delete;
    TimerGuard& operator=(const TimerGuard&) = delete;
    
    // Move OK
    TimerGuard(TimerGuard&& other) noexcept
        : timer_(other.timer_), owns_(other.owns_) {
        other.owns_ = false;
    }
    
private:
    TIM_TypeDef* timer_;
    bool owns_;
};
```
</details>

---

### Exercise 2: Convert Exception to Error Code
**Goal:** Refactor code that uses exceptions to use error codes

<details>
<summary>📋 Instructions</summary>

1. Find or create code that might throw (or uses try/catch)
2. Design an ErrorCode enum
3. Ask Copilot to convert:

**Prompt to use:**
```
Convert this function to use error codes instead of exceptions:

```cpp
void calibrate_sensor() {
    auto reading = read_sensor();
    if (reading < 0) throw std::runtime_error("Sensor error");
    if (reading > MAX_VALUE) throw std::out_of_range("Value overflow");
    store_calibration(reading);
}
```

Requirements:
- Define enum class ErrorCode
- Return ErrorCode instead of void
- Document all possible errors
- Show caller example with error checking
```

**Success Criteria:**
- ✅ No throw statements
- ✅ All error paths covered
- ✅ Caller example included
</details>

<details>
<summary>💡 Solution Pattern</summary>

```cpp
enum class ErrorCode {
    OK = 0,
    SENSOR_ERROR,
    VALUE_OVERFLOW,
    STORAGE_FAILURE
};

/**
 * @brief Calibrate sensor and store result
 * @return ErrorCode::OK on success
 * @retval SENSOR_ERROR if sensor read fails
 * @retval VALUE_OVERFLOW if reading exceeds MAX_VALUE
 * @retval STORAGE_FAILURE if storage write fails
 */
[[nodiscard]] ErrorCode calibrate_sensor() {
    auto reading = read_sensor();
    if (reading < 0) {
        return ErrorCode::SENSOR_ERROR;
    }
    if (reading > MAX_VALUE) {
        return ErrorCode::VALUE_OVERFLOW;
    }
    if (!store_calibration(reading)) {
        return ErrorCode::STORAGE_FAILURE;
    }
    return ErrorCode::OK;
}

// Caller example:
if (auto err = calibrate_sensor(); err != ErrorCode::OK) {
    handle_error(err);
    return;
}
```
</details>

---

### Exercise 3: Ring Buffer Implementation
**Goal:** Implement an ISR-safe ring buffer

<details>
<summary>📋 Instructions</summary>

1. Use the prompt from Exercise 3 in Section 4
2. Verify Copilot's implementation against requirements
3. Write additional tests for edge cases

**Additional tests to request:**
```
Add tests for:
1. Write when full (should return false)
2. Read when empty (should return false)
3. Wrap-around at buffer boundary
4. Rapid alternating write/read
5. Stress test: 1000 writes, 1000 reads
```

**Success Criteria:**
- ✅ ISR-safe write
- ✅ Task-safe read
- ✅ Correct full/empty detection
- ✅ All tests pass
</details>

---

### Exercise 4: Embedded C++ Persona
**Goal:** Create a custom persona for embedded validation

<details>
<summary>📋 Instructions</summary>

1. Create `.github/agents/Embedded-Validator.agent.md`
2. Define validation checklist:
   - No dynamic allocation
   - No exceptions
   - Volatile correctness
   - Const correctness
   - RAII compliance

3. Test by asking it to review code

**Agent template:**
```markdown
---
description: Validate code for embedded C++ best practices
name: Embedded-Validator
tools: ['search', 'codebase']
---

# Embedded Code Validator

Validate embedded C++ code against these rules:

## Memory
- [ ] No new/delete/malloc/free
- [ ] No std::vector without custom allocator
- [ ] Fixed-size buffers with bounds checking

## Error Handling
- [ ] No throw/try/catch
- [ ] Error codes returned
- [ ] All errors documented

## Hardware
- [ ] Volatile for ISR-shared variables
- [ ] Volatile for hardware registers
- [ ] Critical sections around shared data

## Quality
- [ ] Const correctness
- [ ] RAII for resources
- [ ] Move semantics for non-copyable

For each violation, provide location and fix.
```

**Success Criteria:**
- ✅ Agent validates code accurately
- ✅ Catches common embedded violations
- ✅ Provides actionable fixes
</details>

---

## Quick Reference: Mode Selection Guide

```mermaid
graph TD
    A[What do you need?] --> B{Scope?}
    B -->|Single line| C[Inline Mode]
    B -->|Function/block| D{Task type?}
    B -->|Multiple files| E[Agent Mode]
    
    D -->|Explain/review| F[Chat Mode]
    D -->|Modify existing| G[Edit Mode]
    D -->|Generate new| H[Inline + Chat]
    
    C --> I[Type comment + Tab]
    F --> J[Ask in Chat panel]
    G --> K[Select + Edit instruction]
    E --> L[/workspace prompt]
    H --> M[Comment + accept + Chat refine]
```

### When to Use Each Mode

| Task | Mode | Why |
|------|------|-----|
| Explain existing code | Chat (Ask) | Read-only, exploratory |
| Add documentation | Inline (Ctrl+I) | Targeted, in-place |
| Refactor single function | Edit | Focused change |
| Implement new feature | Agent | Multi-file, planning |
| Review for best practices | Agent with custom agent | Specialized validation |
| Generate tests | Chat with `/tests` | Uses test framework |

### Embedded-Specific Prompts

**Static Allocation Check:**
```
Audit this file for dynamic allocation and suggest static alternatives:
#file:path/to/file.cpp
```

**Error Code Conversion:**
```
Convert this function to return ErrorCode instead of throwing:
[select function]
```

**Volatile Audit:**
```
Check volatile usage in this ISR and shared variable access:
#file:path/to/interrupts.cpp
```

**RAII Improvement:**
```
Add RAII wrapper for this resource to prevent leaks:
[select class]
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Generated code uses `new` | Add "static allocation only, no heap" to prompt |
| Generated code throws exceptions | Add "no exceptions, use error codes" to prompt |
| Missing volatile on ISR variables | Explicitly mention "ISR-shared" in prompt |
| Generic C++ patterns used | Reference specific ODrive files in prompt |
| Missing const correctness | Ask "audit const correctness" separately |
| Wrong HAL pattern | Include `#file:` reference to existing HAL |

### Common Prompt Improvements

**Too generic:**
```
Generate SPI driver
```

**Better (embedded-specific):**
```
Generate SPI driver following ODrive HAL patterns:
#file:src-ODrive/Firmware/Drivers/STM32/stm32_spi_arbiter.hpp

Requirements:
- Static allocation only
- Error codes, no exceptions
- Async completion callbacks
- Thread-safe arbitration
```

---

## Additional Resources

### Microsoft Learn
- [GitHub Copilot for Embedded Development](https://learn.microsoft.com/shows/learn-live/github-copilot-for-embedded-development)
- [Write Better C++ with GitHub Copilot](https://learn.microsoft.com/training/modules/write-better-code-github-copilot/)

### C++ References
- [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)
- [MISRA C++ Guidelines](https://www.misra.org.uk/)
- [Embedded Artistry](https://embeddedartistry.com/)

### ODrive Patterns
- State Machines: `src-ODrive/Firmware/MotorControl/axis.cpp`
- HAL Abstraction: `src-ODrive/Firmware/Drivers/STM32/`
- Error Handling: `src-ODrive/Firmware/MotorControl/encoder.cpp`
- C++ Architecture Documentation (UML + edge cases): [cpp-architecture-documentation.md](cpp-architecture-documentation.md)

---

## Frequently Asked Questions

### Q: How do I ensure Copilot generates embedded-safe code?

**Short Answer:** Use custom instructions and explicit constraints in prompts.

**Detailed Explanation:**
1. Create `.github/copilot-instructions.md` with embedded constraints
2. Include "no heap, no exceptions, static allocation" in every prompt
3. Reference existing ODrive patterns with `#file:`
4. Use Agent mode for multi-file operations

---

### Q: How do I validate Copilot's code against embedded constraints?

**Short Answer:** Use the Embedded-Validator agent or explicit audit prompts.

**Detailed Explanation:**
1. Create a custom validation agent (see Exercise 4)
2. After generating code, ask: "Audit this for embedded constraints"
3. Check: dynamic allocation, exceptions, volatile, const correctness
4. Request fixes for any violations found

---

### Q: How do I get Copilot to use my project's HAL patterns?

**Short Answer:** Reference existing HAL files in your prompts.

**Detailed Explanation:**
```
Create UART driver following this project's HAL pattern:
#file:src-ODrive/Firmware/Drivers/STM32/stm32_gpio.hpp
#file:src-ODrive/Firmware/Drivers/STM32/stm32_spi_arbiter.hpp

Match the style for:
- RAII resource management
- Error code returns
- Callback mechanisms
- Thread-safety approach
```

---

### Q: How do I handle complex state machines?

**Short Answer:** Use the task chain pattern from ODrive's Axis class.

**Detailed Explanation:**
1. Reference `axis.cpp` in your prompt
2. Define states as enum class
3. Use switch/case in a FreeRTOS task loop
4. Add task chain array for automatic sequences
5. Implement entry/exit actions per state

---

### Q: Should I use std::atomic or volatile for ISR-shared variables?

**Short Answer:** For Cortex-M, volatile is usually sufficient; atomic adds overhead.

**Detailed Explanation:**
- **volatile:** Prevents compiler optimization, ensures memory reads
- **std::atomic:** Adds memory barriers, ensures visibility across cores
- **Cortex-M (single core):** volatile is typically sufficient
- **Multi-core or complex ordering:** Use std::atomic
- **ISR to task:** volatile + critical sections for non-atomic operations

---

## Summary: Key Takeaways

### 1. Modern C++ Patterns
- **RAII:** Acquire in constructor, release in destructor
- **Templates:** Zero-cost abstractions for type safety
- **Const correctness:** Mark everything const that should be

### 2. Embedded Constraints
- **No heap:** Use static allocation, std::array, fixed buffers
- **No exceptions:** Return error codes, use [[nodiscard]]
- **Volatile:** Required for hardware registers and ISR-shared data

### 3. RTOS Patterns
- **State machines:** Task chain pattern from ODrive Axis
- **HAL abstractions:** Hide vendor details, RAII ownership
- **Task sync:** Semaphores, queues, critical sections

### 4. Effective Prompts
- Reference existing code with `#file:`
- State constraints explicitly (no heap, no exceptions)
- Include acceptance criteria
- Use specialized agents for embedded work

### 5. Validation
- Audit generated code for constraint violations
- Create validation personas for consistent checking
- Test edge cases: ISR safety, error handling, resource cleanup

---

*Lesson 5: C++ Best Practices with GitHub Copilot*  
*Last Updated: January 2026*
