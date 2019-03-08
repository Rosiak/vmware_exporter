# vmware_exporter

VMware vCenter Exporter for Prometheus.

Get VMware vCenter information:
- Basic VM and Host metrics
- Current number of active snapshots
- Datastore size and other stuff
- Snapshot Unix timestamp creation date

## Badges
![Docker Stars](https://img.shields.io/docker/stars/pryorda/vmware_exporter.svg)
![Docker Pulls](https://img.shields.io/docker/pulls/pryorda/vmware_exporter.svg)
![Docker Automated](https://img.shields.io/docker/automated/pryorda/vmware_exporter.svg)

[![Travis Build Status](https://travis-ci.org/pryorda/vmware_exporter.svg?branch=master)](https://travis-ci.org/pryorda/vmware_exporter)
![Docker Build](https://img.shields.io/docker/build/pryorda/vmware_exporter.svg)
[![Join the chat at https://gitter.im/vmware_exporter/community](https://badges.gitter.im/vmware_exporter/community.svg)](https://gitter.im/vmware_exporter/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

## Usage

- Install with `$ python setup.py install` or via pip `$ pip install vmware_exporter`. The docker command below is preferred.
- Create `config.yml` based on the configuration section. Some variables can be passed as environment variables
- Run `$ vmware_exporter -c /path/to/your/config`
- Go to http://localhost:9272/metrics?vsphere_host=vcenter.company.com to see metrics

Alternatively, if you don't wish to install the package, run it using `$ vmware_exporter/vmware_exporter.py` or use the following docker command:

```
docker run -it --rm  -p 9272:9272 -e VSPHERE_USER=${VSPHERE_USERNAME} -e VSPHERE_PASSWORD=${VSPHERE_PASSWORD} -e VSPHERE_HOST=${VSPHERE_HOST} -e VSPHERE_IGNORE_SSL=True --name vmware_exporter pryorda/vmware_exporter
```

### Configuration and limiting data collection

Only provide a configuration file if enviroment variables are not used. If you do plan to use a configuration file, be sure to override the container entrypoint or add -c config.yml to the command arguments.

If you want to limit the scope of the metrics gathered, you can update the subsystem under `collect_only` in the config section, e.g. under `default`, or by using the environment variables:

    collect_only:
        vms: False
        vmguests: True
        datastores: True
        hosts: True
        snapshots: True

This would only connect datastores and hosts.

You can have multiple sections for different hosts and the configuration would look like:
```
default:
    vsphere_host: "vcenter"
    vsphere_user: "user"
    vsphere_password: "password"
    ignore_ssl: False
    collect_only:
        vms: True
        vmguests: True
        datastores: True
        hosts: True
        snapshots: True

esx:
    vsphere_host: vc.example2.com
    vsphere_user: 'root'
    vsphere_password: 'password'
    ignore_ssl: True
    collect_only:
        vms: False
        vmguests: True
        datastores: False
        hosts: True
        snapshots: True

limited:
    vsphere_host: slowvc.example.com
    vsphere_user: 'administrator@vsphere.local'
    vsphere_password: 'password'
    ignore_ssl: True
    collect_only:
        vms: False
        vmguests: False
        datastores: True
        hosts: False
        snapshots: False

```
Switching sections can be done by adding ?section=limited to the URL.

#### Environment Variables
| Variable                      | Precedence             | Defaults | Description                                      |
| ---------------------------- | ---------------------- | -------- | --------------------------------------- |
| `VSPHERE_HOST`               | config, env, get_param | n/a      | vsphere server to connect to   |
| `VSPHERE_USER`               | config, env            | n/a      | User for connecting to vsphere |
| `VSPHERE_PASSWORD`           | config, env            | n/a      | Password for connecting to vsphere |
| `VSPHERE_IGNORE_SSL`         | config, env            | False    | Ignore the ssl cert on the connection to vsphere host |
| `VSPHERE_COLLECT_HOSTS`      | config, env            | True     | Set to false to disable collection of host metrics |
| `VSPHERE_COLLECT_DATASTORES` | config, env            | True     | Set to false to disable collection of datastore metrics |
| `VSPHERE_COLLECT_VMS`        | config, env            | True     | Set to false to disable collection of virtual machine metrics |
| `VSPHERE_COLLECT_VMGUESTS`   | config, env            | True     | Set to false to disable collection of virtual machine guest metrics |
| `VSPHERE_COLLECT_SNAPSHOTS`  | config, env            | True     | Set to false to disable collection of snapshot metrics |

You can create new sections as well, with very similiar variables. For example, to create a `limited` section you can set:

| Variable                      | Precedence             | Defaults | Description                                      |
| ---------------------------- | ---------------------- | -------- | --------------------------------------- |
| `VSPHERE_LIMITED_HOST`               | config, env, get_param | n/a      | vsphere server to connect to   |
| `VSPHERE_LIMITED_USER`               | config, env            | n/a      | User for connecting to vsphere |
| `VSPHERE_LIMITED_PASSWORD`           | config, env            | n/a      | Password for connecting to vsphere |
| `VSPHERE_LIMITED_IGNORE_SSL`         | config, env            | False    | Ignore the ssl cert on the connection to vsphere host |
| `VSPHERE_LIMITED_COLLECT_HOSTS`      | config, env            | True     | Set to false to disable collection of host metrics |
| `VSPHERE_LIMITED_COLLECT_DATASTORES` | config, env            | True     | Set to false to disable collection of datastore metrics |
| `VSPHERE_LIMITED_COLLECT_VMS`        | config, env            | True     | Set to false to disable collection of virtual machine metrics |
| `VSPHERE_LIMITED_COLLECT_VMGUESTS`   | config, env            | True     | Set to false to disable collection of virtual machine guest metrics |
| `VSPHERE_LIMITED_COLLECT_SNAPSHOTS`  | config, env            | True     | Set to false to disable collection of snapshot metrics |

You need to set at least `VSPHERE_SECTIONNAME_USER` for the section to be detected.

### Prometheus configuration

You can use the following parameters in the Prometheus configuration file. The `params` section is used to manage multiple login/passwords.

```
  - job_name: 'vmware_vcenter'
    metrics_path: '/metrics'
    static_configs:
      - targets:
        - 'vcenter.company.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9272

  - job_name: 'vmware_esx'
    metrics_path: '/metrics'
    file_sd_configs:
      - files:
        - /etc/prometheus/esx.yml
    params:
      section: [esx]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9272

# Example of Multiple vCenter usage per #23

- job_name: vmware_export
    metrics_path: /metrics
    static_configs:
    - targets:
      - vcenter01
      - vcenter02
      - vcenter03
    relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: exporter_ip:9272
```

## Current Status

- vCenter and vSphere 5.5/6.0/6.5/6.7 have been tested.
- VM information, Snapshot, Host and Datastore basic information is exported, i.e:
```
# HELP vmware_vm_power_state VMWare VM Power state (On / Off)
# TYPE vmware_vm_power_state gauge
vmware_vm_power_state{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",vm_name="My Super Virtual Machine"} 1.0

# HELP vmware_vm_boot_timestamp_seconds VMWare VM boot time in seconds
# TYPE vmware_vm_boot_timestamp_seconds gauge
vmware_vm_boot_timestamp_seconds{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",vm_name="My Super Virtual Machine"} 1234567890.123456

# HELP vmware_vm_num_cpu VMWare Number of processors in the virtual machine
# TYPE vmware_vm_num_cpu gauge
vmware_vm_num_cpu{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",vm_name="My Super Virtual Machine"} 1.0

# HELP vmware_vm_memory_max VMWare VM Memory Max availability in Mbytes
# TYPE vmware_vm_memory_max gauge
vmware_vm_memory_max{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",vm_name="My Super Virtual Machine"} 1024.0

# HELP vmware_vm_guest_disk_free Disk metric per partition
# TYPE vmware_vm_guest_disk_free gauge
vmware_vm_guest_disk_free{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",partition="/",vm_name="My Super Virtual Machine"} 123456789.0

# HELP vmware_vm_guest_disk_capacity Disk capacity metric per partition
# TYPE vmware_vm_guest_disk_capacity gauge
vmware_vm_guest_disk_capacity{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",partition="/",vm_name="My Super Virtual Machine"} 1234567890.0

# HELP vmware_vm_guest_tools_running_status VM tools running status
# TYPE vmware_vm_guest_tools_running_status gauge
vmware_vm_guest_tools_running_status{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",tools_status="toolsOk",vm_name="My Super Virtual Machine"} 1.0

# HELP vmware_vm_guest_tools_version VM tools version
# TYPE vmware_vm_guest_tools_version gauge
vmware_vm_guest_tools_version{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",tools_version="123456789",vm_name="My Super Virtual Machine"} 1.0

# HELP vmware_vm_guest_tools_version_status VM tools version status
# TYPE vmware_vm_guest_tools_version_status gauge
vmware_vm_guest_tools_version_status{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",tools_version_status="guestToolsCurrent",vm_name="My Super Virtual Machine"} 1.0




# HELP vmware_datastore_capacity_size VMware Datastore capacity in bytes
# TYPE vmware_datastore_capacity_size gauge
vmware_datastore_capacity_size{dc_name="dc1",ds_cluster="ds1",ds_name="ESX1-LOCAL"} 67377299456.0

# HELP vmware_datastore_freespace_size VMware Datastore freespace in bytes
# TYPE vmware_datastore_freespace_size gauge
vmware_datastore_freespace_size{dc_name="dc1",ds_cluster="ds1",ds_name="ESX1-LOCAL"} 66349694976.0

# HELP vmware_datastore_uncommited_size VMware Datastore uncommitted in bytes
# TYPE vmware_datastore_uncommited_size gauge
vmware_datastore_uncommited_size{dc_name="dc1",ds_cluster="ds1",ds_name="ESX1-LOCAL"} 0.0

# HELP vmware_datastore_provisoned_size VMware Datastore provisoned in bytes
# TYPE vmware_datastore_provisoned_size gauge
vmware_datastore_provisoned_size{dc_name="dc1",ds_cluster="ds1",ds_name="ESX1-LOCAL"} 1027604480.0

# HELP vmware_datastore_hosts VMware Hosts number using this datastore
# TYPE vmware_datastore_hosts gauge
vmware_datastore_hosts{dc_name="dc1",ds_cluster="ds1",ds_name="ESX1-LOCAL"} 1.0

# HELP vmware_datastore_vms VMware Virtual Machines number using this datastore
# TYPE vmware_datastore_vms gauge
vmware_datastore_vms{dc_name="dc1",ds_cluster="ds1",ds_name="ESX1-LOCAL"} 0.0

# HELP vmware_datastore_maintenance_mode VMWare datastore maintenance mode (normal / inMaintenance / enteringMaintenance)
# TYPE vmware_datastore_maintenance_mode gauge
vmware_datastore_maintenance_mode{dc_name="dc1",ds_cluster="ds1",ds_name="ESX1-LOCAL",mode="normal"} 1.0

# HELP vmware_datastore_type VMWare datastore type (VMFS, NetworkFileSystem, NetworkFileSystem41, CIFS, VFAT, VSAN, VFFS)
# TYPE vmware_datastore_type gauge
vmware_datastore_type{dc_name="dc1",ds_cluster="ds1",ds_name="ESX1-LOCAL",ds_type="VMFS"} 1.0

# HELP vmware_datastore_accessible VMWare datastore accessible (true / false)
# TYPE vmware_datastore_accessible gauge
vmware_datastore_accessible{dc_name="dc1",ds_cluster="ds1",ds_name="ESX1-LOCAL"} 1.0




# HELP vmware_host_power_state VMware Host Power state (On / Off)
# TYPE vmware_host_power_state gauge
vmware_host_power_state{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com"} 1.0

# HELP vmware_host_connection_state VMWare Host connection state (connected / disconnected / notResponding)
# TYPE vmware_host_connection_state gauge
vmware_host_connection_state{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",state="connected"} 1.0

# HELP vmware_host_maintenance_mode VMWare Host maintenance mode (true / false)
# TYPE vmware_host_maintenance_mode gauge
vmware_host_maintenance_mode{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com"} 0.0

# HELP vmware_host_boot_timestamp_seconds VMWare Host boot time in seconds
# TYPE vmware_host_boot_timestamp_seconds gauge
vmware_host_boot_timestamp_seconds{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com"} 123456789.0

# HELP vmware_host_cpu_usage VMWare Host CPU usage in Mhz
# TYPE vmware_host_cpu_usage gauge
vmware_host_cpu_usage{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com"} 12345.0

# HELP vmware_host_cpu_max VMWare Host CPU max availability in Mhz
# TYPE vmware_host_cpu_max gauge
vmware_host_cpu_max{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com"} 12345.0

# HELP vmware_host_num_cpu VMWare Number of processors in the Host
# TYPE vmware_host_num_cpu gauge
vmware_host_num_cpu{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com"} 12.0

# HELP vmware_host_memory_usage VMWare Host Memory usage in Mbytes
# TYPE vmware_host_memory_usage gauge
vmware_host_memory_usage{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com"} 12345.0

# HELP vmware_host_memory_max VMWare Host Memory Max availability in Mbytes
# TYPE vmware_host_memory_max gauge
vmware_host_memory_max{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com"} 12345.0




# HELP vmware_vm_snapshots VMWare current number of existing snapshots
# TYPE vmware_vm_snapshots gauge
vmware_vm_snapshots{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",vm_name="My Super Virtual Machine"} 1.0

# HELP vmware_vm_snapshot_timestamp_seconds VMWare Snapshot creation time in seconds
# TYPE vmware_vm_snapshot_timestamp_seconds gauge
vmware_vm_snapshot_timestamp_seconds{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",vm_name="My Super Virtual Machine",vm_snapshot_name="12345"} 12345.0




# HELP vmware_vm_cpu_ready_summation vmware_vm_cpu_ready_summation
# TYPE vmware_vm_cpu_ready_summation gauge
vmware_vm_cpu_ready_summation{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",vm_name="My Super Virtual Machine"} 12.0

# HELP vmware_vm_cpu_usage_average vmware_vm_cpu_usage_average
# TYPE vmware_vm_cpu_usage_average gauge
vmware_vm_cpu_usage_average{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",vm_name="My Super Virtual Machine"} 12345.0

# HELP vmware_vm_cpu_usagemhz_average vmware_vm_cpu_usagemhz_average
# TYPE vmware_vm_cpu_usagemhz_average gauge
vmware_vm_cpu_usagemhz_average{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",vm_name="My Super Virtual Machine"} 12345.0

# HELP vmware_vm_disk_usage_average vmware_vm_disk_usage_average
# TYPE vmware_vm_disk_usage_average gauge
vmware_vm_disk_usage_average{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",vm_name="My Super Virtual Machine"} 12.0

# HELP vmware_vm_disk_read_average vmware_vm_disk_read_average
# TYPE vmware_vm_disk_read_average gauge
vmware_vm_disk_read_average{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",vm_name="My Super Virtual Machine"} 0.0

# HELP vmware_vm_disk_write_average vmware_vm_disk_write_average
# TYPE vmware_vm_disk_write_average gauge
vmware_vm_disk_write_average{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",vm_name="My Super Virtual Machine"} 12.0

# HELP vmware_vm_mem_usage_average vmware_vm_mem_usage_average
# TYPE vmware_vm_mem_usage_average gauge
vmware_vm_mem_usage_average{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",vm_name="My Super Virtual Machine"} 12.0

# HELP vmware_vm_net_received_average vmware_vm_net_received_average
# TYPE vmware_vm_net_received_average gauge
vmware_vm_net_received_average{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",vm_name="My Super Virtual Machine"} 12.0

# HELP vmware_vm_net_transmitted_average vmware_vm_net_transmitted_average
# TYPE vmware_vm_net_transmitted_average gauge
vmware_vm_net_transmitted_average{cluster_name="cluster1",dc_name="dc1",host_name="esx1.company.com",vm_name="My Super Virtual Machine"} 12.0
```

## References

The VMware exporter uses theses libraries:
- [pyVmomi](https://github.com/vmware/pyvmomi) for VMware connection
- Prometheus [client_python](https://github.com/prometheus/client_python) for Prometheus supervision
- [Twisted](http://twistedmatrix.com/trac/) for HTTP server

The initial code is mainly inspired by:
- https://www.robustperception.io/writing-a-jenkins-exporter-in-python/
- https://github.com/vmware/pyvmomi-community-samples
- https://github.com/jbidinger/pyvmomi-tools

Forked from https://github.com/rverchere/vmware_exporter. I removed the fork so that I could do searching and everything.

## Maintainer

Daniel Pryor [pryorda](https://github.com/pryorda)

## License

See LICENSE file
