### Kopi Suite — Quick Start (Beginner Friendly)

Follow these steps exactly. Copy/paste the commands in your terminal.

---

## 1) Install Java

- You need Java 8 or newer. To check:

```bash
java -version
```

If you don’t have it, install JDK 8 or 11 (your OS package manager or SDKMAN).

---

## 2) Get the code

```bash
git clone https://github.com/your-org-or-fork/kopi.git
cd kopi
```

If you already have the code open, just `cd` into the project directory.

---

## 3) Create folders and set environment

These two folders are where the build writes files and where you’ll put helper libraries.

```bash
mkdir -p .classroot extlibs
export CLASSROOT="$(pwd)/.classroot"
export EXTDIRS="$(pwd)/extlibs"
```

Tip for Windows (PowerShell):
```powershell
$env:CLASSROOT = "$PWD\.classroot"
$env:EXTDIRS   = "$PWD\extlibs"
```

---

## 4) Download two helper libraries (one-time)

Put these .jar files into the `extlibs` folder:
- JDOM 1.1.3 (file name like `jdom-1.1.3.jar`)
- GNU Getopt 1.0.14 (file name like `java-getopt-1.0.14.jar`)

How to get them quickly (example using Maven Central direct downloads or your browser):
- Search the web for: “jdom 1.1.3 jar download” and “java-getopt 1.0.14 jar download”
- Save both jars into the `extlibs` directory you created in step 3.

You can add more later if you use advanced features, but these two are enough to start.

---

## 5) Build the project (one command)

```bash
./gradlew run
```

This will:
- generate sources (lexers/parsers/options/messages)
- compile everything
- put all compiled classes into `.classroot`

If it fails with “CLASSROOT is null”, re-run step 3 to set the variables, then run again.

---

## 6) Try a tool (KJC Java compiler)

```bash
java -Djava.ext.dirs="$EXTDIRS" -cp "$CLASSROOT" org.kopi.kopi.comp.kjc.Main -help
```

You should see usage/help text. That confirms your build worked.

To compile a sample file:
```bash
echo 'public class Hello { public static void main(String[] a){ System.out.println("Hi"); }}' > Hello.java
mkdir -p out
java -Djava.ext.dirs="$EXTDIRS" -cp "$CLASSROOT" \
  org.kopi.kopi.comp.kjc.Main -d out Hello.java
```

---

## 7) Use Kopi in your own project (simplest way)

Option A: Add the compiled classes as a jar to your project.

```bash
mkdir -p build
jar cf build/kopi-all-classes.jar -C "$CLASSROOT" .
```

Then copy `build/kopi-all-classes.jar` and your `extlibs/*.jar` into your app’s `libs/` folder and add them to your project classpath (IDE or build tool).

Option B: Call Kopi tools from your build.
- Point your build (Gradle/Maven) to `CLASSROOT` as a classpath entry
- When running tools with `java`, pass `-Djava.ext.dirs=/path/to/extlibs`

Start simple with Option A. Move to Option B when you want to automate.

---

## Common issues

- “Class not found: org.jdom…” → Make sure `jdom-1.1.3.jar` is inside `extlibs/` and you passed `-Djava.ext.dirs="$EXTDIRS"`.
- “CLASSROOT is null or missing” → Repeat step 3 in the same terminal, then run the build again.
- Different terminal session? Re-export `CLASSROOT` and `EXTDIRS` after opening a new terminal.

---

## License

Libraries are LGPL-2.1; compilers/tools are GPL-2. See `COPYING` and `COPYING.LIB`.

