# Spark

[Apache Spark](https://spark.apache.org/) is a fast and general engine for big
data processing, with built-in modules for streaming, SQL, machine learning and
graph processing.

## Variables

- `spark_versions` - Spark versions to be installed (default:
  `["1.2.1", "1.2.2", "1.3.0", "1.3.1"]`).
- `spark_default_version` - Default Spark version to use (default: `1.3.0`).
- `spark_user` - Linux user to run Spark jobs (default: `centos`).
- `spark_install_dir` - Local directory where to put Spark binary and Spark
  client (default: `/usr/local/share/spark`).
- `spark_mesos_lib` - Path to `libmesos.so` (default:
  `/usr/local/lib/libmesos.so`).
- `zookeeper_master` - Hostname of Zookeeper master (default:
  `zookeeper.service.consul`).

## Example Playbook

    ---
    - hosts: mesos_followers
      gather_facts: no
      roles:
        - spark
