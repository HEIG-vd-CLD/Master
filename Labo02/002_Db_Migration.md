# Database migration

In this task you will migrate the Drupal database to the new RDS database instance.

![Schema](./img/CLD_AWS_INFA.PNG)

## Task 01 - Securing current Drupal data

### [Get Bitnami MariaDb user's password](https://docs.bitnami.com/aws/faq/get-started/find-credentials/)

```bash
[INPUT]
//help : path /home/bitnami/bitnami_credentials
sudo cat /home/bitnami/bitnami_credentials

[OUTPUT]
Welcome to the Bitnami package for Drupal

******************************************************************************
The default username and password is 'user' and 'vec2PoZB3aQ:'.
******************************************************************************

You can also use this password to access the databases and any other component the stack includes.

Please refer to https://docs.bitnami.com/ for more details.

```

### Get Database Name of Drupal

```bash
[INPUT]
mariadb -u root -p
// with password from previous step
//add string connection

show databases;

[OUTPUT]
+--------------------+
| Database           |
+--------------------+
| bitnami_drupal     |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+
6 rows in set (0.002 sec)
```

### [Dump Drupal DataBases](https://mariadb.com/kb/en/mariadb-dump/)

```bash
[INPUT]
mariadb-dump -u root -p bitnami_drupal > drupal_backup.sql 
[OUTPUT]
None
```

### Create the new Data base on RDS

```sql
[INPUT]
mariadb -h dbi-devopsteam10.cshki92s4w5p.eu-west-3.rds.amazonaws.com -u admin -p 
CREATE DATABASE bitnami_drupal;
```

### [Import dump in RDS db-instance](https://mariadb.com/kb/en/restoring-data-from-dump-files/)

Note : you can do this from the Drupal Instance. Do not forget to set the "-h" parameter.

```sql
[INPUT]
mariadb -h dbi-devopsteam10.cshki92s4w5p.eu-west-3.rds.amazonaws.com --user admin --password bitnami_drupal < drupal_backup.sql

[OUTPUT]
None
# Check if correctly loaded
SHOW TABLES FROM bitnami_drupal;
+----------------------------------+
| Tables_in_bitnami_drupal         |
+----------------------------------+
| block_content                    |
| block_content__body              |
| block_content_field_data         |
| block_content_field_revision     |
| block_content_revision           |
| block_content_revision__body     |
| cache_bootstrap                  |
| cache_config                     |
| cache_container                  |
| cache_data                       |
| cache_default                    |
| cache_discovery                  |
| cache_dynamic_page_cache         |
| cache_entity                     |
| cache_menu                       |
| cache_page                       |
| cache_render                     |
| cache_toolbar                    |
| cachetags                        |
| comment                          |
| comment__comment_body            |
| comment_entity_statistics        |
| comment_field_data               |
| config                           |
| file_managed                     |
| file_usage                       |
| help_search_items                |
| history                          |
| key_value                        |
| menu_link_content                |
| menu_link_content_data           |
| menu_link_content_field_revision |
| menu_link_content_revision       |
| menu_tree                        |
| node                             |
| node__body                       |
| node__comment                    |
| node__field_image                |
| node__field_tags                 |
| node_access                      |
| node_field_data                  |
| node_field_revision              |
| node_revision                    |
| node_revision__body              |
| node_revision__comment           |
| node_revision__field_image       |
| node_revision__field_tags        |
| path_alias                       |
| path_alias_revision              |
| router                           |
| search_dataset                   |
| search_index                     |
| search_total                     |
| semaphore                        |
| sequences                        |
| sessions                         |
| shortcut                         |
| shortcut_field_data              |
| shortcut_set_users               |
| taxonomy_index                   |
| taxonomy_term__parent            |
| taxonomy_term_data               |
| taxonomy_term_field_data         |
| taxonomy_term_field_revision     |
| taxonomy_term_revision           |
| taxonomy_term_revision__parent   |
| user__roles                      |
| user__user_picture               |
| users                            |
| users_data                       |
| users_field_data                 |
| watchdog                         |
+----------------------------------+
72 rows in set (0.001 sec)

```

### [Get the current Drupal connection string parameters](https://www.drupal.org/docs/8/api/database-api/database-configuration)

```bash
[INPUT]
//help : same settings.php as before
cat stack/drupal/sites/default/settings.php | grep -A 3 "^\$databases\['default'\]\['default'\]"

[OUTPUT]
//at the end of the file you will find connection string parameters
//username = bn_drupal
//password = XXXXXXX

$databases['default']['default'] = array (
  'database' => 'bitnami_drupal',
  'username' => 'bn_drupal',
  'password' => '2b9defd18a354804a1d4c4742c252fb39d808c12cfc2046ffc8f31432ae8a060',

```

### Replace the current host with the RDS FQDN

```
//settings.php

$databases['default']['default'] = array (
   [...] 
  'host' => 'dbi-devopsteam10.cshki92s4w5p.eu-west-3.rds.amazonaws.com',
   [...] 
);
```
```bash
sudo nano stack/drupal/sites/default/settings.php
'host' => 'dbi-devopsteam10.cshki92s4w5p.eu-west-3.rds.amazonaws.com',

```

### [Create the Drupal Users on RDS Data base](https://mariadb.com/kb/en/create-user/)

Note : only calls from both private subnets must be approved.
* [By Password](https://mariadb.com/kb/en/create-user/#identified-by-password)
* [Account Name](https://mariadb.com/kb/en/create-user/#account-names)
* [Network Mask](https://cric.grenoble.cnrs.fr/Administrateurs/Outils/CalculMasque/)

```sql
[INPUT]
mariadb -h dbi-devopsteam10.cshki92s4w5p.eu-west-3.rds.amazonaws.com -u admin -p
CREATE USER 'bn_drupal'@'10.0.10.0/255.255.255.240' IDENTIFIED BY '2b9defd18a354804a1d4c4742c252fb39d808c12cfc2046ffc8f31432ae8a060';

GRANT ALL PRIVILEGES ON bitnami_drupal.* TO 'bn_drupal'@'10.0.10.0/255.255.255.240';

//DO NOT FORGET TO FLUSH PRIVILEGES
FLUSH PRIVILEGES;

```

```sql
//validation
[INPUT]
SHOW GRANTS for 'bn_drupal'@'10.0.10.0/255.255.255.240';

[OUTPUT]
+----------------------------------------------------------------------------------------------------------------------------------+
| Grants for <yourNewUser>                                                                                                         |
+----------------------------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO <yourNewUser> IDENTIFIED BY PASSWORD 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'                           |
| GRANT ALL PRIVILEGES ON `bitnami_drupal`.* TO <yourNewUser>                                                                      |
+----------------------------------------------------------------------------------------------------------------------------------+

| Grants for bn_drupal@10.0.10.0/255.255.255.240                                                                                   |
+----------------------------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `bn_drupal`@`10.0.10.0/255.255.255.240` IDENTIFIED BY PASSWORD '*774097D0FF922910DD5E38A8BE4E6886FD3CA240' |
| GRANT ALL PRIVILEGES ON `bitnami_drupal`.* TO `bn_drupal`@`10.0.10.0/255.255.255.240`                                            |
+----------------------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.000 sec)


```

### Validate access (on the drupal instance)

```sql
[INPUT]
mariadb -h dbi-devopsteam10.cshki92s4w5p.eu-west-3.rds.amazonaws.com -u bn_drupal -p 
// enter password: 2b9defd18a354804a1d4c4742c252fb39d808c12cfc2046ffc8f31432ae8a060
[INPUT]
show databases;

[OUTPUT]
+--------------------+
| Database           |
+--------------------+
| bitnami_drupal     |
| information_schema |
+--------------------+
2 rows in set (0.001 sec)
```

* Repeat the procedure to enable the instance on subnet 2 to also talk to your RDS instance.

```sql
[INPUT]
mariadb -h dbi-devopsteam10.cshki92s4w5p.eu-west-3.rds.amazonaws.com -u admin -p
CREATE USER 'bn_drupal'@'10.0.10.128/255.255.255.240' IDENTIFIED BY '2b9defd18a354804a1d4c4742c252fb39d808c12cfc2046ffc8f31432ae8a060';

GRANT ALL PRIVILEGES ON bitnami_drupal.* TO 'bn_drupal'@'10.0.10.128/255.255.255.240';

//DO NOT FORGET TO FLUSH PRIVILEGES
FLUSH PRIVILEGES;

SHOW GRANTS for 'bn_drupal'@'10.0.10.128/255.255.255.240';
+------------------------------------------------------------------------------------------------------------------------------------+
| Grants for bn_drupal@10.0.10.128/255.255.255.240                                                                                   |
+------------------------------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `bn_drupal`@`10.0.10.128/255.255.255.240` IDENTIFIED BY PASSWORD '*774097D0FF922910DD5E38A8BE4E6886FD3CA240' |
| GRANT ALL PRIVILEGES ON `bitnami_drupal`.* TO `bn_drupal`@`10.0.10.128/255.255.255.240`                                            |
+------------------------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.000 sec)
```