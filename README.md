# Introduction

[![APEX Community](https://cdn.rawgit.com/Dani3lSun/apex-github-badges/78c5adbe/badges/apex-community-badge.svg)]() [![APEX Tool](https://cdn.rawgit.com/Dani3lSun/apex-github-badges/b7e95341/badges/apex-tool-badge.svg)]() [![APEX 18.2](https://cdn.rawgit.com/Dani3lSun/apex-github-badges/2fee47b7/badges/apex-18_2-badge.svg)]() [![APEX Built with Love](https://cdn.rawgit.com/Dani3lSun/apex-github-badges/7919f913/badges/apex-love-badge.svg)]()

With DOCKAWEX you can easily create your local APEX development environment consisting of Oracle Database 18c XE, Tomcat with ORDS and an additional node proxy. Additionally you have the possibility to remotely build a container architecture for your APEX project via docker-machine and the included drivers. Here your containers will be additionally secured via Let's Encrypt and nginx and made known in the cloud.


---

## System Requirements

- [Docker](https://www.docker.com)
- [Docker-Compose](https://www.docker.com)
- [AWS-CLI](https://aws.amazon.com/de/cli/) (optional)

## Download Software

For licensing reasons, you must host or provide the software packages to be installed yourself.


File                                           | What / Link
---------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
oracle-database-xe-18c-1.0-1.x86_64.rpm        | https://www.oracle.com/database/technologies/xe-downloads.html <br/>- Oracle DB 18c XE
jre-8u251-linux-x64.tar.gz                     | https://javadl.oracle.com/webapps/download/AutoDL?BundleId=242050_3d5a2bb8f8d4428bbe94aed7ec7ae784 <br/>- JRE which will run tomcat
apache-tomcat-8.5.54.tar.gz                    | http://mirror.23media.de/apache/tomcat/tomcat-8/v8.5.54/bin/apache-tomcat-8.5.54.tar.gz <br/>- Applicationserver, here will ORDS installed to
ords-19.4.0.352.1226.zip                       | https://www.oracle.com/database/technologies/appdev/rest.html <br/>- Oracle REST Data Services, will provide access to APEX
apex_20.1.zip                                  | https://www.oracle.com/tools/downloads/apex-downloads.html <br/>- APEX complete
instantclient-basiclite-linux.x64-19.6.0.0.0dbru.zip   | http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html <br/>- Clientssoftware, to connect to oracle db
instantclient-sqlplus-linux.x64-19.6.0.0.0dbru.zip | http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html <br/>- SQLPlus<br/>- install APEX and run your scripts and deployments


When you are building against **local** machine, you can pack all files into the directories called "_binaries"
```shell
infrastructure/docker/appsrv/_binaries
  apache-tomcat-8.5.54.tar.gz
  apex_20.1.zip
  instantclient-basiclite-linux.x64-19.6.0.0.0dbru.zip
  instantclient-sqlplus-linux.x64-19.6.0.0.0dbru.zip
  jre-8u251-linux-x64.tar.gz
  ords-19.4.0.352.1226.zip
infrastructure/docker/oradb/_binaries
  oracle-database-xe-18c-1.0-1.x86_64.rpm
```
From here all files are copied into the respective image.  
Otherwise when building against remote machine, load the files into a directory of your choice (your website, S3, ...) from where they must be accessible via http(s). The URL for downloading these files has to be placed in your corresponding env-file.

## Local installation as development environment

The configurations of the individual machines are stored in the "machines/*" directory. Here, each directory represents a machine. The local machine is also stored here. It is located in the folder **default**. Each directory must contain a .env - file. Here you have to store some configurations. Feel free to copy the **_template**-folder, rename it and change vars as you need.

### 1. Modify environment vars inside machines/default/.env

```bash
# Binaries to use
export DOWNLOAD_URL=https://your-url-pointing-to-binaries

export FILE_DB=oracle-database-xe-18c-1.0-1.x86_64.rpm
export FILE_ORDS=ords-19.4.0.352.1226.zip
export FILE_TOMCAT=apache-tomcat-8.5.54.tar.gz
export FILE_APEX=apex_20.1.zip
export FILE_SQLPLUS=instantclient-sqlplus-linux.x64-19.6.0.0.0dbru.zip

export FILE_JRE=jre-8u251-linux-x64.tar.gz
# if you extract the tar.gz this is the name of the directory inside
export FILE_JRE_VERSION=jre1.8.0_251  
 
export FILE_CLIENT=instantclient-basiclite-linux.x64-19.6.0.0.0dbru.zip
# if you extract the zip this is the name of the directory inside
export FILE_INSTANT_CLIENT_VERION=instantclient_19_6

# Timezone 
export TIME_ZONE=Europe/Berlin

# DB Passes
export DB_SID=xepdb1
export DB_PASSWORD=SecurePwd123
export TOM_PASSWORD=SecurePwd123
export ORDS_PASSWORD=SecurePwd123
export PORTAINER_PWD=SecurePwd123

# APEX properties
export APEX_USER=APEX_200100
export INTERNAL_MAIL=your.admin-mail@somewhere.com

# Point to and certificate 
export VIRTUAL_HOST=your.domain.com
export LETSENCRYPT_HOST=your.domain.com
export LETSENCRYPT_EMAIL=your.admin-mail@somewhere.com
# during test comment that out because letsencrypt does not allow
# more than 5 calls per week
#- LETSENCRYPT_TEST=true
# with that info we say hello to our dyndns-service

# curl to
export DDNS_USER=your-ddns-user
export DDNS_PASSWORD=with-password-when-using-curl
export DDNS_URL=ddns.server.org

# mail properties
export SMTP_HOST_ADDRESS=your.smtp-server.com
export SMTP_FROM=default-from-name
export SMTP_USERNAME=your-user
export SMTP_PASSWORD=and-pwd
```

### 2. Call script **infra_local.sh** to manage your local setup

1. change your working directory to dockawex: ```cd dockawex```
2. build images: ```dockawex$> ./infra_local.sh build```
3. start container: ```dockawex$> ./infra_local.sh start```


> ready...
At http://localhost:8080/ords (or 192.168.99.100/ords when using docker-toolbox) APEX is waiting for you

More parameters will be displayed if you omit the parameters. 
```bash
  $ ./infra_local.sh 
  Please call script by using all params in this order!
      ././infra_local.sh command
  -----------------------------------------------------

    command
      > build  > builds only images
      > start  > start images / containers
      > run    > builds and start images / containers
      > logs   > shows logs by executing logs -f
      > list   > list services
      > config > view compose files
      > print  > print compose call
      > stop   > stops services
      > clear  > removes all container and images, prunes allocated space
```

> If you have filled the email and smtp vars you will receive a message. 
```sql
apex_mail.send(p_from => '${SMTP_FROM}'
              ,p_to   => '${INTERNAL_MAIL}'
              ,p_subj => 'DOCKAWEX successfully installed'
              ,p_body => 'Test Hey ho, that works');
```

| Typ              | Link |
|------------------|-------------------------------------------|
| APEX             | http://localhost:8080/ords                |
| Portainer        | http://localhost:9000                     |
| SQLDeveloper Web | http://localhost:8080/ords/sql-developer  |
| DB               | \<user>/\<pass>@localhost:1521/xepdb1     |



---

## Remote installation for the public

### Prerequisite [Create an AWS account and authorized user](_docs/prepare_aws.md)

### 1. Modify AWS Settings

After you have installed and configured aws-cli and created an IAM user, you can now write your account parameters to an environment file **account-alias.env**.

1. copy the file _template.env and name it descriptive e.g. your-aws-settings.env
2. Change following parameters:

```bash
ACCESS_KEY_ID=<Your AWS Access Key ID>
SECRET_ACCESS_KEY=<Your AWS Secret Access Key>
VPC_ID=<Your AWS VPC ID>
```

### 2. Create a copy of the directory "_template" an name it descriptive e.g.  your-machine

### 3. Customizing the Infrastructure Settings ```machines/your-machine/custom-compose.yml```
#TODO: Hier fehlt text


### 4. Call script **setup_remote.sh** to manage your remote setup

1. change your working directory to the machines path: ```cd dockawex/machines```
2. build images: ```dockawex/machines$> ./setup_remote.sh your-aws-settings your-machine-name build```
3. start container: ```dockawex/machines$> ./setup_remote.sh your-aws-settings your-machine-name start```

> More parameters will be displayed if you omit the parameters. ```awex/machines$> ./awex_remote.sh```

ready...
Check https://your-sub.domain.de/ords APEX is waiting ...


---
# FAQ

1. What is the password to internal?
> It is the same as for the user sys, except with an exclamation mark at the end!

2. How can I use machines on PC2 when the are created on PC1?
> see: https://www.npmjs.com/package/machine-share

3. How can I test access to external resources via APIproxy?
```sql
    select apex_web_service.make_rest_request(
             p_url => 'http://nodeprx:3000/https/postman-echo.com/time/start?timestamp=2020-04-29&unit=month',
             p_http_method => 'GET')
      from dual
```

4. What are the login-credentials when using SQL Developer Web?
> you have to REST enable the database-schema
``` sql
  begin
    ords.enable_schema(p_enabled => TRUE,
                       p_schema => 'HR',
                       p_url_mapping_type => 'BASE_PATH',
                       p_url_mapping_pattern => 'hr',
                       p_auto_rest_auth => FALSE);
    commit;
  end;
  ```
  After that you can publish RESTful Service, REST Enable object and login SQL Developer Web. You can switch that off by editing infrastructure/docker/appsrv/scripts/ords_params.properties.


---
# Credits
Dockerfiles are based on and with the influence of:
- https://github.com/araczkowski/docker-oracle-apex-ords
- https://github.com/jwilder/nginx-proxy

Some inspirations are coming from:
- https://github.com/Dani3lSun/docker-db-apex-dev

