# experiment 1.2
![alt text](experiment1.2.png)
The output order is due to how the executor and task scheduling work in async runtime:

spawner.spawn(async { ... })
This schedules the async block (which prints "howdy!", waits 2 seconds, then prints "done!") to run on the executor.
However, it does not run immediately; it just queues the task.

println!("Joseph's Computer: hey hey")
This line runs immediately in the main thread, before the executor starts running tasks.
So "Joseph's Computer: hey hey" is printed first.

executor.run()
Now the executor starts running tasks from the queue:

It picks up the spawned async task.
The task prints "Joseph's Computer: howdy!".
Then it awaits the timer (TimerFuture::new(Duration::new(2, 0)).await;).
At this point, the future is not ready, so the executor pauses this task and waits.
Timer completes after 2 seconds
The task is woken up and scheduled again.

The executor resumes the task.
The task prints "Joseph's Computer: done!".

# experiment 1.3
![alt text](image.png)
![alt text](image-1.png)
![alt text](image-2.png)

based on first and second image, the done message appear in random order. When i comment the drop(spawner), as can be seen in third image, the process never stops.

- Why does the order of done!, done2!, done3! change every run?
All three tasks (howdy!, howdy2!, howdy3!) are spawned and start almost at the same time.
Each prints its howdy! message, then awaits a timer for 2 seconds.
After 2 seconds, all timers complete around the same time. The executor wakes up all three tasks and puts them back in the queue.
The executor then polls the ready tasks in the order they arrive in the queue.
But because timers use threads and channels internally, the order in which the tasks are woken up and re-queued is not deterministic.
So, the order of "done!", "done2!", "done3!" is random and can change every run.

- Why does commenting out drop(spawner); cause the program to never stop?
The executor’s run() method keeps running as long as it can receive tasks from the channel.
The channel (sync_channel) is only closed when all senders (the Spawner and its clones) are dropped.
If I don’t drop the spawner, the channel stays open, and executor.run() waits forever for new tasks, even after all tasks are done.
That’s why I have to press Ctrl+C to stop it.