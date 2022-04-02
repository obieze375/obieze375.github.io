#Variables 

Variables 

Just like any other Scripting or programming language we can use variable in ansible playbooks. Variables could store different values for different items. Variables help us to have shorter and more readable playbooks. Imagine we want to apply patches on hundreds of servers, the only thing we need is single playbook with some variables for all hundred servers! It's the variables that store information about different IP addresses, host names, username or passwords,... 

Naming Variables 

Variable names must start with a letter, and they can only contain letters, numbers, and underscores. The following table illustrates the difference between invalid and valid variable names.
INVALID VARIABLE NAMES
VALID VARIABLE NAMES
web server
web_server
remote.file
remote_file
1st file
file_1, file1
remoteserver$1
remote_server_1, remote_server1 

Defining Variables 

Variables can be defined in a variety of places in an Ansible project. However, this can be simplified to three basic scope levels:
	• Global scope: Variables set from the command line or Ansible configuration.
	• Play scope: Variables set in the play and related structures.
	• Host scope: Variables set on host groups and individual hosts by the inventory, fact gathering, or registered tasks.
If the same variable name is defined at more than one level, the level with the highest precedence wins. A narrow scope takes precedence over a wider scope: variables defined by the inventory are overridden by variables defined by the playbook, which are overridden by variables defined on the command line.

Host scope: We have already seen using variables when we talked about Ansible inventory files. As as example lets write down a playbook to configure multiple firewall configuration. We want to make it reusable for some one else to change ports, for that lets move variables to the inventory file:

#Sample inventory file with variables-inventory.txt 

centos http_port=8080 snmp_port=161-162 internal_ip_range=192.168.100.0

~~~~
# sample firewall playbook  firewall-playbook.yaml

-
  name: Set Firewall Configurations
  hosts: centos
  become: true
  tasks:
    -  firewalld:
         service: https
         permanent: true
         state: enabled

    -  firewalld:
         port: "{{ http_port }}/tcp"
         permanent: true
         state: disabled

    -  firewalld:
         port: "{{ snmp_port }}/udp"
         permanent: true
         state: disabled

    -  firewalld:
         source: "{{ internal_ip_range }}/24"
         permanent: true
         zone: internal
         state: enabled
~~~~

[user1@controller demo-var]$  ansible-playbook -i inventory.txt  firewall-playbook.yaml
![image](https://user-images.githubusercontent.com/91855277/160940861-101108af-1981-4bf9-a19b-971178e9d931.png)



#Conditionals 

Conditionals are used where one needs to run a specific step based on a condition.
 
~~~~
#Tsting 
- hosts: all 
   vars: 
      test1: "Hello Vivek" 
   tasks: 
      - name: Testing Ansible variable 
      debug: 
         msg: "Equals" 
         when: test1 == "Hello Vivek"

~~~~

#Conditions based on registered variables:
Often in a playbook you want to execute or skip a task based on the outcome of an earlier task. To create a conditional based on a registered variable:  

	1. Register the outcome of the earlier task as a variable.
	2. Create a conditional test based on the registered variable.You create the name of the registered variable using the register keyword. A registered variable always contains the status of the task that created it as well as any output that task generated. You can use registered variables in conditional , also it is possible to access the string contents of the registered variable : 



#sample playbook for conditionals  condition-playbook1.yaml
~~~~
- hosts: all
  become: yes

  tasks:
    - name: install apache2
      apt: name=apache2 state=latest
      ignore_errors: yes
      register: results

    - name: install httpd
      yum: name=httpd state=latest
      failed_when: "'FAILED' in results"
~~~~
![image](https://user-images.githubusercontent.com/91855277/160940756-01d21e9d-b999-488d-bd73-1b8a2367a970.png)

play scope: Ansible playbook supports defining the variable in two forms, Either a Single liner variable declaration like we do in any common programming languages or as a separate file with full of variables and values like a properties file. 

	• vars to define inline variables within the playbook
	• vars_files to import files with variables  
	
 Lets repeat previous example by moving variables in to the playbook ,It can be done with vars like this: 


---

#sample firewall playbook with vars firewall-playbook.yaml

play scope: Ansible playbook supports defining the variable in two forms, Either a Single liner variable declaration like we do in any common programming languages or as a separate file with full of variables and values like a properties file. 

	• vars to define inline variables within the playbook
	• vars_files to import files with variables  
	
 Lets repeat previous example by moving variables in to the playbook ,It can be done with vars like this: 


---

#sample firewall playbook with vars firewall-playbook.yaml
~~~~
-
  name: Set Firewall Configurations
  hosts: centos
  become: true
  vars:
    http_port: 8080
    snmp_port: 161-162
    internal_ip_range: 192.168.100.0
  tasks:
    -  firewalld:
         service: https
         permanent: true
         state: enabled

    -  firewalld:
         port: "{{ http_port }}/tcp"
         permanent: true
         state: disabled

    -  firewalld:
         port: "{{ snmp_port }}/udp"
         permanent: true
         state: disabled

    -  firewalld:
         source: "{{ internal_ip_range }}/24"
         permanent: true
         zone: internal
         state: enabled
~~~~
![image](https://user-images.githubusercontent.com/91855277/160941485-3637278b-8789-45fa-982f-f65cafc78dd7.png)
~~~~
-
  name: Set Firewall Configurations
  hosts: centos
  become: true
  vars:
    http_port: 8080
    snmp_port: 161-162
    internal_ip_range: 192.168.100.0
  tasks:
    -  firewalld:
         service: https
         permanent: true
         state: enabled

    -  firewalld:
         port: "{{ http_port }}/tcp"
         permanent: true
         state: disabled

    -  firewalld:
         port: "{{ snmp_port }}/udp"
         permanent: true
         state: disabled

    -  firewalld:
         source: "{{ internal_ip_range }}/24"
         permanent: true
         zone: internal
         state: enabled
~~~~

If you want to keep the variables in a separate file and import it with vars_filesYou have to first save the variables and values in the same format you have written in the playbook and the file can later be imported using vars_files like this: 

# vars.yaml 
~~~~
http_port: 8080
snmp_port: 161-162
internal_ip_range: 192.168.100.0
~~~~

---

#sample firewall playbook with var_files firewall-playbook.yaml
~~~~
-
  name: Set Firewall Configurations
  hosts: centos
  become: true
  vars_files:
    - vars.yaml
  tasks:
    -  firewalld:
         service: https
         permanent: true
         state: enabled

    -  firewalld:
         port: "{{ http_port }}/tcp"
         permanent: true
         state: disabled

    -  firewalld:
         port: "{{ snmp_port }}/udp"
         permanent: true
         state: disabled

    -  firewalld:
         source: "{{ internal_ip_range }}/24"
         permanent: true
         zone: internal
         state: enabled
~~~~
[user1@controller demo-var]$ ansible-playbook  firewall-playbook.yaml


Jinja2 templating : the format {{ }} we are using to use variables is called Jinja2 templating. Be careful about Quotes :
	• source: {{ http_port }}
	• source: "{{ http_port }}"
	• source: " Somthing {{ http_port }} Somthing "

![image](https://user-images.githubusercontent.com/91855277/160942597-44ed3504-a20b-4bd7-a629-6494e14624ae.png)

Variable in playbooks are very similar to using variables in any programming language. It helps you to use and assign a value to a variable and use that anywhere in the playbook. One can put conditions around the value of the variables and accordingly use them in the playbook.


Example 

~~~~
- hosts : <your hosts> 
vars:
tomcat_port : 8080 
~~~~
In the above example, we have defined a variable name tomcat_port and assigned the value 8080 to that variable and can use that in your playbook wherever needed.


Now taking a reference from the example shared. The following code is from one of the roles (install-tomcat) −

~~~~
block: 
   - name: Install Tomcat artifacts 
      action: > 
      yum name = "demo-tomcat-1" state = present 
      register: Output 
          
   always: 
      - debug: 
         msg: 
            - "Install Tomcat artifacts task ended with message: {{Output}}" 
            - "Installed Tomcat artifacts - {{Output.changed}}" 
~~~~
Here, the output is the variable used.


Let us walk through all the keywords used in the above code −


block − Ansible syntax to execute a given block.


name − Relevant name of the block - this is used in logging and helps in debugging that which all blocks were successfully executed.


action − The code next to action tag is the task to be executed. The action again is a Ansible keyword used in yaml.


register − The output of the action is registered using the register keyword and Output is the variable name which holds the action output.


always − Again a Ansible keyword , it states that below will always be executed.


msg − Displays the message.


Usage of variable - {{Output}}
This will read the value of variable Output. Also as it is used in the msg tab, it will print the value of the output variable.


Additionally, you can use the sub properties of the variable as well. Like in the case checking {{Output.changed}} whether the output got changed and accordingly use it.

![image](https://user-images.githubusercontent.com/91855277/160942665-b70e9e19-3b73-4f41-a935-dd0a4a95720f.png)

#Conditionals based on ansible_facts 

Often you want to execute or skip a task based on facts. As we mentioned before Facts are attributes of individual hosts, including IP address, operating system, the status of a filesystem, and many more. With conditionals based on facts: 

	• You can install a certain package only when the operating system is a particular version.
	• You can skip configuring a firewall on hosts with internal IP addresses.
	• You can perform cleanup tasks only when a filesystem is getting full.
	Not all facts exist for all hosts. For example, the ‘lsb_major_release’ fact only exists when the lsb_release package is installed on the target host. 
	
 In example below, we remove web server package from each server based on its os family:

---
#sample conditional with facts condition-playbook2.yaml
~~~~
- hosts: all
  become: yes

  tasks:

    - name: Remove Apache on Ubuntu Server
      apt: name=apache2 state=absent
      when: ansible_os_family == "Debian"

    - name: Remove Apache on CentOS  Server
      yum: name=httpd  state=absent
      when: ansible_os_family == "RedHat"
~~~~
![image](https://user-images.githubusercontent.com/91855277/160943363-847f5e5b-bb87-4490-9219-bb084da9ad20.png)

#Loops 

Below is the example to demonstrate the usage of Loops in Ansible.


The tasks is to copy the set of all the war files from one directory to tomcat webapps folder.


Most of the commands used in the example below are already covered before. Here, we will concentrate on the usage of loops.


Initially in the 'shell' command we have done ls *.war. So, it will list all the war files in the directory.


Output of that command is taken in a variable named output.


To loop, the 'with_items' syntax is being used.


with_items: "{{output.stdout_lines}}" --> output.stdout_lines gives us the line by line output and then we loop on the output with the with_items command of Ansible.


Attaching the example output just to make one understand how we used the stdout_lines in the with_items command.

~~~~
--- 
#Tsting 
- hosts: tomcat-node 
   tasks: 
      - name: Install Apache 
      shell: "ls *.war" 
      register: output 
      args: 
         chdir: /opt/ansible/tomcat/demo/webapps 
      
      - file: 
         src: '/opt/ansible/tomcat/demo/webapps/{{ item }}' 
         dest: '/users/demo/vivek/{{ item }}' 
         state: link 
      with_items: "{{output.stdout_lines}}"

~~~~


![image](https://user-images.githubusercontent.com/91855277/160943468-25cd8499-acdd-45d1-bdc8-83e072423216.png)

#Mutli-package Installation with loops

When automating server setup, sometimes you’ll need to repeat the execution of the same task using different values. For instance, you may need to install  multiple packages , or ... .  

---

#sample playbook install packages noloop-playbook.yaml

- hosts: ubuntu
  become: yes

  tasks:
    - yum: name=vim state=present
    - yum: name=nano state=present
    - yum: name=apache2 state=present

![image](https://user-images.githubusercontent.com/91855277/160943531-62e15fd3-0b09-4e55-83cb-49ed84968619.png)

#Loops with_items 

[user1@controller demo-var]$ cat loop-playbook1.yaml
---

# sample loop with with_itemp loop-playbook1.yaml

- hosts: ubuntu
  become: yes

  tasks:
   - name: install pakcages
     apt: name={{ item }} update_cache=yes state=latest
     with_items:
      - vim
      - nano
      - apache2


[user1@controller demo-var]$ ansible-playbook loop-playbook1.yaml

PLAY [ubuntu] *********************************************************************************

TASK [Gathering Facts] ************************************************************************
ok: [ubuntu]

TASK [install pakcages] ***********************************************************************
changed: [ubuntu] => (item=[u'vim', u'nano', u'apache2'])

PLAY RECAP ************************************************************************************
ubuntu                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


[user1@controller demo-var]$ ansible-playbook loop-playbook1.yaml

PLAY [ubuntu] ***************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [ubuntu]

TASK [install pakcages] *****************************************************************************************************************
ok: [ubuntu] => (item=[u'vim', u'nano', u'apache2'])

PLAY RECAP ******************************************************************************************************************************
ubuntu                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

![image](https://user-images.githubusercontent.com/91855277/160943830-3fa1fa45-58b3-42ce-b535-e03da03e27ca.png)

#Loops with_file 

---

# sample loop with with_file loop-playbook2.yaml

- hosts: ubuntu
  become: yes

  tasks:
   - name: show file(s) contents
     debug: msg={{ item }}
     with_file:
      - myfile1.txt
      - myfile2.txt 


[user1@controller demo-var]$ cat myfile1.txt
This is myfile1.txt, first line :-)
[user1@controller demo-var]$
[user1@controller demo-var]$ cat myfile2.txt
This is myfile2.txt, first line :-0


[user1@controller demo-var]$ ansible-playbook loop-playbook2.yaml

PLAY [ubuntu] ***************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************
ok: [ubuntu]

TASK [show file(s) contents] ************************************************************************************************************
ok: [ubuntu] => (item=This is myfile1.txt, first line :-)) => {
    "msg": "This is myfile1.txt, first line :-)"
}
ok: [ubuntu] => (item=This is myfile2.txt, first line :-0) => {
    "msg": "This is myfile2.txt, first line :-0"
}

PLAY RECAP ******************************************************************************************************************************
ubuntu                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

![image](https://user-images.githubusercontent.com/91855277/160943893-c742635b-8f43-4673-a997-410c7de8e6f8.png)