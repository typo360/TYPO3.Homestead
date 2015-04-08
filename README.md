TYPO3 Homestead
===============

TYPO3 Homestead is your one-stop [TYPO3](http://typo3.org) development environment. Just run `vagrant up` and a full Linux Ubuntu distribution will be built with all the packages and configuration needed to start development right away.

This environment is inteded as as a local environment. Security-wise it is in no way fit for production.

*[You must have a Linux / OSX control machine to run ansible.](http://docs.ansible.com/intro_windows.html#reminder-you-must-have-a-linux-control-machine)*

Features
--------

TYPO3 Homestead comes with the following stack:

* composer
* hhvm
* mariadb
* memcached
* nginx
* nodejs
* php-apcu
* php-fpm
* postfix nullmailer (outgoing only)
* self signed ssl certificates
* TYPO3 CMS
* TYPO3 FLOW
* TYPO3 NEOS
* xdebug
* xhprof / blackfire
* zsh

The flexible configuration allows you to create any combination of TYPO3 source and PHP backend with or without SSL.

Requirements
------------

* [Virtualbox](https://www.virtualbox.org/) or another virtualization product - Free!
* [Vagrant](http://www.vagrantup.com/) - Free!
* [Ansible](http://docs.ansible.com/intro_installation.html) - Free! (only the Tower ui will cost you)

Dependencies
------------

TYPO3 Homestead uses several roles from the ansible-galaxy:

* [nbz4live.php-fpm](https://galaxy.ansible.com/list#/roles/304)
* [jdauphant.nginx](https://galaxy.ansible.com/list#/roles/466)
* [geerlingguy.composer](https://galaxy.ansible.com/list#/roles/429)
* [laggyluke.nodejs](https://galaxy.ansible.com/list#/roles/285)

These roles need to be installed before you can run the playbook.

You can use the requirements.yml you find in the root of your freshly cloned TYPO3.Homestead:

```bash
git clone https://github.com/Tuurlijk/TYPO3.Homestead.git
cd TYPO3.Homestead
ansible-galaxy install -r requirements.yml
```

Ansible versions lower than 1.8 do not support yaml markup in the requirements file. If you have an old version of ansible, you can run:

```bash
git clone https://github.com/Tuurlijk/TYPO3.Homestead.git
cd TYPO3.Homestead
ansible-galaxy install -r requirements_pre-1.8.txt
# After that you will need to move the roles:
cd roles
mv nbz4live.php-fpm php-fpm
mv jdauphant.nginx nginx
mv tersmitten.composer composer
mv laggyluke.nodejs nodejs
mv f500.memcached memcached
```

Installation
------------

Installation is pretty straight forward. You're just a few steps away from 'great success'.

1). Clone TYPO3.Homestead and cd into it:
```bash
git clone https://github.com/Tuurlijk/TYPO3.Homestead.git
cd TYPO3.Homestead
```

2). Install the required ansible-galaxy roles:
```bash
ansible-galaxy install -r requirements.yml
```

3). Copy any configuration files you wish to change from `Defaults/` to `Configuration/`. Optionally setup a shared directory to hold your TYPO3 sources and sites in the `Configuration/vagrant.yml` file:

```yaml
synced_folders:
  - name: Development
    src: ~/Projects/TYPO3/Development
    target: /var/www
```

4). Then boot the machine:
```bash
vagrant up
```

The first time it boots it will take some time to install all needed packages and sources. So now is a good time to get a nice drink for yourself.

If you wish to look back at what happened during provisioning, you should have logged the output:
```bash
vagrant up 2>&1 | tee vagrant-up.log.txt
```

When the installation process has finished, you can visit [http://homestead.local.typo3.org](http://homestead.local.typo3.org). And also any of the pre-configured sites or any site you configured. The default sites are:

* [4.5.cms.local.typo3.org/typo3/install/](http://4.5.cms.local.typo3.org/typo3/install/)
* [4.5.40.cms.local.typo3.org/typo3/install/](http://4.5.40.cms.local.typo3.org/typo3/install/)
* [6.2.cms.local.typo3.org/typo3/](http://6.2.cms.local.typo3.org/typo3/)
* [6.2.11.cms.local.typo3.org/typo3/](http://6.2.11.cms.local.typo3.org/typo3/)
* [7.1.cms.local.typo3.org/typo3/](http://7.1.cms.local.typo3.org/typo3/)
* [dev-master.cms.local.typo3.org/typo3/](http://dev-master.cms.local.typo3.org/typo3/)
* [1.2.neos.local.typo3.org/setup/](http://1.2.neos.local.typo3.org/setup/)
* [dev-master.neos.local.typo3.org/setup/](http://dev-master.neos.local.typo3.org/setup/)

All CMS installations (except 4.5.*) are fully set-up. You can login to the backend with: `admin`/`supersecret`.

The error log can be inspected using: `multitail /var/log/nginx/error.log`.

The database credentials can be found in `roles/mariadb/vars/main.yml`. The typo3 user has access to all databases. The install tool password is the TYPO3 default.

The amount of cpu's avaialble on the host machine will also be available on the guest machine. 25% of the available host machine memory will be made available on the guest machine. The minimum amount of memory will be envforced to 1024 MB. You should not have to pass any extra parameters when starting the box.

The CNAME *.local.typo3.org resolves to the IP 192.168.144.120. This means you will have magic auto-resolving hostnames. So if you change the IP, you will need to take care of your hostname resolving yourself, either by hardcoding all the hostnames you wish to use or by some other means.

Variables
---------

You can override any of the role variables in the configuration files in the `/Configuration/` directory. The options have been tuned for usage with TYPO3, so the ones you will most likely be changing are the `typo3.yml` and the `websites.yml` files. In the `typo3.yml` file you can configure your typo3.org username and what TYPO3 versions you wish to have available.

The site *key* will be the domain name under which you can access the site. The site *value* is the TYPO3 version that will be checked out for that site. For TYPO3 4.x sites, you can specify a git branch or tag to checkout. For TYPO3 versions from 6.2.6 and up, we use [TYPO3 console](https://github.com/helhum/typo3_console) to install TYPO3. This means you can use [composer](https://getcomposer.org/) notation to specify a TYPO3 tag.

```yaml
typo3:
  cms:
    sites:
      4.5.cms.local.typo3.org: 'TYPO3_4-5'
      4.5.40.cms.local.typo3.org: 'TYPO3_4-5-40'
      6.2.cms.local.typo3.org: '6.2.*'
      6.2.11.cms.local.typo3.org: '6.2.11'
      7.1.0.cms.local.typo3.org: '7.1.0'
      dev-master.cms.local.typo3.org: '*'
```

TYPO3 Homestead uses *wilcard* server names in the Nxinx configuration. So you don't have to manually add any site configuration (except if you want to test ssl). The request url will determine the document root. The regular site requests will use the default php-fpm backend. The other backends currently available are: *hhvm* and *xhprof*. Xhprof will need further pool configuration though.

If you prefix your site name with 'hhvm' for example, your request will be served by the hhvm backend:

* [http://php.6.2.11.cms.local.typo3.org/typo3/](http://php.6.2.11.cms.local.typo3.org/typo3/)
* [http://hhvm.6.2.11.cms.local.typo3.org/typo3/](http://hhvm.6.2.11.cms.local.typo3.org/typo3/)

You can see what backend is used by inspecting the `X-TYPO3-Homestead-backend` response header.

You can now 'launch' a new test site just by placing a new folder in your Development directory. If you link to the TYPO3 source yourself, you don't even need a mapping configuration in the typo3.yml file. Your site will be instantly available (if the folder name matches one of the wildcard regexes).

If you change any typo3 configuration (Add new tags / branches) after you have provisioned your server, you will need to re-provision using:

```bash
ANSIBLE_ARGS='--tags=typo3,nginx' vagrant provision
```

Please note that typo3 needs to run first because the ssl certificates and web directories need to be available for nginx to pass the configuration check. If you change only nginx configuration after you have provisioned your server, you can re-provision using:

```bash
ANSIBLE_ARGS='--tags=nginx' vagrant provision
```

Need to enable an install tool? The following command will create the **ENABLE_INSTALL_TOOL** files in all TYPO3 CMS instances:

```bash
ANSIBLE_ARGS='--tags=typo3-cms-installtool' vagrant provision
```

The NEOS configuration is WIP. The TYPO3 CMS configuration sets up an empty database and an empty site. So you will need to run through the install tool and set things up.

Configuration Examples
----------------------

You can choose between different PHP upstream backends:
* php: php-fpm
* xhprof: php-fpm-xhprof (not enabled yet: wip)
* hhvm: hhvm

```yaml
nginx_sites:
  default:
    - set $upstream hhvm
    - listen 80 default_server
    - server_name _
    - root "{{ typo3_webroot }}/homestead.local.typo3.org/"
    - "{{ nginx_fastcgi }}"
  default-ssl:
    - set $upstream php
    - listen 443 default_server
    - server_name _
    - root "{{ typo3_webroot }}/homestead.local.typo3.org/"
    - "{{ nginx_fastcgi }}"
    - ssl on
    - ssl_certificate /etc/ssl/certs/homestead.local.typo3.org.crt
    - ssl_certificate_key /etc/ssl/private/homestead.local.typo3.org.key
  typo3.cms:
    - set $upstream php
    - server_name ~(?<serverNameUpstream>php|xhprof|blackfire|hhvm)?\.?(?<version>.*)\.cms.local.typo3.org$
    - if ($serverNameUpstream ~ (php|xhprof|blackfire|hhvm)) { set $upstream $serverNameUpstream; }
    - root "{{ typo3_webroot }}${version}.cms.local.typo3.org/";
    - "{{ nginx_fastcgi }}"
```

Please take care to add a domain-name to source mapping in the typo3.yml for each site you configure. Unless you manually set up your sites in your shared folder and link to the available sources yourself.

The `typo3_ssl_certificates` variable is an array of domain names for which self signed ssl certificates will be generated. A wildcard certificates will be generated for *.local.typo3.org.

Contributing
------------

In lieu of a formal styleguide, take care to maintain the existing coding style. Add unit tests and examples for any new or changed functionality.

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

TODO
----

* Complete the preconfiguration of TYPO3 NEOS instances
* Nginx configuration snippets?
  https://github.com/h5bp/server-configs-nginx/blob/master/h5bp/
* Make PHP configuration so flexible it can also handle any recent php version (https://github.com/phpbrew/phpbrew)
* Enable configuration through yml file like http://laravel.com/docs/5.0/homestead

License
-------

[GNU General Public License version 3](https://www.gnu.org/licenses/gpl-3.0.html)

References
----------

- [konomae/ansible-laravel-settler](https://github.com/konomae/ansible-laravel-settler)
- [laravel/homestead](https://github.com/laravel/homestead)
- [laravel/settler](https://github.com/laravel/settler)
