---
layout: post
title:  "The Ansible variable problem, and how we solved it."
date:   2017-06-29 18:06:00 +0100
categories: blog
---

### Introduction
So [Ansible](https://www.ansible.com) is at the time of writing, my favorite CM
tool out there.  It's agent-less by default, quick to learn, quick to test, quick
to fail (more on that later) and probably most importantly, gives me the most return
on my investment over any other CM tool.  It uses basic SSH & Python (or winrm)
to connect and modify the servers (python 2.7 is standard on all but the most
niche linux systems today) and is therefore incredibly easy to get going, no agents
to install, no SSL certs to manage.

My most recent assignment at work has involved introducing Ansible to configure and
deploy to a mass of servers (ballpark figure around 100).  As the site was not using
Ansible already we had to develop a way of managing variables in a half way decent
manner which wasn't going to quickly become a burden!  

### The problem
We need to be able to provide sane defaults for the ultimately hundereds of users
which are going to contribute to the code base.  We need a good variable system
where we can define variables that are side-wide (default usernames & passwords)
environment specific (LDAP Host for the environment) and then down to os/service
specific variables so that when a playbook is ran, we have a predictable variable
resolution order and when someone is creating a new variable in their day to day
operations which may be of use to others, they have a clear place to put it.

I am aware I am not the first (and probably not the last) person to face this
issue, anyone using Ansible in anger on large estates will invariably come up
against this challenge sooner or later.  However I thought I would like to first
think of a solution without consulting the almighty google in the first instance.  

### Hieradata (puppets solution to this)
I will admit there is one element of puppet I really do like, hiera.  This is a
hierarchal variable tree which can be referenced in your puppet manifests at will
and can hierarchically search bottom to top for a variable and as you can imagine,
this can trivialise the management of multiple servers running in different
locations / timezones / environments / etc.  Unfortunately Ansible provide no such
neat little tool to manage variables and data so therefore we need to see what
Ansible do provide and work with that.

Before anyone comments saying "Why not just use hiera?".  I did think about this
however the client in question has a 6 week lead time (minimum) on approving
new software for deployment onto it's estate (even in dev).  This made that route
less preferable over trying to come up with something "Ansible native".  I would
have gone for hiera if a) the client accepted it and b) Ansible could use it.

### Variable precedence
In the [Ansible Docs](http://docs.ansible.com/ansible/playbooks_variables.html)
they do give us a decent idea of which files / locations have the  highest order
of precedence when a task is looking for variables (it's essentially the load
order, last loaded is the one you get).  So we can use this to our advantage...

### The host inventory and :children
This is where we made the most headway in solving our issue, consider the following:

We have two sites and multiple environments within each one of them, for arguments
sake lets call one site `jupiter` and the other `pluto` and in each of those sites
three environments: `dev`, `pre-prod` and `prod`.  

Each environment has it's own set of servers and even then the servers have different
responsibilities or "roles" as we call them.  One may be a database, one a webserver
and so on.  The feature we found which cracked this was that you can group together
groups of servers int other groups with ansible, so you can have a `jupiter` group
which has a list of all of the environment groups in there `jupiter-dev` for e.g.
Then in the `jupiter-dev` group more groups to further break down the environment
into whatever groups you need `jupiter-dev-dbservers` or `jupiter-dev-frontend`
for example.

### Some testing
Before I carved up the group_vars folder we had (it had a few loose yaml files)
I thought it woud be prudent to do some local laptop testing first.  So lets
create some structure for this:

```
group_vars/
           jupiter/
                   vars.yaml
                   vault.yaml
           jupiter-dev/
                   vars.yaml
                   vault.yaml
           jupiter-dev-dbservers
                   vars.yaml
                   vault.yaml
```
I have left off the second site & other environments for the sake of brevity,
however you get the picture, a folder for the group, then two yaml files within each, one for encrypted variables (passwords, authentication tokens, etc) and one for plain
text variables (hostnames, install paths etc).

Next up we need to create a hosts file for Ansible to use in conjunction with
this directory layout.  We went for something like this:

```
[jupiter-dev-dbservers]
dbserver_1.jupiter.dev
dbserver_2.jupiter.dev

[jupiter-dev-frontend]
fe_svr01.jupiter.dev
fe_svr02.jupiter.dev

[jupiter-preprod-dbservers]
dbserver_1.jupiter.net
dbserver_1.jupiter.net
dbserver_1.jupiter.net

[jupiter-preprod-frontend]
fe_svr01.jupiter.dev
fe_svr02.jupiter.dev

[jupiter-dev:children]
jupiter-dev-dbservers
jupiter-dev-frontend

[jupiter-preprod:children]
jupiter-preprod-dbservers
jupiter-preprod-frontend

[jupiter:children]
jupiter-dev
jupiter-preprod
```

With these sorted out all thats left to do is populate some of the variable files
with variables which have the same name and ensure that the group_vars children
resolution order works, consider the following variable in `jupiter/vars.yaml`:

```
ldap_hostname: ldap.jupiter.net
```

And defined again in `jupiter-dev/vars.yaml`:

```
ldap_hostname: ldap.jupiter.dev
```

Now we need to test to ensure that when a server which belongs to any group in
jupiter-dev it picks up the correct ldap hostname to talk to.  We can do this
by writing a real quick and dirty play:

{%highlight yaml %}
---
- name: Test variable load order
  hosts:
    - jupiter-dev-dbservers
  tasks:
    - name: Shout the ldap_host
      debug:
        msg: '{{site.ldap_hostname}}'
{%endhighlight %}

This will allow us to test by adding and removing variables from the group_vars
and perform an Ansible run, our desired behaviour is that the resolution order
should be as follows (This list is a cut & paste from Ansibles website):

* _role defaults_
* inventory file or script group vars
* _inventory group_vars/all_
* playbook group_vars/all
* _inventory group_vars/*_
* playbook group_vars/*
* inventory file or script host vars
* inventory host_vars/*
* playbook host_vars/*
* _host facts_
* _play vars_
* play vars_prompt
* play vars_files
* role vars (defined in role/vars/main.yml)
* block vars (only for tasks in block)
* task vars (only for the task)
* _role (and include_role) params_
* include params
* include_vars
* set_facts / registered vars
* extra vars (always win precedence)

So now we have set it up, what we are looking for is for it to return the value
`ldap.jupiter.dev`.  Let's test!

```
Bens-MacBook-Pro-2:ansible ben$ ansible-playbook test.yaml

PLAY [Test variable load order] ************************************************

TASK [setup] *******************************************************************
ok: [192.168.10.97]

TASK [Shout the ldap_host] *****************************************************
ok: [192.168.10.97] => {
    "msg": "ldap.jupiter.dev"
}

PLAY RECAP *********************************************************************
192.168.10.97              : ok=2    changed=0    unreachable=0    failed=0   

```
Happy days!  The most specific variable for the group has been loaded!  Winning!

### Conclusion
I think this has turned out quite well, we now have a clear set of group_vars
which we can easily add site-wide, environment-specific and all the way down to
individual groups of hosts.  We can also specify individual host variables (if
needed) using the standard host_vars folder (not described here as its standard)
Ansible fanfare.  I know an entire blog on the management of variables is probably
overkill and if anyone is left reading this, I commend you!  But this is more for
my own records as I will doubtless be repeating this pattern in my own Ansible
repositories and projects.  As always any errors I have made or comments just
come find me on twitter.
