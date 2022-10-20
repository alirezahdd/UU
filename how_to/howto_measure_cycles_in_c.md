## **How to measure the number of cycles in C?** 
save and include the following header file as utils.h.   
```c
#ifndef _UTILS_H
#define _UTILS_H

#include <stdlib.h>
#include <string.h>
#include <stdio.h>
#include <sys/time.h>
#include <sys/mman.h>
#include <unistd.h>
#include <inttypes.h>

static __inline__ int64_t rdtsc_s(void)
{
  unsigned a, d;
  asm volatile("cpuid" ::: "%rax", "%rbx", "%rcx", "%rdx");
  asm volatile("rdtsc" : "=a" (a), "=d" (d));
  return ((unsigned long)a) | (((unsigned long)d) << 32);
}

static __inline__ int64_t rdtsc_e(void)
{
  unsigned a, d;
  asm volatile("rdtscp" : "=a" (a), "=d" (d));
  asm volatile("cpuid" ::: "%rax", "%rbx", "%rcx", "%rdx");
  return ((unsigned long)a) | (((unsigned long)d) << 32);
}

#endif
```  
use it in your program like this:
```c
int64_t start_time, end_time, exe_time;
start_time = rdtsc_s();
//YOUR CODE
end_time = rdtsc_e();
exe_time = end_time - start_time;
printf("The number of cycles spent on "YOUR CODE" is %d\n",exe_time);
```
