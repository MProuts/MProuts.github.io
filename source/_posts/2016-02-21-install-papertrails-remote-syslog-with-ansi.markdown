---
layout: post
title: "Install Papertrail's remote_syslog with Ansible"
date: 2016-02-21 11:25:15 -0500
comments: true
categories: Ansible, Papertrail, syslog, unix
---

## Files

#### /files/remote_syslog.conf

Download the
[example config](https://github.com/papertrail/remote_syslog/blob/master/examples/remote_syslog.upstart.conf)
from Papertrail.  Change the last line to:
`exec /usr/local/bin/remote_syslog -D`

#### /templates/log_files.yml

Download the
[example config](https://github.com/papertrail/remote_syslog/blob/master/examples/log_files.yml.example)
from Papertrail.  Change `host` and `port` under `destination`.  You
might want to store these values as variables in `/group_vars` in case
they vary across groups (e.g. between your Staging and Production
environments).
Also, edit paths under `files` to reflect that log files you want to
send to Papertrail.

## Tasks

#### /tasks/main.yml

```
---
# #########
# Papertail
# #########

- name: Download remote_syslog tarball
  get_url:
    url: https://github.com/papertrail/remote_syslog2/releases/download/v0.16/remote_syslog_linux_amd64.tar.gz
    dest: /home/deploy/remote_syslog.tar.gz
    # only downloadeds if destination file doesn't already exist
    force: no

- name: Unarchive remote_syslog tarball
  unarchive:
    src: /home/deploy/remote_syslog.tar.gz
    dest: /home/deploy/
    # only runs if this file doesn't already exist
    creates: /home/deploy/remote_syslog
    copy: no

- name: Copy remote_syslog executable into place
  file:
    src: /home/deploy/remote_syslog/remote_syslog
    dest: /usr/local/bin/remote_syslog
    state: link

- name: Copy remote_syslog config file
  template:
    src: log_files.yml.j2
    dest: "/etc/log_files.yml"
    owner: root
    group: root

- name: Copy remote_syslog service config file
  copy:
    src: remote_syslog.upstart.conf
    dest: "/etc/init/remote_syslog.conf"
    mode: 0644
    owner: root
    group: root
  notify: Restart remote_syslog

- name: Register remote_syslog service
  service: name=remote_syslog enabled=yes
  notify: Restart remote_syslog
```

#### /handlers/main.yml

```
- name: Restart remote_syslog
  service:
    name: remote_syslog
    state: restarted
```
