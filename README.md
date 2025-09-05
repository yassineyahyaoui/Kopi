### Kopi Suite — Build, Run, and Integrate

Kopi is a suite of Java tools and libraries that includes:
- **KJC**: a Java compiler
- **XKOPI**: Java/SQL integration tools
- **VKOPI**: UI/runtime libraries
- **Bytecode tools**: classfile manipulation, optimizer, SSA, assembler/disassembler
- **Build tools**: lexer/parser generators (JFlex/ANTLR), options/messages generators, etc.

This repo is Gradle-based (wrapper included) and also contains legacy Make/Ant build scripts.

---

## TL;DR (Linux/macOS)

```bash
# 1) Prerequisites
sdk install java 8.0.392-open   # or use any JDK 8+ installed on your system

# 2) Prepare output and external libs directories (adjust paths as you like)
mkdir -p .classroot extlibs
export CLASSROOT="$(pwd)/.classroot"
export EXTDIRS="$(pwd)/extlibs"

# 3) Put required third‑party jars in $EXTDIRS
#   Required for many tools (minimum):
#   - org.jdom:jdom:1.1.3
#   - gnu.getopt:java-getopt:1.0.14
#   - JFlex 1.4.x (e.g., de.jflex:jflex:1.4.3)
#   Optional/feature-specific: JavaMail, Activation, HylaFax client, etc.

# 4) Build everything into $CLASSROOT
./gradlew run

# 5) Run a tool, e.g. KJC Java compiler
java -Djava.ext.dirs="$EXTDIRS" -cp "$CLASSROOT" org.kopi.kopi.comp.kjc.Main -help
```

Windows PowerShell: set environment variables with `$env:CLASSROOT` and `$env:EXTDIRS` and use `gradlew.bat`.

---

## Requirements

- **JDK**: 8+ (JDK 8 or 11 recommended). If compiling explicitly for Java 7, set `JDK_7` to a valid JDK 7 home.
- **Gradle**: Wrapper provided (Gradle 7.2), no local install needed.
- **Environment variables (required/important):**
  - `CLASSROOT`: absolute path where compiled/generated classes and resources are written.
  - `EXTDIRS`: colon/semicolon-separated directory list with required external jars (see below).

External jars you’ll typically need in `EXTDIRS`:
- **JDOM**: `org.jdom:jdom:1.1.3`
- **GNU Getopt**: `gnu.getopt:java-getopt:1.0.14`
- **JFlex 1.4.x**: e.g., `de.jflex:jflex:1.4.3` (Kopi targets 1.4)
- Optional (feature-specific): JavaMail, Activation (JAF), IBM BigDecimal (for XKOPI), HylaFAX Java client, MESP expression parser, etc.

Notes:
- Gradle build automatically adds jars found under each directory in `EXTDIRS` to the runtime classpath.
- Several tools are launched via `java` with `-Djava.ext.dirs=$EXTDIRS` to load these jars.

---

## Building

Kopi supports three build paths. Prefer the Gradle wrapper.

### A) Gradle (recommended)

```bash
export CLASSROOT="/absolute/path/to/.classroot"
export EXTDIRS="/absolute/path/to/extlibs"

./gradlew run      # default build; generates sources, compiles, and stages artifacts into $CLASSROOT

# Build only a subset by package or folder (faster iterative builds):
./gradlew -Ppackage=org.kopi.kopi.comp.kjc run
./gradlew -Pfolder=src/org/kopi/kopi/comp/kjc run

# Cleaning
./gradlew clean            # clean Gradle outputs
./gradlew clean-classes    # clean compiled classes
```

Notes:
- The build writes outputs to `CLASSROOT`. Keep this directory stable to avoid unnecessary rebuilds.
- Vaadin/UI parts resolve from Maven Central and Vaadin Addons; other classic dependencies come from `EXTDIRS`.

### B) Make (legacy)

```bash
export CLASSROOT="/absolute/path/to/.classroot"
export EXTDIRS="/absolute/path/to/extlibs"
make               # from repo root; delegates to src/org/kopi
```

### C) Ant (legacy)

```bash
export CLASSROOT="/absolute/path/to/.classroot"
export EXTDIRS="/absolute/path/to/extlibs"
ant -f build.xml   # from repo root or src/build.xml
```

---

## Running the tools

After a successful `./gradlew run`, you can invoke tools directly. Use `-Djava.ext.dirs=$EXTDIRS` so Java can find required third‑party jars; use `-cp $CLASSROOT` to load Kopi classes.

- **KJC (Java compiler)**
```bash
java -Djava.ext.dirs="$EXTDIRS" -cp "$CLASSROOT" \
  org.kopi.kopi.comp.kjc.Main -help

# Example: compile a file and emit class files under out/
mkdir -p out
java -Djava.ext.dirs="$EXTDIRS" -cp "$CLASSROOT" \
  org.kopi.kopi.comp.kjc.Main -d out src/main/java/Hello.java
```

- **SQLC / XKJC (SQL and Java+SQL tools)**
```bash
java -Djava.ext.dirs="$EXTDIRS" -cp "$CLASSROOT" org.kopi.xkopi.comp.sqlc.Main -help
java -Djava.ext.dirs="$EXTDIRS" -cp "$CLASSROOT" org.kopi.xkopi.comp.xkjc.Main -help
```

- **Bytecode tools**
```bash
java -Djava.ext.dirs="$EXTDIRS" -cp "$CLASSROOT" org.kopi.bytecode.dis.Main -help   # Disassembler
java -Djava.ext.dirs="$EXTDIRS" -cp "$CLASSROOT" org.kopi.bytecode.ksm.Main -help   # Assembler
java -Djava.ext.dirs="$EXTDIRS" -cp "$CLASSROOT" org.kopi.bytecode.optimize.Main -help
java -Djava.ext.dirs="$EXTDIRS" -cp "$CLASSROOT" org.kopi.bytecode.ssa.Main -help
```

- **Build-time generators**
```bash
java -Djava.ext.dirs="$EXTDIRS" -cp "$CLASSROOT" org.kopi.compiler.tools.msggen.Main path/to/Messages.xml
java -Djava.ext.dirs="$EXTDIRS" -cp "$CLASSROOT" org.kopi.compiler.tools.optgen.Main  path/to/Options.xml
java -Djava.ext.dirs="$EXTDIRS" -cp "$CLASSROOT" JFlex.Main -skel "$CLASSROOT/org/kopi/compiler/skeleton.shared" path/to/Scanner.flex
```

Tip: There is a generic dispatcher `org.kopi.drivers.kopi.Main` which tries to run `org.kopi.<your.module>.Main`, e.g. `compiler.tools.msggen`. For clarity and better error handling, prefer calling tool Main classes explicitly as above.

---

## Packaging a jar (optional)

The Gradle build writes compiled classes to `CLASSROOT`. If you want a single jar for consumption:

```bash
mkdir -p build
jar cfv build/kopi-all-classes.jar -C "$CLASSROOT" .
```

Then use `build/kopi-all-classes.jar` in your downstream projects (see integration below). Keep external jars from `EXTDIRS` alongside it.

---

## Using Kopi in your projects

You can integrate Kopi in two ways: as runtime libraries or as build-time tools.

### 1) As runtime/library dependency (Gradle example)

Place `kopi-all-classes.jar` and any required third‑party jars into your project’s `libs/` directory, then add:

```groovy
repositories { flatDir { dirs 'libs' } }

dependencies {
  implementation files('libs/kopi-all-classes.jar')
  implementation fileTree(dir: 'libs', include: ['*.jar'])  // pulls JDOM, getopt, etc. you placed in libs/
}
```

### 2) As build-time tools (Gradle JavaExec tasks)

If you want to run KJC or the generators during your build, add tasks that point to your local Kopi build output and EXTDIRS:

```groovy
def kopiClassroot = file('/absolute/path/to/kopi/.classroot')
def extDirs = file('/absolute/path/to/kopi/extlibs')

tasks.register('kjc', JavaExec) {
  mainClass = 'org.kopi.kopi.comp.kjc.Main'
  classpath = files(kopiClassroot)
  jvmArgs "-Djava.ext.dirs=${extDirs.absolutePath}"
  args '-d', "$buildDir/classes/java/main", 'src/main/java/Hello.java'
}
```

You can mirror this pattern for `msggen`, `optgen`, `JFlex.Main`, etc.

### 3) As a source/submodule (advanced)

Add this repository as a Git submodule and either:
- add `kopi/src` to your project’s compile sources, or
- create a dedicated build that runs `./gradlew run` in Kopi first, then consumes `CLASSROOT` via `files(kopiClassroot)`.

---

## Troubleshooting

- NullPointerException in Gradle due to `classRoot!!`: ensure `CLASSROOT` is set to an existing directory.
- ClassNotFoundException for JDOM/Getopt/JFlex: ensure the jars are present under `EXTDIRS` and that you’re passing `-Djava.ext.dirs=$EXTDIRS` when launching tools.
- Building only part of the tree: use `-Ppackage=...` or `-Pfolder=...` to reduce build time.

---

## License

Libraries are LGPL-2.1; compilers/tools are GPL-2. See `COPYING` and `COPYING.LIB`.

