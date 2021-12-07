---
author: Dennis Kruyt
categories:
- ansible
- cicd
- pipeline
- flow
- git
- merge
- cd
- gitlab
- ci
date: "2020-01-17T11:44:48Z"
description: ""
draft: false
cover:
  image: /images/2020/10/photo-1567789884554-0b844b597180.jpeg
slug: ansible-ci-with-gitlab
tags:
- ansible
- cicd
- pipeline
- flow
- git
- merge
- cd
- gitlab
- ci
title: Ansible CICD pipeline with GitLab
---


In this post I will show how I use GitLab CICD with Ansible. I'll show the pipelines and how the merge requests are handled for full control and auditing.

## GitLab flow

The GitLab flow is as following, you have one or more dev/working branches. To push code to the staging environment you do this via an merge request in GitLab, en to get this code in to production. Again via a merge request from the master branch to the production branch. This way we have automation but also control and auditing via merge request.

{{< figure src="/images/2020/01/ansibleflow.png" caption="Ansible GitLab flow" >}}

This can be be adjusted to suit environments and policies you need.  It will not matter if you have a test and acceptance environment, or use tags, the principle will be the same.

## GitLab CICD Pipeline

To use GitLab's CICD pipeline we need to create a _.gitlab-ci.yml_ file, this is the heart of CI in GitLab.

{{< figure src="/images/2020/01/cicd_pipeline_infograph.png" caption="CICD pipeline overview" >}}

Below a breakdown of of this _yaml_ file. On my system it will spin up a Kubernetes Ubuntu pod, in this pod the verify test will run and the Ansible playbooks will be runned. But it should also work on a Gitlab runner with Docker.

### before_script

First we need a _before_script_ which will be executed by the GitLab runner at the start. We will use a local Docker image that has the environment for Ansible. This already has installed _python,_  _ansible_ and _ansible-lint_. Also we will put the private SSH key here. This key is defined as a variable in GitLab so this key is **not** in the git repository.

```yaml
cover:
  image: localhost:5000/ubuntu_ansible

before_script:
  - whoami
  - apt-get update -qy #update system
  - git submodule update --init
  - ansible --version
  - ansible-lint --version
  - mkdir secret # create secret dir
  - echo "$ANSIBLE_SSHKEY" > secret/ansible.key # create priv key from gitlab variable
  - chmod 400 secret/ansible.key
  - export ANSIBLE_HOST_KEY_CHECKING=False # disable host key checking for ansible

```

```bash
#Download base image ubuntu 18.04
FROM ubuntu:18.04

# Update Ubuntu Software repository
RUN apt-get -qy update
RUN apt install -qy python software-properties-common git
RUN apt-add-repository --yes --update ppa:ansible/ansible
RUN apt install -qy ansible ansible-lint

CMD ["/bin/bash"]
```

### Stages

We have defined a couple of stages we want to run on certain events in GitLab.

```yaml
stages:
  - verify
  - prestaging
  - staging
  - predeploy
  - deploy
```

### Verify commit

For each commit into GitLab there will be a verify pipeline running. That will do an _ansible-lint_ and a _ansible-playbook --check_. This is to make sure that the code is syntax error free.

```yaml
 #verify syntaxs etc...
ansible-verify:
  stage: verify
  script:
    - ansible-lint -v *.yml
    - ansible-playbook --inventory inventory/production --syntax-check *.yml
  rules:
    - if: '$CI_BUILD_BEFORE_SHA == "0000000000000000000000000000000000000000"'
      when: always
    - if: '$CI_COMMIT_BRANCH != "master" && $CI_COMMIT_BRANCH != "production"'
      when: always
```

{{< figure src="/images/2020/01/Screenshot-from-2020-01-15-14-49-14.png" caption="commit to work branch, pass the verify pipeline" >}}

### Merge request to master (staging)

When you need to push code to the staging environment you do this via a merge request to master. Then again will the verify stage be runned, but also there wil be a pre-staging and staging pipe be runned. The pre-staging pipeline will do an Ansible ping. To make sure all hosts are up and working for Ansible. If this fails the pipeline will be cancelled. If the staging hosts are ok, the the Ansible playbook will be runned and the code will be commit to master.

{{< figure src="/images/2020/01/Screenshot-from-2020-01-15-14-51-33.png" caption="Create a merge request from work-branch to master (staging)" >}}

We can tick the option to automatically merge to master when the pipeline succeeds.

{{< figure src="/images/2020/01/Screenshot-from-2020-01-15-15-00-04.png" caption="running staging pipeline" >}}

The yaml code that will check is staging hosts are online and working via Ansible ping. If they are execute the playbook, on success merge to master.

```yaml
# make sure all staging hosts are online and can be manged by ansible if not stop pipeline
prestaging:
  stage: prestaging
  script:
    - ansible --private-key secret/ansible.key --user ansible --inventory inventory/staging all -m ping
  rules:
    - if: '$CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master"'
      when: always

# test playbook on staging envirioment only run on merge requests to master
staging:
  stage: staging
  script:
    - ./cicd-info.sh > roles/common/files/cicd-info.txt # Run script to get GitLab CI env vars. Ansible will copy then to hosts.
    - ansible-playbook --private-key secret/ansible.key --user ansible --inventory inventory/staging common.yml
  rules:
    - if: '$CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master"'
      when: always
```

{{< figure src="/images/2020/01/Screenshot-from-2020-01-15-13-27-02.png" caption="merge to master (staging) pipeline" >}}

### Merge request to production from master (staging)

To deploy to production will also happen via a merge request. It is the same principle as from work-brand to master. The only difference is that Ansible will use production hosts.

```yaml
# make sure all production hosts are online and can be manged by ansible if not stop pipeline
predeploy:
  stage: predeploy
  script:
    - ansible --private-key secret/ansible.key --user ansible --inventory inventory/production all -m ping
  rules:
    - if: '$CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "production"'
      when: always

# deploy playbooks to production
deploy:
  stage: deploy
  script:
    - ./cicd-info.sh > roles/common/files/cicd-info.txt # Run script to get GitLab CI env vars. Ansible will copy then to hosts.
    - ansible-playbook --private-key secret/ansible.key --user ansible --inventory inventory/production common.yml
  rules:
    - if: '$CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "production"'
      when: always
```

{{< figure src="/images/2020/01/Screenshot-from-2020-01-15-15-07-55.png" caption="merge to production pipeline" >}}

## Job overview

Here we see all jobs that are involved from the commit in the work-branch al the way to production. These jobs are lint and syntax checked, have passed staging and have been successful deployed on production.

{{< figure src="/images/2020/01/Screenshot-from-2020-01-15-15-16-23.png" caption="job overview" >}}

## [cicd-info.sh](__GHOST_URL__/ansible-ci-with-gitlab/cicd-info.sh)

I also created a small script, that will gather some usefull GitLab CI envirioment varibles on the runner. This script will put these in a file and Ansible will pick this file up and put the on the hosts.

```bash
#!/bin/bash

echo "Gitlab CICD Ansible run info"
echo
echo "started on `date`"
echo "Project name: $CI_PROJECT_NAME"
echo "Commit SHA: $CI_COMMIT_SHORT_SHA"
echo "On runner: $HOSTNAME with ID: $CI_RUNNER_ID"
echo "Job info url: $CI_JOB_URL"
echo "By user: $GITLAB_USER_LOGIN <$GITLAB_USER_EMAIL>"
echo "Commit: $CI_COMMIT_TITLE"
```

```yaml
---
- name: Copy GitLab cicd envirioment run to host
  copy:
    src: cicd-info.txt
    dest: /etc/cicd-info.txt
    owner: root
    group: root
    mode: '0755'
```

The output on the hosts will look something like this. With these info we can always look at the latest Ansible run for this host in GitLab.

```
0 dennis@ragnarok:~> cat /etc/cicd-info.txt 
Gitlab CICD Ansible run info

started on Wed Jan 15 14:09:34 UTC 2020
Project name: ansible
Commit SHA: ed2cc1b0
On runner: runner-bhpg76e-project-2-concurrent-0dgklh with ID: 2
Job info url: http://192.168.66.148/dkruyt/ansible/-/jobs/192
By user: dkruyt <dennis@nospam.org>
Commit: Merge branch 'work-branch' into 'master'
```

## Protect master and production branch

Make sure your master and production branch are protected, so no one can commit directly in to these branches. Because we want to run the correct pipelines on merge requests.

{{< figure src="/images/2020/01/Screenshot-from-2020-01-15-10-06-53.png" caption="protect your branches" >}}



