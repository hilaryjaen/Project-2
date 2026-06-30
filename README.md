# Project 2 - Readers/Writers
Names: Hilary Jaen, Matthew Cisco

Group: 6

## Implementation Note

This project was outlined using `pthread.h` together with `semaphore.h`. However, on macOS the `sem_init()` approach used in this project does not work, so the implementation was adapted to use Pthreads mutexes instead.

### Readers-Writers Variation Used

This implementation follows the first readers-writers variation shown in the course material. We have read the book and modules. In this version, multiple readers may access the shared resource at the same time, but the writers need exclusive access. This variation can lead to starvation-like behavior depending on the timing and number of threads.

## Synchronization Mapping

The following table shows the relationship between the semaphore operations and the Pthreads operations used in this implementation.

- `wait(mutex)`  
  Replaced by: `pthread_mutex_lock(&mutex)`  
  Purpose: protect access to `readCount`.

- `signal(mutex)`  
  Replaced by: `pthread_mutex_unlock(&mutex)`  
  Purpose: release protection of `readCount`.

- `wait(rw_mutex)`  
  Replaced by: `pthread_mutex_lock(&rwMutex)`  
  Purpose: lock the shared resource so that a writer can write or the first reader can block writers.

- `signal(rw_mutex)`  
  Replaced by: `pthread_mutex_unlock(&rwMutex)`  
  Purpose: release the shared resource.

- `sem_init(..., 1)` for binary semaphores  
  Replaced by: `pthread_mutex_init(&mutex, 0)` and `pthread_mutex_init(&rwMutex, 0)`  
  Purpose: initialize the locks used in the readers/writers solution.

- `sem_destroy(...)`  
  Replaced by: `pthread_mutex_destroy(...)`  
  Purpose: release synchronization resources at the end of execution.

## Pthreads Functions Used

- `pthread_create()`  
  Creates reader and writer threads.

- `pthread_join()`  
  Waits for threads to finish before the program exits.

- `pthread_mutex_init()`  
  Initializes mutex locks.

- `pthread_mutex_lock()`  
  Locks a mutex before accessing shared state.

- `pthread_mutex_unlock()`  
  Unlocks a mutex after leaving a protected section.

- `pthread_mutex_destroy()`  
  Destroys mutex locks at the end of the program.

## Shared State Used

- `readCount`  
  Number of readers currently inside the reading section.

- `totalReads`  
  Total completed read operations.

- `totalWrites`  
  Total completed write operations.

- `keepgoing`  
  Controls when the threads should stop running.

## Experiment 1: Attempt to starve reader threads
Command used:

./ReadersWriters 1000000000 500000000 1000000000 500000000 1000000 1000 1000000 1000 1 6

### What We observed:
- Run time: 10.28 seconds
- Total number of reads: 16
- Total number of writes: 6585

This experiment created a workload with one reader and six fast writers. The writers were able to reach the critical section more often because their sleep times were much shorter. As a result, the reader had few opportunities to perform work during the same time interval. Although the reader was not completely blocked, the large gap between 16 reads and 6585 writes shows an imbalance against the reader and indicates reader starvation-like behavior under these timing conditions.

## Experiment 2: Attempt to stave writer threads
Command used:

./ReadersWriters 1000000 1000 1000000 1000 1000000000 500000000 1000000000 500000000 6 1

### What We observed:
- Run time: 10.31 seconds
-  Total number of reads: 22499
- Total number of writes: 14

This test created a workload with six fast readers and one slower writer. The readers completed a large number of operations while the writer progressed more slowly. The writer was still able to enter the critical section and complete 14 write operations. This means that the writer experienced a disadvantage, but not complete starvation. 

## Conclusion

The results of the experiments reflected the behavior described in the PowerPoint for the first readers-writers variation. The experiments showed that changing the number of threads and their sleep times can create an imbalance between readers and writers. In the second experiment, the readers gained a very large advantage over the writer, which matches the behavior discussed in the PowerPoint. The experiments showed that the synchronization logic worked as expected and that the effects described in the course material can be observed in practice.
