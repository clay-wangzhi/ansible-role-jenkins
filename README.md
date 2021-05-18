# Ansible Role: Jenkins CI

Installs Jenkins CI on RHEL/CentOS servers.

根据`geerlingguy.jenkins`roles更改，主要更改有：

* 更改了rpm包下载地址，和plugin下载源地址，更改为了国内的清华源
* 增加了常用的插件
* readme 中增加安装完成后的优化，拉到下面可以看到

## Requirements

Requires `curl` to be installed on the server. Also, newer versions of Jenkins require Java 8+ (see the test playbooks inside the `molecule/default` directory for an example of how to use newer versions of Java for your OS).

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    jenkins_package_state: present

The state of the `jenkins` package install. By default this role installs Jenkins but will not upgrade Jenkins (when using package-based installs). If you want to always update to the latest version, change this to `latest`.

    jenkins_hostname: localhost

The system hostname; usually `localhost` works fine. This will be used during setup to communicate with the running Jenkins instance via HTTP requests.

    jenkins_home: /var/lib/jenkins

The Jenkins home directory which, amongst others, is being used for storing artifacts, workspaces and plugins. This variable allows you to override the default `/var/lib/jenkins` location.

    jenkins_http_port: 8080

The HTTP port for Jenkins' web interface.

    jenkins_admin_username: admin
    jenkins_admin_password: admin

Default admin account credentials which will be created the first time Jenkins is installed.

    jenkins_admin_password_file: ""

Default admin password file which will be created the first time Jenkins is installed as /var/lib/jenkins/secrets/initialAdminPassword

    jenkins_jar_location: /opt/jenkins-cli.jar

The location at which the `jenkins-cli.jar` jarfile will be kept. This is used for communicating with Jenkins via the CLI.

    jenkins_plugins: []

Jenkins plugins to be installed automatically during provisioning.

    jenkins_plugins_install_dependencies: true

Whether Jenkins plugins to be installed should also install any plugin dependencies.

    jenkins_plugins_state: present

Use `latest` to ensure all plugins are running the most up-to-date version.

    jenkins_plugin_updates_expiration: 86400

Number of seconds after which a new copy of the update-center.json file is downloaded. Set it to 0 if no cache file should be used.

    jenkins_updates_url: "https://updates.jenkins.io"

The URL to use for Jenkins plugin updates and update-center information.

    jenkins_plugin_timeout: 30

The server connection timeout, in seconds, when installing Jenkins plugins.

    jenkins_version: "1.644"
    jenkins_pkg_url: "http://www.example.com"

(Optional) Then Jenkins version can be pinned to any version available on `http://pkg.jenkins-ci.org/debian/` (Debian/Ubuntu) or `http://pkg.jenkins-ci.org/redhat/` (RHEL/CentOS). If the Jenkins version you need is not available in the default package URLs, you can override the URL with your own; set `jenkins_pkg_url` (_Note_: the role depends on the same naming convention that `http://pkg.jenkins-ci.org/` uses).

    jenkins_url_prefix: ""

Used for setting a URL prefix for your Jenkins installation. The option is added as `--prefix={{ jenkins_url_prefix }}` to the Jenkins initialization `java` invocation, so you can access the installation at a path like `http://www.example.com{{ jenkins_url_prefix }}`. Make sure you start the prefix with a `/` (e.g. `/jenkins`).

    jenkins_connection_delay: 5
    jenkins_connection_retries: 60

Amount of time and number of times to wait when connecting to Jenkins after initial startup, to verify that Jenkins is running. Total time to wait = `delay` * `retries`, so by default this role will wait up to 300 seconds before timing out.

    # For RedHat/CentOS (role default):
    jenkins_repo_url: http://pkg.jenkins-ci.org/redhat/jenkins.repo
    jenkins_repo_key_url: http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
    # For Debian (role default):
    jenkins_repo_url: deb http://pkg.jenkins-ci.org/debian binary/
    jenkins_repo_key_url: http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key

This role will install the latest version of Jenkins by default (using the official repositories as listed above). You can override these variables (use the correct set for your platform) to install the current LTS version instead:

    # For RedHat/CentOS LTS:
    jenkins_repo_url: http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
    jenkins_repo_key_url: http://pkg.jenkins-ci.org/redhat-stable/jenkins-ci.org.key
    # For Debian/Ubuntu LTS:
    jenkins_repo_url: deb http://pkg.jenkins-ci.org/debian-stable binary/
    jenkins_repo_key_url: http://pkg.jenkins-ci.org/debian-stable/jenkins-ci.org.key

It is also possible stop the repo file being added by setting  `jenkins_repo_url = ''`. This is useful if, for example, you sign your own packages or run internal package management (e.g. Spacewalk).

    jenkins_java_options: "-Djenkins.install.runSetupWizard=false"

Extra Java options for the Jenkins launch command configured in the init file can be set with the var `jenkins_java_options`. For example, if you want to configure the timezone Jenkins uses, add `-Dorg.apache.commons.jelly.tags.fmt.timeZone=America/New_York`. By default, the option to disable the Jenkins 2.0 setup wizard is added.

    jenkins_init_changes:
      - option: "JENKINS_ARGS"
        value: "--prefix={{ jenkins_url_prefix }}"
      - option: "JENKINS_JAVA_OPTIONS"
        value: "{{ jenkins_java_options }}"

Changes made to the Jenkins init script; the default set of changes set the configured URL prefix and add in configured Java options for Jenkins' startup. You can add other option/value pairs if you need to set other options for the Jenkins init file.

    jenkins_proxy_host: ""
    jenkins_proxy_port: ""
    jenkins_proxy_noproxy:
      - "127.0.0.1"
      - "localhost"

If you are running Jenkins behind a proxy server, configure these options appropriately. Otherwise Jenkins will be configured with a direct Internet connection.

## Example Playbook

```yaml
---
- hosts: jenkins
  vars:
    jenkins_version: "2.210"
    jenkins_plugins: [
      "ansible",
      "ansicolor",
      "antisamy-markup-formatter",
      "authentication-tokens",
      "authorize-project",
      "blueocean",
      "branch-api",
      "build-timeout",
      "build-timestamp",
      "cloudbees-folder",
      "credentials-binding",
      "docker-commons",
      "docker-workflow",
      "email-ext",
      "git",
      "github-branch-source",
      "gitlab-plugin",
      "git-server",
      "gradle",
      "ldap",
      "localization-zh-cn",
      "mailer",
      "matrix-auth",
      "maven-plugin",
      "pam-auth",
      "pipeline-build-step",
      "pipeline-github-lib",
      "pipeline-graph-analysis",
      "pipeline-input-step",
      "pipeline-maven",
      "pipeline-milestone-step",
      "pipeline-model-api",
      "pipeline-model-declarative-agent",
      "pipeline-model-definition",
      "pipeline-rest-api",
      "pipeline-stage-step",
      "pipeline-stage-tags-metadata",
      "pipeline-stage-view",
      "ssh-slaves",
      "subversion",
      "timestamper",
      "workflow-aggregator",
      "workflow-multibranch",
      "ws-cleanup"
    ]
    jenkins_admin_username: admin
    jenkins_admin_password: 123456
  roles:
    - role: clay_wangzhi.jenkins
      become: yes
```

## Todo

安装好所有插件，再执行这些步骤
ansible 安装 jenkins 后，会有三个 warning，下面操作会关闭这些 warning。

1.打开系统设置，点击保存

2.打开全局安全配置

a)勾选跨站请求伪造保护，勾选默认碎片生成器

b)去掉”匿名用户具有可读权限“的勾

 c)勾选Agent → Master Security下面的选项

## 插件对应名称

declarative Pipeline 对应的插件为 pipeline-model-definition
pipeline对应的插件为workflow-aggregator

## License

MIT (Expat) / BSD

## Author Information

This role was created in 2014 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).

Updated by clay at 2019.