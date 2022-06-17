<h3>### Загрузка системы ###</h3>

<h4>Описание домашнего задания</h4>

<ol>
<li>Попасть в систему без пароля несколькими способами;</li>
<li>Установить систему с LVM, после чего переименовать VG;</li>
<li>Добавить модуль в initrd.</li>
</ol>
<br />



<h4># 1. Попасть в систему без пароля несколькими способами</h4>

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

![image](https://user-images.githubusercontent.com/96518320/174389830-1e629399-cac4-4c3b-b3f7-66f0708d9083.png)

<p>и нажимаем сtrl-x для загрузки в систему.</p>

<p>Попадаем в консоль рутовой файловой системы в emergency mode<br />switch_root:/#</p>

![image](https://user-images.githubusercontent.com/96518320/174390039-c9dcd1cc-30a8-469c-966e-02c607e81e4e.png)

<p>При этом файловая система монтируется в режиме Read-Only. Нам нужно перемонтировать в Read-Write командой: <br />switch_root:/# mount -o remount,rw /sysroot</p>

![image](https://user-images.githubusercontent.com/96518320/174390621-14516510-0e92-454f-82b4-daef506b44f2.png)

<p>Переходим в sysroot <br />switch_root:/# chroot /sysroot</p>

![image](https://user-images.githubusercontent.com/96518320/174390910-b234c206-fd60-4c08-97b6-5f1aff2eb024.png)

<p>Теперь можем установить новый пароль, например, vagrant456 для пользователя vagrant командой: <br />sh-4.4# passwd vagrant</p>

![image](https://user-images.githubusercontent.com/96518320/174391136-c328d701-f3e1-48bc-b07e-a592b1f54b56.png)

<p>В корне создадим файл .autorelabel: <br />sh-4.4# touch /.autorelabel</p>

![image](https://user-images.githubusercontent.com/96518320/174391304-ea22a769-9c4d-4558-85f7-41bc193fd705.png)

<p>Выходим <br />sh-4.4# exit</p>

![image](https://user-images.githubusercontent.com/96518320/174391526-e1cae08a-4d2c-4f19-9311-e7ada5b7ebba.png)

<p>Возвращаем файловую систему в режим Read-Only: <br />switch_root:/# mount -o remount,ro /sysroot</p>

![image](https://user-images.githubusercontent.com/96518320/174391747-7e949884-e1ed-49b6-9876-954196f0af3f.png)

<p>Перезапустим виртуальную машину: <br />switch_root:/# reboot</p>

![image](https://user-images.githubusercontent.com/96518320/174391912-9599feb8-e485-4ba3-964e-c1b8ae45e770.png)

![image](https://user-images.githubusercontent.com/96518320/174392129-cce4444d-8f0a-46ee-95c3-60f08f46cba3.png)

<p>Вводим логин vagrant и пароль vagrant456</p>

![image](https://user-images.githubusercontent.com/96518320/174392225-49f54d24-9143-48fe-a8d9-b828ec11e3c8.png)

<p>как видим, мы вошли в систему под пользователем vagrant с новым паролем.</p>

<br />



<p><b>Способ 3. rw init=/sysroot/bin/sh</b></p>

<p>В строке, которая начинается с linux вместо ro заменим на rw, а в конце добавим init=/sysroot/bin/sh, заодно удалим rhgb и quiet для полноты информации во время загрузки:</p>

![image](https://user-images.githubusercontent.com/96518320/174393201-05aba37f-4167-4133-993a-747d615714a3.png)

<p>и нажимаем сtrl-x для загрузки в систему.</p>

<p>Попадаем в консоль рутовой файловой системы <br /> :/#</p>

![image](https://user-images.githubusercontent.com/96518320/174393352-1657c8b5-a6d1-40e3-869a-7f281a9ab84f.png)

<p>При этом файловая система сразу монтируется в режиме Read-Write.</p>

<p>Установим новый пароль, например, vagrant789 для пользователя vagrant командой: <br />:/# passwd vagrant</p>

![image](https://user-images.githubusercontent.com/96518320/174394152-1874b403-6475-4d2a-94a1-c56772b4adf1.png)

<p>В корне создадим файл .autorelabel: <br />:/# touch /.autorelabel</p>

![image](https://user-images.githubusercontent.com/96518320/174394259-26114eb8-18ca-47dc-bf5c-b62a9ea64cfa.png)

<p>Перезапустим виртуальную машину.</p>

![image](https://user-images.githubusercontent.com/96518320/174394401-6532ee64-7e35-45f8-8464-612661fca18c.png)

![image](https://user-images.githubusercontent.com/96518320/174394738-b6602c05-c703-4e52-a040-dc788b35c2ce.png)

<p>Вводим логин vagrant и пароль vagrant789</p>

![image](https://user-images.githubusercontent.com/96518320/174394812-ec30dfb5-eb72-4f06-a5ee-72ef60da48c2.png)

<p>как видим, мы вошли в систему под пользователем vagrant с новым паролем.</p>

<br />



<h4># 2. Установить систему с LVM, после чего переименовать VG</h4>


