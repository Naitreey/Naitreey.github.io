---
layout: post
title: Two Simple-minded Utility Programs: catf and wcf
tags: [C, Linux]
---

I wrote two very simple programs (oh please, if you really call them that way.) as an exercise for my ongoing C programming study. One is my version of `cat` and another is `wc`.
`cat` and `wc` are simple command-line utilities shipped with Linux. 

`cat` can be used to concatenate (namely `cat`) files and print them to the standard output (normally, the screen). And `wc` is a word-counting program.
IMHO, my version of `cat` and `wc` are slightly "better" in functionalities. To distinguish with the original ones, I will suffix them with letter 'f' ('f' for "fake"), namely `catf` and `wcf`.

## catf ##

`catf` supports 4 forms of syntax:

1. `catf`
input from `stdin` (line-buffered), output to `stdout`.
2. `catf file1.in file2.in ...`
concatenate `*.in` files and output to `stdout`.
3. `catf -o file.out`
input from `stdin` and output to `file.out`.
4. `catf -o file.out file1.in file2.in ...`
input from `*.in` files and output to `file.out`.

Isn't that surreal?
Notice that, using `catf`, not only can you output concatenated files to `stdout`, you can also output them to another file. (Oh yeah, some REAL improvement.)

Without further description, I'll simply paste the source code. (Please forgive the naïveté of an incompetent C programmer. Any harsh criticism is welcomed.) 

```c
/* a simple `cat` program
 * aim:
 * 1. catf
 * 2. catf file_in1 file_in2 ...
 * 3. catf -o file_out
 * 4. catf -o file_out file_in1 file_in2 ...
 */

#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define LEN 4096

//read in the content of stream `in`, write to stream `out`.
void ReadAndWriteFile(FILE * in, FILE * out);
//close stream `pf`. the `name` of file is passed to assist printing error message
void CloseFile(FILE * pf, char * name);

int main(int argc, char *argv[])
{
	FILE *in, *out;			//in/out file pointer
	int i;

	if( argc==1 )			//func 1: if only `ccat` is given, read in from `stdin` and print out to `stdout`.
	{
		ReadAndWriteFile(stdin, stdout);
	}
	else if( argv[1][0]=='-' )	//func 3,4: if `-` arg is given, prepare a file for output
	{
		if( strcmp(argv[1], "-o")!=0 )	//check validity
		{
			fprintf(stderr, "%s: argument '%s' can not be recognized.\n" , argv[0], argv[1]);
			exit(EXIT_FAILURE);
		}

		if( (out=fopen(argv[2], "a"))==NULL )	//open the output file in `append` mode
		{
			fprintf(stderr, "can not open file '%s' for appending.\n" , argv[2]);
			exit(EXIT_FAILURE);
		}

		if( argc==3 )		//func 3: if no input file, read from stdin
		{
			ReadAndWriteFile(stdin, out);
			CloseFile(out, argv[2]);
		}
		else			//func 4: otherwise read in every input file and write its content to output file
		{
			for( i=3 ; i<argc ; i++ )
			{
				if( (in=fopen(argv[i], "r"))==NULL )	//open every input file in `read` mode
				{
					fprintf(stderr, "can not open file '%s' for reading.\n" , argv[i]);
					exit(EXIT_FAILURE);
				}

				ReadAndWriteFile(in, out);
				CloseFile(in, argv[i]);
			}
			CloseFile(out, argv[2]);
		}
	}
	else				//func 2: if no output file, open input files for reading and write to stdout
	{
		for( i=1 ; i<argc ; i++ )
		{
			if( (in=fopen(argv[i], "r"))==NULL )
			{
				fprintf(stderr, "can not open file '%s' for reading.\n" , argv[i]);
				exit(EXIT_FAILURE);
			}

			ReadAndWriteFile(in, stdout);
			CloseFile(in, argv[i]);
		}
	}

	return 0;
}

void CloseFile(FILE * pf, char * name)
{
	if( fclose(pf)!=0 )
	{
		fprintf(stderr, "file '%s' didn't closed successfully.\n" , name);
		exit(EXIT_FAILURE);
	}
}

void ReadAndWriteFile(FILE * in, FILE * out)
{
	static char buffer[LEN];
	while( fgets(buffer, LEN, in)!=NULL )
	{
		fputs(buffer, out);
	}
}
```

## wcf ##

`wcf` supports two forms of syntax:
1. `wcf`
input from stdin, output to stdout.
2. `wcf file.in`
input from `file.in`, output to stdout.

I had a hard time to design a word-count program with both efficiency and a clean flow of logic. That said, I do not like my version at all.

Here is the source code.

```c
#include <stdio.h>
#include <stdlib.h>
#include <ctype.h>

int main(int argc, char **argv)
{
	FILE *file;			//file stream to be counted
	int cc=0, wc=0, lc=0;		//char, word, line count
	char ch;			//store input char
	
	if( argc<2 )			//if no input file, use stdin
	{
		file=stdin;
	}
	else if( (file=fopen(argv[1], "r"))==NULL)	//if there is input file, open it
	{
		fprintf(stderr, "file %s open error.\n" , argv[1]);
		exit(EXIT_FAILURE);
	}
	//cc. every byte counts for a character
	//wc. every alpha-numeric combination counts for a word
	//lc. every '\n' indicates a complete line
	while ( (ch=getc(file))!=EOF )
	{
		if( isalnum(ch) )			//idea is: encountering an alnum char indicates entering a word, wc++,
		{					//then characters of the word is counted by inner loop
			wc++;				//other characters are counted by outer loop
		}					//lines and EOF are checked by outer loop
		while( isalnum(ch) )
		{
			cc++;
			ch=getc(file);
		}
		if( ch=='\n' )
		{
			lc++;
		}
		if( ch==EOF )
			break;
		cc++;
	}
	fclose(file);
	printf( "characters: %d\nwords: %d\nlines: %d\n" , cc, wc, lc );

	return 0;
}
```
    
## Afterthought ##

I, the creator of these pieces of software, use `catf` and `wcf` _all the time_. But, I would still gentlely advise anyone as sane as me, to use the version that is shipped with their distribution. And you can get their source code via [GNU's ftp][coreutils].

[coreutils]: http://ftp.gnu.org/gnu/coreutils/

That's it. Life well wasted.
