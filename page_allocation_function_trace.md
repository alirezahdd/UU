For page allocations, there are 2 functions in kernel. 
```c
alloc_page(gfp_mask)

```
The function trace for `alloc_pages`:  
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

list_first_entry(ptr, type, member)
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
  
