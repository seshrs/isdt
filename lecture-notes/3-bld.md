# Lecture 2

### Targets and rules

*Targets* are the core of a Makefile. They are the results you need to
eventually build; the names of the files on disk you want to see by the end of
the build process[^phony]. In the example below, a Makefile has a target called
`mybinary` -- the name preceding the colon (`:`).

[^phony]: Not always; sometimes there are "phony" targets that are names for
    batches of rules and do not produce files.

```
mybinary:
    gcc main.c
    mv a.out mybinary
```

It also has *rules* to build that target: `gcc main.c`. The rules section
consists of a list of shell commands that are run in
succession[^different-shells] to build the target. In this case, the two rules
-- `gcc main.c` and `mv a.out mybinary` will run one after another.

[^different-shells]: They are run in different shell processes, so state like
    variables and working directory do not persist between commands.

This is a bit of a silly example since `gcc` has a `-o` option to specify the
output filename, but there are certainly situations where multiple shell
commands might be necessary.

### Dependency relations

Let's look again at the Makefile from above:

```
mybinary:
    gcc main.c
    mv a.out mybinary
```

We can run this once with `make mybinary` and then we will have a `mybinary` on
disk. During the course of development, though, we will likely make a change to
`main.c` and try to rebuild with `make mybinary`. Make will give an unhelpful
response and do nothing:

```
$ make mybinary
gcc main.c
mv a.out mybinary
$ vim main.c
...
$ make mybinary
make: 'mybinary' is up to date.
$
```

This is because there is no way for Make to know about the implicit
relationship between `mybinary` and `main.c`. As far as Make is concerned, the
contents of the rules are opaque shell commands; it does not attempt to
introspect on them, nor does it have an innate idea of what `gcc` means. You,
the programmer, have to specify the relationship manually.

### A more complicated example

Imagine you have the following C file with a declaration for the function
`myfunction`:

```c
// main.c
#include <stdio.h>

int myfunction();

int main() {
  printf("The result is %d\n", myfunction());
  return 0;
}
```

and corresponding library with the *definition* for `myfunction`:

```c
// mylibrary.c
int myfunction() {
  return 4;
}
```

If you have a Makefile for this

### The dependency graph

a DAG, just like Git uses!

### What happens when you type make

### Why is a Makefile better than a shell script?

Better performance and correctness
