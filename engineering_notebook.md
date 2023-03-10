Design choice:

1. The LamportClock class is designed to provide a logical clock implementation with three methods: increment, update, and get_time. These methods are used to increment the clock value, update the clock with the maximum of other_time and the current time, and get the current clock time, respectively. A threading lock is used to ensure thread-safe access to the clock value.

2. The VirtualMachine class is designed to represent each virtual machine instance and contain all the logic required to run the simulation. It contains methods for connecting to other machines, sending messages, and running the virtual machine. It also contains a message queue and a log file for tracking the messages and events that occur during the simulation. Each VirtualMachine instance also has its own LamportClock instance with a random tick rate between 1 and 6.

3. The run method of the VirtualMachine class runs the main logic of the simulation. It starts by creating a list of two sockets connected to other machines and then enters an infinite loop. If there are messages in the message queue, it processes them by updating the local clock value with the maximum of other_time and the current time plus one, logging the received message, and removing the message from the queue. If there are no messages, it randomly selects one of four possible actions: sending a message to the first machine, sending a message to the second machine, sending messages to both machines with a random delay between them, or incrementing the local clock value and logging an internal event. The clock value is incremented after each action, and a wait method is called to sleep for a random amount of time based on the tick rate of the LamportClock instance.

4. The init_machine function is used to initialize the server-side logic for each VirtualMachine instance. It creates a socket and binds it to the specified port, listens for incoming connections, and starts a new thread to handle each incoming connection. When a new connection is accepted, it starts a new thread to receive messages on that connection and logs the connection.

5. The machine function is used to create and run a new VirtualMachine instance. It creates the instance with the specified configuration and ID, starts a new thread to initialize the server-side logic, and then starts a new thread to run the main logic of the simulation. A delay is added to allow time for the server-side logic to initialize before the main logic starts running.

6. For each process, we created a listening thread and sending thread. For each of these threads, we created a socket correpsoding to them. Therefore, in total, we have 6 threads and 6 sockets with a one to one mapping.

Debug process:

1. Our first bug was that the generated log files had no output available. We checked the threads for receiving messages and found that we haven't created threads for each machine to receive messages. Therefore, we added one listening thread for each process to constantly receive messages from other two processes.

2. Our second bug was that we had encounted assigned address already in use error when running our code. The fix turned out to be adding a clock wait after the while loop in the function run so that we gave the clock enough time for it to finish the designated event before requesting next connection.

3. Our third bug was that we found the logical clock time fluctuates in our log files which indicated that the ordering of our events was not sequential and did not follow partial ordering. Since we noticed that the fluctuations appeared in receiving messages, we checked our implementation for receiving messages and did the following two changes: First, we changed the logical clock time from time_val to self.clock.get_time() because we wanted to use the updated clock time as the final logical clock time shown on the log files. Second, we deleted the clock update in the physical receive of messages (append to the machine's queue). We only kept the clock update in the logical receive of messages (popping the message out of the queue)

4. Our fourth bug was that we would always create one socket per send_message by initializing a new connection the machine we wanted to send the message to and getting that connection. This turned out to be unnecessary and inefficient as we only needed one socket for each thread (either send or receive). Therefore, we created a bucket of sockets only once per run and used the corresponding sockets later by indexing into the previously created sockets list.

Discussions:

The size of jumps in the values for the logical clocks may vary depending on the clock rate of each machine and the frequency of message exchanges. In general, if the clock rate of a machine is faster (higher clock rate) than that of other machines, its logical clock values may jump more frequently and by fewer increments (+1 everytime). If the clock rate of a machine is lower (slower clock rate), its logical clock may jump less frequently but with larger increments. An interesting observation is that the slower machine continues to receive messages for a long time after initialization because based on the implementation specification, a machine cannot send unless it receives all messages sent to its queue. Additionally, if messages are exchanged frequently between machines, the logical clock values may be synchronized more often, resulting in smaller jumps.

The drift in the values of the local logical clocks may also vary depending on the clock rate of each machine. Machines with faster clock rates may experience greater drift because they are more likely to process more events in a given time period. 

The impact of different timings on the length of the message queue can also vary depending on the clock rates and message exchange patterns. If machines with slower clock rates (lower value) are unable to process messages as quickly as they are received, the message queue may grow longer over time. Additionally, if messages are exchanged infrequently, the message queue may not change much in length.

If the variation in clock cycles is smaller, it's likely that the jumps in the logical clock values will also be smaller. This is because events that occur out of order will likely have timestamps that are closer together, so the adjustments made by the logical clocks will also be smaller. Additionally, if the probability of internal events is smaller, there will be fewer internal events to log and fewer messages to write. This may result in a more predictable and stable system overall.

On the other hand, if the variation in clock cycles is larger and/or the probability of internal events is higher, it's likely that the system will be more prone to jumps in logical clock values and other inconsistencies. This is because there will be more opportunities for events to occur out of order or for machines to get out of sync with each other. This may make the system more complex and difficult to manage. 

Overall, the behavior of the distributed system modeled by the logical clocks will depend on a variety of factors, including the clock cycles of the individual machines, the probability of internal events, and the frequency and timing of message delivery. 
