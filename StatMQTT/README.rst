.. contents:: Table of Contents

=======
Summary
=======

StatMQTT: Cross-Platform Utility for System Metrics Reporting

**StatMQTT** is a cross-platform utility for reporting system and processes
metrics (CPU, memory, disks, network, smart disk data) to an MQTT broker.

Overview

In the ever-evolving landscape of IoT and system monitoring, StatMQTT stands out as a versatile and efficient solution for reporting system and processes metrics. This open-source project seamlessly integrates with MQTT (Message Queuing Telemetry Transport), ensuring streamlined and reliable data communication. With StatMQTT, you gain real-time insights into CPU usage, memory consumption, disk health, network performance, and SMART disk data across diverse platforms.

Key Features:

StatMQTT boasts a robust feature set:

1. Cross-Platform Compatibility: StatMQTT works seamlessly on Windows, macOS, and Linux, making it a go-to choice for monitoring across various environments.

2. Comprehensive Metrics: Monitor an extensive range of metrics, from CPU utilization to memory usage, disk health, network performance, and SMART disk data. Customize reporting to suit your specific needs.

3. Real-Time Reporting: Harness the power of real-time data reporting to make informed decisions and proactively address potential issues.

4. MQTT Integration: Leveraging MQTT's efficiency and reliability, StatMQTT securely transmits data to an MQTT broker, ensuring data integrity.

5. Customizable Alerts: Configure alerts and notifications based on predefined thresholds, enabling swift responses to anomalies.

6. User-Friendly Interface: StatMQTT offers an intuitive and user-friendly interface, simplifying setup and configuration, even for those new to system monitoring.

Use Cases:

StatMQTT finds applications in a wide range of scenarios:

1. IoT Deployments: Ideal for IoT projects where real-time system metrics reporting is crucial for ensuring seamless operation.

2. Server Management: Streamline server management by closely monitoring key performance indicators and resource utilization.

3. Data Centers: Maintain the reliability and efficiency of data centers by continuously monitoring hardware and system health.

4. Remote Monitoring: StatMQTT facilitates remote IT management by providing insights into system metrics across geographically dispersed locations.

Get Started

Ready to get started with StatMQTT? Visit my GitHub repository for comprehensive documentation, installation guides, and sample configurations. Whether you want to contribute to the project, report issues, or suggest improvements, StatMQTT welcomes your involvement in this project.

Harness the power of MQTT and take control of your system and processes metrics reporting with StatMQTT. Join me on this journey toward efficient and reliable monitoring.

**StatMQTT**  is written in Python and is based on:

* `paho-mqtt <https://github.com/eclipse/paho.mqtt.python>` to communicate with
the MQTT broker;
* `psutil <https://github.com/giampaolo/psutil>` to collect metrics;
* `pySMART <https://github.com/jakogut/pySMART>` to collect SMART data;
* `recurrent <https://github.com/kvh/recurrent>` to describe reporting
schedule.
* `jinja2 <https://github.com/alex-foundation/jinja2>` to format the data.

============
Installation
============
Just install the required Python libraries using `pip <https://pip.pypa.io/en/stable/installing/>`_::

   pip install -r requirements.txt

After you can run main file using::

  python StatMQTT.py


===============================================
General information about tasks and MQTT topics
===============================================

Every utilization task has its own special name. It always starts with task name that could be followed with unique parameter name or/and device number/name.

E.g. task to get percent of CPU used by user apps use name **cpu_times_percent/user**. All possible tasks are described below.

Results for task TASK are pushed to the MQTT topic **StatMQTT/COMPUTER_NAME/TASK** (prefix "StatMQTT/COMPUTER_NAME/" is configurable)

In case of task execution error message is sent to the topic **StatMQTT/COMPUTER_NAME/error/TASK**


Very often it could be useful to provide several parameters from the same task using one request. In such case next formats are used:

- TASK/* - to get all possible parameters (MQTT topic per parameter)

- or TASK/\*; - to get all possible parameters in one topic (combined to JSON string)

Examples::

   Task cpu_times_percent/* provides
   StatMQTT/COMPUTER_NAME/cpu_times_percent/user 12.0
   StatMQTT/COMPUTER_NAME/cpu_times_percent/nice  1.0
   StatMQTT/COMPUTER_NAME/cpu_times_percent/system 5.0
   etc

   Task cpu_times_percent/*; provides
   StatMQTT/COMPUTER_NAME/cpu_times_percent/*; {"user": 12.0, "nice": 1.0, "system": 5.0, ...}


=================
Formatting output
=================

Output of task could be formatted using `Jinja2 <http://jinja.pocoo.org/>`_ templates. Append template to the task after one more "/" separator.

E.g.
    cpu_times_percent/user/{{x}}%
To append % symbol after CPU usage.

For task providing many parameters (having \*) all parameters are available by name if they are named or by index as x[1] if they are numbered.

NOTE: After formatting tasks providing many parametes are combined to single one.

Unnamed parameters are avaiable as x.

All additional filters are defined at the filters.py file. You also can add custom filters there.

Next filters are implemented now::

    KB,MB,GB - to format value in bytes as KBytes, MBytes or GBytes.
    uptime - to format boot_time as uptime string representation.

Examples::

    virtual_memory/*/{{(100*free/total)|int}}% - free virtual memory in %
    boot_time/{{x|uptime}} - uptime
    cpu_times_percent/user/*/{{x[0]+x[1]}} - user CPU times for first and second processors total
    virtual_memory/free/{{x|MB}} - Free RAM in MB

=============
Configuration
=============
All configuration is present in **StatMQTT.conf** file in the app's directory or any other file referenced by **StatMQTTCONFIG** environment variable.

The configuration file is parsed using Python interpreter and contains constants for MQTT broker connection and tasks that have to be executed periodically (schedule).

There are two ways how to force sending some system state parameter over MQTT topic

1. Schedule
2. MQTT request

========
Schedule
========
**schedule** parameter in **StatMQTT.conf** is a Python map having human-readable period as a key and task name (or list of task names) as a value.

You can check examples of recurring period definitions `here <https://github.com/kvh/recurrent>`_.

Also value for scheduled task could be specified as Python dict ({"key": "value"} notation) to send result to the topic different to the task name.

E.g. {"boot_time/{{x|uptime}}": "uptime"} to have boot time posted to the StatMQTT/COMPUTER_NAME/uptime topic.

**NOTE**: Please note that keys in Python dict (**schedule**) should be unique and if there are several schedules with the same period - only last one will be used.
To avoid such issue please use period mapped to the list (or dict) of tasks.

============
MQTT request
============
It's better to describe how to use it using example.
To get information for task "cpu_percent" with MQTT prefix "StatMQTT/COMPUTER_NAME/" you need to send any string on topic::

  StatMQTT/COMPUTER_NAME/request/cpu_percent

and result will be pushed on the topic::

  StatMQTT/COMPUTER_NAME/cpu_percent


=====
Tasks
=====
CPU
::

   cpu_times/* - CPU times information. Topic per parameter
   cpu_times/*;  - CPU times information in one topic (JSON string)
   cpu_times/{user/nice/system/idle/iowait/irq/softirq/steal/guest} - CPU times separate parameters
   cpu_percent - CPU total usage in percent
   cpu_percent/* - CPU usage in percent. Topic per CPU number
   cpu_percent/*; - CPU usage in percent per CPU in one topic (JSON string)
   cpu_percent/{0/1/2/etc} - CPU usage for single CPU
   cpu_times_percent/* - CPU times in percent. Topic per parameter
   cpu_times_percent/*;  - CPU times in percent in one topic (JSON string)
   cpu_times_percent/{user/nice/system/idle/iowait/irq/softirq/steal/guest} - CPU times in percent separate parameters
   cpu_times_percent/{user/nice/system/idle/iowait/irq/softirq/steal/guest}/* - CPU times in percent separate parameters. Topic per CPU number
   cpu_times_percent/{user/nice/system/idle/iowait/irq/softirq/steal/guest}/*; - CPU times in percent separate parameters per CPU number in one topic (JSON string)
   cpu_times_percent/{user/nice/system/idle/iowait/irq/softirq/steal/guest}/{0/1/2/etc} - CPU times in percent separate parameters for single CPU
   cpu_times_percent/*/{0/1/2/etc} - CPU times in percent for single CPU. Topic per parameter
   cpu_times_percent/*;/{0/1/2/etc} - CPU times in percent for single CPU in one topic (JSON string)
   cpu_stats/* - CPU statistics. Topic per parameter
   cpu_stats/*;  - CPU statistics in one topic (JSON string)
   cpu_stats/{ctx_switches/interrupts/soft_interrupts/syscalls} - CPU statistics separate parameters

Memory
::

   virtual_memory/* - Virtual memory. Topic per parameter
   virtual_memory/*;  - Virtual memory in one topic (JSON string)
   virtual_memory/{total/available/percent/used/free/active/inactive/buffers/cached} - Virtual memory separate parameters
   swap_memory/* - Swap memory. Topic per parameter
   swap_memory/*;  - Swap memory in one topic (JSON string)
   swap_memory/{total/used/free/percent/sin/sout} - Swap memory separate parameters

Disks
::

   disk_partitions/{device/mountpoint/fstype/opts}/* - Disk partitions separate parameters. Topic per disk number
   disk_partitions/{device/mountpoint/fstype/opts}/*; - Disk partitions separate parameters per disk number in one topic (JSON string)
   disk_partitions/{device/mountpoint/fstype/opts}/{0/1/2/etc} - Disk partitions separate parameter for single disk number
   disk_partitions/*/{0/1/2/etc} - Disk partitions parameters for single disk number. Topic per parameter
   disk_partitions/*;/{0/1/2/etc} - Disk partitions parameters for single disk number in one topic (JSON string)
   disk_usage/{total/used/free/percent}/{drive} - Disk usage single parameter (slashes in drive should be replaced with vertical slash)
   disk_usage/*/{drive} - Disk usage separate parameters. Topic per parameter
   disk_usage/*;/{drive} - Disk usage separate parameters in one topic (JSON string)
   disk_io_counters/* - Disk I/O counters. Topic per parameter
   disk_io_counters/*;  - Disk I/O counters in one topic (JSON string)
   disk_io_counters/{read_count/write_count/read_bytes/write_bytes/read_time/write_time/read_merged_count/write_merged_count/busy_time} - Disk I/O counters separate parameters
   disk_io_counters/{read_count/write_count/read_bytes/write_bytes/read_time/write_time/read_merged_count/write_merged_count/busy_time}/* - Disk I/O counters separate parameters. Topic per disk number
   disk_io_counters/{read_count/write_count/read_bytes/write_bytes/read_time/write_time/read_merged_count/write_merged_count/busy_time}/*; - Disk I/O counters separate parameters per disk number in one topic (JSON string)
   disk_io_counters/{read_count/write_count/read_bytes/write_bytes/read_time/write_time/read_merged_count/write_merged_count/busy_time}/{0/1/2/etc} - Disk IO counters separate parameters for single disk
   disk_io_counters/*/{0/1/2/etc} - Disk I/O counters for single disk. Topic per parameter
   disk_io_counters/*;/{0/1/2/etc} - Disk I/O counters for single disk in one topic (JSON string)

Network
::

   net_io_counters/* - Network I/O counters. Topic per parameter
   net_io_counters/*;  - Network I/O counters in one topic (JSON string)
   net_io_counters/{bytes_sent/bytes_recv/packets_sent/packets_recv/errin/errout/dropin/dropout} - Network I/O counters separate parameters
   net_io_counters/{bytes_sent/bytes_recv/packets_sent/packets_recv/errin/errout/dropin/dropout}/* - Network I/O counters separate parameters. Topic per device name
   net_io_counters/{bytes_sent/bytes_recv/packets_sent/packets_recv/errin/errout/dropin/dropout}/*; - Network I/O counters separate parameters per device in one topic (JSON string)
   net_io_counters/{bytes_sent/bytes_recv/packets_sent/packets_recv/errin/errout/dropin/dropout}/{eth0/wlan0/etc} - Network I/O counters separate parameters for single device
   net_io_counters/*/{eth0/wlan0/etc} - Network I/O counters for single device. Topic per parameter
   net_io_counters/*;/{eth0/wlan0/etc} - Network I/O counters for single device in one topic (JSON string)

Temperature
::

   sensors_temperatures/* - Sensors current temperatures. Topic per sensor
   sensors_temperatures/*;  - Sensors current temperatures in one topic (JSON string)
   sensors_temperatures/{SENSOR_NAME} - Single sensor current temperature (could be array value if sensor has several devices)
   sensors_temperatures/{SENSOR_NAME}/* - Single sensor temperatures. Topic per temperature
   sensors_temperatures/{SENSOR_NAME}/*; - Single sensor temperatures in one topic (JSON string)
   sensors_temperatures/{SENSOR_NAME}/{DEVICE_NUMBER/DEVICE_LABEL} - Single sensor device by number/label current temperature
   sensors_temperatures/{SENSOR_NAME}/{DEVICE_NUMBER/DEVICE_LABEL}/* - Single sensor device by number/label temperature. Topic per parameter
   sensors_temperatures/{SENSOR_NAME}/{DEVICE_NUMBER/DEVICE_LABEL}/*; - Single sensor device by number/label temperature in one topic (JSON string)
   sensors_temperatures/{SENSOR_NAME}/{DEVICE_NUMBER/DEVICE_LABEL}/{label/current/high/critical} - Single sensor device by number/label temperature separate parameters

Fan speed
::

   sensors_fans/* - Fans current speeds. Topic per fan
   sensors_fans/*;  - Fans current speeds in one topic (JSON string)
   sensors_fans/{SENSOR_NAME} - Single fan current speed (could be array value if fan has several devices)
   sensors_fans/{SENSOR_NAME}/* - Single fan speeds. Topic per speed
   sensors_fans/{SENSOR_NAME}/*; - Single fan speeds in one topic (JSON string)
   sensors_fans/{SENSOR_NAME}/{DEVICE_NUMBER/DEVICE_LABEL} - Single fan device by number/label current speed
   sensors_fans/{SENSOR_NAME}/{DEVICE_NUMBER/DEVICE_LABEL}/* - Single fan device by number/label speed. Topic per parameter
   sensors_fans/{SENSOR_NAME}/{DEVICE_NUMBER/DEVICE_LABEL}/*; - Single fan device by number/label speed in one topic (JSON string)
   sensors_fans/{SENSOR_NAME}/{DEVICE_NUMBER/DEVICE_LABEL}/{label/current/high/critical} - Single fan device by number/label speed separate parameters

Battery
::

   sensors_battery/* - Battery state. Topic per parameter
   sensors_battery/*;  - Battery state parameters in one topic (JSON string)
   sensors_battery/{percent/secsleft/power_plugged} - Battery state separate parameters
        where secsleft could be
            -1 if time is unknown
            -2 for unlimited time (power is plugged)
            or time in seconds

Other system info
::

   users/{name/terminal/host/started}/* - Active users separate parameters. Topic per user
   users/{name/terminal/host/started}/*; - Active users separate parameters per user in one topic (JSON string)
   users/{name/terminal/host/started}/{0/1/2/etc} - Active users separate parameter for single user
   users/*/{0/1/2/etc} - Active users parameters for single user. Topic per parameter
   users/*;/{0/1/2/etc} - Active users parameters for single user in one topic (JSON string)
   boot_time - System boot time as a Unix timestamp
   boot_time/{{x|uptime}} - String representation of up time


Processes
::

    pids/* - all system processes IDs. Topic per process
    pids/*; - all system processes IDs in one topic (JSON string)
    pids/{0/1/2/etc} - single process ID
    pids/count - total number of processes
    processes/{PROCESS_ID}/{PARAMETER_NAME} - single process parameter(s)
        where PROCESS_ID could be one of
            - numeric ID of the process
            - top_cpu - top CPU consuming process
            - top_cpu[N] - CPU consuming process number N
            - top_memory - top memory consuming process
            - top_memory[N] - memory consuming process number N
            - pid[PATH] - process with ID specified in the file having PATH path (.pid file). Slashes in path should be replaced with vertical slash
            - name[PATTERN] - process with name matching PATTERN pattern (use * to match zero or more characters, ? for single character)
            - * - to get value of some property for all processes. Topic per process ID
            - *; - to get value of some property for all processes in one topic (JSON string)
        and PARAMETER_NAME could be one of
            - pid - process ID
            - ppid - parent process ID
            - name - process name
            - exe - process executable file
            - cwd - process working directory
            - cmdline/* - command line. Topic per line
            - cmdline/*; - command line in one topic (JSON string)
            - cmdline/count - number of command line lines
            - cmdline/{0/1/etc} - command line single line
            - status - process status (running/sleeping/idle/dead/etc)
            - username - user started process
            - create_time - time when process was started (Unix timestamp)
            - terminal - terminal of the process
            - uids/* - process user IDs. Topic per parameter
            - uids/*; - process user IDs in one topic (JSON string)
            - uids/{real/effective/saved} - process user IDs single parameter
            - gids/* - process group IDs. Topic per parameter
            - gids/*; - process group IDs in one topic (JSON string)
            - gids/{real/effective/saved} - process group IDs single parameter
            - cpu_times/* - process CPU times. Topic per parameter
            - cpu_times/*; - process CPU times in one topic (JSON string)
            - cpu_times/{user/system/children_user/children_system} - process CPU times single parameter
            - cpu_percent - CPU percent used by process
            - memory_percent - memory percent used by process
            - memory_info/* - memory used by process. Topic per parameter
            - memory_info/*; - memory used by process in one topic (JSON string)
            - memory_info/{rss/vms/shared/text/lib/data/dirty/uss/pss/swap} - memory used by process single parameter
            - io_counters/* - process I/O counters. Topic per parameter
            - io_counters/*; - process I/O counters in one topic (JSON string)
            - io_counters/{read_count/write_count/read_bytes/write_bytes} - process I/O single counter
            - num_threads - number of threads
            - num_fds - number of file descriptors
            - num_ctx_switches/* - number of context switches. Topic per parameter
            - num_ctx_switches/*; - number of context switches in one topic (JSON string)
            - num_ctx_switches/{voluntary/involuntary} - context switches single counter
            - nice - nice value
            - * - all process properties. Topic per property
            - *; - all process properties in one topic (JSON string)
            - ** - all process properties and sub-properties. Topic per property
            - **; -  all process properties and sub-properties in one topic (JSON string)


============
Useful tasks
============
**boot_time/{{x|uptime}}** - Up time

**cpu_percent** - CPU usage in percent

**virtual_memory/percent** - RAM usage in percent

**virtual_memory/free/{{x|MB}}** - Free RAM in MB

**disk_usage/percent/|** - root drive (slash replaced with vertical slash) usage in percent (Linux)

**disk_usage/free/|/{{x|GB}}** - space left in GB for root drive (Linux)

**disk_usage/percent/C:** - C:/ drive usage in percent (Windows)

**processes/top_cpu/name** - name of top process consuming CPU

**processes/top_memory/exe** - executable file of top process consuming memory

**smart/nvme0/** - all the device 'nvme0' SMART data, requries SUDO

**smart/nvme0/temperature** - 'nvme0' temperature, requires SUDO
