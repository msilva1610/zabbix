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
