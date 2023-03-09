# Домашнее задание к занятию «Файловые системы»


## Задание

1. Узнайте о [sparse-файлах](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных).

### Ответ 

**РазрЕженный файл - использующий механизм zero detect, то есть не записывает нули, а хранит в метаданных информацию о том, где эти нули должны быть.**


2. Могут ли файлы, являющиеся жёсткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?

### Ответ 

**Не могут, потому что по сути это один и тот же файл, имеющий одинаковое описание и inode, если посмотреть stat. Просто как бы "расположенный в разных местах". Права можно поменять на симлинк для этого файла, если каким-то пользователям нужно выдать его, например, без права редактирования**

3. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```ruby
    path_to_disk_folder = './disks'

    host_params = {
        'disk_size' => 2560,
        'disks'=>[1, 2],
        'cpus'=>2,
        'memory'=>2048,
        'hostname'=>'sysadm-fs',
        'vm_name'=>'sysadm-fs'
    }
    Vagrant.configure("2") do |config|
        config.vm.box = "bento/ubuntu-20.04"
        config.vm.hostname=host_params['hostname']
        config.vm.provider :virtualbox do |v|

            v.name=host_params['vm_name']
            v.cpus=host_params['cpus']
            v.memory=host_params['memory']

            host_params['disks'].each do |disk|
                file_to_disk=path_to_disk_folder+'/disk'+disk.to_s+'.vdi'
                unless File.exist?(file_to_disk)
                    v.customize ['createmedium', '--filename', file_to_disk, '--size', host_params['disk_size']]
                end
                v.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', disk.to_s, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
            end
        end
        config.vm.network "private_network", type: "dhcp"
    end
    ```

    Эта конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2,5 Гб.

### Ответ 

**Сделано**

4. Используя `fdisk`, разбейте первый диск на два раздела: 2 Гб и оставшееся пространство.

### Ответ 

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_11.png "1")

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_3.png "2")

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_4.png "3")

5. Используя `sfdisk`, перенесите эту таблицу разделов на второй диск.

### Ответ 

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_5.png "4")

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_6.png "5")

6. Соберите `mdadm` RAID1 на паре разделов 2 Гб.

### Ответ 

**Ой, ну я сначала сделала 7 пункт, потом 6, получается:**

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_7.png "5")

7. Соберите `mdadm` RAID0 на второй паре маленьких разделов.

### Ответ 

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_8.png "6")

8. Создайте два независимых PV на получившихся md-устройствах.

### Ответ 

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_9.png "7")

9. Создайте общую volume-group на этих двух PV.

### Ответ 

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_10.png "8")

10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.

### Ответ 

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_12.png "9")

11. Создайте `mkfs.ext4` ФС на получившемся LV.

### Ответ 

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_13.png "10")

12. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.

### Ответ 

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_14.png "11")

13. Поместите туда тестовый файл, например, `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.

### Ответ 

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_15.png "12")

14. Прикрепите вывод `lsblk`.

### Ответ 

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_16.png "13")

15. Протестируйте целостность файла:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```

### Ответ 

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_17.png "14")

16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.

### Ответ 

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_18.png "15")

17. Сделайте `--fail` на устройство в вашем RAID1 md.

### Ответ 

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_19.png "16")

18. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.

### Ответ 

![Скрин](https://github.com/Jlljully/File_systems/blob/main/Screenshot_20.png "17")

19. Протестируйте целостность файла — он должен быть доступен несмотря на «сбойный» диск:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```

### Ответ 

**Выше**

20. Погасите тестовый хост — `vagrant destroy`.

### Ответ 




