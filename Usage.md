## Preliminary ##

This tool will parse two files:

  * logcat from your android device, containing a stacktrace - the stuff that looks like:

```
12-10 22:20:04.839 I/DEBUG   (  551): *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
12-10 22:20:04.839 I/DEBUG   (  551): Build fingerprint: 'generic/sdk/generic/:1.5/CUPCAKE/150240:eng/test-keys'
12-10 22:20:04.839 I/DEBUG   (  551): pid: 1003, tid: 1003  >>> com.example <<<
12-10 22:20:04.849 I/DEBUG   (  551): signal 11 (SIGSEGV), fault addr 00000004
12-10 22:20:04.849 I/DEBUG   (  551):  r0 00000000  r1 0019ed48  r2 00000001  r3 0019eae4
12-10 22:20:04.859 I/DEBUG   (  551):  r4 0019ea9c  r5 0019ed48  r6 001935b0  r7 0000a9d0
12-10 22:20:04.859 I/DEBUG   (  551):  r8 00000001  r9 0019eae4  10 804df39c  fp 0019ea9c
12-10 22:20:04.869 I/DEBUG   (  551):  ip 00000071  sp beb6f4a8  lr 8041d8d9  pc 8041cc6c  cpsr 00000030
12-10 22:20:04.948 I/DEBUG   (  551):          #00  pc 0000e3b4  /system/lib/libdvm.so
12-10 22:20:04.948 I/DEBUG   (  551): stack:
12-10 22:20:04.948 I/DEBUG   (  551):     beb6f468  00000000
12-10 22:20:04.959 I/DEBUG   (  551):     beb6f46c  0019ede3  [heap]
12-10 22:20:04.959 I/DEBUG   (  551):     beb6f470  00000002
...
```

The logcat file can have more than one stack trace - in which case, all stack traces will be processed.

  * asm file of the library being debugged. You generate this file by using the NDK's objdump tool:

```
...android-ndk/android-ndk-1.6_r1/build/prebuilt/linux-x86/arm-eabi-4.2.1/bin/arm-eabi-objdump -S mylib.so > mylib.asm
```

## Running ##

Use the script by running:
```
python parse_stack.py <asm-file> <logcat-file>
```

## Output ##

A sample output of the script would be:

```
0x0001d9dc:                   native_func_3 + 0x095c
0x0001d8d4:                   native_func_2 + 0x0854
0x0001cc6c:                   native_func_1 + 0x0048
0x0001b768:Java_com_example_MyClass_jniFunc + 0x0084
```

Each of these lines represents a function call (actually - a return address, which points to the instruction immediately after the function call).

In our case, `Java_com_example_MyClass_jniFunc` called `native_func_1` which called `native_func_2`, and so on.

`native_func_1` called `native_func_2` when it was 0x48 bytes inside the function (again - return address).

You can try to analyze exactly what happened there from looking at the assembly code - in our example, at address 0x0001cc6c.