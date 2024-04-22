Instead of putting raw bash commands we can write a reusable Ansible role invoke it from the playbook that will be used inside Docker container to provision it.

This is how I do it

```
FROM debian:9

# Bootstrap Ansible via pip
RUN apt-get update && apt-get install -y wget gcc make python python-dev python-setuptools python-pip libffi-dev libssl-dev libyaml-dev
RUN pip install -U pip
RUN pip install -U ansible

# Prepare Ansible environment
RUN mkdir /ansible
COPY . /ansible
ENV ANSIBLE_ROLES_PATH /ansible/roles
ENV ANSIBLE_VAULT_PASSWORD_FILE /ansible/.vaultpass

# Launch Ansible playbook from inside container
RUN cd /ansible && ansible-playbook -c local -v mycontainer.yml

# Cleanup
RUN rm -rf /ansible
RUN for dep in $(pip show ansible | grep Requires | sed 's/Requires: //g; s/,//g'); do pip uninstall -y $dep; done
RUN apt-get purge -y python-dev python-pip
RUN apt-get autoremove -y && apt-get autoclean -y && apt-get clean -y
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp* /usr/share/doc/*

# Environment setup
ENV HOME /home/test
WORKDIR /
USER test

CMD ["/bin/bash"]
```

Drop this Dockerfile to the root of your Ansible repo and it will build Docker image using your playbooks, roles, inventory and vault secrets.

It works, it’s reusable, e.g. I have some base roles that applied for docker container and on bare metal machines, provisioning is easier to maintain in Ansible. But still, it feels awkward.
