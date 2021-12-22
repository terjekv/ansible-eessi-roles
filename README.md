# EESSI setup

This repository contains ansible roles for setting up a computer as a part of the EESSI network. The supported roles are:

- stratum0 (the main repository server for all EESSI software)
- stratum1 (a distribution node to offload stratum0)
- proxy (offloads stratum1 servers)
- client (a computer that actually uses EESSI software for actual work)

# Using these roles

Fetch by doing `ansible-galaxy install -r requirements.yml`. Add `--force` to guarantee updating the requirements. This is useful if your version is a branch like `develop`. 

## Stratum0

#### **`requirements.yaml`**
``` yaml
roles:
  - name: eessi.roles
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
  - name: eessi.roles
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

    # Everything after this is optional. The defaults should provide a working stratum1
    # with local monitoring enabled.

    # Set the node as a public node. If the node is set as public the prometheus
    # instance will allow monitoring.eessi-infra.org to connect to scrape data.
    eessi_public: false

    # Enable monitoring on nodes, defaults to true.
    # This will install grafana, prometheus, node_exporter, and cvmfs_exporter
    # on the monitored node.
    eessi_monitoring: true

    # All services bind to localhost, unless eessi_public is set to
    # true, in that case grafana and prometheus binds to "*". 
    eessi_service_ports:
      grafana: 3000
      prometheus: 9090
      node_exporter: 9100
      cvmfs_exporter: 9101

    # If you want to use your own Squid configuration template for the Stratum 1
    local_stratum1_cvmfs_squid_conf_src: "/path/to/your/template"
```

## Proxy

#### **`requirements.yaml`**
``` yaml
roles:
  - name: eessi.roles
    src: https://github.com/EESSI/ansible-eessi-roles
    version: 1.0.0
```
#### **`main.yaml`**
``` yaml
- hosts: eessi_proxy
  roles:
  - eessi.proxy

  vars:
    # List of clients allowed to access your local proxies.
    # Add individual IPs and/or use CIDR notation.
    local_cvmfs_http_proxies_allowed_clients:
      - 192.168.0.0/12
      - 10.0.0.15

    # If you want to use your own Squid configuration template for the local proxies
    local_proxies_cvmfs_squid_conf_src: "/path/to/your/template"
```

## Clients

#### **`requirements.yaml`**
``` yaml
roles:
  - name: eessi.roles
    src: https://github.com/EESSI/ansible-eessi-roles
    version: 1.0.0
```
#### **`main.yaml`**
``` yaml
- hosts: eessi_client
  roles:
  - eessi.client

  vars:
    # List of all http proxies that should be configured for the clients.
    # Remove or comment the line if you do not want to use a proxy; in this
    # case it will be set to "DIRECT" in the client configuration.
    local_cvmfs_http_proxies:
      - your-proxy-1:3128
      - your-proxy-2:3128
```
