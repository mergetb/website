---
title: "Hello World"
linkTitle: "Hello World"
weight: 1
description: >
    A hello world experiment in Merge.
---

Welcome to your first guided tour of networked system experimentation in Merge.
Start by [installing the experimentation tools](/docs/experimentation/tools) on your
local system. Now let's dive in.

This guide assumes the username `murphy`, adjust for your username accordingly.
We also assume that you are using the reference portal at `mergetb.net`. If you
are using a different portal, substitute portal address references accordingly.

## Create an Experiment

To get started we'll create an experiment

```shell
mrg new experiment hello.murphy 'My first experiment'
```

Every experiment is a part of a project. In the command above, `hello.murphy`
follows the form `<experiment>.<project>`. Every user has a _personal project_
that is automatically created when they first join a Merge portal. The personal
project has the same name as the user. Here we are using `murphy`'s personal
project for the home of our `hello` experiment.

Dot delineated hierarchical naming is pervasive in the `mrg` command line tool,
all objects are referenced using this form.

## Assessing Experiment Status

Now that our experiment is created, we can see it in Merge in a few ways. The
most basic is asking merge for a list of experiments.

```shell
mrg list experiments
```
```
Name.Project    Description            Mode
------------    -----------            ----
hello.murphy    My first experiment    Public
```

You can use prefixes for any `mrg` command to avoid excessive typing. For
example, the last command can be executed as

```shell
mrg li e
```

More detailed information about the experiment is available through the `show`
command.

```shell
mrg show experiment hello.murphy
```
```
Repo: https://git.mergetb.net/murphy/hello
Mode: Public
Description: My first experiment
Members:
  Member    State     Role
  ------    -----     ----
  murphy    Active    Creator
```

In addition to the information we've already seen, this display shows us two new
things.

1. An address for a Merge-hosted Git repository for our experiment
   `https://git.mergetb.net/murphy/hello`
2. The experiment members associated with this experiment and what their role
   is.

## Pushing Experiment Source

Without any source, an experiment is just an empty shell. We add source by
pushing to the Git repository associated with our experiment. We identified this
repository in the previous step as `https://git.mergetb.net/murphy/hello`.

Let's start by grabbing an existing experiment from the official Merge examples
on GitLab.com.

```shell
git clone https://gitlab.com/mergetb/examples/hello-world
cd hello-world
```

This is a very basic hello world experiment with two nodes interconnected by a
link.

```python
from mergexp import *

# Create a netwok topology object.
net = Network('hello-world')

# Create two nodes.
a,b = [net.node(name) for name in ['a', 'b']]

# Create a link connecting the two nodes.
link = net.connect([a,b])

# Give IP addresses to the nodes on the link just created.
link[a].socket.addrs = ip4('10.0.0.1/24')
link[b].socket.addrs = ip4('10.0.0.2/24')

# Make this file a runnable experiment based on our two node topology.
experiment(net)
```

More details on this experiment can be found in the _Topology Modeling_ section
[here](/docs/hello-world).

**The important thing to note about source repositories is that there is a
requirement about where the experiment topology file lives:**

- For Python sources, the main topology file must be `model.py`
- For Go sources, the main topology file must be `model.go`

Now that we have the source in a local Git repository, let's push it to Merge.
Start by adding a new remote to the repository.
```
git remote add mergetb https://git.mergetb.net/murphy/hello
```

Before continuing we need to make sure we are ready to enter authentication
information. Only experiment or project members can push sources to Merge
experiment Git repos. Git authentication in Merge works through tokens, to
access your token do the following.

```
mrg whoami -t
```

This will display a glob of text, that is your access token. Now let's push some
source to our experiment.

```
git push -u mergetb
```

This will ask you for a username and password. **For the username enter the
token, leave the password blank**. If the push was successful, your experiment
now has source.

## Realizing and Experiment

The next step toward creating a working experiment is realization. Realization
is the act of finding a suitable set of resources for your experiment, and
allocating those resources for your experiment's exclusive use.

A realization is based on a specific `revision of an experiment`. Let's use
`git` to look at the latest revision for our experiment.

```
git log -1
```
```
commit 000cf5171f4248dfc71d222468a4abe83a6912df (HEAD -> master, origin/master, origin/HEAD)
Author: Ryan Goodfellow <rgoodfel@isi.edu>
Date:   Tue Dec 29 08:58:24 2020 -0800

    initial

    Signed-off-by: Ryan Goodfellow <rgoodfel@isi.edu>
```

Here we see the latest revision has the identifier
`000cf5171f4248dfc71d222468a4abe83a6912df`. So let's go ahead and create a
realization for that version of our source called `world`.

```
mrg realize world.hello.murphy 000cf5171f4248dfc71d222468a4abe83a6912df
```

## Assessing Realization Status

Now we can take a look at our realization in pretty much the same way as we did
for our experiment. Start by listing your realizations.

```
mrg list realizations
```
```
Realization           Nodes    Links    Status
-----------           -----    -----    ------
world.hello.murphy    2        1        Succeeded
```

This display tells us that we have 1 realization with two nodes and 1 link that
was successful. To find out more information, show the realization.

```
mrg show realization world.hello.murphy
```
```
Nodes:
a -> osprey0 (VirtualMachine)
b -> osprey0 (VirtualMachine)
Links:
a.0~b.0
  ENDPOINTS:
  WAYPOINTS:
```

This display shows us how our experiment was mapped onto testbed resources. In
this case both of our nodes were mapped to virtual machines running on the same
physical host named `osprey0`. We see a link that has no endpoints or waypoints,
which is expected in this case as both VMs reside on the same machine so there
is no actual physical link connecting them.

## Running Multiple Versions of the Same Experiment

Now let's say we have a slightly different version of this experiment in a
different branch of our repository. In this version of the experiment, we want
bare-metal nodes instead of virtual machines. Let's check out that branch and
take a look at the difference from our master branch.

```
git checkout metal
git diff master
```
```diff
diff --git a/model.py b/model.py
index 2e0b578..748c285 100644
--- a/model.py
+++ b/model.py
@@ -4,7 +4,7 @@ from mergexp import *
 net = Network('hello-world')

 # Create two nodes.
-a,b = [net.node(name) for name in ['a', 'b']]
+a,b = [net.node(name, metal==True) for name in ['a', 'b']]

 # Create a link connecting the two nodes.
 link = net.connect([a,b])
```

This shows us there is a single change, to specify the `metal` property for each
node. This ensures that we will get bare metal nodes for `a` and `b`. 

Let's push this branch to our experiment source repo.

```
git push -u mergetb metal
```

Now let's determine the latest commit in this branch,

```
git log -1
```
```
commit 8dd799637047e3bf845a5a16f44228d24d10f6ff (HEAD -> metal, origin/metal, mergetb/metal)
Author: Ryan Goodfellow <rgoodfel@isi.edu>
Date:   Thu Dec 31 13:35:03 2020 -0800
...
```
and create a realization from that revision of the source
```
mrg realize metal.hello.murphy 8dd799637047e3bf845a5a16f44228d24d10f6ff
```
which yields
```
mrg show realization metal.hello.murphy
```
```
Nodes:
a -> finch0 (BareMetal)
b -> finch1 (BareMetal)
Links:
a.0~b.0
  ENDPOINTS:
  [1000] a@finch0 &{Phy:name:"eno1"}
  [1000] b@finch1 &{Phy:name:"eno1"}
  WAYPOINTS:
  [1000] xfleaf0: &{Access:port:"swp0"  vid:47}
  [1000] xfleaf0: &{Access:port:"swp1"  vid:47}
```

This time we see that our experiment has mapped onto bare-metal nodes with
physical links interconnecting them.
