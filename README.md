# Cobalt Strike Ansible Deployment Guide

## Environmental Prerequisite Setup

- **Ensure** that you have `python3` installed, or that that the value returned from `python --version` returns a version greater or equal to 3.1.

- In order to work as expected, it is ***highly encouraged*** to take advantage of Python's virtualized environments, as the default installation namespace is global.  
    ```sh
    python -m pip install virtualenv
    python -m virtualenv -p python3 Ansible_Virtual_Environment
    source Ansible_Virtual_Environment/bin/activate

    # Install modules local to this active virtualized environment
    (Ansible_Virtual_Environent)$: pip install ansible dnspython
    (Ansible_Virtual_Environent)$: ansible-galaxy collection install community.general
    ```


## Playbook Prerequisite Setup

- `Ansible` operates primarily over `SSH`, and will require passwordless `SSH` keys added to any and all `Linux` hosts in question (teamserver and teamshare).
    - To generate a new `SSH` key locally:
        ```sh
        # Just hit enter for any prompts that occur.
        # This will allow create with no password.

        ssh-keygen -t rsa
        ```

- Start the relevant `SSH` server on the remote host(s):
    ```sh
    # On the remote server(s)
    service openssh start
    ```

- Add your newly generated `SSH` public key on the remote server(s):
    ```sh
    # From your current client host. E.g. your "attack" virtual machine.
    ssh-copy-id -i ~/.ssh/id_rsa root@remote_host
    ```

- Create an entry in your `SSH` configuration file, to have a reference to our endpoint(s), commonly residing in `~/.ssh/config`:
    ```sh
    # Example configuration
    Host teamserver
	    HostName 10.1.1.2
	    User root
	    IdentityFile ~/.ssh/id_rsa

    Host teamshare
	    HostName 10.1.1.3
        User root
	    IdentityFile ~/.ssh/id_rsa
    ```

- `cd` into the Ansible directory, and edit the variables contained in the `playbook.yml` file:
    ```sh
    # Teamserver variables 
    cs_license: ""
    provided_domain: ""
    provided_password: ""
    namecheap_user: ""
    namecheapapi_key: ""

    # Teamshare variables
    share_user: ""
    share_pass: ""
    ```

- You ***must*** replace the `NULL`'d out `cobaltstrike.zip` file, located in `Ansible/roles/teamservers/files/cobalt_strike`


## Playbook Role Definitions and Features
#### Teamservers
    - Provisions the `teamserver` as dicated in the `Ansible` inventory file
        - Installs pre-requisite software
        - Transfers `Cobalt Strike` profile (change as you see fit)
        - Populates initial `DNS` records within `Namecheap`
            - Awaits output on a thirty-second interval to determine if they have populated
            - Populates `DMARC` and `DKIM` records after successful propagation
        - Configures the `LetsEncrypt` `SSL` certificates for `@` and `www` primary/subdomain(s)
        - Configures the mailserver components (including users and passwords)


#### Teamshares

    - Creates a "teamshare" with subdirectories for organization of common operational efforts
        - Allows creation of a primary account, under which, the user's home is the shared directory root


## Playbook Execution

- From within the `Ansible` directory, run the following command:
    ```sh
    ansible-playbook -i inventory.yml playbook.yml
    ```