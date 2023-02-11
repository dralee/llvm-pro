### 使用clang
#### 1.作为编译器
```shell
$ clang test.c
$ ./a.out
hello world
```

#### 2.作为预处理器
加上-E参数
```shell
$ clang test1.c -E
# 1 "test1.c"
# 1 "<built-in>" 1
# 1 "<built-in>" 3
# 374 "<built-in>" 3
# 1 "<command line>" 1
# 1 "<built-in>" 2
# 1 "test1.c" 2
......
# 2 "test1.c" 2
void func() {
    int a[100];
}

int main() {
    printf("hello world\n");
    return 0;
}
```
#### 3.编译前端
-cc1参数保证编译器前端，而不是编译器驱动
打印语法树
```shell
$ clang -cc1 test1.c -ast-dump
TranslationUnitDecl 0x7fffbf3f3d58 <<invalid sloc>> <invalid sloc>
|-TypedefDecl 0x7fffbf3f4580 <<invalid sloc>> <invalid sloc> implicit __int128_t '__int128'
| `-BuiltinType 0x7fffbf3f4320 '__int128'
|-TypedefDecl 0x7fffbf3f45f0 <<invalid sloc>> <invalid sloc> implicit __uint128_t 'unsigned __int128'
| `-BuiltinType 0x7fffbf3f4340 'unsigned __int128'
|-TypedefDecl 0x7fffbf3f48f8 <<invalid sloc>> <invalid sloc> implicit __NSConstantString 'struct __NSConstantString_tag'
| `-RecordType 0x7fffbf3f46d0 'struct __NSConstantString_tag'
|   `-Record 0x7fffbf3f4648 '__NSConstantString_tag'
|-TypedefDecl 0x7fffbf3f4990 <<invalid sloc>> <invalid sloc> implicit __builtin_ms_va_list 'char *'
| `-PointerType 0x7fffbf3f4950 'char *'
|   `-BuiltinType 0x7fffbf3f3e00 'char'
|-TypedefDecl 0x7fffbf3f4c88 <<invalid sloc>> <invalid sloc> implicit __builtin_va_list 'struct __va_list_tag[1]'
| `-ConstantArrayType 0x7fffbf3f4c30 'struct __va_list_tag[1]' 1
|   `-RecordType 0x7fffbf3f4a70 'struct __va_list_tag'
|     `-Record 0x7fffbf3f49e8 '__va_list_tag'
|-FunctionDecl 0x7fffbf44c090 <test1.c:5:1, line:7:1> line:5:6 func 'void ()'
| `-CompoundStmt 0x7fffbf44c2a0 <col:13, line:7:1>
|   `-DeclStmt 0x7fffbf44c288 <line:6:5, col:15>
|     `-VarDecl 0x7fffbf44c220 <col:5, col:14> col:9 a 'int[100]'
|-FunctionDecl 0x7fffbf44c310 <line:9:1, line:12:1> line:9:5 main 'int ()'
| `-CompoundStmt 0x7fffbf44c6f8 <col:12, line:12:1>
|   |-CallExpr 0x7fffbf44c670 <line:10:5, col:27> 'int'
|   | |-ImplicitCastExpr 0x7fffbf44c658 <col:5> 'int (*)(const char *, ...)' <FunctionToPointerDecay>
|   | | `-DeclRefExpr 0x7fffbf44c588 <col:5> 'int (const char *, ...)' Function 0x7fffbf44c3e8 'printf' 'int (const char *, ...)'
|   | `-ImplicitCastExpr 0x7fffbf44c6b0 <col:12> 'const char *' <NoOp>
|   |   `-ImplicitCastExpr 0x7fffbf44c698 <col:12> 'char *' <ArrayToPointerDecay>
|   |     `-StringLiteral 0x7fffbf44c5e8 <col:12> 'char[13]' lvalue "hello world\n"
|   `-ReturnStmt 0x7fffbf44c6e8 <line:11:5, col:12>
|     `-IntegerLiteral 0x7fffbf44c6c8 <col:12> 'int' 0
`-FunctionDecl 0x7fffbf44c3e8 <line:10:5> col:5 implicit used printf 'int (const char *, ...)' extern
  |-ParmVarDecl 0x7fffbf44c4e0 <<invalid sloc>> <invalid sloc> 'const char *'
  |-BuiltinAttr 0x7fffbf44c488 <<invalid sloc>> Implicit 833
  `-FormatAttr 0x7fffbf44c550 <col:5> Implicit printf 1 2
```

#### 4.代码生成器
打印LLVM汇编
```shell
$ clang test1.c -S -emit-llvm -o -
; ModuleID = 'test1.c'
source_filename = "test1.c"
target datalayout = "e-m:e-p270:32:32-p271:32:32-p272:64:64-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86_64-pc-linux-gnu"

@.str = private unnamed_addr constant [13 x i8] c"hello world\0A\00", align 1

; Function Attrs: noinline nounwind optnone uwtable
define dso_local void @func() #0 {
  %1 = alloca [100 x i32], align 16
  ret void
}

; Function Attrs: noinline nounwind optnone uwtable
define dso_local i32 @main() #0 {
  %1 = alloca i32, align 4
  store i32 0, ptr %1, align 4
  %2 = call i32 (ptr, ...) @printf(ptr noundef @.str)
  ret i32 0
}

declare i32 @printf(ptr noundef, ...) #1

attributes #0 = { noinline nounwind optnone uwtable "frame-pointer"="all" "min-legal-vector-width"="0" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }
attributes #1 = { "frame-pointer"="all" "no-trapping-math"="true" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+cx8,+fxsr,+mmx,+sse,+sse2,+x87" "tune-cpu"="generic" }

!llvm.module.flags = !{!0, !1, !2, !3, !4}
!llvm.ident = !{!5}

!0 = !{i32 1, !"wchar_size", i32 4}
!1 = !{i32 7, !"PIC Level", i32 2}
!2 = !{i32 7, !"PIE Level", i32 2}
!3 = !{i32 7, !"uwtable", i32 2}
!4 = !{i32 7, !"frame-pointer", i32 2}
!5 = !{!"Ubuntu clang version 15.0.7"}
```

输出机器码汇编
```shell
$ clang test1.c -S -o -
        .text
        .file   "test1.c"
        .globl  func                            # -- Begin function func
        .p2align        4, 0x90
        .type   func,@function
func:                                   # @func
        .cfi_startproc
# %bb.0:
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset %rbp, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register %rbp
        popq    %rbp
        .cfi_def_cfa %rsp, 8
        retq
.Lfunc_end0:
        .size   func, .Lfunc_end0-func
        .cfi_endproc
                                        # -- End function
        .globl  main                            # -- Begin function main
        .p2align        4, 0x90
        .type   main,@function
main:                                   # @main
        .cfi_startproc
# %bb.0:
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset %rbp, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register %rbp
        subq    $16, %rsp
        movl    $0, -4(%rbp)
        leaq    .L.str(%rip), %rdi
        movb    $0, %al
        callq   printf@PLT
        xorl    %eax, %eax
        addq    $16, %rsp
        popq    %rbp
        .cfi_def_cfa %rsp, 8
        retq
.Lfunc_end1:
        .size   main, .Lfunc_end1-main
        .cfi_endproc
                                        # -- End function
        .type   .L.str,@object                  # @.str
        .section        .rodata.str1.1,"aMS",@progbits,1
.L.str:
        .asciz  "hello world\n"
        .size   .L.str, 13

        .ident  "Ubuntu clang version 15.0.7"
        .section        ".note.GNU-stack","",@progbits
        .addrsig
        .addrsig_sym printf
```