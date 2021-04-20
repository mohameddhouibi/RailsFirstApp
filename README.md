# Jenkins Server Documentation

## WELCOME

Welcome to the Jenkins user documentation - for people wanting to use Jenkins’s existing functionality and plugin features.

---
The server is provisioned via the [ansible-2021](https://gitlab.conlance.org/devOps/ansible-2021) script.
---

## What is Jenkins?

Jenkins is a self-contained, open source automation server which can be used to automate all sorts of tasks related to building, testing, and delivering or deploying software.

Jenkins can be installed through native system packages, Docker, or even run standalone by any machine with a Java Runtime Environment (JRE) installed.



## Getting Started

### Prerequisite Requirements

- OpenJDK JDK / JRE 8 - 64 bits
- Jenkins (LTS)
- Docker
- Docker-Compose
- Ruby 2.7.2 

---
Make sure you have Ruby and Postgres installed in your environment.Check in the Gemfile for the exact supported version. use bundler to install all required gems.We are also making the assumption that you're familiar with how Rails apps are setup and deployed.  If this is not the case then you'll want to refer to documentation that will bridge any gaps in the instructions below.
---

Start the service:

```bash
sudo systemctl start jenkins
```
Runs on port 8080.

### how to add an application
    - Create a new job (Multi-Branch pipeline)
    - Add source code Managment as (Gitlab) 
    - select concerning branch 
    - add your jenkinsFile into the branch you want to build 
    - add your Docker-Compose file too 
    - start the created Job
   
## How to setup GitLab with Jenkins

### SSH Connection : 
-----
- [generate ssh keys on own server](https://docs.gitlab.com/ee/ssh/README.html) -> `PUBLIC` and `PRIVATE` ( no passphrase )
- go to GitLab profile settings -> SSH keys -> add `PUBLIC` ssh key to GitLab = this will allow your Jenkins server to access your GitLab
- also create full access GitLab `API access token` for Jenkins on your gitlab profile settings and keep it
---
- install Git + GitLab plugin on Jenkins ( make sure they are enabled (ticked) in Jenkins !!!)
- setup git username and mail in manage Jenkins
- restart jenkins
---
- go to manage Jenkins -> Gitlab section
- create new gitlab connection
    - set connection name
    - set gitlab server (https://gitlab.com)
    - add gitlab API credentials (create them with the `API access token` from beginning)
---
- create/choose GitLab project
- create Jenkins job:
    - **GitLab connection** = should be automatically selected to the connection you've just created
    - **Source Code Management** -> select Git
        - **Repository URL** = copy the `SSH url` from GitLab project page ( not the https !)
        - **Credentials** = now we need to give the SSH `PRIVATE` key to Jenkins -> Add new Credentials
            - Kind = SSH Username with private key
            - Username = your username for git
            - Private Key -> tick "Enter directly" and enter your `PRIVATE` key i.e.:
            ```
            -----BEGIN OPENSSH PRIVATE KEY-----
            asfkapokfrpa2kL2K3K32323
            -----END OPENSSH PRIVATE KEY-----
            ```
            - set description like Gitlab ssh key or something
            - click Add and make sure the SSH Credentials are selected
    - **Build Triggers** -> Build when a change is pushed to GitLab. :
        - this will also show you the `webhook URL`, keep it we'll need it later
        - probably retarded note ignore this if yo got no errors or running Jenkins on 8080 : notice that if you have wrong  Jenkins address set up in Jenkins, it will use the set up Jenkins address, not the real URL one so make sure the domain is right in the webhook url
        - **IMPORTANT :** expand the setting by clicking on Advanced and generate a `Secret token` = keep this `Secret token`, we'll use it for gitlab webhook
    - **Build** = via your jenkinsfile
    - **Post-build action** -> multi-branch pipeline will automatically notify Gitlab of the Build result 
    - run the job
-----

## Run Jenkins Behind Nginx 

----
- Check if Nginx is installed if not install it 

 ```bash

nginx -v
```
- Otherwise if you have the latest version start configuring it 

 ```bash

vim /etc/nginx/sites-available jenkins.conlance.org
```
---
- Add this server into sites-available : 
    --
        server
        {
        server_name jenkins.conlance.org ;
        location / {
            proxy_set_header        Host $host:$server_port;
            proxy_set_header        X-Real-IP $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header        X-Forwarded-Proto $scheme;

            proxy_pass          http://127.0.0.1:8080;
        }
        }       
---

- Jenkins Configuration 
   
    edit the jenkins default config file :
---
 ```bash

vim /etc/default jenkins
```
---   
    -Add below HTTP_PORT :
        HTTP_HOST = 127.0.0.1
    -Edit Jenkins_ARGS by adding to it end : --httpListenAddress=$HTTP_HOST
---    




