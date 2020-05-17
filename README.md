# Guide step to step to get a Prysm dashboard using Grafana and Prometheus with Telegram system alerts
###### Written by Ocaa/grums, contact me on prysm discord if you face any issue or bug

Now that you have your validator and node process running, you will probably need a nice dashboard and alert system to ensure at maximum the profitability of your staking ETH. Here is a simple guide that explain how to get one, without any developer skill.

Here is the result of what you will get if you follow this guide!
![Grafana dashboard for prysm node and validator](https://imgur.com/nbI9KPP.png "Grafana dashboard for prysm node and validator")


## Metrics from validators and node process
First thing you need to do, is to add this flag to the end of the validator command
```
--enable-account-metrics
```

Now you have to make sure that you have access to both metrics:
- on the machine running the node, you will find the node metrics on http://localhost:8080/metrics
- on the machine running the validator, you will find the validator metrics on http://localhost:8081/metrics

## Prometheus
### Installation
[Download Prometheus](https://prometheus.io/download/), then go to the directory where the **prometheus.exe** file is, and edit the **prometheus.yml** file to replace the content of the file by this:
```# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).
# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  - job_name: 'validator'
    static_configs:
      - targets: ['localhost:8081']
  - job_name: 'beacon node'
    static_configs:
      - targets: ['localhost:8080']
```
#### Node and validator running on different machine
You still can regroup your metrics on a single machine.

##### **Local network**
If you don’t run your validator and your node on the same machine but on the same network, you will need to replace `localhost` by the **private** IP of the machine that run the process in the **prometheus.yml** file.

For the job `beacon node` you will need to replace `localhost` by the **private** IP of the machine where the `beacon node` is running. Same goes for validator.

To determine your  **private**  IP address, or run the appropriate command for your OS:

###### **GNU/Linux:**

```
ip addr show | grep "inet " | grep -v 127.0.0.1
```

###### **Windows:**

```
ipconfig | findstr /i "IPv4 Address"
```

###### **macOS:**

```
ifconfig | grep "inet " | grep -v 127.0.0.1
```

##### **Different network**
If you don’t run your validator and your node on the same machine and also on different network, you will need to replace `localhost` by the **public** IP of the machine that run the process in the **prometheus.yml** file.

For the job `beacon node` you will need to replace `localhost` by the **public** IP of the machine where the `beacon node` is running. Same goes for validator.

To determine your  **public**  IP address, visit ([http://v4.ident.me/](http://v4.ident.me/)) or run this command:

```
curl v4.ident.me
```

  > **NOTICE:** In the case of different network, you might need to configure your firewalls for [port forwarding](https://github.com/wgknowles/documentation/blob/15da3fb1ea477f260ef287497fe047b0a78879b3/docs/prysm-usage/p2p-host-ip.md#port-forwarding).
 

### Verification
Once the **prometheus.yml** file has been updated, you can run the **prometheus.exe** file, then open this web page http://localhost:9090/graph
If everything is working, you should see a page similar to this
![Prometheus page](https://i.imgur.com/MR7rckX.png "Prometheus page")

Now you have to make sure that you can find both metrics in the previous image: `validator_balance` and `total_voted_target_balances`. You need to find both metrics to have access to all of them.

### Setting the storage time to 31 days (recommended)
You'll probably need to add this flag while running **prometheus** to upgrade to 31 days the storage time of the metrics. By default, this value is set to 15 days
```
--storage.tsdb.retention.time=31d
```
  > **NOTICE:** If you don't plan to have panels that will require metrics older than 15 days then you can skip this part


### Windows: Prometheus running in background (facultative)
For that you need to open a **Windows Powershell** prompt, then go to the directory where is located the **prometheus.exe** file (using command `cd`) and to run this:
```
Start-Process "prometheus.exe" "--storage.tsdb.retention.time=31d" -WindowStyle Hidden
```
If you ever want to stop the prometheus process, you can find it under the name prometheus.exe in the windows task manager (shortcut ctrl+alt+del), then you can end the task.


## Grafana
### Installation
It’s now time to [download Grafana](https://grafana.com/grafana/download) and install it.
After the installation, you can now open the web page http://localhost:3000. By default, the username and the password are both ‘admin’

Create a data source and choose Prometheus, and enter in the URL field http://localhost:9090 then click on **Save & Test**. You should have a green notification on the corner upper right “Datasource updated”

### Enable alerts

On the left menu of Grafana, select **Notification channels** under the bell icon. Click on **New channel**.
You can now chose the channel you want to create to be alerted. You can find some of them explained below.

#### Telegram
 Select now the type **Telegram** and configure it how you would like the alerts to be done.

To complete the **Telegram API settings** you will have to create a bot and a telegram channel. Follow this [guide](https://gist.github.com/ilap/cb6d512694c3e4f2427f85e4caec8ad7) to create it, **BUT** there is an error in it. At this step
```
Invite @BotFather to that channel as admin
``` 
you should invite your own bot that you just created instead.
Use the **Send test** button to make sure that your bot is sending alerts to your telegram channel. Once it’s working, save it.

You can now create your own alert rules if needed, the guide above explain how.

### Creating/importing dashboards
You can now create your own dashboard how you feel like to. Or you can also just import the Prysm dashboards:
- [dashboard designed for small amount of validator keys](https://raw.githubusercontent.com/GuillaumeMiralles/prysm-grafana-dashboard/master/less_10_validators.json)
- [dashboard designed for more than 10 validator keys](https://raw.githubusercontent.com/GuillaumeMiralles/prysm-grafana-dashboard/master/more_10_validators.json).

It’s the json for a node/validator grafana dashboard made by myself. To import this json into your Grafana dashboard, you click on the **+** icon on the left menu and select Import, you then just have to paste the json and click **Load** button.



