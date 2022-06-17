<h3>### Загрузка системы ###</h3>

<h4>Описание домашнего задания</h4>

<ol>
<li>Попасть в систему без пароля несколькими способами;</li>
<li>Установить систему с LVM, после чего переименовать VG;</li>
<li>Добавить модуль в initrd.</li>
</ol>

<h4># Создадим виртуальную машину</h4>

<p>В домашней директории создадим директорию loadsystem, в которой будут храниться настройки виртуальной машины:</p>

<pre>[user@localhost otus]$ mkdir ./loadsystem
[user@localhost otus]$</pre>

<p>Перейдём в директорию loadsystem:</p>

<pre>[user@localhost otus]$ cd ./loadsystem/
[user@localhost loadsystem]$</pre>

<p>Создадим файл Vagrantfile:</p>

<pre>[student@pv-homeworks1-10 nfs]$ vi ./Vagrantfile</pre>

<p>Заполним следующим содержимым:</p>

<pre># -*- mode: ruby -*-
# # vi: set ft=ruby :

Vagrant.configure(2) do |config|
#  config.vm.box = "centos/7"
  config.vm.box = 'almalinux/8'
  config.vm.network :forwarded_port, guest: 22, host: 4000

  config.vm.define "loadsystem" do |loadsystem|

    loadsystem.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end

    loadsystem.vm.disk :disk, size: "1GB", name: "disk1"
    loadsystem.vm.host_name = 'loadsystem'
    loadsystem.vm.network :private_network, ip: "192.168.56.141"

  end
end
</pre>

<p>Запустим виртуальную машину:</p>

<pre>[user@localhost loadsystem]$ vagrant up</pre>

<p>Остановим виртуальную машину:</p>

<pre>[user@localhost loadsystem]$ vagrant halt</pre>

<p>Откроем VirtualBox GUI:</p>

![image](https://user-images.githubusercontent.com/96518320/174353634-399a5d82-1f28-460e-9e53-57b02389f95c.png)

<p>Запустим виртуальную машину loadsystem в VirtualBox GUI:</p>

![image](https://user-images.githubusercontent.com/96518320/174353840-6833d0fd-de66-4d1a-bf4b-3978ecb07e17.png)

<p>В момент списка ядер для загрузки нажимаем "e" для изменения параметров загрузки:</p>

![image](https://user-images.githubusercontent.com/96518320/174353922-6fb16908-00b7-4ea9-b07d-29136227c1b6.png)

<br />



<p><b>Способ 1. init=/bin/sh</b></p>

<p>В конце строки, которая начинается с linux добавим init=/bin/sh, заодно удалим rhgb и quiet для полноты информации во время загрузки:</p>

![image](https://user-images.githubusercontent.com/96518320/174355050-accef91b-940e-465c-9d2f-879d365abb7a.png)

<p>и нажимаем сtrl-x для загрузки в систему.</p>

<p>Попадаем в консоль рутовой файловой системы <br /> sh-4.4#</p>

![image](https://user-images.githubusercontent.com/96518320/174355564-95826eeb-1b18-4e97-9dd3-95264f3f4731.png)

<p>При этом файловая система монтируется в режиме Read-Only. Нам нужно перемонтировать в Read-Write командой: <br />sh-4.4# mount -o remount,rw /</p>

![image](https://user-images.githubusercontent.com/96518320/174355943-393db096-b6d3-40b4-9cf0-0ad22cee404d.png)

<p>Теперь можем установить новый пароль, например, vagrant123 для пользователя vagrant командой: <br />sh-4.4# passwd vagrant</p>

![image](https://user-images.githubusercontent.com/96518320/174356217-2da90a87-3c9a-4100-a143-ebf521d932a4.png)

<p>В корне создадим файл .autorelabel: <br />sh-4.4# touch /.autorelabel</p>

![image](https://user-images.githubusercontent.com/96518320/174356409-e97c0220-98ed-459c-9d7c-f63171ffcd05.png)

<p>Возвращаем файловую систему в режим Read-Only: <br />sh-4.4# mount -o remount,ro /</p>

![image](https://user-images.githubusercontent.com/96518320/174356567-afef3d19-c020-4600-9017-4f80261d460a.png)

<p>Перезапустим виртуальную машину.</p>

![image](https://user-images.githubusercontent.com/96518320/174356640-2135821f-bf3c-42a2-a396-df11fedf820d.png)

![image](https://user-images.githubusercontent.com/96518320/174357025-9c7a53e2-2e44-4c8c-922f-94f8505c731f.png)

<p>Вводим логин vagrant и пароль vagrant123</p>

![image](https://user-images.githubusercontent.com/96518320/174357148-0d6f0876-130c-43c8-99c6-61a4ad5205cc.png)

<p>как видим, мы вошли в систему под пользователем vagrant с новым паролем.</p>

<br />



<p><b>Способ 2. rd.break</b></p>

<p>В конце строки, которая начинается с linux добавим rd.break, заодно удалим rhgb и quiet для полноты информации во время загрузки:</p>

![image](https://user-images.githubusercontent.com/96518320/174355050-accef91b-940e-465c-9d2f-879d365abb7a.png)

<p>и нажимаем сtrl-x для загрузки в систему.</p>

<p>Попадаем в консоль рутовой файловой системы <br /> sh-4.4#</p>

![image](https://user-images.githubusercontent.com/96518320/174355564-95826eeb-1b18-4e97-9dd3-95264f3f4731.png)

<p>При этом файловая система монтируется в режиме Read-Only. Нам нужно перемонтировать в Read-Write командой: <br />sh-4.4# mount -o remount,rw /</p>

![image](https://user-images.githubusercontent.com/96518320/174355943-393db096-b6d3-40b4-9cf0-0ad22cee404d.png)

<p>Теперь можем установить новый пароль, например, vagrant123 для пользователя vagrant командой: <br />sh-4.4# passwd vagrant</p>

![image](https://user-images.githubusercontent.com/96518320/174356217-2da90a87-3c9a-4100-a143-ebf521d932a4.png)

<p>В корне создадим файл .autorelabel: <br />sh-4.4# touch /.autorelabel</p>

![image](https://user-images.githubusercontent.com/96518320/174356409-e97c0220-98ed-459c-9d7c-f63171ffcd05.png)

<p>Возвращаем файловую систему в режим Read-Only: <br />sh-4.4# mount -o remount,ro /</p>

![image](https://user-images.githubusercontent.com/96518320/174356567-afef3d19-c020-4600-9017-4f80261d460a.png)

<p>Перезапустим виртуальную машину.</p>

![image](https://user-images.githubusercontent.com/96518320/174356640-2135821f-bf3c-42a2-a396-df11fedf820d.png)

![image](https://user-images.githubusercontent.com/96518320/174357025-9c7a53e2-2e44-4c8c-922f-94f8505c731f.png)

<p>Вводим логин vagrant и пароль vagrant123</p>

![image](https://user-images.githubusercontent.com/96518320/174357148-0d6f0876-130c-43c8-99c6-61a4ad5205cc.png)

<p>как видим, мы вошли в систему под пользователем vagrant с новым паролем.</p>

