Dataverse puppet module
=======================

Table of Contents
-----------------

1. [Overview - What is the Dataverse module?](#overview)
2. [License](#license)
3. [Version numbering](#version-numbering)
4. [Module Description - What does the module do?](#module-description)
5. [Dataverse releases](#dataverse-releases)
6. [Setup - The basics of getting started with Dataverse module](#setup)
7. [Before you begin - Pre setup conditions](#before-you-begin)
8. [Configuring the infrastructure - Installing dataverse](#configuring-your-infrastructure)
9. [Public Classes and Defined Types](#public-classes-and-defined-types)
10. [Private Classes and Defined Types](#private-classes-and-defined-types)
11. [Examples](#examples)
12. [Known issues](#known-issues)
13. [To do](#todo)
14. [Contributors](#contributors)

Overview
--------

The Dataverse module allows you to install Dataverse with Puppet.

License
-------

GPLv3 - Copyright (C) 2015 International Institute of Social History <socialhistory.org>.

Version numbering
------------------

This module's branch versions and tags reflect the API's status as meant by [Semantic Versioning](http://semver.org/).
The API contract is specified in the [Classes and Defined Types](#classes-and-defined-types) section.

Module Description
-------------------

Dataverse is an [open sourced](https://github.com/IQSS/dataverse) web application to share, preserve, cite, explore and
analyze research data. It facilitates making data available to others, and allows you to replicate others work.
Researchers, data authors, publishers, data distributors, and affiliated institutions all receive appropriate credit via
a data citation with a persistent identifier (e.g., DOI, or Handle).The module offers support for basic management of
common security settings.

This module will install dataverse with default settings and allow customisation of those settings.

Dataverse releases
------------------

It is likely that Dataverse will continue releasing new packages with new installation features.
That does not mean the version of this module is not compatible with such releases. It is just
that you may have to manually apply [configurations or patches](https://github.com/IQSS/dataverse/releases) that this
module does not know about at the time of this release.
     
Setup
-----

**What this Dataverse setup affects:**

* Packages/service/configuration files for Dataverse, R and R packages, PostgreSQL, Solr and TwoRavens.

**What this Dataverse setup does not affect:**

* Optimizations such as the best settings for Glassfish and PostgreSQL.
* Shibboleth configurations (as they have an experimental status), although the packages are installed.

Before you begin
----------------

* Apply a first-time update of each of the host's package repository using 'apt-get update' or 'yum update'.
* Maybe take a look at the [examples](example) if you are unfamiliar with puppet or need some suggestions.

Configuring your infrastructure
-------------------------------

To install Dataverse and TwoRavens with all the out-of-the-box settings use:

    class {
      'iqss::globals':   # The global settings
    }->class {
      'iqss::rserve':    # RServe
    }->class {
      'iqss::database':  # Our PostgreSQL database
    }->class {
      'iqss::solr':      # Apache Solr 4
    }->class {
      'iqss::dataverse': # Dataverse ( Glassfish and war )
    }
    
    class {
      'iqss::tworavens': # TwoRavens add-on
        require => Class['iqss::globals'];
    }
    
To install on different machines you can deploy per server per component. E.g.:

    database-0.domain.org   class { 'iqss::globals':}->class { 'iqss::database': }
    dataverse-1.domain.org  class { 'iqss::globals':}->class { 'iqss::dataverse': }
    dataverse-2.domain.org  class { 'iqss::globals':}->class { 'iqss::dataverse': }
    dataverse-3.domain.org  class { 'iqss::globals':}->class { 'iqss::dataverse': }
    rserve-0.domain.org     class { 'iqss::globals':}->class { 'iqss::rserve': }
    solr-0.domain.org       class { 'iqss::globals':}->class { 'iqss::solr': }
    tworavens-0.domain.org  class { 'iqss::globals':}->class { 'iqss::tworavens': }
    
###Public Classes and Defined Types

Public classes can be set when declaring them in a manifest and with (#hieradata).

This module modifies configuration files and directories with the following public classes:

* [Class: Iqss::Database](#class-iqssdatabase)
* [Class: Iqss::Dataverse](#class-iqssdataverse)
* [Class: Iqss::Globals](#class-iqssglobals)
* [Class: Iqss::Rserve](#class-iqssrserve)
* [Class: Iqss::Solr](#class-iqsssolr)
* [Class: Iqss::TwoRavens](#class-iqsstworavens)

####Class: Iqss::Database

Installs Postgresql, the database user and database. For example:

    class {
      'iqss::database':
        name     => 'dataverse',
        user     => 'dataverse',
        password => 'secret',
    }
    
Use the Iqss::Globals class to override settings. This will create a running Postgresql server with the
database, users and access policies.

It also contains settings for
   
#####`createdb`
   
The user can create database. Defaults to 'false'.

#####`createrole`

The user can create roles. Defaults to 'false'.
 
#####`encoding`

This will set the default encoding encoding for all databases created with this module. On certain operating systems
this will be used during the `template1` initialization as well so it becomes a default outside of the module too.
Defaults to 'UTF-8'.

#####`hba_rule`
 
The access rules that determine who can connect to what database from where. Defaults to:

    IPv4 local connections => {
        description => 'Open up a IP4 connection from localhost',
        type        => 'host',
        database    => 'dvndb',
        user        => 'dvnApp',
        address     => '127.0.0.1/32',
        auth_method => 'md5'
        },
    IPv6 local connections => {
        description => 'Open up a IP6 connection from localhost',
        type        => 'host',
        database    => 'dvndb',
        user        => 'dvnApp',
        address     => '::1/128',
        auth_method => 'md5'
        }

#####`listen_addresses`

This value defaults to `*`, meaning the postgres server will accept connections accept connections from any remote
machine. Alternately, you can specify a comma-separated list of hostnames or IP addresses. (For more info, have a look
at the `postgresql.conf` file from your system's postgres package).

#####`locale`

This will set the default database locale for all databases created with this module. Defaults to 'en_US.UTF-8'.

#####`login`

The fact the user can login or not. Defaults to 'true'.

#####`manage_package_repo`

Setup the official PostgreSQL repositories on your host. Defaults to `true`.

#####`replication`

This role can replicate. Defaults to 'false'.

#####`superuser`

This role is a superuser. Defaults to 'false'.

#####`version`

The version of PostgreSQL. Defaults to '9.3'. 

####Class: Iqss::Dataverse

This class installs Glassfish, the domain settings and depending on the configuration builds a war or pulls a war
distribution from a repository. Example:

    class {
        'iqss::dataverse':
            package => 'dataverse-4.2',
            repository => 'https://github.com/IQSS/dataverse/releases/download/v4.2/dataverse-4.2.war', 
    }
    
This will create three services:

* The Dataverse 4.2 distribution plus glassfish service: $ service glassfish start|stop|status
* An R-daemon: $ service rserve start|stop|status
* The Apache web server

It also contains settings for

#####`auth_password_reset_timeout_in_minutes`

The time in minutes for a password reset. Defaults to '60'.

#####`doi_baseurlstring`

The DOI endpoint for the EZID Service. Defaults to 'https://ezid.cdlib.org'.
 
#####`doi_username`

The username to connect to the EZID Service. Defaults to 'apitest'. 

#####`doi_password`

The password to connect to the EZID Service. Defaults to 'apitest'. 

#####`files_directory`

The location of the uploaded files and their tabular derivatives. Defaults to '/home/glassfish/dataverse/files'.

#####`fqdn`

If the Dataverse server has multiple DNS names, this option specifies the one to be used as the "official" host name.
For example, you may want to have dataverse.foobar.edu, and not the less appealing server-123.socsci.foobar.edu to
appear exclusively in all the registered global identifiers, Data Deposit API records, etc. Defaults to 'localhost'.

Do note that whenever the system needs to form a service URL, by default, it will be formed with https:// and port 443.
I.e.,
https://{dataverse.fqdn}/

If that does not suit your setup, use the `Iqss::Dataverse::site_url` option.

#####`glassfish_parent_dir`

The Glassfish parent directory. Defaults to '/home/glassfish'.

#####`glassfish_domain_name`

The domain name. Defaults to 'domain1'.

#####`glassfish_fromaddress`

The e-mail -from field in the mail header. Defaults to 'do-not-reply@localhost'.

#####`glassfish_jvmoption`

An array of jvm options. 
Defaults to ["-Xmx1024m", "-Djavax.xml.parsers.SAXParserFactory=com.sun.org.apache.xerces.internal.jaxp.SAXParserFactoryImpl"].

#####`glassfish_mailhost`

The mail relay hostname. Defaults to 'localhost'.

#####`glassfish_mailuser`

The user name that is allowed by the mail relay to sent mails. Defaults to 'dataversenotify'.

#####`glassfish_mailproperties`

Key-value pairs sent with to the mail relay, such as credentials.
Defaults to dummy values 'username=a_username:password=a_password'.

#####`glassfish_service_name`

The service handle to submit start, stop, status commands. E.g. service dataverse start. Defaults to 'glassfish'

#####`glassfish_tmp_dir`

The download path of the glassfish package. Defaults to '/opt/glassfish'.

#####`glassfish_user`

The user running the glassfish domain. Defaults to 'glassfish'.

#####`glassfish_version`

The Glassfish J2EE Application server version. Defaults to '4.1'.

#####`package`

The release tag: name and version of dataverse 4. Defaults to 'dataverse-4.2.1'.

#####`port`

The SSL port on which dataverse can be reached. Defaults to `Globals:dataverse_port`.

#####`repository`

This indicates there the package comes from. It can be 'git' to build a war from the IQSS repository; or the repository
url of a Dataverse war file.
Defaults to 'https://github.com/IQSS/dataverse/releases/download/v4.2.1/dataverse-4.2.1.war'.

#####`rserve_host`

The Rserve service endpoint. Defaults to 'localhost'.

#####`rserve_password`

The password needed to access the Rserve service. Defaults to 'rserve'.

#####`rserve_port`

The Rserve service port. Defaults to 6311.

#####`rserve_user`

The username that can access the Rserce service. Defaults to 'serve'. 
 
#####`site_url`

The url to a dataverse web application. Defaults to 'https://localhost:443'.

####Class: Iqss::Rserve

Installs the Rserve or Binary R server daemon. Example:
                            
    class { 'iqss::rserve':
        pwd  => 'potato',
    }
    
This will install R and a number of R packages including the RServe package. A password file /etc/Rserve.pwd is set with
content: rserve potato. Rserve is run by the 'rserve' user.

#####`auth`

If you need remote access use set auth 'required' and plaintext to 'disable'. Defaults to 'disable'.

#####`chroot`

The jail directory. Defaults tot undef.

#####`encoding`

This means that strings are converted to the given encoding before being sent to the client and also all strings from
the client are assumed to come from the given encoding. Defaults to 'utf8'.

#####`eval`

Preload packages with expressions that you would otherwise have to load from scripts. Defaults to undef. 

#####`fileio`

Allow clients to perform filesystem operations via the RServe deamon. Defaults to 'enable'.

#####`gid`

The group id of the 'rserve' user runnning the daemon. Defaults to 97.

#####`interactive`

Undocumented. Defaults to 'yes'.

#####`maxinbuf`

The maximum allowed buffer size send from the client per connection. Defaults to 262144 Kb.

#####`maxsendbuf`

The maximum allowed buffer size send from the server per connection. Defaults to 0 Kb.

#####`plaintext`

Allows for sending credentials as plaintext. Defaults to 'disable'.

#####`password`

The password for connecting to the Binary R server daemon. Defaults to 'rserve'.

#####`port`

The TCP port the daemon listens too. Defaults to '6311'

#####`pwdfile`

The password file containing the authentication credentials. Defaults to '/etc/Rserve.pwd'.

#####`remote`

Allows remote connections when enabled. Defaults to 'enable'.

#####`socket`

Undocumented. Defaults to undef.

#####`sockmod`

Undocumented. Defaults to undef.

#####`source`

Location to a file to preload packages that you would otherwise have to load from your scripts. Defaults to undef.

#####`su`

Undocumented. Defaults to undef.

#####`uid`

The user id of the 'rserve' user running the daemon. Defaults to 97.

#####`umask`

Controls how file permissions are set for files. Defaults to 0.

#####`workdir`
 
R working directory.Defaults to '/tmp/Rserv'

####Class: Iqss::Solr

Installs Solr. Example:

    class { 'iqss::solr':
        parent_dir => '/usr/share',
    }
    
This will create a Jetty server with a running Solr instance : $ service solr stop|status|start

It also contains settings for

#####`core`

The handle of the Solr core. Defaults to 'collection1'.

#####`jetty_home`

The Jetty home directory which contains start.jar. Defaults to '/home/solr/solr-4.6.0/example'

#####`jetty_host`

The IP to listen to. Use 0.0.0.0 as host to listen on all IP connections. Defaults to 'localhost'.

#####`jetty_java_options`

JVM options for Jetty. 

#####`jetty_port`

The port Jetty will listen to. Defaults to '8983'.

#####`jetty_user`

The user running the Jetty Solr instance. Defaults to 'solr'.

#####`solr_home`

The Solr home used for the jvm setting -Dsolr.solr.home. Defaults to '/home/${jetty_user}/solr-4.6.0/example/solr'.

#####`url`

The download url for solr. Preferably a mirror. Defaults to 'http://archive.apache.org/dist/lucene/solr'.

#####`version`

The Apache Solr version. Defaults to '4.6.0'.

####Class: Iqss::Tworavens

This class installs the Apache RApache handler and the Tworavens web application. For example:

    class {
      'iqss::tworavens':
        tworavens_package => 'https://github.com/IQSS/TwoRavens/archive/master.zip',
    }
    
It also contains settings for
    
#####`dataverse_fqdn`
The public domain name of the Dataverse web application. Defaults to 'localhost'.
    
#####`dataverse_port`

The public port of the Dataverse web application. Defaults to '443'.  

#####`fqdn`

The public domain name of the TwoRavens web application. Defaults to 'localhost'.

#####`package`

The download url of TwoRavens. Defaults to 'https://github.com/IQSS/TwoRavens/archive/master.zip'.

#####`parent_dir`

The installation directory of the TwoRavens web application. Defaults to '/var/www/html'. 

#####`port`

The public port of the TwoRavens web application. Defaults to '443'.

#####`protocol`

The protocol of the TwoRavens web application. Defaults to 'https'.

#####`rapache_version`

The rapache version to be installed. Defaults to '1.2.6'.

###Private Classes and Defined Types

Private classes should not be called directly from the module or manifests.
But their parameters can be altered with (#hieradata).

#### Class: Iqss::Apache2

Installs apache.

#####`purge_configs`

Removes all other Apache configs and vhosts. Setting this to 'false' is a stopgap measure to allow the apache module
to coexist with existing or otherwise-managed configuration. Defaults to true.

#### Class: Iqss::Rpackager 

Installs R, required libraries and then a range of packages used by dataverse (RServe) and TwoRavens (Rook, Zelig, and
others).

#####`repo`

The repository to download the packages from. Defaults to 'http://cran.r-project.org'.

#####`packages`

A list of R packages to install.

Known issues
------------

* Rserve does not start after a puppet run ( it does on boot ). You need to start the service manually with
`service rserve restart`
* Vagrant installations could have various issues per OS and image. If the machine does not play along, then try out the
know issues mentioned the [Vagrantfile](Vagrantfile).
* If you run Vagrant and install TwoRavens with a localhost domain, the application that runs on the client host will
not be able to reach port 443 as it will use localhost:9999 to call the dataverse API.

You may get around this last issue by setting a firewall rule in your client machine:

    $ iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 9999
    $ iptables-save
    
Or create an additional VirtualHost definition for port 9999 to the Apache configuration:

    Listen 9999
    <VirtualHost *:9999>
    etc... just copy the <VirtualHost *:443> settings 
    </VirtualHost>

To do
-----

* Shibboleth
* The [master-agent example](example/master-agent.md)

Examples
--------

See the [demo installations](example) of this module.

Contributors
------------

* Lucien van Wouw, IISH
* Plenty room for more. Please read the [contributing guide](CONTRIBUTING.md) on how to donate.
