cloud-config(5) -- User data format used by `ucd(1)`
====================================================

## SYNOPSIS

`user-data`

`#cloud-config`

cloud-config data is provided by the cloud infrastructure to a cloud
instance. This data is parsed by `cloud-init(1)` which then configures
the cloud instance based on the information in the cloud-config.

## DESCRIPTION

The user-data as provided can be of several formats. `cloud-init(1)`
supports the `cloud-config` format, but also supports generic shell
scripts. If the user-data starts with `#!`, it is assumed that the
user-data is a generic shell script, and `cloud-init` will attempt
to execute the data as a script. If the user-data starts with the
`#cloud-config` stanza, it is assumed the user-data is in the
`cloud-config` format, described below.

The `cloud-config` format is used to structure configuration data
provided to the cloud instance. This data is structured in the YAML
format (`http://yaml.org`). Note that `cloud-init`(1) uses the
libyaml library, which supports YAML version 1.1.

In the YAML content, the root node (the document) contains a sequence of
directives. These directives inform cloud-init that the content of the
directive are processed by a directive-specific handler, or "module".

cloud-init includes many modules, each expecting a specifically constructed
content. This document lists the correct format and organization of each data
section for the supported modules, describes their effect, and states
various parameters for each option.

Table of implemented directives. Column 3 ("Ref") and column 4 ("CoreOS")
list whether the reference specification or the CoreOS implementation support
these directives. Note that implementation details may differ, due to the
underlying differences in Operating Systems.

```
Directive           |Description                          |Ref. |CoreOS.
--------------------|-------------------------------------|-----|-------
groups              |Configure user account groups        |yes  |no
package_upgrade     |Perform a software update            |yes  |no
packages            |Install additional software          |yes  |no
runcmd              |Execute system commands              |yes  |no
service             |Perform system service configuration |no   |no
ssh_authorized_keys |Provision SSH public keys            |yes  |yes
users               |Configure user accounts              |yes  |yes
write_files         |Write content to arbitrary files     |yes  |yes
hostname            |Define the system's hostname         |yes  |yes
envar               |Set environment variables            |no   |no
bootcmd             |Execute system commands on first boot|no   |no
```

## OPTIONS
For each of the directives listed in the table above, zero or more options may
be provided. The list below documents the implemented options per directive.
The "Type" field values are:

### TYPES

```
Type       |Description
-----------|-------------------------------------------------------------------
boolean    |Either true, false, yes, no, 0 or 1 (upper case and initial capitol
           |letter versions treated identical)
string     |A generic string encoded value. Can be multiline or YAML encoded
           |content
integer    |A string encoded decimal integer value
octal      |A string encoded octal integer value
hex        |A string encoded hexadecimal integer value
[]         |if a type is listed with [] appended, it means there may be zero or
           |more values, lists of values or any arbitrary depth of these nested
*          |Indicates this value isn't a separate key, but directly the
           |associated value of the root node. This is applicable for directives
           |that only have a single configuration parameter,
           |e.g. "package_upgrade"
```

### groups

```
Option    |Type     |Required    |Function
----------|---------|------------|-----------------------------------
name      |string[] |no          |Create a new user account group and
          |         |            |place listed user account names in
          |         |            |that group
```

### package_upgrade

```
Option    |Type     |Required    |Function
----------|---------|------------|-----------------------------------
*         |boolean  |yes         |Enables or disables whether software
          |         |            |update is performed
```

### packages

```
Option    |Type     |Required    |Function
----------|---------|------------|-----------------------------------
name      |string[] |no          |Enables installation of software bundles
```

### runcmd

```
Option    |Type     |Required    |Function
----------|---------|------------|-----------------------------------
*         |string[] |no          |Executes a command, if the item is a list,
          |         |            |the list will be converted to a string
          |         |            |and executed as a command line.
```

### hostname

```
Option    |Type     |Required    |Function
----------|---------|------------|-----------------------------------
*         |string   |yes         |Defines the system's hostname
```

### service

```
Option    |Type     |Required    |Function
----------|---------|------------|-----------------------------------
start     |string[] |no          |Start units
stop      |string[] |no          |Stop units
restart   |string[] |no          |Restart units
enable    |string[] |no          |Enable units to start automatically
disable   |string[] |no          |Disable units from starting automatically
reload    |string[] |no          |Reload service units
isolate   |string[] |no          |Change target to a new unit
mask      |string[] |no          |Prevent units from starting
unmask    |string[] |no          |Remove unit start prevention mask
```

### ssh_authorized_keys

```
Option    |Type     |Required    |Function
----------|---------|------------|-----------------------------------
*         |string[] |no          |Specifies an SSH public key value, as
          |         |            |string. This will be added to the default
          |         |            |user account's SSH configuration
```

### users

```
Option    |Type     |Required    |Function
----------|---------|------------|-----------------------------------
*         |[]       |no          |This directive expects a list of user
          |         |            |definitions. Each definition can
          |         |            |additionally specify the following
          |         |            |sub-options:
name      |string   |yes         |A user account name to be created
gecos     |string   |no          |A real user name, can be left empty
homedir   |string   |no          |A home directory path. Default under "/home"
primary-group|string|no          |A group name.
groups    |string   |no          |Comma-separated list of group names or
          |         |            |single group name. Specifies additional
          |         |            |groups to put this user account into
lock-passwd|boolean |no          |Lock the user account
inactive  |boolean  |no          |Mark the user account as inactive
passwd    |string   |no          |Password hash for the user account
no-create-home|boolean|no        |Omits creating a home directory
no-user-group|boolean|no         |Omits group creating for the account
no-log-init|boolean |no          |Omits this account from lastlog/faillog
expiredate|string   |no          |A date at which to expire the password
ssh-authorized-keys|string[]|no  |Add SSH public keys to ssh configuration
sudo      |string[] |no          |Add sudoers lines for this account, the account
          |         |            |name is automatically prepended
system    |boolean  |no          |Make the account a system account
```

### write_files

```
Option    |Type     |Required    |Function
----------|---------|------------|-----------------------------------
*         |[]       |no          |This directive expects a list of write_files
          |         |            |definitions. Each definition itself contains the
          |         |            |following options:
content   |string   |yes         |The content to write to a file
path      |string   |yes         |The full path and filename of the file to
          |         |            |be written out
owner     |string   |no          |Username and optionally group name, separated
          |         |            |by ":" or ".". Defaults to "root.root"
permissions|octal   |no          |Octal value describing the file permissions
          |         |            |default value is influenced according to
          |         |            |`umask`
```

## envar

```
Option    |Type     |Required    |Function
----------|---------|------------|-----------------------------------
*         |string   |yes         |Add environment variables to /etc/profile.d/cloud-init.sh
          |         |            |and to current cloud-init process
```

### bootcmd

```
Option    |Type     |Required    |Function
----------|---------|------------|-----------------------------------
*         |string[] |no          |Similar to runcmd but bootcmd will run only on first boot
```

## COPYRIGHT

* Copyright (C) 2017 Intel Corporation, License: CC-BY-SA-3.0

## SEE ALSO

`cloud-init`(1)

## NOTES

Creative Commons Attribution-ShareAlike 3.0 Unported

 * http://creativecommons.org/licenses/by-sa/3.0/

