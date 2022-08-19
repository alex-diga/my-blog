---
title: 工程化
date: 2022-05-28
---

## 规范

1. lint-staged，执行一些操作，如 eslint、prettier
2. husky、git 钩子
3. standard-version，更新 changeLog.md 以及 package.json 的 version

## 构建部署/版本控制

> 基于git和jenkins

### 开发

1. 开发完成后，提交代码到远程仓库，远程仓库有三个基本分支（其他的都是功能分支），分别是beta、preview、master
2. 一般本地开发完成会把自己的功能分支合并到beta分支提交上去测试
3. 测试没问题就合并到主分支，然后执行release命令生成修改版本号以及生成changeLog
4. 最后push到远程分支

### 构建部署

1. 主要基于jenkins构建
2. master分支可以分为三个环境：beta、preview、prod环境，非master只能发布beta、preview环境
3. master分支需要根据tag进行发布（可以根据tag进行回滚），非master分支会根据分支进行发布
4. 公司项目很多，构建很频繁，而且构建项目的时候非常占用cpu和内存，单独一个服务器用来jenkins构建
5. 在构建的时候，会先执行构建的项目里面的构建脚本，构建完成后，将构建好的文件压缩，会通过scp命令上传到公司的cdn服务器
6. 然后添加对应的nginx配置即可

## 前端工程化

工程化能够方便管理、专注开发、快速迭代。

从项目规范化管理：git 提交、代码风格统一

### git 工作流

代码运行环境：本地开发环境、日常开发环境、预发测试环境、线上生成环境
git 分支：功能分支 feature、开发分支 dev、预发分支 release、生产分支 master

所有分支都是基于 master 分支检出：

- master：保护分支
- realse，开发完成的分支通过审批合并到 master
- feature，功能分支，用于开发功能
- dev，开发分支，用于日常开发环境，主要用于日常测试
- hotfix，bug 紧急修复分支，直接合并到 master 分支

### 代码风格统一

通过 eslint、stylelint、prettier，统一项目代码风格
增加 husky 配置 git 钩子，可以在提交代码前对项目进行校验和对 git 操作进行管理。

## 部署

使用 jenkins 自动部署前端代码，高部署的速度和效率

## 参考

1. [小团队前端工程化](https://juejin.cn/post/6867861517603438605)
