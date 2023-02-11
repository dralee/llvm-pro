#### testfile.ll
优化
``` shell
# 优化掉冗余代码，指令合并
opt -S -instcombine testfile.ll -o output1.ll
# 优化进行无用参数消除
opt -S -deadargelim testfile.ll -o output2.ll
```

#### multiply.c
生成IR
``` shell
# 通过clang -emit-llvm生成IR
clang -emit-llvm -S multiply.c -o multiply.ll
# 通过cc1生成IR
clang -cc1 -emit-llvm multiply.c -o multiply1.ll
```

#### test.ll
将IR转bitcode
llvm-as汇编器
llc静态编译器，将bitcode输出为汇编
```shell
llvm-as test.ll -o test.bc
# 查看位流
hexdump -C test.bc
# 输出汇编
llc test.bc -o test.s
# 使用clang输出汇编
clang -S test.bc -o test1.s -fomit-frame-pointer # clang默认不消除帧指针而llc默认消除
```
##### 指定目录架构生成
-march=architecture参数，可生成特定目标架构汇编码
-mcpu=cpu参数则可指定其CPU
-reg alloc=basic greedy fast/pbqp则可指定寄存器分配类型

llvm-dis把bitcode转回IR
```shell
llvm-dis test.bc -o test1.ll
```

#### 使用opt工具把IR转换成其他形式及对代码实施多个优化

```shell
# 将c编译为未优化的IR
clang -emit-llvm -S multiply.c -o multiply.ll
# 将变量从内存提升到寄存器
opt -mem2reg -S multiply.ll -o multiply1.ll
```
opt参数：-analyze参数，会在输入源码上执行不同分析，并在标准输出流或错误流打印分析结果

#### llvm-link链接bitcode
```shell
# 先编译成IR
clang -emit-llvm -S test1.c -o test1.ll
clang -emit-llvm -S test2.c -o test2.ll
# 输出成bitcode
llvm-as test1.ll -o test1.bc
llvm-as test2.ll -o test2.bc
# 链接为单个bitcode
llvm-link test1.bc test2.bc -o output.bc
# 查看链接后生成bitcode对应的IR
llvm-link -S output.bc -o output.ll
```

#### lli执行LLVM bitcode
```shell
$ lli output.bc
number is 10
```
