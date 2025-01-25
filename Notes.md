# Preface

This document serves as a self-contained introduction to core and modern C++ programming concepts, tailored for beginners preferring intricate details. 
It covers the C++ compilation process, templates, memory management, and several modern C++ features & STLs, along with their associated syntax, example usage, and implementation details.

**Disclaimer**: OOP (classes, inheritance, polymorphism, virtual functions, operator overloading), algos, data structs are NOT covered. 
Familiarity with basic syntax of imperative programming languages is assumed. Basic knowledge of assembly is recommended.

This set of notes was initiated on Dec 9th 2024 and completed on Jan 9th 2025.

---
# C++ Compilation

## Headers (.h) and Source (.cc) Files

### Header Files Typically Contain:
- **Class Declarations**
- **Function Prototypes (Signatures)**
- **Constants, Typedefs, and Template Definitions**

### The Role of `#include`
- `#include` literally instructs the preprocessor to literally copy and paste the entire contents of the included file at the line where it appears. This expanded code is what the compiler processes.
- **Redefinition Error**: C++ prohibits redefining a class with the same name. This could occur when multiple modules are included, such as:
  ```cpp
  #include "Photo.h"
  #include "Album.h" // Album.h includes Photo.h
  ```
  To prevent this, **header guards** are used:
  ```cpp
  #ifndef PHOTO_H
  #define PHOTO_H
  // Class declarations
  #endif
  ```

### Misconceptions About Header Files
Contrary to popular belief, the utility of C++ header files is **NOT to separate interface and implementation**, but rather:
1. To serve as a reference for the compiler to perform type checking.
2. To provide a convenient way to view function APIs without verbose implementation details.
Headers does NOT enable individual compilation of multiple .cc files or funcs. Imagine if there are no headers and inclusions. 
- The fact that non-inline functions are subroutines naturally leads to independent compilation of functions. 
- Each `.cc` file could be independently compiled to subroutine code, and each call site is a jump to subroutine (JSR).
- The linker resolves absolute/relative addresses of all subroutines and jumps.

---

## Inline Functions and `inline` Keyword
- Inline functions are implemented directly in the header file and are like simple in-program functions in an LC-3 assembly program that are not subroutines.
  ```cpp
  inline int add(int a, int b) { return a + b; }
  ```
- `inline` is not a command to the compiler but rather a suggestion; the compiler may ignore the `inline` directive and treat it as a normal function if it sees fit.
  - In modern C++, marking a function as `inline` no longer affects the compiler behavior. Rather, it is used to enable multiple definitions of the same function across translation units (Linker Error without `inline`). This is necessary when implementing non template functions in headers.
---

## Revisiting LC-3 Compilation Process

### LC-3 Machine Code Challenges
When coding directly in machine code, you must use PC-relative addressing, which is tedious:
- Requires manually tracking offsets for branch (BR) or jump to subroutine (JSR) instructions.

### Assembly Streamlines This Process
- **Labels** simplify addressing, allowing the compiler to calculate PC offsets automatically.

#### Compilation Steps in LC-3:
1. **First Pass**:
   - Converts the main program and subroutines into machine code but does **not compute PC offsets**.
   - Produces a symbol table that maps labels to addresses.
2. **Second Pass**:
   - Uses the symbol table to calculate PC offsets for branch and JSR instructions, finalizing the machine code.

---

## C++ Compilation Process

### Steps in Compilation
1. **Preprocessing**:
   - Handles all `#` directives and performs text substitution (e.g., copying header files into the source code).
2. **Compilation**:
   - Each `.cc` file is compiled into an **object file** (`.obj` or `.o`), also known as a **translation unit**.
   - Header files are **not compiled** but only included. They do not produce machine code.
   - During compilation:
     - A symbol table for function calls is created, but actual addresses remain unresolved.
     - Machine code is generated for the main program and subroutines, but function calls are incomplete.
   - This is analogous to an **incomplete first pass** in LC-3 (symbol table creation only).
3. **Assembly** (optional):
   - Some compilers skip this step in favor of intermediate representations (IR) or direct machine code generation.
4. **Linking**:
   - Combines all object files into the final executable (e.g., `.exe` or `.out`).
   - Resolves function call addresses, finalizing symbol table and replacing placeholders in the machine code with actual addresses.
   - This step corresponds to the **second pass** in LC-3.

#### Why We Don’t Include `.cc` Files
Including `.cc` files is essentially inlining the funcs, which results in:
1. Paste the same subroutine code into multiple files, resulting in wasteful duplication.
2. Require recompilation of all dependent files whenever the .cc file is edited. E.g. all PC offsets may need to be recalculated. 

---

## Statically-Typed Languages

### Static Typing in C++
Static typing is a key feature of languages like C++ and Java. It:
1. Prevents **undefined behavior** by enforcing type constraints at compile time.
2. Detects errors earlier compared to dynamically-typed languages like Python, reducing runtime error rates.

### Type Checking in Compilation
- The compiler requires the **function signature** (declaration) before the call site to:
  - Ensure type correctness for arguments and return values.
  - Prevent mismatched types during function calls.
- If types do not match, the compiler raises an error during compilation.

#### Cost of Static Typing
While static typing reduces runtime errors, it imposes extra burdens on programmers:
1. **Header Files**:
   - Programmers must write and include headers for type checking.
   - The compiler uses this information to verify function signatures and argument types.
2. **Declaration First**:
   - Function declarations must appear before their call sites.
   - The compiler processes code in a **top-down order**, requiring declarations for type checking before the implementation. 
---

# Template Functions in C++

Template functions allow for the creation of generic functions that work with a variety of data types. They are a powerful compile-time abstraction in C++ and differ significantly from regular functions in terms of implementation, instantiation, and machine code generation.

---

## Syntax of Template Functions

The basic syntax for defining a template function is:

```cpp
template <typename T1, typename T2, ...>
return_type function_name(parameters) {
    // Function body using T_i as a placeholder for any data type
}
```

### Key Features:
1. **Automatic Type Deduction**:
   - The compiler automatically deduces the types for both the arguments and the return value based on the function call.
   - This is more flexible than `auto` functions, which still require specifying argument types explicitly.


## Template Instantiation and Machine Code Generation

### Templates Exist as Blueprints
- Templates in C++ do not generate any machine code by themselves.
- A template that is **not instantiated** remains as a **compile-time abstraction** and does not contribute to the final machine code.
  
### Machine Code Creation on Instantiation
- Machine code for a template function is generated **only when it is instantiated** with a specific set of types.
- Instantiation occurs when a template function is called with concrete types, e.g.:
  ```cpp
  template <typename T>
  T add(T a, T b);

  int result = add<int>(3, 4); // Instantiates add<int>
  ```

---

## Why Templates Are Implemented in Headers

C++ templates are implemented in header files due to their unique nature:
1. **Templates Are Shared Across Translation Units**:
   - `.cc` files only call **instantiated templates** but do not define or implement them.
   - Since templates are designed to be reused by multiple translation units, their definitions must be accessible wherever they are instantiated.
   
2. **Blueprint Inclusion**:
   - The template definition is included in any `.cc` file that instantiates it.
   - For example:
     ```cpp
     template <typename T>
     T add(T a, T b) {
         return a + b;
     }
     ```
     This definition must be included in the header file so that the compiler can use it as a blueprint to create machine code for specific type instances.

---

## Practical Implementation of Templates

When implementing templates, follow these best practices:

### Declaration Inside the Class
Declare template functions inside the class without specifying `template <typename T1, typename T2, ...>`. Example:
```cpp
template <typename T>
class MyArray {
public:
    void push_back(const T& value); // Declaration
};
```

### Implementation Outside the Class
Provide the implementation of template functions **outside the class**, but still inside the header file, with the `template <typename T1, typename T2, ...>` syntax:
```cpp
template <typename T>
void MyArray<T>::push_back(const T& value) {
    // Function implementation
}
```

By separating the declaration and implementation, templates remain modular and easier to manage.

---

## One Definition Rule (ODR) and Redundancy Prevention

The **One Definition Rule (ODR)** states that a function must be defined exactly once across all translation units. 
It ensures:
   - **No Duplicate Instantiation**: A template function's instantiation for a specific type is generated only once for the final machine code.
   - During linking, the linker detects multiple translation units providing the same machine code for a template instantiation (e.g., `add<int>`) or a inline function (e.g. inline functions in headers).
   - **Linker Deduplication**: The linker eliminates redundant copies, retaining only one instantiation in the final binary.
   - **One Definition**: A Linker Error occurs if a function that's neither a template instantiation nor marked as inline is defined across multiple translation units.

This behavior ensures efficiency and prevents bloat in the compiled code.

### Template Functions and Inline Behavior
Template functions are effectively treated as **inline functions**:
- They are instantiated wherever they are used.
- However, redundant copies are eliminated during linking, just like inline functions.

---

## The Role of `typename` in Templates

The `typename` keyword is required when dealing with **dependent types** in templates. A dependent type is a type that depends on a template parameter.

### Why `typename` Is Needed
When the compiler encounters a dependent type like `MyArray<T>::iterator`, it must determine whether `iterator` is:
1. A **type** (e.g., a nested class or `typedef`).
2. A **static member**.

By default, the compiler assumes it is a static member unless explicitly told otherwise. To clarify that it is a type, you must use `typename`:
```cpp
template <typename T>
void displayElements(MyArray<T>& array) {
    typename MyArray<T>::iterator it = array.begin(); // Explicitly specify `iterator` as a type
    for (; it != array.end(); ++it) {
        std::cout << *it << " ";
    }
}
```

Without the `typename` keyword, the compiler would raise an error, as it cannot infer the nature of the dependent type.

---

### Variadic Templates
Variadic templates allow a function or class to accept a variable number of template arguments.
```cpp
template <typename... Args>
MyUniquePtr<T> makeMyUniquePtr(Args&&... args) {
    return MyUniquePtr<T>(new T(std::forward<Args>(args)...)); 
    // Upon instantiation the constructor for that specific arrgmt of args num&type will be invoked
}
```
- `typename... Args`: This declares a parameter pack named `Args`:
  - Can accept any number of **type arguments** (not values), including zero.
  - `Args` is not a container but a compile time abstraction.
  - Cannot do: `cout << Args`, but you can get the number of types with the operator `sizeof...()`
  - Needs to be expanded in func declaration, as `Args` (without `...`) refers to the pack itself, not the individual types within the pack. A func signature expects the latter.
- `args`: This declares a function parameter pack named `args`:
  - represents actual params passed, appears in func param list
  - Can print values in args through expansion: `cout << ... << args`, but still not a STL container.
  - Needs unpacking with `...` when used inside func body. Usage:
    - `f(args...)` is interpreted as `f(arg1, arg2, arg3)`
    - `f(g(args)...)` is interpreted as `f(g(arg1), g(arg2), g(arg3))`
    - A standalone `f(args)` is never correct when dealing with parameter packs because it attempts to pass the entire pack as a single argument.
    - In `std::cout`: to print each arg, use `(std::cout << ... << args) << '\n'` 
  - **Double Expansion Is Redundant**: In func declaration `void f(Args... args)`, a second ellipsis after args (args...) is unnecessary because the compiler already knows that args is a parameter pack by its association with the expanded `Args...`.

---

# C++ Functors/Function Objects

A function object is any class that implements the **function call operator()**. This allows instances of the class to be used like functions. 


Unlike a regular function, a functor can maintain state through its member variables, enabling a Finite-State Machine-like behavior.

Example:
```cpp
#include <iostream>
class BankAccount {
private:
    double fund;
    
public:
    void operator()(double deposit) {
        fund += deposit;
        fund <= 0 ? std::cout << "Bankrupt!\n" : std::cout << "Balance: " << fund << std::endl;
    }
};

int main() {
    double check = 300.0;
    BankAccount clerk;
    clerk(check);
    return 0;
}
```

---

# C++ lambda functions

Lambdas are Local Variable-like function objects that capture variables. 
- "Syntactic sugar" for function objects: they're basically a more convenient way to write small functor classes.

## Syntax

```cpp
// (declaring variables to be captured)
// assigning to a variable and calling:
auto func = [capture](parameters)->return-type{body} // "->return-type" can be omitted starting from C++14, as the compiler could deduce it.
func(params);
// calling immediately without assigning to a variable:
[capture](parameters)->return-type{body}(); // the () op following the declaration calls the lambda immediately.
```
## Differences between lambdas and Regular Functions:
Lambdas:
- Exists only within the scope where it is defined.
- Anonymous by default.
- Can capture variables from the **enclosing scope** and **current scope** surrounding scope using a capture list.
- Auto deduces the return type, does not require explicitly defining return type.
- Cannot be virtual or overridden.
- When you write a lambda, the compiler conceptually creates an unnamed class type with an overloaded operator().

## Capturing syntax

| Syntax       | Description                                                                                       | Comment                                                                                                      |
|--------------|---------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------|
| `[]`         | No external variables are captured.                                                                | Any use of an undeclared variable inside the lambda will result in an error.                                  |
| `[x, &y]`    | `x` is captured by value, and `y` is captured by reference.                                         | Useful when specific variables need to be captured in different ways, e.g., to prevent unintended side effects. |
| `[&]`        | All external variables used in the lambda are captured by reference.                               | This is a shorthand for referencing all variables used in the lambda by reference.                           |
| `[=]`        | All external variables used in the lambda are captured by value.                                   | This is a shorthand for capturing all variables used in the lambda by value, avoiding unintended references.  |
| `[&, x]`     | All external variables are captured by reference except `x`, which is captured by value.          | Useful when you want most variables by reference but need to specifically capture one by value.               |
| `[=, &z]`    | All external variables are captured by value, except `z`, which is captured by reference.          | Allows selective modification of variables while avoiding unintended side effects from reference captures.     |

Note: 
- `this` ptr (and member vars associated with it) also has to be explicitly captured as `[this]`.
- lambdas can capture vars by name iff they have been declared before the lambda declaration.
- A lambda's captures are determined and fixed at the point where the lambda is declared, regardless of:
  - Where the lambda is later called
  - Changes to variables after capture (for capture-by-value, not for capture-by-ref)
- There's no way to capture variables from inner scopes or variables that will be declared later, even with `[&]`.

---

# std::function

`std::function` is a callable wrapper that could store, copy, and invoke any callable with a specific signature. While providing significant convenience, it does incur a little runtime overhead compared to direct function calls and func ptr.



**Syntax**
```cpp
std::function<void()> f;
std::function<int(int)> g = [](int x) { return x * x; };
std::function<int(int)> h = f; // copy
std::function<int(int)> j = std::move(h) // move
auto lambda = [](int x, int y) { return x + y; };
std::function<int(int)> k = lambda // construct by storing a callable
if (k)  {int result = k(5);}
k.reset(); // empty
```

## Benefit: Flexible, Clean, and Safe Handling of Different Callable Types:  
Assume we have a factory function that uses callbacks (callables invoked under certain conditions).
 One  would have to handle different types of callables (free funcs, lambdas, functors) with 
 different overloads and explicit type declarations (free funcs (as func ptr), lambdas (with specific type of lambdas, yielded by `decltype(lambda_name)`), functors (as functor type))


`std::function` abstracts away the specific type of the callable, allowing different types of callable entities (e.g., free functions, lambdas, or functors) to be treated uniformly as long as they conform to the same signature.

```cpp
#include <iostream>
#include <functional>  // For std::function

// Free function
void freeFunction() {
    std::cout << "Free function called" << std::endl;
}

// Functor
struct Functor {
    void operator()() const {
        std::cout << "Functor called" << std::endl;
    }
};

// Lambda (with explicit type)
auto lambda = []() {
    std::cout << "Lambda called" << std::endl;
};

// Specialized functions accepting different callable types is necessary
void callFree(void (*func)()) {
    func();
}

template <typename Callable>
void callFunctor(Callable func) {
    func();
}

// Using std::function to handle different callables uniformly
std::function<void()> func;

// Assigning a free function
func = freeFunction;
func();  // Output: Free function called

// Assigning a lambda
func = []() {
    std::cout << "Lambda called" << std::endl;
};
func();  // Output: Lambda called

// Assigning a functor
func = Functor();
func();  // Output: Functor called
```

Containers like `std::vector` or `std::map` cannot store heterogeneous callable types directly. By wrapping callables in std::function, you can store and manage them uniformly.

```cpp
#include <iostream>
#include <functional>
#include <vector>

// A function to execute all stored callables
void executeAll(const std::vector<std::function<void()>>& callables) {
    for (const auto& callable : callables) {
        callable(); // Execute each callable
    }
}

// Standalone function
void sayHello() {
    std::cout << "Hello from a function!" << std::endl;
}

int main() {
    // Lambda function
    auto greetLambda = []() {
        std::cout << "Hello from a lambda!" << std::endl;
    };

    // Functor
    struct GreetFunctor {
        void operator()() const {
            std::cout << "Hello from a functor!" << std::endl;
        }
    };

    // Create a vector of std::function<void()>
    std::vector<std::function<void()>> callables;

    // Store heterogeneous callable types
    callables.push_back(sayHello);        // Function pointer
    callables.push_back(greetLambda);     // Lambda
    callables.push_back(GreetFunctor());  // Functor

    // Execute all callables
    executeAll(callables);

    return 0;
}
```

A key difference between func ptrs and `std::function` is that func ptrs allows implicit conversions while `std::function` enforces exact type matching and will not allow implicit conversions.
```cpp
#include <functional>

void foo(long x) {}

int main() {
    // Function pointer - allows implicit conversion
    void (*fptr)(int) = foo;  // Compiles! Even though foo takes long

    // std::function - stricter type checking
    std::function<void(int)> f = foo;  // Compiler error: cannot convert 'void(long)' to target type
    
    return 0;
}
```



---

# C++ Runtime

## Stack

The **stack** is a region of memory used for function calls and local variables. It operates in a **Last In, First Out (LIFO)** manner, with simple linear growth. The stack typically grows **upward to lower (in value)** memory addresses (a convention in modern architectures). It allows for nested function calls and is managed using a **stack pointer** (SP) that points to the top of the stack. 

Data at any location on the stack with an address higher (in value) than the stack pointer can be accessed via `stack_ptr + offset`, but cannot be inserted or removed. 

### Example Access Operations:
- **Access data**: `LDR R6, #offset`
- **Push**: 
  ```assembly
  STR R6, #-1
  ADD R6, R6, #-1
  ```
- **Pop**:
  ```assembly
  ADD R6, R6, #1
  ```

Note: When popping, the value is not explicitly deleted but is nullified and later overwritten.

---

## Stack Frames

A **stack frame** is a memory structure on the stack that stores:
- Local variables
- Parameters
- Other essential data such as the return address and saved registers.

The structure of the stack frame is determined at **compile time**, while the actual memory is allocated at **runtime**.

### Stack Frame Structure

| **Section**         | **Description**                                                                                                   |
|----------------------|-------------------------------------------------------------------------------------------------------------------|
| Return Address (PC)  | Address of the instruction in the caller function to return to.                                                  |
| Saved Base Pointer   | Previous base pointer (BP/EBP) to restore the caller's frame.                                                   |
| Function Arguments   | Space for function arguments (passed by value/reference).                                                       |
| Local Variables      | Space for the function's local variables.                                                                       |
| Saved Registers      | Borrowed registers to be preserved across calls (callee-saved registers).                                       |
| Stack Pointer        | Points to the top of the stack, moves as data is pushed/popped.                                                 |

**Memory Layout (high address to low address)**:

Each time a function is called:
1. A new stack frame is created.
2. When the function returns, the stack frame is discarded.

This structure supports nested function calls and is essential for managing execution state, particularly in **recursive functions**.

---

## Example: LC-3 Assembly Program

This LC-3 assembly program demonstrates the use of stack frames in a subroutine. It includes the typical **function prologue**, **body**, and **epilogue**, ensuring proper return of the program counter (PC) and stack pointer (SP) in nested function calls.

```assembly
.ORIG x3000       ; Start of program

;-------------------------------
; Constants and Memory Allocation
;-------------------------------
STACK_START  .FILL x4000          ; Stack starts at x4000
RET_ADDR     .FILL 0              ; Placeholder for return address

;-------------------------------
; Main Program
;-------------------------------
MAIN
    LEA R6, STACK_START          ; Initialize the stack pointer (R6)
    LD R0, ARG_VAL               ; Load argument value (e.g., 10) into R0
    JSR FUNC                     ; Call FUNC (JSR saves the return address in R7)
    HALT                         ; Stop program after return

;-------------------------------
; Function (FUNC)
;-------------------------------
FUNC
    ; Function Prologue
    STR R7, R6, #-1              ; Save return address on the stack
    ADD R6, R6, #-1              ; Adjust SP
    STR R5, R6, #-1              ; Save base pointer (R5) of the caller
    ADD R6, R6, #-1              ; Adjust SP
    ADD R5, R6, #0               ; Set new base pointer (R5 = SP)
    STR R4, R6, #-1              ; Save callee-saved register (R4)
    ADD R6, R6, #-1              ; Adjust SP

    ; Function Body
    ADD R4, R0, #0               ; Use R4 to hold argument value
    LD R0, VALUE                 ; Load a value into R0 (e.g., 5)
    ADD R0, R4, R0               ; Compute sum (argument + VALUE) and save the result in R0

    ; Function Epilogue
    LDR R4, R6, #0               ; Restore callee-saved register (R4)
    ADD R6, R6, #1               ; Adjust SP
    ADD R6, R5, #0               ; Restore stack pointer to saved BP
    LDR R5, R6, #0               ; Restore previous base pointer
    ADD R6, R6, #1               ; Adjust SP
    LDR R7, R6, #0               ; Restore return address
    ADD R6, R6, #1               ; Adjust SP

    RET                          ; Return to caller

;-------------------------------
; Data Section
;-------------------------------
ARG_VAL      .FILL 10             ; Example argument value
VALUE        .FILL 5              ; Example value for function body

.END
```

### Schematic of the Stack During Execution of `FUNC` (at Maximum Depth)

| **Address** | **Content**                          |
|-------------|--------------------------------------|
| x3FFC       | Saved R4                             |
| x3FFD       | Saved base pointer R5               | *(current function's BP is also here)* |
| x3FFE       | Return address R7                   |

---

## Why Include a Base Pointer (BP)?

In most cases, the **base pointer (BP)** might seem redundant because the **stack pointer (SP)** naturally points to the same position as the saved BP in the stack during the function epilogue. However, the BP is still useful in specific cases:

### Benefits of Using BP
1. **Accessing Function Arguments**: 
   - Arguments passed to a function (above the return address) are easier to access with a fixed reference point (BP) rather than recalculating offsets from the dynamic SP.
   - Without BP, additional resources are required to track the SP and offsets.
   
2. **Debugging**:
   - Debuggers rely on the consistent placement of BP to trace function calls and retrieve local variables.

3. **Compiler and Calling Conventions**:
   - Many compilers and calling conventions enforce the use of BP for compatibility and maintainability, even when technically redundant.

---

# Exception Handling

## Overview

In C++, any function that isn't marked `noexcept` could potentially throw exceptions, either directly or indirectly through functions it calls. To handle this possibility, the compiler inserts **implicit exception handling infrastructure** into functions, even if you do not write explicit `try-catch` blocks.

### Example Function Without Explicit Exception Handling

```cpp
void normal_function() {
    std::string str = "hello";  // Might throw on allocation
    process(str);               // Might throw
}
```

### Compiler's Conceptual Behavior

Although the compiler does not generate C++ code internally, the conceptual process can be illustrated as follows:

```cpp
void normal_function() {
    // Implicit try block starts
    void* stack_cleanup_info = __register_cleanup_info();  // Register stack info for unwinding
    
    std::string str = "hello";
    process(str);

    __cleanup_successful(stack_cleanup_info);  // Normal cleanup
    return;
    
    // Implicit exception handling
    __handle_exception:
        // Clean up local variables (call destructors)
        str.~string();
        // Pass exception to caller
        __unwind_stack_and_continue(stack_cleanup_info);
}
```

### Key Steps in Exception Handling
1. **Resource Cleanup**: Local variables (e.g., objects) are cleaned up by calling their destructors.
2. **Stack Unwinding**: The stack is unwound to restore the previous function's state (similar to LC3 function epilogues).
3. **Exception Propagation**: The exception is passed to the caller.
4. **Metadata Usage**: Exception metadata (e.g., tables) are used to map exceptions to their handlers.

---

## Exception Handling in Assembly

### x86-64 Assembly Support

In x86-64 assembly, exception handling infrastructure is supported through directives such as `.cfi_`. These provide **Call Frame Information (CFI)** to manage stack unwinding and exception handling. CFI metadata is essential for tools like debuggers and exception handlers.

#### Example: Exception-Handling Enabled Function

```asm
regular_function:
    ; Function setup
    push    rbp                  ; Save old stack frame pointer
    mov     rbp, rsp             ; Set up new stack frame
    
    ; Exception handling setup
    .cfi_startproc               ; Start of exception info
    .cfi_personality 0x3         ; Info about how to handle exceptions
    .cfi_lsda 0x3                ; Exception table location (Language-Specific Data Area)
    
    ; Actual function code
    mov     eax, edi             ; Move parameter to eax register
    add     eax, 1               ; Add 1 to it
    
    ; Function cleanup
    pop     rbp                  ; Restore stack frame
    ret                          ; Return from function
    
    ; Hidden exception data
    .section    .gcc_except_table ; Exception handling data
    ; ... more exception table data ...
```

---

## `noexcept`

### What Is `noexcept`?

`noexcept` is a promise to the compiler that the function will not throw exceptions. This allows the compiler to generate simpler and more efficient code by omitting exception-handling infrastructure.

### Compiler Behavior with `noexcept`

- The compiler may statically analyze whether the function can throw and generate a **compilation error** if it detects a potential exception in a `noexcept` function.
- If a `noexcept` function attempts to throw, the program will terminate via `std::terminate`.

---

### Assembly Comparison: `noexcept` vs Regular Functions

#### Regular Function

```asm
regular_function:
    ; Function setup
    push    rbp                  ; Save old stack frame pointer
    mov     rbp, rsp             ; Set up new stack frame

    ; Exception handling setup
    .cfi_startproc               ; Start of exception info
    .cfi_personality 0x3         ; Info about how to handle exceptions
    .cfi_lsda 0x3                ; Exception table location
    
    ; Actual function code
    mov     eax, edi             ; Move parameter to eax register
    add     eax, 1               ; Add 1 to it
    
    ; Function cleanup
    pop     rbp                  ; Restore stack frame
    ret                          ; Return from function
```

#### `noexcept` Function

```asm
safe_function:
    ; Function setup
    push    rbp                  ; Save old stack frame pointer
    mov     rbp, rsp             ; Set up new stack frame

    ; Actual function code
    mov     eax, edi             ; Move parameter to eax register
    add     eax, 1               ; Add 1 to it
    
    ; Function cleanup
    pop     rbp                  ; Restore stack frame
    ret                          ; Return from function
```

**Key Difference**: The `noexcept` function omits exception handling setup (e.g., `.cfi_` directives, exception tables).

---

### Consequences of Throwing in a `noexcept` Function

If a `noexcept` function throws an exception, the program terminates immediately. 

#### Example: Program Termination (`std::terminate`)

```asm
__cxa_bad_exception:
    JSR terminate_program        ; Call to terminate the program

terminate_program:
    HALT                         ; Stop execution or trigger termination logic
```

#### Consequences:
1. **No Stack Unwinding**: The stack is not unwound.
2. **No Destructor Calls**: Destructors for local objects are not called.
3. **Resource Leaks**: Resources may leak (though the OS typically cleans them up when the program terminates).

---

## Why Are Move Operations Marked `noexcept`?

### Reasoning

- **Move Operations**:
  - Transfer ownership of resources (e.g., pointers).
  - Do not allocate new resources or perform complex operations.
  - Are inherently safe and unlikely to throw exceptions.

- **Copy Operations**:
  - Involve allocating new resources (e.g., memory on the heap).
  - Can fail (e.g., heap overflow).

In addition, during certain container operations such as resizing, the compiler will prioritize the use of move whenever possible.

```cpp
T data;
std::vector<T> vec;
vec.push_back(data);  // Uses move for T if noexcept, otherwise falls back to copy
```


# C++ Dynamic Memory
## Scope

When you group lines of expression into `{}`, you create a new **local scope/block**.
- Vars declared inside the block are local to that block and are destroyed as scope ends.
- Vars in an outer scope can be shadowed (i.e. using the same name) by vars in inner scopes.
  ```cpp
    int x = 5;
  {
      int x = 10; // Shadows the outer x
      std::cout << x << std::endl; // Prints 10
  }
  std::cout << x << std::endl; // Prints 5 (outer x)
  ```


## Shallow Copy vs Deep Copy

- **Shallow Copy** and **Deep Copy** behave the same when objects consist only of primitive types or fixed-size data.
- The distinction becomes significant when objects contain pointers to dynamically allocated memory (e.g., `new` or `malloc` in C++):
  - **Shallow Copy**:
    - Copies only the pointer value, not the memory it references.
    - Both objects share the same memory location, so modifying one affects the other.
    - Example:
      ```cpp
      this->data = src.data; // Shallow copy
      ```
  - **Deep Copy**:
    - Duplicates the actual memory referenced by the pointer, ensuring independent data in the copied object.
    - Example:
      ```cpp
      for (int i = 0; i < src.size; ++i) {
          *(this->data + i) = *(src.data + i); // Deep copy
      }
      ```

- **Summary**:
  - Shallow copy: creates a reference.
  - Deep copy: creates an identical but independent object.
  - By default, `a = b` creates a shallow copy. This is analogous to `str1 = str2` in Java, where only a reference is created.
  - The copy constructor and assignment op corresponds to deep copying.
  - Shallow copy is not entirely useless! We will revisit it again in move semantics.

- **Overloading the Assignment Operator**:
  - To ensure deep copying during assignments, override the assignment operator in classes:
    ```cpp
    MyClass& operator=(const MyClass& other) {
      if (this != &other) { // Note: compare ptr not obj, obj may not have comparison overridden
        // copy logic 
      }
    }
    ```
  - This operator ensures deep copying when using `obj1 = obj2`.

---

## Memory Leak

- A **memory leak** occurs when a program allocates memory but fails to release it after it's no longer needed. This creates a situation where the memory is inaccessible to the program, akin to water leaking out of a pipe—it’s lost, but still exists.
 
- Example of memory leak:
  ```cpp
  int* ptr = new int[10];
  ptr = data; // Memory previously allocated is now orphaned
  ```
### delete

  `delete` acts on **pointers pointing to heap-allocated memory only**. When a ptr is deleted, a "deep cleaning" executes automatically as follows:
  1. The object's destructor is called, if any.
  2. Member objects' destructors are called, if any. (facilitating a recursive deep cleaning)
  3. The memory of the object itself is deallocated (on the heap). For trivial types, this is the only step.

- delete[] acts on pointers pointing to heap arrays ("`new T[size]`"), iterating the above process for the array.
- **Best Practice**:
  - Always `delete` dynamically allocated memory before reassigning a pointer or when no longer using a pointer:
    ```cpp
    delete[] ptr; // Prevent memory leaks
    ```
- **No double deletion**: Once memory is deallocated, it is returned to the system or reused. Re-deleting such memory can corrupt the heap or interfere with other allocations.

---

## `malloc` and `free`

- **`malloc`** is an alternative to `new` for dynamic memory allocation in C++.
  - Syntax:
    ```cpp
    int* ptr = (int*)malloc(5 * sizeof(int));
    ```
  **Comparison with `new`**:
  - `new` returns `T*`, while `malloc` returns a `void*` pointer, which needs to be type-casted with `(T*)`.
  - `new` can take an addresss as a param: `new (address) Type(args...);` (placement new), which doesn't alloc mem instead calls the constructor at the specified mem addr. `malloc` cannot accept an address as a parameter.
    - When an object is constructed using placement new, it must also be explicitly destroyed using an explicit destructor call because the memory was manually allocated and does not automatically trigger destruction.
- **`free`** is used to deallocate memory allocated with `malloc`:
  ```cpp
  free(ptr);
  ```
- **Note**: In C++, prefer `new` and `delete` for memory management. malloc is more of a C thing.

---

## The Rule of Three and RAII

- If a class involves resource management (e.g., dynamic memory allocation), it should implement:
  1. **Destructor**:
     - Releases dynamically allocated memory of the obj. 
      - NOTE: the destructor should NOT release the resrces it doesn't directly manage, i.e. those owned by its non-trivial members, nor should it call the destructors of these non-trivial members. As discussed in the section on `delete`, destructors are automatically recursively called.
      - The destructor is rarely explictly called. It is meant to be automatically invoked when obj goes out of scope, and manual calling could result in double deletion. There are exceptions where it should be explicitly called, e.g. in unions, placement new, partial destruction, etc.
     - Example:
       ```cpp
       ~MyClass() {
           delete[] data;
       }
       ```
  2. **Copy Constructor**:
     - Defines how a new object is initialized as a copy of an existing object.
     - Example:
       ```cpp
       MyClass(const MyClass& other) {
           // Deep copy logic here
       }
       ```
  3. **Copy Assignment Operator**:
     - Defines how an object is assigned to another existing object.
     - Example:
       ```cpp
       MyClass& operator=(const MyClass& other) {
           if (this != &other) {
               delete[] data; // Free existing memory
               // Deep copy logic here
           }
           return *this;
       }
       ```
       - To allow chaining `a1 = a2 = a3`, we cannot have return by value and pass args by reference.



- **RAII (Resource Acquisition Is Initialization)**
- RAII is a fundamental idiom in C++ used to manage resources, such as memory, file handles, sockets, and other system resources, in a way that ensures their proper release without explicitly writing cleanup code everywhere.
- RAII ties the lifetime of resources to the lifetime of the object. Dictates the following workflow:
  - Acquisition: The resource is acquired during the object's initialization (in the constructor).
  - Release: The resource is released automatically when the object's lifetime ends (in the destructor).
- The **automatic calling of the destructor as obj goes out of scope** is the essence of RAII. 
  - This mechanism ensures that resources tied to an object's lifetime are cleaned up automatically and deterministically when the object goes out of scope
  - It integrates resource mgmt with C++'s scoping mechanism seamlessly.

- **Why the Rule of Three?**
  - Prevents memory leaks.
  - Solves issues caused by shallow copying:
    - Shared ownership leads to dangling pointers or double deletion.

- **Automatic resource mgmt for stack allocated obj vars**
  - When an object goes out of scope, a "deep cleaning" executes automatically as follows:
  1. The object's destructor is called, if any.
  2. Member objects' destructors are called, if any. (facilitating a recursive deep cleaning)
  3. The memory of the object itself is deallocated (on the stack). For trivial types, this is the only step.
  Recall that the same happens for an obj when a ptr pointing to an obj is deleted. The difference is in the last step the mem storing the obj itself could be on the heap.
---

## Ownership

- **Definition**:
  - Ownership in C++ involves:
    1. Allocating resources (e.g., creating objects or allocating memory).
    2. Managing resources during their lifetime.
    3. Releasing resources when they are no longer needed.
      - Examples of Resources: dynam mem, file handles, internet sockets.
  - For example, a class owns the memory allocated for its array pointer. It is therefore responsible for resrc mgmt, i.e. implementing a destructor ensuring proper cleanup.

---

## The Rule of Five

Modern C++ (starting from **C++11**) introduces **move semantics** for more efficient resource management.  
The **Rule of Five** expands on the **Rule of Three** by adding two additional functions to manage move semantics. These functions help optimize performance by enabling **resource transfers** instead of resource duplications.

### The Big Five and Move Semantics

1. **Destructor**: Cleans up resources when an object goes out of scope.
2. **Copy Constructor**: Creates a deep copy of an object.
3. **Copy Assignment Operator**: Assigns the state of one object to another (deep copy).
4. **Move Constructor**: Transfers ownership of resources from a temporary (rvalue) object.
5. **Move Assignment Operator**: Transfers ownership during assignment from a temporary (rvalue) object.

#### Example: Move Constructor and Move Assignment Operator

Move semantics "steal" the data from an rvalue and leave the original object in a safe, empty state (e.g., `nullptr`). This avoids unnecessary allocation and copying.

```cpp
// Move Constructor
MyClass(MyClass&& other) noexcept : data(other.data) {
    other.data = nullptr;  // Transfer ownership
    std::cout << "Move constructor called\n";
}

// Move Assignment Operator
MyClass& operator=(MyClass&& other) noexcept {
    if (this != &other) {
        delete data;        // Release any old resources
        data = other.data;  // Transfer ownership
        other.data = nullptr; // Prevent double deletion
    }
    std::cout << "Move assignment operator called\n";
    return *this;
}
```

### Why Move Semantics?

There are cases where it's undesirable to keep the source after an init/assignment. Copying is wasteful in terms of spacetime complexity.
Spacewise, at some instant, two copies of the same data in the memory. Timewise, deep copying involves traversing the data struct.


---
Move semantics does not incur space usage and transfers the data in constant time with a "safe" shallow copying operation.

#### Value categories: lvalue vs rvalue

- **lvalue**:  
  An object with a **name** and a **memory address**.  
  Example:  
  ```cpp
  int x = 10;  // 'x' is an lvalue
  ```

- **rvalue**:  
  A **temporary object** or value without a name, typically appearing on the **right-hand side** of an assignment. Function return values and literals are rvalues. 
  Examples:  
  ```cpp
  x = 10;            // '10' is an rvalue
  BinaryTree tree = BSTify(tree)   // the func return is an rvalue.
  ```

  - **Lifetime of rvalues**:  
    Rvalues exist only for the duration of the **current expression** and are destroyed immediately afterward.  
    Examples:  
    ```cpp
    func(8);  // The temporary '8' will live until the function finish executing
    string str = "string";  // The RHS "string" is a temporary rvalue
    ```

#### rvalue References (`T&&`)

- WHY new syntax 'T&&' for rvalue ref and banning rvalue to bind to normal ref?
  - Disambiguation in the Big Five: If we only had T&, then the compiler wouldn;t be able to decide which to use between copy and move. 
  - Intention: Normally, passing an rvalue to a reference (`T&`) is illegal because references usually imply **persistent modification and access**. Modifying a temp obj leads to confusing semantics. In the only occasion where temps should be used,  

#### Lifetime Extension of rvalues:

- `const T&` can bind to **both lvalues and rvalues**, but is **read-only**.
- Binding rvalues to `const T&` or `T&&` (const or non-const) would **extend their lifetime to the current scope**.
- Technically one could use a T&& as a regular variable, but using it for purposes other than move semantics leads to confusing semantics.

---

### Universal/Forwarding References (`T&&` / `auto&&`)

In templated functions, `T&&` can bind to **both lvalues and rvalues**. This is known as a **forwarding reference** because it preserves the value category of the argument.

#### `T&&` Deduction Rules:

1. If the argument is an **lvalue**, `T` is deduced as a reference type (`Type&`), and `T& &&` collapses to `T&`.
2. If the argument is an **rvalue**, `T` is deduced as a non-reference type (`Type`), and `T&&` remains as `T&&`.

- These rules Does NOT apply to the deduction of plain type `T`.
- However, note that all template type deduction happens based on the function declaration/signature, not from usage inside the function body.
  - Consider
    ```cpp
    template<class T>
    void wrapper(T&& param) {
    foo(std::forward<T>(param));
    }
    ```
    The type deduction for `T` happens in the signature, thus follows the above deduction rules. That same deduced T is then used everywhere else in the function, including in `std::forward<T>`.


#### Reference Collapsing Rules:

When references combine, the following collapsing rules apply:

- `& &` → `&`
- `& &&` → `&`
- `&& &` → `&`
- `&& &&` → `&&`

This behavior ensures that the correct value category is preserved in template functions.

---

### `std::move`

`std::move` is a utility function that casts an **lvalue** to an **rvalue reference** (`T&&`), enabling move semantics.

Key points about `std::move`:

- **Does not actually move anything.**  
  It changes the **value category** of an object, signaling that it can be "moved from."
  
- **Example**:
  ```cpp
  std::string str = "Hello";
  std::string moved_str = std::move(str);  // Enables move constructor
  ```

- **Implementation**:
  ```cpp
  template <typename T>
  constexpr typename std::remove_reference<T>::type&& move(T&& t) noexcept {
    return static_cast<typename std::remove_reference<T>::type&&>(t); // equiv to std::remove_reference_t<T>&& from C++ 14
  }
  ```

  - **How the signature works**:
    - If the deduced type is a reference type (e.g., `T&` or `T&&`), `std::remove_reference<T>` removes the reference qualifiers, leaving only the base type. The ::type part accesses the resulting type after reference removal.
    - The `&&` at the end ensures the resulting type is an **rvalue reference** when an lvalue is passed (Recall the collapse rules).

---

## `std::forward<T>`

### Value Categories vs Declared Types: The Named Rvalue Reference Paradox

Assume you declared a rvalue ref: 
```cpp
Test&& x = Test(42);
Test y = Test(x);
```
One may be tempted to think the move constructor is called here as we have passed `x`, which is declared to be a rvalue ref, to the constructor. However, this is NOT the case -- in fact the copy constructor is called! 

In C++, a named rvalue reference exhibits a fundamental duality:

- Its declared type is an rvalue reference (T&&);
- Expressions using it are lvalue expressions.
  - All function arguments are expressions
  - Therefore, a named rvalue reference used as a function argument is always an lvalue expression!
  - For example, this function accepting a rvalue ref wouldn't be able to recognize the rvalue nature of the named rvalue ref `named_rref`:
    ```cpp
    void foo(Test&& w);
    Test&& named_rref = Test{};
    foo(named_rref);  // ERROR: can't bind rvalue reference to lvalue
    // Using named_rref creates an lvalue expression because:
    // - It has a name (identity)
    // - It can be used multiple times; it's not a result of a function call (unlike std::move(named_rref))
    // - You can take its address
    // These are fundamental properties of lvalues
    ```
  - `named_rref` must be moved again to be passed to `foo`!
Rvalue refs are having an existential crisis here -- what's the point of having them if functions are never aware of their rvalue nature?
#### decltype: The unique observer of declared types
`decltype` inspects the type of an expression and yields its type as the result.
Example:
```cpp
const int x = 42;
decltype(x) y = x; // y is of type const int
```

**`decltype` sees through**: 
`decltype` uniquely reveals the duality of rvalue refs through different syntax. Consider the following snippet:
```cpp
class Test {};

void demonstrate() {
    Test&& named_rref = Test{};  // type is Test&&
    
    // The variable's type is Test&&
    static_assert(std::is_rvalue_reference_v<decltype(named_rref)>, "Type is rvalue reference");
    
    // BUT the expression is an lvalue!
    static_assert(std::is_lvalue_reference_v<decltype((named_rref))>,  "Expression is lvalue");  // Note extra parentheses
}
```
Here, the extra parentheses make a huge difference:
- decltype(name) gives you the **declared type of a name/variable**
- decltype((name)) gives you the value category of an **expression**


It's the only context where there's this special distinction between a "name" and an "expression".
The extra parentheses are the switch between these two different rules.
Extra parentheses does not affect the behavior of regular functions.

### Perfect Forwarding Restores the Value Category of Named rvalue refs
Consider a templated function that accepts several `T&&` params and passes them to functions involving move semantics (i.e. functions with signature `Type&&`) within it.
- Since the params are **named** and are thus treated as lvalues, passing them directly wouldn't work as intended. 
- `std::forward<T>(arg)` is used to restore the rvalue nature of these named rvalue ref params, while preserving the lvalues as they are.

**Usage**:
```cpp
class Test {
public:
    Test() = default;
    Test(const Test&) { std::cout << "Copy!\n"; }
    Test(Test&&) { std::cout << "Move!\n"; }
};

// Having T&& alone doesn't preserve move semantics
template<typename T>
void onlyRvalueRef(T&& w) {
    Test newTest(w);  // Still calls copy constructor as w as an expression is an lvalue!
}

// Need both T&& and std::forward
template<typename T>
void withForward(T&& w) {
    Test newTest(std::forward<T>(w));  // Now calls move constructor!
}

int main() {
    onlyRvalueRef(Test{});  // Prints: "Copy!"
    withForward(Test{});    // Prints: "Move!"
}
```
**Implementation of `std::forward<T>(arg)`**

```cpp
template<typename T>
T&& forward(typename std::remove_reference_t<T>& param) {  // #1 lvalue overload
    return static_cast<T&&>(param);
}

template<typename T>
T&& forward(typename std::remove_reference_t<T>&& param) { // #2 rvalue overload
    return static_cast<T&&>(param);
}
```
- Note that the `T` for `std::forward` is not meant to be deduced within its signature, but should already be deduced and specified by its caller, which is typically a templated function with signature `T&&`.

**How this works**:
As the caller receives an lvalue/rvalue, `T` will be deduced as `Type&`/`Type` according to the deduction rules of `T&&`. 
The ref is removed by `std::remove_reference`, and the two overloads become
```cpp
forward(Type& param)  {static_cast<Type&>(param)}   // Note the use of reference collapsing rules
forward(Type&& param)  {static_cast<Type&&>(param)}
```
Now, only the first/second overload would be compatible with an lvalue/rvalue. 
(When encountering lvalue/rvalue, decide to always read it as lvalue or always as rvalue, but not both or alternating.)


## Move semantics practices:
One could:
- Use on normal objs as an efficient alternative to copy. 
  - Note: Requires casting to rvalue with std::move() and clears the src obj.
- Use on init/assign of objs by func returns or literals.
---
## `constexpr`

A **`constexpr`** function or variable can be evaluated at **compile time** if all arguments are constant.

### Why Use `constexpr`?

1. **Compile-Time Optimization**: Reduces runtime overhead by performing operations at compile time.
2. **Safer Code**: Ensures correctness by catching errors at compile time.

### Limits of `constexpr`:

- A `constexpr` function must have a body that can be fully evaluated at compile time when invoked with constant arguments.
- If used with non-constant arguments, it will execute at runtime.

---
# Smart Pointers

**Smart ptrs are essentially RAII wrapper classes for raw ptrs.** They eliminate manual memory management errors like leaks and double deletions by automating resource cleanup and enforcing ownership semantics.

- Key functionality: by wrapping raw ptrs in a class with a destructor, we ensure that when a smart ptr goes out of scope, the resrc is auto deallocd.
---
## Unique Pointers


**`std::unique_ptr` is a lightweight, non-copyable smart pointer w/ unique ownership.**

The most straightforward smart ptrs is simply a ptr wrapper class with a destructor. Lacking a mechanism to
track "mutual owners" holding a resrc, they must ensure exclusive ownership to avoid double deletion. 
Such a smart ptr is `std::unique_ptr`. 
- A resrc is owned by a single `std::unique_ptr` at any given time. This is ensured by move-only semantics, i.e. `= delete`-ing copy constructor & assignment op.
  - There's no protection against creating multiple `std::unique_ptr`s with the same raw ptr!
---
### Custom Deleters
There are scenarios where the resrcs managed by a smart ptr is not heap memory, e.g. file handles, sockets, CUDAmem, to which `delete` doesn't apply. Smart ptrs must therefore be equipped with resrc-specific deleters.
- The custom deleter is part of the `std::unique_ptr`'s type and is kept as a class attribute.
- `std::unique_ptr`s with different deleter type cannot be moved from each other.
- Don't pass `std::unique_ptr`s to callables by value. Doing so will invoke the move contructor and transfer the ownership to the copy inside the callable. Use `const std::unique_ptr&` for the callable to access the obj.

**Implementation**:

`unique_ptr` class supporting custom deleters: 
```cpp
template <typename T, typename Deleter = sts::default_delete<T>> // default: delete
class unique_ptr  {
private:
  T* ptr;
  Deleter deleter;
public:
  explicit unique_ptr(T* p) noexcept : ptr(p)  {}
  unique_ptr(T* p, Deleter d) noexcept : ptr(p), deleter(std::move(d))  {}
  void reset(T* p = nullptr) noexcept {
    if (ptr)  deleter(ptr);
    ptr = p;
  }
  ~unique_ptr() {reset();}
  T* get() noexcept {
    return ptr;
  }
  T* release() noexcept {
    T* temp = ptr;
    ptr = nullptr;
    return temp;
  }
  // ...
}
```
Managing CUDAmem with such a `unique_ptr`:
```cpp
#include <memory>
#include <cuda_runtime.h>

// Imple custom deleter w/ functor (func ptr, lambda, std::function would all do)
template <typename T>
struct CustomDeleter  {
  void operator()(T* ptr) {cudaFree(ptr);}
}



// Init a unique_ptr w/ custom deleter
int main()  {
  float* devicemem;
  cudaMalloc(&devicemem, 1024*sizeof(float));
  unique_ptr<float, CustomDeletor> ptr1(devicemem) // CustomDeletor can be implicitly instantiated

  // w/ lambda:
  auto del = [](float* ptr){cudaFree(ptr);};
  unique_ptr<float, decltype(del)> ptr2(ptr1.release(), del);
  return 0;
}
```
---
### Constructor Wrapper: `make_unique`
`make_unique` is an alternative to explicit calls to the constructor. 

**Implementation**:
```cpp
template<typename T, typename... Args>
make_unique(Args&&... args) {
return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}
```
**Usage**:
```cpp
int* raw = new int(5);
// Alt 1: explicitly calling the constructor:
auto ptr1 = std::unique_ptr<int>(raw);
// Alt 2: using make_unique:
auto ptr1 = std::make_unique<int>(5);
```

**Disadvantage**: Does not support initializing w/ custom deleter.


**Benefits**:
- Less redundant typing:
```cpp
// Explicit way - type repeated
std::unique_ptr<MyClass> ptr(new MyClass());

// make_unique way - cleaner, could leverage auto
auto ptr = std::make_unique<MyClass>();
```

- Exception safety - make_unique prevents memory leaks in case of exceptions. Consider:

```cpp
func(std::unique_ptr<MyClass>(args), expr_that_may_throw)
// First entry: a safe shorthand for std::unique_ptr<MyClass>(new MyClass(args)) after C++ 14.
// Second entry: a callable (func ptr, lambda, functor, std::function) 
```
The expression is not atomic: it is composed of 
1. Call to MyClass constructor
2. Call to `unique_ptr` constructor
3. `expr_that_may_throw`

The compiler may take its freedom to optimize and decide to execute these components in non-standard order (though less so after C++ 17). If the arg evaluation happened in the following order:
1. Call to MyClass constructor
2. `expr_that_may_throw` throws!
3. Call to `unique_ptr` constructor

If step 2 throws, then memory allocd in step 1 will be leaked as the `unique_ptr` is never created to take ownership.


**`make_unique` wraps step 1 and 3 into a single expression, enforcing the evaluation order and thus providing exception safety**. This is a key reason why we have various "make" functions in the STL. 

They all serve dual purposes:
- Forcing eval order
- Providing cleaner syntax
---
## Shared Pointers

**`std::shared_ptr` is a heavier, reference-counted smart pointer allowing shared ownership.**

**Ownership Count**:
As discussed, a straightforward ptr wrapper must ensure unique ownership. To enable shared ownership while keeping proper mem mgmt (namely, no double nor premature deletion), one must maintain an additional metadata: ownership count, with behavior:
- initializes to 1 upon the creation of a `std::shared_ptr`
- increments when a `std::shared_ptr` managing the resrc in question is copy init/assigned to another `std::shared_ptr`
- decrements when a `std::shared_ptr` owning the resrc is reassigned or goes out of scope
- when `== 0`, free the resrc


**Control Block**: The ownership count, along with the raw ptr to the resrc being managed and the optional custom deleter, are stored in a **control block**. 
- A control block is shared by a group of `std::shared_ptr`s created by copy init/assign from the same `std::shared_ptr` or its copy.
- Unlike `std::unique_ptr`, `std::shared_ptr`s access the managed obj through the control block, adding an additional layer of abstraction.
- Deleter is NOT part of a `std::shared_ptr`'s type. Including deleter type in declaration as in `std::unique_ptr` results in a compilation error.
- `std::shared_ptr`s with different deleters could be copied/moved from each other. They essentially belongs to different control blocks. No control block overwriting or mixing shall happen.
- `std::shared_ptr`s created by different `make_shared` or explicit constructor calls will have independent control blocks
  - even if they manage the same resrc (leads to undefined behavior when one control block is destroyed!)


Assigning `shared_ptrs` with different deleters example:
```cpp
#include <memory>
#include <iostream>
int main()  {
struct Deleter1 { 
    void operator()(int* p) { 
        std::cout << "Deleter1\n"; 
        delete p; 
    }
};
struct Deleter2 { 
    void operator()(int* p) { 
        std::cout << "Deleter2\n"; 
        delete p; 
    }
};

std::shared_ptr<int> p1(new int, Deleter1{});
{
    std::shared_ptr<int> p2(new int, Deleter2{});
    p1 = p2;  // p1's original object and control block is deleted with Deleter1, by copy assignment implementation
              // p1 now shares p2's control block with Deleter2
}
// Nothing happens here - p2 out of scope, p1 still owns the object

// Finally Deleter2 runs
return 0;
}
```

**Aliasing `std::shared_ptr`**: 
An **aliasing `std::shared_ptr`** is one that shares the control block with a group of `std::shared_ptr`s while pointing to a different resrc, normally a subobject.

Syntax:
```cpp
#include <memory>
struct Obj  {
  int x = 10;
  bool y = false;
}
int main()  {
std::shared_ptr<Obj> sp = std::make_shared<Obj>();
std::shared_ptr<int> aliasing_sp(sp, &(sp->x)); 
std::cout << "Aliased value: " << *aliasing_sp << "\n"; // Outputs 10
std::cout << "Use count: " << sp.use_count() << "\n";   // Outputs 2
return 0;
}
```
---
### Circular Reference Problem

Circular References are a logical issue of `std:shared_ptr` that leads to memory leak. 
- Circular reference are not typical memory leaks, which refers to cases where an object becomes unreachable, i.e. no pointers or refs point to it. Thus the AddressSanitizer (ASan) will not notify the user of the mem leak cause by circular refs.


The following code demonstrates a **circular reference problem** with `std::shared_ptr` that leads to a memory leak.
```cpp
#include <iostream>
#include <memory>

struct  B {}; // Forward declaration

struct  A {
  std::shared_ptr<B> b_ptr;
  ~A()  {std::cout << "A destroyed\n"}
}

struct B  {
  std::shared_ptr<A> a_ptr;
  ~B()  {std::cout << "B destroyed\n"}
}

int main()  {

  {
    std::shared_ptr<A> a = std::make_shared<A>();
    // objA created, owned by std::shared_ptr a.
    // objA ownership count: 1 (a)
    std::shared_ptr<B> b = std::make_shared<B>();
    // objB created, owned by std::shared_ptr b
    // objB ownership count: 1 (b)

    a->b_ptr = b;
    // objB ownership count: 2 (b, objA.b_ptr)
    b->a_ptr = a;
    // objA ownership count: 2 (a, objB.a_ptr)
  }

  // a, b goes out of scope, but objA, objB is never destroyed!
  // This is because objA, objB still holds each other -- ownership count never reaches 0
  // Memory leaks!
  return 0;
}
```
---
## Weak Pointers

A `std::weak_ptr` is a smart ptr that holds a non-owning reference to an obj managed by `std::shared_ptr`s.
- Weak ptrs can only be created by implicitly converting a shared ptr;
- Weak ptrs cannot be directly dereferenced. To access the object, one must first call `lock()` on the weak ptr:
```cpp
// To create a weak ptr:
std::weak_ptr<int> wp = sp;
// To access the obj:
std::shared_ptr locked_sp = wp.lock();
// lock(): obj destroyed ? valid shared_ptr : nullptr
if  (locked_sp) {
  Do_Something(*locked_sp);
}
```
- Weak ptrs doesn't contribute to ownership count.
  - Object WILL be destroyed as shared ptr count goes to zero, even if there are still weak ptrs present.
- Weak ptrs also access the managed obj through the control block.
- However, the control block maintains a separate weak count.
- Control block is NOT destroyed until weak count is zero, even if shared count is zero.
  - Destroying the control block regardless of weak count would render weak ptrs dangling and unable to provide proper null check.
---
### Use cases:

#### Weak ptrs break circular reference

Replacing a shared ptr in a circular reference loop by a weak ptr prevents the memory leak. 
The object pointed to by the weak ptr would not have "support".


Note that weak ptrs doesn't have its own circular reference problem even though the control block is kept alive until the weak count is zero. Control blocks doesn't hold weak ptrs, after all.

#### Weak ptrs as Observer

Weak ptrs can be useful in cases where there is a potential task to be executed, but the task is not supposed to affect the lifetime of the object, and we want to ensure safe behavior in cases where the object is destroyed. 

- If one passes a `std::shared_ptr` to a persisting callable by value, it may unintentionally alter the lifetime of the resrc; 
- or if a reference is passed and there is no extra mechanism to cancel the callback when no shared ptrs are alive, the reference would lead to undefined behavior.

By letting the callback capture weak ptrs:

- Lifetime of obj wouldn't be affected;
- Safe null check through `lock()`.

Following example illustrates such a use case.
```cpp
void Element::reloadAsync(const std::vector<models::pixeldraw::Data>& pds) {
  auto* looper = Loopers::getInstance(Loopers::Name::kGPU);
  if (looper == nullptr) {
      return;
  }

  // Create a shared pointer to a LoadingCtx object
  auto loading_ctx = std::make_shared<LoadingCtx>();

  for (const auto& pd : pds) {
    loading_ctx->data.push(pd);
  }

  // Define the task using a lambda function
  loading_ctx->task = std::function<void()>([looper, this, weak_ctx = std::weak_ptr<LoadingCtx>(loading_ctx)]() {
    auto ctx = weak_ctx.lock();
    if (!ctx) {
      return;
    }

    if (!ctx->data.empty()) {
      vibe::util::ScopedTimedLogger time_load("Load one path");
      draw(&(ctx->painter), ctx->data.front());
      ctx->data.pop();

      // Post the task again for later execution
      looper->postLater(ctx->task);
    } else {
      loading_ctx.reset();  // Reset the shared pointer when done
    }
  });

  // Post the task for later execution
  looper->postLater(loading_ctx->task);
}
```
---


# C++ Miscellaneous


## `this` Pointer

- `this` is a pointer pointing to the object of the current context.
- It is used in class definitions and behaves similarly to `self` in Python.
- Access object attributes using either:
  ```cpp
  this->attr; // Explicit access to attributes
  this->func; // Explicit calling member functions
  attr;       // Implicit access, also OK but less clear in some contexts
  ```

---

## Accessing Object Attributes Through Object Pointers

In C++, accessing an object's attributes through a pointer can be done using the `->` operator, which is equivalent to dereferencing the pointer and then accessing the attribute using the `.` operator.

### Example:
```cpp
ptr->value  // Equivalent to
(*ptr).value
```

The `->` operator is simply syntactic sugar for the more verbose `(*ptr).value`. Both achieve the same result.

---

## Pre-Increment (`++it`) vs. Post-Increment (`it++`)

### Pre-Increment (`++it`)

- **Behavior**: Increments the pointer/iterator and returns the incremented value.
- **Function Signature**: Returns by reference (`Iterator& operator++()`).
- **Implementation**:
  ```cpp
  Iterator& operator++() {
      ++(this->ptr);   // Increment the pointer
      return *this;    // Return the incremented iterator by reference
  }
  ```
- **Efficiency**: 
  - Does not create a copy, avoiding unnecessary overhead.
  - Returning by reference ensures no additional object is created.

---

### Post-Increment (`it++`)

- **Behavior**: Returns the current value of the pointer/iterator before incrementing it.
- **Function Signature**: Returns by value (`Iterator operator++(int)`).
- **Implementation**:
  ```cpp
  Iterator operator++(int) {
      Iterator temp = *this;  // Store the current state
      ++(this->ptr);          // Increment the pointer
      return temp;            // Return the copy of the original state
  }
  ```
- **Efficiency**:
  - Since it returns the original value, a copy is created (return by value).
  - This makes post-increment slightly less efficient than pre-increment, especially with heavier objects.

---

### Built-in Pointer Behavior

For any pointer type (e.g., `T* ptr`), C++ provides a built-in `++` operator. This operator:
1. **Pre-Increment**: Moves the pointer to the next memory location based on the type size.
   ```cpp
   ++ptr;  // Advances ptr by sizeof(T)
   ```
2. **Post-Increment**: Returns the current location of the pointer before moving it to the next memory location.
   ```cpp
   ptr++;  // Advances ptr but returns the original value
   ```

---


## Type Conversion in Inheritance Relationships

Derived class objs/ptrs could be implcitly converted to based class objs/ptrs, as the former is guaranteed to have all the members of the base class. 


The reverse requires an explicit downcast with either:
- `static_cast<Derived*>()`. No checking if the ptr actually points to a derived class obj.
- `dynamic_cast<Derived*>()`. Checks if obj pointed to is actually a derived class obj, if not, return `nullptr`.


## Constructor Implcit Conversion and `explicit` Keyword:
```cpp
#include<memory>
class other {};

class A {
public:
  A() {};
  A(int b)  {};
  explicit A(other c) {}; // Constructor marked as "explicit"
};

int main()  {

other src;

A obj1(42); // Legal: explicit call to constructor

A obj2 = 42; // Legal: implicit conversion through single-param constructor

A obj3 = src; // Error! "explicit" prevents implicit conversion through constructor

std::unique_ptr<A> up = new A();     // Error! smart ptr constructors are "explicit"

return 0;
}
```

Smart ptr constructors are marked `explicit` for more safety against accidental transfers of ownership.

## On the Use of `const`

### Example Function Declaration

```cpp
const T1& func(const T2& arg) const {
    return var;
}
```

This declaration involves **three usages of `const`**, each serving a specific purpose:

---

### 1. First `const` (Return Value)

The `const T1&` ensures that the caller cannot modify the returned object via the reference. E.g.
```cpp
const int& getValue() const {
    return value;
}

obj.getValue() = 42;  // Compiler error: 'const int&' cannot be modified
```
This immutability contract cannot be bypassed by simply assigning to a non const ref: 
#### Example:
```cpp
const int& getValue() {
    static int value = 42;
    return value;
}

int main() {
    int& ref = getValue();  // ERROR: Cannot bind 'int&' to 'const int&'
    ref = 50;               // This line will never compile
}
```
It can only be bypassed by explicitly using `const_cast`, which is discouraged.


---

### 2. Second `const` (Function Argument)

The `const T& arg` ensures that the function does not modify the argument. 

#### Example:
```cpp
T obj;
func(obj);  // The function cannot modify obj
```

- **Best Practices**:
  - Use `const T&` for arguments that do not need to be modified. 
  - For arguments that require modification, use `T&` (non-const reference).
  - For operations that require a copy (e.g., traversing a tree), create a temporary copy:
    ```cpp
    T temp = arg;
    ```

---

### 3. Third `const` (Member Function)

The `const` at the end of the function declaration ensures that the function does not modify the object it is called on.

#### Example:
```cpp
class MyClass {
    int var;
public:
    int getVar() const {
        return var;  // This function guarantees it won't modify the object
    }
};
```

- **Common Use Cases**:
  - This is frequently used for **getter functions** or other operations that do not alter the state of the object.

---

### Summary of `const` Usages

| **Type of `const`**     | **Purpose**                                                                                      |
|--------------------------|------------------------------------------------------------------------------------------------|
| **First `const` (return)** | Prevents the caller from modifying the returned value via the reference.|
| **Second `const` (arg)**  | Prevents the function from modifying the argument.                                             |
| **Third `const` (method)**| Prevents the function from modifying the object it is called on.                               |

**Note**: The const keyword before the return type does not contribute to function overloading or differentiation. Two functions with or without const at the front of the return type are considered the same function by the C++ compiler.

---
### Benefits of `const`

By using const appropriately, developers not only 
- improve code readability and maintainability 
- but also allow the compiler to produce highly optimized executables, such as:
  - loop unrolling -- no more branching and iterator increment during runtime.
  - inlining, due to their prefictability and simplicity.
  - Using read-only memory -- generally faster and protects against accidental modification.

  ---
## std::variant

### Union

A union in C and C++ is a special data structure that allows multiple variables (of different types) to share the same memory location.
- A struct, unlike a union, allocates separate memory for each member.
- A union only allows one active member at a time, with its size being that of the largest member.
- Unions are useful in cases where you need to store multiple types of data but do not need to store them all at once. 
- Unions do not store any information about the active type nor any about the avaliable types. No overhead beyond the stored variable.
Syntax:

```cpp
#include <iostream>
using namespace std;

union Data {
    int i;
    float f;
    char c;
};

int main() {
    Data data;

    // Assign an integer value
    data.i = 42;
    cout << "data.i: " << data.i << endl;

    // Assign a float value (overwrites previous value)
    data.f = 3.14;
    cout << "data.f: " << data.f << endl;

    // Assign a char value (overwrites previous value)
    data.c = 'A';
    cout << "data.c: " << data.c << endl;

    return 0;
}
```
**Destructor not auto-invoked**: Upon reassignment or going out of scope, current union member's destructor is not automatically called, and explicit invocation of the member destructor is needed for safe behavior. Recall that unions do not track their active member. Consider a union with nontrivial members:

```cpp
union nontrivial {
    Obj obj;    // Non-trivial
    int value;  // Trivial
};

nontrivial u;
new (&u.obj) Obj(); // Placement new to construct obj
u.obj.~Obj();       // Manually destroy obj or mem leak
u.value = 42;       // Assign to value
```

In cases of handling unions with nontrivial types, `std::variant` would be a better approach.

### std::variant

`std::variant` is a type-safe union.
- Tracks avaliable types. 
  - Attempting to access or assign a type that is not listed in the variant's type signature will result in a compilation error.
- Tracks active type. 
  - A runtime index documents the active type.
  - The runtime index determines the behavior of `std::get_if`, `std::visit`, and `std::holds_alternative`.

#### Syntax:

```cpp
#include <iostream>
#include <variant>
#include <string>

int main() {
    std::variant<int, std::string> v = 42;

    if(std::holds_alternative<int>(v))  { //returns a bool
      std::cout << "The variant holds an int with value: " << std::get<int>(v) << std::endl;
    }

    // Attempting to access a type not in the variant
    // std::get<double>(v) or v = 3.14; // Compilation error: `double` is not in the variant

    /*
    std::get_if<T>(std::variant*)
    Input: A pointer to a std::variant.
    Output: Returns a pointer to the stored value if the std::variant holds the specified type; otherwise, returns nullptr.
    */

    if (auto intPtr = std::get_if<int>(&v)) { // Returns pointer to int
        std::cout << "Using std::get_if: " << *intPtr << "\n";
    } else {
        std::cout << "std::get_if failed: type mismatch\n";
    }


    /*
    std::visit(Callable&& visitor, Variant&&... variants);
    Input:
      Callable: A function, function object, or lambda that will be invoked for the active value(s).
      Variant: One or more std::variant objects whose active values will be passed to the callable. If multiple, passed in that order.
    */

    // Example usage of std::visit with templates:

    template <typename Visitor, typename... Variants>
    void visit_multiple(Visitor&& visitor, Variants&&... variants) {
        // Expanding all variants into a tuple and applying the visitor
        (std::visit(std::forward<Visitor>(visitor), std::forward<Variants>(variants)), ...);
    }
    std::variant<int, std::string> v1 = 42;
    std::variant<float, double> v2 = 3.14;
    
    // Define a visitor (lambda)
    auto visitor = [](auto&& value) {
        std::cout << "Active value: " << value << "\n";
    };
    
    // Apply the visitor to multiple variants
    visit_multiple(visitor, v1, v2);
    return 0;
}
```
  ---
