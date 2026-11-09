# CSI-4337-Programming-Assignment-3-Virtual-Memory-Manager-Simulator
Programming Assignment 3: Virtual Memory Manager Simulator
Jump to navigationJump to search

Concurrent Main Memory Management Simulator

Chapter: Chapter 9 – Main Memory
Programming language: C++11
Submission: One source file, main.cpp
Objective

This assignment reinforces main memory management concepts through a simulated memory management unit (MMU). A professor-provided thread pool submits requests from four clients. Your task is to implement the shared memory manager used by those threads.
After completing the assignment, you should be able to:
Allocate physical memory frames to logical pages.
Maintain an independent page table for each client.
Translate logical addresses into physical addresses.
Use a shared translation lookaside buffer (TLB) with FIFO replacement.
Protect shared memory-management data from concurrent access.
Explain the difference between a TLB miss and an unallocated page.
You must not implement or modify the thread pool. The professor provides the worker threads, request queue, test workload, and output of individual request results. Your work is limited to the designated student implementation section of the starter file.
Problem Description

Four clients concurrently request translations of logical memory addresses. Each client has its own logical address space, but all clients share one simulated physical memory consisting of twelve frames.
When a client references a page that has not yet been assigned a frame, the memory manager must allocate the lowest-numbered free frame. Once a page has a frame, its assignment remains unchanged for the remainder of the simulation.
For each successful request, the manager returns the logical page number, offset, physical frame number, calculated physical address, and TLB hit/miss result. If a request is invalid or no frame is available for a previously unallocated page, the request fails.
All pages remain resident once allocated. Do not implement demand paging from disk, swapping, deallocation, or physical-memory page replacement. TLB replacement is separate from physical-frame allocation.
Shared Memory Architecture Diagram.png

<img width="1200" height="1022" alt="image" src="https://github.com/user-attachments/assets/a4304e4b-e8bb-4dfc-a6cb-25ec1062b42f" />

System Configuration

Parameter	Value
Number of clients	4 (client IDs 0–3)
Professor-provided worker threads	4
Logical pages per client	4 (pages 0–3)
Page size	256 bytes
Logical address range per client	0–1023
Physical frames	12 (frames 0–11)
Total physical memory	3,072 bytes
Shared TLB capacity	4 entries
TLB replacement policy	First-in, first-out (FIFO)
Frame allocation policy	Lowest-numbered available frame
Each page-table entry must indicate whether the page has been assigned a frame and, if so, which frame. Each TLB entry must identify both the client and the logical page, because different clients may use the same page number.
Required Implementation

Part A – Physical Frame Allocation (25 points)
Implement the physical-frame allocation mechanism and per-client page tables.
Initially, all twelve frames are free and all page-table entries are unmapped.
On the first access to an unmapped valid page, assign the lowest-numbered free frame and record that mapping.
On later accesses to the same client/page pair, reuse its existing frame.
Never assign one physical frame to two different client/page pairs.
Once all frames are allocated, reject first accesses to additional unmapped pages with a failed result.
Continue to translate addresses belonging to pages already allocated.
The mapping of a client/page pair must remain stable throughout the simulation.
Part B – Address Translation (20 points)
Calculate the page number and offset from the logical address:
page   = logicalAddress / 256

offset = logicalAddress % 256
After obtaining the mapped frame number, calculate the physical address:
physicalAddress = frameNumber * 256 + offset
For example, logical address 600 belongs to page 2 and has offset 88. If that client's page 2 has been allocated frame 5, the resulting physical address is:
physicalAddress = 5 * 256 + 88 = 1368
The physical frame assigned to a page depends on the order of concurrent requests. Therefore, the exact physical address may differ between program executions, but it must always be consistent with the page table for that execution.
Reject an invalid client ID or any logical address greater than 1023. The provided access() interface uses unsigned integer arguments; it cannot receive a negative value as a negative integer. Invalid requests must not change allocation or TLB statistics.
Part C – Shared TLB (20 points)
Implement a four-entry TLB shared by all clients.
Each TLB entry must contain:
Client ID
Logical page number
Physical frame number
Valid indicator
On a valid request:
Search the TLB for an entry matching the client ID and page number.
If a matching entry exists, use its frame number and record a TLB hit.
Otherwise, find the mapping in that client's page table or allocate a frame if the page is not yet mapped.
If translation succeeds, record a TLB miss and insert the translation into the TLB.
If insertion requires replacement, evict the entry that was inserted earliest.
A TLB hit must not change FIFO order. A TLB miss for an already mapped page must not allocate another frame. A failed allocation must not insert a TLB entry or increment the hit/miss counters.
Initially all TLB entries are invalid. At the end of the program, print the valid TLB entries from oldest to newest.
Part D – Concurrent Access and Synchronization (20 points)
The professor-provided thread pool submits requests concurrently to one shared MemoryManager instance.
Use C++11 synchronization primitives, such as std::mutex and std::lock_guard, to protect shared state.
At minimum, the following operations must be thread-safe:
Checking and updating page-table entries
Checking and allocating physical frames
Searching, inserting, and replacing TLB entries
Updating shared counters
Reading summary information after processing ends
Treat one call to access() as an atomic memory-management transaction with respect to other such calls. A single mutex protecting the operation is acceptable; fine-grained locking is not required.
In particular, two simultaneous requests for the same unmapped page must not result in the allocation of two different frames.
Part E – Statistics, Summary, and Documentation (15 points)
Implement printSummary() to display:
Total requests processed
Successful translations
Failed requests
Number of allocated frames
TLB hits
TLB misses
TLB hit rate and miss rate
Page-table mappings for all clients
Final TLB contents, oldest entry first
For this assignment, only successful translations count toward TLB hits and misses. Accordingly:
TLB hits + TLB misses = successful translations

TLB hit rate  = 100 * TLB hits / successful translations
TLB miss rate = 100 * TLB misses / successful translations
If no translations succeed, report both rates as 0.00% rather than dividing by zero. Display the rates to two decimal places.
Follow Booth's Code Documentation Standard, including the required file header, main program documentation (already present in the instructor framework where applicable), and appropriate documentation for all student-written structures and functions. Keep all source code lines at or below 80 characters.
At the end of the file, include a comment block answering these questions:
Why is synchronization necessary when concurrent threads allocate physical frames?
Why must a TLB entry include the client ID as well as the page number?
What is the difference between a TLB miss and a request for a page that has not yet been allocated a frame?
Why can two clients use the same logical address but obtain different physical addresses?
Professor-Provided Framework

The instructor supplies a C++11 starter program with:
A fixed four-worker thread pool
A thread-safe queue of memory requests
Three passes through the four clients and their four logical pages
A Result structure and MemoryManager interface
Per-request output generated by the worker threads
Implement the constructor, access(), printSummary(), and any supporting data members or helper functions inside the designated student section of the supplied file.
Do not modify code outside the student implementation section. The starter code uses an unsigned client ID and logical address. The instructor workload supplies valid IDs and addresses; your implementation must nevertheless reject out-of-range values.
CSI4337_MainMemory_ThreadPool_Starter.cpp
Instructor Workload and Expected Results

The instructor harness produces three passes of sixteen requests. In each pass, every client accesses one address in each of its four logical pages. The offsets are 10, 44, 88, and 132 for pages 0, 1, 2, and 3, respectively.
Client 0:  10   300   600   900
Client 1:  10   300   600   900
Client 2:  10   300   600   900
Client 3:  10   300   600   900

Repeat the above requests for three passes.
There are sixteen distinct client/page pairs, but only twelve available physical frames. With no deallocation or page replacement, exactly twelve pairs will receive frames and four pairs will remain unmapped. Each pair is requested three times.
Statistic	Expected value
Total requests	48
Successful translations	36
Failed requests	12
Allocated frames	12
Unmapped client/page pairs	4
TLB hits and misses combined	36
Concurrency note: Worker scheduling may change which twelve client/page pairs obtain frames, their assigned frame numbers, request output order, TLB hit/miss counts, and final TLB entries. The figures above are invariant, but an exact transcript of individual requests is not. Do not hard-code any allocation order or TLB performance result.
Testing Requirements

Test the completed program across repeated executions. Verify the following conditions:
Test	Required behavior
First access to an unmapped valid page	Allocates exactly one free frame
Repeat access to an allocated page	Returns the same frame as before
Two clients use the same page number	Translations remain isolated by client ID
TLB hit	Uses the cached frame without changing FIFO order
TLB miss for an allocated page	Uses the page table without allocating a new frame
Fifth distinct TLB insertion	Replaces the oldest inserted entry
All twelve frames occupied	New unmapped pages fail; existing mappings still work
Invalid client ID or logical address	Request fails without changing allocation or TLB statistics
Concurrent first accesses	Never allocate two frames for one client/page pair
Final statistics	Match the invariant totals in Section 6
All successful physical addresses must be in the range 0–3071. Each mapped client/page pair must have a unique frame, and all computed offsets must be in the range 0–255.
Implementation Constraints

Use C++11 and submit one file named main.cpp.
Keep the instructor-provided thread pool and request workload unchanged.
Implement the shared memory manager within the designated student section.
You may use fixed-size arrays and simple struct types for page tables, frames, and the TLB.
Do not use std::map or std::unordered_map to implement the TLB.
Use a C++11 mutex to protect shared memory-management data.
Do not use an operating system memory allocation API to simulate frame allocation.
Do not implement paging to disk, swapping, or deallocation.
Compile with:
g++ -std=c++11 -Wall -Wextra -pedantic -pthread main.cpp -o memorySim
Execute with:
./memorySim
Grading Rubric

Category	Points
Physical frame allocation and page tables	25
Logical-to-physical address translation	20
Shared TLB and FIFO replacement	20
Thread synchronization and shared-state correctness	20
Statistics, summary, written explanations, and documentation	15
Total	100
Submission

Submit the completed main.cpp to the course Upload Site. The submission must compile with the command above and run without modifying the professor-provided framework.
Learning Outcome

This assignment demonstrates the distinction between logical and physical memory, the purpose of per-process page tables, the role of a TLB, and the synchronization required when concurrent workers use a shared memory manager. A TLB miss indicates that a translation was not found in the translation cache; it does not necessarily mean that physical memory allocation is required.
