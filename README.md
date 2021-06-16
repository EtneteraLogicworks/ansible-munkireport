# Role munkireport

Role pro nasazení webové aplikace MunkiReport. Pro nasazení nové verze je nutné
nastavit proměnnou `update` na true například na příkazové řádce: `-e update=true`.

Předpoklady:

- Připravený webový prostor
- Připravená MariaDB/MySQL databáze
- `composer` nainstalovaný v `/usr/local/bin`

Všechny tyto požadavky řeší role `webhosting`.

## Proměnné

Parametry role jsou definovány ve slovníku `munkireport`. Tato proměnná má následující položky:

| Proměnná         | Povinná | Výchozí     | Popis |
| ---------------- | ------- | ----------- | ----- |
| repo             | ne      | `https://github.com/munkireport/munkireport-php.git` | git repozitář s MunkiReport projektem |
| version          | ne      | master      | verze aplikace instalované skrz git |
| user             | ne      | `www-data`  | Uživatel vlastnící soubory webu |
| group            | ne      | `www-data`  | Skupina vlastníka souborů webu |
| webroot          | ano     |             | Cesta k webroot adresáři, kde je web umístěn |
|                  |         |             |       |
| site             | ne      |             | Slovník s konfigurací webových aspektů aplikace |
| .name            | ne      | MunkiReport | Jméno webové aplikace (viz. `SITENAME`) |
|                  |         |             |       |
| customers        | ne      |             | Pole slovníků s informacemi o Munki zákaznících. Typicky se chceme odkázat na datovou strukturu sdílenou s rolí `munki` |
| .munkireport     | ne      |             | Slovník s informacemi pro munkireport. |
| ..business_unit  | ne      |             | Jméno MunkiReport BU |
| ..groups         | ne      |             | Pole slovníků s informacemi o MunkiReport skupinách daného zákazníka. |
| ...name          | ne      |             | Jméno skupiny. |
| ...uuid          | ne      |             | UUID skupiny. Používá se jako shared secret pro autorizaci klientů |
|                  |         |             |       |
| auth             | ne      |             | Slovník pro konfiguraci uživatelů a autentizace |
| .local_users     | ne      |             | Pole slovníků pro konfiguraci lokálních uživatelů |
| ..name           | ano     |             | Jméno lokálního uživatele |
| ..password_hash  | ano     |             | Hash uživatele. Možno vygenerovat na URL `/index.php?/auth/generate` |
| .admin_users     | ne      |             | Pole uživatelů s rolí admin (viz. `ROLES_ADMIN`)  |
| .methods         | ne      |             | Pole povolených autentizačních mehod (viz. `AUTH_METHODS`)  |
| .ad              | ne      |             | Slovník pro konfiguraci ActiveDirectory/LDAP autentizace |
| ..enabled        | ne      |             | Zapnutí konfigurace ActiveDirectory/LDAP autentizace |
| ..hosts          | ano     |             | Čárkou oddělený seznam AD/LDAP serverů |
| ..port           | ne      | 389         | TCP port AD/LDAP serveru |
| ..tls            | ne      | true        | Zapne TLS pro spojení s AD/LDAP serverem |
| ..base_dn        | ano     |             | Báze vyhledávání |
| ..schema         | ne      | Active Directory | Typ schéma. (viz. `AUTH_AD_SCHEMA`) |
| ..account_prefix | ano     |             | Část DN před jménem uživatele |
| ..account_suffix | ano     |             | Část DN za jménem uživatele |
| ..username       | ano     |             | Jméno servisního AD/LDAP uživatele |
| ..password       | ano     |             | Heslo servisního AD/LDAP uživatele |
| ..allowed_users  | ano     |             | Seznam uživatelů, který bude povolena ActiveDirectory/LDAP autentizace |
|                  |         |             |       |
| database         | ano     |             | Slovník pro konfiguraci přístupu do databáze |
| .driver          | ne      | mysql       | Typ databáze |
| .host            | ne      | 127.0.0.1   | Adresa nebo doménové jméno databázového serveru |
| .port            | ne      | 3306        | TCP port databázového serveru |
| .name            | ano     |             | Jméno databáze |
| .user            | ano     |             | Databázový uživatel |
| .password        | ano     |             | Heslo databázového uživatele |
| .engine          | ne      | InnoDB      | MySQL table engine |
| .dump_path       | ne      | /var/backups/munkireport_preupgrade.sql | Cesta k souboru zálohy prováděné před upgrade schéma databáze |
|                  |         |             |       |
| http_proxy       | ne      |             | Slovník pro konfiguraci HTTP proxy |
| .server          | ne      |             | Adresa HTTP proxy serveru |
| .port            | ne      |             | TCP port HTTP proxy serveru |
|                  |         |             |       |
| apps_to_track    | ne      |             | Seznam sledovaných aplikací (viz. `APPS_TO_TRACK`) |
|                  |         |             |       |
| modules          | ano     |             | Slovník s nastavením MunkiReport modulů |
| .enabled         | ano     |             | Seznam zapnutých MunkiReport modulů |
| .extra           | ne      |             | Slovník pro konfigurace extra MunkiReport modulů pro composer |
| ..name           | ano     |             | Název extra modulu |
| ..vendor         | ano     |             | Vývojář extra modulu |
| ..requirement    | ano     |             | Composer požadavek na verzi modulu |
| .hide_inactive   | ne      | false       | Skrytí neaktivních modulů z web UI (viz. `HIDE_INACTIVE_MODULES`) |
| .raw_config      | ne      |             | RAW MunkiReport konfigurace modulů |
| .search_paths    | ne      |             | Cesta pro vyhledávání MunkiReport extra modulů (viz. `MODULE_SEARCH_PATHS`) |


## Příklad konfigurace

```yaml
munki_customers
  - name: 'customer_xyz'
      password: '{{ munki_customer_xyz_password }}'
      munkireport:
        business_unit: 'Customer XYZ'
        groups:
          - name: 'customer_xyz_corporate'
            uuid: '{{ munki_customer_xyz_corporate_uuid }}'
  ...

munkireport:
  user: 'w_testreport'
  group: 'w_testreport'
  webroot: '/srv/www/sites/c_lws/w_testreport/www'
  customers: '{{ munki_customers }}'
  site:
    name: 'MunkiReport Beta'
  auth:
    local_users:
      - name: 'admin'
        password_hash: '{{ munkireport_password_hash_admin }}'
    admin_users:
      - 'mradmin'
      - 'ad.user'
    methods:
      - 'LOCAL'
      - 'AD'
    ad:
      enabled: true
      hosts: 'ldap.logicworks.cz'
      base_dn: 'o=logicworks,dc=logicworks,dc=cz'
      schema: 'OpenLDAP'
      account_prefix: 'uid='
      account_suffix: ',ou=people,o=logicworks,dc=logicworks,dc=cz'
      username: 'cn=readuser,dc=logicworks,dc=cz'
      password: '{{ munkireport_ldap_password }}'
      allowed_users:
        - 'ad.user'
  database:
    engine: 'InnoDB ROW_FORMAT=DYNAMIC'
    name: 'w_testreport'
    user: 'w_testreport'
    password: '{{ munkireport_db_password }}'
  http_proxy:
    server: '192.168.0.1'
    port: '3128'
  apps_to_track:
    - 'Safari'
    - 'Skype'
    - 'VLC'
  modules:
    enabled:
      - 'applications'
      - 'ard'
      - 'bluetooth'
      ...
    extra:
      - name: 'teamviewer'
        vendor: 'tuxudo'
        requirement: '>=1.1'
    hide_inactive: true
    raw_config: |
      DISK_REPORT_THRESHOLD_DANGER=10
      DISK_REPORT_THRESHOLD_WARNING=20
    search_paths:
      - '/srv/www/sites/c_lws/w_testreport/www/vendor/tuxudo'
```
