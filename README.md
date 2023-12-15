# IKEv2-setup
## Установка
1. Выберите доменное имя для сервера VPN и убедитесь, что оно уже привязано к правильному IP-адресу, создав соответствующую запись A в DNS и убедившись в распространении информации о ней. Let's Encrypt нуждается в этом для создания сертификата для вашего сервера.
2. Установите чистую Ubuntu Server.
3. По желанию настройте аутентификацию SSH на основе ключей (в качестве альтернативы, это может быть автоматически обработано вашим поставщиком сервера, или вы можете выбрать сохранение аутентификации на основе пароля).

  Для этого вам может потребоваться выполнить указанные ниже команды с соответствующими подстановками на компьютере, с которого вы собираетесь войти.
```
ssh-keygen -t rsa -b 4096 -C "dpzab.online"

ssh root@myvpn.example.net

#Linux 
ssh-copy-id -i ~/.ssh/id_ed25519.pub root@myvpn.example.net

#Windows
mkdir ./.ssh
touch ./.ssh/authorized_keys 
type %USERPROFILE%\.ssh\id_rsa.pub | ssh root@147.45.196.58 "cat >> .ssh/authorized_keys"

ssh root@myvpn.example.net
```
  Далее вам понадобится отключить авторизацию по паролю. Для отключения аутентификации по паролю отредактируйте файл /etc/ssh/sshd_config на удаленном сервере.
```
sudo nano /etc/ssh/sshd_config
```
  Найдите данную строку:
```
#PasswordAuthentication yes
```
  Измените её
```
PasswordAuthentication no
```
  Сохраните файл и перезапустите SSH сервис.
```
sudo systemctl restart ssh
```
4. При установке нового сервера подключитесь в качестве суперпользователя (root), загрузите скрипт, предоставьте ему права на выполнение и запустите его.
```
wget https://raw.githubusercontent.com/ManZill/IKEv2-setup/master/setup.sh
chmod u+x setup.sh
./setup.sh
```
5. После завершения обновления программного обеспечения и установки вам будет предложено ввести все необходимые данные. Если вы не используете аутентификацию по ключу SSH, убедитесь, что вы выберете действительно надежный пароль для учетной записи входа при запросе, иначе ваш сервер может быть скомпрометирован.

    Часть сессии, в которой скрипт задает вам вопросы, должна выглядеть примерно так:
```
 --- Configuration: VPN settings ---

 Network interface: eth0
 External IP: 100.100.100.100

 ** Note: hostname must resolve to this machine already, to enable Let's Encrypt certificate setup **
 Hostname for VPN: dpzab.online
 VPN username: manzill
 VPN password (no quotes, please): 
 Confirm VPN password: 

 Public DNS servers include:

 176.103.130.130,176.103.130.131  AdGuard               https://adguard.com/en/adguard-dns/overview.html
 176.103.130.132,176.103.130.134  AdGuard Family        https://adguard.com/en/adguard-dns/overview.html
 1.1.1.1,1.0.0.1                  Cloudflare/APNIC      https://1.1.1.1
 84.200.69.80,84.200.70.40        DNS.WATCH             https://dns.watch
 8.8.8.8,8.8.4.4                  Google                https://developers.google.com/speed/public-dns/
 208.67.222.222,208.67.220.220    OpenDNS               https://www.opendns.com
 208.67.222.123,208.67.220.123    OpenDNS FamilyShield  https://www.opendns.com
 9.9.9.9,149.112.112.112          Quad9                 https://quad9.net
 77.88.8.8,77.88.8.1              Yandex                https://dns.yandex.com
 77.88.8.88,77.88.8.2             Yandex Safe           https://dns.yandex.com
 77.88.8.7,77.88.8.3              Yandex Family         https://dns.yandex.com
 
 DNS servers for VPN users (default: 1.1.1.1,1.0.0.1): 176.103.130.130,176.103.130.131

 --- Configuration: general server settings ---

 Timezone (default: Europe/London): 
 Email address for sysadmin (e.g. j.bloggs@example.com): me@my-domain.tld
 Desired SSH log-in port (default: 22): 2222
 New SSH log-in user name: george
 Copy /root/.ssh/authorized_keys to new user and disable SSH password log-in [Y/n]? y
 New SSH user's password (e.g. for sudo): 
 Confirm new SSH user's password: 
```
6. Как только вы запустите систему, используйте эти команды для получения информации о текущем состоянии:
```
sudo ipsec statusall           # status, who's connected, etc.
sudo iptables -L -v            # how much traffic has been forwarded, dropped, etc.?
sudo tail -f /var/log/syslog   # real-time logs of (dis)connections etc.
```

## Пользователи
Чтобы добавить или изменить VPN пользователей, введите это:
```
sudo nano /etc/ipsec.secrets
```
Измените имена пользователей и пароли по своему усмотрению (но не трогайте первую строку, которая указывает сертификат сервера). Формат строки для каждого пользователя следующий:
```
someusername : EAP "somepassword"
```
Чтобы StrongSwan уловил изменения, нужно: 
```
sudo ipsec secrets
```
