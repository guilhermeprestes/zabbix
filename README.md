# Zabbix 6 em 3 camadas 
documentação para instalar o zabbix em 3 camadas, uma com banco de dados, uma com parte web e a outra com o zabbix server.

## Requisitos:

| 1 server para Banco de Dados (MySQL) | 1 server para o Frontend (NGINX) | 1 server para o Backend (Zabbix server)  |
|----------|----------|----------|
| Memory: 8 GB      | Memory: 8 GB      | Memory: 8 GB      |
| VCPU: 6      | VCPU: 6      | VCPU: 6      |
| Disk: 50 GB      | Disk: 32 GB      | Disk: 32 GB      |
| SO: Ubuntu 22.04      | SO: Ubuntu 22.04      | SO: Ubuntu 22.04      |


## Server para Banco de Dados (MySQL)

### Instalar e configurar o Mysql

com o pacotes da maquina atualizada digite o comando para instalar a instancia do MySQL

```BASH
sudo apt install mysql-server
```
inicie o serviço:
```BASH
sudo systemctl start mysql.service
```
ative o serviço para iniciar com sistema
```BASH
sudo systemctl enable mysql.service
```
execute o script para configuração
```BASH
sudo mysql_secure_installation
```
acesse com usuario root para testar 

```BASH
mysql -u root -p
```
> a senha é a que você definiu na execução do script
{.is-warning}

para sair execute o comando <kbd> quit </kbd>
___
Por padrão, o MySQL fica “ouvindo” conexões somente para localhost, isto é, apenas para
a própria máquina. Como o Zabbix Server precisa conectar o banco pela rede, temos que fazer uma alteração dentro do arquivo de configuração do Server MySQL. Portanto:
```BASH
 vim /etc/mysql/mysql.conf.d/mysqld.cnf 
```

altere ou adicione:
> bind_address=0.0.0.0

Salve e saia do arquivo; 
e faça um restart no serviço do MySQL
```BASH
sudo systemctl restart mysqld.service
```

Confirme se a configuração ficou ok usando o comando
```BASH
ss -ptln
```
![ss_-ptln.png](/zabbix/ss_-ptln.png)

### Criar e configurar o Banco e usuarios

acesse com usuario root

```BASH
mysql -u root -p
```

Dentro da console do MySQL, vamos criar o banco de dados com o nome zabbix, um
usuário zabbix e a senha zabbix, com permissão para acessar seu próprio banco:

```MySQL
create database zabbix character set utf8 collate utf8_bin;
```
```MySQL
create user zabbix@'%' identified by 'SENHAFORTE';
```
```MySQL
grant all privileges on zabbix.* to zabbix@'%';
```
```MySQL
set global log_bin_trust_function_creators = 1;
```
para sair execute o comando <kbd> quit </kbd>

### Baixar e executar os scripts para gerar as tabelas do banco
Instale o repositorio do zabbix
```BASH
wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4+ubuntu22.04_all.deb
dpkg -i zabbix-release_6.0-4+ubuntu22.04_all.deb
apt update
```
execute a instalação dos scripts
```BASH
apt install zabbix-sql-scripts
```
execute o script para gerar as tabelas
```BASH
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

## Server para o Backend (Zabbix server)
Instale o repositório do zabbix 
```BASH
wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4+ubuntu22.04_all.deb
dpkg -i zabbix-release_6.0-4+ubuntu22.04_all.deb
apt update
```
execute a instalação do zabbix server
```BASH
apt install zabbix-server-mysql zabbix-agent
```
Edite o arquivo ``vim /etc/zabbix/zabbix_server.conf``, procure os parâmetros <kbd>DBPassword</kbd> e <kbd>DBHost</kbd> e coloque a senha do usuário zabbix para o banco de dados.

>No parâmetro DBHost, insira o ip da máquina em que será criado o banco de dados. Então,
temos:
DBHost=IP-SERVIDOR
DBPassword=SENHAFORTE

Reinicie a aplicação do zabbix e ative 
```BASH
systemctl restart zabbix-server
systemctl enable zabbix-server
```

## Server para o Frontend (NGINX)

Instale o repositório do zabbix 
```BASH
wget https://repo.zabbix.com/zabbix/6.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.0-4+ubuntu22.04_all.deb
dpkg -i zabbix-release_6.0-4+ubuntu22.04_all.deb
apt update
```
execute a instalação do Nginx
```BASH
apt install zabbix-frontend-php zabbix-nginx-conf zabbix-agent
```

Editar o arquivo
` vim /etc/nginx/conf.d/zabbix.conf `
Descomentar os parâmetros ` listen ` e ` server_name `
Remover <kbd>#</kbd> da frente dos parâmetros:
>listen 8080;
server_name IP-SERVIDORZABBIX


Editar o arquivo

`` vim /etc/zabbix/php-fpm.conf ``

acrescente o seguinte valor para time zone
>php_value[date.timezone] = America/Sao_Paulo

reinicie e ative os serviços do NGINX e PHP

```BASH
systemctl restart nginx php8.1-fpm
systemctl enable nginx php8.1-fpm
```

> se tiver com Apache instalado remova com o comando `Purge`
{.is-warning}


```BASH
apt purge apache2
```

para acessar utilize o ip do servidor Frontend
> http://IP-SERVIDOR:8080/

![zabbix6.png](/zabbix/zabbix6.png)

