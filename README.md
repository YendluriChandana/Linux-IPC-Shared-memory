# Ex06-Linux IPC-Shared-memory

# AIM:
To Write a C program that illustrates two processes communicating using shared memory.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Shared Memory

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that illustrates two processes communicating using shared memory.
### shmry1.c
```
#include <unistd.h> 
#include <stdlib.h> 
#include <stdio.h> 
#include <string.h> 
#include <sys/shm.h>

#define TEXT_SZ 2048 

struct shared_use_st {
    int written_by_you;
    char some_text[TEXT_SZ];
};

int main() {
    int running = 1;
    void *shared_memory = (void *)0; 
    struct shared_use_st *shared_stuff; 
    int shmid;
    
    srand((unsigned int)getpid()); 
    
    shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666 | IPC_CREAT);
    printf("Shared memory id is %d \n", shmid);
    
    if (shmid == -1) {
        fprintf(stderr, "shmget failed\n");
        exit(EXIT_FAILURE);
    }
    
    shared_memory = shmat(shmid, (void *)0, 0);
    if (shared_memory == (void *)-1) {
        fprintf(stderr, "shmat failed\n");
        exit(EXIT_FAILURE);
    }
    
    printf("Memory Attached at %p\n", shared_memory);
    shared_stuff = (struct shared_use_st *)shared_memory;
    shared_stuff->written_by_you = 0;

    while (running) {
        if (shared_stuff->written_by_you) {
            printf("You Wrote: %s", shared_stuff->some_text);
            sleep(rand() % 4);
            shared_stuff->written_by_you = 0;
            if (strncmp(shared_stuff->some_text, "end", 3) == 0) {
                running = 0;
            }
        }
    }

    if (shmdt(shared_memory) == -1) {
        fprintf(stderr, "shmdt failed\n");
        exit(EXIT_FAILURE);
    }

    if (shmctl(shmid, IPC_RMID, 0) == -1) {
        fprintf(stderr, "failed to delete\n");
        exit(EXIT_FAILURE);
    }

    exit(EXIT_SUCCESS);
}

```
### shmry2.c
```
#include <unistd.h> 
#include <stdlib.h> 
#include <stdio.h> 
#include <string.h>
#include <sys/shm.h>

#define TEXT_SZ 2048 

struct shared_use_st {
    int written_by_you;
    char some_text[TEXT_SZ];
};

int main() {
    int running = 1;
    void *shared_memory = (void *)0; 
    struct shared_use_st *shared_stuff; 
    char buffer[BUFSIZ];
    int shmid;
    
    shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666 | IPC_CREAT);
    printf("Shared memory id = %d \n", shmid);
    
    if (shmid == -1) {
        fprintf(stderr, "shmget failed\n");
        exit(EXIT_FAILURE);
    }
    
    shared_memory = shmat(shmid, (void *)0, 0);
    if (shared_memory == (void *)-1) {
        fprintf(stderr, "shmat failed\n");
        exit(EXIT_FAILURE);
    }
    
    printf("Memory Attached at %p\n", shared_memory); 
    shared_stuff = (struct shared_use_st *)shared_memory; 
    
    while (running) {
        while (shared_stuff->written_by_you == 1) {
            sleep(1);
            printf("waiting for client.\n");
        }
        
        printf("Enter Some Text: ");
        fgets(buffer, BUFSIZ, stdin);
        strncpy(shared_stuff->some_text, buffer, TEXT_SZ);
        shared_stuff->written_by_you = 1;
        
        if (strncmp(buffer, "end", 3) == 0) {
            running = 0;
        }
    }

    if (shmdt(shared_memory) == -1) {
        fprintf(stderr, "shmdt failed\n");
        exit(EXIT_FAILURE);
    }

    exit(EXIT_SUCCESS);
}

```
## OUTPUT
### $ ./shm2.o
```
darshanig@Darshani-laptop:~$ ./shm2.o
Shared memory id = 0
Memory Attached at f727d000
Enter Some Text: hello
waiting for client.
waiting for client.
Enter Some Text: hi
waiting for client.
waiting for client.
waiting for client.
Enter Some Text: operating systems
waiting for client.
waiting for client.
```
### $ ./shm1.o
```
darshanig@Darshani-laptop:~$ ./shm1.o
Shared memory id is 0
Memory Attached at 48d89000
You Wrote: hello
You Wrote: hi
You Wrote: operating systems
```
### $ ipcs
```
darshanig@Darshani-laptop:~$ ipcs

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
0x000004d2 0          darshanig  666        2052       1


------ Semaphore Arrays --------
key        semid      owner      perms      nsems
```

# RESULT:
The program is executed successfully.
