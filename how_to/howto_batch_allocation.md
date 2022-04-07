Below you can find an interface for the _batch allocator_ in linux.
```c
unsigned long alloc_pages_bulk_array(gfp_t gfp, unsigned long nr_pages, struct page **page_array)
```
The interface, takes `GFP` (Get Free Pages) flags. These flags tells the allocator what can and can't be done while allocating.
A list of GFP flags can be found under [/include/linux/gfp.h](https://elixir.bootlin.com/linux/v5.14.1/source/include/linux/gfp.h#L323).  
It also takes `nr_pages` (number of pages to allocate), and `page_array` (a page array) to fill.
  
Below you can find a system call that uses mentioned interface.
```c
#include <linux/kernel.h>
#include <linux/syscalls.h>
#include <linux/gfp.h>

SYSCALL_DEFINE1(balloc, int , nr_pages)
{
  struct page * page_array[1024];
  
  //batch allocation
  unsigned long allocated = alloc_pages_bulk_array(GFP_USER,nr_pages,page_array);
  
  /*//allocating page one by one
  int i;
  for (i=0; i<nr_pages; i++){
    page_array[i] = alloc_page(GFP_USER);
  }
  */
  
  return 0;
}
```
