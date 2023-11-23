# 4-rtos - Inter-Process Communication w OS Linux
## 2 & 3
1) Code fragments responsible for the shared memory area:

E.g. Using the shm_open() function, a shared memory object is created, then using the mmap() function it is recommended to map the given object to the mmap memory - a system call instructing the operating system to map a given part of the selected file in the process address space. This operation allows the file area to be referenced as a regular array of bytes in memory, eliminating the need for additional read or write system calls. For this reason, this operation is often used to speed up operations on large files.

PROT_READ – pages can be read; PROT_WRITE – pages can be written.

MAP_SHARED - command to share mapping with all other processes that map this object. Saving this data in a given area will be equivalent to saving a file

![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/429723a8-2882-4832-be06-65e28be2b3a6)

Server_ShMem:
![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/ca773180-aa63-41d0-b64d-23dd6c789e39)

2) #define ByteSize 0x100000 - here you can allocate a larger memory size, e.g. by adding 0
When writing as an example: a) 10,000,000 and b) 1,000,000,000, analyzing the result of the top –H command:
For a smaller allocated memory size, the first console has lower usage (1282.1, and the second console has a larger usage of 1419.5)
A slightly higher percentage of CPU time is used for a larger amount of allocated memory, but you can see that subsequent activities need a percentage of less %Mem memory in relation to the total amount of memory.
For more memory in the second photo there is less free memory consumption and less buffering, as well as Avila MEM.

![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/2acf41a2-e976-4638-b142-c2fd079f4da9)
![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/d697f2b7-4f0f-4387-84aa-d9ccf1f98300)

Processes have different virtual indicators:
![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/974ec0e7-3ab6-4b27-a79b-30f907a5eb7e)
![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/a93aea4f-7ce5-49b2-abba-80de9160830f)
[image](https://github.com/AsiaEwa/4-rtos/assets/101841759/0e2a49ee-aa45-4165-aad6-fa0f0b82c4a0)
![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/a3333413-94ac-49cc-9293-fb6031139626)

4) Cannot access memory from different privilege groups, message appears: Can't open shared mem segment ...: Permission denied

## 2.2.3
1) Functions that create the communication system in the server file: 
![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/45aa7a62-6006-4a94-93f7-198ff13622df)
2) The semaphore name is located in the namespace
![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/79ee3126-e7c9-43d2-a739-aa606aabfc18)

3) Message queues are special lists (queues) in the kernel, containing appropriately formatted data and enabling their exchange by any processes in the system.
Semaphores are created in the kernel address space and accessed via an appropriate call.
One msqid_ds structure is responsible for each message queue. The messages of a given queue are stored on a list whose elements are msg structures - each of them has information about the message type, a pointer to the next msg structure and a pointer to the place in memory where the actual message content is stored. Additionally, each message queue is allocated two wait_queue queues, on which processes suspended while performing reading or writing operations to a given queue sleep.
msg_perm: This is an instance of the ipc_perm structure, defined in the linux/ipc.h file. Contains information about access rights to a given queue and its creator.
msg_type: The type of the message stored. The sender assigns e.g. a positive natural number to the message sent to the queue.

4) When both sides are run as root, the program works correctly as intended (otherwise, without root privileges when starting the client "./Client" working... Segmentation fault - i.e. terminal returns no access to start the process.
Theoretically, messages can be transferred between processes under certain assumptions.
There is a pool of N units of a given type of resource in the system. Processes can take resources from the pool and return them. When there is no resource and a certain process tries to download it, it is blocked. A process will be unblocked when another process returns a resource unit.
1. There are two groups of processes in the system - producers and consumers, and a buffer for elements.
2. Manufacturers produce certain items and place them in a buffer.
3. Consumers retrieve items from the buffer and consume them.
4. Producers and consumers become active at undeterminable moments in time.
5. The buffer has a capacity for N elements.

#### Correctly organized system operation:
1. When there is free space in the buffer, the producer can place his element there. When there is no space in the buffer for elements, the manufacturer must wait. When vacancies appear, the producer will be unblocked.
2. When there are some elements in the buffer, the consumer downloads them. When there are no items in the buffer, the consumer must wait. When an item appears, the consumer will be unlocked

The buffer can be organized on various principles:
1. FIFO queue (cyclic buffer).
2. LIFO queue (stack).
Placing/retrieving an item from the buffer is a critical section.
5) We use a named semaphore to cooperate (synchronize work) between two different processes: the producer (sender-creator) and the customer (recipient). Thanks to the name, it is easier to identify and independence of further dependent processes. When using a name, there is no need to place the semaphore in shared memory. Named semaphores are better for unrelated processes, e.g. producer and consumer. They may be written by two different developers and run as completely unrelated processes. However, they must share a certain resource, which must be protected by a semaphore. A named semaphore gives them a path to the semaphore.
Unnamed POSIX semaphores can be used by unrelated processes. Simply store the data structure in shared memory available to the appropriate processes and initialize it with a shared flag set to one.
Another process mapping shared memory to its address space can access the same semaphore and synchronize with the original process.
Typically, an unnamed semaphore is used for a thread belonging to the same process. A named semaphore can be shared with multiple unrelated processes.
![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/6f7837b0-011a-42af-a03b-a3c9e9fbef54)
![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/e7ca7d76-cc60-4f9b-8915-7da4765be069)

## 2.3.2
1) Functions of individual signals
SIGCONT – instructs the operating system to continue (restart) a process stopped by SIGSTOP or SIGTSTP signals.
SIGTSTP - this signal is sent to the process by the terminal controlling it in order to order it to stop.
SIGFPE - a signal sent to the process at the time of an unusual event (not necessarily incorrect), e.g. Division by 0. 2) Sending signals

![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/bc6bc0a0-31d6-43a3-a8b4-64504b95d508)
![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/2f954aa2-2c8a-4cda-a6c2-a00540030f21)
![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/d5be0d35-d3ca-4990-8142-739f59bf7bbb)
![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/0d2feb43-119d-4681-876b-a62d89dc8228)
![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/527eb8ab-7bcf-4221-8687-cbd73d8e1596)
![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/9235d40f-077e-4da2-8ada-aef2ee17463c)

a) The program stops after sending the SIGTSTP signal b) After sending the SIGCONT signal, the program resumes operation 3. Check the program operation with the top -H command:
During operation, the program assumes values in bold (highlighted). This is due to its continuous operation, it reaches the divisor value equal to 1 (because this is how its operation was written, hence the following system response) and then the values v1 and v2 return to the original value.
![image](https://github.com/AsiaEwa/4-rtos/assets/101841759/1a2ce2bd-bc32-486a-95e6-6e15d76d85d7)

4) Division by 0
When dividing by 0, the program receives the SIGFPE signal and then restarts.
Signal SIGFPE is responsible for handling the division error, which allows the program to execute (when v1 and v2 ==0, they are to return to the initial values) again from the beginning (hence the "looped" restart)




