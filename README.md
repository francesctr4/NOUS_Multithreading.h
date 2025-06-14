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

**Dependencies:**  
- ISO C++11 Standard (compile with `/std:c++11` or `-std=c++11`) or newer

---

**Instructions:**  
Add NOUS_Multithreading.h to Your Project
Simply copy the NOUS_Multithreading.h header file into your project directory or include path.

Include the Header in Your Source Files
#include "NOUS_Multithreading.h"

Start Using the Thread Pool and Job System
Refer to the usage examples and API documentation provided in this repository to integrate the job system into your application.

---

**Documentation:**  
Register the Main Thread
1. Begin by calling RegisterMainThread() to retrieve and store information about the main thread.
   
Initialize the Job System
2. Create and configure an instance of the NOUS_JobSystem, typically as a pointer. If no thread count is specified during initialization, the system will automatically use the maximum number of hardware threads minus one, reserving one for the main thread.
   
Submit Jobs
3. Once the job system is initialized, jobs can be submitted using the SubmitJob() function. Submitted jobs will be automatically queued, managed, and executed by the available worker threads.

Wait for Pending Jobs
4. Before shutting down the system, it is recommended to explicitly call WaitForPendingJobs() to ensure all jobs have completed. While this step is already handled in the job system’s destructor, doing it manually is considered good practice for clarity and control.

Shutdown and Cleanup
5. After all jobs have been processed, destroy the job system instance and call UnregisterMainThread() to clean up the main thread information.

---

**Quick start:**  
```
#include "NOUS_Multithreading.h"

int main(int argc, char** argv)
{
   NOUS_Multithreading::RegisterMainThread();
   
   // Create the job system dynamically
   NOUS_Multithreading::NOUS_JobSystem* jobSystem = new NOUS_Multithreading::NOUS_JobSystem();
   
   // Submit jobs
   jobSystem->SubmitJob([]() { /* task 1 */ }, "Task1");
   jobSystem->SubmitJob([]() { /* task 2 */ }, "Task2");
   
   // Wait for completion
   jobSystem->WaitForPendingJobs();
   
   // Clean up when done
   delete jobSystem;

   NOUS_Multithreading::UnregisterMainThread();
   
   return 0;
}
```
