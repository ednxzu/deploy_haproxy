deploy_haproxy
=========
> This repository is only a mirror. Development and testing is done on a private gitea server.

This role lets you install and configure haproxy on debian-based systems. It can either deploy it natively, or within a docker container.

Requirements
------------

None.

Role Variables
--------------
Available variables are listed below, along with default values. A sample file for the default values is available in `default/deploy_haproxy.yml.sample` in case you need it for any `group_vars` or `host_vars` configuration.

```yaml
deploy_haproxy_deploy_method: host # by default, set to host
```
This variable defines the method of deployment of vault. The `host` method installs the binary directly on the host, and runs vault as a systemd service. The `docker` method install vault as a docker container.

```yaml
deploy_haproxy_version: 2.8 # by default, set to 2.8
```
This variable sets the version of haproxy to install. Please note that not all versions are available on all systems. Please refer to [this site](https://haproxy.debian.net/) to check if your os/version combo is supported. This is not a problem if using `deploy_haproxy_deploy_method: docker`, as the underlying os won't matter.

```yaml
deploy_haproxy_env_variables: # by default, set to {}
  ENV_VAR: value
```
This value is a list of key/value that will populate the haproxy.env (for host deployment method) or /etc/default/haproxy (for docker deploy method) file.

```yaml
deploy_haproxy_start_service: true # by default, set to true
```
This variable defines if the haproxy service should be started once it has been configured. This is usefull in case you're using this role to build golden images, in which case you might want to only enable the service, to have it start on the next boot (when the image is launched).

```yaml
deploy_haproxy_cert_dir: "" # by default, set to ""
```
This variable lets you specify the directory (local) from which to copy TLS certificates, if you choose to enable TLS on your endpoint(s). If set to anything other than `""`, therole will try to copy from your local machine to `{{ deploy_haproxy_cert_dir_dst }}` (which will be automatically created). `{{ deploy_haproxy_cert_dir_dst }}` is set to `"{{ deploy_haproxy_chroot }}/certs"`, so you should reference that as the path inside your haproxy configuration when pointing to TLS certificates copied by this role.

> **Warning**
> haproxy expect for each certificate, a <cert_name>.pem file, as well as a <cert_name>.pem.key file. You should copy both if you choose to use the `deploy_haproxy_cert_dir` directory to import your certificates.

```yaml
deploy_haproxy_extra_container_volumes: [] # by default, set to []
```
This variable lets you defines more volumes to mount inside the container when using the docker deployment method. This is a list of string in the form: "/path/on/host:/path/on/container". These volumes will not be created/checked before being mounted, so they need to exist prior to running this role.

```yaml
deploy_haproxy_global: # by default, set to the following
  - log /dev/log local0
  - log /dev/log local1 notice
  - stats socket {{ deploy_haproxy_socket }} level admin
  - chroot {{ deploy_haproxy_chroot }}
  - daemon
  - description hashistack haproxy
```
This variable lets you specify all the configuration parameters to pass to the `global` block within the `haproxy.cfg` file. It is a list of string, each string is a configuration parameter.

```yaml
deploy_haproxy_defaults:
  - log global
  - mode http
  - option httplog
  - option dontlognull
  - timeout connect 5000
  - timeout client  5000
  - timeout server  5000
```
This variable lets you specify all the configuration parameters to pass to the `defaults` block within the `haproxy.cfg` file. It is a list of string, each string is a configuration parameter.

```yaml
deploy_haproxy_frontends: [] # by default, set to []
```
This variable lets you specify multiple `frontend` blocks within the `haproxy.cfg` file. One frontend block would look like:

```yaml
deploy_haproxy_frontends:
  - name: default
    options:
      - description default frontend
      - mode http
      - bind :80
      - default_backend default
```
The `name` will be used as the block's name, while the `options` will be passed as parameters to this block. The `options` key is a list of string, each string is a configuration parameter.

```yaml
deploy_haproxy_backends: [] # by default, set to []
```
This variable lets you specify multiple `backend` blocks within the `haproxy.cfg` file. One backend block would look like:

```yaml
deploy_haproxy_frontends:
  - name: default
    options:
      - description default backend
      - option forwardfor
      - option httpchk
      - http-check send meth GET uri /
      - server srv_nginx1 172.17.0.4:80 check inter 5s
      - server srv_nginx2 172.17.0.3:80 check inter 5s
```
The `name` will be used as the block's name, while the `options` will be passed as parameters to this block. The `options` key is a list of string, each string is a configuration parameter.

```yaml
deploy_haproxy_listen: # by default, set to the following
  - name: monitoring
    options:
      - bind :9000
      - mode http
      - option httpchk
      - stats enable
      - stats uri /stats
      - stats refresh 30s
      - stats show-desc
      - stats show-legends
      - stats auth admin:password
      - http-check send meth GET uri /health ver HTTP/1.1 hdr Host localhost
      - http-check expect status 200
      - acl health_check_ok nbsrv() ge 1
      - monitor-uri /health
      - http-request use-service prometheus-exporter if { path /metrics }
```
This variable lets you specify multiple `listen` blocks within the `haproxy.cfg` file. By default, this role exposes on **port 9000**, the haproxy dashboard, on `/stats`, and health endpoint (returning HTTP 200 while haproxy is alive) on `/health`, and the prometheus exporter on `/metrics`.

Dependencies
------------

`ednxzu.docker_systemd_service` if installing haproxy in a container.

Example Playbook
----------------

```yaml
# calling the role inside a playbook with either the default or group_vars/host_vars
- hosts: servers
  roles:
    - ednz_cloud.deploy_haproxy
```

License
-------

MIT / BSD

Author Information
------------------

This role was created by Bertrand Lanson in 2024.
