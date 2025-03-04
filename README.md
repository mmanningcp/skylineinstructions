# Check Point Skyline + Prometheus, Grafana on Ubuntu 20.04
Instructions for Youtube Video [Check Point for Grafana (Skyline) ](https://youtu.be/FO2Rp9x31i0)
## 1. Update Ubuntu 20.04 server
1.1 Once Ubuntu 20.04 has been downloaded and installed, ensure it had has 
- Network access
- DNS  
- NTP

1.2 Update Ubuntu; fetch the latest version of the package list from your distro's software repository.

	sudo apt-get update || apt-get upgrade

## 2. Download and install Prometheus

2.1 The following command will download and install Prometheus

    wget https://github.com/prometheus/prometheus/releases/download/v2.38.0/prometheus-2.38.0.linux-amd64.tar.gz
    sudo useradd --no-create-home --shell /bin/false prome
    sudo useradd --no-create-home --shell /bin/false node_exporter
    sudo mkdir /etc/prometheus
    sudo mkdir /var/lib/prometheus
    sudo tar xvf prometheus-2.38.0.linux-amd64.tar.gz
    sudo cp prometheus-2.38.0.linux-amd64/prometheus /usr/local/bin/
    sudo cp prometheus-2.38.0.linux-amd64/promtool /usr/local/bin/
    sudo chown prome:prome /usr/local/bin/prometheus
    sudo chown prome:prome /usr/local/bin/promtool
    sudo cp -r prometheus-2.38.0.linux-amd64/consoles /etc/prometheus
    sudo cp -r prometheus-2.38.0.linux-amd64/console_libraries /etc/prometheus
    sudo chown -R prome:prome /etc/prometheus/consoles
    sudo chown -R prome:prome /etc/prometheus/console_libraries
    sudo touch ~/prometheus.yml
    sudo prometheus --web.enable-remote-write-receiver

2.2 Test if Prometheus is running > http://x.x.x.x:9090 in a web browser.


## 3.  Run Prometheus on boot (Optional)

3.1 Add the following line to your **prometheus.yml** file usually located in */home/username/prometheus-2.38.0.linux-amd64/*

    remote_write:
        - url: "http://x.x.x.x:9090/api/v1/write"


3.2  Create **"prometheus.service"** services file

    sudo touch /etc/systemd/system/prometheus.service
    sudo vi /etc/systemd/system/prometheus.service

3.3 Copy below into **"prometheus.service"** 
```
[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After-network=online.target

[Service]
User=root
Restart=on-failure

ExecStart=/home/username/prometheus-2.38.0.linux-amd64/prometheus --config.file=/home/username/prometheus-2.38.0.linux-amd64/prometheus.yml --web.enable-remote-write-receiver

[Install]
WantedBy=multi-user.target
```

3.4 Setup service

    sudo systemctl daemon-reload
    sudo systemctl start prometheus
    sudo systemctl status prometheus
    sudo systemctl enable prometheus

## 4. Download and install  Grafana

4.1 Get Grafana, install it, and run it. Commands:

    wget https://dl.grafana.com/enterprise/release/grafana-enterprise_9.1.1_amd64.deb
    sudo dpkg -i grafana-enterprise_9.1.1_amd64.deb
    sudo /bin/systemctl enable grafana-server
    sudo /bin/systemctl start grafana-server

Note: If you get the error *"dpkg: Error processing package grafana-enterprise"* run the command below, otherwise skip to 4.2

    sudo apt --fix-broken install

4.2 To test if Grafana is running navigate to http://x.x.x.x:3000 in a web browser.

4.3 Login to Grafana with admin/admin and change the default password.

4.4 Add a new datasource for you Prometheus server

4.5 Import the Check Point provided templates here:
[sk178566](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk178566#Download)

# 5. Setup Check Point Firewall open telemery (Change name)

5.1 Create **payload-no-tls.json**

    touch payload-no-tls.json
    vi payload-no-tls.json

5.2 Copy below into "payload-no-tls.json"

```
{
	"enabled":true,
	"export-targets": {"add": [
	{
		"enabled":true,
		"type": "prometheus-remote-write",
		"url": "http://x.x.x.x:9090/api/v1/write"
	}
	]}
}
```

5.3 Run the following command

    sklnctl export --set "$(cat payload-no-tls.json)"

5.4  Check your Grafana server for data. 

