# Forking with `pcntl_fork()` in PHP

Most of the applications we build aren’t `NASA-level` systems where every millisecond counts. They're often simpler — but with the right design and a good understanding of core concepts, we can still make them fast and efficient.

Sometimes, we just need to:

- Speed things up (e.g., parallel API calls or data processing)
- Avoid blocking the main thread (e.g., slow DB queries or report generation)
- Keep things simple (no overkill like Redis queues or full worker systems)

In such cases, `pcntl_fork()` might be a useful tool — **if used wisely.**

---

## What is `pcntl_fork()`?

`pcntl_fork()` is a PHP function (available on Unix-based systems) that creates a child process by duplicating the current process. It’s part of the Process Control extension (pcntl), and is commonly used for multitasking in CLI applications.

## Can I Control When Child Behavior?

- Each child process behaves according to the code it executes.

- You control what each child does by writing its code block appropriately. Always make sure it exits cleanly to avoid leaving zombie processes.


## When Do Child Processes Start Executing?

- **When pcntl_fork() succeeds, children run as soon as they’re forked.**

    - The child process becomes independent and starts executing right away (concurrently with the parent and siblings).

    - The OS scheduler determines when exactly it gets CPU time (usually near-instant).

## How Control Child Processes Start Executing?

| Number     | Method     | Control Level | Easy to Use | Suitable For |
| ----- | ---------------------------- | ------------- | ----------- | ------------ |
| 1     | Delay in Child (`sleep`)     | Low           |  Easy      | Demos/tests  |
| 2     | Parent Signal (`posix_kill`) | High       |  Medium   | 1-to-1 sync  |
| 3     | Semaphores (`sem_get`)       | Very High |  Complex   | Multi-child  |

- I personally picked method 3
    - Best for multiple processes
    - Supports complex coordination
    - requires --enable-sysvsem
    - to check run in your terminal -m | grep sysvsem


## What Are Its Hard Limitations?

- **Not available on Windows**  
  Works only on Unix-based systems (Mac and Linux).

- **Can be dangerous**  
  Improper use can lead to **zombie processes**, memory leaks, or race conditions — always make sure to **terminate child processes properly**.

- **Limited to CLI scripts**  
  Do **not** use it to build web servers or run it inside HTTP request handlers.

- **Requires careful handling**  
  Each child receives a **copy-on-write** version of the parent’s memory. To share data between processes, you’ll need tools like **sockets**, **shared memory**, or **temporary files**.

### When to consider it:

- You need to perform tasks in **parallel**
- You want to **avoid blocking** while waiting (e.g., on APIs or slow I/O)
- You don’t want to set up **external queues or workers**


---

## Final Thoughts


- It also powers the Pest testing CLI framework.

- It won’t solve every problem, but in the right hands, it can help you speed up tasks without requiring major architectural changes.

---

