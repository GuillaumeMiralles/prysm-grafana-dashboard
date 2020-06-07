## Get Cryptowat
To get the currency converter feature you will need to download and run a third party tool on [this repository](https://github.com/nbarrientos/cryptowat_exporter/tree/e4bcf6e16dd2e04c4edc699e795d9450dee486ab "cryptowat")

1. You can simply run the command below to download the specific state of this repository

```curl -OL https://github.com/nbarrientos/cryptowat_exporter/archive/e4bcf6e16dd2e04c4edc699e795d9450dee486ab.zip```

> **NOTICE:** This repository being a third party, we can't be sure that it will stay safe forever. To prevent stakers to download the new versions that could maybe contain malicious purpose. Be sure to download the version recommended above.

2. Unzip the file.

## Run Cryptowat
1. In order to run cryptowat, you will need to get Golang if this is not already done yet.

###### Linux & Mac 
Download and install golang
```sudo apt-get install golang```

###### Windows
Download the latest version of [Golang](https://golang.org/dl/) for Windows and run the file.

2. Ensure that go is correctly installed

```go version```

You should get an output like `go version go1.14.1 windows/amd64`

3. Open a terminal, and navigate inside of the cryptowat directory downloaded previously.

4. Create the binary

```go build```

You should now have in the cryptowat directory a file named `cryptowat_exporter`

5. Run the cryptowat_exporter binary

```./cryptowat_exporter --cryptowat.pairs=etheur,ethusd,ethgbp,ethcad,ethchf,ethjpy,ethbtc --cryptowat.exchanges=kraken```

> **NOTICE:** Command for Windows users: `cryptowat_exporter.exe --cryptowat.pairs=etheur,ethusd,ethgbp,ethcad,ethchf,ethjpy,ethbtc --cryptowat.exchanges=kraken`

6. Ensure that the program is running properly, visit the webpage http://localhost:9745/metrics. If this is working properly, you should have access to the metrics from here.


## Update Prometheus configuration

1. In order for Prometheus to receive the metrics from cryptowat, you will need to replace the **prometheus.yml** file content with this:

```
# my global config
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
  - job_name: 'cryptowat'
    static_configs:
      - targets: ['localhost:9745']
  - job_name: 'validator'
    static_configs:
      - targets: ['localhost:8081']
  - job_name: 'beacon node'
    static_configs:
      - targets: ['localhost:8080']
```

2. Restart **prometheus** to reload the new version of the yml file.

3. Ensure that you have access to the cryptowat metrics through prometheus in http://localhost:9090/graph. You need to be able to find the metric named **crypto_currency**







