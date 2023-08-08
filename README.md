# **pg_monz - Template para monitoramento de PostgreSQL e PgPool**

Este repositório foi um fork do projeto [pg_monz](https://github.com/pg-monz/pg_monz.git) que até este momento foi o único projeto disponível para a comunidade que realmente propôs o monitoramento em conjunto das duas ferramentas.
Porém há algum tempo, cerca de 2 anos, o projeto não foi atualizado e muitas issues acabaram ficando esquecidas.
Diante da minha necessidade de utilização, consegui realizar alterações nos códigos afim de colocar o monitoramento para funcionar.
E este é o resultado de alguns esforços.

A versão do zabbix utilizado foi a 5.0 LTS, com PostgreSQL v15.3 e PgPool v4.4.


## Instalação

1. Requisitos:

- Zabbix server version 2.0+
- monitored PostgreSQL  9.2+
- monitored pgpool-II   3.4.0
- installing zabbix_agentd to PostgreSQL and pgpool-II host.
- installing zabbix_sender to PostgreSQL and pgpool-II host.
- ServerActive setting in zabbix_agentd.conf (zabbix_sender read this file)
- psql command(and serch path setting) on each PostgreSQL/pgpool server

2. Importe os templates de monitoramento no Zabbix Web.

  Template_App_PostgreSQL.xml
  Template_App_PostgreSQL_SR.xml
  Template_App_PostgreSQL_SR_Cluster.xml
  Template_App_pgpool-II.xml
  Template_App_pgpool-II-36.xml
  Template_App_pgpool-II_watchdog.xml

3. Copie os arquivos da pasta `usr-local-etc` (pgsql_funcs.conf and pgpool_funcs.conf) para `/usr/local/etc`:

```bash 
cp usr-local-etc/* /usr/local/etc

```
Valores padrão de variáveis do arquivo `pgsql_funcs.conf`

```bash
   ----
   PGHOST=127.0.0.1
   PGPORT=5432
   PGROLE=postgres
   PGDATABASE=postgres
   ----
```

Valores padrão de variáveis do arquivo `pgpool_funcs.conf`
```bash
   ----
   PGPOOLHOST=127.0.0.1
   PGPOOLPORT=9999
   PGPOOLROLE=postgres
   PGPOOLDATABASE=postgres
   PGPOOLCONF=/usr/local/etc/pgpool.conf
   ----
```

> A definição do diretório de configuração é feita via variável macro no tempalte.
```bash
       {$PGSCRIPT_CONFDIR}
       {$PGPOOLSCRIPTS_CONFDIR}
```

4. Copie os arquivos da pasta `usr-local-bin`  para `/usr/local/bin` e dê permissão de execução para todos:

```bash
cp usr-local-bin/* /usr/local/bin
chmod +x /usr/local/bin/*.sh
```

> A definição do diretório de scripts de execução é feita via variável macro no template.
```bash
       {$PGSCRIPTDIR}
       {$PGPOOLSCRIPTDIR}
```

5. Copie o arquivo de configuração do parâmetro do usuário para o agente Zabbix `userparameter_pgsql.conf` para o local especificado da máquina que possui o agente instalado.

Por exemplo, se o agente Zabbix estiver instalado em `/etc/zabbix/`, copie o arquivo para o seguinte local:

```bash
cp zabbix_agentd.d/userparameter_pgpsql.conf /etc/zabbix/zabbix_agentd.conf.d/userparameter_pgsql.conf
```

> Certifique-se que no arquivo de configuração do agent o `Include` para esta pasta está ativado conforme exemplo abaixo:
```bash
 Include=/etc/zabbix/zabbix_agentd.conf.d/
```

6. Definição de grupos usando Zabbix Web.

- Crie o grupo "PostgreSQL" e adicione o Host PostgreSQL no grupo criado.
- Crie o grupo "pgpool" e adicione o host pgpool-II no grupo "pgpool".

> Cada grupo é referenciado nos templates Template_App_PostgreSQL_SR_Cluster.xml e Template_App_pgpool-II_watchdog.xml usando a variável `{$HOST_GROUP}` no macro dotemplate.

7. Verifique o caminho de diretório do arquivo zabbix_agentd.conf.

Se o arquivo `zabbix_agentd.conf` não está em `/etc/zabbix/zabbix_agentd.conf`, adicione o caminho correto como valor na variável `{$ZABBIX_AGENTD_CONF}` macro.

> Exemplo de como definir do caminho do arquivo `zabbix_agentd.conf`  na variável macro no host ou template.
```bash
     {$ZABBIX_AGENTD_CONF} => /etc/zabbix/zabbix_agentd.conf

```

8. Link templates para os hosts.

Link "Template App PostgreSQL SR" para os hosts PostgreSQL.

Link "Template App pgpool-II" para os hosts pgpool-II.

"Template App PostgreSQL SR Cluster"/ "Template App pgpool-II-watchdog" estão simplesmente contando:

- o número de serviço em execução (sr/pgpool-II),
- o número do servidor principal (sr),
- o número de servidor em espera (sr),
- o número de delegate_ip (pgpool-II),

Usando a chave agregada no zabbix `{$HOST_GROUP}`.

> Se você desejar monitorar o split-brain ou o número de servidores primários, basta vincular os templates acima ao host arbitrário.
> exemplo: "PostgreSQL Cluster" como host virtual.

## Dicas

A autenticação nos serviços PostgreSQL e PgPool pode ser feita de 2 (duas) formas, a primeira indicando o arquivo pgpass dentro do arquivo `pgsql_funcs.conf` e a segunda forma é colocando o pgpass para ser utilizado na sessão do usuário Zabbix.
Vou mostrar as duas formas, eu utilizei o pgpass no usuário Zabbix e funcionou certinho.

### Primeira forma:

Adicione a linha abaixo no arquivo `pgsql_funcs.conf`:

```bash
export PGPASSFILE=/usr/local/etc/pgpass
```

Crie o arquivo `/usr/local/etc/pgpass` com o editor de sua preferência, e coloque as informações de conexão no padrão abaixo:

```bash
127.0.0.1:5432:*:postgres:somepassword
```

Após isso dê permissão de somente leitura no arquivo e também para o usuário zabbix.

```bash
chmod 0600 /usr/local/etc/pgpass
chown zabbix:zabbix /usr/local/etc/pgpass
```

### Segunda forma:

Crie o arquivo `/home/zabbix/.pgpass` com o editor de sua preferência, e coloque as informações de conexão no padrão abaixo:

```bash
127.0.0.1:5432:*:postgres:somepassword
```

Após isso dê permissão de somente leitura no arquivo e também para o usuário zabbix.

```bash
chmod 0600 /home/zabbix/.pgpass
chown zabbix:zabbix /home/zabbix/.pgpass
```

A alteração da permissão é importante pois a criação do arquivo é feita com o usuário corrente e assim as permissões são herdadas deste.

Esta forma de configuração facilita pois já é uma prática utilizada para conexão via sessão utilizando o utilitário psql.


pg_monz 2.0
============================
pg_monz (PostgreSQL monitoring template for Zabbix) is a Zabbix template for
monitoring PostgreSQL. It enables various types of monitoring of PostgreSQL
such as alive, resource, performance, etc.
Pg_monz also supports automatic discovery of databases and tables using the
discovery feature of Zabbix and can automatically start monitoring.


Changes from 1.0
----------------
The following is a summary of the major changes.


### Support for monitoring of PostgreSQL Streaming Replication
pg_monz 2.0 now supports monitoring of Streaming Replication which is embedded in PostgreSQL since 9.0.  
Various monitoring items such as Primary / Standby servers alive monitoring, delay of replication data propagation and conflicts occurred by operation to Primary and Standby are available.
It is also that a trigger which can detect the occurence of write block query when useing synchronous replication is provided.


### Support for monitoring of pgpool-II
pg_monz 2.0 now supports monitoring of pgpool-II which is a dedicated middleware for PostgreSQL.  
Various types of monitoring and triggers for the main features of pgpool-II such as Connection Pooling, Replication, In memory query Cache, Load Balance, Automatically Failover of PostgreSQL are provided.

Please see [pgpool-II user manual](http://www.pgpool.net/docs/latest/pgpool-en.html), [pgpool Wiki](http://www.pgpool.net/mediawiki/index.php/Main_Page) for more detailed informations.


### Support for monitoring of cluster system with PostgreSQL + pgpool-II
And more, it make it possible to monitor a cluster system which is configured with PostgreSQL Streaming replication and pgpool-II or pgpol-II watchdog which add high availability to themselves.  
Useful triggers which can detects the occurence of split brain, failover are provided through monitoring of postgres, pgpool-II processes.


### Group items
Monitoring items are grouped by each application to clarify them.  
The following are main applications.


#### Applications | PostgreSQL
|application name   |summary of monitoring                                                                            |
|:------------------|-------------------------------------------------------------------------------------------------|
|pg.transactions    |Connection count, state to PostgreSQL, the number of commited, rolled back transactions          |
|pg.log             |log monitoring for PostgreSQL                                                                    |
|pg.size            |garbage ratio, DB size                                                                           |
|pg.slow_query      |slow query count which exceeds the threshold value                                               |
|pg.sr.status       |conflict count, write block existence or non-existence, process count using Streaming Replication|
|pg.status          |PostgreSQL processes working state                                                               |
|pg.stat_replication|delay of replication data propagation using Streaming Replication                                |
|pg.cluster.status  |PostgreSQL processes count as a cluster                                                          |


#### Applications | pgpool-II
|application name   |summary of monitoring                                                                            |
|:------------------|-------------------------------------------------------------------------------------------------|
|pgpool.cache       |cash informations using In Memory query Cache                                                    |
|pgpool.connections |frontend, backend connection count through pgpool-II                                             |
|pgpool.log         |log monitoring for pgpool-II                                                                     |
|pgpool.nodes       |backend state, load balance ratio and replication delay viewed from pgpool-II                                          |
|pgpool.status      |pgpool-II processes working state, vip existence or non-existence                                |
|pgpool.watchdog    |pgpool-II processes working state, vip existence or non-existence as a cluster                   |


### Improve performance of gathering monitoring items
Previously, pg_monz accesses the monitoring DB every when gathering one monitoring item about DB, which may affect the performance of monitoring DB.
With this update, to reduce the frequency of DB accesse, pg_monz gathers collectable monitoring items all at once.


System requirements
-------------------
pg_monz requires the following software products:

* Zabbix server, zabbix agent, zabbix sender 2.0 or later
* PostgreSQL 9.2 or later
* pgpool-II 3.4.0 or later


Installation and usage
----------------------
Please see the included quick-install.txt.  
pg_monz 2.0 does not have backward compatibility with the 1.0. When upgrading from 1.0, please install the new version again.


License
-------
pg_monz is distributed under the Apache License Version 2.0.
See the LICENSE file for details.