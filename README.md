kubeadm
=========

`kubeadm` is an ansible role to deploy kubernetes cluster based on kubeadm.
to learn more about kubeadm, follow the official doc: [kubeadm docs](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/).


**NOTE**: **Be sure you have read at least the `Requirements`and `Dependencies`README's sections before using the role**.

Requirements
------------
The module `community.general.modprobe` is called in this role.
in order to make this works properly, you have to install the `community.general`collection of ansible. the command to achieve this is right below.

```shell
$ ansible-galaxy collection install community.general
```

Role Variables
--------------
This section describe all settable variables for this role

### Role variables in `default/main.yml`
---------------------------------------

systems packages variables (`defaults/main.yml`).
```yaml
kubeadm_system_package:
  - apt-transport-https
  - ca-certificates
  - curl
  - jq
  - python3
  - python3-pip
  - python3-kubernetes
```

kubeadm required packages and versions (`defaults/main.yml`).
```yaml
kubeadm_packages:
  - kubectl
  - kubelet=1.28.0-00
  - kubeadm=1.28.0-00
```

kubernetes version  (`defaults/main.yml`).
```yaml
kubeadm_kubernetes_version: "v1.27.0"
```

system users for kubectl. kubectl cli will be configured for those users (`defaults/main.yml`).
```yaml
kubeadm_kubectl_users:
  - name: ubuntu
    group: ubuntu
```

kubernetes networks modules (`defaults/main.yml`).
```yaml
kubeadm_network_modules:
  - overlay
  - br_netfilter
```

kubernetes required network modules parameters (`defaults/main.yml`).
```yaml
kubeadm_network_parameters:
  - key: net.bridge.bridge-nf-call-iptables
    value: 1
  - key: net.bridge.bridge-nf-call-ip6tables
    value: 1
  - key: net.ipv4.ip_forward
    value: 1
```

kubeadm configuration file (`defaults/main.yml`).
```yaml
kubeadm_config_file: /tmp/kubeadm-config.yaml
```

kubernetes configuration directory (`defaults/main.yml`).
```yaml
kubeadm_dir: "/etc/kubernetes"
```

kubernetes pki directory (`defaults/main.yml`).
```yaml
kubeadm_certificates_dir: "{{ kubeadm_dir }}/pki"
```

kubeadm registry variable (`defaults/main.yml`)
```yaml
kubeadm_image_repository: registry.k8s.io
```

Kubeadm configuration file (`defaults/main.yml`)
```yaml
kubeadm_bootsrap_tokens_config:
  tokens:
    - token: "783bde.3f89s0fje9f38fhf"
      description: "kubeadm bootstrap token"
      ttl: "24h"
      usages:
        - signing
        - authentication
      groups:
        - system:bootstrappers:kubeadm:default-node-token
```

kubeadm configuration for nodeRegistration (`defaults/main.yml`).
```yaml
kubeadm_node_registration_config:
  taints:
    - key: "kubeadmNode"
      value: "value1"
      effect: "NoSchedule"
  kubelet_extra_args:
    - key: v
      value: 4
  ignore_preflight_errors:
    - IsPrivilegedUser
    - Swap
  skip_phases: []
  image_pull_policy: IfNotPresent
  certificate_key: ""
```

kubeadm configuration for etcd (`defaults/main.yml`).
```yaml
kubeadm_etcd_config:
  local_etcd:
    image_repo: "{{ kubeadm_default_image_repo | default("registry.k8s.io") }}"
    image_tag: "3.2.24"
    data_dir: "/var/lib/etcd"
    extra_args:
      - key: listen-client-urls
        value: "http://localhost:2379"
    server_cert_sans: []
    peer_cert_sans: []
  external_etcd: []
    # endpoints:
    #   - "10.100.0.1:2379"
    #   - "10.100.0.2:2379"
    # ca_file: "/etcd/kubernetes/pki/etcd/etcd-ca.crt"
    # cert_file: "/etcd/kubernetes/pki/etcd/etcd.crt"
    # key_file: "/etcd/kubernetes/pki/etcd/etcd.key"
```

kubeadm configuration for the cluster's network (`defaults/main.yml`).
```yaml
kubeadm_networking_config:
  cluster_network:
    service_subnet: "10.96.0.0/16"
    pod_subnet: "10.244.0.0/24"
    dns_domain: cluster.local
  apiserver_advertise_address: ""
  apiserver_bind_port: 6443
  apiserver_endpoint: ""
```

kubadm configuration for kubernetes apiserver (`defaults/main.yml`).
```yaml
kubeadm_apiserver_config:
  apiserver:
    extra_args:
      - key: authorization-mode
        value: "Node,RBAC"
    extra_volumes: []
      # - name: some-volume
      #   host_path: "/etc/"
      #   mount_path: "/etc/"
      #   read_only: true
      #   path_type: DirectoryOrCreate
    cert_sans: []
    timeout_for_controlplane: "4m0s"
```

kubeadm configuration for kubernetes controller manager (`defaults/main.yml`).
```yaml
kubeadm_controllermanager_config:
  controllermanager:
    extra_args:
      - key: node-cidr-mask-size
        value: 20
    extra_volumes: []
      # - name: some-volume
      #   host_path: "/etc/"
      #   mount_path: "/etc/"
      #   read_only: true
      #   path_type: Directory
```

kubeadm configuration for kubernetes scheduler (`defaults/main.yml`).
```yaml
kubeadm_scheduler_config:
  scheduler:
    extra_args:
      - key: address
        value: "10.100.0.1"
    extra_volumes: []
      # - name: some-volume
      #   host_path: "/etc/"
      #   mount_path: "/etc/"
      #   read_only: true
      #   path_type: Directory
```

kubeadm configuration for node to join the cluster(`defaults/main.yml`).
```yaml
kubeadm_join_config:
  join:
    registration:
      name: "{{ ansible_hostname }}"
      unsafe_skip_ca_verification: false
      cri_socket: "{{ kubeadm_container_runtime_socket }}"
      taints: []
      kubelet_extra_args:
        - key: enable-controller-attach-detach
          value: false
        - key: node-labels
          value: node
    skip_phases: []
```

kubeadm configuration for kubelet (`defaults/main.yml`).
```yaml
kubeadm_kubelet_config:
  kubelet:
    cgroup_driver: systemd
```

kubeadm configuration for container network interface (cni) (`defaults/main.yml`).
```yaml
kubeadm_cni_name: "canal"
kubeadm_cni_calico_version: 3.26.4
kubeadm_cni_canal_version: 3.26.4
```

### Role variables in `vars/main.yml`
------------------------------------

variables for compatible OS distros (`vars/main.yml`).
```yaml
kubeadm_compatible_os_distros:
  - debian
  - redhat
```

variables for minimum hardawares requirements (`vars/main.yml`)

```yaml
kubeadm_hardwares_min_requirements:
  ram: 2
  cpu: 2
```

kubeadm installation parameters (`vars/main.yml`)
```yaml
kubeadm_google_signing_key_url: "https://packages.cloud.google.com/apt/doc/apt-key.gpg"
kubeadm_google_signing_key: "/etc/apt/keyrings/kubernetes-archive-keyring.gpg"

kubeadm_k8s_apt_repo: "deb https://apt.kubernetes.io/ kubernetes-xenial main"
kubeadm_k8s_apt_repo_file: "/etc/apt/sources.list.d/kubernetes.list"
```

kubeadm configuration's file api (`vars/main.yml`)
```yaml
kubeadm_config_array: []
kubeadm_config: |
  {{
    kubeadm_config_array |
    combine(kubeadm_bootsrap_tokens_config, list_merge='append') |
    combine(kubeadm_node_registration_config, list_merge='append') |
    combine(kubeadm_etcd_config, list_merge='append') |
    combine(kubeadm_networking_config, list_merge='append') |
    combine(kubeadm_apiserver_config, list_merge='append') |
    combine(kubeadm_controllermanager_config, list_merge='append') |
    combine(kubeadm_scheduler_config, list_merge='append') |
    combine(kubeadm_kubelet_config, list_merge='append') |
    combine(kubeadm_join_config, list_merge='append')
  }}
kubeadm_config_apiversion: kubeadm.k8s.io/v1beta3
kubeadm_config_init_apiversion: "{{ kubeadm_config_apiversion }}"
kubeadm_config_kubelet_apiversion: "kubelet.config.k8s.io/v1beta1"
kubeadm_config_kubeproxy_apiversion: "kubeproxy.config.k8s.io/v1alpha1"
kubeadm_config_kind_init: InitConfiguration
kubeadm_config_kind_cluster: ClusterConfiguration
kubeadm_config_kind_kubelet: KubeletConfiguration
kubeadm_config_kind_kubeproxy: KubeProxyConfiguration
kubeadm_config_kind_join: JoinConfiguration
```

Dependencies
------------
As say on top, this role is focus on deploying a kubernetes cluster based on kubeadm tool.

The role `kubeadm`does not install any **`container runtime`** on machines.
you have have to find a way to install a container runtime (containerd, docker ...)

in our case, we have used the public role [geerlingguy/ansible-role-containerd](https://github.com/geerlingguy/ansible-role-containerd) to install a containerd as container runtime.

```yaml
- name: geerlinguy.containerd
  version: 1.3.1
  src: https://github.com/geerlingguy/ansible-role-containerd.git
  scm: git
```

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:
```yaml
- name: Deploy kubernetes cluster
  hosts: all
  roles:
    - kubeadm
```
License
-------

BSD

Author Information
------------------

+ Jean Emmanuel ASSAMOI **`DevSecOps engineer`**
