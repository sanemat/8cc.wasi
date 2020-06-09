8cc.wasi: Pure WebAssembly System Interface (WASI) C Compiler
===================================

This is a WASI port of [8cc](https://github.com/rui314/8cc) built on [ELVM](https://github.com/shinh/elvm).
In other words, this is a complete C compiler written in WASI.

## Compile C Code

Prepare your C code. Following is a 'Hello world' example.

```c
int putchar(int x);

int main() {
  const char* p = "Hello, world!\n";
  for (; *p; p++)
    putchar(*p);
  return 0;
}
```

8cc.wasi compiles C code into ELVM IR (EIR).
```
$ cat hello.c | wasmtime 8cc.c.eir.wasi > hello.eir
```
```
$ cat hello.eir
	.text
main:
	#{push:main}
	mov D, SP
	add D, -1
	store BP, D
	mov SP, D
	mov BP, SP
	sub SP, 1
(snip)
```

elc.wasi compiles EIR into WASI.
```
$ (echo "wasi" && cat hello.eir) | wasmtime elc.c.eir.wasi > hello.wasi
```

```
$ cat hello.wasi
(module
 (import "wasi_unstable" "fd_write" (func $__wasi_fd_write (param i32 i32 i32 i32) (result i32)))
 (import "wasi_unstable" "fd_read" (func $__wasi_fd_read (param i32 i32 i32 i32) (result i32)))
 (import "wasi_unstable" "proc_exit" (func $__wasi_proc_exit (param i32)))
 (memory (export "memory") 1025)
 (global $a (mut i32) (i32.const 0))
 (global $b (mut i32) (i32.const 0))
(snip)
```

Run hello.wasi and get a result!

```
$ wasmtime hello.wasi 
Hello, world!
```

## License

### 8cc.wasi
Copyright 2020 Matt (Sanemat) (Murahashi Kenichi)
[The MIT License](./license.txt)

### 8cc

Copyright (c) 2012 Rui Ueyama, The MIT License

### elvm

Copyright (c) 2016 Shinichiro Hamaji, The MIT License

## Refs
- [backend code for 8cc.wasi](https://github.com/shinh/elvm/commit/9a563b8889e5710f78de1d4351343a755ef9b44c)
