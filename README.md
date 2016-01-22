# SRGX 1.0 Format Specification

## Preface
This specification is still a work in progress and not yet finalized. Therefore, it should be considered subject to
breaking changes without prior notice.

## Introduction
SRGX is a format for compact and straightforward storage of deobfuscation mappings for Java programmings. It is based
off the popular SRG format, extending it to resolve multiple inconsistencies. It is not designed to be
backward-compatible with the original SRG specification.

## General Format
SRGX follows a basic general format. Each line contains a single entry, and each entry occupies a single line. Each
entry is separated into multiple elements, delimited by a single space character. The first element is a two letter
key representing type. Remaining elements will vary in purpose depending on the entry type.

## External Specifications Referenced
The SRGX format relies upon multiple external specifications within its own.

Qualified class and interface names must follow the JVMS binary name specification ([JVMS §4.2.1] [JVMS 4.2.1]).

Descriptors of class members must follow the JVMS descriptor specification ([JVMS §4.3] [JVMS 4.3]).

## Entry Types
### Class
A class entry defines a mapping for a given class name and is denoted by the key `CL`.

The second element (immediately after the key) contains the qualified original (obfuscated) name of the class.

The third element contains the qualified deobfuscated name of the class.

Qualified class names must follow a modified version of the binary name specification ([JLS §13.1] [JLS 13.1]). This
modification replaces all dots (".") with slashes ("/"). (For instance, `com.example.SomeClass` becomes
`com/example/SomeClass`.)

#### Example
```CL com/example/a com/example/SomeClass
```

This example targets the class originally named `com/example/a`, renaming it to `com/example/SomeClass`.

### Field
A field entry defines a mapping for a given field identified by a name and descriptor, and is denoted by the
key `FD`.

The second element contains the qualified original (obfuscated) name of the class containing the field.

The third element contains the unqualified original (obfuscated) name of the field.

The fourth element contains the original (obfuscated) descriptor of the field.

The fifth element contains the unqualified deobfuscated name of the field.

#### Example
```FD com/example/a b Ljava/lang/String; idString
```

This example targets field `b` with descriptor `Ljava/lang/String;` in class `com/example/a`, renaming it to `idString`.

### Method
A method entry defines a mapping for a given method identified by a name and descriptor, and is denoted by the key `MD`.

The second element contains the qualified original (obfuscated) name of the class containing the method.

The third element contains the unqualified original (obfuscated) name of the method.

The fourth element contains the original (obfuscated) descriptor of the method.

The fifth element contains the unqualified deobfuscated name of the method.

#### Example
```MD com/example/a b ()Ljava/lang/String; getName
```

This example targets method `b` with descriptor `()Ljava/lang/String;` in class `com/example/a`, renaming it to
`getName`.

### Method Parameter
A method parameter entry defines a mapping for a given parameter of a specific method, and is denoted by the key `MP`.

The second element contains the qualified original (obfuscated) name of the class containing the target method.

The third element contains the unqualified original (obfuscated) name of the target method (i.e. the one containing the
parameter of interest).

The fourth element contains the zero-indexed ordinal of the parameter of interest (`0` targets the first parameter, `1`
targets the second, etc.).

The fifth element contains the original (obfuscated) descriptor of the parameter's type.

The sixth element contains the new name of the parameter.

#### Example
```MP com/example/a b 0 Ljava/lang/String; idString
```
This example targets the first parameter (index `0`) of method `b` in class `com/example/a` with (parameter) descriptor
`Ljava/lang/String;`, renaming it to `idString`.

## Differences from SRG
This format contains many differences from its parent. Below is a list of all notable changes as well as rationale for
each.

- Removal of colon (":") in first element (key) for entries
  - Unnecessary bloat. It adds nothing of value and removal makes the format slightly simpler to parse.
- Introduction of method parameter mappings
  - Obfuscated or unnamed method parameters can be cryptic and confusing. By permitting these to be mapped, the
  deobfuscation process is allowed be more thorough.
- Separation of parent class and original simple name for field and method mappings
  - The syntax for qualified class members used by SRG is non-standard as well as ambiguous to those not familiar with
  the format. Separation of owning class and simple name makes the different components of the name more obvious, and
  additionally makes the format simpler to parse.
- Removal of parent class from deobfuscated name for field and method mappings
  - Mappings should be self-contained. Per this principle, it is not logical to include the deobfuscated name of a class
  within the mapping of one of its members. Generally, it adds nothing of value, as the deobfuscated class name may be
  easily looked up via its respective mapping.
- Removal of deobfuscated descriptor for method mappings
  - Similar to the previous item, this violates the principle of self-contained mappings and adds nothing of value
  considering it may be calculated with relative ease from already-provided information.
- Requirement of descriptor for field mappings
  - The [JVMS] [JVMS] disallows the existence of fields with identical signatures. However, a signature is comprised of
  a name and a descriptor. Therefore, it is legal for fields with identical names but different descriptors to be
  present within a single class. This makes field references including only a name ambiguous and unsatisfactory.

[JVMS]: https://docs.oracle.com/javase/specs/jvms/se8/html/index.html
[JVMS 4.2.1]: https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.2.1
[JVMS 4.3]: https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.3
