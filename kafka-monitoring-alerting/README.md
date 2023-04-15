# Kafka dashboards and alerting on Cloud Monitoring

## Prerequisite: 
1. Enable Prometheus on the Cluster [Get started with Prometheus ](https://cloud.google.com/stackdriver/docs/managed-prometheus/setup-managed)
2. Deploy [JMX metrics exporter](https://bitnami.com/stack/jmx-exporter)
3. A GKE cluster
4. A deployment of Kafka

## Setup Dashboards
To create a custom dashboard and set up alerts for your Kafka cluster, follow these steps:
1. Clone this repository and navigate into the directory
2. Set up Kafka monitoring and alerting policies
3. Set environment variables
```sh
export PROJECT_ID=<YOUR PROJECT ID>
```

4. Create the dashboards for your Kafka application.
```
gcloud monitoring dashboards create \
     --config-from-file dashboards/kafka-monitoring.json \
     --project $PROJECT_ID
```

4. Create the dashboard for the Kafka cluster.
```
gcloud monitoring dashboards create \
     --config-from-file dashboards/kafka-gke-cluster.json \
     --project $PROJECT_ID
```
## Create the alerting policies.
1. Create an email alert notification channel.
```
CHANNEL=$(gcloud alpha monitoring channels create \
    --channel-labels=email_address=ALERT_EMAIL \
    --display-name="Email to project owner" \
    --type=email \
    --format='value(name)')
```
Replace ALERT_EMAIL with an email address.

2. Create an alert policy for active broker count to ensure 3 brokers are available and an alert for under replicated partitions 
```
for POLICY_FILE in $(find alerting/ -name '*.json')
do
  gcloud alpha monitoring policies create \
      --project=$PROJECT_ID \
      --notification-channels=$CHANNEL \
      --policy-from-file=$POLICY_FILE
done
```

This script creates two alerting policies in Cloud Monitoring:

Kafka Active Broker Count: Send an alert to the email address listed in the previous step when the total number of brokers in the goes below three.
Under Replicated Partition: Send an alert when the number of partitions do not have enough replicas to meet the desired replication factor.
To view these policies:

Go to the Monitoring page.

Go to Monitoring

In the navigation pane, select Alerting.

Review each of your alerting policies listed in the Policies section.