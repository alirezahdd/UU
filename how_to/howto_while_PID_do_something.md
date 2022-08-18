## **How to make Shell Script while loop, where the condition is the existence of a PID**

```bash
./binary & PID=$! 

while kill -0 $PID >/dev/null 2>&1
do
#do something
done
```
