/*************************************************
	* C program to count no of lines, words and 	 *
	* characters in a file.			 *
	*************************************************/

#include <stdio.h>

typedef struct count_t {
	int linecount;
	int wordcount;
	int charcount;
} count_t;

count_t word_count(FILE* fp, long offset, long size)
{
	char ch;
	long rbytes = 0;
	count_t count;
	count.linecount = 0;
	count.wordcount = 0;
	count.charcount = 0;
	if(fseek(fp, offset, SEEK_SET) < 0) {
			printf("fseek error!\n");
	}

	while ((ch=getc(fp)) != EOF && rbytes < size) {
		// Increment character count if NOT new line or space
		if (ch != ' ' && ch != '\n') { ++count.charcount; }
		// Increment word count if new line or space character
		if (ch == ' ' || ch == '\n') { ++count.wordcount; }
		// Increment line count if new line character
		if (ch == '\n') { ++count.linecount; }
		rbytes++;
	}
	return count;
}

int main(int argc, char **argv)
{
	long fsize;
	FILE *fp;
	count_t count;

	if(argc < 2) {
		printf("Filename not provided Exiting\n");
		return 0;
	}
	
	count.linecount = 0;
	count.wordcount = 0;
	count.charcount = 0;
	// Open file in read-only mode
	fp = fopen(argv[1], "r");
	if(fp == NULL) {
		printf("File open error: %s\n", argv[1]);
		printf("usage: wc <filname>\n");
		return 0;
	}

	fseek(fp, 0L, SEEK_END);
	fsize = ftell(fp);

	count = word_count(fp, 0, fsize);
	fclose(fp);

	printf("\n=========================================\n");
	printf("Total Lines : %d \n", count.linecount);
	printf("Total Words : %d \n", count.wordcount);
	printf("Total Characters : %d \n", count.charcount);
	printf("=========================================\n");
	
	return(0);
}