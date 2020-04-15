#Multiprocess Word-Counter with Crash Handling

wc.c is a simple program written in C to count the number of words, characters and lines in a text file.

The single process approach cannot scale to handle large files. Therefore, multiwc.c counts the number of words, characters and lines in a file using multiple child processes. This program receives the number of child processes and an input file name through the command-line argument.

multiwc_crash introduces crash handling to the multiprocessor word counter.  
