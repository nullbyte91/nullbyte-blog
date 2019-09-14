---
layout: post
title:  "What are the valid signatures for c_main function - int main() or void main()"
categories: [ Embedded ]
image: assets/images/c_io_function.jpg
---

Neither main() or void main() are standard C. The former is allowed as it has an implicit intreturn value, making it the same as int main(). The purpose of main’s return value is to return an exit status to the operating system.

The current C standard C11 valid signatures for main are:

int main(void)

int main(int argc, char **argv)

The form you’re using: int main() is an old style declaration that indicates main takes an unspecified number of arguments. Don’t use it – choose one of those above.

Exit Status:

The exit status or return code of a process in computer programming is a small number passed from a child process (or callee) to a parent process (or caller) when it has finished executing a specific procedure or delegated task. In DOS, this may be referred to as an errorlevel.

The C programming language allows programs exiting or returning from the main function to signal success or failure by returning an integer, or returning the macros EXIT_SUCCESS and EXIT_FAILURE. On Unix-like systems these are equal to 0 and 1 respectively.