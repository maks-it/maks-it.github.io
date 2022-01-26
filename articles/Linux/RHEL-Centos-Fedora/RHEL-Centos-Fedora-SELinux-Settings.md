# Intro

Security-Enhanced Linux (SELinux) is a Linux kernel security module that provides a mechanism for supporting access control security policies, including mandatory access controls (MAC).

SELinux is a set of kernel modifications and user-space tools that have been added to various Linux distributions. Its architecture strives to separate enforcement of security decisions from the security policy, and streamlines the amount of software involved with security policy enforcement. The key concepts underlying SELinux can be traced to several earlier projects by the United States National Security Agency (NSA).

## GUI Selinux tools

```bash
$ dnf install policycoreutils-gui
$ dnf install setroubleshoot
```


## SELinux allow NGINX

Run to check SELinux status

``` bash
sestatus -v
```

If this is not permissive, check the audit logs, you should find the access error:

``` bash
ausearch -m avc -ts today | audit2allow
```

``` bash
#!!!! This avc can be allowed using the boolean 'httpd_can_network_connect'
allow httpd_t commplex_main_port_t:tcp_socket name_connect;

#!!!! WARNING: 'usr_t' is a base type.
allow httpd_t usr_t:file append;
```

You must tell SELinux about this by enabling the 'httpd_can_network_connect' boolean.

``` bash
setsebool -P httpd_can_network_connect 1
```

## SELinux fix in case of you move and not copy with wrong permissions

Run to check SELinux status

``` bash
sestatus -v
```

If this is not permissive, check the audit logs, you should find the access error:

``` bash
ausearch -m avc -ts today | audit2allow
```

``` bash
#!!!! This avc can be allowed using the boolean 'httpd_read_user_content'
allow httpd_t user_home_t:file read;

#!!!! WARNING: 'var_t' is a base type.
#!!!! The file '/var/www/nastyarey.com/index.html' is mislabeled on your system.  
#!!!! Fix with $ restorecon -R -v /var/www/nastyarey.com/index.html
allow httpd_t var_t:file getattr;
```

You also probably moved the filed instead of copying it, so the security context of the file might be wrong.

``` bash
ls -lrtZ /var/www/nastyarey.com*
```

and correct it if needed:

``` bash
restorecon -v -R /var/www/nastyarey.com*
```

or

You must tell SELinux about this by enabling the 'httpd_read_user_content' boolean.

``` bash
setsebool -P httpd_read_user_content 1
```

### View SELinux variables

``` bash
getsebool -a | grep httpd
```

## SELinux issues continue (Not necessary for the moment)

error : you get a 403 Forbidden when you try to browse to

``` bash
tail /var/log/nginx/error.log
```

and check for error:

``` bash
2019/06/02 18:39:26 [error] 1699#0: *14 "/var/www/example.com/html/index.html" is forbidden (13: Permission denied), client: 172.16.45.15, server: example.com, request: "GET / HTTP/1.1", host: "www.example.com"
```

Install `setools`:

``` bash
yum install -y setools
```

get semanage (comes with audit2allow):

``` bash
[root@srvweb0001 ~]# yum provides /usr/sbin/semanage
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: ftp.wicks.co.nz
 * epel: mirror.xnet.co.nz
 * extras: ftp.wicks.co.nz
 * updates: ftp.wicks.co.nz
policycoreutils-python-2.5-17.1.el7.x86_64 : SELinux policy core python utilities
Repo        : base
Matched from:
Filename    : /usr/sbin/semanage

[root@srvweb0001 ~]# yum install -y policycoreutils-python-2.5-17.1.el7.x86_64
```

find selinux errors in log, use audit2allow to format out a fix:

``` bash
grep nginx /var/log/audit/audit.log | audit2allow -m nginx
```

``` bash
cat nginx

module nginx 1.0;

require {
        type httpd_t;
        type var_t;
        class file { getattr open read };
}

#============= httpd_t ==============

#!!!! WARNING: 'var_t' is a base type.
#!!!! The file '/var/www/example.com/html/index.html' is mislabeled on your system.
#!!!! Fix with $ restorecon -R -v /var/www/example.com/html/index.html
allow httpd_t var_t:file { getattr open read };
```

create an compiled policy with the -M option:

``` bash
grep nginx /var/log/audit/audit.log | audit2allow -M nginx
******************** IMPORTANT ***********************
To make this policy package active, execute:

semodule -i nginx.pp
```

let’s do it:

``` bash
semodule -i nginx.pp
```

 and then check its installed :

 ``` bash
 semodule -l | grep nginx
 ```

output should be

``` bash
nginx   1.0
```

Now it's working!

``` bash
cat nginx
```

``` bash
module nginx 1.0;

require {
        type httpd_t;
        type var_t;
        class file { getattr open read };
}

#============= httpd_t ==============

#!!!! The file '/var/www/example.com/html/index.html' is mislabeled on your system.  
#!!!! Fix with $ restorecon -R -v /var/www/example.com/html/index.html
allow httpd_t var_t:file open;

#!!!! This avc is allowed in the current policy
allow httpd_t var_t:file { getattr read };

```
































How To Enable Or Disable SELinux In CentOS/RHEL 7
Posted by Jarrod on September 21, 2016 Leave a comment (4)Go to comments
Security Enhanced Linux (SELinux) is enabled and running in enforcing mode by default in CentOS/RHEL based Linux operating systems, and with good reason as it increases overall system security.

Despite this there may be times when you want to temporarily or permanently disable SELinux, which is what we’ll cover here.


Note: SELinux is incredibly valuable as part of an overall Linux system security strategy, and we recommend leaving it enabled in enforcing mode in production environments where possible. If a particular application or package does not work properly with SELinux customized allowances can be made which is the preferred option compared to simply disabling the whole thing.


 

SELinux Basics
First off, a quick overview of the three different SELinux modes. SELinux can be in enforcing, permissive, or disabled mode.

Enforcing:
This is the default. In enforcing mode, if something happens on the system that is against the defined policy, the action will be both blocked and logged.

Permissive:
This mode will not actually block or deny anything from happening, however it will log anything that would have normally been blocked in enforcing mode. It’s a good mode to use if you perhaps want to test a Linux system that has never used SELinux and you want to get an idea of any problems you may have. No system reboot is needed when swapping between permissive and enforcing modes.

Disabled:
Disabled is completely turned off, nothing is logged at all. In order to swap to the disabled mode, a system reboot will be required. Additionally if you are switching from disabled mode to either permissive or enforcing modes a system reboot will also be required.

View Current SELinux Status
As mentioned CentOS/RHEL use SELinux in enforcing mode by default, there are a few ways that we can check and confirm this. My favourites are with the ‘getenforce’ and ‘sestatus’ commands.

[root@centos7 ~]# getenforce
Enforcing

[root@centos7 ~]# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28
As shown above both of these show that we are currently in enforcing mode.

Change SELinux Mode
There are also many ways that we can change the mode of SELinux, with both runtime only options or permanent settings that persist on reboot.

SELinux Runtime Configuration
One of the fastest ways to switch between enforcing and permissive modes is with the ‘setenforce’ command. We can use ‘setenforce 0’ to swap to permissive mode, or ‘setenforce 1’ to swap to enforcing mode.

[root@centos7 ~]# getenforce
Enforcing
[root@centos7 ~]# setenforce 0
[root@centos7 ~]# getenforce
Permissive
[root@centos7 ~]# setenforce 1
[root@centos7 ~]# getenforce
Enforcing
Note that this only changes the runtime setting, if you perform a system reboot the option stored in the /etc/selinux/config file will be used at next boot. We cannot disable selinux at runtime, as swapping to or from the disabled mode requires a system reboot.

SELinux Persistent Configuration
We can edit the /etc/selinux/config text file with our persistent setting, either enforcing, permissive, or disabled. By default this file appears as shown below.

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
We can simply edit the SELINUX variable between enforcing, permissive, or disabled, as outlined in the comments of the file. After editing the file the changes will not be in place immediately and will only change after system reboot.

Troubleshooting SELinux
So you have something that’s not working with SELinux enforcing mode, rather than setting SELinux to permissive or even disabled, we can troubleshoot and investigate the problem to try and fix it which is better than turning the whole thing off. Turning SELinux off should be considered a last resort.

First install the setroubleshoot-server package with ‘yum’.

[root@centos7 ~]# yum install setroubleshoot-server -y
With this package we get the ‘sealert’ command, which will help us uncover any problems along with display recommended ways of fixing the problem.

In this example I have created an index.html file in the /root directory, and then moved it to /var/www/html for Apache to serve out.

[root@centos7 ~]# vim index.html
[root@centos7 ~]# mv index.html /var/www/html/
However when I try to view the index file in Firefox, the index.html page content does not display and I get the below error in the /var/log/messages file.

Aug 28 00:15:51 localhost setroubleshoot: SELinux is preventing /usr/sbin/httpd from getattr access on the file /var/www/html/index.html. For complete SELinux messages. run sealert -l 284cb2c9-1c2e-4708-a48d-415123f558aa
Aug 28 00:15:51 localhost python: SELinux is preventing /usr/sbin/httpd from getattr access on the file /var/www/html/index.html.#012#012*****  Plugin restorecon (99.5 confidence) suggests   ************************#012#012If you want to fix the label. #012/var/www/html/index.html default label should be httpd_sys_content_t.#012Then you can run restorecon.#012Do#012# /sbin/restorecon -v /var/www/html/index.html#012#012*****  Plugin catchall (1.49 confidence) suggests   **************************#012#012If you believe that httpd should be allowed getattr access on the index.html file by default.#012Then you should report this as a bug.#012You can generate a local policy module to allow this access.#012Do#012allow this access for now by executing:#012# grep httpd /var/log/audit/audit.log | audit2allow -M mypol#012# semodule -i mypol.pp#012
This is essentially saying that Apache is not able to access the index.html file as it has the incorrect SELinux context. The SELinux context of the file is shown below with the -Z option from ‘ls’.

[root@centos7 ~]# ls -laZ /var/www/html/
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 .
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 ..
-rw-r--r--. root root unconfined_u:object_r:admin_home_t:s0 index.html
As this file was created in the /root directory, it has the SELinux context of ‘admin_home_t’ and by default Apache will only serve files with a context of ‘httpd_sys_content_t’. The logs suggest that this can be fixed by running the restorecon command, which will fix the SELinux context of the file, and sure enough it does and the page now loads correctly.

[root@client ~]# restorecon -v /var/www/html/index.html
restorecon reset /var/www/html/index.html context unconfined_u:object_r:admin_home_t:s0->unconfined_u:object_r:httpd_sys_content_t:s0
Further information is also logged to the /var/log/audit/audit.log file, however the content is not very human readable. This is where the ‘sealert’ command comes into help.

[root@centos7 ~]# sealert -a /var/log/audit/audit.log
--------------------------------------------------------------------------------

SELinux is preventing /usr/sbin/httpd from getattr access on the file /var/www/html/index.html.

*****  Plugin restorecon (99.5 confidence) suggests   ************************

If you want to fix the label.
/var/www/html/index.html default label should be httpd_sys_content_t.
Then you can run restorecon.
Do
# /sbin/restorecon -v /var/www/html/index.html
The -a will display all alerts, however it can also be used to view specific codes that may be provided in the /var/log/messages file. Again the recommendation here provides an exact command to run to fix the problem, easy! Hopefully you can start to see that with these techniques there’s usually no real reason to disable SELinux.


 

Summary
As shown it’s pretty easy to change between SELinux modes either persistently or at run time only.

Rather than disabling SELinux, it is always recommended to leave it running in enforcing mode and fix any standalone issues rather than compromising the security of the entire system. This is fairly simple to do with the ‘sealert’ command which comes from the setroubleshoot-server package.































Using NGINX and NGINX Plus with SELinux
TWITTER
LINKEDIN
Editor – The blog post titled “NGINX: SELinux Changes when Upgrading to RHEL 6.6 / CentOS 6.6” redirects here. This article provides updated and generalized information.

The default settings for Security-Enhanced Linux (SELinux) on modern Red Hat Enterprise Linux (RHEL) and related distros can be very strict, erring on the side of security rather than convenience. Although the default settings do not limit the functioning of NGINX Open Source and NGINX Plus in their default configurations, other features you might configure can be blocked unless you explicitly allow them in SELinux. This article describes the possible issues and recommended ways to resolve them.

[Editor – This article applies to both NGINX Open Source and NGINX Plus. For ease of reading, the term “NGINX” is used throughout.

CentOS is a related distro originally derived from RHEL and is supported by NGINX and NGINX Plus. In addition, NGINX Plus supports the related Amazon Linux and Oracle Linux distros. Their default SELinux settings might differ from CentOS and RHEL; consult the vendor documentation.]

Overview of SELinux

SELinux is enabled by default on modern RHEL and CentOS servers. Each operating system object (process, file descriptor, file, etc.) is labeled with an SELinux context that defines the permissions and operations the object can perform. In RHEL 6.6/CentOS 6.6 and later, NGINX is labeled with the httpd_t context:

# ps auZ | grep nginx
unconfined_u:system_r:httpd_t:s0 3234 ? Ss 0:00 nginx: master process /usr/sbin/nginx \
                                                -c /etc/nginx/nginx.conf
unconfined_u:system_r:httpd_t:s0 3236 ? Ss 0:00 nginx: worker process
The httpd_t context permits NGINX to listen on common web server ports, to access configuration files in /etc/nginx, and to access content in the standard docroot location (/usr/share/nginx). It does not permit many other operations, such as proxying to upstream locations or communicating with other processes through sockets.

Temporarily Disabling SELinux for NGINX
To temporarily disable SELinux restrictions for the httpd_t context, so that NGINX can perform all the same operations as in non‑SELinux OSs, assign the httpd_t context to the permissive domain. See the next section for details.

# semanage permissive -a httpd_t
Changing SELinux Modes
SELinux can be run in enforcing, permissive, or disabled modes (also referred to as domains). Before you make a NGINX configuration change that might breach the default (strict) permissions, you can change SELinux from enforcing to permissive mode, in your test environment (if available) or production environment. In permissive mode, SELinux permits all operations, but logs operations that would have breached the security policy in enforcing mode.

To add httpd_t to the list of permissive domains, run this command:

# semanage permissive -a httpd_t
To delete httpd_t from the list of permissive domains, run:

# semanage permissive -d httpd_t
To set the mode globally to permissive, run:

# setenforce 0
To set the mode globally to enforcing, run:

# setenforce 1
Resolving SELinux Security Exceptions

In permissive mode, security exceptions are logged to the default Linux audit log, /var/log/audit/audit.log. If you encounter a problem that occurs only when NGINX is running in enforcing mode, review the exceptions that are logged in permissive mode and update the security policy to permit them.

Issue 1: Proxy Connection is Forbidden
By default, the SELinux configuration does not allow NGINX to connect to remote HTTP, FastCGI, or other servers, as indicated by an audit log message like the following:

type=AVC msg=audit(1415714880.156:29): avc:  denied  { name_connect } for  pid=1349 \
  comm="nginx" dest=8080 scontext=unconfined_u:system_r:httpd_t:s0 \
  tcontext=system_u:object_r:http_cache_port_t:s0 tclass=tcp_socket
type=SYSCALL msg=audit(1415714880.156:29): arch=c000003e syscall=42 success=no \
  exit=-115 a0=b \a1=16125f8 a2=10 a3=7fffc2bab440 items=0 ppid=1347 pid=1349 \
  auid=1000 uid=497 gid=496 euid=497 suid=497 fsuid=497 egid=496 sgid=496 fsgid=496 \
  tty=(none) ses=1 comm="nginx" exe="/usr/sbin/nginx" \
  subj=unconfined_u:system_r:httpd_t:s0 key=(null)
The audit2why command interprets the message code (1415714880.156:29):

# grep 1415714880.156:29 /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1415714880.156:29): avc:  denied  { name_connect } for  pid=1349 \
  comm="nginx" dest=8080 scontext=unconfined_u:system_r:httpd_t:s0 \
  tcontext=system_u:object_r:http_cache_port_t:s0 tclass=tcp_socket
 
        Was caused by:
        One of the following booleans was set incorrectly.
        Description:
        Allow httpd to act as a relay
 
        Allow access by executing:
        # setsebool -P httpd_can_network_relay 1
        Description:
        Allow HTTPD scripts and modules to connect to the network using TCP.
 
        Allow access by executing:
        # setsebool -P httpd_can_network_connect 1
The output from audit2why indicates that you can allow NGINX to make proxy connections by enabling one or both of the httpd_can_network_relay and httpd_can_network_connect Boolean options. You can enable them either temporarily or permanently, the latter by adding the ‑P flag as shown in the output.

Understanding Boolean Options
The sesearch command provides more information about the Boolean options, and is available if you install the setools package (yum install setools). Here we show the output for the httpd_can_network_relay and httpd_can_network_connect options.

The httpd_can_network_relay Boolean Option
Here’s the output from the sesearch command about the httpd_can_network_relay option:

# sesearch -A -s httpd_t -b httpd_can_network_relay
Found 10 semantic av rules:
   allow httpd_t gopher_port_t : tcp_socket name_connect ;
   allow httpd_t http_cache_client_packet_t : packet { send recv } ;
   allow httpd_t ftp_port_t : tcp_socket name_connect ;
   allow httpd_t ftp_client_packet_t : packet { send recv } ;
   allow httpd_t http_client_packet_t : packet { send recv } ;
   allow httpd_t squid_port_t : tcp_socket name_connect ;
   allow httpd_t http_cache_port_t : tcp_socket name_connect ;
   allow httpd_t http_port_t : tcp_socket name_connect ;
   allow httpd_t gopher_client_packet_t : packet { send recv } ;
   allow httpd_t memcache_port_t : tcp_socket name_connect ;
This output indicates that httpd_can_network_relay allows processes labeled with the httpd_t context (such as NGINX) to connect to ports of various types, including type http_port_t:

# semanage port -l | grep http_port_t
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
To add more ports (here, 8082) to the set of ports permitted for http_port_t, run:

# semanage port -a -t http_port_t -p tcp 8082
If the output from this command says that a port is already defined, as in the following example, it means the port is included in another set. Do not reassign it to http_port_t, because other services might be negatively affected.

# semanage port -a -t http_port_t -p tcp 8080
/usr/sbin/semanage: Port tcp/8080 already defined
# semanage port -l | grep 8080
http_cache_port_t              tcp      3128, 8080, 8118, 8123, 10001-10010
The httpd_can_network_connect Boolean Option
Here’s the output from the sesearch command about the httpd_can_network_connect option:

# sesearch -A -s httpd_t -b httpd_can_network_connect
Found 1 semantic av rules:
   allow httpd_t port_type : tcp_socket name_connect ;
This output indicates that httpd_can_network_connect allows processes labeled with the httpd_t context (such as NGINX) to connect to all TCP socket types that have the port_type attribute. To list them, run:

# seinfo -aport_type -x
Issue 2: File Access is Forbidden
By default, the SELinux configuration does not allow NGINX to access files outside of well‑known authorized locations, as indicated by an audit log message like the following:

type=AVC msg=audit(1415715270.766:31): avc:  denied  { getattr } for  pid=1380 \
  comm="nginx" path="/www/t.txt" dev=vda1 ino=1084 \
  scontext=unconfined_u:system_r:httpd_t:s0 \
  tcontext=unconfined_u:object_r:default_t:s0 tclass=file
The audit2why command interprets the message code (1415715270.766:31):

# grep 1415715270.766:31 /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1415715270.766:31): avc:  denied  { getattr } for  pid=1380 \
  comm="nginx" path="/www/t.txt" dev=vda1 ino=1084 \
  scontext=unconfined_u:system_r:httpd_t:s0 \
  tcontext=unconfined_u:object_r:default_t:s0 tclass=file
 
    Was caused by:
        Missing type enforcement (TE) allow rule.
 
        You can use audit2allow to generate a loadable module to allow this access.
When file access is forbidden, you have two options.

Option 1: Modify the File Label
Modify the file label so that NGINX (as a process labeled with the httpd_t context) can access the file:

# chcon -v --type=httpd_sys_content_t /www/t.txt
By default, this modification is deleted when the file system is relabeled. To make the change permanent, run:

# semanage fcontext -a -t httpd_sys_content_t /www/t.txt
# restorecon -v /www/t.txt
To modify file labels for groups of files, run:

# semanage fcontext -a -t httpd_sys_content_t /www(/.*)?
# restorecon -Rv /www
Option 2: Extend the httpd_t Domain Permissions
Extend the policy for httpd_t to allow access to additional file locations:

# grep nginx /var/log/audit/audit.log | audit2allow -m nginx > nginx.te
# cat nginx.te
 
module nginx 1.0;
 
require {
        type httpd_t;
        type default_t;
        type http_cache_port_t;
        class tcp_socket name_connect;
        class file { read getattr open };
}
 
#============= httpd_t ==============
allow httpd_t default_t:file { read getattr open };
 
#!!!! This avc can be allowed using one of these booleans:
#     httpd_can_network_relay, httpd_can_network_connect
allow httpd_t http_cache_port_t:tcp_socket name_connect;
To generate a compiled policy, include the -M option:

# grep nginx /var/log/audit/audit.log | audit2allow -M nginx
To load the policy, run semodule -i, then verify success with semodule -l:

# semodule -i nginx.pp
# semodule -l | grep nginx
nginx 1.0
This change persists across reboots.

Issue 3: NGINX Cannot Bind to Additional Ports
By default, the SELinux configuration does not allow NGINX to listen (bind()) to TCP or UDP ports other than the default ones that are whitelisted in the http_port_t type:

# semanage  port -l | grep http_port_t
 http_port_t                    tcp      80, 443, 488, 8008, 8009, 8443
If you try to configure NGINX to listen on a non‑whitelisted port (with the listen directive in the http, stream, or mail context in the NGINX configuration), you get an error when you verify (nginx -t) or reload the NGINX configuration, as indicated by this NGINX log entry:

YYYY/MM/DD hh:mm:ss [emerg] 46123#0: bind() to 0.0.0.0:8001 failed (13: Permission denied)
You can use semanage to add the desired port (here, 8001) to the http_port_t type:

# semanage port -a -t http_port_t -p tcp 8001
Reload NGINX with the new configuration.

# nginx -s reload