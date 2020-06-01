# Домашнее Задание № 4 ZFS
## Подготовка стенда 
  Стенд мной брался из 3го домашнего задания, мной был отредактирован файл Vagrant файл, где я оставил всего два диска с одинаковыми размерами. Чтобы избежать ошибки ""rsync" could not be found on your PATH. Make sure that rsync is properly installed on your system and available on the PATH.", следуя руководству по ссылке https://qna.habr.com/q/271364, необходимо установить плагин vagrant-vbguest:

    vagrant plugin install vagrant-vbguest

и дописать в Vagrant файл:

    config.vm.synced_folder ".", "/vagrant", type: "virtualbox"

## Определение алгоритма с наилучшим сжатием
Первым делом установим необходимые программы...(займёт некоторое время)

    yum install -y -q http://download.zfsonlinux.org/epel/zfs-release.el7_5.noarch.rpm
    yum-config-manager --disable zfs
    yum-config-manager --enable zfs-kmod
    yum install zfs wget -y
    /sbin/modprobe zfs

Создадим mirror pool из дисков sdb и sdc

    zpool create mypool mirror /dev/sdb /dev/sdc

Создадим файловые системы с разным сжатием

    zfs create -o compression=off mypool/dir_off
    zfs create -o compression=lzjb mypool/dir_lzjb
    zfs create -o compression=gzip mypool/dir_gzip
    zfs create -o compression=zle mypool/dir_zle

Скачаем для теста ядро linux и распакуем его в разные файловые системы (займет много времени на медленных дисках)

    wget -q https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.6.14.tar.xz
    tar -xf linux-5.6.14.tar.xz -C /mypool/dir_off
    tar -xf linux-5.6.14.tar.xz -C /mypool/dir_gzip
    tar -xf linux-5.6.14.tar.xz -C /mypool/dir_lzjb
    tar -xf linux-5.6.14.tar.xz -C /mypool/dir_zle

Сравним используемый объем после распаковки:

    zfs list

Вывод комманды:

    NAME              USED  AVAIL  REFER  MOUNTPOINT
    mypool           2.50G  7.13G    28K  /mypool
    mypool/dir_gzip   242M  7.13G   242M  /mypool/dir_gzip
    mypool/dir_lzjb   417M  7.13G   417M  /mypool/dir_lzjb
    mypool/dir_off    988M  7.13G   988M  /mypool/dir_off
    mypool/dir_zle    911M  7.13G   911M  /mypool/dir_zle


## Определяем настройки пула
Скачиваем архив с google drive

    wget -q --load-cookies /tmp/cookies.txt "https://drive.google.com/uc?export=download&confirm=$(wget --quiet --save-cookies /tmp/cookies.txt --keep-session-cookies --no-check-certificate 'https://drive.google.com/uc?export=download&id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg' -O- | sed -rn 's$.*confirm=([0-9A-Za-z_]+).*$\1\n$p')&id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg" -O zfs_task1.tar.gz && rm -rf /tmp/cookies.txt
    tar xf zfs_task1.tar.gz

Пробуем импортировать пул

    zpool import -d /home/vagrant/zpoolexport/

Из-за разных версий ZFS какие то опции недоступны. Смонтируем в режиме read-only...

    zpool import otus -o readonly=on -d /home/vagrant/zpoolexport/
    zpool list otus

Вывод комманд 

    NAME   SIZE  ALLOC   FREE  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
    otus   480M  2.11M   478M         -     0%     0%  1.00x  ONLINE  -

## Найти сообщение от преподавателей

Скачиваем файл..

    wget -q --load-cookies /tmp/cookies.txt "https://drive.google.com/uc?export=download&confirm=$(wget --quiet --save-cookies /tmp/cookies.txt --keep-session-cookies --no-check-certificate 'https://drive.google.com/uc?export=download&id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG' -O- | sed -rn 's$.*confirm=([0-9A-Za-z_]+).*$\1\n$p')&id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG" -O otus_task2.file && rm -rf /tmp/cookies.txt

Восстановим полученный файл..

    cat otus_task2.file | sudo zfs recv -F mypool/task3

Найдем файл и выведем сообщение

    cat `find /mypool/task3/ -name "secret_message"`

Вывод команды: 

    https://github.com/sindresorhus/awesome
    
При выполнении данного домашнего задания помимо методички я пользовался поисками информации в поисковой системе Google, а так же в мануале по ссылке:<br>
https://www.symmcom.com/docs/how-tos/storages/how-to-install-zfs-on-centos-7   
