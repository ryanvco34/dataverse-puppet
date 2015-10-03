Dataverse puppet module=======================Table of Contents-----------------1. [Overview - What is the Dataverse module?](#overview)2. [License](#license)3. [Version numbering](#version-numbering)3. [Module Description - What does the module do?](#module-description)4. [Dataverse versions](#dataverse-versions)5. [Setup - The basics of getting started with Dataverse module](#setup)6. [Before you begin - Pre setup conditions](#before-you-begin)7. [Configuring the infrastructure - Installing dataverse](#configuring-your-infrastructure)8. [Classes and Defined Types](#classes-and-defined-types)9. [Hieradata - Using hieradata](#hieradata)10. [Examples](#examples)11. [Known issues](#known-issues)12. [To do](#todo)13. [Contributors](#contributors)Overview--------The Dataverse module allows you to install Dataverse with Puppet.License-------GPLv3 - Copyright (C) 2015 International Institute of Social History <socialhistory.org>.Version numbering------------------This module's branch versions and tags reflect the API's status as meant by [Semantic Versioning](http://semver.org/).The API contract is specified in the [Classes and Defined Types](#classes-and-defined-types) section.Module Description-------------------Dataverse is an [open sourced](https://github.com/IQSS/dataverse) web application to share, preserve, cite, explore and analyzeresearch data. It facilitates making data available to others, and allows you to replicate others work. Researchers,data authors, publishers, data distributors, and affiliated institutions all receive appropriate credit via a datacitation with a persistent identifier (e.g., DOI, or Handle).The module offers support for basic management of commonsecurity settings.This module will install dataverse with default settings and allow customisation of those settings.Dataverse versions------------------It is likely that Dataverse will continue releasing new packages, so this module may become out of date.That does not mean the version of this module is not compatible with such releases. It is justthat you may have to apply [additional configurations or patches](https://github.com/IQSS/dataverse/releases) that this module does not know about at the time of itsrelease.     Setup-----**What this Dataverse setup affects:*** Packages/service/configuration files for Dataverse, R and R packages, PostgreSQL, Solr and TwoRavens.**What this Dataverse setup does not affect:*** Optimizations such as the best settings for Glassfish and PostgreSQL.* Shibboleth configurations (as they have an experimental status), although the packages are installed.Before you begin----------------* Apply a first-time update of each of the host's package repository ( e.g. using 'apt-get update' or 'yum update'.* Maybe take a look at the [examples](example) if you are unfamiliar with puppet or need some suggestions.Configuring your infrastructure-------------------------------To install Dataverse and TwoRavens with all the out-of-the-box settings use:    class {      'iqss::globals':   # The global settings    }->class {      'iqss::rserve':    # RServe    }->class {      'iqss::database':  # Our PostgreSQL database    }->class {      'iqss::solr':      # Apache Solr 4    }->class {      'iqss::dataverse': # Dataverse ( Glassfish and war )    }        class {      'iqss::tworavens': # TwoRavens add-on        require => Class['iqss::globals'];    }    To install on different machines you can deploy per server per component. E.g.:    Server A-1: class { 'iqss::globals':}->class { 'iqss::database': }    Server B-1: class { 'iqss::globals':}->class { 'iqss::dataverse': }    Server B-2: class { 'iqss::globals':}->class { 'iqss::dataverse': }    Server B-3: class { 'iqss::globals':}->class { 'iqss::dataverse': }    Server C-1: class { 'iqss::globals':}->class { 'iqss::rserve': }    Server D-1: class { 'iqss::globals':}->class { 'iqss::solr': }    Server E-1: class { 'iqss::globals':}->class { 'iqss::tworavens': }    ###Classes and Defined TypesThis module modifies configuration files and directories with the following classes:* [Class: Iqss::Database](#class-iqssdatabase)* [Class: Iqss::Dataverse](#class-iqssdataverse)* [Class: Iqss::Globals](#class-iqssglobals)* [Class: Iqss::Rserve](#class-iqssrserve)* [Class: Iqss::Solr](#class-iqsssolr)* [Class: Iqss::TwoRavens](#class-iqsstworavens)####Class: Iqss::DatabaseInstalls Postgresql, the database user and database. For example:    class {      'iqss::database':        name     => 'dataverse',        user     => 'dataverse',        password => 'secret',    }    Use the Iqss::Globals class to override settings. This will create a running Postgresql server with thedatabase, users and access policies.It also contains settings for   #####`createdb`   When 'true' the user can create databases. Defaults to 'false'.#####`createrole`When 'true' the user can create roles. Defaults to 'false'. #####`hba_rule` The access rules that determine who can connect to what database from where. Defaults to:    IPv4 local connections => {        description => 'Open up a IP4 connection from localhost',        type        => 'host',        database    => 'dvndb',        user        => 'dvnApp',        address     => '127.0.0.1/32',        auth_method => 'md5'        },    IPv6 local connections => {        description => 'Open up a IP6 connection from localhost',        type        => 'host',        database    => 'dvndb',        user        => 'dvnApp',        address     => '::1/128',        auth_method => 'md5'        }        #####`host`The url connection string. Defaults to 'localhost'.#####`login`The fact the user can login or not. Defaults to 'true'.#####`name`Name of the database. Inherited by iqss::globals::database_name. Defaults to 'dvndb'.#####`password`The user password. Defaults to 'dvnAppPass'.#####`port`The connection port to the database. Defaults to '5432'#####`replication`When 'true' this role can replicate. Defaults to 'false'.#####`superuser`When 'true' this role is a superuser. Defaults to 'false'.#####`user`The user name. Defaults to 'dvnApp'.#####`version`The version of Postgresql. Defaults to '9.3'. #####`manage_package_repo`If `true` this will setup the official PostgreSQL repositories on your host. Defaults to `true`.#####`encoding`This will set the default encoding encoding for all databases created with this module. On certain operating systems this will be used during the `template1` initialization as well so it becomes a default outside of the module as well. Defaults to 'UTF-8'.#####`locale`This will set the default database locale for all databases created with this module. Defaults to 'en_US.UTF-8'.#####`listen_addresses`This value defaults to `localhost`, meaning the postgres server will only accept connections from localhost. If you'd like to be able to connect to postgres from remote machines, you can override this setting. A value of `*` will tell postgres to accept connections from any remote machine. Alternately, you can specify a comma-separated list of hostnames or IP addresses. (For more info, have a look at the `postgresql.conf` file from your system's postgres package).####Class: Iqss::DataverseThis class installs Glassfish, the domain settings and depending on the configuration builds a war or pulls a war distribution from a repository. Example:    class {        'iqss::dataverse':            package => 'dataverse-4.0.1',            repository => 'https://github.com/IQSS/dataverse/releases/download/v4.0.1/dataverse-4.0.1.war',     }    This will create three services:* The Dataverse 4.0.1 distribution plus glassfish service: $ service glassfish start|stop|status* An R-daemon: $ service rserve start|stop|status* The Apache web serverIt also contains settings for#####`auth_password_reset_timeout_in_minutes`A JVM option: the time in minutes for a password reset. Defaults to '60'.#####`doi_baseurlstring`The DOI endpoint for the EZID Service. Defaults to 'https://ezid.cdlib.org'. #####`doi_username`The username to connect to the EZID Service. Defaults to 'apitest'. #####`doi_password`The password to connect to the EZID Service. Defaults to 'apitest'. #####`files_directory`The location of the uploaded files and their tabular derivatives. Defaults to '/home/glassfish/dataverse/files'.#####`glassfish_parent_dir`The Glassfish parent directory. Defaults to '/usr/local'.#####`glassfish_domain_name`The domain name. Defaults to 'domain1'.#####`glassfish_fromaddress`The e-mail -from field in the mail header. Defaults to 'do-not-reply@localhost'.#####`glassfish_jvmoption`An array of jvm options. Defaults to ["-Xmx1024m", "-Djavax.xml.parsers.SAXParserFactory=com.sun.org.apache.xerces.internal.jaxp.SAXParserFactoryImpl"].#####`glassfish_mailhost`The mail relay hostname. Defaults to `Globals:dataverse_fqdn`.#####`glassfish_mailuser`The user name that is allowed by the mail relay to sent mails. Defaults to 'dataversenotify'.#####`glassfish_mailproperties`Key-value pairs sent with to the mail relay, such as credentials. Defaults to dummy values 'username=a_username:password=a_password'.#####`glassfish_service_name`The service handle to submit start, stop, status commands. E.g. service dataverse start. Defaults to 'glassfish'#####`glassfish_tmp_dir`The download path of the glassfish package. Defaults to '/opt/glassfish'.#####`glassfish_user`The user running the glassfish domain. Defaults to 'glassfish'.#####`glassfish_version`The Glassfish J2EE Application server version. Defaults to '4.1'.#####`package`The package name and version used for the default `repository` value. Defaults to 'dataverse-4.0.1'.#####`port`The SSL port apache listens to. Defaults to `Globals:dataverse_port`.#####`repository`This indicates there the package comes from. It can be 'git' to build a war from the IQSS repository; or the repositoryurl of a Dataverse war file. Defaults to 'https://github.com/IQSS/dataverse/releases/download/v4.0.1/dataverse-4.0.1.war'.#####`rserve_host`The Rserve service hostname. Defaults to `Globals:dataverse_fqdn`.#####`rserve_password`The password needed to access the Rserve service. Defaults to 'rserve'.#####`rserve_port`The Rserve service port. Defaults to 'rserve'. Defaults to '6311'.#####`rserve_user`The serve service user. Defaults to 'serve'.  #####`site_url`The url to a dataverse web application. Defaults to 'https://`dataverse_fqdn`:443'.####Class: Iqss::GlobalsThis class allows you to configure the global configuration that contain settings shared amongst classes,most notably the database settings. Example:    class {      'iqss::globals':        ensure            => present,        dataverse_fqdn    => 'mysite.org',        database_name     => 'dataverse',        database_user     => 'dataverse',        database_password => 'Cárammë',    }    It also contains settings for#####`apache_purge_configs`Removes all other Apache configs and vhosts. Setting this to 'false' is a stopgap measure to allow the apache module to coexist with existing or otherwise-managed configuration. Defaults to 'true'.#####`dataverse_fqdn`If the Dataverse server has multiple DNS names, this option specifies the one to be used as the “official” host name. For example, you may want to have dataverse.foobar.edu, and not the less appealling server-123.socsci.foobar.edu to appear exclusively in all the registered global identifiers, Data Deposit API records, etc. Defaults to 'localhost'.Do note that whenever the system needs to form a service URL, by default, it will be formed with https:// and port 443. I.e.,https://{dataverse.fqdn}/If that does not suit your setup, use the `Iqss::Dataverse::site_url` option.#####`database_host`The domain of the database. Defaults to `Globals:dataverse_fqdn`.#####`database_port`The port of the database. Defaults to '5432'.#####`database_name`The name of the database. Defaults to 'dvndb'.#####`database_user`The name of the database owner. Defaults to 'dvnApp'.#####`database_password`The password for the database user. Defaults to 'dvnAppPass'.#####`rserve_pwd`The password for the rserve daemon. Defaults to 'rserve'.####Class: Iqss::RserveInstalls the Rserve or Binary R server daemon. Example:                                class { 'iqss::rserve':        pwd  => 'potato',    }    This will install R and a number of R packages including the RServe package. A password file /etc/Rserve.pwd is set with content: rserve potato.Rserve is run by the 'rserve' user.#####`auth`If you need remote access use set auth 'required' and plaintext to 'disable'. Defaults to 'disable'.#####`chroot`The jail directory. Defaults tot undef.#####`encoding`This means that strings are converted to the given encoding before being sent to the client and also all strings from the client are assumed to come from the given encoding. Defaults to 'utf8'.#####`eval`Preload packages with expressions that you would otherwise have to load from scripts. Defaults to undef. #####`fileio`Allow clients to perform filesystem operations via the RServe deamon. Defaults to 'enable'.#####`gid`The group id of the 'rserve' user runnning the daemon. Defaults to 97.#####`interactive`Undocumented. Defaults to 'yes'.#####`maxinbuf`The maximum allowed buffer size send from the client per connection. Defaults to 262144 Kb.#####`maxsendbuf`The maximum allowed buffer size send from the server per connection. Defaults to 0 Kb.#####`plaintext`Allows for sending credentials as plaintext. Defaults to 'disable'.#####`port`The TCP port the daemon listens too. Defaults to '6311'#####`pwdfile`The password file containing the authentication credentials. Defaults to '/etc/Rserve.pwd'.#####`remote`Allows remote connections when enabled. Defaults to 'enable'.#####`socket`Undocumented. Defaults to undef.#####`sockmod`Undocumented. Defaults to undef.#####`source`Location to a file to preload packages that you would otherwise have to load from your scripts. Defaults to undef.#####`su`Undocumented. Defaults to undef.#####`uid`The user id of the 'rserve' user running the daemon. Defaults to 97.#####`umask`Controls how file permissions are set for files. Defaults to 0.#####`workdir` R working directory.Defaults to '/tmp/Rserv'####Class: Iqss::SolrInstalls Solr. Example:    class { 'iqss::solr':        version => '4.7.1',    }    This will create a Jetty server with a running Solr instance : $ service solr stop|status|startIt also contains settings for#####`core`The solr core. Defaults to 'collection1'.#####`jetty_home`The Jetty home directory which contains start.jar. Defaults to '/home/solr-4.6.0/example'#####`jetty_host`Use 0.0.0.0 as host to accept all connections. Defaults to `Globals:dataverse_fqdn`.#####`jetty_java_options`JVM options for Jetty. Defaults to '-Xmx512m'.#####`jetty_port`The port Jetty will bind to. Defaults to '8983'.#####`jetty_user`The user running the Jetty Solr instance. Defaults to 'solr'.#####`solr_home`The Solr home used for the jvm setting -Dsolr.solr.home. Defaults to '/home/solr-4.6.0/example/solr'.#####`parent_dir`The home directory of Solr. Defaults to '/opt/solr-4.6.0'.#####`url`The download url for solr. Preferably a mirror. Defaults to 'http://archive.apache.org/dist/lucene/solr'.#####`version`The Apache Solr version. Defaults to '4.6.0'.####Class: Iqss::TworavensThis class installs the Apache RApache handler and the Tworavens web application. For example:    class {      'iqss::tworavens':        tworavens_package => 'https://github.com/IQSS/TwoRavens/archive/master.zip',        parent_dir        => '/var/www/html',    }    It also contains settings for    #####`domain`The public domain name of the TwoRavens web application. Defaults to 'localhost'.  #####`package`The download url of TwoRavens. Defaults to 'https://github.com/IQSS/TwoRavens/archive/v0.1.zip'.#####`parent_dir`The installation directory of the TwoRavens web application. Defaults to '/var/www/html'. #####`port`The port TwoRavens can be accessed on. Defaults to '443'.#####`protocol`The protocol TwoRavens can be accessed on. Defaults to 'https'.#####`rapache_version`The rapache version to be installed. Defaults to '1.2.6'.#####`tworavens_dataverse_fqdn`The domain name of the dataverse web application this TwoRavens web application will connect to. Defaults to 'localhost'.#####`tworavens_dataverse_port`The port of the dataverse web application. Defaults to '443'.Hieradata---------This example shows how all default settings can be set with a hieradata document. Note that you can alsoinject values like the R packages or package_repo:```javascript{    "iqss::database::createdb": false,    "iqss::database::createrole": false,    "iqss::database::encoding": "UTF-8",    "iqss::database::listen_addresses": "*",    "iqss::database::locale": "en_US.UTF-8",    "iqss::database::login": true,    "iqss::database::manage_package_repo": true,    "iqss::database::hba_rule": {        "IPv4 local connections": {            "description": "Open up a IP4 connection from localhost",            "type": "host",            "database": "dvndb",            "user": "dvnApp",            "address": "127.0.0.1/32",            "auth_method": "md5"        },        "IPv6 local connections": {            "description": "Open up a IP6 connection from localhost",            "type": "host",            "database": "dvndb",            "user": "dvnApp",            "address": "::1/128",            "auth_method": "md5"        }    },    "iqss::database::superuser": false,    "iqss::database::version": "9.3",    "iqss::dataverse::auth_password_reset_timeout_in_minutes": "60",    "iqss::dataverse::files_directory": "/home/glassfish/dataverse/files",    "iqss::dataverse::package": "dataverse-4.0.1",    "iqss::dataverse::port": 443,    "iqss::dataverse::rserve_host": "localhost",    "iqss::dataverse::rserve_password": "rserve",    "iqss::dataverse::rserve_port": "6311",    "iqss::dataverse::rserve_user": "rserve",    "iqss::dataverse::site_url": "https://localhost:443",    "iqss::dataverse::doi_username": "apitest",    "iqss::dataverse::doi_password": "apitest",    "iqss::dataverse::doi_baseurlstring": "https\://ezid.cdlib.org",    "iqss::dataverse::glassfish_domain_name": "domain1",    "iqss::dataverse::glassfish_fromaddress": "do-not-reply@localhost",    "iqss::dataverse::glassfish_jvmoption": [        "-XX\:MaxPermSize=512m",        "-XX\:PermSize=256m",        "-Xmx1024m"    ],    "iqss::dataverse::glassfish_parent_dir": "/home/glassfish",    "iqss::dataverse::glassfish_service_name": "glassfish",    "iqss::dataverse::glassfish_tmp_dir": "/opt/glassfish",    "iqss::dataverse::glassfish_user": "glassfish",    "iqss::dataverse::glassfish_version": "4.1",    "iqss::dataverse::glassfish_mailhost": "localhost",    "iqss::dataverse::glassfish_mailuser": "dataversenotify",    "iqss::dataverse::glassfish_mailproperties": {        "username":"a username",        "password":"a password"    },    "iqss::dataverse::repository": "https://github.com/IQSS/dataverse/releases/download/v4.0.1/dataverse-4.0.1.war",    "iqss::globals::apache2_purge_configs": true,    "iqss::globals::database_host": "localhost",    "iqss::globals::dataverse_fqdn": "localhost",    "iqss::globals::dataverse_port": 443,    "iqss::globals::database_port": 5432,    "iqss::globals::database_name": "dvndb",    "iqss::globals::database_user": "dvnApp",    "iqss::globals::database_password": "dvnAppPass",    "iqss::rpackager::packages": {        "AER": {            "version": "1.2-4"        },        "Amelia": {            "version": "1.7.3"        },        "DescTools": {            "version": "0.99.13"        },        "devtools": {            "version": "1.9.1"        },        "dplyr": {            "version": "0.4.3"        },        "geepack": {            "version": "1.2-0"        },        "jsonlite": {            "version": "0.9.17"        },        "maxLik": {            "version": "1.2-4"        },        "quantreg": {            "version": "5.19"        },        "Rook": {            "version": "1.1-1"        },        "rjson": {            "version": "0.2.15"        },        "RCurl": {            "version": "1.95-4.7"        },        "Rserve": {            "version": "1.7-3"        },        "R2HTML": {            "version": "2.3.1"        },        "VGAM": {            "version": "0.9-8"        }    },    "iqss::rpackager::packages_zelig": "https://github.com/IQSS/Zelig/archive/master.zip",    "iqss::rpackager::repo": "http://cran.r-project.org",    "iqss::rserve::auth": "required",    "iqss::rserve::encoding": "utf8",    "iqss::rserve::fileio": "enable",    "iqss::rserve::interactive": "yes",    "iqss::rserve::maxinbuf": 262144,    "iqss::rserve::maxsendbuf": 0,    "iqss::rserve::plaintext": "disable",    "iqss::rserve::port": 6311,    "iqss::rserve::pwd": "rserve",    "iqss::rserve::pwdfile": "/etc/Rserv.pwd",    "iqss::rserve::remote": "enable",    "iqss::rserve::sockmod": 0,    "iqss::rserve::umask": 0,    "iqss::rserve::workdir": "/tmp/Rserv",    "iqss::solr::url": "http://archive.apache.org/dist/lucene/solr",    "iqss::solr::version": "4.6.0",    "iqss::solr::parent_dir": "/home/solr-4.6.0",    "iqss::solr::jetty_user": "solr",    "iqss::solr::jetty_host": "localhost",    "iqss::solr::jetty_port": "8983",    "iqss::solr::jetty_java_options": "-Xmx512m",    "iqss::solr::jetty_home": "/home/solr-4.6.0/example",    "iqss::solr::solr_home": "/home/solr-4.6.0/example/solr",    "iqss::solr::core": "collection1",    "iqss::tworavens::dataverse_fqdn": "localhost",    "iqss::tworavens::dataverse_port": 443,    "iqss::tworavens::domain": "localhost",    "iqss::tworavens::package": "https://github.com/IQSS/TwoRavens/archive/master.zip",    "iqss::tworavens::parent_dir": "/var/www/html",    "iqss::tworavens::port": 443,    "iqss::tworavens::protocol": "https",    "iqss::tworavens::rapache_version": "1.2.6",    "iqss::tworavens::tworavens_dataverse_port": 443}``` Known issues------------* TwoRavens is installed and shows the pebbles, but not the averages, means, distributions in it. Are we lacking packages ?* Rserve does not start after a puppet run ( it does on boot ). You need to start the service manually with `service rserve restart`* Vagrant installations could have various issues per OS and image. If the machine does not play along, then try out the know issues mentioned the [Vagrantfile](Vagrantfile).* If you run Vagrant and install TwoRavens with a localhost domain, the application that runs on the client host will not be able to reach port 443 as it will use localhost:9999 to call the dataverse API.You can get around this last issue by setting a firewall rule in your client machine:    $ iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 9999    $ iptables-saveTo do-----* ShibbolethExamples--------See [the demo installations in the example folder](example) of this module.Contributors-----------* Lucien van Wouw, IISH