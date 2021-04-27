# Git 学习笔记

## Git 用户名和邮箱配置
安装完 Git 后一般要配置用户名和邮箱，以便在每次提交中记录下来，方便查找每次提交的用户。Git 的配置一共有三个级别：system（系统级）、global（用户级）、local（版本库）。system 的配置整个系统只有一个，global 的配置每个账号只有一个，local 的配置存在于 Git 版本库中，可以对不同的版本库配置不同的 local 信息。这三个级别的配置是逐层覆盖的关系，当用户提交修改时，首先查找 system 配置，其次查找 global 配置，最后查找 local 配置，逐层查找的过程中，若查询到配置信息，则会覆盖上一层配置，记录在提交记录中。
### system 配置
```shell
git config --system user.name "username"
git config --system user.email user@email.com
```
### global 配置
```shell
git config --global user.name "username"
git config --global user.email user@email.com
```
### local 配置
```shell
git config --local user.name "username"
git config --local user.email user@email.com
```
当有多个账号信息时，为了区分不同账户提交的记录。可以配置 global 为常用的用户和邮箱信息。对于不常用的，可以在对应的版本库里配置单独的用户和邮箱信息。