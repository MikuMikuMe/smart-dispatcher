# smart-dispatcher

Certainly! Below is a complete Python program for a project called `smart-dispatcher`. This program is designed to efficiently allocate tasks to workers in a distributed team. It uses a simple task scheduling algorithm with error handling, and this setup can be further enhanced based on more specific requirements or integration with other systems.

First, let's outline what this basic version will do:
1. Distribute tasks to workers optimally to minimize downtime.
2. Support for adding new tasks and workers dynamically.
3. Basic load balancing by considering the current load on each worker.

Here's the code:

```python
import heapq
from typing import List, Dict, Any, Tuple

class Worker:
    def __init__(self, worker_id: int, capability: int):
        self.id = worker_id
        self.capability = capability  # Represents how many tasks a worker can handle concurrently
        self.current_tasks = 0  # Track the number of ongoing tasks

    def assign_task(self):
        if self.current_tasks < self.capability:
            self.current_tasks += 1
        else:
            raise Exception(f"Worker {self.id} is overburdened!")

    def complete_task(self):
        if self.current_tasks > 0:
            self.current_tasks -= 1

    def __lt__(self, other):
        return self.current_tasks < other.current_tasks  # For priority queue based on workload

    def __str__(self):
        return f"Worker {self.id} - Tasks: {self.current_tasks}/{self.capability}"


class SmartDispatcher:
    def __init__(self):
        self.workers: List[Worker] = []
        heapq.heapify(self.workers)  # Min-heap based on current tasks

    def add_worker(self, worker_id: int, capability: int):
        worker = Worker(worker_id, capability)
        heapq.heappush(self.workers, worker)
        print(f"Added {worker}")

    def add_task(self, task_id: int) -> Tuple[bool, str]:
        if not self.workers:
            return False, "No workers available"

        worker = heapq.heappop(self.workers)
        try:
            worker.assign_task()
            print(f"Task {task_id} assigned to {worker}")
            heapq.heappush(self.workers, worker)
            return True, f"Task {task_id} assigned to Worker {worker.id}"
        except Exception as ex:
            heapq.heappush(self.workers, worker)
            return False, str(ex)

    def complete_task(self, worker_id: int):
        for worker in self.workers:
            if worker.id == worker_id:
                worker.complete_task()
                heapq.heapify(self.workers)
                print(f"Task completed by Worker {worker_id}, current load: {worker.current_tasks}")
                return True
        return False

    def current_load(self):
        load_status = [(worker.id, worker.current_tasks) for worker in self.workers]
        for ws in load_status:
            print(f"Worker {ws[0]}: {ws[1]} tasks")
        return load_status

# Example Usage:
def main():
    dispatcher = SmartDispatcher()
    dispatcher.add_worker(worker_id=1, capability=3)
    dispatcher.add_worker(worker_id=2, capability=2)

    # Adding tasks
    print(dispatcher.add_task(task_id=101))
    print(dispatcher.add_task(task_id=102))
    print(dispatcher.add_task(task_id=103))
    print(dispatcher.add_task(task_id=104))  # This should go to worker 2, assuming previous tasks were allocated

    # Check current load
    dispatcher.current_load()

    # Complete tasks
    dispatcher.complete_task(worker_id=1)
    dispatcher.complete_task(worker_id=2)

    # Check load again
    dispatcher.current_load()

if __name__ == "__main__":
    main()
```

### Explanation

- **Worker Class**: Represents each worker with `id`, `capability` (how many tasks they can handle concurrently), and `current_tasks` (number of tasks currently assigned). It has methods to assign and complete tasks, and it implements a comparison method for sorting based on workload.

- **SmartDispatcher Class**: Manages the list of workers using a min-heap (`heapq`), which automatically sorts workers by their current tasks, favoring the least loaded worker. The `add_task` method pops a worker from the heap, attempts to assign a task, and re-inserts the worker. This ensures balanced task distribution.

- **Error Handling**: Error handling is done while assigning tasks: if a worker can't take more tasks, an exception is raised and caught to prevent that worker from being overloaded.

- **Example Usage**: Demonstrates adding workers and tasks, completing tasks, and outputting the current load.

This code is a simplified version. For real-world applications, consider integrating with databases or message queues for task management across distributed systems, and potentially implementing more complex scheduling logic.