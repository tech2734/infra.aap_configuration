# infra.eda_configuration.project

## Description

An Ansible Role to create Projects in EDA Controller.

## Variables

|Variable Name|Default Value|Required|Description|Example|
|:---:|:---:|:---:|:---:|:---:|
|`eda_host`|""|yes|URL to the EDA Controller (alias: `eda_hostname`)|127.0.0.1|
|`aap_username`|""|yes|Admin User on the Ansible Automation Platform Server. Either username / password or oauthtoken need to be specified.||
|`aap_password`|""|yes|Platform Admin User's password on the EDA Controller Server.  This should be stored in an Ansible Vault at vars/platform-secrets.yml or elsewhere and called from a parent playbook.||

|`aap_validate_certs`|`False`|no|Whether or not to validate the Ansible Automation Platform Server's SSL certificate.||
|`aap_request_timeout`|`10`|no|Specify the timeout Ansible should use in requests to the Automation Platform host.||
|`aap_configuration_async_dir`|`null`|no|Sets the directory to write the results file for async tasks. The default value is set to `null` which uses the Ansible Default of `/root/.ansible_async/`.||
|`aap_request_timeout`|`10`|no|Specify the timeout in seconds Ansible should use in requests to the Ansible Automation Platform host.||
|`eda_projects`|`see below`|yes|Data structure describing your projects, described below.||

### Secure Logging Variables

The following Variables compliment each other.
If Both variables are not set, secure logging defaults to false.
The role defaults to False as normally the add project task does not include sensitive information.
eda_configuration_project_secure_logging defaults to the value of aap_configuration_secure_logging if it is not explicitly called. This allows for secure logging to be toggled for the entire suite of EDA Controller configuration roles with a single variable, or for the user to selectively use it.

|Variable Name|Default Value|Required|Description|
|:---:|:---:|:---:|:---:|
|`eda_configuration_project_secure_logging`|`False`|no|Whether or not to include the sensitive Project role tasks in the log.  Set this value to `True` if you will be providing your sensitive values from elsewhere.|
|`aap_configuration_secure_logging`|`False`|no|This variable enables secure logging as well, but is shared across multiple roles, see above.|

### Asynchronous Retry Variables

The following Variables set asynchronous retries for the role.
If neither of the retries or delay or retries are set, they will default to their respective defaults.
This allows for all items to be created, then checked that the task finishes successfully.
This also speeds up the overall role.

|Variable Name|Default Value|Required|Description|
|:---:|:---:|:---:|:---:|
|`aap_configuration_async_retries`|50|no|This variable sets the number of retries to attempt for the role globally.|
|`eda_configuration_project_async_retries`|`aap_configuration_async_retries`|no|This variable sets the number of retries to attempt for the role.|
|`eda_configuration_async_delay`|1|no|This sets the delay between retries for the role globally.|
|`eda_configuration_project_async_delay`|`eda_configuration_async_delay`|no|This sets the delay between retries for the role.|

## Data Structure

### Project Variables

|Variable Name|Default Value|Required|Type|Description|
|:---:|:---:|:---:|:---:|:---:|
|`name`|""|yes|str|Project name. Must be lower case containing only alphanumeric characters and underscores.|
|`new_name`|""|no|str|Setting this option will change the existing name (looked up via the name field.)|
|`description`|""|no|str|Description to use for the Project.|
|`url`|""|yes|str|A URL to a remote archive, such as a Github Release or a build artifact stored in Artifactory and unpacks it into the project path for use. (Alias: scm_url)|
|`organization`|""|no|str|Organization this project belongs to.|
|`credential`|""|no|str|The token needed to utilize the SCM URL.|
|`state`|`present`|no|str|Desired state of the project.|

### Standard Project Data Structure

#### Yaml Example

```yaml
---
eda_projects:
  - name: my_project
    description: my awesome project
    url: https://github.com/ansible/ansible-rulebook.git
    tls_validation: true
    credential: test_token
```

## Playbook Examples

### Standard Role Usage

```yaml
---
- name: Add project to EDA Controller
  hosts: localhost
  connection: local
  gather_facts: false
  vars:
    eda_validate_certs: false
  # Define following vars here, or in eda_configs/eda_auth.yml
  # controller_host: ansible-eda-web-svc-test-project.example.com
  # eda_token: changeme
  pre_tasks:
    - name: Include vars from eda_configs directory
      ansible.builtin.include_vars:
        dir: ./vars
        extensions: ["yml"]
      tags:
        - always
  roles:
    - infra.eda_configuration.projects
```

## License

[GPLv3+](https://github.com/redhat-cop/eda_configuration#licensing)

## Author

[Chris Renwick](https://github.com/crenwick93/)
