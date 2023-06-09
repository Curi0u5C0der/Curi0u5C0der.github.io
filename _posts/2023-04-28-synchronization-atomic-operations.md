---
layout: default
title:  Synchronization in Linux - Atomic Operations
---

# Synchronization in Linux - Atomic Operations

Linux kernel is a multi-threaded and multi-user operating system that allows multiple processes and threads to run concurrently. Linux kernel provides multiple synchronization mechanisms to ensure that these processes do not interfere with each other and maintain consistency thereby preventing race conditions. Here is a detailed explanation of some of the synchronization mechanisms used in the Linux kernel.

- Atomic Operations
- Barriers
- Mutexes
- Spinlocks
- Reader / Writer Locks

Let's look at each synchronization mechanisms in detail:

## Atomic Operations

**Atomic Operations**: Atomic operations are used to manipulate shared data structures in a way that guarantees consistency without the need for locks. Atomic operations are operations that cannot be interrupted or interleaved by other processes. In Linux kernel, atomic operations are implemented using special CPU instructions that ensure the operation is executed atomically.

Let's look at an example on how to prevent a race condition using atomic operations:

```cpp
#include <iostream>
#include <thread>
#include <atomic>

int counter;

void incrementCounter() {
    for (int i = 0; i < 1000000; i++) {
        counter++;
    }
}

int main() {
    std::thread t1(incrementCounter);
    std::thread t2(incrementCounter);

    t1.join();
    t2.join();

    std::cout << "Counter value: " << counter << std::endl;

    return 0;
}
```

In the above example we have a global data `counter` that is shared between two threads `(t1 and t2)`,  The `incrementCounter` function is called by both threads, and increments the counter variable 1,000,000 times using the counter `++` operation,  since the `counter` is shared and both the threads (t1, and t2) tries to access the data at the same time, there could be a race condition where both threads try to increment the counter at the same time, leading to unpredictable results.

So when we run the above program the result is unpredictable,

```bash
❯ g++ t.cpp -lpthread -o a.out
❯ ./a.out     

❯ Run 1:
❯ Counter value: 117284

❯ Run 2:
❯ Counter value: 1048799

❯ Run 3:
❯ Counter value: 1048799
```

However, if we declare `counter` as an atomic varaible, with this we can performance atomic operations theryby avoiding race conditions, atomic operation is a single indivisible operation without interleaving of operations across threads.


```cpp
#include <iostream>
#include <thread>
#include <atomic>

std::atomic<int> counter(0);

void incrementCounter() {
    for (int i = 0; i < 1000000; i++) {
        counter++;
    }
}

int main() {
    std::thread t1(incrementCounter);
    std::thread t2(incrementCounter);

    t1.join();
    t2.join();

    std::cout << "Counter value: " << counter << std::endl;

    return 0;
}
```

By using the `std::atomic` type for the counter variable, and the `counter++` operation, we ensure that each increment operation is executed atomically, preventing race conditions.

When we run the program, we can see that the counter value is always the expected value (2,000,000), indicating that the atomic operation prevented any race conditions:

```bash
❯ g++ t.cpp -lpthread -o a.out
❯ ./a.out     

❯ Run 1:
❯ Counter value: 2000000

❯ Run 2:
❯ Counter value: 2000000

❯ Run 3:
❯ Counter value: 2000000
```

When we create an `std::atomic<int> variable` like counter with an initial value of 0, the compiler generates machine code to ensure that any operation on the variable is executed atomically, without interference from other threads.

To see the machine code generated by the compiler, we can use the objdump tool. Let us first analyse the code without atomic variable:

We can compile and examine this code with objdump again:

```bash
$ g++ -S -std=c++11 -O2 -o thread.s thread.cpp
$ objdump -d thread.s
```
Here is the disassembly for the `incrementCounter` function:

```bash
    _Z16incrementCounterv:
    .LFB2902:
            .cfi_startproc
            pushq   %rbp
            .cfi_def_cfa_offset 16
            .cfi_offset 6, -16
            movq    %rsp, %rbp
            .cfi_def_cfa_register 6
            movl    $0, -4(%rbp)
    .L11:
            cmpl    $999999, -4(%rbp)
            jg      .L12
            movl    counter(%rip), %eax
            addl    $1, %eax             // Here the increment operation is performed using addl,
            movl    %eax, counter(%rip)
            addl    $1, -4(%rbp)           
            jmp     .L11
```

Here the increment operation to the counter value is not atomic, as there is a tiny window between the processor reading the old value of _x and writing the new value back; if another CPU writes to _x at that exact moment, that value will be overwritten.

The reason why is that:

Because the whole increment is not just done with `addl $9,_x(%rip)` but as three operations, 

- (1) FETCH_MEM_TO_MREG counter
- (2) ADD_TO_MREG 1
- (3) STORE_MREG_TO_MEM counter


```bash
    movl    counter(%rip), %eax
    addl    $1, %eax             // Here the increment operation is performed using addl,
    movl    %eax, counter(%rip)
```

When two CPUs are simultaneously accessing the shared data `counter`,

Here's a valid sequence of events,

```bash
    # this is a valid sequence
    # cpu 1                         # cpu 2
    movl     counter(%rip), %eax
    addl     $1, %eax
    movl     %eax, counter(%rip)
                                    movl     counter(%rip), %eax
                                    addl     $1, %eax
                                    movl     %eax, counter(%rip)
```

But, we could also get the below invalid sequence,

```bash
    # this is an invalid sequence
    # cpu 1                          # cpu 2
    movl     counter(%rip), %eax
                                        movl     counter(%rip),%eax
    addl     $1, %eax                   addl     $1,%eax
    movl     %eax, counter(%rip)
                                        movl     %eax,counter(%rip)
```
This will cause the results to be inconsistent if multiple CPUs are modifying this shared `counter`, as the instructions (fetch, add, store) can be interleaved.


To avoid this race condition, `std::atomic<int>` type preserves atomicity by generating machine code that uses the lock prefix on atomic operations, which ensures that the operations are executed atomically, without interference from other threads.

Now let us look at the disassembly for the code with the atomic varaible declaration,

Here is the disassembly for the `incrementCounter` function:

```bash
    222 
    223 00000000000012c9 <_Z16incrementCounterv>:                
    224     12c9:       f3 0f 1e fa             endbr64
    225     12cd:       55                      push   %rbp
    226     12ce:       48 89 e5                mov    %rsp,%rbp
    227     12d1:       48 83 ec 10             sub    $0x10,%rsp
    228     12d5:       c7 45 fc 00 00 00 00    movl   $0x0,-0x4(%rbp)
    229     12dc:       eb 18                   jmp    12f6 <_Z16incrementCounterv+0x2d>
    230     12de:       be 00 00 00 00          mov    $0x0,%esi
    231     12e3:       48 8d 05 6a 2e 00 00    lea    0x2e6a(%rip),%rax
    232     12ea:       48 89 c7                mov    %rax,%rdi
    233     12ed:       e8 68 02 00 00          call   155a <_ZNSt13__atomic_baseIiEppEi>
    234     12f2:       83 45 fc 01             addl   $0x1,-0x4(%rbp)
    235     12f6:       81 7d fc 3f 42 0f 00    cmpl   $0xf423f,-0x4(%rbp)
    236     12fd:       7e df                   jle    12de <_Z16incrementCounterv+0x15>
    237     12ff:       90                      nop
    238     1300:       90                      nop
    239     1301:       c9                      leave
    240     1302:       c3                      ret
    .
    .
    .
    .
    .
    .
    432 000000000000155a <_ZNSt13__atomic_baseIiEppEi>:
    433     155a:       f3 0f 1e fa             endbr64 
    434     155e:       55                      push   %rbp
    435     155f:       48 89 e5                mov    %rsp,%rbp
    436     1562:       48 89 7d e8             mov    %rdi,-0x18(%rbp)
    437     1566:       89 75 e4                mov    %esi,-0x1c(%rbp)
    438     1569:       48 8b 45 e8             mov    -0x18(%rbp),%rax
    439     156d:       48 89 45 f8             mov    %rax,-0x8(%rbp)
    440     1571:       c7 45 f0 01 00 00 00    movl   $0x1,-0x10(%rbp)
    441     1578:       c7 45 f4 05 00 00 00    movl   $0x5,-0xc(%rbp)
    442     157f:       8b 55 f0                mov    -0x10(%rbp),%edx
    443     1582:       48 8b 45 f8             mov    -0x8(%rbp),%rax
    444     1586:       f0 0f c1 10             lock xadd %edx,(%rax)  #atomic operation
    445     158a:       89 d0                   mov    %edx,%eax
    446     158c:       5d                      pop    %rbp
    447     158d:       c3                      ret
```

Here `lock` instruction is used to preserve the atomicity, When a CPU sees the lock prefix, it will lock the bus and ensure that no other CPU can access the memory location being modified until the instruction completes. This ensures that the `counter++` operation is executed atomically, without interference from other threads.

Overall, the `std::atomic<int>` type preserves atomicity by generating machine code that uses the lock prefix on atomic operations, which ensures that the operations are executed atomically, without interference from other threads.