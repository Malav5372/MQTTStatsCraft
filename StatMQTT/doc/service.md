# Setting up `StatMQTT` as a Service

It is possible to:

* install the required packages into the site packages and
* run `StatMQTT.py` as a service upon system boot.

## Current Ubuntu flavors

Make a copy of `StatMQTT` directory e.g., to `/opt/StatMQTT`, for use by a service:
```sh
sudo cp -r ../StatMQTT /opt
```

Modify the included `StatMQTT.service`:

* specify `WorkingDirectory=/opt/StatMQTT`
* adjust `ExecStart=/usr/bin/python3 /opt/StatMQTT/StatMQTT.py`

Ensure that `StatMQTT.conf` in `WorkingDirectory`:

* points to your MQTT broker
* schedule and tasks make sense

Then:

```sh
sudo cp StatMQTT.service /etc/systemd/system
sudo systemctl enable StatMQTT.service
sudo systemctl start StatMQTT.service
```

Check the service status with `sudo systemctl status StatMQTT`.

Look into syslog for potential errors:

```sh
sudo tail /var/log/syslog
```

In my case I noticed: `ModuleNotFoundError: No module named 'recurrent'`.

To resolve the problem I did `sudo pip3 install -r requirements.txt` to install
the packages into the site-packages.

And then:

```
sudo systemctl restart StatMQTT
```
