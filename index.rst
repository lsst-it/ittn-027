
Introduction
============

Puppet deployment consisting on monitoring system based on a master-client design, 
in which all puppet clients will automatically enroll to the icinga master, also 
being reflect in the Web Interface through IcingaWeb2, handled by Icinga Director 
and deploying graphs through PNP.

The taken approach allow us to have multiple master instances, and that every node
response to their own master. For future developments, the idea is to join the 
masters into a replicate configuration to visualize all instances.

Requirements
============

In order to deploy Icinga2 and IcingaWeb2, there are several dependancies that have
to be taken care off. The built code includes those references and is in a "ready to
deploy" status

+-------------------------+---------------------------------------------------------+----------+
| Package                 |    Repo                                                 | Version  |
+=========================+=========================================================+==========+
|Remi                     | https://github.com/hfm/puppet-remi                      | v1.11.0  |
+-------------------------+---------------------------------------------------------+----------+
|Icinga2                  | https://github.com/Icinga/puppet-icinga2                | v2.3.2   |
+-------------------------+---------------------------------------------------------+----------+
|IcingaWeb2               | https://github.com/Icinga/puppet-icingaweb2             | v2.3.1   |
+-------------------------+---------------------------------------------------------+----------+
|Nginx                    | https://github.com/voxpupuli/puppet-nginx               | v1.1.0   |
+-------------------------+---------------------------------------------------------+----------+
|MySql Server             | https://github.com/puppetlabs/puppetlabs-mysql          | v10.4.0  |
+-------------------------+---------------------------------------------------------+----------+
|IcingaWeb Director       | (part of icinga repo)                                   | v1.7.2   |
+-------------------------+---------------------------------------------------------+----------+
|IcingaWeb PNP            | https://github.com/Icinga/icingaweb2-module-pnp.git     | v1.1.0   |
+-------------------------+---------------------------------------------------------+----------+
|IcingaWeb Reactbundle    | https://github.com/Icinga/icingaweb2-module-reactbundle | v0.7.0   |
+-------------------------+---------------------------------------------------------+----------+
|IcingaWeb IPL            | https://github.com/Icinga/icingaweb2-module-ipl         | v0.3.0   |
+-------------------------+---------------------------------------------------------+----------+
|IcingaWeb Incubator      | https://github.com/Icinga/icingaweb2-module-incubator   | v0.5.0   |
+-------------------------+---------------------------------------------------------+----------+

Code References
===============

The configured code can be found in both the public and private repo:
 - Icinga Master
   https://github.com/lsst-it/lsst-itconf/blob/master/site/profile/manifests/core/icinga_master.pp
 - Icinga Agent
   https://github.com/lsst-it/lsst-itconf/blob/master/site/profile/manifests/core/icinga_agent.pp


Services, Hosts and Templates
=============================

There are 4 things (or objects) to take in consideration: Host templates, Service templates, Hosts and Services.
Host Templates are responsable of providing the final host object an inheritad default configuration, as
well as a default configuration.
Service templates are the ones that have the "check_command" on them, which means, is the one that defines
which operation is going to be perform in the remote host (without defining yet the host).
Services per se, links or vinculates a Host Template with a Service Template, so all hosts that use the
defined Host Template, will automatically inherit the previously linked Service Template.
Finally, when you define a Host, you simply use the defined Host Template, and the host will add itself
with the Service Template (check_command, monitoring operation that will be perform) into the icinga
master.

Host Templates
--------------

To create a Host Template, you must specify the following fields:
   - accept_config:         Allow the host to receive instructions from the icinga master.
   - check_command:         Default command to run to the hosts; preferably leave it in "hostalive".
   - has_agent:             Agent must have installed and be configured with icinga agent.
   - master_should_connect: Set to "yes".
   - max_check_attempts:    Number of attempts the master will try to check the host.
   - object_name:           The name to which you will later reffer to this object.
   - object_type:           Set to "template".

.. code-block:: json

   {
    "accept_config": true,
    "check_command": "hostalive",
    "has_agent": true,
    "master_should_connect": true,
    "max_check_attempts": "5",
    "object_name": "${host_template_name}",
    "object_type": "template"
   }

Service Templates
-----------------

To create a Service Template, you must specify the following field:
   - check_command: The monitoring command you which to report, such us dhcp, dns, ldap.
   - object_name:   The name to which you will later reffer to this object.
   - object_type:   Set to "template".
   - use_agent:     Allow connection to the icinga agent installation in the host.
   - vars:          Define additional variables, such as warning land critical level, ldap parameters, ping response time.
   - zone:          Zone in which the icinga master is configured.

.. code-block:: json

   {
    "check_command": "ldap",
    "object_name": "${service_template_name}",
    "object_type": "template",
    "use_agent": true,
    "vars": {
      "ldap_address": "localhost",
      "ldap_base": "dc=lsst,dc=cloud"
    },
    "zone": "master"
   }

Services
--------

In order to create a Service, you must use the exact same object names that were used in the Host Template and the Service Template:
   - host:        For a Service object, you must provide the host_template_name (same as the one used before).
   - imports:     For a Service object, you must provide the service_template_name (same as the one used before).
   - object_name: The name to which you will later reffer to this object.
   - object_type: Set to "object".

.. code-block:: json

   {
    "host": "${host_template}",
    "imports": [
    "${$service_template}"
    ],
    "object_name": "${service_name}",
    "object_type": "object"
   }

Hosts
-----

Finally to add the Host:
   - address:      The IP address of the host.
   - display_name: For consistency, use the Fully Qualified Domain Name.
   - imports:      For a Host object, you must provide the host_template_name that you wish this host is part of.
   - object_name": For consistency, use the Fully Qualified Domain Name. This name will be use for future reference.
   - object_type": Set to object.
   - vars:         Define additional variables that are specific to this host.

.. code-block:: json

   {
    "address": "${host_ip}",
    "display_name": "$host_fqdn",
    "imports": [
      "${host_template}"
    ],
    "object_name":"${host_fqdn}",
    "object_type": "object",
    "vars": {
      "safed_profile": "3"
    }
   }


Icinga2 Master modules and configuration
========================================

This comprise only the require information for the master's configuration. There are two ways of setting up a feature:
One, but adding it into the feature list (with the features default values), and the second declaring the class and 
specifying the values. The constant TicketSalt is required when using icinga master as a CA (which is the case).

.. code-block:: puppet

   class { '::icinga2':
      manage_repo => true,
      confd       => false,
      constants   => {
         'ZoneName'   => 'master',
         'TicketSalt' => $ca_salt,
      },
      features    => ['checker','mainlog','statusdata','compatlog','command'],
   }

The IdoMySQL features
---------------------

Uses a prexistent (allready created) MySQL database. In the class, you must add the name, pasword 
and database name you gave to the prexistent database.

.. code-block:: puppet

   class { '::icinga2::feature::idomysql':
      user          => $mysql_icingaweb_user,
      password      => $mysql_icingaweb_pwd,
      database      => $mysql_icingaweb_db,
      host          => $master_ip,
      import_schema => true,
      require       => Mysql::Db[$mysql_icingaweb_db],
   }

API feature
-----------

Is the method in which objects are read and published in between Icinga2 and IcingaWeb2. By setting the pki 
to none, you will need to provide a certificate and a CA that validates it; if set to icinga (default), it 
sets the icinga master as a CA and all agents will request a certificate to it. There is also the option to 
use 'puppet' as a parameter, but that will only work among the same puppetserver. When the master is set to
'none', but the class '::icinga2::pki::ca' is included, then the icinga master acts as a CA and generates and 
validates a certificate and key to itself.

.. code-block:: puppet

   class { '::icinga2::feature::api':
      pki             => 'none',
      accept_config   => true,
      accept_commands => true,
      ensure          => 'present',
      endpoints       => {
         $master_fqdn    => {
         'host'  =>  $master_ip
         },
      },
      zones           => {
         'master'    => {
         'endpoints' => [$master_fqdn],
         },
      },
   }
   include ::icinga2::pki::ca

Notification feature
--------------------

Enables notifications.

.. code-block:: puppet

   class { '::icinga2::feature::notification':
      ensure    => present,
      enable_ha => true,
   }

Perfdata feature
----------------

Enables the processing of the data in order to generate graphs and data management.

.. code-block:: puppet

  class {'::icinga2::feature::perfdata':
    ensure => present,
  }

Api User
--------

The API user is am object (that in difference of the templates, it is being generated through puppet code), creates the 
user and password in which the Icinga2 and IcingaWeb2 instances communicate to each other. Bear in mind that the same
user adn password must be later used for the IcingaWeb2 configuration.

.. code-block:: puppet

   icinga2::object::apiuser { $api_user:
      ensure      => present,
      password    => $api_pwd,
      permissions => [ '*' ],
      target      => '/etc/icinga2/features-enabled/api-users.conf',
   }

Icinga2 Agent modules and configuration
========================================

The configuration only requires to establish that the repository is going to be handled by the icinga2 module, that the 
conf.d folder is not going to be considered and the only require feature (with defaul values) is mainlog.

.. code-block:: puppet

   class { '::icinga2':
      manage_repo => true,
      confd       => false,
      features    => ['mainlog'],
   }


API feature
-----------

The icinga agents must be aware of who the icinga master is, so it's important to declare it at the endpoint and zones. 
The pki will not be set, which translate in the default (icinga2), then the agent is going to ask for a certificate to
the icinga master; also is needed to set the agent to acept configurations and commands from the master. 


.. code-block:: puppet

   class { '::icinga2::feature::api':
      ensure          => 'present',
      ca_host         => $icinga_master_ip,
      ticket_salt     => $ca_salt,
      accept_config   => true,
      accept_commands => true,
      endpoints       => {
         $icinga_agent_fqdn  => {
         'host'  =>  $icinga_agent_ip
         },
         $icinga_master_fqdn => {
         'host'  =>  $icinga_master_ip,
         },
      },
      zones           => {
         $icinga_agent_fqdn => {
         'endpoints' => [$icinga_agent_fqdn],
         'parent'    => 'master',
         },
         'master'           => {
         'endpoints' => [$icinga_master_fqdn],
         },
      }
   }

IcingaWeb2 modules and configuration
========================================

This installation is only carried in the icinga master. As the configuration per se is quite reduced: The repo is being 
handled by the icinga2 module, hence the need of requiring that the icinga2 class gets deployed first. Also, setting
the log level to Info provides a better troubleshooting.

.. code-block:: puppet

   class {'::icingaweb2':
      manage_repo   => false,
      logging_level => 'INFO',
      require       => Class['::icinga2'],
   }

Monitoring Module
-----------------

This module is the one responsable of communicating with the icinga2 service over the API user/password, then this two MUST
be the same one as provided during the icinga agent module configuration.
The ido parameters are the ones that connects with the MySQL database; this ones, as well as with the API credentials, must
be the same as the one provided in the icinga module configuration.

.. code-block:: puppet

   class {'icingaweb2::module::monitoring':
      ensure            => present,
      ido_host          => $master_ip,
      ido_type          => 'mysql',
      ido_db_name       => $mysql_icingaweb_db,
      ido_db_username   => $mysql_icingaweb_user,
      ido_db_password   => $mysql_icingaweb_pwd,
      commandtransports => {
         $api_name => {
         transport => 'api',
         host      => $master_ip,
         port      => 5665,
         username  => $api_user,
         password  => $api_pwd,
         }
      }
   }

LDAP Configuration
------------------

There are four modules responsable of deploying, without further configuration, the LDAP service in the IcingaWeb platform.

Resources
^^^^^^^^^

The resource name will be associated with the set ldap_resource value. Is important, that later on, you will need that 
value to reference this particular resource.

.. code-block:: puppet
  
   icingaweb2::config::resource{ $ldap_resource:
      type         => 'ldap',
      host         => $ldap_server,
      port         => 389,
      ldap_root_dn => $ldap_root,
      ldap_bind_dn => $ldap_user,
      ldap_bind_pw => $ldap_pwd,
   }

Authmethod
^^^^^^^^^^

There are two ways to authenticate to the IcingaWeb WUI: ldap or mysql. Since we already declared the resource, the previously 
declared resource name must be used at the 'resource' value. Also worth mentioning that the backend value has to be set to ldap.

.. code-block:: puppet

   icingaweb2::config::authmethod { 'ldap-auth':
      backend                  => 'ldap',
      resource                 => $ldap_resource,
      ldap_user_class          => 'inetOrgPerson',
      ldap_filter              => $ldap_user_filter,
      ldap_user_name_attribute => 'uid',
      order                    => '05',
   }

Groupbackend
^^^^^^^^^^^^

Using the ldap resource, now the IcingaWeb reads and filters the groups from the remote LDAP server.

.. code-block:: puppet

   icingaweb2::config::groupbackend { 'ldap-groups':
      backend                   => 'ldap',
      resource                  => $ldap_resource,
      ldap_group_class          => 'groupOfNames',
      ldap_group_name_attribute => 'cn',
      ldap_group_filter         => $ldap_group_filter,
      ldap_base_dn              => $ldap_group_base,
   }

Roles
^^^^^

Finally there is the roles. A role defines one of the read groups and grants them priviledges, which can go from read only,
modify readdings, delete agents, force a 'check now' on a host/service, or even change configurations.

.. code-block:: puppet

   icingaweb2::config::role { 'Admin User':
      groups      => 'icinga-admins',
      permissions => '*',
   }

IcingaWeb Director
------------------

The Director uses it's own MySQL database and acts as an intermediary between Icinga2 and IcingaWeb2. This is the final piece
that puts toghether all the recopilated data into one place and displays it through the WUI. The API user and password MUST be
the same ones as provided during the Icinga2 and IcingaWeb2 IDO configuration

.. code-block:: puppet

   class {'icingaweb2::module::director':
      git_revision  => 'v1.7.2',
      db_host       => $master_ip,
      db_name       => $mysql_director_db,
      db_username   => $mysql_director_user,
      db_password   => $mysql_director_pwd,
      import_schema => true,
      kickstart     => true,
      endpoint      => $master_fqdn,
      api_host      => $master_ip,
      api_port      => 5665,
      api_username  => $api_user,
      api_password  => $api_pwd,
      require       => Mysql::Db[$mysql_director_db],
   }

The Director module requires three additional modules to properly work: Reactbundle, IPL and Incubator.

.. code-block:: puppet

   class {'icingaweb2::module::reactbundle':
      ensure         => present,
      git_repository => 'https://github.com/Icinga/icingaweb2-module-reactbundle',
      git_revision   => 'v0.7.0',
      require        => Class['::icingaweb2'],
   }
   class {'icingaweb2::module::ipl':
      ensure         => present,
      git_repository => 'https://github.com/Icinga/icingaweb2-module-ipl',
      git_revision   => 'v0.3.0',
      require        => Class['::icingaweb2'],
   }
   class {'icingaweb2::module::incubator':
      ensure         => present,
      git_repository => 'https://github.com/Icinga/icingaweb2-module-incubator',
      git_revision   => 'v0.5.0',
      require        => Class['::icingaweb2'],
   }

IcingaWeb PNP
-------------

Is a graphing add on to IcingaWeb, that along with npcd and npcdmod collects performance data files, that are later
processed by Icinga2 Perfdata. Then, the PNP module uses the processed data and displayed them into graphs.


.. code-block:: puppet

   vcsrepo { '/usr/share/icingaweb2/modules/pnp':
      ensure   => present,
      provider => git,
      source   => 'git://github.com/Icinga/icingaweb2-module-pnp.git',
      revision => 'v1.1.0',
      require  => Class['::icingaweb2'],
   }
   exec { 'icingacli module enable pnp':
      cwd     => '/var/tmp/',
      path    => ['/sbin', '/usr/sbin', '/bin'],
      onlyif  => ['test ! -d /etc/icingaweb2/enabledModules/pnp'],
      require => Class['icingaweb2::module::director'],
   }

Nginx
-----

Latest but not less important, is the Nginx module configuration. Nginx is a web server, that can also be used as a reverse proxy or 
loadbalancer. 

Nginx Server
^^^^^^^^^^^^

The core of the web service, is to declare the server. You must choose a name (to which later will be used to reference it). 

The components are explain as:
   - server_name:    This value is the one the server will respond to, so it's suggested to use the FQDN.
   - ssl:            Set to 'true' to enable ssl. If true, cert and key must be provided.
   - ssl_cert:       Absolute path to the SSL Certificate.
   - ssl_key:        Absolute path to the SSL Key.
   - ssl_redirect:   If 'true', all http requests will be redirect to https.
   - index_files:    Default index file that will be presented to the end user.
   - www_root:       Absolute path to the root folder of the service (most commonly /var/www/html).
   - include_files:  Absolute path to any other particular configuration file that must the considered.

In this case, letsencrypt was used to provied signed certificate, which will write the cert and key to the specify path that the 
module has set.

.. code-block:: puppet

   letsencrypt::certonly { $master_fqdn:
         plugin      => 'dns-route53',
         manage_cron => true,
   }

   nginx::resource::server { 'icingaweb2':
      server_name          => [$master_fqdn],
      ssl                  => true,
      ssl_cert             => "${le_root}/cert.pem",
      ssl_key              => "${le_root}/privkey.pem",
      ssl_redirect         => true,
      index_files          => ['index.php'],
      use_default_location => false,
      www_root             => '/usr/share/icingaweb2/public',
      include_files        => ['/etc/nginx/sites-available/pnp4nagios.conf'],
   }

Nginx Locations
^^^^^^^^^^^^^^^

IcingaWeb uses fastcgi, which is a protocol that allows interface for web server to interact with programs. In order for the locations
to work, the server name specified above must match the one of the locations. Is also important to set in every one of the the ssl and
ssl_only flags to 'true'.

.. code-block:: puppet

   
   nginx::resource::location { 'root':
      location    => '/',
      server      => 'icingaweb2',
      try_files   => ['$1', '$uri', '$uri/', '/index.php$is_args$args'],
      index_files => [],
      ssl         => true,
      ssl_only    => true,
   }
   nginx::resource::location { 'icingaweb':
      location       => '~ ^/icingaweb2(.+)?',
      location_alias => '/usr/share/icingaweb2/public',
      try_files      => ['$1', '$uri', '$uri/', '/icingaweb2/index.php$is_args$args'],
      index_files    => ['index.php'],
      server         => 'icingaweb2',
      ssl            => true,
      ssl_only       => true,
   }
   nginx::resource::location { 'icingaweb2_index':
      location      => '~ ^/index\.php(.*)$',
      server        => 'icingaweb2',
      ssl           => true,
      ssl_only      => true,
      index_files   => [],
      try_files     => ['$uri =404'],
      fastcgi       => '127.0.0.1:9000',
      fastcgi_index => 'index.php',
      fastcgi_param => {
         'ICINGAWEB_CONFIGDIR' => '/etc/icingaweb2',
         'REMOTE_USER'         => '$remote_user',
         'SCRIPT_FILENAME'     => '/usr/share/icingaweb2/public/index.php',
      },
   }

.. sectnum::

.. TODO: Delete the note below before merging new content to the master branch.

