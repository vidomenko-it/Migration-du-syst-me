---
title: "Migration du système vers un disque RAM à la volée."
categories:
  - Blog
tags:
  - vm RAM
  - tmpfs
---

#  4. Migration du système vers un disque RAM à la volée. 
## 1.1. Créer des répertoires pour l'ancien root / :
```
mkdir –p /mnt/oldroot
```
## 1.2. 
```
mount --bind / /mnt/oldroot
```
## 1.3.
```
mkdir –p /mnt/ramdisk
```
## 1.4. Créer un système de fichiers en mémoire:
```
mount -t tmpfs -o size=4G tmpfs /mnt/ramdisk
```
## 1.5. Copier le système de fichiers racine:
```
rsync -aAXv /* /mnt/ramdisk --exclude=/mnt --exclude=/proc --exclude=/sys --exclude=/tmp --exclude=/dev --exclude=/run --exclude=/media --exclude=/swapfile
```
## 1.6. Monter des répertoires système importants sur un disque RAM:
```
cd /mnt/ramdisk
mkdir -p dev proc sys run
mount --bind /dev /mnt/ramdisk/dev
mount --bind /proc /mnt/ramdisk/proc
mount --bind /sys /mnt/ramdisk/sys
mount --bind /run /mnt/ramdisk/run
```
## 1.7. Changer le système de fichiers racine (chroot)
```
chroot /mnt/ramdisk /bin/bash
```
## 1.8. Vérifiez si le disque est utilisé:
**Avant de déconnecter, assurez-vous que le disque ou ses partitions ne sont plus occupés:**
```
lsof | grep /dev/sda
```
**Si quelque chose utilise le disque, vous devez mettre fin à ces processus ou démonter les partitions.**
## 1.9. Démonter toutes les partitions du disque:
**Utilisez umount pour démonter les partitions. Par exemple:**
```
umount /dev/sda1
```
**Si la partition est montée sur plusieurs points, utilisez l'option -l (lazy):**
```
umount -l /dev/sda1
```
## 1.10. Arrêter d'accéder au disque (si le disque n'est plus nécessaire):
**Pour ce faire, utilisez la commande :**
```
echo 1 &gt; /sys/block/sda/device/delete
```
**Cela désactivera le périphérique /dev/sda au niveau du noyau. Le lecteur ne sera plus visible sur le système.**
## 1.11. Vérifiez le statut:
**Assurez-vous que le lecteur n’apparaît plus dans la liste des périphériques :**
```
lsblk
```
#ITEnterprise #ContinuitéActivité #InfrastructureSécurisée
