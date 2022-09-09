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

<h4>1. Теоретическая часть</h4>

<p>В теоретической части нам необходимо продумать топологию сети, а также:
<ul>
<li>Найти свободные подсети</li>
<li>Посчитать количество узлов в каждой подсети, включая свободные</li>
<li>Указать Broadcast-адрес для каждой подсети</li>
<li>Проверить, нет ли ошибок при разбиении</li>
</ul>
Первым шагом мы рассмотрим все сети, указанные в задании.<br />
Посчитаем для них количество узлов, найдём Broadcast-адрес,<br />
проверим, нет ли ошибок при разбиении.</p>

<p>У нас есть сеть directors 192.168.0.0/28<br />
192.168.0.0 — это сама сеть, 28 — это маска. Маска показывает нам, границы сети 192.168.0.0. Маска может быть записана в 2-х видах:<br />
1) /28<br />
2) 255.255.255.240</p>

<p>Пример перевода маски /28 в формат 255.255.255.240:</p>

<p>Маска Ipv4-адреса — это 4 октета, т.е. 4 блока по 8 цифр (1 или 0).<br />
/28 — это 28 единиц с начала маски:<br />
11111111.11111111.11111111.11110000<br />
Всегда при разбиении сетей, после знака / указывается количество
единиц с начала маски.</p>
<p>11111111.11111111.11111111.11110000 — это двоичный код маски,<br />
если мы переведем данное значение в деситеричную систему
счисления, то получим 255.255.255.240</p>

<p>Далее посчитаем количество устройств в сети:<br />
Количество устройств в сети рассчитывается по формуле =<br />
 (32−маска)<br />
2           − 2<br />
Таким образом, количество устройств для подсети /28 будет =<br />
 (32−28)<br />
2        − 2 = 16 − 2 = 14</p>

<p>Цифра 2 вычитается, так как:<br />
● Первый адрес (192.168.0.0) — это наименование подсети, его нельзя задать устройству<br />
● Последний адрес (192.168.0.15) — это всегда broadcast-адрес.<br />
Broadcast-адрес нужен для рассылки всем устройствам сети.</p>

<p>Таким образом мы можем сформировать таблицу топологии нашей сети:</p>

<table>
<tr>
    <th>Name</th>
    <th>Network</th>
    <th>Netmask</th>
    <th>N</th>
    <th>Hostmin</th>
    <th>Hostmax</th>
    <th>Broadcast</th>
</tr>
<tr>
    <th colspan="7">Central Network</th>
</tr>
<tr>
    <td align=center>directors</td>
    <td align=center>192.168.0.0/28</td>
    <td align=center>255.255.255.240</td>
    <td align=center>14</td>
    <td align=center>192.168.0.1</td>
    <td align=center>192.168.0.14</td>
    <td align=center>192.168.0.15</td>
</tr>
<tr>
    <td align=center>office hardware</td>
    <td align=center>192.168.0.32/28</td>
    <td align=center>255.255.255.240</td>
    <td align=center>14</td>
    <td align=center>192.168.0.33</td>
    <td align=center>192.168.0.46</td>
    <td align=center>192.168.0.47</td>
</tr>
<tr>
    <td align=center>wifi</td>
    <td align=center>192.168.0.64/26</td>
    <td align=center>255.255.255.192</td>
    <td align=center>62</td>
    <td align=center>192.168.0.65</td>
    <td align=center>192.168.0.126</td>
    <td align=center>192.168.0.127</td>
</tr>
<tr>
    <th colspan="7">Office1 Network</th>
</tr>
<tr>
    <td align=center>dev</td>
    <td align=center>192.168.2.0/26</td>
    <td align=center>255.255.255.192</td>
    <td align=center>62</td>
    <td align=center>192.168.2.1</td>
    <td align=center>192.168.2.62</td>
    <td align=center>192.168.2.63</td>
</tr>
<tr>
    <td align=center>test servers</td>
    <td align=center>192.168.2.64/26</td>
    <td align=center>255.255.255.192</td>
    <td align=center>62</td>
    <td align=center>192.168.2.65</td>
    <td align=center>192.168.2.126</td>
    <td align=center>192.168.2.127</td>
</tr>
<tr>
    <td align=center>managers</td>
    <td align=center>192.168.2.128/26</td>
    <td align=center>255.255.255.192</td>
    <td align=center>62</td>
    <td align=center>192.168.2.129</td>
    <td align=center>192.168.2.190</td>
    <td align=center>192.168.2.191</td>
</tr>
<tr>
    <td align=center>office hardware</td>
    <td align=center>192.168.2.192/26</td>
    <td align=center>255.255.255.192</td>
    <td align=center>62</td>
    <td align=center>192.168.2.193</td>
    <td align=center>192.168.2.254</td>
    <td align=center>192.168.2.255</td>
</tr>
<tr>
    <th colspan="7">Office2 Network</th>
</tr>
<tr>
    <td align=center>dev</td>
    <td align=center>192.168.1.0/25</td>
    <td align=center>255.255.255.128</td>
    <td align=center>126</td>
    <td align=center>192.168.1.1</td>
    <td align=center>192.168.1.126</td>
    <td align=center>192.168.1.127</td>
</tr>
<tr>
    <td align=center>test servers</td>
    <td align=center>192.168.1.128/26</td>
    <td align=center>255.255.255.192</td>
    <td align=center>62</td>
    <td align=center>192.168.1.129</td>
    <td align=center>192.168.1.190</td>
    <td align=center>192.168.1.191</td>
</tr>
<tr>
    <td align=center>office hardware</td>
    <td align=center>192.168.1.192/26</td>
    <td align=center>255.255.255.192</td>
    <td align=center>62</td>
    <td align=center>192.168.1.193</td>
    <td align=center>192.168.1.254</td>
    <td align=center>192.168.1.255</td>
</tr>
<tr>
    <th colspan="7">InetRouter - CentralRouter Network</th>
</tr>
<tr>
    <td align=center>inet-central</td>
    <td align=center>192.168.255.0/30</td>
    <td align=center>255.255.255.252</td>
    <td align=center>2</td>
    <td align=center>192.168.255.1</td>
    <td align=center>192.168.255.2</td>
    <td align=center>192.168.255.3</td>
</tr>
</table>

<p>После создания таблицы топологии, мы можем заметить, что ошибок в задании нет, также мы сразу видим следующие свободные сети:</p>

<p>192.168.0.16/28<br />
192.168.0.48/28<br />
192.168.0.128/25</p>

<p>192.168.255.4/30<br />
192.168.255.8/29<br />
192.168.255.16/28<br />
192.168.255.32/27<br />
192.168.255.64/26<br />
192.168.255.128/25</p>

<p>Сформируем таблицу топологии свободных подсетей:</p>

<table>
<tr>
    <th>Name</th>
    <th>Network</th>
    <th>Netmask</th>
    <th>N</th>
    <th>Hostmin</th>
    <th>Hostmax</th>
    <th>Broadcast</th>
</tr>
<tr>
    <th colspan="7">Central Network</th>
</tr>
<tr>
    <td align=center></td>
    <td align=center>192.168.0.16/28</td>
    <td align=center>255.255.255.240</td>
    <td align=center>14</td>
    <td align=center>192.168.0.17</td>
    <td align=center>192.168.0.30</td>
    <td align=center>192.168.0.31</td>
</tr>
<tr>
    <td align=center></td>
    <td align=center>192.168.0.48/28</td>
    <td align=center>255.255.255.240</td>
    <td align=center>14</td>
    <td align=center>192.168.0.49</td>
    <td align=center>192.168.0.62</td>
    <td align=center>192.168.0.63</td>
</tr>
<tr>
    <td align=center></td>
    <td align=center>192.168.0.128/25</td>
    <td align=center>255.255.255.128</td>
    <td align=center>126</td>
    <td align=center>192.168.0.129</td>
    <td align=center>192.168.0.254</td>
    <td align=center>192.168.0.255</td>
</tr>
<tr>
    <th colspan="7">InetRouter - CentralRouter Network</th>
</tr>
<tr>
    <td align=center></td>
    <td align=center>192.168.255.4/30</td>
    <td align=center>255.255.255.252</td>
    <td align=center>2</td>
    <td align=center>192.168.255.5</td>
    <td align=center>192.168.255.6</td>
    <td align=center>192.168.255.7</td>
</tr>
<tr>
    <td align=center></td>
    <td align=center>192.168.255.8/29</td>
    <td align=center>255.255.255.248</td>
    <td align=center>6</td>
    <td align=center>192.168.255.9</td>
    <td align=center>192.168.255.14</td>
    <td align=center>192.168.255.15</td>
</tr>
<tr>
    <td align=center></td>
    <td align=center>192.168.255.16/28</td>
    <td align=center>255.255.255.240</td>
    <td align=center>14</td>
    <td align=center>192.168.255.17</td>
    <td align=center>192.168.255.30</td>
    <td align=center>192.168.255.31</td>
</tr>
<tr>
    <td align=center></td>
    <td align=center>192.168.255.32/27</td>
    <td align=center>255.255.255.224</td>
    <td align=center>30</td>
    <td align=center>192.168.255.33</td>
    <td align=center>192.168.255.62</td>
    <td align=center>192.168.255.63</td>
</tr>
<tr>
    <td align=center></td>
    <td align=center>192.168.255.64/26</td>
    <td align=center>255.255.255.192</td>
    <td align=center>62</td>
    <td align=center>192.168.255.65</td>
    <td align=center>192.168.255.126</td>
    <td align=center>192.168.255.127</td>
</tr>
<tr>
    <td align=center></td>
    <td align=center>192.168.255.128/25</td>
    <td align=center>255.255.255.128</td>
    <td align=center>126</td>
    <td align=center>192.168.255.129</td>
    <td align=center>192.168.255.254</td>
    <td align=center>192.168.255.255</td>
</tr>
</table>

<h4>Практическая часть</h4>

<p>Изучив таблицу топологии сети и Vagrant-стенд из задания, мы можем построить полную схему сети:</p>



<p>Знак облака означает сеть, которую необходимо будет настроить на сервере.<br />
Значки роутеров и серверов означают хосты, которые нам нужно будет создать.</p>

<p>На схеме, мы сразу можем увидеть, что нам потребуется создать дополнительно 2 сети (на схеме обозначены полужирными фиолетовыми линиями):<br />
Для соединения office1Router c centralRouter — 192.168.255.8/30<br />
Для соединения office2Router c centralRouter — 192.168.255.4/30<br />
На основании этой схемы мы получаем готовый список серверов. Для более подробного изучения, сделаем новые хосты с другими ОС.</p>

<table>
<tr>
    <th>Server</th>
    <th>IP and Bitmask</th>
    <th>OS</th>
</tr>
<tr>
    <td rowspan="2">inetRouter</td>
    <td>Default-NAT address VirtualBox</td>
    <td rowspan="2">CentOS 7</td>
</tr>
<tr>
    <td>192.168.255.1/30</td>
</tr>
<tr>
    <td rowspan="6">centralRouter</td>
    <td>192.168.255.2/30</td>
    <td rowspan="6">CentOS 7</td>
</tr>
<tr>
    <td>192.168.0.1/28</td>
    <td>192.168.0.33/28</td>
    <td>192.168.0.65/26</td>
    <td>192.168.255.9/30</td>
    <td>192.168.255.5/30</td>
</tr>
<tr>
    <td>centralServer</td>
    <td>192.168.0.2/28</td>
    <td>CentOS 7</td>
</tr>
<tr>
    <td rowspan="5">office1Router</td>
    <td>192.168.255.10/30</td>
    <td rowspan="5">Ubuntu 20</td>
</tr>
<tr>
    <td>192.168.2.1/26</td>
    <td>192.168.2.65/26</td>
    <td>192.168.2.129/26</td>
    <td>192.168.2.193/26</td>
</tr>
<tr>
    <td>office1Server</td>
    <td>192.168.2.130/26</td>
    <td>Ubuntu 20</td>
</tr>
<tr>
    <td rowspan="4">office2Router</td>
    <td>192.168.255.6/30</td>
    <td rowspan="4">Debian 11</td>
</tr>
<tr>
    <td>192.168.1.1/26</td>
    <td>192.168.1.129/26</td>
    <td>192.168.1.193/26</td>
</tr>
<tr>
    <td>office2Server</td>
    <td>192.168.1.2/26</td>
    <td>Debian 11</td>
</tr>
</table>

<p>В домашней директории создадим директорию netarchitecture, в котором будут храниться настройки виртуальных машин:</p>

<pre>[user@localhost otus]$ mkdir ./netarchitecture
[user@localhost otus]$</pre>

<p>Перейдём в директорию backup:</p>

<pre>[user@localhost otus]$ cd ./netarchitecture/
[user@localhost netarchitecture]$</pre>

<p>Скачаем Vagrantfile из репозитория<br />
https://github.com/erlong15/otus-linux/tree/network<br />
В inetRouter заменим версию CentOS/6 на CentOS/7, после
изменений мы получим следующий файл:</p>

<pre># -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :inetRouter => {
        :box_name => "centos/7",
        #:public => {:ip => '10.10.10.1', :adapter => 1},
        :net => [
                   {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                ]
  },
  :centralRouter => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                   {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {ip: '192.168.0.33', adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: "hw-net"},
                   {ip: '192.168.0.65', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "mgt-net"},
                ]
  },
  :centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {adapter: 3, auto_config: false, virtualbox__intnet: true},
                   {adapter: 4, auto_config: false, virtualbox__intnet: true},
                ]
  },
}
Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s
        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
        if boxconfig.key?(:public)
          box.vm.network "public_network", boxconfig[:public]
        end
        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
                cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL
        case boxname.to_s
        when "inetRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sysctl net.ipv4.conf.all.forwarding=1
            iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
            SHELL
        when "centralRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sysctl net.ipv4.conf.all.forwarding=1
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            SHELL
        when "centralServer"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            SHELL
        end
      end
  end
end</pre>

<p>Запустим эти виртуальные машины:</p>

<pre>[student@pv-homeworks1-10 netarchitecture]$ vagrant up</pre>

<p>Данный Vagrantfile развернет нам 3 хоста: inetRouter, centralRouter и centralServer:</p>

<pre>[student@pv-homeworks1-10 netarchitecture]$ vagrant status
Current machine states:

inetRouter                running (virtualbox)
centralRouter             running (virtualbox)
centralServer             running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
[student@pv-homeworks1-10 netarchitecture]$</pre>

<p>Исходя их схемы нам ещё потребуется развернуть 4 сервера:<br />
● office1Router<br />
● office1Server<br />
● office2Router<br />
● office2Server<br />
Опираясь на таблицу и схему мы можем дописать хосты в Vagrantfile:</p>

<pre># -*- mode: ruby -*-
# vi: set ft=ruby :

MACHINES = {
  :inetRouter => {
    :box_name => "centos/7",
    :vm_name => "inetRouter",
    #:public => {:ip => '10.10.10.1', :adapter => 1},
    :net => [
      {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
      {ip: '192.168.50.10', adapter: 8},
    ]
  },
  :centralRouter => {
    :box_name => "centos/7",
    :vm_name => "centralRouter",
    :net => [
      {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
      {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
      {ip: '192.168.0.33', adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: "hw-net"},
      {ip: '192.168.0.65', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "mgt-net"},
      {ip: '192.168.255.9', adapter: 6, netmask: "255.255.255.252", virtualbox__intnet: "office1-central"},
      {ip: '192.168.255.5', adapter: 7, netmask: "255.255.255.252", virtualbox__intnet: "office2-central"},
      {ip: '192.168.50.11', adapter: 8},
    ]
  },
  :centralServer => {
    :box_name => "centos/7",
    :vm_name => "centralServer",
    :net => [
      {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
      #{adapter: 3, auto_config: false, virtualbox__intnet: true},
      #{adapter: 4, auto_config: false, virtualbox__intnet: true},
      {ip: '192.168.50.12', adapter: 8},
    ]
  },
  :office1Router => {
    :box_name => "ubuntu/focal64",
    :vm_name => "office1Router",
    :net => [
      {ip: '192.168.255.10', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office1-central"},
      {ip: '192.168.2.1', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "dev1-net"},
      {ip: '192.168.2.65', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "test1-net"},
      {ip: '192.168.2.129', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "managers-net"},
      {ip: '192.168.2.193', adapter: 6, netmask: "255.255.255.192", virtualbox__intnet: "office1-net"},
	  {ip: '192.168.50.20', adapter: 8},
    ]
  },
  :office1Server => {
    :box_name => "ubuntu/focal64",
    :vm_name => "office1Server",
    :net => [
      {ip: '192.168.2.130', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "managers-net"},
      {ip: '192.168.50.21', adapter: 8},
    ]
  },
  :office2Router => {
    :box_name => "debian/bullseye64",
    :vm_name => "office2Router",
    :net => [
      {ip: '192.168.255.6', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office2-central"},
      {ip: '192.168.1.1', adapter: 3, netmask: "255.255.255.128", virtualbox__intnet: "dev2-net"},
      {ip: '192.168.1.129', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "test2-net"},
      {ip: '192.168.1.193', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "office2-net"},
	  {ip: '192.168.50.30', adapter: 8},
    ]
  },
  :office2Server => {
    :box_name => "debian/bullseye64",
    :vm_name => "office2Server",
    :net => [
      {ip: '192.168.1.2', adapter: 2, netmask: "255.255.255.128", virtualbox__intnet: "dev2-net"},
      {ip: '192.168.50.31', adapter: 8},
    ]
  },
}
Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s
      boxconfig[:net].each do |ipconf|
        box.vm.network "private_network", ipconf
      end
      if boxconfig.key?(:public)
        box.vm.network "public_network", boxconfig[:public]
      end
      box.vm.provision "shell", inline: <<-SHELL
        mkdir -p ~root/.ssh
        cp ~vagrant/.ssh/auth* ~root/.ssh
      SHELL
      case boxname.to_s
      when "inetRouter"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          sysctl net.ipv4.conf.all.forwarding=1
          iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
        SHELL
      when "centralRouter"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          sysctl net.ipv4.conf.all.forwarding=1
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          systemctl restart network
        SHELL
      when "centralServer"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          systemctl restart network
        SHELL
      end
#      if boxconfig[:vm_name] == "office2Server"
#        box.vm.provision "ansible" do |ansible|
#          ansible.playbook = "ansible/provision.yml"
#          ansible.inventory_path = "ansible/hosts"
#          ansible.host_key_checking = "false"
#          ansible.limit = "all"
#        end
#      end
    end
  end
end</pre>

<p>В данный Vagrantfile мы добавили информацию о 4 новых серверах, также к старым серверам добавили 2 интерфейса для соединения сетей офисов (в коде выделены полужирным).<br />
Дополнительно в коде добавили сетевые устройства из подсети 192.168.50.0/24 — они потребуются для настройки хостов с помощью Ansible.</p>

<p>Снова запустим эти виртуальные машины:</p>

<pre>[student@pv-homeworks1-10 netarchitecture]$ vagrant up</pre>

<p>Смотрим состояние запущенных виртуальных машин:</p>

<pre>[student@pv-homeworks1-10 netarchitecture]$ vagrant status
Current machine states:

inetRouter                running (virtualbox)
centralRouter             running (virtualbox)
centralServer             running (virtualbox)
office1Router             running (virtualbox)
office1Server             running (virtualbox)
office2Router             running (virtualbox)
office2Server             running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
[student@pv-homeworks1-10 netarchitecture]$</pre>

<p>После того, как все 7 серверов у нас развернуты, нам нужно настроить маршрутизацию и NAT таким образом, чтобы доступ в Интернет со всех хостов был через inetRouter и каждый сервер должен быть доступен с любого из 7 хостов.<br />
Часть настройки у нас уже выполнена, давайте рассмотрим подробнее команды из Vagrantfile.</p>

<h4>Настройка NAT</h4>







