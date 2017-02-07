City-of-Bloomington.github.io
=============================

The City of Bloomington on Github

To make changes to this content, start with a clone of the repository:

    git clone https://github.com/City-of-Bloomington/City-of-Bloomington.github.io.git

Install Jekyll (either locally or on a VM):

https://github.com/City-of-Bloomington/system-playbooks/blob/master/roles/cob.jekyll/tasks/main.yml

> See here for instructions on using Ansible:
>
> https://github.com/City-of-Bloomington/system-playbooks

    cd /srv/system-playbooks
    ansible-playbook playbooks/jekyll.yml -i hosts

Using Jekyll
-----------------

Connect to the host that you installed Jekyll on. Start Jekyll.

    cd /srv/city-of-bloomington.github.io
    jekyll serve --host=0.0.0.0
    
For a new site structure, use:

    jekyll new [path]
    
(path should not exist yet)

From here you should be able to edit and update markdown files that make up the site.
