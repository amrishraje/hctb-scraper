# HCTB Scraper Documentation

## Account Setup
You must provide your HCTB Username and Password in order for the scaper to log into your account and get location data. This is no different than logging into the HCTB frontend itself, and your credentials will not leave HomeAssistant.

You must also provide a school code which is provided by HCTB. For individuals who are tracking more than one school code, this configuration option can be given as a comma-separated list.

## Device Setup
For each child in your HCTB account, the add-on sends live coordinates to a Home Assistant **webhook**, and you create a matching **template device tracker** so the bus shows up on your map. This is the modern replacement for the old `device_tracker.see` action, which Home Assistant removes in 2027.5.

The webhook id must be the child's first name followed by `_bus_location` (lowercase). For a child named John, add this to your `configuration.yaml` and restart Home Assistant:

```yaml
template:
  - triggers:
      - trigger: webhook
        webhook_id: john_bus_location
        allowed_methods: [POST]
        local_only: true
    device_tracker:
      - name: John Bus
        latitude: "{{ trigger.json.latitude }}"
        longitude: "{{ trigger.json.longitude }}"
        location_accuracy: "{{ trigger.json.gps_accuracy }}"
```

This creates `device_tracker.john_bus`. Repeat the block for each child, matching the `webhook_id` (`<firstname>_bus_location`) and `name`. If you previously ran this add-on with the old `device_tracker.see` output, remove the leftover `*_bus` entries from `known_devices.yaml` to avoid a conflict.

## Parking the Bus
HCTB will only show relevant rides when they are running, otherwise it just won't give any data. For these situations, HomeAssistant doesn't really have a way to "disappear" a device_tracker, so we must "park the bus" somewhere by providing a default location to fall back to.

You can park the bus anywhere you want, but for the sake of setting up notificaitons with zones in HomeAssistant, it is recommended to set the default location for the bus at the school. This must be configured by setting a default latitude and longitude in the add-on configuration.

You can use [latlong.net](https://www.latlong.net/) to find the coordinates you want to use.

## Scheduling
Due to the nature of scraping data like this, because HCTB doesn't allow for any kind of direct API access, we essentially have to poll their service to get location changes. In order to keep this process as efficient as possible, task scheduling has been implemented. This script will run every 30 seconds based on a cron schedule that you can help to provide if you wish. By default, the schedule is set to a normal school year for most kids in the US. If you need to run checks on a different schedule, you can formulate a crontab string at [crontab.guru](https://crontab.guru/). Keep in mind that you will only be providing the final four entries of a crontab, as the seconds and minutes are baked in.
