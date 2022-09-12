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

<p>Сформируем таблицу топологии свободных подсетей, где также указаны количество хостов и broadcast адреса в каждой подсети:</p>

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

<img src="https://github.com/SergSha/netarchitecture/blob/main/infra.png" alt="infra.png" />

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
</tr>
<tr>
    <td>192.168.0.33/28</td>
</tr>
<tr>
    <td>192.168.0.65/26</td>
</tr>
<tr>
    <td>192.168.255.9/30</td>
</tr>
<tr>
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
</tr>
<tr>
    <td>192.168.2.65/26</td>
</tr>
<tr>
    <td>192.168.2.129/26</td>
</tr>
<tr>
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
</tr>
<tr>
    <td>192.168.1.129/26</td>
</tr>
<tr>
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

<pre>[user@localhost netarchitecture]$ vagrant up</pre>

<p>Данный Vagrantfile развернет нам 3 хоста: inetRouter, centralRouter и centralServer:</p>

<pre>[user@localhost netarchitecture]$ vagrant status
Current machine states:

inetRouter                running (virtualbox)
centralRouter             running (virtualbox)
centralServer             running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
[user@localhost netarchitecture]$</pre>

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
#      case boxname.to_s
#      when "inetRouter"
#        box.vm.provision "shell", run: "always", inline: <<-SHELL
#          sysctl net.ipv4.conf.all.forwarding=1
#          iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
#        SHELL
#      when "centralRouter"
#        box.vm.provision "shell", run: "always", inline: <<-SHELL
#          sysctl net.ipv4.conf.all.forwarding=1
#          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
#          echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
#          systemctl restart network
#        SHELL
#      when "centralServer"
#        box.vm.provision "shell", run: "always", inline: <<-SHELL
#          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
#          echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
#          systemctl restart network
#        SHELL
#      end
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
Дополнительно в коде добавили сетевые устройства из подсети 192.168.50.0/24 — они потребуются для настройки хостов с помощью Ansible.<br />
Так как все настройки будем выполнять с помощью Ansible, закомментировали в Vagrantfile почти все shell блоки.</p>

<p>Снова запустим эти виртуальные машины:</p>

<pre>[user@localhost netarchitecture]$ vagrant up</pre>

<p>Смотрим состояние запущенных виртуальных машин:</p>

<pre>[user@localhost netarchitecture]$ vagrant status
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
[user@localhost netarchitecture]$</pre>

<p>После того, как все 7 серверов у нас развернуты, нам нужно настроить маршрутизацию и NAT таким образом, чтобы доступ в Интернет со всех хостов был через inetRouter и каждый сервер должен быть доступен с любого из 7 хостов.<br />
Все настройки будем выполнять с помощью Ansible.</p>

<p>Создадим директорий roles:</p>

<pre>[user@localhost ansible]$ mkdir ./roles
[user@localhost ansible]$</pre>

<p>Перейдём в этот директорий и с помощью команды ansible-galaxy init создадим структуру директорий:</p>

<pre>[user@localhost ansible]$ cd ./roles/
[user@localhost roles]$ ansible-galaxy init netarchitecture
- Role netarchitecture was created successfully
[user@localhost roles]$</pre>

<h4>Настройка NAT</h4>

<p>Для того, чтобы на всех серверах работал интернет, на сервере inetRouter должен быть настроен NAT. Он настраивается с помощью команды:</p>

<pre>iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE</pre>

<p>При настройке NAT таким образом, правило удаляется после перезагрузки сервера. Для того, чтобы правила применялись после перезагрузки, в CentOS 7 нужно выполнить следующие действия:<br />
1)Подключиться по SSH к хосту: ssh root@inetRouter<br />
2)Проверить, что отключен другой firewall: systemctl status firewalld<br />
Если служба будет запущена, то нужно её отключить и удалить из автозагрузки:</p>

<pre>systemctl stop firewalld
systemctl disable firewalld</pre>

<p>3)Установить пакеты iptables и iptables-services:</p>

<pre>yum -y install iptables iptables-services</pre>

<p>4) Добавить службу iptables в автозапуск: systemctl enable iptables<br />
5) Отредактировать файл /etc/sysconfig/iptables: vi /etc/sysconfig/iptables<br />
Данный файл содержит в себе базовые правила, которые появляются с установкой iptables.<br />
Обращаем внимание на следующие правила:</p>

<pre>-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited</pre>

<p>Они запрещают ping между хостами, через данный сервер. В данном ДЗ эти команды закомментируем, чтобы они не применялись.<br />
Файл /etc/sysconfig/iptables не обязательно писать с нуля. Можно поступить следующим образом:<br />
Установить iptables и iptables-services<br />
Запустить службу iptables<br />
Внести необходимые правила iptables (и удалить ненужные)<br />
Выполнить команду iptables-save > /etc/sysconfig/iptables<br />
Данная команда сохранит в файл измененные правила.<br />
Для их применения нужно перезапустить службу iptables.<br />
Если просто запустить команду iptables-save, то на экране консоли появится полный список всех правил, которые действуют на текущий момент. Данная команда очень удобна для поиска проблем в iptables.</p>

<p>Вышеперечисленные действия выполним с помощью Ansible.<br />
Для этого в файл ./netarchitecture/tasks/main.yml добавим следующие команды:</p>

<pre>[user@localhost roles]$ vi ./netarchitecture/tasks/main.yml</pre>

<pre>---
# tasks file for netarchitecture

- name: Set up NAT on inetRouter
  block:
  - name: install iptables
    yum:
      name:
      - iptables
      - iptables-services
      state: present
      update_cache: true

  - name: copy iptables config
    template:
      src: iptables
      dest: /etc/sysconfig/iptables
      owner: root
      group: root
      mode: 0600
    notify:
      - start and enable iptables service
  when: (ansible_hostname == "inetRouter")</pre>

<p>Первый модуль «install iptables» устанавливает нам необходимые пакеты. Второй модуль "copy iptables config" копирует нам конфигурационный файл правил Iptables (который мы рассматривали в настройке NAT вручную).</p>

<pre>[user@localhost roles]$ vi ./netarchitecture/files/iptables</pre>

<pre># Generated by iptables-save v1.4.21 on Fri Sep 09 09:53:32 2022
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
# Deny ping trafic
# -A INPUT -j REJECT --reject-with icmp-host-prohibited
# -A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
# Completed on Mon Jan 17 09:53:32 2022
# Generated by iptables-save v1.4.21 on Fri Sep 09 09:53:32 2022
*nat
:PREROUTING ACCEPT [1:161]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
COMMIT
# Completed on Fri Sep 09 09:53:32 2022</pre>

<p>В файл ./netarchitecture/handlers/main.yml добавим модуль, который производит старт службы iptables и её добавление в автозапуск:</p>

<pre>[user@localhost roles]$ vi ./netarchitecture/handlers/main.yml</pre>

<pre>---
# handlers file for netarchitecture

- name: start and enable iptables service
  service:
    name: iptables
    state: restarted
    enabled: true</pre>

<h4>Маршрутизация транзитных пакетов (IP forward)</h4>

<p>Важным этапом настройки сетевой лаборатории, является маршрутизация транзитных пакетов. Если объяснить простыми словами — это возможность сервера Linux пропускать трафик через себя к другому серверу. По умолчанию эта функция отключена в Linux.<br />
Включить её можно командой:</p>

<pre>sysctl net.ipv4.conf.all.forwarding=1</pre>

<p>Посмотреть статус форвардинга можно командой:</p>

<pre>sysctl net.ipv4.ip_forward</pre>

<p>Если параметр равен 1, то маршрутизация транзитных пакетов включена, если 0 — отключена.В нашей схеме необходимо включить данную маршрутизацию на всех роутерах.</p>

<p>В Ansible есть специальный блок для внесений изменений в параметры ядра:</p>

<pre>[user@localhost roles]$ vi ./netarchitecture/tasks/main.yml</pre>

<pre>...
- name: set up forward packages across routers
  sysctl:
    name: net.ipv4.conf.all.forwarding
    value: '1'
    state: present
  when: "'routers' in group_names"</pre>

<p>В условии указано, что изменения будут применяться только для группы «routers», группа routers создана в hosts-файле:</p>

<pre>[user@localhost netarchitecture]$ vi ./ansible/hosts</pre>

<pre>[routers]
inetRouter ansible_host=192.168.50.10 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/inetRouter/virtualbox/private_key
centralRouter ansible_host=192.168.50.11 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/centralRouter/virtualbox/private_key
office1Router ansible_host=192.168.50.20 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/office1Router/virtualbox/private_key
office2Router ansible_host=192.168.50.30 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/office2Router/virtualbox/private_key

[servers]
centralServer ansible_host=192.168.50.12 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/centralServer/virtualbox/private_key
office1Server ansible_host=192.168.50.21 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/office1Server/virtualbox/private_key
office2Server ansible_host=192.168.50.31 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/office2Server/virtualbox/private_key</pre>

<p>Файл hosts — это файл инвентаризации, в нем указан список серверов, их адреса, группы и способы доступа на сервер.</p>

<h4>Отключение маршрута по умолчанию на интерфейсе eth0</h4>

<p>При разворачивании нашего стенда Vagrant создает в каждом сервере свой интерфейс, через который у сервера появляется доступ в интернет. Отключить данный порт нельзя, так как через него Vagrant подключается к серверам. Обычно маршрут по умолчанию прописан как раз на этот интерфейс, данный маршрут нужно отключить.Для отключения дефолтного маршрута нужно в файле <br />
/etc/sysconfig/network-scripts/ifcfg-eth0 <br /> 
найти строку <br />
DEFROUTE=yes и поменять её на DEFROUTE=no<br />
Vagrant по умолчанию не добавляет строку DEFROUTE=yes, <br />
поэтому нам можно просто добавить строку DEFROUTE=no <br />
Добавление строки:</p>

<pre>echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0</pre>

<p>Данное действие нужно выполнить только на CentOS-серверах centralRouter и centralServer. <br />
После удаление маршрута по умолчанию, нужно добавить дефолтный маршрут на другой порт. Делается это с помощью идентичной команды, например, команда добавления маршрута по умолчанию на сервере centralServer будет такой:</p>

<pre>echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1</pre>

<p>После внесения данных изменений нужно перезапустить сетевую службу:</p>

<pre>sytemctl restart network</pre>

<p>Для выполнения идентичных изменений с помощью Ansible, воспользуемся следующим блоком:</p>

<pre>[user@localhost roles]$ vi ./netarchitecture/tasks/main.yml</pre>

<pre># echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
- name: deisable default route
lineinfile:
dest: /etc/sysconfig/network-scripts/ifcfg-eth0
line: DEFROUTE=no
when: (ansible_hostname == "centralRouter") or
(ansible_hostname == "centralServer")
# echo "GATEWAY=192.168.255.1" >>
/etc/sysconfig/network-scripts/ifcfg-eth1
- name: add default gateway for centralRouter
lineinfile:
dest: /etc/sysconfig/network-scripts/ifcfg-eth1line: GATEWAY=192.168.255.1
when: (ansible_hostname == "centralRouter")
# echo "GATEWAY=192.168.0.1" >>
/etc/sysconfig/network-scripts/ifcfg-eth1
- name: add default gateway for centralServer
lineinfile:
dest: /etc/sysconfig/network-scripts/ifcfg-eth1
line: GATEWAY=192.168.0.1
when: (ansible_hostname == "centralServer"</pre>

<p>Модуль lineinfile добавляет строку в файл. Если строка уже добавлена, то второй раз она не добавится.</p>

<p>Далее нам нужно настроить статическую маршрутизацию на всех серверах.</p>

<h4>Настройка статических маршрутов</h4>

<p>Для настройки статических маршрутов используется команда ip route. Данная команда работает в Debian-based и RHEL-based системах.</p>

<p>Давайте рассмотрим пример настройки статического маршрута на сервере office1Server. Исходя из схемы мы видим, что трафик с данного сервера будет идти через office1Router. Office1Server и office1Router у нас соединены через сеть managers (192.168.2.128/26). В статическом маршруте нужно указывать адрес следующего хоста. Таким образом мы должны указать на сервере office1Server маршрут, в котором доступ к любым IP-адресам у нас будет происходить через адрес 192.168.2.129, который расположен на сетевом интерфейсе office1Router. Команда будет выглядеть так: ip route add 0.0.0.0/0 via 192.168.2.129</p>

<p>Посмотреть список всех маршрутов: ip route</p>

<pre>root@office1Server:~# ip r
default via 192.168.2.129 dev enp0s8 proto static
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15
10.0.2.2 dev enp0s3 proto dhcp scope link src 10.0.2.15 metric 100
192.168.2.128/26 dev enp0s8 proto kernel scope link src 192.168.2.130
192.168.50.0/24 dev enp0s19 proto kernel scope link src 192.168.50.21
root@office1Server:~#</pre>

<p>Удалить маршрут: </p>

<pre>ip route del 0.0.0.0/0 via 192.168.2.129</pre>

<p>Важно помнить, что маршруты, настроенные через команду ip route удаляются после перезагрузки или перезапуске сетевой службы.</p>
<p>Для того, чтобы маршруты сохранялись после перезагрузки нужно их указывать непосредственно в файле конфигурации сетевого интерфейса:<br />
● В CentOS нужно создать файл route-<имя интерфейса>, например для интерфейса<br />
/etc/sysconfig/network-scripts/ifcfg-eth1 необходимо создать файл<br />
/etc/sysconfig/network-scripts/route-eth1 и указать там правила в формате: <Сеть назначения>/<маска> via <Next hop address><br />
Пример файла /etc/sysconfig/network-scripts/route-eth1<br />
192.168.0.0/22 via 192.168.255.2<br />
192.168.255.4/30 via 192.168.255.2<br />
192.168.255.8/30 via 192.168.255.2</p>

<p>Применение маршрутов: service network restart<br />
● В современных версиях Ubuntu, для указания маршрута нужно поправить netplan-конфиг. Конфиги netplan хранятся в виде YAML-файлов и обычно лежат в каталоге /etc/netplan<br />
В нашем стенде такой файл - /etc/netplan/50-vagrant.yaml<br />
Для добавления маршрута, после раздела addresses нужно добавить блок:</p>

<pre>routes:
- to: <сеть назначения>/<маска>
  via: <Next hop address></pre>

<p>Пример файла /etc/netplan/50-vagrant.yaml</p>

<pre>---
network:
  version: 2renderer: networkd
  ethernets:
    enp0s8:
      addresses:
      - 192.168.2.130/26
      routes:
      - to: 0.0.0.0/0
        via: 192.168.2.129
    enp0s19:
      addresses:
      - 192.168.50.21/24</pre>

<p>В YAML-файле очень важно следить за правильными отступами, ошибка в один пробел не даст сохранить изменения.</p>

<p>Применение изменений:</p>

<pre>root@office1Server:~# netplan apply
root@office1Server:~# netplan try
Warning: Stopping systemd-networkd.service, but it can still be activated
by:
systemd-networkd.socket
Do you want to keep these settings?

Press ENTER before the timeout to accept the new configuration

Changes will revert in 121 seconds
Configuration accepted.
root@office1Server:~#</pre>

<p>● В Debian маршрут указывается в файле с сетевыми интерфейсами /etc/network/interfaces. Маршрут указывается после описания самого интерфейса и напоминает команду ip route: up ip route add <сеть назначения>/<маска> via <next hop address></p>

<p>Пример файла /etc/network/interfaces:</p>

<pre># interfaces(5) file used by ifup(8) and ifdown(8)
# Generate from Ansible
# Include files from /etc/network/interfaces.d:
source-directory /etc/network/interfaces.d

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug eth0
iface eth0 inet dhcp
# The contents below are automatically generated by Ansible. Do not modify.
auto eth1
iface eth1 inet static
address 192.168.1.2
netmask 255.255.255.128
#default route
up ip route add 0.0.0.0/0 via 192.168.1.1

# The contents below are automatically generated by Ansible. Do not modify.
auto eth2
iface eth2 inet static
      address 192.168.50.31
      netmask 255.255.255.0</pre>

<p>Для применения изменений нужно перезапустить сетевую службу:</p>

<pre>systemctl restart networking</pre>

<p>При настройке интерфейсов и маршрутов в любой из ОС можно оставлять комментарии в файле. Перед комментарием должен стоять знак «#»</p>

<p>Важно помнить, что помимо маршрутов по умолчанию, нам нужно будет использовать обратные маршруты.</p>

<p>Давайте разберем пример такого маршрута: допустим мы хотим отправить команду ping с сервера office1Server (192.168.2.130) до сервера centralRouter (192.168.0.1)<br />
Наш трафик пойдёт следующим образом: office1Server — office1Router — centralRouter — office1Router — office1Server</p>
<p>Office1Router знает сеть (192.168.2.128/26), в которой располагается сервер office1Server, а сервер centralRouter, когда получит запрос от адреса 192.168.2.130 не будет понимать, куда отправить ответ. Решением этой проблемы будет добавление обратного маршрута.</p>
<p>Обратный маршрут указывается также как остальные маршруты.<br />
Изучив схему мы видим, что связь между сетеми 192.168.2.0/24 и 192.168.0.0/24 осуществляется через сеть 192.168.255.8/30. Также мы видим что сети office1 подключены к centralRouter через порт eth5. На основании этих данных мы можем создать файл<br />
/etc/sysconfig/network-scripts/route-eth5<br />
и добавить в него маршрут 192.168.2.0/24 via 192.168.255.1</p>

<p>Так как для настройки используем Ansible, нам необходимо подготовить файлы с маршрутами для всех серверов. Далее с помощью модуля template мы можем их добавлять:</p>

<pre>[user@localhost roles]$ vi ./netarchitecture/tasks/main.yml</pre>

<pre>...
- name: set up route on office1Server
  template:
    src: office1Server_route.j2
    dest: /etc/netplan/50-vagrant.yaml
    owner: root
    group: root
    mode: 0644
  when: (ansible_hostname == "office1Server")
  
- name: set up route on office2Server
  template:
    src: office2Server_route.j2
    dest: /etc/network/interfaces
    owner: root
    group: root
    mode: 0644
  when: (ansible_hostname == "office2Server")
  
- name: set up route on centralRouter eth1
  template:
    src: centralRouter_route_eth1.j2
    dest: /etc/sysconfig/network-scripts/route-eth1
    owner: root
    group: root
    mode: 0644
  when: (ansible_hostname == "centralRouter")</pre>

<p>Конфигурационный файл office1Server_route.j2:</p>

<pre>[user@localhost roles]$ vi ./netarchitecture/templates/office1Server_route.j2</pre>

<pre>---
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
      addresses:
      - 192.168.2.130/26
      routes:
      - to: 0.0.0.0/0
        via: 192.168.2.129
    enp0s19:
      addresses:
      - 192.168.50.21/24</pre>

<p>Также модуль template позволяет использовать jinja2-шаблоны, с помощью которых файл может генериться автоматически,забирая информацию из данных о хосте (Ansible facts) и файлов переменных.</p>

<p>Для того, чтобы Ansible запускался сразу после развертывания серверов командой vagrant up, в текущий Vagrantfile добавим блок запуска Ansible. Данный блок рекомендуется добавить после блока разворачивания виртуальных машин:</p>

<pre>...
if boxconfig[:vm_name] == "office2Server"
  box.vm.provision "ansible" do |ansible|
    ansible.playbook = "ansible/playbook.yml"
    ansible.inventory_path = "ansible/hosts"
    ansible.host_key_checking = "false"
    ansible.limit = "all"
  end
end
...</pre>

<p>При добавлении блока Ansible в Vagrant, Ansible будет запускаться после создания каждой ВМ, это создат ошибку в разворачивании стенда и стенд не развернется. Для того, чтобы Ansible запустился после создания виртуальных машин, можно добавить условие, которое будет сравнивать имя виртуальной машины, и, когда условный оператор увидит имя последней созданной ВМ (office2Server), он запустит Ansible.</p>

<p>Разворачивание 7 виртуальных машин — процесс достаточно затраный для ресурсов компьютера. При разворачивании стенда с помощью Ansible иногда могут вылетать ошибки из-за сетевой связности. Это не страшно, после того как ВМ будут созданы, можно просто ещё раз запустить процесс настройки через ansible с помощью команды: </p>

<pre>vagrant provision</pre>

<p>На данном этапе настройка серверов закончена. После настройки серверов перезагружаем все хосты, чтобы
проверить, что правила не удаляются после перезагрузки.</p>

<p>Для проверки нашего стенда на все хосты установим утилиту traceroute.<br />
Установка traceroute:<br />
● CentOS 7: yum -y install traceroute<br />
● Debian,Ubuntu: apt install -y traceroute</p>

<h4>Проерка работы нашего стенда "архитектура сети"</h4>

<p>Запустим наш стенд с помощью vagrant+ansible:</p>

<pre>[user@localhost netarchitecture]$ vagrant up</pre>

<p>Зайдем на сервер, например, office2Server с помощью утилиты traceroute проверим выход в интернет через сервер inetRoute (ip:192.168.255.1):</p>

<pre>[user@localhost netarchitecture]$ vagrant ssh office2Server
Linux office2Server 5.10.0-16-amd64 #1 SMP Debian 5.10.127-1 (2022-06-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Mon Sep 12 20:40:00 2022 from 192.168.50.1
vagrant@office2Server:~$ traceroute 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  192.168.1.1 (192.168.1.1)  1.745 ms  1.167 ms  1.976 ms
 2  192.168.255.5 (192.168.255.5)  3.934 ms  3.888 ms  4.666 ms
 3  192.168.255.1 (192.168.255.1)  4.298 ms  3.539 ms  6.372 ms
 4  * * *
 5  * * *
 6  * * *
 7  * * *
 8  ae1-3121.edge8.Frankfurt1.level3.net (4.69.158.186)  13.438 ms  15.269 ms ae2-3221.edge8.Frankfurt1.level3.net (4.69.158.190)  21.262 ms
 9  142.250.165.106 (142.250.165.106)  20.350 ms  14.409 ms  15.798 ms
10  * * *
11  dns.google (8.8.8.8)  10.478 ms  9.256 ms  9.393 ms
vagrant@office2Server:~$</pre>

<p>В данном примере, в первых трёх переходах мы видим что запрос идёт через сервера: office2Router(ip:192.168.1.1) — centralRouter(ip:192.168.255.5) — inetRouter(ip:192.168.255.1).</p>

<p>Проверим, что с этого хоста доступны сервера centralServer(ip:192.168.0.2) и office1Server(ip:192.168.2.130):</p>

<pre>vagrant@office2Server:~$ traceroute 192.168.0.2
traceroute to 192.168.0.2 (192.168.0.2), 30 hops max, 60 byte packets
 1  192.168.1.1 (192.168.1.1)  2.276 ms  1.468 ms  2.384 ms
 2  192.168.255.5 (192.168.255.5)  7.492 ms  6.951 ms  5.743 ms
 3  192.168.0.2 (192.168.0.2)  6.450 ms  5.798 ms  5.309 ms
vagrant@office2Server:~$</pre>

<pre>vagrant@office2Server:~$ traceroute 192.168.2.130
traceroute to 192.168.2.130 (192.168.2.130), 30 hops max, 60 byte packets
 1  192.168.1.1 (192.168.1.1)  1.810 ms  1.710 ms  2.092 ms
 2  192.168.255.5 (192.168.255.5)  4.179 ms  2.439 ms  3.372 ms
 3  192.168.255.10 (192.168.255.10)  8.992 ms  10.567 ms  9.613 ms
 4  192.168.2.130 (192.168.2.130)  12.631 ms  13.877 ms  13.311 ms
vagrant@office2Server:~$ </pre>

<p>Проверим выход в интернет с хоста office1Server:</p>

<pre>[user@localhost netarchitecture]$ vagrant ssh office1Server
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-125-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Sep 12 21:11:51 UTC 2022

  System load:  0.1               Users logged in:          0
  Usage of /:   3.9% of 38.70GB   IPv4 address for enp0s19: 192.168.50.21
  Memory usage: 22%               IPv4 address for enp0s3:  10.0.2.15
  Swap usage:   0%                IPv4 address for enp0s8:  192.168.2.130
  Processes:    116


0 updates can be applied immediately.

New release '22.04.1 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Mon Sep 12 20:40:01 2022 from 192.168.50.1
vagrant@office1Server:~$ traceroute 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  _gateway (192.168.2.129)  2.574 ms  1.917 ms  3.202 ms
 2  192.168.255.9 (192.168.255.9)  6.005 ms  12.233 ms  11.457 ms
 3  192.168.255.1 (192.168.255.1)  12.983 ms  12.177 ms  13.673 ms
 4  * * *
 5  * * *
 6  * * *
 7  * * *
 8  ae2-3221.edge8.Frankfurt1.level3.net (4.69.158.190)  15.485 ms ae1-3121.edge8.Frankfurt1.level3.net (4.69.158.186)  17.234 ms ae2-3221.edge8.Frankfurt1.level3.net (4.69.158.190)  14.074 ms
 9  142.250.165.106 (142.250.165.106)  16.567 ms  12.972 ms  15.944 ms
10  * * *
11  dns.google (8.8.8.8)  12.949 ms  13.079 ms  10.180 ms
vagrant@office1Server:~$</pre>

<p>Как видим, с сервера office1Server выходим в интернет через сервера: office1Router(ip:192.168.2.129) — centralRouter(ip:192.168.255.9) — inetRouter(ip:192.168.255.1).</p>

<p>Проверим доступ к серверам centralServer(ip:192.168.0.2) и offic2Server(192.168.1.2):</p>

<pre>vagrant@office1Server:~$ traceroute 192.168.0.2
traceroute to 192.168.0.2 (192.168.0.2), 30 hops max, 60 byte packets
 1  _gateway (192.168.2.129)  1.951 ms  2.149 ms  2.019 ms
 2  192.168.255.9 (192.168.255.9)  10.492 ms  9.773 ms  10.879 ms
 3  192.168.0.2 (192.168.0.2)  27.866 ms  27.337 ms  27.663 ms
vagrant@office1Server:~$</pre>

<pre>vagrant@office1Server:~$ traceroute 192.168.1.2
traceroute to 192.168.1.2 (192.168.1.2), 30 hops max, 60 byte packets
 1  _gateway (192.168.2.129)  2.259 ms  3.671 ms  1.871 ms
 2  192.168.255.9 (192.168.255.9)  5.580 ms  14.448 ms  14.851 ms
 3  192.168.255.6 (192.168.255.6)  20.851 ms  20.062 ms  20.965 ms
 4  192.168.1.2 (192.168.1.2)  21.439 ms  20.736 ms  21.502 ms
vagrant@office1Server:~$</pre>

<p>Проверим выход в интернет с сервера centralServer:</p>

<pre>[root@centralServer ~]# traceroute 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  gateway (192.168.0.1)  2.660 ms  1.560 ms  1.291 ms
 2  192.168.255.1 (192.168.255.1)  5.397 ms  4.279 ms  3.449 ms
 3  * * *
 4  * * *
 5  * * *
 6  * * *
 7  ae1-3121.edge8.Frankfurt1.level3.net (4.69.158.186)  11.036 ms  11.678 ms  12.235 ms
 8  142.250.165.106 (142.250.165.106)  11.700 ms  9.387 ms  9.883 ms
 9  * * *
10  dns.google (8.8.8.8)  9.615 ms  8.515 ms  8.346 ms
[root@centralServer ~]#</pre>

<p>Как видим, с сервера centralServer выходим в интернет через сервера: centralRouter(ip:192.168.0.1) — inetRouter(ip:192.168.255.1).</p>

<p>Проверим доступ к серверам office1Server(ip:192.168.2.130) и offic2Server(ip:192.168.1.2):</p>

<pre>[root@centralServer ~]# traceroute 192.168.2.130
traceroute to 192.168.2.130 (192.168.2.130), 30 hops max, 60 byte packets
 1  gateway (192.168.0.1)  2.444 ms  1.813 ms  1.035 ms
 2  192.168.255.10 (192.168.255.10)  5.220 ms  5.712 ms  4.525 ms
 3  192.168.2.130 (192.168.2.130)  22.001 ms  20.248 ms  18.445 ms
[root@centralServer ~]#</pre>

<pre>[root@centralServer ~]# traceroute 192.168.1.2
traceroute to 192.168.1.2 (192.168.1.2), 30 hops max, 60 byte packets
 1  gateway (192.168.0.1)  1.972 ms  1.979 ms  1.647 ms
 2  192.168.255.6 (192.168.255.6)  2.960 ms  1.888 ms  3.179 ms
 3  192.168.1.2 (192.168.1.2)  6.990 ms  7.226 ms  6.368 ms
[root@centralServer ~]#</pre>

<p>Из всего этого мы делаем вывод, что все сервера имеют выход через сервер inetRouter и доступны друг другу.</p>

