# libdmtx-java-wrapper

> **High‑performance Data Matrix barcode encoding/decoding for Java** via a tiny JNI bridge to the [libdmtx](https://github.com/dmtx/libdmtx) C library.  No external executables, just drop‑in JAR + native libs for Linux and Windows.

---

[![Build with Maven](https://img.shields.io/badge/build-maven-blue.svg)](https://maven.apache.org/)  
[![License: LGPL v2.1+](https://img.shields.io/badge/license-LGPL%20v2.1%2B-green.svg)](LICENSE)

## Features

* **Pure‑Java API** – deal with `BufferedImage` or raw RGB arrays; get back `DMTXTag` objects.
* **Bundled binaries** – 64‑bit **Linux** (`.so`) and **Windows** (`.dll`) shared libraries are included.
* **Fast & proven** – libdmtx is widely used in industry for reliable Data Matrix support.
* **Flexible licences** – core C code is Simplified BSD; the wrapper is LGPL v2.1+, making commercial use straightforward.

## Quick start

### 1. Add the dependency

```xml
<dependency>
    <groupId>org.libdmtx</groupId>
    <artifactId>libdmtx-wrapper</artifactId>
    <version>0.7.7-ab7954cc020201caa5eea301a27bb3e34cf05d8e</version>
</dependency>
```

### 2. Decode a code in three lines

```java
import org.libdmtx.*;
import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.File;

public class DecodeDemo {
    public static void main(String[] args) throws Exception {
        DMTXLoader.loadLibrary();                     // 1. Load JNI/native libs
        BufferedImage img = ImageIO.read(new File("dm.png")); // 2. Read image
        DMTXTag[] tags = new DMTXImage(img).getTags(10, 1000); // 3. Decode (max 10 tags, 1s timeout)
        for (DMTXTag t : tags) System.out.println(t.id);
    }
}
```

### 3. Create/encode a tag

```java
import org.libdmtx.*;
import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.File;

public class Demo {
    public static void main(String[] args) throws Exception {
        // 1. Load the native library once
        DMTXLoader.loadLibrary();

        // 2. Read an image and copy pixel data
        BufferedImage img = ImageIO.read(new File("qr.png"));
        int[] pixels = img.getRGB(0, 0, img.getWidth(), img.getHeight(), null, 0, img.getWidth());

        // 3. Wrap and decode
        DMTXImage qimg = new DMTXImage(img.getWidth(), img.getHeight(), pixels);
        DMTXTag[] results = qimg.getTags(1, 50);

        for (DMTXTag tag : results) System.out.println("Decoded: " + tag.id);
    }
}
```

## Build from source

*Requires JDK 8+ and a recent GCC/Clang toolchain on Linux.*  The project uses Maven; the steps below are **required** to compile C++ objects and JNI.

### Required manual build step

1. Open shell
2. Execute `cd projroot/src/main/resources/jni/linux`
3. Compile JNI
    - `g++ -c -fPIC -I/path/to/jvm/include/ -I/path/to/jvm/include/linux org_libdmtx_DMTXImage.cpp -o ../../../c/org_libdmtx_DMTXImage.o -I../../native/linux`
    - `g++ -shared -fPIC -o libdmtx.so ../../../c/org_libdmtx_DMTXImage.o ../../native/linux/libdmtx.a -lc`
4. Execute Maven `clean, compile, install` (or `deploy`)

### Automated build

From the project root:

```bash
mvn clean package
```

This compiles the Java sources, builds the JNI objects, and assembles the JAR with `libdmtx.so` inside `target/`.

### Rebuilding native libraries for other platforms

1. Build `libdmtx` as a shared library (`libdmtx.so`/`libdmtx.dll`) for your OS & architecture.
2. Copy the artefact into `src/main/resources/native/<os>/` (and `src/main/resources/jni/<os>/` if you recompile the JNI stub).
3. Adjust `DMTXLoader` search paths if you use a non‑standard directory layout.

See the [libdmtx build guide](https://github.com/dmtx/libdmtx#building) for compiler flags.

## Project layout

```
libdmtx-java-wrapper/
 ├── src/main/java           # Java wrapper classes (LGPL)
 │   └── org/libdmtx/*
 ├── src/main/resources/
 │   ├── native/linux        # Pre‑built static & shared libs
 │   └── native/win          # Windows DLL + import lib
 ├── pom.xml                 # Maven build
 ├── LICENSE                 # MIT licence
 ├── NOTICE                  # Attribution & BSD licence for libdmtx
 └── README.md               # You are here
```

## Contributing

Pull requests welcome!

## Credits

* **Mike Laughton** and the libdmtx contributors – original C library.
* **Pete Calvert & Dikran Seropian** – initial Java JNI wrapper.
* **Matteo Riva** – modernisation, build and packaging.

## Licence

* **Java wrapper code** – GNU **LGPL v2.1 or later** (see `LICENSE`).
* **Bundled libdmtx** – **Simplified BSD** (see `NOTICE`).
