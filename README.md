NetApp Cookbook
===============

The NetApp cookbook manages Clustered Data ONTAP clusters using the NetApp Manageability SDK. Both cluster-wide and Storage Virtual Machine (SVM, formerly known as Vservers) specific operations are supported.

The NetApp cookbook resources can be used to manage Cloud ONTAP instances in Amazon Web Services.

Requirements
------------
#### NetApp Manageability SDK Library v5.0

You can download it from [NetApp](http://mysupport.netapp.com/NOW/cgi-bin/software?product=NetApp+Manageability+SDK&platform=All+Platforms) after you have created an account on [NetApp NOW](https://support.netapp.com/eservice/public/now.do)

- Save the NetApp SDK to this NetApp cookbook in the "libraries" dir.

- Update the NaServer.rb to specify the path of NaElement. Replace the line:
` require NaElement
`
With -
` require File.dirname(__FILE__) + "/NaElement"
`

NetApp connection
-----------------
The ZAPI connection is made over HTTP or HTTPS, with a user account that exists on the NetApp storage cluster. If you specify an account that only has SVM administration privileges (rather than cluster administration privileges), some features of the NetApp cookbook will not work. The connection settings are managed by attributes in the cookbook but are also exposed in Common attributes for the NetApp resources.

    ['netapp']['url'] = 'https://root:secret@pfiler01.example.com/svm01'

or

    ['netapp']['https'] boolean, default is 'true'.
    ['netapp']['user'] string
    ['netapp']['password'] string
    ['netapp']['fqdn'] string
    ['netapp']['vserver'] string
    ['netapp']['asup'] boolean, default is 'true'.

The ASUP option, if set to 'true', will cause a log message to be sent to the storage cluster. This log message will be included in ASUP bundles that are sent back to NetApp, if configured to do so on the system. If ASUP is not enabled on the system or on the attribute listed above, no log message will be sent to NetApp.

NetApp Resources
================

Common Attributes
-----------------
In addition to those provided by Chef itself (`ignore_failure`, `retries`, `retry_delay`, etc.), the connection attribute(s) are exposed all NetApp Resources even though they are typically set by attributes.

Common Actions
--------------
The `:nothing` action is provided by Chef for all Resources for use with notifications and subscriptions.

netapp_user
-----------
Cluster management of user creation, modification and deletion.

### Actions ###
This resource has the following actions:

* `:create` Default.
* `:delete` Removes the user

### Attributes ###
This resource has the following attributes:

* `name` User name. Required
* `password` Required for non-snmp users
* `application` Name of the application. Possible values: 'console', 'http', 'ontapi', 'rsh', 'snmp', 'sp', 'ssh', 'telnet'
* `comment`
* `role` Array of roles
* `snmpv3-login-info` SNMPv3 user login information for 'usm' authentication method
* `vserver` Name of vserver
* `authentication` Authentication method for the application. Possible values: 'community', 'password', 'publickey', 'domain', 'nsswitch' and 'usm'

### Example ###

````ruby
netapp_user "clogeny" do
  vserver "my-vserver"
  role "admin"
  application "ontapi"
  authentication "password"
  password "my-password1"
  action :create
end
````

````ruby
netapp_user "clogeny" do
  vserver "my-vserver"
  application "ontapi"
  authentication "password"
  action :delete
end
````

netapp_group
------------
Cluster management of group creation, modification and deletion.

### Actions ###
This resource has the following actions:

* `:create` Default.
* `:delete` Removes the group

### Attributes ###
This resource has the following attributes:

* `name` string, name attribute. Required
* `comment` string.
* `roles` Array of roles for this group.

### Example ###

````ruby
netapp_group 'admins' do
  comments 'keep the trains on time'
  roles ['security']
  action :create
end
````

````ruby
netapp_group 'read-only' do
  action :delete
end
````

netapp_role
-----------
Cluster management of role creation, modification and deletion.

netapp_role resource can be used to manage roles on the Cloud ONTAP instances as well. 

### Actions ###
This resource has the following actions:

* `:create` Default.
* `:delete` Removes the role

### Attributes ###
This resource has the following attributes:

* `name` Name attribute. Required
* `svm` Name of vserver. Required
* `command_directory` The command or command directory to which the role has an access. Required
* `access_level` Access level for the role. Possible values: 'none', 'readonly', 'all'. The default value is 'all'.
* `return_record` If set to true, returns the security login role on successful creation. Default: false
* `role_query` Example: The command is 'volume show' and the query is '-volume vol1'

### Example ###

````ruby
netapp_role 'security' do
  svm 'my-vserver'
  command_directory 'volume'
  action :create
end
````

````ruby
netapp_role 'superusers' do
  svm 'my-vserver'
  command_directory 'DEFAULT'
  action :delete
end
````

netapp_feature
--------------
Cluster management of NetApp features by license. See API docs for "license-v2".

### Actions ###
This resource has the following action:

* `:enable` Default. Ensures the NetApp provides this feature.

### Attributes ###
This resource has the following attributes:

* `codes` Array, license code when adding a package. 24 or 48 uppercase alpha only characters.

### Example ###

````ruby
netapp_feature 'iscsi' do
  codes ['ABCDEFGHIJKLMNOPQRSTUVWX']
  action :enable
end
````

netapp_svm
----------
Cluster-level management of a data Storage Virtual Machines (SVMs). SVM-level management is done through other resources. After the cluster setup, a cluster administrator must create data SVMs and add volumes to these SVMs to facilitate data access from the cluster. A cluster must have at least one data SVM to serve data to its clients.

### Actions ###
This resource has the following actions:

* `:create` Default.
* `:delete` Removes the svm

### Attributes ###
This resource has the following attributes:

* `name` name attribute. Required. SVM names can contain a period (.), a hyphen (-), or an underscore (_), but must not start with a hyphen, period, or number. The maximum number of characters allowed in SVM names is 47.
* `nsswitch` Required.
* `volume` Required
* `aggregate` Required. Aggregate on which you want to create the root volume for the SVM. The default aggregate name is used if you do not specify one.
* `security` Required. Determines the type of permissions that can be used to control data access to a volume. Default is `unix`.
* `comment`
* `is_repository_vserver`
* `language` If you do not specify the language, the default language `C.UTF-8` or `POSIX.UTF-8` is used.???
* `nmswitch`
* `quota_policy`
* `return_record`
* `snapshot_policy`

### Example ###

````ruby
netapp_svm "example-svm" do
  security "unix"
  aggregate "aggr1"
  volume "vol1"
  nsswitch ["nis"]
  action :create
end
````

netapp_volume
-------------
SVM-management of volume creation, modification and deletion including auto-increment, snapshot schedules and volume options.

netapp_volume resource provisions additional volume on the Cloud ONTAP instances. It Creates the volume on an existing aggregate that has sufficient free space.

### Actions ###
This resource has the following actions:

* `:create` Default.
* `:delete` Removes the volume

### Attributes ###
This resource has the following attributes:

* `name` string, name attribute. Volume name. Required.
* `svm` string. Name of managed SVM. Required
* `aggregate` string. Required
* `size` string (1-9kmgt). Required

### Example ###

````ruby
netapp_volume '/foo' do
  svm 'vs1.example.com'
  aggregate 'aggr1'
  size '5t'
  action :create
end
````

````ruby
netapp_volume 'bar' do
  action :delete
end
````

netapp_lif
----------
SVM-management of logical interface (LIF) creation, modification and deletion.

### Actions ###
This resource has the following actions:

* `:create` Default. Ensures the lif is in this state.
* `:delete` Removes the lif

### Attributes ###
This resource has the following attributes:

* `name` name attribute. LIF name. Required
* `svm` Name of managed SVM. Required
* `address`
* `administrative_status` valid values "up", "down", "unknown"
* `comment`
* `data_protocols`
* `dns_domain_name`
* `failover_group`
* `failover_policy` valid values "nextavail", "priority", "disabled"
* `firewall_policy`
* `home_node`
* `home_port`
* `is_auto_revert`
* `is_ipv4_link_local`
* `listen_for_dns_query`
* `netmask`
* `netmask_length`
* `return_record`
* `role` valid values "undef", "cluster", "data", "node_mgmt", "intercluster", "cluster_mgmt"
* `routing_group_name`
* `use_failover_group` valid values "system_defined", "disabled", "enabled"
*

### Example ###

````ruby
netapp_lif 'private' do
  svm 'vs1.example.com'
  action :create
end
````

````ruby
netapp_lif 'public' do
  action :delete
end
````

netapp_iscsi
----------
SVM-management of iSCSI target creation, modification and deletion.

### Actions ###
This resource has the following actions:

* `:create` Default. Creates iSCSI service.
* `:delete` Removes the target

### Attributes ###
This resource has the following attributes:

* `svm` Name of managed SVM. Required
* `alias`
* `node`
* `start` True or False. True by default.

### Example ###

````ruby
netapp_iscsi 'foo' do
  svm 'vs1.example.com'
  action :create
end
````

````ruby
netapp_iscsi 'bar' do
  action :delete
end
````

netapp_nfs
----------
SVM-management of NFS export rule creation, modification and deletion including NFS export security. Rule changes are persistent.

You do not need to enter any information to configure NFS on the SVM. The NFS configuration is created when you specify the protocol value as `nfs`.

### Actions ###
This resource has the following actions:

* `:create` Default. Ensures the NFS export is in this state.
* `:delete` Removes the NFS export

### Attributes ###
This resource has the following attributes:
* `pathname` string, name attribute. Required
* `svm` string. Name of managed SVM. Required
* `security_rules` hash. Access block information for lists of hosts.

### Example ###

````ruby
netapp_nfs '/vol/vol0' do
  svm 'vs1.example.com'
  action :create
end
````

````ruby
netapp_export '/vol/vol1' do
  svm 'vs1.example.com'
  action :delete
end
````

netapp_qtree
------------
SVM-management of qtree creation, modification and deletion. Qtrees are a special subdirectory of the root of a volume that acts as a virtual subvolume with special attributes.

netapp_qtree resource can be used to create logically defined file system on Cloud ONTAP instances.

### Actions ###
This resource has the following actions:

* `:create` Default. Ensures the QTree is in this state.
* `:delete` Removes the QTree

### Attributes ###
This resource has the following attributes:

* `name` name attribute. The path of the qtree, relative to the volume. Required
* `svm` Name of managed SVM. Required
* `volume` Name of the volume on which to create the qtree. Required.
* `export_policy` Export policy of the qtree. If this input is not specified, the qtree will inherit the export policy of the parent volume.
* `mode` The file permission bits of the qtree, similar to UNIX permission bits. If this argument is missing, the permissions of the volume is used.
* `oplocks` Opportunistic locks mode of the qtree. Possible values: "enabled", "disabled". Default value is the oplock mode of the volume.
* `security` Security style of the qtree. Possible values: "unix", "ntfs", or "mixed". Default value is the security style of the volume.
* `force` True or false

### Example ###

````ruby
netapp_qtree '/share' do
  svm 'vs1.example.com'
  volume '/foo'
  action :create
end
````

````ruby
netapp_role '/bar' do
  svm 'vs1.example.com'
  volume '/foo'
  action :delete
end
````



Contributing
------------
1. Fork the repository on Github
2. Create a named feature branch (like `add_component_x`)
3. Write your change
4. Write tests for your change (if applicable)
5. Run the tests, ensuring they all pass
6. Submit a Pull Request using Github

License and Authors
-------------------
- Authors:: Arjun Hariharan (Arjun.Hariharan@Clogeny.com)

```text
Copyright 2014 Chef Software, Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
