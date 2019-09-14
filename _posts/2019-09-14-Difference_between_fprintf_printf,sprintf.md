---
layout: post
title:  "Difference between, fprintf, printf and sprintf"
categories: [ Embedded ]
image: assets/images/c_io_function.jpg
---

<b>Printf</b> – writes output to the standard output stream(stdout).

<b>Syntax</b>:
int printf(const char* str, …);

```python
Example:
// Example program to demonstrate printf()
#include<stdio.h>
int main()
{
printf(“hello Innovation Begins Here\n”);
return 0;
}
```
O/P: hello Innovation Begins Here
Man page: printf(3): formatted output conversion

Sprintf   –   writes output to a buffer that you allocate(char*).

Syntax:
int sprintf(char *str, const char *string,…);
String print function it is stead of printing on console store it on char buffer which are specified in sprint.

```python
Example:
// Example program to demonstrate sprintf()
#include<stdio.h>
int main()
{
char buffer[10];
int a = 1, b = 2, c;
c = a + b;
sprintf(buffer, “Sum of %d and %d is %d”, a, b, c);
// The string “sum of 1 and 2 is 3” is stored
// into buffer instead of printing on stdout
printf(“%s”, buffer);
return 0;
}
```

O/P: Sum of 10 and 20 is 30
Man page: sprintf(3): formatted output conversion

Fprintf  –  writes output to a file handle (FILE *).

Syntax:
int fprintf(FILE *fptr, const char *str, …);

```python
Example:
#include<stdio.h>
int main()
{
int i, n=1;
char str[50];
//open file sample.txt in write mode
FILE *fptr = fopen(“sample.txt”, “w”);
if (fptr == NULL)
{
printf(“Could not open file”);
return 0;
}
for (i=0; i<n; i++)
{
puts(“Enter a name”);
gets(str);
fprintf(fptr,”%d.%s\n”, i, str);
}
fclose(fptr);
return 0;
}
```
O/P: Input  from your keyboard Innovation Begins Here
Output: sample.txt file now having output as
Innovation Begins Here
Man page: fprintf(3): formatted output conversion