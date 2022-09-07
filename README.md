<h3>### NETARCHITECTURE ###</h3>

<h4>Описание домашнего задания</h4>

<p>Vagrantfile - для стенда урока 9 - Network</p>

<h4>Дано</h4>

<p>https://github.com/erlong15/otus-linux/tree/network<br />
(ветка network)<br />
Vagrantfile с начальным построением сети</p>

<ul>
<li>inetRouter</li>
<li>centralRouter</li>
<li>centralServer</li>
</ul>
<p>тестировалось на virtualbox</p>

<h4>Планируемая архитектура</h4>

<p>Построить следующую архитектуру:<br />
Сеть office1
<ul>
<li>192.168.2.0/26 - dev</li>
<li>192.168.2.64/26 - test servers</li>
<li>192.168.2.128/26 - managers</li>
<li>192.168.2.192/26 - office hardware</li>
</ul></p>
<p>Сеть office2
<ul>
<li>192.168.1.0/25 - dev</li>
<li>192.168.1.128/26 - test servers</li>
<li>192.168.1.192/26 - office hardware</li>
</ul></p>
<p>Сеть central
<ul>
<li>192.168.0.0/28 - directors</li>
<li>192.168.0.32/28 - office hardware</li>
<li>192.168.0.64/26 - wifi</li>
</ul>
...</p>

<p>Office1 ---\
<ul><li>----> Central --IRouter --> internet</li></ul>
Office2----/
...</p>

<p>Итого должны получится следующие сервера
<ul>
<li>inetRouter</li>
<li>centralRouter</li>
<li>office1Router</li>
<li>office2Router</li>
<li>centralServer</li>
<li>office1Server</li>
<li>office2Server</li>
</ul></p>

<h4>Теоретическая часть</h4>
<ul>
<li>Найти свободные подсети</li>
<li>Посчитать сколько узлов в каждой подсети, включая свободные</li>
<li>Указать broadcast адрес для каждой подсети</li>
<li>проверить нет ли ошибок при разбиении</li>
</ul>

<h4>Практическая часть</h4>
<ul>
<li>Соединить офисы в сеть согласно схеме и настроить роутинг</li>
<li>Все сервера и роутеры должны ходить в инет черз inetRouter</li>
<li>Все сервера должны видеть друг друга</li>
<li>у всех новых серверов отключить дефолт на нат (eth0), который вагрант поднимает для связи</li>
<li>при нехватке сетевых интервейсов добавить по несколько адресов на интерфейс</li>
</ul>
<p>Формат сдачи ДЗ - vagrant + ansible</p>




<h4>1. Создаём виртуальные машины backup и client</h4>

<p>В домашней директории создадим директорию backup, в котором будут храниться настройки виртуальных машин backup и client:</p>

<pre>[user@localhost otus]$ mkdir ./backup
[user@localhost otus]$</pre>

<p>Перейдём в директорию backup:</p>

<pre>[user@localhost otus]$ cd ./backup/
[user@localhost backup]$</pre>

<p>Создадим файл Vagrantfile:</p>

<pre>[user@localhost backup]$ vi ./Vagrantfile</pre>

<p>Заполним следующим содержимым:</p>

<pre># -*- mode: ruby -*-
# vi: set ft=ruby :

hosts = [
  {
  :name => "backup",
  :box_name => "centos/7",
  :ip_addr => "192.168.50.160",
  :disks => {
    :sata1 => {
      :dfile => './disks/sata_backup1.vdi',
      :size => 2048,
      :port => 1
      }
    }
  },
  {
  :name => "client",
  :box_name => "centos/7",
  :ip_addr => "192.168.50.150",
  :disks => {}
  }
]

Vagrant.configure("2") do |config|
  hosts.each do |opts|
    config.vm.define opts[:name] do |config|
      config.vm.box = opts[:box_name]
      config.vm.hostname = opts[:name].to_s
#      config.vm.opts[:name] = "%s" % opts[:name]
      config.vm.network "private_network", ip: opts[:ip_addr]
      config.vm.provider :virtualbox do |vb|
        vb.name = opts[:name]
        vb.customize ["modifyvm", :id, "--memory", "512"]
        needsController = false
        opts[:disks].each do |dname, dconf|
          unless File.exist?(dconf[:dfile])
            vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
            needsController =  true
          end
        end
        if needsController == true
          vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
          opts[:disks].each do |dname, dconf|
            vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
          end
        end
      end
      config.vm.provision "shell", inline: <<-SHELL
        mkdir -p ~root/.ssh; cp ~vagrant/.ssh/auth* ~root/.ssh
        sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
        systemctl restart sshd
      SHELL
#      if opts[:name] == hosts.last[:name]
#        config.vm.provision "ansible" do |ansible|
#          ansible.playbook = "playbook.yml"
#          ansible.inventory_path = "hosts"
#          ansible.host_key_checking = "false"
#          ansible.limit = "all"
#        end
#      end
    end
  end
end
</pre>

<p>Запустим эти виртуальные машины:</p>

<pre>[user@localhost backup]$ vagrant up</pre>

<p>Проверим состояние созданных и запущенных машин:</p>

<pre>[user@localhost backup]$ vagrant status
Current machine states:

backup                       running (virtualbox)
client                       running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
[user@localhost backup]$</pre>

<p>Заходим на сервер backup:</p>

<pre>[user@localhost backup]$ vagrant ssh backup
[vagrant@backup ~]$</pre>

<p>Заходим под правами root:</p>

<pre>[vagrant@backup ~]$ sudo -i
[root@backup ~]#</pre>

<p>Подключаем EPEL репозиторий с дополнительными пакетами:</p>

<pre>[root@backup ~]# yum install -y epel-release
...
Installed:
  epel-release.noarch 0:7-11

Complete!
[root@backup ~]#</pre>
