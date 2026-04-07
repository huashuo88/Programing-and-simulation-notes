# VASP 6.3.2 安装与编译指南

## VASP 6.3.2 安装

参考链接：[Vasp6.3.2安装](https://www.bilibili.com/opus/682064175698018357)

## VTST 编译

参考链接：[VTST编译](https://zhuanlan.zhihu.com/p/8591076280)

### 编译问题

在编译 VTST-199 时，可能会遇到报错 `chain.F` 中缺少 `endif` 的问题。  
参考链接：[链式报错解决方法](http://bbs.keinsci.com/thread-46112-1-1.html)

## 编译 Vaspsol++-VTST-VASP 6.3.2

1. **打补丁**  
   在 VASP 6.3.2 文件中运行以下命令：  
   ```bash
   patch -p1 < src/vaspsol++-vtst-vasp_6.3.2.patch
   ```

2. **编译**  
   打补丁成功后，运行以下命令进行编译：  
   ```bash
   make all
   ```

## 从 CP-VASP 编译

1. **打补丁**  
   将 `Vaspsol++.patch` 文件复制到 VASP 的 `src` 目录中，运行以下命令：  
   ```bash
   patch -p1 < Vaspsol++.patch
   ```

2. **编译**  
   打补丁成功后，运行以下命令进行编译：  
   ```bash
   make all
   ```
