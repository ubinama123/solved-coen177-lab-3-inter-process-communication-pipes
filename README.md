Download Link: https://assignmentchef.com/product/solved-coen177-lab-3-inter-process-communication-pipes
<br>



<strong>Objectives</strong>

<h5>1.     To develop multi-process application programs</h5>

<h5>2.     To demonstrate the use of pipes as an inter-process communication (IPC) mechanism</h5>

<h5></h5>

<h5><strong>Guidelines</strong></h5>

<h5>In Lab2, you have learned to develop multi-processing and multi-threading applications using fork(), and pthread_create(), respectively. With fork() the kernel creates and initializes a new process control block (PCB) with a new address space that copies the entire content of the address space of the parent process. The new process is a child process and it inherits the execution context of the parent (e.g. open files). Unlike threads, processes do not share their address space. Therefore, processes use an IPC mechanism to exchange information. The kernel provides three IPCs, namely: Pipes, Shared Memory, and Message Queues.</h5>

<h5></h5>

<h5>Pipes are traditional Unix inter-process communication. The | symbol is used in the command line to denote a pipe. Try the following:</h5>

<h5>–       who | sort</h5>

<h5>–       cat /etc/passwd | grep root</h5>

<h5>Pipes cam be created in a C program using int pipe(int P[2]) system call, where:</h5>

<h5>–       P[1]: file descriptor on upstream end of pipe</h5>

<h5>–       P[0]: file descriptor on downstream end of pipe</h5>

<h5>Typically Process A (parent) creates a pipe , forks twice, creating B and C. Each process closes the ends of the pipe it does not need. Process B closes downstream end, process C closes upstream end, and process A closes both ends. Processes B and C execute other programs, using exec (file descriptors are retained). You may need to use int dup2(int OldFileDes, int NewFileDes); for redirection of the standard file descriptors, e.g. 0 (stdin), 1(stdout).</h5>




<h5>With shared memory, one process creates a shared segment using id = shmget( key, MSIZ, IPC_CREAT | 0666); system call, other process(es) get ID of the created segment id = shmget( key, MSIZ, 0 ). Using the same key, each process attaches to itself the created shared segment using  ctrl = (struct info *) shmat( id, 0, 0); , then process can be begin to read and write to the shared memory segment. At the end processes detached the segment.</h5>




<h5>With message queues, one process creates a message queue using id = msgget( key, IPC_CREAT | 0644); system call. Other process(es) get ID of the created segment using id = shmget( key,0 ) system call. Processes send a message to a queue and receive a message from a queue, using v = msgsnd( msid, ptr, length, IPC_NOWAIT) ; and v = msgrcv( msid,ptr,length,type,IPC_NOWAIT ); system calls, respectively.</h5>

<h5></h5>

<h5>In this lab, you will practice and use pipes for IPC.  Include the following libraries for IPC.</h5>

#include &lt;sys/types.h&gt;

#include &lt;sys/ipc.h&gt;




<strong>C Program with pipe IPC</strong>

Demonstrate each of the following steps to the TA to get a grade on this part of the lab assignment

<ul>

 <li>Compile and run the following program</li>

</ul>

/*Sample C program for Lab assignment 3*/

#include &lt;stdio.h&gt;

#include &lt;unistd.h&gt;

#include &lt;stdlib.h&gt;

#include &lt;sys/wait.h&gt;

//main

int main() {

int fds[2];

pipe(fds);

/*child 1 duplicates downstream into stdin */

if (fork() == 0) {

dup2(fds[0], 0);

close(fds[1]);

execlp(“sort”, “sort”, 0);

}

/*child 2 duplicates upstream into stdout */

else if (fork() == 0) {

dup2(fds[1], 1);

close(fds[0]);

execlp(“ls”, “ls”, 0);

}

else {  /*parent closes both ends and wait for children*/

close(fds[0]);

close(fds[1]);

wait(0);

wait(0);

}

return 0;

}




<ul>

 <li>Compile and run the following program</li>

</ul>

/*Sample C program for Lab assignment 3*/

#include &lt;stdio.h&gt;

#include &lt;unistd.h&gt;

#include &lt;stdlib.h&gt;

#include &lt;string.h&gt;

#include &lt;sys/wait.h&gt;

// main

int main(int argc,char *argv[]){

int  fds[2];

char buff[60];

int count;

int i;

pipe(fds);

if (fork()==0){

printf(“
Writer on the upstream end of the pipe -&gt; %d arguments 
”,argc);

close(fds[0]);

for(i=0;i&lt;argc;i++){

write(fds[1],argv[i],strlen(argv[i]));

}

exit(0);

}

else if(fork()==0){

printf(“
Reader on the downstream end of the pipe 
”);

close(fds[1]);

while((count=read(fds[0],buff,60))&gt;0){

for(i=0;i&lt;count;i++){

write(1,buff+i,1);

write(1,” “,1);

}

printf(“
”);

}

exit(0);

}

else{

close(fds[0]);

close(fds[1]);

wait(0);

wait(0);

}

return 0;

}







<ul>

 <li>Modify the program in Step 2. so that the writer process passes the output of “ls” command to the upstream end of the pipe. You may use dup2(fds[1],1); for redirection and execlp(“ls”, “ls”, 0); to run the “ls” command.</li>

</ul>




<ul>

 <li>Write a C program that implements the shell command: cat /etc/passwd | grep root</li>

</ul>




<strong> </strong>

<strong>Producer – consumer with pipes</strong>

<ul>

 <li>In Computer Science, the producer–consumer problem is a classic multi- process synchronization example. The producer and the consumer share a common fixed-size buffer. The producer puts messages to the buffer while the consumer removes messages from the buffer. Pipes provide a perfect solution to this problem because of their built-in synchronization capability. So as a programmer, you do not have to worry about whether or not the buffer is empty or full to produce or consume messages.</li>

</ul>




Write a C program to implement the producer-consumer message communication using pipes.