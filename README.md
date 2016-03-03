# Sample playbooks for Ansible 2.0

## Overview

In blog post [Ansible 2.0 Has Arrived](https://www.ansible.com/blog/ansible-2.0-launch), following two problems with Ansible 2.0 were shown. This repository provides sample playbooks for confirming these problems.

* Dynamic Include Problems
    * Tags on tasks are not seen until the include is processed, so tags should now be specified on the include task rather than on individual tasks within the include, otherwise they will not be seen. Likewise, the --list-tags option will not show tags contained only in the include files.
    * Handlers in includes will not be seen when a task attempts to notify them, so handlers should avoid using includes at this time.
* Missing Tags
    * Ansible 2.0 does not currently raise an error if a non-existent tag is specified via --tags or --skip-tags.  This is also related to the dynamic include problem above, and we intend to address this once the above include problems are resolved.

## Playbooks

| Playbook      | tags for tasks                        | handlers                    | 
|:--------------|:--------------------------------------|:----------------------------|
| playbook1.yml | on include task in tasks/main.yml     | in handlers/main.yml        |
| playbook2.yml | on individual tasks in included files | in handlers/main.yml        |
| playbook3.yml | on include task in tasks/main.yml     | in included file            |

## Results

I executed these playbooks by Ansible 1.9.4 and 2.0.1 as follows.

* --list-tags
    * `$ ansible-playbook playbook1.yml -c local -i inventory --list-tags`
* running tasks without tag
    * `$ ansible-playbook playbook1.yml -c local -i inventory`
* running tasks with tag "application1"
    * `$ ansible-playbook playbook1.yml -c local -i inventory -t application1`

The following table shows the results.

| Ansible version and playbook  | --list-tags                  | running tasks without tag         | running tasks with tag "application1" | 
|:------------------------------|:-----------------------------|:--------------------|:------------------------|
| Ansible 1.9.4 & playbook1.yml | [application1, application2] | ok                  | ok                      |
| Ansible 1.9.4 & playbook2.yml | [application1, application2] | ok                  | ok                      |
| Ansible 1.9.4 & playbook3.yml | [application1, application2] | ok                  | ok                      |
| Ansible 2.0.1 & playbook1.yml | []                           | ok                  | ok                      |
| Ansible 2.0.1 & playbook2.yml | [application1, application2] | ok                  | ok                      |
| Ansible 2.0.1 & playbook3.yml | []                           | handler was ignored | handler was ignored     |

The tags on individual tasks in the included file were not ignored when running tasks. On the other hand, the handlers in the included file were silently ignored.

## Conclusion

When migrating from Ansible 1.9 to Ansible 2.0,

* We do not have to avoid using "tags" in included task files if we do not use "--list-tags" option.
* We have to avoid using "include" in handler files.
