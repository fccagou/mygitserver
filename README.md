![Cagou git](doc/cagou-git.png)
# mygitserver
## Description

Create a simply git server based on [cgit](https://git.zx2c4.com/cgit/about/)
for teams who do not want to install complex externals systems as
[GitLab](https://about.gitlab.com/)

The solution must use package available in Linux distros:
* http web server
* cgit
* ssh
* ssl
* kerberos/ldap...

The solution must easily create multiple servers for different users/groups.

The solution must give cli interface.


## What's done

The **mygitserver** script creates a httpd user environment using
[apache](http://httpd.apache.org/docs/current/) configuration.

It's developed on [Arch](https://www.archlinux.org/) and tested on [CentOS
7](https://www.centos.org)

    Usage: my_httpd_server [--help|-h] [--systemd-enable] [--systemd-uninstall] <prefix_env>
    
    	--help|-h                  : This help
    	--systemd-enable           : configure user systemd service and runs it
    	--systemd-uninstall        : remove user systemd configuration
    
    	prefix_env                 : dir where cgit and http data will be created

Will create :
    
    prefix_env
     ├── cgit
     │   ├── cgitrc
     │   └── repos
     └── web
         ├── conf
         │   ├── extra
         │   │   └── httpd-cgit.conf
         │   ├── httpd.conf
         │   ├── magic -> /etc/httpd/conf/magic
         │   └── mime.types -> /etc/httpd/conf/mime.types
         ├── modules -> /etc/httpd/modules
         ├── var
         │   ├── httpd.pid
         │   └── log
         │       ├── access_log
         │       └── error_log
         └── www
             └── index.html


Then, the git bare repos have to be created in _prefix_env/cgit/repos_.

Cgit is configured to _scan-path_ and _section-from-path_. So, you
can organize bare repos in subdirs.

It's also possible to link existing git repos.



## TODO
* Define port when multi user on the same host.
* Allow user defined logo.
* Add httpd server param to manage configuration (apache/nginx/ligthttp...)
* Certainly many others good features.
* Add scripts to manage git bare repos
* Add ssl support
* Add kerberos/ldap support
* Install required distro packages
* Put each feature in a separate script

