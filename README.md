# NOUS_Multithreading.h

---

### A Thread Pool based Job System Implementation

A single-header, cross-platform C++11 library for managing concurrent task execution.

---

**Author:** Francesc Teruel Rodriguez ([francesctr4](https://github.com/francesctr4) on GitHub)  
**Created:** 30/06/2025  
**Version:** 1.0  
**Repository:** [https://github.com/francesctr4/NOUS_Multithreading.h](https://github.com/francesctr4/NOUS_Multithreading.h)  

**License:** MIT License  
© 2025 Francesc Teruel Rodriguez

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

### **Documentation**  

#### NOUS_Job
Represents an executable task with a name and function.

/// @brief NOUS_Job constructor.
```NOUS_Job(const std::string& name, std::function<void()> func)```

/// @brief Executes the stored function inside the job.
```void Execute();```

/// @return std::string with the NOUS_Job name identifier.
```const std::string& GetName();```

#### NOUS_Thread
std::thread wrapper with additional metadata (name, state, timer,...)

```Start, join, and track execution time```

```Set and get job/thread name/state```

```static SleepMS(ms) utility```

#### NOUS_ThreadPool
Manages a pool of worker threads and job distribution between them.

/// @brief NOUS_ThreadPool constructor.
/// @note Marked explicit to prevent implicit conversions and copy-initialization from a single argument.
```explicit NOUS_ThreadPool(uint8_t numThreads);```

/// @brief Adds a job to the queue and notifies a worker.
/// @param job The job to be executed.
```void SubmitJob(NOUS_Job* job);```

/// @brief Deletes pending jobs, joins all threads and cleans up resources afterwards.
```void Shutdown();```

/// @return A vector of NOUS_Thread contained inside the thread pool.
```const std::vector<NOUS_Thread*>& GetThreads();```

/// @return A queue of NOUS_Job to be executed by the thread pool.
```const std::queue<NOUS_Job*>& GetJobQueue();```

#### NOUS_JobSystem
High-level interface for job submission and management.

/// @brief NOUS_JobSystem constructor.
/// @param size: Number of worker threads available inside the thread pool.
/// @note If size is not specified, c_MAX_HARDWARE_THREADS is used.
```NOUS_JobSystem(const uint8_t size = c_MAX_HARDWARE_THREADS);```

/// @brief Submits a job to the thread pool, to be executed by a free worker thread.
/// @note Job executes immediately if thread pool size is 0 (running on Main Thread).
/// @param userJob: The function to execute.
/// @param jobName: Optional name identifier.
```void SubmitJob(std::function<void()> userJob, const std::string& jobName = "Unnamed");```

/// @brief Blocks until all submitted jobs complete.
```void WaitForPendingJobs();```

/// @brief Resizes the thread pool to the specified number of threads.
/// @param newSize: The new number of worker threads in the pool.
/// @note If the size passed is 0, the program becomes single-threaded.
/// @note Ensures all current jobs finish before resizing.
```void Resize(uint8_t newSize);```

/// @return Reference to the underlying thread pool.
```const NOUS_ThreadPool& GetThreadPool() const;```

/// @return Number of pending unprocessed jobs.
```int GetPendingJobs() const;```

## 🔧 Global Functions

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
