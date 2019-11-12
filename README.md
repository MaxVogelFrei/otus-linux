# Домашнее задание 2  

## Работа с mdadm

* добавить в Vagrantfile еще дисков
* собрать R0/R5/R10 на выбор
* сломать/починить raid
* прописать собранный рейд в конф, чтобы рейд собирался при загрузке
* создать GPT раздел и 5 партиций
* доп. задание - Vagrantfile, который сразу собирает систему с подключенным рейдом

## Процесс решения

### Добавление дисков

В имеющийся Vagrantfile в раздел "disks" добавляю строки

                :sata5 => {
                        :dfile => './sata5.vdi',
                        :size => 250, # Megabytes
                        :port => 5
                },
                :sata6 => {
                        :dfile => './sata6.vdi',
                        :size => 250, # Megabytes
                        :port => 6
						
### собрать R0/R5/R10 на выбор
собираю raid 10 из 6 дисков

		mdadm --zero-superblock --force /dev/sd{b,c,d,e,f,g}
		mdadm --create --verbose /dev/md0 -l 10 -n 6 /dev/sd{b,c,d,e,f,g}

### сломать/починить raid
искуственно делаю ошибочными диски sde sdb

		mdadm /dev/md0 --fail /dev/sde
		mdadm /dev/md0 --fail /dev/sdb
		sleep 2
		mdadm /dev/md0 --remove /dev/sdb
		mdadm /dev/md0 --remove /dev/sde
		mdadm /dev/md0 --add /dev/sdb
		mdadm /dev/md0 --add /dev/sde

### прописать собранный рейд в конф, чтобы рейд собирался при загрузке
создаю папку и файл для mdadm

		mkdir /etc/mdadm
		touch /etc/mdadm/mdadm.conf
		echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
		mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf

### создать GPT раздел и 5 партиций

		parted -s /dev/md0 mklabel gpt
		parted /dev/md0 mkpart primary ext4 0% 20%
		parted /dev/md0 mkpart primary ext4 20% 40%
		parted /dev/md0 mkpart primary ext4 40% 60%
		parted /dev/md0 mkpart primary ext4 60% 80%
		parted /dev/md0 mkpart primary ext4 80% 100%
		for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
		mkdir -p /raid/part{1,2,3,4,5}
		for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done
		echo "/dev/md0p1      /raid/part1     ext4    defaults    1 2" >> /etc/fstab
		echo "/dev/md0p2      /raid/part2     ext4    defaults    1 2" >> /etc/fstab
		echo "/dev/md0p3      /raid/part3     ext4    defaults    1 2" >> /etc/fstab
		echo "/dev/md0p4      /raid/part4     ext4    defaults    1 2" >> /etc/fstab
		echo "/dev/md0p5      /raid/part5     ext4    defaults    1 2" >> /etc/fstab

### доп. задание - Vagrantfile, который сразу собирает систему с подключенным рейдом

в раздел "box.vm.provision "shell", inline:" vagrantfile добавляем приведенные выше строки по порядку
