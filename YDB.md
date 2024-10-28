Step to setup Minio with sample YDB
------------------------

1) Download this role https://github.com/elabpro/ansible-role-minio/tree/main

2) Put it into ansible (folder _roles_)

3) create a playbook minio.yml

```yml
- name: "Install Minio"
  hosts: all
  become: yes
  gather_facts: true
  
  roles:
    - role: ansible-role-minio
      minio_bind_console_addr: ":9001"
      minio_bind_api_addr: ":9000"
      minio_server_datadirs:
        - /mnt/minio-data1
        - /mnt/minio-data2
        - /mnt/minio-data3
      minio_server_make_datadirs: true
      minio_server_cluster_nodes:
        - https://ydb-node0{1...3}.ru-central1.internal:9000/mnt/minio-data{1...3}
      minio_root_user: minioadmin
      minio_root_password: PASSWORD
      copy_minio_ssl_certificate: "../TLS/CA/certs/2024-10-11_20-33-18/{{ inventory_hostname }}/node.crt"
      copy_minio_ssl_certificate_key: "../TLS/CA/certs/2024-10-11_20-33-18/{{ inventory_hostname }}/node.key"
      copy_minio_ssl_ca: "../TLS/CA/certs/2024-10-11_20-33-18/ca.crt"

```
4) Deploy minio

5) Configure minio on a server
    - create alias 
    mc alias set mycloud https://ydb-node01.ru-central1.internal:9000 minioadmin PASSWORD
    - add user
    mc admin user add mycloud USER1 SECRETFROUSER1
    - check cluster
    mc admin info mycloud
    - create bucket
    mc mb mycloud/ydbbucket
    - add policy
    mc admin policy create mycloud user1-policy ~/user1-policy.json
    - assign policy to the user
    mc admin policy attach mycloud user1-policy --user USER1

```text
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "s3:PutBucketPolicy",
        "s3:GetBucketPolicy",
        "s3:DeleteBucketPolicy",
        "s3:ListAllMyBuckets",
        "s3:ListBucket"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::ydbbucket"
      ],
      "Sid": ""
    },
    {
      "Action": [
        "s3:AbortMultipartUpload",
        "s3:DeleteObject",
        "s3:GetObject",
        "s3:ListMultipartUploadParts",
        "s3:PutObject"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::ydbbucket/*"
      ],
      "Sid": ""
    }
  ]
}
```