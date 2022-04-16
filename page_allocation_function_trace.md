Page allocation in linux has many interfaces, among which two of them are simpler, `alloc_page()` and `alloc_pages_bulk_array()`.
These function try to fetch pages from [Per CPU Pages (PCP) lists](https://elixir.bootlin.com/linux/latest/source/include/linux/mmzone.h#L369), lists consist of free pages dedicated to each cpu. Find [more info about pcp list here](https://lwn.net/Articles/884448/). If pcp lists are empty, the kernel refills them first and fetch pages from them again. 
In this tutorial we are going to trace kernel functions for:
* [Single Page Allocation](#the-function-trace-for-alloc_page)
* [Bulk Page Allocation](#the-function-trace-for-alloc_pages_bulk_array)
* [Filling Up PCP List](#the-function-trace-for-filling-up-pcp-list)  
 
For page allocations, there are 2 functions in kernel. 
```c
alloc_page(gfp_t gfp_mask);
//or
alloc_pages_bulk_array(gfp_t gfp, unsigned long nr_pages, struct page **page_array);
```
## The Function Trace For `alloc_page()`:  
```c
struct page *alloc_page(gfp_mask);
struct page *alloc_pages(gfp_t gfp, unsigned int order);
static inline struct page *alloc_pages_node(int nid, gfp_t gfp_mask, unsigned int order);
static inline struct page *__alloc_pages_node(int nid, gfp_t gfp_mask, unsigned int order);
struct page *__alloc_pages(gfp_t gfp, unsigned int order, int preferred_nid, nodemask_t *nodemask);
```
the `__alloc_pages` returns a page in either of two following ways:
```c
//first attempt

static struct page *
get_page_from_freelist(gfp_t gfp_mask, unsigned int order, int alloc_flags, const struct alloc_context *ac);
```
or
```c
//second attempt

static inline struct page *
__alloc_pages_slowpath(gfp_t gfp_mask, unsigned int order, struct alloc_context *ac);

//after calling some functions it ends up to the get_page_from _freelist()
```
so we now continue with the `get_page_from_freelist()`.
more info about [__alloc_pages_slowpath()](https://elixir.bootlin.com/linux/latest/source/mm/page_alloc.c#L4867).
```c
static struct page *
get_page_from_freelist(gfp_t gfp_mask, unsigned int order, int alloc_flags, const struct alloc_context *ac);

struct page *rmqueue(struct zone *preferred_zone, struct zone *zone, unsigned int order,
			gfp_t gfp_flags, unsigned int alloc_flags, int migratetype);
```
`rmqueue()` returns a page in either of two following ways:
```c
struct page *rmqueue(struct zone *preferred_zone, struct zone *zone, unsigned int order,
			gfp_t gfp_flags, unsigned int alloc_flags, int migratetype);

static struct page *rmqueue_pcplist(struct zone *preferred_zone,
			struct zone *zone, unsigned int order,
			gfp_t gfp_flags, int migratetype,
			unsigned int alloc_flags);

//this function returns the first entry of the pcp list
//this is where pcp os checked and if it is empty it get filled.
struct page *__rmqueue_pcplist(struct zone *zone, unsigned int order,
			int migratetype, unsigned int alloc_flags, struct per_cpu_pages *pcp,
			struct list_head *list);

list_first_entry(ptr, type, member);
```
or
```c
struct page *rmqueue(struct zone *preferred_zone, struct zone *zone, unsigned int order,
			gfp_t gfp_flags, unsigned int alloc_flags, int migratetype);

static __always_inline
struct page *__rmqueue_smallest(struct zone *zone, unsigned int order,
						int migratetype);
  
static inline struct page *get_page_from_free_area(struct free_area *area,
					    int migratetype);
  
list_first_entry_or_null(ptr, type, member);
```
## The Function Trace For `alloc_pages_bulk_array()`:
```c
static inline unsigned long
alloc_pages_bulk_array(gfp_t gfp, unsigned long nr_pages, struct page **page_array);

unsigned long __alloc_pages_bulk(gfp_t gfp, int preferred_nid,
			nodemask_t *nodemask, int nr_pages,
			struct list_head *page_list,
			struct page **page_array);
			
//following function is used in a loop for nr_pages iteration			
struct page *__rmqueue_pcplist(struct zone *zone, unsigned int order,
			int migratetype,
			unsigned int alloc_flags,
			struct per_cpu_pages *pcp,
			struct list_head *list)

list_first_entry(ptr, type, member);
```
## The Function Trace For Filling Up PCP List:
[This](https://elixir.bootlin.com/linux/latest/source/mm/page_alloc.c#L3632) is where an empty pcp list is getting filled.
```c
/*
 * Obtain a specified number of elements from the buddy allocator, all under
 * a single hold of the lock, for efficiency.  Add them to the supplied list.
 * Returns the number of new pages which were placed at *list.
 */
static int rmqueue_bulk(struct zone *zone, unsigned int order,
			unsigned long count, struct list_head *list,
			int migratetype, unsigned int alloc_flags);
			
static __always_inline struct page *
__rmqueue(struct zone *zone, unsigned int order, int migratetype,
						unsigned int alloc_flags)

static __always_inline struct page *__rmqueue_cma_fallback(struct zone *zone,
					unsigned int order);
					
/*
 * Go through the free lists for the given migratetype and remove
 * the smallest available page from the freelists
 */
static __always_inline
struct page *__rmqueue_smallest(struct zone *zone, unsigned int order,
						int migratetype);
						
static inline struct page *get_page_from_free_area(struct free_area *area,
					    int migratetype);
					    
list_first_entry_or_null(ptr, type, member);
```
