---
title: "Operation"
linkTitle: "Operation"
description: >
    How to operate a Merge portal
---


## Identity & User Management

A Merge portal has distinct concepts of identities and users. An identity
associates an email address to a set of access credentials. A user account
references an identity and includes the following

- A home directory on the Merge portal file system (MergeFS).
- A set of projects the user is a member of and is authorized to access.
- A set of certificate-based SSH keys for accessing XDCs and experiment nodes.

### Getting Identity Info

Portal administrators can list identities with

```
mrg list ids
```

which will show something like
```
USERNAME    EMAIL                      ADMIN
ry          ry@goo.com                 false
ops         ops@mergetb.example.net    true
```

### Creating Identities

A new identity can be created through the `mrg` command line tool.

```
mrg register <username> <email> <password>
```

This command is typically called directly by a user to register with a Merge
portal. It may also be used by administrators or project leaders with temporary
passwords for automatic onboarding of team members.

This will create an identity in the system. To create a new user from this
identity, an administrator can do the following.

```
mrg init <username>
```

### Getting User Info

Users can be listed as follows

```
mrg list users
```

Which will show something like

```
USERNAME    NAME    STATE     MODE      UID     GID
ry                  NotSet    Public    1000    1000
```

### Activating Users

When a user account is first created. It is not active, and must be activated by
an administrator.

```
mrg activate <username>
```

When a user is activate, you will see the following from `mrg list users`

```
USERNAME    NAME    STATE     MODE      UID     GID
ry                  Active    Public    1000    1000
```
