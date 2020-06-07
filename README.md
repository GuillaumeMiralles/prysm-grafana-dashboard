Grafana is an open-source data metrics tool that is used to aggregate large amounts of data into a comprehensive visual dashboard for easy analysis. This section includes instructions for installing Grafana on the local machine and configuring Telegram or Discord alerts for monitoring validator status on-the-go.

![Grafana dashboard for prysm node and validator](https://camo.githubusercontent.com/d76a9bf4a104f69f4d4edd04c8d364cbb535e76c/68747470733a2f2f696d6775722e636f6d2f6e4b7a6b7234592e706e67 "Grafana dashboard for prysm node and validator")


### Getting account metrics
 Ensure metrics have been activated by visiting the following dashboards:
  * Node metrics are found at http://localhost:8080/metrics
  * Validator metrics are found at http://localhost:8081/metrics

## Installing Prometheus

Prometheus must first be installed to fetch the data from the beacon node and validator for Grafana to display.

1. [Download the Prometheus files](https://prometheus.io/download/) suited for the host system. 

2. Extract the archive and enter it's new directory. 

3. Locate the `prometheus.yml` file and replace the contents with the following:

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

4. In the same directory, double-click the **prometheus** file (with extension `.exe` in Windows) to start Prometheus,
or do so in a terminal by issuing the command:
```
./prometheus
```
  A terminal will open presenting the Prometheus log. 
 
  > **NOTICE:** Prometheus' default data logging time is 15 days. To extend dashboard statistics to 31 days, add `--storage.tsdb.retention.time=31d` to this startup command.

5. Navigate to http://localhost:9090/graph in a browser. It will present a page similar to this:
![Prometheus page](https://camo.githubusercontent.com/60a6730b181b14b8ff7bbbd49debd1b326dbc88a/68747470733a2f2f692e696d6775722e636f6d2f4d523772636b582e706e67 "Prometheus page")

Take note of the `validator_statuses` and `total_voted_target_balances`, as they are required later.

#### (Optional) Windows: Running Prometheus in the background

In a **Windows Powershell** prompt, navigate to the directory the `prometheus.exe` file is saved and issue the command:
```
Start-Process "prometheus.exe" "--storage.tsdb.retention.time=31d" -WindowStyle Hidden
```
To stop the prometheus process at any time, open the windows task manager and search for `prometheus.exe`, then end the task.


## Installing Grafana

Grafana must now be installed to provide the graphical component of the data analytics.

1. [download Grafana](https://grafana.com/grafana/download) and install it.

2. Open http://localhost:3000 in a browser. By default, the username and the password to this panel are both ‘admin’.

3. Create a data source and choose Prometheus, then enter in the URL field http://localhost:9090. 

4. click on **Save & Test**. 

A green notification saying “Datasource updated” should now be visible on the upper right corner.

## Enabling mobile alerts

1. On the left menu of Grafana, select **Notification channels** under the bell icon. 

2. Click on **New channel**.

### Option 1: Telegram

1. Select Telegram from the list.

2. To complete the **Telegram API settings**, a Telegram channel and bot are required. For instructions on setting up a bot with `@Botfather`, see [this section](https://core.telegram.org/bots#6-botfather) of the Telegram documentation.

3. Once completed, invite the bot to the newly created channel.

### Option 2: Discord

1. Select **Discord** in the type drop down selection. 

2. To complete the set up, a Discord server (and a text channel available) as well as a Webhook URL are required. For instructions on setting up a Discord's Webhooks, see [this section](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks) of their documentation.
  
3. Navigate back to http://localhost:3000 and enter the Webhook URL in the Discord notification settings panel. 

4. Click **Send Test**, which will push a confirmation message to the Discord channel.

## Creating and importing dashboards

1. The dashboard can now be customised to the users preferences. There are two examples that can be used:
- [dashboard designed for small amount of validator keys](/assets/grafana-dashboards/small_amount_validators.json)
- [dashboard designed for more than 10 validator keys](/assets/grafana-dashboards/big_amount_validators.json)

2. To import this json into the Grafana dashboard, click on the **+** icon on the left menu and select 'Import', 

3. Paste the json and click the **Load** button.

## Running nodes and validators on separate hardware

For those running their node and validators on separate machines, simply modify the pasted `prometheus.yml` data from the earlier step and change any instances of `localhost` to the desired IP. For local networks, the _private IP_ is required. For connections over the internet, the _public facing IP_ will be required.

* [Finding a **private IP**](https://docs.prylabs.network/docs/prysm-usage/p2p-host-ip/#private-ip-addresses)
* [Finding a **public IP**](https://docs.prylabs.network/docs/prysm-usage/p2p-host-ip/#public-ip-addresses)

> **NOTICE:** In case of public IPs, [port forwarding](https://github.com/wgknowles/documentation/blob/15da3fb1ea477f260ef287497fe047b0a78879b3/docs/prysm-usage/p2p-host-ip.md#port-forwarding) may need to be configured.
