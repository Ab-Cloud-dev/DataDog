# Datadog Integration with Ubuntu EC2

A technical guide for installing Datadog Agent on Ubuntu EC2, monitoring CPU metrics, and configuring alerts.

## Prerequisites

- Ubuntu EC2 instance ( with sudo access
- Datadog account and API key from [Datadog API Keys page](https://app.datadoghq.com/organization-settings/api-keys)
- SSH access to EC2 instance

## 1. Datadog Agent Installation

### Install Agent via APT
```bash
# Set your Datadog API key
DD_API_KEY=<key> \
DD_SITE="us5.datadoghq.com" \
DD_REMOTE_UPDATES=true \
DD_APM_INSTRUMENTATION_ENABLED=host \
DD_DATA_STREAMS_ENABLED=true \
DD_PROFILING_ENABLED=auto \
DD_ENV=dev \
DD_APM_INSTRUMENTATION_LIBRARIES=java:1,python:3,js:5,php:1,dotnet:3,ruby:2 \
bash -c "$(curl -L https://install.datadoghq.com/scripts/install_script_agent7.sh)"  # Use datadoghq.eu for EU region

# Download and run the installation script
DD_API_KEY=$DD_API_KEY DD_SITE=$DD_SITE bash -c "$(curl -L https://s3.amazonaws.com/dd-agent/scripts/install_script_agent7.sh)"
```

### Verify Installation
```bash
# Check agent status
sudo datadog-agent status

# Verify agent is running
sudo systemctl status datadog-agent
```
![alt text](image.png)

![alt text](image-1.png)

### Configure Hostname (Optional)
```bash
# Edit agent configuration
sudo nano /etc/datadog-agent/datadog.yaml

# Set custom hostname (uncomment and modify):
# hostname: my-ec2-instance

# Restart agent
sudo systemctl restart datadog-agent
```

## 2. CPU Load Generation

### Install Stress Testing Tools
```bash
# Install stress utility
sudo apt-get update
sudo apt-get install -y stress

# Alternative: Install sysbench
sudo apt-get install -y sysbench
```

### Generate CPU Load

**Using `stress`:**
```bash
# Generate 100% CPU load on all cores for 5 minutes
stress --cpu $(nproc) --timeout 300s

```


## 3. Dashboard Setup

### Access Datadog Dashboard

1. Navigate to [Datadog Dashboards](https://app.datadoghq.com/dashboard/lists)
2. Click **New Dashboard**
3. Name: `EC2 CPU Monitoring`

### Add CPU Metrics Widget

**Via UI:**
1. Click **Add Widget** → **Timeseries**
2. Select metric: `system.cpu.user` or `system.cpu.system`
3. Filter by: `host:<YOUR_HOSTNAME>`
4. Click **Save**

![alt text](image-2.png)

This enables the built-in “Process Check” for the Agent.

## 4. Alert Configuration

### Create CPU Alert Monitor

**Via UI:**
1. Go to [Monitors → New Monitor](https://app.datadoghq.com/monitors/create)


- In the Datadog UI go to **Monitors → New Monitor → Metric**. [Datadog]
- Configure as follows:

- **Metric**: `system.cpu.user` (or choose `system.cpu.system` / `100 - system.cpu.idle`)
- **Scope**: `host:${HOSTNAME}`
- **Condition**: `avg(last_5m) > 80`
- **Name**: “High CPU on ${HOSTNAME}”
![alt text](image-4.png)
![alt text](image-3.png)

![alt text](image-5.png)
### Monitor Configuration Details

| Parameter | Value | Description |
|-----------|-------|-------------|
| **Metric** | `system.cpu.user` | User CPU usage percentage |
| **Threshold** | `> 80` | Alert when CPU exceeds 80% |
| **Evaluation Window** | `last_5m` | Average over 5 minutes |
| **Notification** | Email | Sends to specified email address |

### Email Notification Format

Replace `your-email@example.com` with your actual email:
```
"message": "⚠️ **High CPU Alert**\n\nCPU utilization is above 80%\n\n- Host: {{host.name}}\n- Current value: {{value}}%\n\n@your-email@example.com"
```

## Validation

### Test the Complete Setup
```bash
# 1. Verify agent is reporting
sudo datadog-agent status | grep -A 5 "API Keys status"

# 2. Generate CPU load
stress --cpu $(nproc) --timeout 600s

# 3. Check metrics in Datadog (wait 1-2 minutes for data)
# Navigate to: Metrics Explorer → system.cpu.user

# 4. Verify alert triggers (wait ~5 minutes during load test)
# Check: Monitors page for alert status
```

## Troubleshooting
```bash
# View agent logs
sudo tail -f /var/log/datadog/agent.log

# Check connectivity
sudo datadog-agent diagnose

# Restart agent
sudo systemctl restart datadog-agent

# Force metric submission
sudo datadog-agent check cpu
```

## Cleanup
```bash
# Stop load generation
killall stress sysbench dd

# Uninstall agent (optional)
sudo apt-get remove --purge datadog-agent -y
sudo rm -rf /etc/datadog-agent /opt/datadog-agent /var/log/datadog
```

## Additional Resources

- [Datadog Agent Documentation](https://docs.datadoghq.com/agent/)
- [Datadog API Reference](https://docs.datadoghq.com/api/latest/)
- [System Check Metrics](https://docs.datadoghq.com/integrations/system/)