# prometheus_exporters

Cookbook to install and configure various Prometheus exporters on systems to be monitored by Prometheus.

Currently supported exporters are node, postgres, redis, snmp, and wmi. More may be added in the future. Please contact the author if you have specific requests.

All of the exporters are available as chef custom resources that can be instantiated from other cookbooks.


# Supports

Ubuntu 14 & 16 (and probably other Debian based distributions)

CentOS 6 and 7 (and probably other RHEL based distributions)

Windows Server 2012 & 2016 (wmi_exporter recipe only)

# Resources

## node_exporter

* `web_listen_address`, String, default: ':9100'
* `web_telemetry_path`, String, default: '/metrics'
* `log_level`, String, default: 'info'
* `log_format`, String, default: 'logger:stdout'
* `collectors_enabled`, Array
* `collector_textfile_directory`, String
* `collector_netdev_ignored_devices`, String
* `collector_diskstats_ignored_devices`, String
* `collector_filesystem_ignored_fs_types`, String
* `collector_filesystem_ignored_mount_point`, String
* `custom_options`, String

Use `custom_options` for your configuration if defined proterties are not satisfying your needs.

```ruby

listen_ip = '127.0.0.1'

node_exporter 'main' do
  web_listen_address "#{listen_ip}:9100"
  action [:enable, :start]
end
```

or just set

* `node['prometheus_exporters']['listen_interface']`
* `node['prometheus_exporters']['node']['collectors']`
* `node['prometheus_exporters']['node']['textfile_directory']`
* `node['prometheus_exporters']['node']['ignored_net_devs']`

and add `recipe['prometheus_exporters::node]` to your run_list.

## postgres_exporter

The postgres_exporter resource supports running multiple copies of PostgreSQL exporter the same system. This is useful if you have multiple copies of PostgreSQL running on the same system
(eg. different versions) or you are connecting to multiple remote PostgreSQL servers across the network.

* `instance_name` name of PostgreSQL exporter instance. (**name attribute**)
* `data_source_name` PostgreSQL connection string. E.g. `postgresql://login:password@hostname:port/dbname`
* `extend_query_path` Path to custom queries to run
* `log_format` If set use a syslog logger or JSON logging. Example: logger:syslog?appname=bob&local=7 or logger:stdout?json=true. Defaults to stderr.
* `log_level` Only log messages with the given severity or above. Valid levels: [debug, info, warn, error, fatal].
* `web_listen_address` Address to listen on for web interface and telemetry. (default "127.0.0.1:9187")
* `web_telemetry_path` Path under which to expose metrics. (default "/metrics")
* `user` System user to run exporter as. (default "postgres")

```ruby
postgres_exporter '9.5_main' do
  data_source_name 'postgresql://localhost:5432/example'
  user 'postgres'
end
```

## mysqld_exporter

The mysqld_exporter resource supports running multiple copies of the MySQL exporter on the same system.

* `instance_name` name of MySQL exporter instance. (**name attribute**)
* `data_source_name` MySQL connection string
* `config_my_cnf` Path to .my.cnf file to read MySQL credentials from. (default: ~/.my.cnf)
* `log_format` If set use a syslog logger or JSON logging. Example: logger:syslog?appname=bob&local=7 or logger:stdout?json=true. Defaults to stderr.
* `log_level` Only log messages with the given severity or above. Valid levels: [debug, info, warn, error, fatal].
* `web_listen_address` Address to listen on for web interface and telemetry. (default "127.0.0.1:9104")
* `web_telemetry_path` Path under which to expose metrics. (default "/metrics")
* `user` System user to run exporter as. (default "mysql")

```ruby
mysqld_exporter 'main' do
  data_source_name '/'
  config_my_cnf '~/.my/cnf'
  user 'mysql'
end
```

## wmi_exporter

Expects the Chocolatey package manager to already be installed.  This is up to individuals to provide by including the [Chocolatey cookbook](https://github.com/chocolatey/chocolatey-cookbook) in their own wrapper cookbooks.

* `version`, String, default: '0.2.7'
* `enabled_collectors`, String, default: 'cpu,cs,logical_disk,net,os,service,system'
* `listen_address`, String, default: '0.0.0.0'
* `listen_port`, String, default: '9182'
* `metrics_path`, Strin, default: '/metrics'

Use the given defaults or set the attributes...

* `node['prometheus_exporters']['wmi']['version']['listen_interface']`
* `node['prometheus_exporters']['wmi']['listen_address']`
* `node['prometheus_exporters']['wmi']['listen_port']`
* `node['prometheus_exporters']['wmi']['metrics_path']`

and add `recipe['prometheus_exporters::wmi]` to your run_list.

# Discovery

Each exporter will set an attribute when it's enabled, in the form of `node['prometheus_exporters'][exporter_name]['enabled']`. This makes it possible to search for
exporters within your environment using knife search or from within other cookbooks using a query such as:

```knife search node 'prometheus_exporters_node_enabled:true'```

This query will return all nodes with configured node exporters which can be used for automatically configuring Prometheus servers.

# Known Issues

* The snmp_exporter requires a configuration file that is usually created by a config generator. Currently this functionality must be provided by a wrapper cookbook.
