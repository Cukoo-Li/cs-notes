# ArchLinux

## 好用的Linux命令

```bash
|grep xxx						# 在结果中筛选包含xxx结果并显示
```

## Pacman包管理

Pacman是Arch Linux的包管理器，它用于安装、删除、查询软件等。

```bash
sudo pacman -S package_name     # 安装软件包
sudo pacman -Ss xxx				# 搜索包含xxx的软件
sudo pacman -Sc					# 删除安装包缓存
sudo pacman -Su					# 更新所有软件
sudo pacman -Syy                # 更新系统（软件源）
sudo pacman -Syyu               # 更新系统（软件源）并更新所有软件 yy标记强制更新
sudo pacman -R package_name     # 删除软件包
sudo pacman -Rs package_name    # 删除软件包，及其所有没有被其他已安装软件包使用的依赖包
sudo pacman -Rns package_name	# 删除软件包，及其所有没有被其他已安装软件包使用的依赖包和全局配置
sudo pacman -Q					# 查询本地软件
sudo pacman -Qe					# 查询本地自己安装的软件
sudo pacman -Qdt                # 查询孤立包 d标记依赖包 t标记不需要的包
sudo pacman -Rs $(pacman -Qtdq) # 删除孤立软件包 q表示不显示版本号
sudo pacman -Fy                 # 更新命令查询文件列表数据库
sudo pacman -F xxx              # 当不知道某个命令属于哪个包时，用来查询某个xxx命令属于哪个包
```

## Aur

yay和paru是两个常用的Aur助手，其使用方法与pacman类似

## 其他

### 更改grub配置

```bash
sudo -e /etc/default/grub
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

