# Instalação do Zabbix
Esta instalação será realziada com o selinux habilitado

### Arquivo Vagrant para subir o Centos

```vagrantFile
  config.vm.define :zabbixCentos8 do |zabbixCentos8_config|
    zabbixCentos8_config.vm.box = 'generic/centos8'
    zabbixCentos8_config.vm.hostname = 'zabbix'
    zabbixCentos8_config.vm.network :private_network, ip: '192.168.50.60'
    zabbixCentos8_config.vm.synced_folder "E:/Arquivos/Vagrant/share", "/vagrant/share", nfs:true
    zabbixCentos8_config.vm.provider "virtualbox" do |v|
      v.memory = 1024
      v.cpus = 1
      v.name = "zabbix"
    end
  end  
```

### Verificando a versão do CentOS

```
cat /etc/redhat-release 
```

### Verificando o getenforce

```
getenforce
``` 
Resultado

```
Enforcing
```

### Verificando o Seliniun status
```
[vagrant@zabbix ~]$ sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      31
```

## Instalação do MariaDB

```
sudo dnf -y install mariadb-server
```

### Criando um link de inicialização do MariaDB

```
sudo systemctl enable mariadb
```

### Iniciando o Mariadb

```
sudo systemctl start mariadb
```

Por padrão a senha de rrot do mariadb vem em branco.

```
mysql -uroot -p
```

### Definindo senha de root do Mariadb

```
mysql_secure_installation
```

### Download do Zabbix

```
https://www.zabbix.com/br/download?zabbix=4.0&os_distribution=red_hat_enterprise_linux&os_version=8&db=mysql&ws=apache
```

Seguir os comandos de instalação do site Zabbix

Comandos

a. repositório
```
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/8/x86_64/zabbix-release-5.0-1.el8.noarch.rpm
dnf clean all
```

b. Instale o servidor, o frontend e o agente Zabbix
```
dnf install zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-agent
```
c.Criar banco de dados inicial
```
# mysql -uroot -p
password
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> quit;
```

No servidor do Zabbix, importe o esquema inicial e os dados. Vocá será solicitado a inserir a senha que foi criada anteriormente.
```
zcat /usr/share/doc/zabbix-server-mysql/create.sql.gz |mysql -uzabbix -p zabbix
```

d. Configure o banco de dados para o servidor Zabbix
Editar arquivo /etc/zabbix/zabbix_server.conf

```
DBPassword=password
```

e. Configure o PHP para o frontend Zabbix
Editar arquivo /etc/php-fpm.d/zabbix.conf, descomente e defina o fuso horário correto.
```
php_value[date.timezone] = America/Sao_Paulo
```

Como o SeLinux está habilitado, vamos força-lo para o modo permissivo. Evita do Selinux bloquear a subida do Zabbix.

```
sudo setenforce 0
```

Verifincando o status do selinux

```
[vagrant@zabbix ~]$ getenforce
Permissive
```

Todo o bloqueio que o Selinux execurtaria, será logado. Essas informações serão uteis.

Instalar o pacote de troubleshooting do selinux

```
sudo dnf -y install setroubleshoot
```

Agora sim, podemos seguir com os procedimentos do site do Zabbix

f. Inicie o servidor Zabbix e os processos do agente
Inicie o servidor Zabbix e os processos do agente e configure-os para que sejam iniciados durante o boot do sistema.
```
sudo systemctl restart zabbix-server zabbix-agent httpd php-fpm
sudo systemctl enable zabbix-server zabbix-agent httpd php-fpm
```

Verificar se tudo bem com o Zabbix-server

```
sudo systemctl enable zabbix-server zabbix-agent httpd php-fpm

```

Resultado

```
zabbix-server.service - Zabbix Server
   Loaded: loaded (/usr/lib/systemd/system/zabbix-server.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2021-01-24 19:05:43 UTC; 1min 42s ago
 Main PID: 6866 (zabbix_server)
    Tasks: 38 (limit: 5047)
   Memory: 50.8M
   CGroup: /system.slice/zabbix-server.service
           ├─6866 /usr/sbin/zabbix_server -c /etc/zabbix/zabbix_server.conf
           ├─7087 /usr/sbin/zabbix_server: configuration syncer [synced configuration in 0.015741 sec, idle 60 sec]
           ├─7098 /usr/sbin/zabbix_server: housekeeper [startup idle for 30 minutes]
           ├─7099 /usr/sbin/zabbix_server: timer #1 [updated 0 hosts, suppressed 0 events in 0.000481 sec, idle 59 sec]
           ├─7100 /usr/sbin/zabbix_server: http poller #1 [got 0 values in 0.000370 sec, idle 5 sec]
           ├─7101 /usr/sbin/zabbix_server: discoverer #1 [processed 0 rules in 0.000381 sec, idle 60 sec]
           ├─7102 /usr/sbin/zabbix_server: history syncer #1 [processed 0 values, 0 triggers in 0.000022 sec, idle 1 sec]
           ├─7103 /usr/sbin/zabbix_server: history syncer #2 [processed 0 values, 0 triggers in 0.000015 sec, idle 1 sec]
           ├─7104 /usr/sbin/zabbix_server: history syncer #3 [processed 2 values, 2 triggers in 0.005147 sec, idle 1 sec]
           ├─7105 /usr/sbin/zabbix_server: history syncer #4 [processed 0 values, 0 triggers in 0.000011 sec, idle 1 sec]
           ├─7106 /usr/sbin/zabbix_server: escalator #1 [processed 0 escalations in 0.000454 sec, idle 3 sec]
           ├─7107 /usr/sbin/zabbix_server: proxy poller #1 [exchanged data with 0 proxies in 0.000007 sec, idle 5 sec]
           ├─7108 /usr/sbin/zabbix_server: self-monitoring [processed data in 0.000027 sec, idle 1 sec]
           ├─7109 /usr/sbin/zabbix_server: task manager [processed 0 task(s) in 0.000420 sec, idle 5 sec]
           ├─7110 /usr/sbin/zabbix_server: poller #1 [got 0 values in 0.000006 sec, idle 2 sec]
lines 1-22
```

