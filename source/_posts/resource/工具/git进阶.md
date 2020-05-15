---
title: git 进阶
abbrlink: 413432141
date: 2020-1-18 10:43:39
tag: 
    - git
category: 工具
---


## 分支的理解

1. 只有一条时间线

![]( https://static.liaoxuefeng.com/files/attachments/919022325462368/0 )

2. 新建分支只是增加一个指向时间线的指针

   ![](https://www.liaoxuefeng.com/files/attachments/919022363210080/l)

3. 继续更新时间线

   ![](https://www.liaoxuefeng.com/files/attachments/919022387118368/l)

4. 合并 让master的指针指向dev就行了，这时候如果要删除dev分支就只需要删除这个dev指针就行了：

   ![](https://www.liaoxuefeng.com/files/attachments/919022412005504/0)

### 5. 解决冲突

- 当多个分支同时，就会变成这样：

  ![](https://www.liaoxuefeng.com/files/attachments/919023000423040/0)

  这种情况，git无法像刚刚一样，直接将master指针移向feature1那样，如果直接输入合并命令，就会合并失败

- 合并失败后git会将工作区冲突的文件变成这样：

  ```
  <<<<<<< HEAD
  ddd
  =======
  dev
  >>>>>>> dev
  ```

- 修改过后再add commit一次 就会合并成功

  就像这样：

  ![](https://www.liaoxuefeng.com/files/attachments/919023031831104/0)

### 6. bug分支 feature分支

## 多人协作

多人协作是git的重要特性

### 冲突解决

如果 origin被其他人push过后，自己再想push就会有冲突，必须先pull pull后文件也会像这样：

```
<<<<<<< HEAD
		System.out.println("dev");
		System.out.println("fdsf");
=======
>>>>>>> d46723cb2f115382b802f7c02f9020beea4714cb
```

必须解决冲突后提交后再push

## 打标签 

标签就是对那串哈希出来的码取的特殊名字

1. 给当前版本，当前分支打标签：git tag v1.0

2. 查看所有标签：git tag

3. 给指定的commit id 打标签：git tag v0.9 f52c633

4. 还可以创建带有说明的标签，用`-a`指定标签名，`-m`指定说明文字：

   ```
   $ git tag -a v0.1 -m "version 0.1 released" 1094adb
   ```

5. 用命令`git show `可以看到说明文字：

   ```
   $ git show v0.1
   tag v0.1
   Tagger: Michael Liao <askxuefeng@gmail.com>
   Date:   Fri May 18 22:48:43 2018 +0800
   
   version 0.1 released
   
   commit 1094adb7b9b3807259d8cb349e7df1d4d6477073 (tag: v0.1)
   Author: Michael Liao <askxuefeng@gmail.com>
   Date:   Fri May 18 21:06:15 2018 +0800
   
       append GPL
   
   diff --git a/readme.txt b/readme.txt
   ...
   ```

## GitHub

使用唯一远程仓库进行多台电脑的远程开发方式就用以上方式，不过对于GitHub而言，这样必授权其他电脑拥有改写自己整个账号代码的权利，配合GitHub，如果我们想改其他人的代码，可以先fork下来，这样就变成你账号下的项目了，自己随心所欲修改后可以提交pull request，别人如果心情好，可能就采纳l


