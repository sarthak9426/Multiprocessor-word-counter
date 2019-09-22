/*************************************************
 * C program to count no of lines, words and *  
 * characters in a file with multiple child processes *
 *************************************************/

#include <stdio.h>
#include <sys/wait.h>
#include<stdlib.h>
#include<sys/types.h>
#include<unistd.h>

#define MAX_PROC 100
# define MAX_FORK 100
# define FILEPATH "Multiprocessor word counter";

typedef struct count_t {
  int linecount;
  int wordcount;
  int charcount;

}
count_t;

typedef struct plist_t {
  int pid;
  int offset;
  int pipefd[2];
}
plist_t;

count_t word_count(FILE * fp, long offset, long size) {
  char ch;
  long rbytes = 0;

  count_t count;
  // Initialize counter variables
  count.linecount = 0;
  count.wordcount = 0;
  count.charcount = 0;
  
  if (fseek(fp, offset, SEEK_SET) < 0) {
    printf("[pid %d] fseek error!\n", getpid());
  }

  while ((ch = getc(fp)) != EOF && rbytes < size) {
    // Increment character count if NOT new line or space
    if (ch != ' ' && ch != '\n') {
      ++count.charcount;
    }
    // Increment word count if new line or space character
    if (ch == ' ' || ch == '\n') {
      ++count.wordcount;
    }
    // Increment line count if new line character
    if (ch == '\n') {
      ++count.linecount;
    }
    rbytes++;
  }

  return count;
}

int main(int argc, char ** argv) {
  long fsize;
  FILE * fp;
  int numJobs;
  plist_t p[MAX_PROC];
  count_t total, count;
  int i, pid, status;
  int nFork = 0;

  if (argc < 3) {
    printf("Invalid input -->>Enter # of processes and filename \n");
    return 0;
  }

  numJobs = atoi(argv[1]);
  
  if (numJobs > MAX_PROC)
    numJobs = MAX_PROC;

  total.linecount = 0;
  total.wordcount = 0;
  total.charcount = 0;

  // Open file in read-only mode
  fp = fopen(argv[2], "r");
  if (fp == NULL) {
    printf("File open error: %s\n", argv[2]);
    return 0;
  }
  fseek(fp, 0L, SEEK_END);
  fsize = ftell(fp);
  fclose(fp);
  // calculate file offset and size to read for each child
  int offset = fsize / numJobs;
  int off[numJobs];
  off[0] = 0;
  for (int i = 1; i < numJobs; i++) {
    off[i] = off[i - 1] + offset;
  }
  for (i = 0; i < numJobs; i++) {
    //set pipe;
    if (nFork++ > MAX_FORK) return 0;
    // open the pipe for all the child process
    if (pipe(p[i].pipefd) == -1) {
      fprintf(stderr, "Pipe of child Failed");
      return 1;
    }

    p[i].pid = fork();
    if (p[i].pid < 0)
      printf("Fork failed.\n");
    else if (p[i].pid == 0) {
      // Child
      fp = fopen(argv[2], "r");
      
      if (i == numJobs - 1)
        count = word_count(fp, off[numJobs - 1], fsize - off[numJobs - 1]);
      else
        count = word_count(fp, off[i], off[i + 1] - off[i]);
      
      close(p[i].pipefd[0]);
      // send the result to the parent through pipe
      write(p[i].pipefd[1], & count, sizeof(struct count_t));
      
      sleep(5);
      close(p[i].pipefd[1]); // Close the writing end.
      
      fclose(fp);
      return 0;
    } 
    else continue;
  }

  wait(NULL);
  printf("counting file in %d processes..\n", numJobs);
  
  // Parent Read the result and write to total
  for (int i = 0; i < numJobs; i++) {
    close(p[i].pipefd[1]); // Close the write end of pipe
    read(p[i].pipefd[0], & count, sizeof(struct count_t));
    close(p[i].pipefd[0]); // Close the read end of pipe
    total.wordcount += count.wordcount;
    total.charcount += count.charcount;
    total.linecount += count.linecount;
  }

  printf("\n=========================================\n");
  printf("Total Lines : %d \n", total.linecount);
  printf("Total Words : %d \n", total.wordcount);
  printf("Total Characters : %d \n", total.charcount);
  printf("=========================================\n");

  return (0);
}