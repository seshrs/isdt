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

In order to instruct Make to rebuild `mybinary` when `main.c` is modified, add
`main.c` as a *dependency* of `mybinary`:

```
mybinary: main.c
    gcc main.c
    mv a.out mybinary
```

Now Make will look at the <dfn><abbr title="modification time">m-time</abbr></dfn>
of both `main.c` and `mybinary`. If `main.c` is newer, it will rebuild
`mybinary`.

It is possible to specify more than one dependency per target. If `main.c` also
included a header, `myheader.h`, you should add `myheader.h` to the
dependencies for `main.c`. Otherwise the binary and the header will be out of
sync. For libraries use headers to specify interfaces, this is bad. At best,
you might get a wrong answer. At worst, a crash or memory corruption.

### Notes on split compilation (object files)

Maybe notes on Rust here?

### The dependency graph

a DAG, just like Git uses!

### What happens when you type make

### Why is a Makefile better than a shell script?

As demonstrated, it's possible to build your projects using a simple shell
script that builds every file every time. This is a fine solution, to a point;
if the number of files or the compilation time for individual files grows, your
build script will get slower over time.

You might get fancy and add some features to compare m-times in your shell
script. Maybe you add a function called `build_if_newer` and get 60% of the
functionality of Make. This will work. But now you have to reason about an
ever-growing shell script and if it perfectly implements your ideal build
semantics. And Make will do it better still.

Make has been around a long time and its performance and correctness are
well understood. Its interface is well understood, too; people who know Make
are at home in most other projects that use Make. Not so for a custom shell
script.
