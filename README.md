minio
=========

Install and Configure Minio S3 Object-Storage.

This role allows you to deploy Minio as single-node, as well as highly available distributed multi-node setup.

It is recommended to deploy a load balancer such as HAProxy when setting up Minio in distributed mode with multiple nodes.

Requirements
------------

- No Requirements

Role Variables
--------------

The default values for the variables are set in `defaults/main.yml`:
```yml
# Minio binaries path
minio_server_bin: /usr/local/bin/minio
minio_client_bin: /usr/local/bin/mc

# Minio release to install. defaults to latest when values are empty or specify binary release (i.e. RELEASE.2022-10-29T06-21-33Z) 
minio_server_release: ""
minio_client_release: ""

# Default download URL for Minio server is 'https://dl.minio.io/server/minio/release/linux-amd64/archive/' and is hardcoded in the installation task.
# Default download URL for Minio client is 'https://dl.minio.io/client/mc/release/linux-amd64/archive/' and is hardcoded in the installation task.

# Optional: The download URL's can be overwritten to use a alternative path:
# minio_server_download_base_url: https://files.repository.mueller.de/minio/server/linux-amd64
# minio_client_download_base_url: https://files.repository.mueller.de/minio/client/linux-amd64

# Runtime user and group for the Minio server service
minio_user: minio
minio_group: minio

# Path to the file containing the ENV variables for the Minio server
minio_server_envfile: /etc/default/minio

# Minio server listen address
minio_bind_console_addr: ":9001"
minio_bind_api_addr: ":9000"

# Minio server data directories
minio_server_datadirs:
  - /var/lib/minio
# When set to true Ansible will create the minio_server_datadirs
minio_server_make_datadirs: true

# Minio server cluster node list. Set when deploying distributed multi-node setup (i.e. - https://minio-0{1...5}.example.com:9000/mnt/minio-disk{1...2})
minio_server_cluster_nodes: [ ]

# Additional environment variables to be set in minio server environment
minio_server_env_extra: ""

# Additional Minio server CLI options
minio_server_opts: ""

# Switches to enable/disable the Minio server and/or Minio client installation
minio_install_server: true
minio_install_client: true

# Default root user. Provide at role execution
minio_root_user: minio
minio_root_password: "changeme!"

# Optional: Path to certificate and private key that should be copied to the server
copy_minio_ssl_certificate: ""
copy_minio_ssl_certificate_key: ""

# Optional: Path to existing certificate and private key files on remote server
existing_minio_ssl_certificate: ""
existing_minio_ssl_certificate_key: ""
```

Dependencies
------------

- No Dependencies

Example Inventory
-----------------

```yml
all:
  children:
    minio:
      hosts:
        minio-01.example.com:
          ansible_host: 10.199.34.152
        minio-02.example.com:
          ansible_host: 10.199.34.153
        minio-03.example.com:
          ansible_host: 10.199.34.154
        minio-04.example.com:
          ansible_host: 10.199.34.155
        minio-05.example.com:
          ansible_host: 10.199.34.156
```

Example Playbook
----------------

```yml
# Example: Configure Minio single-node setup without ssl certificates
- hosts: minio
  become: yes
  gather_facts: yes
  roles:
    - role: minio
      minio_root_user: minioadmin
      minio_root_password: "{{ vault_minio_root_password }}"
```

```yml
# Example: Configure Minio distributed multi-node setup with SSL certificates
- hosts: minio
  become: yes
  gather_facts: yes
  roles:
    - role: minio
      minio_server_release: "RELEASE.2022-11-08T05-27-07Z"
      minio_client_release: "RELEASE.2022-10-09T21-10-59Z"
      minio_bind_console_addr: ":9001"
      minio_bind_api_addr: ":9000"
      minio_server_datadirs:
        - /mnt/minio-data1
        - /mnt/minio-data2
      minio_server_make_datadirs: true
      minio_server_cluster_nodes:
        - https://minio-0{1...5}.example.com:9000/mnt/minio-data{1...2}
      minio_root_user: minioadmin
      minio_root_password: "{{ vault_minio_root_password }}"
      copy_minio_ssl_certificate: "{{ inventory_dir }}/group_vars/secrets/minio-netbox.test.2ln.mueller.de.crt"
      copy_minio_ssl_certificate_key: "{{ inventory_dir }}/group_vars/secrets/minio-netbox.test.2ln.mueller.de.key"
      copy_minio_ssl_ca: "{{ inventory_dir }}/group_vars/secrets/ca.crt"
```
License
-------

GPL-3.0-only

Author Information
------------------

Jakob Oettinger & Oleg Franko
