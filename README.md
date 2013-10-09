## This project is to demonstrate a bug with Gradle and/or JVM dynamic compilation

The problem is that `javac` does not fail the build when compiling in UTF-8 mode and encountering a class with non-UTF-8 characters.

* Prerequisites: Gradle, Java 7

To demonstrate the problem, run:

```bash
$ gradle clean build
```

There are various configurations commented out in `build.gradle` that I've attempted.  The only one that fails the build is to compile in fork mode.

### JVM dynamic compilation

In this project, I tried to create a simple class to run the dynamic compilation in a way similar to how Gradle does it.  To see this in action, try the commands below.  The output should be similar to what I've pasted here:

```bash
$ gradle clean build
(... bunch of output)
$ find . -name Test.class
./build/classes/main/Test.class
$ java -cp build/classes/main/ javax.tools.CompileTest
compiler: class com.sun.tools.javac.api.JavacTool
task: class com.sun.tools.javac.api.JavacTaskImpl
src/main/java/Test.java:3: error: unmappable character for encoding UTF-8
    System.out.println("Testing UTF-8 compilation: C�est dr�le, tout � coup je ne sais pas quoi dire.");
                                                    ^
src/main/java/Test.java:3: error: unmappable character for encoding UTF-8
    System.out.println("Testing UTF-8 compilation: C�est dr�le, tout � coup je ne sais pas quoi dire.");
                                                           ^
src/main/java/Test.java:3: error: unmappable character for encoding UTF-8
    System.out.println("Testing UTF-8 compilation: C�est dr�le, tout � coup je ne sais pas quoi dire.");
                                                                     ^
$ echo $?
0
$ find . -name Test.class
./build/classes/main/Test.class
./build/tmp/Test.class
$ rm build/tmp/Test.class
$ java -cp build/classes/main/ javax.tools.CompileTest useRun
compiler: class com.sun.tools.javac.api.JavacTool
Using compiler.run() instead of compiler.getTask().  Output dir: /Users/jbateskobashigawa/play/gradle/build/tmp
src/main/java/Test.java:3: error: unmappable character for encoding UTF8
    System.out.println("Testing UTF-8 compilation: C�est dr�le, tout � coup je ne sais pas quoi dire.");
                                                    ^
src/main/java/Test.java:3: error: unmappable character for encoding UTF8
    System.out.println("Testing UTF-8 compilation: C�est dr�le, tout � coup je ne sais pas quoi dire.");
                                                           ^
src/main/java/Test.java:3: error: unmappable character for encoding UTF8
    System.out.println("Testing UTF-8 compilation: C�est dr�le, tout � coup je ne sais pas quoi dire.");
                                                                     ^
3 errors
$ echo $?
1
$ find . -name Test.class
./build/classes/main/Test.class
```

Now, if you modify `Test.java` to have a genuine compilation error, the "task" version will fail also:

```bash
$ java -cp build/classes/main/ javax.tools.CompileTest
compiler: class com.sun.tools.javac.api.JavacTool
task: class com.sun.tools.javac.api.JavacTaskImpl
src/main/java/Test.java:3: error: unclosed string literal
    System.out.println("Testing UTF-8 compilation:);
                       ^
src/main/java/Test.java:3: error: ';' expected
    System.out.println("Testing UTF-8 compilation:);
                                                    ^
src/main/java/Test.java:5: error: reached end of file while parsing
}
 ^
3 errors
$ echo $?
1
```
