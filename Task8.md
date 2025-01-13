### Установите пакет samba
`apt-get install -y samba samba-client`

### ЧТо такое побщая папка, зачем оно может быть нужно?
Папка на сервере, которая видна другим пользователям. Нужна для совместного использования файлов и обмена ими между
разными ОС.

### Создайте общую папку без пароля с правами только на чтение файлов
```
mkdir /srv/samba/public
sudo chmod 755 /srv/samba/public
```
Записываем в файл `/etc/samba/smb.conf` :
```
[global]
  workgroup = WORKGROUP
  security = user
  map to guest = bad user
  wins support = no
  dns proxy = no

[public]
  comment = Public Readonly
  path = /srv/samba/public
  public = yes
  force user = nobody
  browsable = yes
  writable = no
```
После каждого изменения файла перезапускаем службу smb

### Создайте общую папку с паролем с правами на чтение и запись
Создаем пользователя sambauser с паролем passwd
```
smbpasswd -a sambauser
mkdir /srv/samba/private
sudo chmod 770 /srv/samba/private
```
В `/etc/samba/smb.conf` :
```
[private]
  comment = Password for Read and Write
  path = /srv/samba/private
  valid users = username
  guest ok = no
  browsable = yes
  writable = yes
```

### Создайте общую папку с доступом для какой-то группы с полными правами
То же самое, но дополнительно создаем группу smbgrp
```
sudo usermod -aG smbgrp sambauser
mkdir /srv/samba/group_share
sudo chown root:smbgrp /srv/samba/group_share
sudo chmod 770 /srv/samba/group_share
```
В `/etc/samba/smb.conf` :
```
[groupShare]
  path = /srv/samba/group_share
  valid users = @smbgroup
  guest ok = no
  browsable = yes
  writable = yes
```

### Создайте общую папку в которой у одной группы будет полный доступ, а у другой только доступ на чтение. Третья группа не должна иметь к ней доступа
Создаем группы fullaccess, readonly, noaccess и пользователей user1, user2 (и задаем им пароли). 
Добавляем в группу с полными правами sambauser'а, с правами на чтение - user2, 
и без прав - user1.

Создаем папку и настраиваем владельца:
```
sudo mkdir /srv/samba/mixed_access
sudo chown root:fullaccess /srv/samba/mixed_access
sudo chmod 770 /srv/samba/mixed_access
```
В `/etc/samba/smb.conf` :
```
[mixedAccess]
  comment = pupupu
  path = /srv/samba/mixed_access
  valid users = @fullaccess @readonly
  guest ok = no
  browsable = yes
  writable = no
```

!(images/image35.png)[image35.png]
