# EESSI setup

This repository contains ansible roles for setting up a computer as a part of the EESSI network. The supported roles are:

- stratum0 (the main repository server for all EESSI software)
- stratum1 (a distribution node to offload stratum0)
- proxy (offloads stratum1 servers)
- client (a computer that actually uses EESSI software for actual work)

# Using these roles

Fetch by doing `ansible-galaxy install -r requirements.yml`. Add `--force` to guarantee updating the requirements.

## Stratum0

#### **`requirements.yaml`**
``` yaml
roles:
  - name: eessi.stratum0
    src: https://github.com/EESSI/ansible-eessi-roles
    version: 1.0.0
```
#### **`main.yaml`**
``` yaml
- hosts: eessi_stratum0
  roles:
  - eessi.stratum0

  vars:
    # Storage device
    cvmfs_srv_device: /dev/sdb
```

## Stratum1

#### **`requirements.yaml`**
``` yaml
roles:
  - name: eessi.stratum1
    src: https://github.com/EESSI/ansible-eessi-roles
    version: 1.0.0
```
#### **`main.yaml`**
``` yaml
- hosts: eessi_stratum1
  roles:
  - eessi.stratum1

  vars:
    # The license key for the Geo API:
    # https://cvmfs.readthedocs.io/en/stable/cpt-replica.html#geo-api-setup
    # For some unclear reason, the Stratum 1 installation fails when this is not set:
    # https://github.com/EESSI/filesystem-layer/issues/2
    cvmfs_geo_license_key: INSERT_YOUR_KEY


    # List of clients allowed to access your local proxies.
    # Add individual IPs and/or use CIDR notation.
    local_cvmfs_http_proxies_allowed_clients:
      - 192.168.0.0/12
      - 10.0.0.15


    # List of all http proxies that should be configured for the clients.
    # Remove or comment the line if you do not want to use a proxy; in this
    # case it will be set to "DIRECT" in the client configuration.
    local_cvmfs_http_proxies:
      - your-proxy-1:3128
      - your-proxy-2:3128
    # The following one-liner can be used to automatically add all the hosts
    # defined in the cvmfslocalproxies group in your hosts file
    # to local_cvmfs_http_proxies, using port number 3128.
    # local_cvmfs_http_proxies: "{{ groups.cvmfslocalproxies | map('regex_replace', '^(.*)$', '\\1:3128') | list }}"


    # Uncomment if you want to use your own Squid configuration template for the local proxies
    # local_proxies_cvmfs_squid_conf_src: "/path/to/your/template"

    # Uncomment if you want to use your own Squid configuration template for the Stratum 1
    # local_stratum1_cvmfs_squid_conf_src: : "/path/to/your/template"
```
