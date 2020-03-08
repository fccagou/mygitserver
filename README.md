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

    Usage: mygitserver [--help|-h]
                  [--run]
                  [--user <user>]
                  [--group <group>]
                  [--systemd-enable]
                  [--systemd-uninstall]
                  [--listen-port <port>]
                  [--add-myself-as-git-sample]
                  <prefix_env>
    
       --help|-h                  : This help
       --run                      : Run the server after creation.
                                    Only works for current user.
       --user                     : Install service for his user.
                                    Default is current user (fccagou)
       --group                    : Install service for this group.
                                    Default is current user group(fccagou).
       --systemd-enable           : configure user systemd service and runs it.
                                    Only works for current user.
       --systemd-uninstall        : remove user systemd configuration
                                    Only works for current user.
       --listen-port              : Define http listen port.
                                    Must be greter then 1024.
                                    Default is 8080.
       --add-myself-as-git-sample : Clone myself in repos.
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

## Warning

The servers can be access by anybody connected with a shell on the host.

## TODO
* Allow user defined logo.
* Add httpd server param to manage configuration (apache/nginx/ligthttp...)
* Certainly many others good features.
* Add scripts to manage git bare repos
* Add ssl support
* Add kerberos/ldap support
* Install required distro packages
* Put each feature in a separate script
* Use getops. Not necessary yet.
* Add --env-update or something like that.
* Add current usercheck to use or not sudo.
* Configure systemd for all users

