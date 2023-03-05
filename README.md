# Starve-free-Readers-Writers-Solution

The reader-writer problem is a classic synchronization problem in computer science, particularly in operating systems. It arises when multiple processes or threads concurrently access a shared resource that can be either read or written, with the constraint that simultaneous reads are allowed but simultaneous writes are not.

The problem arises because allowing multiple writers to access the shared resource simultaneously can result in inconsistent or corrupted data. Hence, it is necessary to provide mutual exclusion between writers to ensure that only one writer can modify the shared resource at a time. However, allowing multiple readers to access the shared resource simultaneously can improve system performance since multiple readers can read the same data simultaneously without any issues. Therefore, it is necessary to ensure that multiple readers can access the shared resource simultaneously while preventing writers from accessing it concurrently with the readers.

In the reader-writer problem, if a large number of readers keep coming in, a writer could potentially be starved if the readers keep acquiring the resource continuously, not allowing the writer to get access to the resource. This could lead to a situation where the writer never gets to execute, which could lead to significant delays or even system failures if the resource is critical.

Therefore, we need a starve-free solution to ensure that all processes (readers and writers) get a fair chance to access the shared resource. In a starve-free solution, no process is indefinitely blocked or prevented from accessing the resource due to another process continuously accessing the resource. 

Solving this problem correctly can help in enhancing system performance, improving resource utilization, and preventing issues such as race conditions, deadlocks, and livelocks.

### Pseudocode of starve free Reader-Writter problem:

#### Global Variables

```cpp
Semaphore* mutex = new Semaphore(1); // To acheive starve free solution
Semaphore* read_semaphore = new Semaphore(1); // To maintain mutual exclusion in reader counting
Semaphore* write_semaphore = new Semaphore(1); // To maintain mutual exclusion in writer
int reader = 0; // To keep count of the readers
```

#### Reader Process

```cpp
void reader(){
    while(true){
        // Entry Section
        wait(mutex); 
        wait(read_semaphore); 
        reader = reader + 1; // Increment the reader count
        if(reader == 1){
            // If any reader is accessing, then we have to block writer
            wait(write_semaphore);
        }
        signal(read_semaphore);
        signal(mutex);

        // Critical Section

        //Exit Section
        wait(read_semaphore);
        reader = reader - 1; // Decrement the reader count
        if(reader == 0){
            // If every reader has read, then we have to unblock writer
            signal(write_semaphore);
        }
        signal(read_semaphore);
    }
}
```

In the entry section, reader acquires the `mutex` to ensure mutual exclusion. It then waits on the `read_semaphore` to ensure that it does not block a writer from accessing the resource. It increments the `reader` variable, and if it is the first reader to access the shared resource, it waits on the `write_semaphore` to ensure that no writer can access the resource. After acquiring the `write_semaphore`, it signals the `read_semaphore` and releases the `mutex`.

In the critical section, the reader accesses the shared resource.

In the exit section, the reader acquires the `read_semaphore`. It decrements the `reader` variable and checks if it is the last reader to exit the critical section. If it is, it release the `write_semaphore` to allow any waiting writer to access the shared resource. It then signals the `read_semaphore`.

#### Writer Process

```cpp
void writer(){
    while(true){
        // Entry Section
        wait(mutex);
        wait(write_semaphore);

        // Critical Section

        //Exit Section
        signal(write_semaphore);
        signal(mutex);
    }
}
```

In the entry section, it acquires the `mutex` to ensure mutual exclusion. It then waits on the `write_semaphore` to ensure that no other writer or reader is currently accessing the shared resource.

In the critical section, the writer accesses the shared resource.

In the exit section, the writer release the `write_semaphore` to release its control over the shared resource and releases the `mutex`.

#### How the above code achieve mutual exclusion?

In the reader function, the reader first waits on the `mutex` semaphore to ensure that it has exclusive access to the shared data. It then waits on the `read_semaphore` semaphore to ensure that no other readers can enter the critical section while it is in progress. It then increments the `reader` variable to indicate that it is currently reading.

If it is the first reader to enter the critical section (i.e., if reader is equal to 1), it waits on the `write_semaphore` semaphore to ensure that no writers can enter the critical section while it is in progress. It then signals the `read_semaphore` semaphore to allow other readers to enter the critical section.

After the critical section is complete, the reader waits on the `read_semaphore` semaphore to ensure that it has exclusive access to the `reader` variable. It then decrements the `reader` variable to indicate that it has finished reading.

If it is the last reader to leave the critical section (i.e., if reader is equal to 0), it signals the `write_semaphore` semaphore to allow any waiting writers to enter the critical section.

In the writer function, the writer first waits on the `mutex` semaphore to ensure that it has exclusive access to the shared data. It then waits on the `write_semaphore` semaphore to ensure that no other writers or readers can enter the critical section while it is in progress.

After the critical section is complete, the writer signals the `write_semaphore` semaphore to allow any waiting writers or readers to enter the critical section.

#### How the above code acheive progress?

The provided solution to the Reader-Writer problem achieves progress because it allows both readers and writers to access the shared resource without blocking each other indefinitely.

The solution gives priority to the writers by using the write semaphore. If a writer is waiting, no new readers are allowed to enter the critical section. This ensures that writers do not get blocked indefinitely by readers constantly accessing the shared resource because all the users whether it is a reader or writer are queued in `FIFO queue` of `mutex` semaphores.

However, the solution also ensures that readers are not blocked indefinitely by allowing multiple readers to enter the critical section simultaneously, as long as there are no writers waiting to enter the critical section.

Therefore, the provided solution achieves progress by ensuring that both readers and writers can access the shared resource without blocking each other indefinitely, while also giving priority to the writers when they are waiting to enter the critical section.

#### Why above code is starve free?

This solution is starve-free because it ensures that both readers and writers get a fair chance to access the shared resource. 

In this solution, the writer can only access the critical section when there are no readers present. If there are multiple readers accessing the shared resource, a writer who arrives later (After the readers present in the critical section) will have to wait until all the existing readers finish, because users whether it is a reader or writer are queued in the `FIFO queue` of `mutex`. This ensures that the writer does not starve, as it will eventually get access to the shared resource.

Similarly, readers also get a fair chance to access the shared resource. The code ensures that no writer is waiting indefinitely to access the shared resource, and a writer who arrives later will not block a reader who is already accessing the shared resource. This way, readers do not starve, and they will eventually get access to the shared resource.