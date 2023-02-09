#### hello.c
``` shell
# 生成可执行文件
clang-15 hello.c -o hello
./hello
# 生成llvm字节码
clang-15 -O3 -emit-llvm hello.c -c -o hello.bc
lli hello.bc

# 查看llvm程序集代码
llvm-dis-15 < hello.bc | less

# llc编译到本地汇编
llc-15 hello.bc -o hello.s

# 将本机汇编成程序
$ /opt/SUNWspro/bin/cc -xarch=v9 hello.s -o hello.native   # On Solaris
$ gcc hello.s -o hello.native                              # On others # ubuntu20.04失败
# 更改命令为 ====> 
$ gcc -no-pie hello.s -o hello.native                       # 成功
./hello.native

```


