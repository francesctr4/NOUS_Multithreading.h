# NOUS_Multithreading.h

---

### Thread Pool-based Job System Implementation

Single-header, cross-platform C++11 library for managing concurrent task execution.

![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)
![C++11](https://img.shields.io/badge/C%2B%2B-11%2B-blue)

---

### Enhancing Software Performance through Multithreading and Parallel Programming Techniques

This repository contains source code developed as part of my **Bachelor's Thesis (TFG)**:

- **Nous Engine Repository:** [https://github.com/francesctr4/Nous-Engine](https://github.com/francesctr4/Nous-Engine)

- **Multithreading Library:** [https://github.com/francesctr4/NOUS_Multithreading.h](https://github.com/francesctr4/NOUS_Multithreading.h)

**Author:** Francesc Teruel Rodriguez ([francesctr4](https://github.com/francesctr4) on GitHub)  
**Created:** 30/06/2025  
**Version:** 1.0  

**License:** MIT License  
© 2025 Francesc Teruel Rodriguez

---

### **Contributing**
Pull requests are welcome! If you'd like to suggest improvements, add features, or report issues, feel free to open a GitHub issue or PR.

---

## 📚 Table of Contents

- [Home](#nous_multithreadingh)
- [Dependencies](#dependencies)
- [Setup](#setup)
- [Usage](#usage)
- [Quick Start](#quick-start)
- [Documentation](#documentation)
  - [`NOUS_Job`](#-nous_job)
  - [`NOUS_Thread`](#-nous_thread)
  - [`NOUS_ThreadPool`](#-nous_threadpool)
  - [`NOUS_JobSystem`](#-nous_jobsystem)
  - [Global Functions](#-global-functions)

---

### **Dependencies**  

- ISO C++11 Standard (compile with `/std:c++11` or `-std=c++11`) or newer

---

### **Setup**

1. Download or copy the file ```NOUS_Multithreading.h``` into your project directory and add it to the project.
   
2. Include the header in your source files: ```#include "NOUS_Multithreading.h"```.
   
3. Refer to the usage examples and API documentation provided to start using the job system into your application.

---

### **Usage**  

#### Register the Main Thread

1. Begin by calling RegisterMainThread() to retrieve and store information about the main thread.
   
#### Initialize the Job System

2. Create and configure an instance of the NOUS_JobSystem, typically as a pointer. If no thread count is specified during initialization, the system will automatically use the maximum number of hardware threads minus one, reserving one for the main thread.
   
#### Submit Jobs

3. Once the job system is initialized, jobs can be submitted using the SubmitJob() function. Submitted jobs will be automatically queued, managed, and executed by the available worker threads.

#### Wait for Pending Jobs

4. Before shutting down the system, it is recommended to explicitly call WaitForPendingJobs() to ensure all jobs have completed. While this step is already handled in the job system’s destructor, doing it manually is considered good practice for clarity and control.

#### Shutdown and Cleanup

5. After all jobs have been processed, destroy the job system instance and call UnregisterMainThread() to clean up the main thread information.

---

### **Quick start**  
```
#include "NOUS_Multithreading.h"

int main(int argc, char** argv)
{
   NOUS_Multithreading::RegisterMainThread();
   
   // Create the job system dynamically (using hardware concurrency)
   NOUS_Multithreading::NOUS_JobSystem* jobSystem = new NOUS_Multithreading::NOUS_JobSystem();
   
   // Submit jobs
   jobSystem->SubmitJob([]() { /* task 1 */ }, "Task1");
   jobSystem->SubmitJob([]() { /* task 2 */ }, "Task2");
   
   // Wait for completion
   jobSystem->WaitForPendingJobs();

   // Optional debug output
   NOUS_Multithreading::JobSystemDebugInfo(*jobSystem);
   
   // Clean up when done
   delete jobSystem;

   NOUS_Multithreading::UnregisterMainThread();
   
   return 0;
}
```

---

### **Documentation**  

## ⚙️ `NOUS_Job`
Represents an individual executable task with a name and function.

---

### `NOUS_Job(const std::string& name, std::function<void()> func)`
Creates a new job with a given name and function.

- **Parameters:**
  - `name` — A string identifier for the job (used for debugging/logging).
  - `func` — The function to execute when the job runs.

---

### `void Execute()`
Executes the job's stored function.

---

### `const std::string& GetName()`
Returns the name identifier of the job.

---

## 🧵 `NOUS_Thread`
A wrapper around `std::thread` with additional metadata for job execution, tracking, and debug support.

---

### Constructors & Assignment

#### `NOUS_Thread()`
Initializes a thread object with default values. Does not start the thread automatically.

#### `~NOUS_Thread()`
Destructor. Automatically joins the thread if it is still running.

---

### `void Start(std::function<void()> func)`
Starts the thread with the specified function.

- **Parameters:**
  - `func` — The function to be executed on the thread.
- **Note:** Does nothing if the thread is already running.

---

### `void Join()`
Joins the thread if it's joinable and marks it as no longer running.

---

### Setters and Getters

#### `void SetName(const std::string& name)`
Sets a name for the thread (for debugging purposes).

#### `const std::string& GetName() const`
Returns the thread's name.

#### `void SetThreadState(ThreadState state)`
Sets the current state (`READY`, `RUNNING`) of the thread.

#### `ThreadState GetThreadState() const`
Gets the current thread state.

#### `void SetCurrentJob(NOUS_Job* job)`
Assigns the job currently running on the thread.

#### `NOUS_Job* GetCurrentJob() const`
Returns a pointer to the currently running job, or `nullptr`.

#### `bool IsRunning() const`
Returns `true` if the thread is still active.

#### `uint16_t GetID() const`
Returns the numeric ID of the thread.

---

### Execution Time Tracking

#### `void StartExecutionTimer()`
Starts a timer to track how long the thread runs its current job.

#### `void StopExecutionTimer()`
Stops the timer after job execution ends.

#### `double GetExecutionTimeMS() const`
Returns the execution duration in milliseconds.  
If the timer is still running, returns the current elapsed time.

---

### Thread Identification Utilities

#### `void SetThreadID(std::thread::id id)`
Manually sets a thread ID using the hash of the given `std::thread::id`.

#### `static uint16_t GetThreadID(std::thread::id id)`
Generates a hashed `uint16_t` ID from a given thread ID.

---

### State Description

#### `static const std::string GetStringFromState(const ThreadState& state)`
Converts a `ThreadState` enum to a string representation (`READY`, `RUNNING`, or `UNKNOWN`).

---

### Sleep Utility

#### `static const void SleepMS(const uint32_t& ms)`
Pauses the current thread for the specified number of milliseconds.

---

## 🧵 `NOUS_ThreadPool`
Manages a pool of worker threads and distributes jobs among them.

---

### `explicit NOUS_ThreadPool(uint8_t numThreads)`
Constructs a new thread pool with a specified number of worker threads.

- **Parameters:**
  - `numThreads` — Number of worker threads to initialize.
- **Note:** Marked `explicit` to prevent accidental implicit conversions.
- **Behavior:** Spawns `numThreads` and assigns each a worker loop.

---

### `void SubmitJob(NOUS_Job* job)`
Adds a job to the internal queue and notifies a waiting worker thread.

- **Parameters:**
  - `job` — Pointer to the job to be executed.

---

### `void Shutdown()`
Gracefully shuts down the thread pool.

- **Description:**  
  - Waits for all jobs to finish.  
  - Deletes all remaining queued jobs.  
  - Joins and destroys all worker threads.  
- **Usage:** Should be called manually if early shutdown is needed (automatically called in destructor).

---

### `const std::vector<NOUS_Thread*>& GetThreads()`
Returns a reference to the internal vector of worker threads.

- **Returns:** `std::vector` of pointers to `NOUS_Thread`.

---

### `const std::queue<NOUS_Job*>& GetJobQueue()`
Returns a reference to the internal job queue.

- **Returns:** `std::queue` of pending `NOUS_Job*` tasks.

---

## 🧠 `NOUS_JobSystem`
High-level interface for job submission and management.

---

### `NOUS_JobSystem(const uint8_t size = c_MAX_HARDWARE_THREADS)`
Creates a new job system with a configurable number of worker threads.

- **Parameters:**
  - `size` — Number of threads in the thread pool.  
    If omitted, defaults to `c_MAX_HARDWARE_THREADS`.
- **Note:** If `size` is `0`, jobs will execute sequentially on the main thread.

---

### `void SubmitJob(std::function<void()> userJob, const std::string& jobName = "Unnamed")`
Submits a job to the thread pool to be executed by an available worker thread.

- **Parameters:**
  - `userJob` — The function (job) to execute.
  - `jobName` — (Optional) A string identifier for the job.
- **Note:** If no worker threads are present, the job is executed immediately on the main thread.

---

### `void WaitForPendingJobs()`
Blocks the calling thread until all submitted jobs are completed.

---

### `void Resize(uint8_t newSize)`
Resizes the thread pool to a new number of worker threads.

- **Parameters:**
  - `newSize` — New number of worker threads.
- **Note:** Waits for all current jobs to finish before resizing.
- **Note:** If `newSize` is `0`, the system switches to single-threaded mode.

---

### `const NOUS_ThreadPool& GetThreadPool() const`
Returns a reference to the internal `NOUS_ThreadPool` used by the job system.

---

### `int GetPendingJobs() const`
Returns the number of pending, unprocessed jobs still in the system.

---

## 🔧 `Global Functions`

### `void RegisterMainThread()`
Initializes the main thread tracking.

- **Description:** Registers the main thread as a `NOUS_Thread` instance for timing and debug purposes.
- **Note:** Must be paired with `UnregisterMainThread()` to prevent memory leaks.

---

### `void UnregisterMainThread()`
Cleans up the main thread instance.

- **Description:** Deletes the registered main thread if it exists.
- **Note:** Should be called during application shutdown.

---

### `NOUS_Thread* GetMainThread()`
Retrieves the registered main thread.

- **Returns:** A pointer to the `NOUS_Thread` representing the main thread. Returns `nullptr` if not registered.

---

### `void JobSystemDebugInfo(const NOUS_JobSystem& system)`
Prints job system diagnostics to the console.

- **Parameters:** `system` – A reference to the active `NOUS_JobSystem` instance.
