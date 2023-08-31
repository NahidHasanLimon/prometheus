# Node Exporter 
## Install node exporter

## Step For config .yml 

mkdir /etc/node_exporter/
touch /etc/node_exporter/config.yml
chmod 700 /etc/node_exporter
chmod 600 /etc/node_exporter/config.yml
chown -R nodeusr:nodeusr /etc/node_exporter


## node exporter service for daemon 
nano /etc/systemd/system/node_exporter.service

change into ExecStart=/usr/local/bin/node_exporter --web.config=/etc/node_exporter/config.yml


## Daemon reload and restart
systemctl daemon-reload
systemctl restart node_exporter

# Node Exporter Auth 
 update node exporter config.yml 
 
  systemctl restart node_exporter


## Now check via curl 

curl -u prometheus:prometheus http://node02:9100/metrics

# Now, let's configure the Prometheus server to use authentication when scraping metrics from node servers.

## update promethues yml

 /etc/prometheus/prometheus.yml

Under - job_name: "nodes" add below lines:

    basic_auth:
      username: prometheus
      password: secret-password




- systemctl restart prometheus


# Now make tls

openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout node_exporter.key -out node_exporter.crt -subj "/C=US/ST=California/L=Oakland/O=MyOrg/CN=localhost" -addext "subjectAltName = DNS:localhost"

Move the crt and key file under /etc/node_exporter/ directory

mv node_exporter.crt node_exporter.key /etc/node_exporter/


chown nodeusr.nodeusr /etc/node_exporter/node_exporter.key
chown nodeusr.nodeusr /etc/node_exporter/node_exporter.crt


vi /etc/node_exporter/config.yml



Add below lines in this file:


tls_server_config:
  cert_file: node_exporter.crt
  key_file: node_exporter.key


  systemctl restart node_exporter
exit



You can verify your changes using curl command:


curl -u prometheus:secret-password -k https://node01:9100/metrics


# Configure Promethues Server 

Copy the certificate from node01 to Prometheus server


scp root@node01:/etc/node_exporter/node_exporter.crt /etc/prometheus/node_exporter.crt



Change certificate file ownership:


chown prometheus.prometheus /etc/prometheus/node_exporter.crt



Edit /etc/prometheus/prometheus.yml file


vi /etc/prometheus/prometheus.yml 



Add below given lines under - job_name: "nodes"


    scheme: https
    tls_config:
      ca_file: /etc/prometheus/node_exporter.crt
      insecure_skip_verify: true



Restart prometheus service

systemctl restart prometheus







