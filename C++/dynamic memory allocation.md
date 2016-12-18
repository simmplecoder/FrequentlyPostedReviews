#When to use dynamic memory allocation?
It is pretty common for beginners to allocate everything in the free store (also called heap, by calling `new` or `malloc()` or similar), but is there a better way? One should always try to make the best choice.

##Performance
Currently most allocators are so complex that it becomes harder and harder to predict if the next allocation will be a system call, which will probably have an implicit mutex lock (it needs to acquire memory from the system which is shared, so there will be some overhead to maanage it). Though it is *guaranteed* that creating variable in the automatic storage (also known as stack) will certainly be faster, plus you don't need to `free()` or `delete` it since automatic storage automatically runs destructors and reclaims the memory. 
Dynamic allocation also has a performance hit from fragmentation, e.g. the automatic storage in which some bookkeeping is stored (instructions) is usually further from the free store, which can be literally anywhere. Fragmentation leads to [cache misses](http://stackoverflow.com/questions/18559342/what-is-a-cache-hit-and-a-cache-miss-why-context-switching-would-cause-cache-mi).
##Ease of use
It is much harder to get program that does dynamic allocation to get correct compared to program that primarily uses automatic storage. 
###Memory leak
The problem lies in obligation of dynamic memory allocation: programmer *must* free it after it is done, otherwise program will have [memory leak](https://en.wikipedia.org/wiki/Memory_leak). There are multiple solutions to deal with this:

 - Smart pointers
 - Standard Containers
 - Garbage Collection
 - General RAII

If the frequent dynamic memory allocation is needed, programmer should take care of using one of the above to make sure that program won't leak.
###Dangling pointers
Dangling pointers (or any reference to "dead" object) are pointers to object on which destructor was run or the memory is no longer belongs to the program (e.g. someone already `free()`d or `delete`d it). When accessed (dereferenced or made any sort of access to the pointed to object), undefined behavior will be invoked. Some people consider it as "ownership" problem. Indeed, someone has to delete the pointer, but who? In modern C++, there are well established rules of thumb:

 - Use raw pointers (C pointers, e.g. T*) to imply no ownership (it shouldn't be deleted or freed)
 - Use smart pointers to indicate ownership (smart pointer will automatically destroy the object and free the memory on getting out of scope or when requested). [This](http://stackoverflow.com/questions/106508/what-is-a-smart-pointer-and-when-should-i-use-one) answer should give more details.

Smart pointers might have some performance impact, but standard library maintainers are trying their best to make it cost the same as handwritten version.
###RAII
RAII (resource aquisition is initialization) is an idiom used to prevent resource leaks (resource includes memory, file handlers, mutexes, etc). It can also be applied to the dynamic allocation:

    class data
    {
    	char* text;
        int* stats;
    public:
    	data()
        {
        	text = new char[count];
            try 
            {
            	stats = new int[count];
            } //if stats throws, text will leak
            catch (const std::bad_alloc& e)
            {
            	delete[] text;
                throw; //rethrow it, since it is an error
            }
        }
        ...
        ~data()
        {
        	delete[] text;
            delete[] stats;
        }
    }

The example above clearly shows that most of the time standard containers (like `std::vector<>`) are better suited for memory management than handwritten version, since they have automatic memory management and programmer won't need to worry about memory leak during exceptiontional situations.
##Caveats of automatic storage
It is usually very small (on Windows, it is around 1MB), also it stores bookkeeping there. This means that allocating too much automatic storage will shorten recursion depth or any other deep function calls. g++ tries to increase automatic storage size, but it is not always possible. In general, size and lack of automatic storage is implementation defined. There is a well known exploit called buffer overflow. Exceeding automatic storage size will cause stack overflow, which is a type of buffer overflow.
##Summary
Always prefer automatic storage to dynamic allocation. If free store allocation is needed, use dedicated containers or smart pointers. Always nail down ownership semantics. Use raw pointers to indicate no ownership.