---
title: "Python Combine Multiple IOs"
date: 2023-05-14T15:06:32+08:00
draft: false
---
```CRUX: How to combine multiple IOs into single one?```

Scenario: In Python, imagine you need to have multiple processes using `subprocess` module and collect all the output streams (standard output/standard error) in a non-blocking way, how this can be acheived?

The idea of this solution is to get a queue to get the output from different IO sources by using daemon Thread, the queue then contains all the message. Thus there is a single source to handle multiple IOs into a single one.

So what is the code?

```python
import sys
from subprocess import PIPE, Popen
from threading  import Thread
from queue import Queue, Empty


# Basic function to enqueue IO output into a given queue
def enqueue_output(out, queue):
    for line in iter(out.readline, b''):
        queue.put(line)
    out.close()

process1 = Popen(
    " ".join(['cat', 'main.py', '&&', 'sleep', '0.01', '&&', 'cat', 'main.py']),
    stdout=PIPE,
    shell=True
)
process2 = Popen(
    ['ls', '-al'],
    stdout=PIPE
)

queue = Queue()

# Generate threads to enqueue the standard output to queue
process1_thread = Thread(target=enqueue_output, args=(process1.stdout, queue,))
process1_thread.daemon = True
process1_thread.start()

process2_thread = Thread(target=enqueue_output, args=(process2.stdout, queue,))
process2_thread.daemon = True
process2_thread.start()

# Print from the queue, a single source of mulitple processes standard output
combined_output = queue.get_nowait()
print(combined_output)
```

Reference: https://stackoverflow.com/a/4896288/8001553



Remark 1: In this example there is another problem occurs, that is multi-threading in Python code cannot pipe to queue and print from queue on time. This is yet to be investigated.
