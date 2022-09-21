# Playbook Ansible установка wordpress-lamp
Плейбук для удаленной установки LAMP+Wordpress с помощью Ansible.
Протестировано на Ubuntu 18.04.6 LTS (ansible-target) с Ubuntu 20.04.4 LTS (ansible-controller).

При необходимости поднимаем виртуальные машины. У меня были подняты хосты ansiblecontroller, ansibletarget1, ansibletarget2, ansibletarget3.
Изменяем имена хостов в контроллере, управляемых хостах (для удобства при работе с хостами по SSH): 
sudo vi /etc/hostname -> ansiblecontroller, sudo vi /etc/hosts -> ansiblecontroller, аналогично на ansibletarget.
Адреса хостов в моем случае: ansiblecontroller 10.0.2.10, ansibletarget1 10.0.2.11, ansibletarget2 10.0.2.12, ansibletarget3 10.0.2.13.


Устанавливаем на хост ansiblecontroller ansible:  
sudo apt-get install ansible

Настраиваем ssh. На каждой из машин отправляем команды:  
sudo systemctl status sshd (проверили наличие ssh)  
sudo apt install openssh-server ssh (установили ssh службу при необходимости)  
sudo systemctl start sshd (запустили службу)  

Проверяем связь по ssh:  
ssh username@remote_host  
(в моем случае: ssh-copy-id osboxes@10.0.2.13)

Далее на хосте ansiblecontroller выполняем команду для генерации ключей ssh:  
ssh-keygen  

Теперь необходимо загрузить ключ на каждый удаленный сервер (в моем случае нужно на управляемый хост ansibletarget3):  
ssh-copy-id username@remote_host (в моем случае: ssh-copy-id osboxes@10.0.2.13), на управляемом хосте должен быть включен параметр в /etc/ssh/sshd_config:  
PubkeyAuthentication yes  

Отключаем ssh авторизацию суперпользователя по паролю и проч. в /etc/ssh/sshd_config (в целях безопасности, останется только по ssh-ключу):  
sudo nano /etc/ssh/sshd_config  
PermitRootLogin no  
ChallengeResponseAuthentication no  
PasswordAuthentication no  
UsePAM no  

Создаем папку для проектов и переходим в нее:  
mkdir projects  
cd projects  
Создаем файл inventory на ansiblecontroller:  
cat > inventory  
target1 ansible_host=10.0.2.11  
target2 ansible_host=10.0.2.12  
target3 ansible_host=10.0.2.13  

Теперь можем проверить отправку команд ansible на ansibletarget3:  
ansible target3 -m ping -i inventory  
Если в ответ получаем "pong", то команды ansible на удаленном хосте выполняются.  

Клонируем на ansiblecontroller репозиторий с плейбуком и переменными:  
cd ~/projects  
git clone https://github.com/m-koleda/ansible-wordpress-lamp.git  
cd ansible-wordpress-lamp/wordpress-lamp_ubuntu1804  

Изменяем переменные в файле default.yml:  
nano vars/default.yml  
Описание переменных:  
mysql_root_password: пароль для root учетной записи MySQL.  
mysql_db: имя базы данных MySQL, которая должна быть создана для WordPress.  
mysql_user: имя пользователя MySQL, которое должно быть создано для WordPress.  
mysql_password: пароль для нового пользователя MySQL для WordPress.  
http_host: доменное имя вашего нового сайта.  
httpd_conf: имя файла конфигурации, который будет создан в Apache.  
http_port: HTTP-порт для этого виртуального хоста, значение по умолчанию 80.  

Для запуска плейбука нужно ввести команду:  
$ ansible-playbook ~/ansible-playbooks/wordpress-lamp_ubuntu1804/playbook.yml -l target3 -i ~/projects/inventory -u osboxes --ask-become-pass  

Итоговый playbook.yml зашифрован командой ansible-vault:  
ansible-vault encrypt playbook.yml  
Для расшифровки необходимо ввести команду:  
ansible-vault decrypt playbook.yml  
а затем пароль osboxes  
