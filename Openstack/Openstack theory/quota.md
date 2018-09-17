# Phần này giới thiệu về quota trong openstack

**Quota**: Được sử dụng để giới hạn tài nguyên của các service trong openstack

- Kiểm tra các quota default hiện có: `openstack quota show --default`

- Kiểm tra quota default của neutron: `neutron quota-default-show`

- Kiểm tra quota default của nova: `nova  quota-defaults`

- Kiểm tra quota default của cinder: `cinder quota-defaults show`

- Set quota cho project:

**Cú pháp set quota**
```
openstack quota set
    # Compute settings
    [--cores <num-cores>]
    [--fixed-ips <num-fixed-ips>]
    [--floating-ips <num-floating-ips>]
    [--injected-file-size <injected-file-bytes>]
    [--injected-files <num-injected-files>]
    [--instances <num-instances>]
    [--key-pairs <num-key-pairs>]
    [--properties <num-properties>]
    [--ram <ram-mb>]
    [--server-groups <num-server-groups>]
    [--server-group-members <num-server-group-members>]

    # Block Storage settings
    [--backups <new-backups>]
    [--backup-gigabytes <new-backup-gigabytes>]
    [--gigabytes <new-gigabytes>]
    [--per-volume-gigabytes <new-per-volume-gigabytes>]
    [--snapshots <new-snapshots>]
    [--volumes <new-volumes>]
    [--volume-type <volume-type>]

    # Network settings
    [--floating-ips <num-floatingips>]
    [--secgroup-rules <num-security-group-rules>]
    [--secgroups <num-security-groups>]
    [--networks <num-networks>]
    [--subnets <num-subnets>]
    [--ports <num-ports>]
    [--routers <num-routers>]
    [--rbac-policies <num-rbac-policies>]
    [--vips <num-vips>]
    [--subnetpools <num-subnetpools>]
    [--members <num-members>]
    [--health-monitors <num-health-monitors>]

    <project>
    
```

```
openstack quota set --class --'tên trường muốn set quota' 'số lượng quota muốn set' default
```

Ví dụ: `openstack quota set --class --instances 5 default`  (*set cho trường instance với số lượng quota là 5 trong project default*)

- Update quota : 
```
nova quota-class-update --'tên trường muốn update' 'số quota cần update' default
```
Ví dụ: `nova quota-class-update --instances 3 default` 

- 
