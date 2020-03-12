![Cagou git](doc/cagou-git.png)
# mygitserver

<!-- vim-markdown-toc GFM -->

* [Description](#description)
* [What's done](#whats-done)
* [Warning](#warning)
* [Selinux](#selinux)
* [How to give user access.](#how-to-give-user-access)
    * [General need](#general-need)
    * [Ok but what about mygitserver ?](#ok-but-what-about-mygitserver-)
        * [Every body can read, nobody can right using Git](#every-body-can-read-nobody-can-right-using-git)
        * [Every body can read and right (DANGEROUS) using Git](#every-body-can-read-and-right-dangerous-using-git)
        * [Every body can read, nobody can right using Apache](#every-body-can-read-nobody-can-right-using-apache)
        * [Every body can read and right (DANGEROUS) using Apache](#every-body-can-read-and-right-dangerous-using-apache)
        * ["Everyone at home and the hippos will be well looked after"  using Apache](#everyone-at-home-and-the-hippos-will-be-well-looked-after--using-apache)
    * [Next ...](#next-)
    * [Links](#links)
* [TODO](#todo)

<!-- vim-markdown-toc -->

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
                  [--update]
                  [--user <user>]
                  [--group <group>]
                  [--systemd-enable]
                  [--systemd-uninstall]
                  [--listen-port <port>]
                  [--system-cgit-script-dir]
                  [--system-cgit-script-name]
                  [--system-cgit-data-dir]
    
    	--help|-h                  : This help
    	--run                      : Run the server after creation.
                                     Only works for current user.
    	--update                   : Update config file in existing env.
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
    	--system-cgit-script-dir   : Define cgit script dir
    	--system-cgit-script-name  : Define cgit script name relative to script dir
    	--system-cgit-data-dir     : Define cgit dir containing web matérial (css, png)
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
## Selinux

On CentOS, the default policy does not allow running cgit. I'ts necessary to
create and install a dedicated module.

First install needed package :

    yum install policycoreutils-devel


then create _cgit.te_ for default httpd server.

    cat > cgit.te <<EOF_TE
    module cgit 1.0;
    
    require {
            type httpd_sys_rw_content_t;
            type git_script_t;
            class dir { getattr open read search };
            class file { getattr open read };
    }
    
    #============= git_script_t ==============
    allow git_script_t httpd_sys_rw_content_t:dir { getattr open read search };
    allow git_script_t httpd_sys_rw_content_t:file { getattr open read };
    EOF_TE

Build the policy

    make -f /usr/share/selinux/devel/Makefile cgit.pp

Install the policy module

    semodule -i cgit.pp


If the git dir is not a default httpd dir, update fcontext

    semanage fcontext -a -t httpd_sys_rw_content_t -r s0  '/srv/git-repos(/.*)?'

## How to give user access.

... DRAFT ...

### General need
There are many options to manage user access to repos.
The general is "which _rights_ do I set for _a user_ on _each repos_"

_note:_ Sometime it could be useful to set rights on a particular branch.

* Alice and Bob can clone all repos
* Alice can push on all repos
* Bob can push on repoA and repoB

The other question is "how can i identify Alice or Bod surely".

This are common questions which have many solution more or less complex. For
examples, 
* [gitolite](https://github.com/sitaramc/gitolite) use ssh key management and
  perl code hooks.
* [gitlab](https://about.gitlab.com/) great complete solution of CI (ssh,http ...)

### Ok but what about mygitserver ?

Has I said before, for this project, I'm looking for _simple native solution_.
Because actual code is based on web server, the solution can use all access
rights mechanism given by those http solutions. 

And this can be easily done filtering some Git requests and setting rights with
the different identification and authentication methods ([apache
access](https://httpd.apache.org/docs/current/howto/access.html))

#### Every body can read, nobody can right using Git

This is actually the defaults configuration. The push is not allowed and failed
with "DAV error" because Git backend do not receive identity.

#### Every body can read and right (DANGEROUS) using Git

As describes in [git-http-backend](https://git-scm.com/docs/git-http-backend), push is disabled by default.
To enable the push in a specific repo, got to the repo directory and run 

     git config http.receivepack true


#### Every body can read, nobody can right using Apache

The following apache configuration can be added 

    <If "(%{QUERY_STRING} -strmatch '*service=git-receive-pack*' || %{REQUEST_URI} =~ m#/git-receive-pack$#)">
    # Deny push
    Require all denied
    </If>
    <Else>
    # Allow read 
    Require all granted
    </Else>

A 403 http error will be returns.


#### Every body can read and right (DANGEROUS) using Apache

The following apache configuration can be added 

    <If "(%{QUERY_STRING} -strmatch '*service=git-receive-pack*' || %{REQUEST_URI} =~ m#/git-receive-pack$#)">
    # Deny push
    Require all granted
    </If>
    <Else>
    # Allow read 
    Require all granted
    </Else>



#### "Everyone at home and the hippos will be well looked after"  using Apache

Apache has a macro mechanism that can be used to configure each repos.

I use Apache since many years, not in an intensive way, I have built a module to
parse and modify HTML page to act has a reverseproxy ... and I just discovered
this cool one...  I love OpenSource !!

So ... a way to do the stuff is :
* loading the macro's module
* writing the macro
* setting the configuration file to use macro

The next example use basic auth.


__Loading the macro's module__

    LoadModule macro_module modules/mod_macro.so


__Writing the macro__

    <Macro Project ${repository} ${developers} ${users}>
      # The macro Project takes 3 parameters
      # - repository: the git repo
      # - developers: list of user separated by <space>
      # - users: list of users separated by <space>

      <LocationMatch "^/${repository}.*">
        # Set Basic authentication for the repo

        AuthType Digest
        AuthName "Git Access"
        AuthDigestProvider file
        AuthUserFile /some/where/apache/can/read/.htpasswd

        <If "(%{QUERY_STRING} -strmatch '*service=git-receive-pack*' || %{REQUEST_URI} =~ m#/git-receive-pack$#)">
          # Allows push for develoers
          Require user ${developers}
        </If>
        <Else>
          # Allows clone/read for every known users.
          Require user ${developers} ${users}
        </Else>

     </LocationMatch>
    </Macro>

    # Include config file to déclare all needed repos.
    IncludeOptional /some/where/apache/can/read/git_access.conf

__Setting the configuration file to use macro__

The config file is a Apache conf containg the macro call for each repository.

    $ cat /some/where/apache/can/read/git_access.conf
    Use Project mygitserver.git "admin puppetupdate" "fccagou"
    Use Project tests "puppetupdate fccagou" ""

__Basic auth passwd file__

A standard basic file created using _htdigest_ command line

    $ htdigest /some/where/apache/can/read/.htpasswd "Git Access" fccagou
    $ htdigest /some/where/apache/can/read/.htpasswd "Git Access" puppetupdate

    $ cat /some/where/apache/can/read/.htpasswd
    fccagou:Git Access:xxxxxxx
    puppetupdate:Git Access:yyyyyyyyy

### Next ...

Hum ... go go go ... code to be alive !

### Links

* [MIT](http://web.mit.edu/git/git-doc/howto/setup-git-server-over-http.html)
* [SSL](https://stackoverflow.com/questions/11621768/how-can-i-make-git-accept-a-self-signed-certificate)
* [PI/lighttp](https://dhole.github.io/post/raspberry_pi_git/)
* [LINUXCOM](https://www.linux.com/tutorials/give-your-git-repository-open-source-web-interface/)


## TODO
* Allow user defined logo.
* Add -v/--verbose parameter.
* Add httpd server param to manage configuration (apache/nginx/ligthttp...)
* Certainly many others good features.
* Add scripts to manage git bare repos
* Add ssl support
* Add kerberos/ldap support
* Install required distro packages
* Put each feature in a separate script
* Use getops. Not necessary yet.
* Add current usercheck to use or not sudo.
* Configure systemd for all users
* make a [puppet/bolt](https://puppet.com/docs/bolt/latest/bolt.html) project.



