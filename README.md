## ansible-deployment

* Deploys an application to a target server or virtual machine in versioned directories (allowing for rollback if needed)
* Creates an init.d daemon service for the application (Optional)
* Makes an initial http request to check the application is up and delivers (assumes you deploy a web application responding to HTTP requests) (Optional)

Currently: Artifact to deploy must be stored on a nexus server.

### TODO:

* README is out of date; adapt to new code
* dependency: httplib is required
* clarify one-app-per-server-assumption

### Usage guide

Example deploying a Scala / Play Framework application

```yml
- name: Deploy Play
  hosts: YOUR_TARGET_HOST_OR_HOST_GROUP
  sudo: yes
  gather_facts: yes
  vars:
    deployment_nexus_url_base              : "http://myNexus"
    deployment_user                        : "myUnixUsername"
    deployment_artifact_name               : "myApp"
    deployment_artifact_organization       : "com.my.company"
    deployment_http_check_port             : "8080"
    deployment_exec_arg                    : "-Dhttp.port={{deployment_http_check_port}}"
    deployment_grep_name                   : "http.port={{deployment_http_check_port}}"
    deployment_bootstrap_wait_sec          : 15
    # used for http checks that the service is operating as expected.
    deployment_http_check_url_suffix       : "/health"
    deployment_http_check_expected_content : "{{ nexus_artifact_info.json.data.version }}"
  roles:
    - { role: ansible-deployment,       tags: ["deploy"] }
```

### Role variables

List of default variables available in the inventory:

```yml
---

deployment_install_initd                    : true

# Directory structure for app
deployment_user                             : "myUnixUsername"
deployment_group                            : "{{ deployment_user }}"
deployment_work_dir                         : "/home/{{deployment_user}}/{{deployment_artifact_name}}"
deployment_current_dir                       : "{{deployment_work_dir}}/current"

# artifact settings
deployment_artifact_name                    : "myApp"
deployment_artifact_organization            : "com.my.company"
deployment_artifact_version                 : "LATEST"
deployment_artifact_extension               : "tgz"
deployment_artifact_destination_name        : "{{ deployment_artifact_name }}.{{ deployment_artifact_extension }}"

# Used within the init.d script
deployment_exec_cmd                         : "{{ deployment_current_dir }}/bin/{{ deployment_artifact_name }}"
deployment_exec_arg                         : ""

# Used for getting PID ID, it should be a unique value
deployment_grep_name                        : "{{ deployment_exec_cmd }}"

# How long to probe for an active process upon service start / restart
deployment_timeout_start_sec                : 5
# Pause time to wait for application to bootstrap. Needed for e.g. apps including the new relic agent.
deployment_bootstrap_wait_sec                   : 0

# HTTP GET request to check that the server is up and running
# the content of the result is then checked and compared with _expected_content
# on comparison failure the ansible task fails to alert that something has gone wrong.
deployment_http_check_expected_content      : "" # some html or json content to be expected in the http body
deployment_http_check_url_suffix            : "" # the url suffix from the base url, e.g "/index.html"
deployment_http_check_protocol              : "http://"
deployment_http_check_port                  : "80"
deployment_http_check_domain                : "{{ansible_default_ipv4.address}}"
deployment_http_check_url                   : "{{deployment_http_check_protocol}}{{deployment_http_check_domain}}:{{deployment_http_check_port}}{{deployment_http_check_url_suffix}}"

# nexus settings
deployment_nexus_url_base                   : ""
deployment_nexus_url_resolver               : "{{ deployment_nexus_url_base }}/nexus/service/local/artifact/maven/resolve?r=releases&g={{ deployment_artifact_organization }}&a={{ deployment_artifact_name }}&e={{ deployment_artifact_extension }}&v={{ deployment_artifact_version }}"
deployment_nexus_url_download               : "{{ deployment_nexus_url_base }}/nexus/content/repositories/releases/{{ nexus_artifact_info.json.data.repositoryPath }}"
```

List of internal variables used by the role:

    deployment_artifact_version_dir


### License

[MIT](LICENSE)

### Authors

* [Joé Schaul](https://github.com/jschaul)
* [Adham Helal](https://github.com/ahelal)
* [Engin Yöyen](https://github.com/enginyoyen)
