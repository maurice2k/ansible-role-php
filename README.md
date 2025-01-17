# Ansible PHP role

This is a very easy role to install PHP on a Debian based system by using [Surys PHP packages](https://packages.sury.org/php/pool/main/p/).

> If you just add this role to your playbook, you'll get a PHP 8.3 CLI installation with a set of pre-defined, often used extensions (see `php_default_extensions`).

### Why another PHP role?
Most roles you'll find on GitHub or Ansible Galaxy use a myriad of variables and options to install PHP. This role is meant to be as simple as possible and only installs PHP with the extensions you need. Having a variable for each and every PHP ini setting makes the role hard to maintain and hard to use.

This role assumes that the PHP defaults are basically fine and you only adjust some ini settings here and there. This is also done using additional ini files so that you don't have to touch the main `php.ini` which makes package updates much easier.


## Configuration

The role can be configured by setting the following variables in your playbook.
If nothing is set, it will install PHP 8.3 with the extensions defined in `php_default_extensions`. 

The timezone is automatically set to `Europe/Berlin` (but can be adjusted using the `php_timezone` variable).

```yaml
php_version: "8.3"

php_extensions:  ## this is installed in addition to the default extensions
  - gd
  - grpc

php_pecl_extensions:
  - something-not-packaged-by-sury
```

### Extensions

The default way is to go with the set of pre-defined extensions in `php_default_extensions`. If you need additional extensions, you can add them to `php_extensions`.

If you require full control over the extensions that get installed, you can set `php_default_extensions` to an empty list and add all extensions you need to `php_extensions`.

PECL extensions can be specified in `php_pecl_extensions`.

### php.ini
If you need adjustments to `php.ini`, the best way is to overwrite them with additional ini files for CLI and FPM (see below) respectively.

> All ini files are Jinja2 templates and you can use variables in them.

Create the following file structure in your `{{ playbook_dir }}/files` directory:

```
php
|── cli
│   └── conf.d
│       └── 99-custom.ini
└── fpm
    |── conf.d
    |   └── 99-custom.ini
    └── pool.d
        └── app.conf    ## see FPM section
    
```

### FPM

By default, no FPM extension will be installed. If you want to use FPM, create one or more pool configuration files in `{{ playbook_dir }}/files/php/fpm/pool.d`. This will automatically enable FPM with the pool(s).

> All pool files are Jinja2 templates and you can use variables in them.

A sample configuration (i.e. `app.conf`) could look like this:

```
[app]

user = www-data           ; this should be your project user
group = www-data

listen = 127.0.0.1:9001
listen.owner = www-data   ; this is fine for nginx
listen.group = www-data

pm = dynamic
pm.max_children = 100
pm.start_servers = 20
pm.min_spare_servers = 10
pm.max_spare_servers = 30
pm.max_requests = 1000

php_admin_value[error_log] = /var/log/php/fpm-php.$pool.log
```

For a full reference see https://github.com/php/php-src/blob/master/sapi/fpm/www.conf.in

### Further configuration

If you need different ini files or pool configurations for different hosts or environments you could either use Jinja2 variables in those files or adjust your file structure under `{{ playbook_dir }}/files/php` (i.e. add a layer for the host or environment) so that you have multiple sets of these files.

Using variables is only recommended if differences between hosts or environments are rather small, otherwise your files might get unreadable quickly and this is what this role tries to fix.

For larger differences, adjust the `php_cli_confd_files`, `php_fpm_confd_files` and `php_fpm_poold_files` search paths to match your needs for maintaining multiple sets of ini files and pool configurations for different hosts or environments. 
