

Monitoring over Icinga2
#######################

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


  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. TODO: Delete the note below before merging new content to the master branch.

.. note::

   **This technote is not yet published.**

   Puppet deployment of monitoring system based on a master-client design, in which all puppet clients will automatically enroll to the icinga master, also being reflect in the Web Interface through IcingaWeb2, handled by Icinga Director and deploying graphs through PNP

.. Add content here.
.. Do not include the document title (it's automatically added from metadata.yaml).

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
