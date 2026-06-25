# EasyBitPacking

### Context

Bit packing stores multiple logical values inside one primitive such as
`byte`, `short`, `int`, or `long`.

This is useful when:

- the full range of a primitive is not needed for every value
- heap allocations should be avoided
- compact data layouts matter

The downside is that handwritten bitpacking code is repetitive, hard to read,
and easy to break with small masking or shifting mistakes.

### Goal

This project provides an annotation-based Java annotation processor that
generates bitpacking helper classes automatically.

You describe a packed type with a normal Java class and annotate its fields.
The processor then generates a utility class with packing, unpacking, and
update methods.

### Modules

- `bitpacking-annotation`: contains `@BitPacked`, `@BitField`, and `PackedStorage`
- `bitpacking-processor`: contains the annotation processor and code generator
- `bitpacking-demo`: contains example specs and tests

### How To Use

1. Update your pom to contain

```xml
<repositories>
    <repository>
        <id>jitpack.io</id>
        <url>https://jitpack.io</url>
    </repository>
</repositories>


<dependency>
    <groupId>com.github.Carcassonne-SGE.EasyBitPacking</groupId>
    <artifactId>bitpacking-annotation</artifactId>
    <version>v3</version>
</dependency>

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.13.0</version>
    <configuration>
        <generatedSourcesDirectory>${project.build.directory}/generated-sources/annotations</generatedSourcesDirectory>
        <annotationProcessorPaths>
            <path>
                <groupId>com.github.Carcassonne-SGE.EasyBitPacking</groupId>
                <artifactId>bitpacking-processor</artifactId>
                <version>v3</version>
            </path>
        </annotationProcessorPaths>
    </configuration>
</plugin>

2. Annotate a class with `@BitPacked`
3. every field that shall be added to the layout shall be marked with @BitField`
4. compile  and reload maven 
```

### Example

Specification:

```java
package demo;

import annotation.BitField;
import annotation.BitPacked;
import annotation.PackedStorage;

@BitPacked(storage = PackedStorage.SHORT, useValidBit = true)
class HausSpec {

    @BitField(bits = 5)
    int rooms;

    @BitField(bits = 3)
    int floors;

    @BitField(bits = 1)
    boolean garage;
}
```

Generated class:

- `HausSpecBit` is generated in the same package
- `rooms`, `floors`, and `garage` are packed in declaration order
- one extra valid bit is reserved and always set to `1`

Usage:

```java
short packed = HausSpecBit.pack(12, 3, true);
int rooms = HausSpecBit.getRooms(packed);
short updated = HausSpecBit.addRooms(packed, 2);
```
