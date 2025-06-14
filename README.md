# NOUS_Multithreading.h

---

### A Thread Pool based Job System Implementation

A single-header, cross-platform C++11 library for managing concurrent task execution.

---

**Author:** Francesc Teruel Rodriguez (francesctr4 on GitHub)  
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
- add the .h to your project

**Documentation:**  
```
#include "NOUS_Multithreading.h"

int main() {
    using namespace NOUS_Multithreading;
    RegisterMainThread();  // Enable main thread tracking

    // Create job system (auto-size to hardware threads)
    NOUS_JobSystem jobSystem;

    // Submit jobs
    jobSystem.SubmitJob([] {
        std::cout << "Job 1 running on thread " 
                  << std::this_thread::get_id() << "\n";
    }, "Job_1");

    jobSystem.SubmitJob([] {
        std::cout << "Job 2 running on thread " 
                  << std::this_thread::get_id() << "\n";
    }, "Job_2");

    jobSystem.WaitForPendingJobs();  // Block until done

    JobSystemDebugInfo(jobSystem);   // Print diagnostics
    UnregisterMainThread();
    return 0;
}
```
