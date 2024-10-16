---
title: git 签名从 GPG 迁移到 ssh
tags:
  - git
  - ssh
  - gpg
date: 2024-10-16
draft: true
---

最近把 git 签名从 gpg 迁移到 ssh。具体的配置可以参考:

- [gitlab](https://docs.gitlab.com/ee/user/project/repository/signed_commits/ssh.html)
- [github](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification#ssh-commit-signature-verification)

> 配置推荐参考 gitlab 的文档，设置流程更清晰。

其他参考信息
- [git](https://github.com/git/git/pull/1041)

以前使用 gpg 签名的时候，感觉设置比较繁琐，迁移到 ssh 之后，一个 ssh 密钥就可以搞定，感觉方便很多。
