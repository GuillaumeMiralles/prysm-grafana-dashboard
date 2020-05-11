# Guide monitoring + telegram alerts for prysm node and validator with Prometheus and Grafana

###### Written by Ocaa/Grums. Feel free to ask me if you face any issue on the prysm discord

Now that you have your validator and node process running, you will probably need a nice dashboard and alert system to ensure at maximum the profitability of your staking ETH. Here is a simple guide that explain how to get one, without any developer skill.

Here is the result of what you will get if you follow this guide!
![Grafana dashboard for prysm node and validator](https://imgur.com/nbI9KPP.png "Grafana dashboard for prysm node and validator")


## Metrics from validators and node process
First thing you need to do, is to add the flag `--enable-account-metrics` to the end of the validator command

Now you have to make sure that you have access to both metrics:
- on the machine running the node, you will find the node metrics on http://localhost:8080/metrics
- on the machine running the validator, you will find the validator metrics on http://localhost:8081/metrics

## Prometheus
#### Installation
[Download Prometheus](https://prometheus.io/download/), then go to the directory where the prometheus.exe file is, and edit the prometheus.yml file to replace the content of the file by this:
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
##### Node and validator running in same network but different machine
If you don’t run your validator and your node on the same machine but on the same network, you still can have access to both metrics on a single machine. This works the same way as for the flag `--beacon-rpc-provider` on the validator command.
So for example, let’s say you are running your validator on machine A and node on machine B, and you want to create all your metrics on machine A, then you will have to replace `localhost:8080` by `machine_B_local_Ip_V4:8080` in the prometheus.yml file.


#### Verification
Once the prometheus.yml file has been updated, you can run the prometheus.exe file, then open this web page http://localhost:9090/graph
If everything is working, you will see a GUI to create your own graphs by selecting the metrics that you want to monitor. Make sure to find validator and node metrics (`validator_balance` is a validator metric and `total_voted_target_balances` is a node metric)

#### Setting the storage time (default 15 days)
This is particularly useful if you plan on getting my dashboard, or if you plan to have long term charts. Basicaly, Prometheus will store all the data in a kind of database on your machine, but to prevent having a database too huge, it will remove old data, and by default this is set to 15 days. So to change that, you'll need to run Prometheus with the flag `--storage.tsdb.retention.time`.
For people who wants to run my dashboard, you'll need this `--storage.tsdb.retention.time=31d`at least. If you are using windows the section below shows how to run Prometheus in bakground with this flag included

#### Prometheus running in background for Windows (facultative)
Now that everything is working on prometheus, you might have figured that you keep having a window open for prometheus, and if you close it then you won’t have access to http://localhost:9090/graph no more. So we will see here how to run prometheus in the background.
For that you need to open a Windows Powershell prompt, then go to the directory where is located the prometheus.exe file (using command cd) and to run this command:
`Start-Process "prometheus.exe" "--storage.tsdb.retention.time=31d" -WindowStyle Hidden`
If you ever want to stop the prometheus process, you can find it under the name prometheus.exe in the windows task manager (shortcut ctrl+alt+del), then you can stop it if wanted.


## Grafana
#### Installation
It’s now time to [download Grafana](https://grafana.com/grafana/download) and install it.
After the installation, you can now open the web page http://localhost:3000. By default, the username and the password are both ‘admin’

Create a data source and choose Prometheus, and enter in the URL field http://localhost:9090 then click on “Save & Test”. You should have a green notification on the corner upper right “Datasource updated”

#### Alerts on Telegram
On the left menu of Grafana, select “Notification channels” under the bell icon. Click on “New channel”. Select now the type “Telegram” and configure it how you would like the alerts to be done.

To complete the Telegram API settings you will have to create a bot and a telegram channel. I will let you follow this [guide](https://gist.github.com/ilap/cb6d512694c3e4f2427f85e4caec8ad7) that helped me, BUT there is an error in this guide. At this step `Invite @BotFather to that channel as admin` you shouldn’t invite @BotFather but your own bot that you just created instead.
Use the “Send test” button to make sure that your bot is sending alerts to your telegram channel. Once it’s working, save it.

You can now create your own alert rules if needed, the guide above explain how.

#### Creating/importing dashboards
You can now create your own dashboard how you feel like to. Or you can also just import my own dashboards:
- [dashboard designed for small amount of validator keys](https://raw.githubusercontent.com/GuillaumeMiralles/prysm-grafana-dashboard/master/less_10_validators.json)
- [dashboard designed for more than 10 validator keys](https://raw.githubusercontent.com/GuillaumeMiralles/prysm-grafana-dashboard/master/more_10_validators.json).

It’s the json for a node/validator grafana dashboard made by myself. To import this json into your Grafana dashboard, you click on the “+” icon on the left menu and select Import, you then just have to paste the json and click “Load” button.

If you ever want to stop the grafana server process, you can find it under the name grafana-server.exe in the windows task manager (shortcut ctrl+alt+del), then you can stop it if wanted.




