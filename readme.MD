
## Sample Ansible Playbooks for Cisco Wireless Controller Discovery ##

The playbooks contained within this repository are examples only.  They are intended to illustrate some basic usage as well as the evolution of a playbook from 
- an initial playbook with embedded credentials suitable for a lab environment
- a playbook more suitable for a production environment which uses constructs which anticipate the structured date returned and how it might be used at a later time
- a playbook more suitable for a production environment which uses Ansible Vault to protect credentials 
- an interactive playbook to showcase including credentials as part of the command line and limiting the target hosts

These playbooks utilize the [**aireos_command**](https://docs.ansible.com/ansible/2.4/aireos_command_module.html) module which to date still requires the provider construct.  


### Ansible Vault ###

The wlc_show_json_vault.yml playbook incorporates an include file which is an encrypted file containing the required user name and password.  The sample creds.yml file was encrypted using the 'ansible-vault encrypt <filename>' command and given then ansible vault password of 'Cisco123'.  

The only modifications to the original json playbook are:

An additional task which include the new encrypted creds.yml file (which contains the variables v_username and v_password).
An update to the username and password arguments which now reference the v_ variables rather than the actual values.

```
  tasks:
    - name: Get Vault Variables
      include_vars: ./group_vars/creds.yml
    - name: Execute Show Commands on WLCs
      aireos_command:
        username: "{{ v_username }}"
        password: "{{ v_password }}"
```

EXECUTION:
```ansible-playbook -i hosts wlc_show_json_vault.yml --ask-vault-pass```


#### Vault Example ####

```
root@0bd4731cb725:/ansible/cisco_wlc# ls
ansible.cfg  ansible.log  group_vars  hosts  labv.yml  logs  readme.MD  wlc_show_interactive.yml  wlc_show_json.yml  
wlc_show_json_vault.yml  wlc_show_lab.yml

root@0bd4731cb725:/ansible/cisco_wlc# cd group_vars/

root@0bd4731cb725:/ansible/cisco_wlc/group_vars# ls
creds.yml

root@0bd4731cb725:/ansible/cisco_wlc/group_vars# cat creds.yml
$ANSIBLE_VAULT;1.1;AES256
32336235323861313265393432343839613631643066666134643765626632623532396434376234
6431336535666536366436316336326335626463316665390a323536313631363032656135313761
66643239306261653666393236323361646431653737376464346364656139633333623435353431
3833333261386565300a356439373437623161653362316163343734326536356662313065616234
35323466383963326138663463653239343332333364616534653933396535633962333065386232
3866346332626631633537313338626264333331363962663538

root@0bd4731cb725:/ansible/cisco_wlc/group_vars# ansible-vault view creds.yml
Vault password:
v_username: admin
v_password: Cisco123
root@0bd4731cb725:/ansible/cisco_wlc/group_vars# history | grep ansible-vault
  204  ansible-vault encrypt creds.yml
  205  ansible-vault view creds.yml

```


### Environment ##

The lab environment used for development and initial testing included a Cisco Virtual Wireless Controller hosted on VMWare Workstation.

The Ansible Control System was hosted on a Docker Cotainer for Ansible 2.4 (upgraded to Ansible 2.6 using the pip install command below)


```
 pip install ansible==2.6.8.0
```

##### Docker Container #####

https://hub.docker.com/r/cldeluna/xenial-ansible/





```
root@0bd4731cb725:/ansible/cisco_wlc# ansible --version
ansible 2.6.8
  config file = /ansible/cisco_wlc/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.12 (default, Nov 19 2016, 06:48:10) [GCC 5.4.0 20160609]
root@0bd4731cb725:/ansible/cisco_wlc#
```

##### Considerations ######

As of Ansible 2.5 significant changes were introduced to some well established methods (provider, username, password) for connecting to network devices.  In short, these [new methods](https://docs.ansible.com/ansible/latest/porting_guides/porting_guide_2.5.html#adding-persistent-connection-types-network-cli-and-netconf) allow for consistency in connecting to any type of target be it server or network device.  This is good for the many, to be sure, but will result in rework for those of us who have older playbooks.  We will also have to keep an eye out on our favorite modules to ascertain when they can support the new constructs.


[Ansible 2.5 Porting Guide](https://docs.ansible.com/ansible/latest/porting_guides/porting_guide_2.5.html)

>Top-level connection arguments will be removed in 2.9

>Top-level connection arguments like username, host, and password are deprecated and will be removed in version 2.9.

[IOS Platform Options](https://docs.ansible.com/ansible/latest/network/user_guide/platform_ios.html#using-cli-in-ansible-2-6)

##### Disclaimer ######

*THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NON INFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.*