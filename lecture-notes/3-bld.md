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
```

It also has a *rule* to build that target: `gcc main.c`. Rules are composed of
shell commands that are run in succession[^different-shells].

[^different-shells]: They are run in different shell processes, so state like
    variables and working directory do not persist between commands.

### Dependency relations

### The dependency graph

a DAG, just like Git uses!

### What happens when you type make

### Why is a Makefile better than a shell script?

Better performance and correctness
