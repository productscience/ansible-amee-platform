AMEE Platform Role Name
========

This ansible role sets up and deploys the amee platform to a single host, installing the necessary software dependencies, like java, rabbitMQ, nginx and so on, and checking out a specified release from the openamee repo on Github.


Requirements
------------

This assumes:

1. There is a host running Ubuntu, with an ubuntu user account, setup with sudo capabilities, that ansible can log into.
2. For the indexing step, a sizable amount (30-40GB for the production amee database) of disk space is needed, while an index of the data is created. While the index itself for the production amee platform is less than a gigabyte in size, about 31GB of logs are generated when creating the index itself.
3. There is already a MySQL database server, or AWS RDS endpoint to connect to, and relevant firewall settings have been setup
4. The amee platform is the only application running (no other nginx virtual hosts)

#### Security and crypto:

Concerning the platform securely, this also assumes that :

1. The Oracle java infinite strength cryptography libraries have been downloaded and are available at path specified by `platform_local_policy_jar_path`, and `platform_US_export_policy_jar_path`
2. The necessary SSL certificates and keys are available at local paths specified by `platform_ngx_ssl_cert_file` and `platform_ngx_ssl_key_file` respectively
3. The required salt and key files used by the platform itself for crypto are available at the paths specificed by `platform_salt_path`, and `platform_salt_key_path`
4. There is a deploy key generated for the _deploy_user_, so it can checkout git repositories over SSH.

Role Variables
--------------

 - **platform_app**: the name of the application itself
 - **platform_server_name**: the full hostname of the server, used by nginx
 - **platform_local_policy_jar_path**: for the oracle crypto libraries
 - **platform_US_export_policy_jar_path**: as above
 - **platform_salt_path**: the salt used by the application
 - **platform_key_path**: the key used by the application
 - **platform_db_user**: this is the database user account used to connect to MySQL
 - **platform_indexing_mode**: whether the application it set to rebuild its index (usually false). Set this to true to blow away the existing index and rebuild it. _This takes a few hours, and generates about 31GB of log files._
 - **platform_index_path**: where the lucene index is stored on the server
 - **platform_log_dir**: which directory the platform logs to. The platform creates new log files once they grow beyond 500MB in size, and names them numerically
 - **platform_mq_user**: the rabbit mq user account to connect to the server with
 - **platform_mq_user_password**: the password for the rabbit mq user account
 - **platform_train_route**: the full url for the the railservice. This is used to perform of the distance by rail of calculations. The data is a few years old, this serice is not used.
 - **platform_ngx_ssl_cert_file**: the path for nginx to look for the SSL certificate, when serving requests over HTTPS
 - **ngx_ssl_cert_key**: the path for nginx to look for its SSL key, when serving requests over HTTPS


Dependencies
------------

This depends on:

- the **smola.java** role to install the Oracle's flavour of Java 7 without needing to manually accept their Ts and Cs
- the **productscience.deploy** user, to generate the deploy user, that checks out code, and runs the application as.

Example Playbook
-------------------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
---
- hosts: aws-east
  sudo: yes
  vars:
    deploy_user_name: APP_DEPLOY_NAME
    deploy_user_ssh_priv_key_path: ../credentials/APP_DEPLOY_NAME.deploy
    platform_server_name: live.company.com
    platform_app: APP_NAME
    git_branch: 3.11.0
    java_packages:
      - oracle-java7-installer
      - oracle-java7-set-default
    platform_db_user: ADD_DB_USER_HERE
    platform_db_password: ADD_PASSWORD_HERE
    platform_db_endpoint: "database-server.company.com:3306"
    platform_indexing_mode: false
    platform_index_path: "/mnt/amee-platform/index"
    platform_log_dir: "/var/log/amee-platform/logs"
    platform_mq_user: ADD_MQ_USER_PASSWORD_HERE
    platform_mq_user_password: ADD_MQ_USER_PASSWORD_HERE
    platform_train_route: "http://trainservice.company.com"
    ngx_ssl_cert_path: "/etc/ssl/certs/company.crt"
    ngx_ssl_cert_key: "/etc/ssl/certs/company.key"

  roles:
    - productscience.deploy_user
    - smola.java
    - productscience.amee_platform
```

License
-------

BSD

Author Information
------------------

This ansible role was created by Chris Adams of Product Science and David Keen, for deploying the amee platform to AWS infrastructure.

Please send any queries to hello@productscience.co.uk, or create an issue in the project tracker.
