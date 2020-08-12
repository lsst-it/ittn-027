
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
specifying the values.

.. code-block:: puppet

   class { '::icinga2':
      manage_repo => true,
      confd       => false,
      constants   => {
         'ZoneName'   => 'master',
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
to none, you will need to provide a certificate and a CA that validates it; if set to icinga, it sets the icinga master as
a CA and all agents will request a certificate to it. Then, there's the option that has been used here, which is use puppet
master as a CA and use the already generated certificates by the puppet server.

.. code-block:: puppet

   class { '::icinga2::feature::api':
      pki             => 'puppet',
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

Enables the proccesing of the data in order to generate graphs and data management.

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


.. sectnum::

.. TODO: Delete the note below before merging new content to the master branch.

