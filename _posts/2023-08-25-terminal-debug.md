---
layout: distill
title: Terminal debugging
description: practice
giscus_comments: false
date: 2023-08-25

toc:
  - name: Terminal debugging tool gdb
  - name: File content investigation
  - name: Reference
---

## Terminal debugging tool gdb

Tool <a href="https://www.sourceware.org/gdb/">gdb</a>: The GNU Project Debugger. This is the <a href="https://sourceware.org/gdb/current/onlinedocs/gdb"> user manual</a>.


```bash
# Start target executable app running under GDB control
gdb [--args] <target_app> [args list]

# attach to a running process
gdb -p <pid>



# we can set break points
break <line_idendifier>

# Use the run command to start your program under GDB.
r

# we can step to the next line
n
# or next step 
s
# or continue
c

# print var values that show up during the step execution.
p <var_name>

# quit after done
quit
```

Use the pytorch segmentation fault as an example:
```bash
python -c "import torch; print(torch)"
# segmentation fault (core dumped)
```

```bash
gdb python
```

```bash
# option -c: use content/file as a core dump to examine.
r -c "import torch; print(torch)"

# backtrace
bt

#0  0x0000000000002360 in ?? ()
#1  0x00007ffff7de5a6a in call_init (l=<optimized out>, argc=argc@entry=3, 
    argv=argv@entry=0x7fffffffde28, env=env@entry=0xb85860) at dl-init.c:72
#2  0x00007ffff7de5b7b in call_init (env=0xb85860, argv=0x7fffffffde28, 
    argc=3, l=<optimized out>) at dl-init.c:30
#3  _dl_init (main_map=main_map@entry=0xeb4270, argc=3, argv=0x7fffffffde28, 
    env=0xb85860) at dl-init.c:120
#4  0x00007ffff7deab86 in dl_open_worker (a=a@entry=0x7fffffffb840)
    at dl-open.c:575
#5  0x00007ffff6d88d64 in __GI__dl_catch_error (objname=0x7fffffffb830, 
    errstring=0x7fffffffb838, mallocedp=0x7fffffffb82f, 
    operate=0x7ffff7dea7a0 <dl_open_worker>, args=0x7fffffffb840)
    at dl-error-skeleton.c:198
#6  0x00007ffff7dea0d9 in _dl_open (
    file=0x7fffdf813a60 "/usr/local/lib/python3.6/dist-packages/torch/_C.cpython-36m-x86_64-linux-gnu.so", mode=-2147483391, 
    caller_dlopen=0x58de76 <_PyImport_FindSharedFuncptr+390>, nsid=-2, 
    argc=<optimized out>, argv=<optimized out>, env=0xb85860) at dl-open.c:660
#7  0x00007ffff79b2ff6 in dlopen_doit (a=a@entry=0x7fffffffba50) at dlopen.c:66
#8  0x00007ffff6d88d64 in __GI__dl_catch_error (objname=0xb85d10, 
    errstring=0xb85d18, mallocedp=0xb85d08, 
    operate=0x7ffff79b2fa0 <dlopen_doit>, args=0x7fffffffba50)
    at dl-error-skeleton.c:198
#9  0x00007ffff79b3759 in _dlerror_run (
    operate=operate@entry=0x7ffff79b2fa0 <dlopen_doit>, 
    args=args@entry=0x7fffffffba50) at dlerror.c:163
#10 0x00007ffff79b3092 in __dlopen (file=<optimized out>, mode=<optimized out>)
    at dlopen.c:87
#11 0x000000000058de76 in _PyImport_FindSharedFuncptr ()
#12 0x0000000000576b18 in _PyImport_LoadDynamicModuleWithSpec ()
#13 0x00000000005743c5 in ?? ()
#14 0x00000000004c47ed in PyCFunction_Call ()
#15 0x0000000000557040 in _PyEval_EvalFrameDefault ()
#16 0x000000000054efc1 in ?? ()
#17 0x000000000054f24d in ?? ()
#18 0x0000000000553aaf in _PyEval_EvalFrameDefault ()
#19 0x000000000054e4c8 in ?? ()
#20 0x000000000054f4f6 in ?? ()
#21 0x0000000000553aaf in _PyEval_EvalFrameDefault ()
#22 0x000000000054e4c8 in ?? ()
#23 0x000000000054f4f6 in ?? ()
#24 0x0000000000553aaf in _PyEval_EvalFrameDefault ()
#25 0x000000000054e4c8 in ?? ()
#26 0x000000000054f4f6 in ?? ()
#27 0x0000000000553aaf in _PyEval_EvalFrameDefault ()
#28 0x000000000054e4c8 in ?? ()
#29 0x000000000054f4f6 in ?? ()
#30 0x0000000000553aaf in _PyEval_EvalFrameDefault ()
#31 0x000000000054e4c8 in ?? ()
#32 0x00000000005582c2 in _PyFunction_FastCallDict ()
#33 0x0000000000459981 in _PyObject_FastCallDict ()
#34 0x000000000045b034 in _PyObject_CallMethodIdObjArgs ()
#35 0x000000000057581a in PyImport_ImportModuleLevelObject ()
#36 0x0000000000556add in _PyEval_EvalFrameDefault ()
#37 0x000000000054efc1 in ?? ()
#38 0x000000000054ff73 in PyEval_EvalCode ()
#39 0x000000000054cfda in ?? ()
#40 0x00000000004c47ed in PyCFunction_Call ()
#41 0x0000000000557040 in _PyEval_EvalFrameDefault ()
#42 0x000000000054efc1 in ?? ()
#43 0x000000000054f24d in ?? ()
#44 0x0000000000553aaf in _PyEval_EvalFrameDefault ()
#45 0x000000000054e4c8 in ?? ()
#46 0x000000000054f4f6 in ?? ()
#47 0x0000000000553aaf in _PyEval_EvalFrameDefault ()
#48 0x000000000054e4c8 in ?? ()
#49 0x000000000054f4f6 in ?? ()
#50 0x0000000000553aaf in _PyEval_EvalFrameDefault ()
#51 0x000000000054e4c8 in ?? ()
#52 0x000000000054f4f6 in ?? ()
#53 0x0000000000553aaf in _PyEval_EvalFrameDefault ()
#54 0x000000000054e4c8 in ?? ()
#55 0x00000000005582c2 in _PyFunction_FastCallDict ()
#56 0x0000000000459981 in _PyObject_FastCallDict ()
#57 0x000000000045b034 in _PyObject_CallMethodIdObjArgs ()
#58 0x000000000057581a in PyImport_ImportModuleLevelObject ()
#59 0x0000000000556add in _PyEval_EvalFrameDefault ()
#60 0x000000000054efc1 in ?? ()
#61 0x000000000054ff73 in PyEval_EvalCode ()
#62 0x000000000042c37a in PyRun_SimpleStringFlags ()
#63 0x0000000000441178 in Py_Main ()
#64 0x0000000000421f64 in main ()
```

## File content investigation

### Investigate dynamic lib files

Tool <a href="https://linux.die.net/man/1/ldd">ldd</a>: print shared library dependencies.

```bash
ldd /usr/local/lib/python3.6/dist-packages/torch/_C.cpython-36m-x86_64-linux-gnu.so

# linux-vdso.so.1 =>  (0x00007ffcce1f5000)
#   libshm.so => /usr/local/lib/python3.6/dist-packages/torch/lib/libshm.so (0x00007f37cdcac000)
#   libcudart.so.9.0 => /usr/local/lib/python3.6/dist-packages/torch/lib/libcudart.so.9.0 (0x00007f37cda3d000)
#   libnvToolsExt.so.1 => /usr/local/lib/python3.6/dist-packages/torch/lib/libnvToolsExt.so.1 (0x00007f37cd833000)
#   libcudnn.so.7 => /usr/local/lib/python3.6/dist-packages/torch/lib/libcudnn.so.7 (0x00007f37bc5c0000)
#   libTH.so.1 => /usr/local/lib/python3.6/dist-packages/torch/lib/libTH.so.1 (0x00007f37b9a91000)
#   libTHS.so.1 => /usr/local/lib/python3.6/dist-packages/torch/lib/libTHS.so.1 (0x00007f37b9860000)
#   libTHNN.so.1 => /usr/local/lib/python3.6/dist-packages/torch/lib/libTHNN.so.1 (0x00007f37b951d000)
#   libATen.so.1 => /usr/local/lib/python3.6/dist-packages/torch/lib/libATen.so.1 (0x00007f37b8d0f000)
#   libTHC.so.1 => /usr/local/lib/python3.6/dist-packages/torch/lib/libTHC.so.1 (0x00007f37a4353000)
#   libTHCS.so.1 => /usr/local/lib/python3.6/dist-packages/torch/lib/libTHCS.so.1 (0x00007f37a3fc3000)
#   libTHCUNN.so.1 => /usr/local/lib/python3.6/dist-packages/torch/lib/libTHCUNN.so.1 (0x00007f379e8ef000)
#   libnccl.so.1 => /usr/local/lib/python3.6/dist-packages/torch/lib/libnccl.so.1 (0x00007f379be65000)
#   libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f379bc4e000)
#   libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f379ba2f000)
#   libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f379b64f000)
#   /lib64/ld-linux-x86-64.so.2 (0x00007f37cf854000)
#   librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007f379b447000)
#   libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f379b0f1000)
#   libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f379aeed000)
#   libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f379ab65000)
#   libgomp.so.1 => /usr/local/lib/python3.6/dist-packages/torch/lib/libgomp.so.1 (0x00007f379a94e000)
#   libcublas.so.9.0 => /usr/local/lib/python3.6/dist-packages/torch/lib/libcublas.so.9.0 (0x00007f3797515000)
#   libcurand.so.9.0 => /usr/local/lib/python3.6/dist-packages/torch/lib/libcurand.so.9.0 (0x00007f37935b0000)
#   libcusparse.so.9.0 => /usr/local/lib/python3.6/dist-packages/torch/lib/libcusparse.so.9.0 (0x00007f378fe45000)
```

### List all symbols defined in the lib file

Reference: <a href="https://stackoverflow.com/questions/34732/how-do-i-list-the-symbols-in-a-so-file">How do I list the symbols in a .so file</a>

Using GNU `nm`:
```bash
nm -gD yourLib.so

# Add the "-C" option to see symbols of a C++ library
nm -gDC yourLib.so
```

If your .so file is in elf format, you have two options:

Either `objdump` (`-C` is also useful for demangling C++):
```bash
objdump -TC libz.so

libz.so:     file format elf64-x86-64

DYNAMIC SYMBOL TABLE:
0000000000002010 l    d  .init  0000000000000000              .init
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 free
0000000000000000      DF *UND*  0000000000000000  GLIBC_2.2.5 __errno_location
0000000000000000  w   D  *UND*  0000000000000000              _ITM_deregisterTMCloneTable
```
or use `readelf`:
```bash
readelf -Ws libz.so
Symbol table '.dynsym' contains 112 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 0000000000002010     0 SECTION LOCAL  DEFAULT   10
     2: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND free@GLIBC_2.2.5 (14)
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __errno_location@GLIBC_2.2.5 (14)
     4: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterTMCloneTable
```
## Reference

<a href="https://github.com/pytorch/pytorch/issues/4101">Segmentation Fault when importing Torch</a>