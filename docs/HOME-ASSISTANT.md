# Home Assistant Integration

Send notifications from Sonarr/Radarr to Home Assistant.

## Prerequisites

- Home Assistant accessible from your Docker network
- Gluetun's `FIREWALL_OUTBOUND_SUBNETS` includes your LAN (e.g., `192.168.0.0/24`)

**Important:** Use `.lan` TLD, not `.local`. Docker containers can't resolve `.local` domains (mDNS reserved).

## Step 1: Create HA Automation

In Home Assistant: Settings → Automations → Create → Edit in YAML:

```yaml
alias: Arr Stack Notifications
trigger:
  - platform: webhook
    webhook_id: arr-notifications
    local_only: false
action:
  - service: notify.persistent_notification
    data:
      title: >
        {% if trigger.json.series %}
          {{ trigger.json.series.title }}
        {% elif trigger.json.movie %}
          {{ trigger.json.movie.title }}
        {% else %}
          {{ trigger.json.eventType }}
        {% endif %}
      message: >
        {% if trigger.json.episodes %}
          S{{ trigger.json.episodes[0].seasonNumber }}E{{ trigger.json.episodes[0].episodeNumber }} - {{ trigger.json.episodes[0].title }}
        {% elif trigger.json.movie %}
          ({{ trigger.json.movie.year }}) - {{ trigger.json.eventType }}
        {% else %}
          {{ trigger.json.eventType }}
        {% endif %}
```

Change `notify.persistent_notification` to `notify.mobile_app_your_phone` for push notifications.

## Step 2: Configure Sonarr/Radarr

**Sonarr:** Settings → Connect → Add → Webhook
- URL: `http://homeassistant.lan:8123/api/webhook/arr-notifications`
- Events: On Grab, On Download, On Upgrade

**Radarr:** Same URL and events.

Click **Test** to verify.

## Uptime Kuma → Home Assistant

Requires `docker-compose.utilities.yml` deployed.

In Uptime Kuma: Settings → Notifications → Setup Notification
- Type: Home Assistant
- URL: `http://homeassistant.lan:8123`
- Long-Lived Access Token: (create in HA → Profile → Long-Lived Access Tokens)
