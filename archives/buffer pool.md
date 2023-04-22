## OVERVIEW

During the semester, you will be building a new disk-oriented storage manager for the BusTub DBMS. Such a storage manager assumes that the primary storage location of the database is on disk.

您将为 BusTub DBMS 构建一个新的面向磁盘的存储管理器。 这样的存储管理器假定数据库的主要存储位置在磁盘上。

The first programming project is to implement a buffer pool in your storage manager. The buffer pool is responsible for moving physical pages back and forth from main memory to disk. It allows a DBMS to support databases that are larger than the amount of memory that is available to the system. The buffer pool's operations are transparent to other parts in the system. For example, the system asks the buffer pool for a page using its unique identifier (page_id_t) and it does not know whether that page is already in memory or whether the system has to go retrieve it from disk.

第一个编程项目是在您的存储管理器中实现一个缓冲池。 缓冲池负责将物理页面从主内存来回移动到磁盘。 它允许 DBMS 支持大于系统可用内存量的数据库。 缓冲池的操作对系统中的其他部分是透明的。 例如，系统使用其唯一标识符 (page_id_t) 向缓冲池请求一个页面，并且它不知道该页面是否已经在内存中或者系统是否必须从磁盘中检索它。

Your implementation will need to be thread-safe. Multiple threads will be accessing the internal data structures at the same and thus you need to make sure that their critical sections are protected with latches (these are called "locks" in operating systems).

您的实现需要是线程安全的。 多个线程将同时访问内部数据结构，因此您需要确保它们的关键部分受到latches的保护（这些在操作系统中称为“lock”）。

You will need to implement the following two components in your storage manager:

您需要在存储管理器中实现以下两个组件：

- Clock Replacement Policy
时钟更换政策

- Buffer Pool Manager
缓冲池管理器

All of the code in this programming assignment must be written in C++ (specifically C++17). It is generally sufficient to know C++11. If you have not used C++ before, here is a short tutorial on the language. More detailed documentation of language internals is available on cppreference. A Tour of C++ and Effective Modern C++ are also digitally available from the CMU library.

此编程作业中的所有代码都必须用 C++（特别是 C++17）编写。 通常了解 C++11 就足够了。 cppreference 上提供了更详细的语言内部文档。 *A Tour of C++* 和 *Effective Modern C++* 也是被推荐的。

There are many tutorials and walkthroughs available to teach you how to use gdb effectively. Here are some that we have found useful:

有许多教程和演练可以教您如何有效地使用 gdb。 以下是我们发现有用的一些内容：

- Debugging Under Unix: gdb Tutorial
- GDB Tutorial: Advanced Debugging Tips For C/C++ Programmers
- Give me 15 minutes & I'll change your view of GDB [VIDEO]

This is a single-person project that will be completed individually (i.e. no groups).

## PROJECT SPECIFICATION

For each of the following components, we are providing you with stub classes that contain the API that you need to implement. You should not modify the signatures for the pre-defined functions in these classes. If you modify the signatures, the test code that we use for grading will break and you will get no credit for the project. You also should not add additional classes in the source code for these components. These components should be entirely self-contained.

对于以下每个组件，我们都为您提供存根类(这意味实现了接口，但是失陷后的每个方法都是空的)，其中包含您需要实现的 API。 您不应修改这些类中预定义函数的签名。 如果您修改签名，我们用于评分的测试代码将被破坏，您将无法获得该项目的分数。 您也不应该在这些组件的源代码中添加其他类。 这些组件应该是完全独立的。

If a class already contains data members, you should not remove them. For example, the BufferPoolManager contains DiskManager and Replacer objects. These are required to implement the functionality that is needed by the rest of the system. On the other hand, you may need to add data members to these classes in order to correctly implement the required functionality. You can also add additional helper functions to these classes. The choice is yours.

如果一个类已经包含数据成员，则不应删除它们。 例如，BufferPoolManager 包含 DiskManager 和 Replacer 对象。 这些是实现系统其余部分所需功能所必需的。 另一方面，您可能需要向这些类添加数据成员以正确实现所需的功能。 您还可以向这些类添加其他辅助函数。 这是你的选择。

You are allowed to use any built-in C++17 containers in your project unless specified otherwise. It is up to you to decide which ones you want to use. Note that these containers are not thread-safe and that you will need to include latches in your implementation to protect them. You may not bring in additional third-party dependencies (e.g. boost).

除非另有说明，否则您可以在项目中使用任何内置的 C++17 容器。 由您决定要使用哪些。 请注意，这些容器不是线程安全的，您需要在实现中包含闩锁以保护它们。 您可能不会引入额外的第三方依赖项（例如 boost）。

### TASK #1 - CLOCK REPLACEMENT POLICY

This component is responsible for tracking page usage in the buffer pool. You will implement a new sub-class called ClockReplacer in src/include/buffer/clock_replacer.h and its corresponding implementation file in src/buffer/clock_replacer.cpp. ClockReplacer extends the abstract Replacer class (src/include/buffer/replacer.h), which contains the function specifications.

该组件负责跟踪缓冲池中的页面使用情况。 您将在 src/include/buffer/clock_replacer.h 中实现一个名为 ClockReplacer 的新子类，并在 src/buffer/clock_replacer.cpp 中实现其对应的实现文件。 ClockReplacer 扩展了抽象 Replacer 类 (src/include/buffer/replacer.h)，其中包含函数规范。

The size of the ClockReplacer is the same as buffer pool since it contains placeholders for all of the frames in the BufferPoolManager. However, not all the frames are considered as in the ClockReplacer. The ClockReplacer is initialized to have no frame in it. Then, only the newly unpinned ones will be considered in the ClockReplacer. Adding a frame to or removing a frame from a replacer is implemented by changing a reference bit of a frame. The clock hand initially points to the placeholder of frame 0. For each frame, you need to track two things: 1. Is this frame currently in the ClockReplacer? 2. Has this frame recently been unpinned (ref flag)?

ClockReplacer 的大小与缓冲池相同，因为它包含 BufferPoolManager 中所有帧的占位符。 然而，并非所有帧都被视为在 ClockReplacer 中。 ClockReplacer 被初始化为没有帧。 然后，在 ClockReplacer 中只会考虑新取消固定的那些。 通过更改帧的引用位来实现向替换器添加帧或从替换器中删除帧。 时钟指针最初指向帧 0 的占位符。对于每一帧，您需要跟踪两件事：1. 该帧当前是否在 ClockReplacer 中？ 2. 该帧最近是否已取消固定（ref flag）？

In some scenarios, the two are the same. For example, when you unpin a page, both of the above are true. However, the frame stays in the ClockReplacer until it is pinned or victimized, but its ref flag is modified by the clock hand.

在某些情况下，两者是相同的。 例如，当您取消固定某个页面时，以上两种情况均成立。 但是，该框架会一直保留在 ClockReplacer 中，直到它被固定或受害，但它的 ref 标志会被the clock hand修改。

You will need to implement the clock policy discussed in the class. 
您将需要实施课堂上讨论的时钟策略。

You will need to implement the following methods:

- Victim(T*) : Starting from the current position of clock hand, find the first frame that is both in the `ClockReplacer` and with its ref flag set to false. If a frame is in the `ClockReplacer`, but its ref flag is set to true, change it to false instead. This should be the only method that updates the clock hand.

    从时钟指针的当前位置开始，找到第一个既在 ClockReplacer 中又将其 ref 标志设置为 false 的帧。 如果帧在“ClockReplacer”中，但其 ref 标志设置为 true，请将其更改为 false。 这应该是更新时钟指针的唯一方法。

- Pin(T) : This method should be called after a page is pinned to a frame in the BufferPoolManager. It should remove the frame containing the pinned page from the ClockReplacer.

    在将页面固定到 BufferPoolManager 中的帧后，应调用此方法。 它应该从 ClockReplacer 中删除包含固定页面的帧。

- Unpin(T) : This method should be called when the pin_count of a page becomes 0. This method should add the frame containing the unpinned page to the ClockReplacer.

    

- Size() : This method returns the number of frames that are currently in the ClockReplacer.
The implementation details are up to you. You are allowed to use built-in STL containers. You can assume that you will not run out of memory, but you must make sure that the operations are thread-safe.

TASK #2 - BUFFER POOL MANAGER
Next, you need to implement the buffer pool manager in your system (BufferPoolManager). The BufferPoolManager is responsible for fetching database pages from the DiskManager and storing them in memory. The BufferPoolManager can also write dirty pages out to disk when it is either explicitly instructed to do so or when it needs to evict a page to make space for a new page.

To make sure that your implementation works correctly with the rest of the system, we will provide you with some of the functions already filled in. You will also not need to implement the code that actually reads and writes data to disk (this is called the DiskManager in our implementation). We will provide that functionality for you.

All in-memory pages in the system are represented by Page objects. The BufferPoolManager does not need to understand the contents of these pages. But it is important for you as the system developer to understand that Page objects are just containers for memory in the buffer pool and thus are not specific to a unique page. That is, each Page object contains a block of memory that the DiskManager will use as a location to copy the contents of a physical page that it reads from disk. The BufferPoolManager will reuse the same Page object to store data as it moves back and forth to disk. This means that the same Page object may contain a different physical page throughout the life of the system. The Page object's identifer (page_id) keeps track of what physical page it contains; if a Page object does not contain a physical page, then its page_id must be set to INVALID_PAGE_ID.

Each Page object also maintains a counter for the number of threads that have "pinned" that page. Your BufferPoolManager is not allowed to free a Page that is pinned. Each Page object also keeps track of whether it is dirty or not. It is your job to record whether a page was modified before it is unpinned. Your BufferPoolManager must write the contents of a dirty Page back to disk before that object can be reused.

Your BufferPoolManager implementation will use the ClockReplacer class that you created in the previous steps of this assignment. It will use the ClockReplacer to keep track of when Page objects are accessed so that it can decide which one to evict when it must free a frame to make room for copying a new physical page from disk.

You will need to implement the following functions defined in the header file (src/include/buffer/buffer_pool_manager.h) in the source file (src/buffer/buffer_pool_manager.cpp):

FetchPageImpl(page_id)
NewPageImpl(page_id)
UnpinPageImpl(page_id, is_dirty)
FlushPageImpl(page_id)
DeletePageImpl(page_id)
FlushAllPagesImpl()
For FetchPageImpl,you should return NULL if no page is available in the free list and all other pages are currently pinned. FlushPageImpl should flush a page regardless of its pin status.

Refer to the function documentation for details on how to implement these functions. Don't touch the non-impl versions, we need those to grade your code.

INSTRUCTIONS
CREATING YOUR OWN PROJECT REPOSITORY
Go to the BusTub GitHub page and click the Fork button to create a copy of the repository under your account.

You can then clone your private repository on your local machine in the filesystem, say under the ~/git directory. If you have not used git before, here's some information on source code management using git.

SETTING UP YOUR DEVELOPMENT ENVIRONMENT
To build the system from the commandline, execute the following commands:

$ mkdir build
$ cd build
$ cmake ..
$ make
To speed up the build process, you can use multiple threads by passing the -j flag to make. For example, the following command will build the system using four threads:

$ make -j 4
TESTING
You can test the individual components of this assigment using our testing framework. We use GTest for unit test cases. There are two separate files that contain tests for each component:

ClockReplacer: test/buffer/clock_replacer_test.cpp
BufferPoolManager: test/buffer/buffer_pool_manager_test.cpp
You can compile and run each test individually from the command-line:

$ mkdir build
$ cd build
$ make clock_replacer_test
$ ./test/clock_replacer_test
You can also run make check-tests to run ALL of the test cases. Note that some tests are disabled as you have not implemented future projects. You can disable tests in GTest by adding a DISABLED_ prefix to the test name.

Important: These tests are only a subset of the all the tests that we will use to evaluate and grade your project. You should write additional test cases on your own to check the complete functionality of your implementation.

FORMATTING
Your code must follow the Google C++ Style Guide. We use Clang to automatically check the quality of your source code. Your project grade will be zero if your submission fails any of these checks.

Execute the following commands to check your syntax. The format target will automatically correct your code. The check-lint and check-clang-tidy targets will print errors and instruct you how to fix it to conform to our style guide.

$ make format
$ make check-lint
$ make check-clang-tidy
DEVELOPMENT HINTS
Instead of using printf statements for debugging, use the LOG_* macros for logging information like this:

LOG_INFO("# Pages: %d", num_pages);
LOG_DEBUG("Fetching page %d", page_id);
To enable logging in your project, you will need to reconfigure it like this:

$ mkdir build
$ cd build
$ cmake -DCMAKE_BUILD_TYPE=DEBUG ..
$ make
The different logging levels are defined in src/include/common/logger.h. After enabling logging, the logging level defaults to LOG_LEVEL_INFO. Any logging method with a level that is equal to or higher than LOG_LEVEL_INFO (e.g., LOG_INFO, LOG_WARN, LOG_ERROR) will emit logging information. Note that you will need to add #include "common/logger.h" to any file that you want to use the logging infrastructure.

We encourage you to use gdb to debug your project if you are having problems.

 Post all of your questions about this project on Piazza. Do not email the TAs directly with questions.

GRADING RUBRIC
Each project submission will be graded based on the following criteria:

Does the submission successfully execute all of the test cases and produce the correct answer?
Does the submission execute without any memory leaks?
Does the submission follow the code formatting and style policies?
Note that we will use additional test cases to grade your submission that are more complex than the sample test cases that we provide you.

LATE POLICY
See the late policy in the syllabus.

SUBMISSION
After completing the assignment, you can submit your implementation to Gradescope:

https://www.gradescope.com/courses/58985/
You only need to include the following files:

src/include/buffer/clock_replacer.h
src/buffer/clock_replacer.cpp
src/include/buffer/buffer_pool_manager.h
src/buffer/buffer_pool_manager.cpp
You can submit your answers as many times as you like and get immediate feedback. (Once the autograder is up -- working on it!)

COLLABORATION POLICY
Every student has to work individually on this assignment.
Students are allowed to discuss high-level details about the project with others.
Students are not allowed to copy the contents of a white-board after a group meeting with other students.
Students are not allowed to copy the solutions from another colleague.