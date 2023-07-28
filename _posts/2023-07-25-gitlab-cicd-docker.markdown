---
layout: post
title:  "GitLab CI/CD & Docker"
date:   2023-07-25 20:19:55 -0400
---

GitLab is a great alternative to GitHub, especially if your prefer to host yourself. GitLab comes with an easy to use, but powerful CI/CD pipeline process. We'll use it to fetch, build, and deploy a Docker container via GitLab Runner.


## Environment
- Ubuntu 20.04 Server OS, within VirtualBox. GitLab is a behemoth, so we'll test out its capabilities on a local virtual machine. Current settings are 4 CPU(s), 8GB of memory, and 20GB of space.


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
  Install Docker through the official Docker repository. [DigitalOcean - Installing Docker](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04#step-1-installing-docker){:target="_blank"}
  - `sudo apt install apt-transport-https software-properties-common`
  - `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`
  - `sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"`
  - `apt-cache policy docker-ce`
  - `sudo apt install docker-ce`
  - `sudo systemctl status docker`
  - `sudo docker --version`
  - To run docker as non-root user, we can add the current user to the `docker` group. Check this out for why you may or may not want to do that. [Docker Post-Install](https://docs.docker.com/engine/install/linux-postinstall/){:target="_blank"}
  - `sudo usermod -aG docker $USER`
  - Log out and in to reset permissions
  - `docker --version`

  At this point we should now have Docker running. Next we'll download a demo Spring Boot project and setup a local build environment.


  ## Spring Boot with Docker
  [Spring Initializer](https://spring.io/guides/topicals/spring-boot-docker/){:target="_blank"} comes with a command line interface we can use to download project templates or custom configuration. Check out `curl https://start.spring.io` to see all the options. We'll download the Web Project in Java 17 demo.
  - `curl -G https://start.spring.io/starter.zip -d dependencies=web -d javaVersion=17 -o demo.zip`
  - `unzip demo.zip`
  - The project comes with a Gradle Wrapper, which is a prebuilt jar to execute Gradle for us. However, we'll need to download a JDK, in our case 17.
  - `sudo apt-get install openjdk-17-jdk`
  - `java --version`
  - Building the project should now work with the Gradle Wrapper `./gradlew clean build`

  Todo - Dockerfile next
  