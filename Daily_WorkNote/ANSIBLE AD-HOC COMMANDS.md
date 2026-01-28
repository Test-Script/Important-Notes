========================================================================================================================
                                                ANSIBLE AD-HOC COMMANDS
========================================================================================================================

BASIC SYNTAX

ansible <host-pattern> -m <module> -a "<arguments>" [options]

INVENTORY TARGETING

ansible all -m ping
ansible webservers -m ping
ansible db01 -m ping
ansible 'prod*' -m ping

1. CONNECTIVITY CHECK

ansible all -m ping

ansible all -m ping -f 20

2. RUN SHELL / COMMAND

ansible all -m command -a "uptime"

ansible all -m shell -a "df -h | grep /dev"

3. CHECK DISK / MEMORY

ansible all -m shell -a "df -h"
ansible all -m shell -a "free -m"

4. PACKAGE MANAGEMENT

Install package:

    ansible all -m yum -a "name=httpd state=present" -b

Remove package:

    ansible all -m yum -a "name=httpd state=absent" -b

Ubuntu:

    ansible all -m apt -a "name=nginx state=present update_cache=yes" -b

5. SERVICE MANAGEMENT

Start service:

    ansible all -m service -a "name=httpd state=started" -b

Restart service:

    ansible all -m service -a "name=nginx state=restarted" -b

6. FILE & DIRECTORY OPERATIONS

Create directory:

    ansible all -m file -a "path=/opt/app state=directory mode=0755" -b

Delete file:

    ansible all -m file -a "path=/tmp/test.txt state=absent" -b

7. COPY FILES

    ansible all -m copy -a "src=/etc/hosts dest=/tmp/hosts" -b

8. USER MANAGEMENT

Create user:

    ansible all -m user -a "name=devuser state=present" -b

Delete user:

    ansible all -m user -a "name=devuser state=absent" -b