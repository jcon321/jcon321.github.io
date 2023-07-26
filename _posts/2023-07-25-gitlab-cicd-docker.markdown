---
layout: post
title:  "GitLab CI/CD & Docker"
date:   2023-07-25 20:19:55 -0400
---

GitLab is a great alternative to GitHub, especially if your prefer to host yourself. GitLab comes with an easy to use, but powerful CI/CD pipeline process. We'll use it to fetch, build, and deploy a docker container via GitLab Runner.


## Environment
- Ubuntu Server OS, within VirtualBox. GitLab is a behemoth, so we'll test out its capabilities on a local virtual machine. Current settings are 4 CPU(s) amd 8GB of memory.


## GitLab Installation
GitLab's official documentation is a great source for information. [Install self-managed GitLab](https://about.gitlab.com/install/){:target="_blank"}

- Omnibus Package
  - `sudo apt update`
  - `sudo apt install ca-certificates curl openssh-server postfix tzdata perl`
  - `curl -LO https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh`
  - `sudo bash /tmp/script.deb.sh`
  - `sudo apt install gitlab-ce`
  - `sudo vi /etc/gitlab/gitlab.rb`
    - In my case, I set `external_url` to `http://192.168.0.15` which was my virtual machine's IP address according to the host. (I used a bridged network connection in VirtualBox.)
    - In the past, I've had problem with prometheus' resource consumption - we'll be disabling it entirely for this demo. Set `prometheus_monitoring['enable'] = false`
  - `sudo gitlab-ctl reconfigure`
  - `sudo vi /etc/gitlab/initial_root_password`

  At this point you should be able to log into GitLab via the external URL and the `root` user & password just created.

  - GitLab Runner - Same host as GitLab
  - `curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash`
  - `sudo apt-get install gitlab-runner`
  - Log into GitLab, Admin Area --> CI/CD --> Runners; create a new instance runner. At that time a token will be given, use it in the next step to link the two.
  - `sudo gitlab-runner register`

  GitLab should show an online Runner.


  ## Docker Installation