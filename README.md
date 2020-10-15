# Automated AppDynamics Agent Download 

A simple script that lets you programmatically download AppDynamics agents. 

## Supported agent types

*You may create an issue if you need an agent that is not in the list* 

| Agent type | Description |
|--|--|
|`sun-java`    or     `java`      | Agent to monitor Java applications (for Sun and JRockit JVM) running on legacy JRE versions (1.6 and 1.7) |
|`sun-java8`   or     `java8`   | Agent to monitor Java applications (All Vendors) running on JRE version 1.8 and above |
|`machine` | 64 Bit Machine agent ZIP bundle with JRE to monitor your Linux servers |
|`machine-win` | 64 Bit Machine agent ZIP bundle with JRE to monitor your windows servers. |
|`dotnet-core` | AppDynamics .NET agent for Linux allows you to monitor your .NET core applications in production running on Linux |
|`db` | Agent to monitor any combination of DB2, Oracle, SQL Server, Sybase, MySQL, Sybase IQ and PostgreSQL database platforms. Linux install |
|`db-win` | Agent to monitor any combination of DB2, Oracle, SQL Server, Sybase, MySQL, Sybase IQ and PostgreSQL database platforms. Windows Install|



## Steps: 

1. Clone the repo 
2. Make the script executable `chmod +x get-agent.sh`
3. Ensure you have `jq` installed - https://stedolan.github.io/jq/download/

## Examples:

### Get Help 
 
 use -h or --help to get help 
 
 ` ./get-agent.sh -h`
 
````Usage: get-agent.sh [OPTIONS...]
  -h, --help, help                 Print this help

  download AGENT                   Agent to download (choices: sun-java | java | sun-java8 | java8 | ibm-java | machine | machine-win | dotnet | dotnet-core | db | db-win)
    -v, --version  version             Version number for the supplied agent
    -d, --dryrun                       Rturns only the download URL if specificed, it is recommended to use this arg for provisioning tools such as ansible, chef, etc
`````

### Download the Java Agent 

`./get-agent.sh download sun-java -v 20.6.0 `

The above command will dowload the sun-java agent and unzip it's content in a folder in the script's path. 


### Dry run 

It is recommended to use the `--dryrun` or `-d` arg if you intend to embed this script into a configuration management and/or provisioning tools like Ansible, Chef, Puppet, Terraform, Pulumi etc. 

`./get-agent.sh download machine-win -v 20.6.0 --dryrun ` or

`./get-agent.sh download dotnet -v 20.8.0 -d `

outputs only the download URL, for example

`https://download-files.appdynamics.com/download-file/machine-bundle/20.6.0.2676/machineagent-bundle-64bit-windows-20.6.0.2676.zip`

which you can pass to a curl or similar command on the target machine. 

####  Ansible Task Example 

##### getAgent playbook 
````
--
- hosts: all
  tasks:
    - include_role:
        name: java
      vars:
        agent_version: 20.8.0
        agent_type: sun-java
        application_environment: "production"
        # Your controller details 
        controller_account_access_key: "ffdsfs" # Please add this to your Vault 
        controller_host_name: "saas.appdynamics.com" # Your AppDynamics controller 
        controller_account_name: "customer1" # Please add this to your Vault 
        ssl_enabled: "false"
        controller_port: "443"
        
````

##### getAgent task: main.yaml 

````

---
- name: "Get AppDynamics Agent Download URL type"
  command: "{{ role_path }}/files/get-agent.sh download {{ agent_type }} -v {{ agent_version }} --dryrun"
  register: agent_download_url
  delegate_to: localhost
  failed_when:
   (agent_download_url.rc != 0) or 
   (agent_download_url.stderr != '') or 
   (agent_download_url.stdout is not match("https://download-files.appdynamics.com/download-file/"))
  changed_when: False

- debug: 
   msg: "{{ agent_download_url.stdout }}"
   
````





