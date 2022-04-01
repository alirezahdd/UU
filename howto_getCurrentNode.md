## **How to get current CPU and node?** 
To get current CPU and current node use the following example:   
```c
#include <stdio.h>
#include <utmpx.h>

int main(void) {
  int cpu = sched_getcpu();
  int node = numa_node_of_cpu(cpu);
  printf("CPU: %d , node: %d \n", cpu, node);
  return 0;
}
```  
the above code should be compiled using `-lnuma` flag
```bash
gcc test.c -lnuma
./a.out
```
