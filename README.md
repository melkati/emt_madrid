_Please :star: this repo if you find it useful_

# EMT Madrid for Home Assistant

Custom integration for Home Assistant that provides real-time bus arrival information from EMT Madrid (Empresa Municipal de Transportes). Automatically finds nearby bus stops based on your location.

Thanks to [EMT Madrid MobilityLabs](https://mobilitylabs.emtmadrid.es/) for providing the data and [documentation](https://apidocs.emtmadrid.es/).

## Features

- **Dynamic stop detection** - Automatically finds bus stops near your configured location
- **Config Flow UI** - Easy setup through Home Assistant UI (no YAML needed)
- **Voice assistant ready** - Built-in speech text for Alexa integration
- **Custom coordinates** - Use home location or specify custom coordinates
- **Additional stops** - Optionally monitor specific stop IDs

## Prerequisites

You need application credentials from [EMT MobilityLabs](https://mobilitylabs.emtmadrid.es/).

### Register your own app

1. Go to [mobilitylabs.emtmadrid.es](https://mobilitylabs.emtmadrid.es/) and register
2. Create a new application in the portal
3. You'll receive an `X-ClientId` (UUID) and `passKey` (hex string)
4. In Home Assistant, enter these as **Email** (`X-ClientId`) and **Password** (`passKey`)

> **Note**
> This integration authenticates using application credentials only. Direct login with a MobilityLabs account email/password is not supported.

## Installation

### HACS (Recommended)

1. Open HACS in Home Assistant
2. Go to "Integrations" → Three dots menu → "Custom repositories"
3. Add `https://github.com/melkati/emt_madrid` as Integration
4. Search for "EMT Madrid" and install
5. Restart Home Assistant

### Manual

1. Copy the `custom_components/emt_madrid` folder to your Home Assistant `config/custom_components/` directory
2. Restart Home Assistant

## Configuration

1. Go to **Settings** → **Devices & Services** → **Add Integration**
2. Search for **"EMT Madrid"**
3. Enter your credentials and options:

### Migration from YAML

If you have an existing YAML configuration, it will be automatically imported on restart. After migration:
1. The integration will appear in **Settings** → **Devices & Services**
2. You can remove the `emt_madrid:` section from your `configuration.yaml`
3. Use the UI to manage settings going forward

### Edit Configuration

To change settings after setup:
1. Go to **Settings** → **Devices & Services**
2. Find **EMT Madrid** → Click **Configure**
3. Modify radius, coordinates, or stop IDs

### Options

| Option | Required | Default | Description |
|--------|----------|---------|-------------|
| Email | Yes | - | `X-ClientId` (UUID from your EMT MobilityLabs app) |
| Password | Yes | - | `passKey` (hex string from your EMT MobilityLabs app) |
| Radius | No | 300 | Search radius in meters (50-1000) |
| Latitude | No | zone.home | Custom latitude coordinate |
| Longitude | No | zone.home | Custom longitude coordinate |
| Stop IDs | No | - | Additional stop IDs (comma-separated) |

## Sensor

The integration creates a sensor `sensor.emt_nearby_buses` that shows:

**State:** Next bus arrival (e.g., "27 en 3 min")

### Attributes

| Attribute | Description |
|-----------|-------------|
| `arrivals` | List of upcoming bus arrivals |
| `stops_count` | Number of stops being monitored |
| `speech` | Voice-ready text in Spanish |
| `radius` | Configured search radius |
| `latitude` | Current latitude |
| `longitude` | Current longitude |
| `extra_stops` | Additional configured stop IDs |

## Service

### `emt_madrid.get_nearby_arrivals`

Get bus arrivals for any location.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `latitude` | zone.home | Latitude coordinate |
| `longitude` | zone.home | Longitude coordinate |
| `radius` | 300 | Search radius (50-1000m) |
| `max_results` | 5 | Max arrivals (1-20) |

**Response:**
```yaml
arrivals:
  - line: "27"
    minutes: 3
    stop_name: "Cibeles"
    destination: "Plaza Castilla"
speech: "Línea 27 en 3 minutos, Línea 5 en 7 minutos."
count: 2
```

## Alexa Integration

Ask Alexa when the next bus arrives:

### 1. Add to `configuration.yaml`:

```yaml
input_boolean:
  emt_nearby_buses_trigger:
    name: EMT Buses Trigger
    icon: mdi:bus

script:
  emt_nearby_buses:
    alias: "Nearby EMT Buses"
    sequence:
      - action: emt_madrid.get_nearby_arrivals
        response_variable: bus_data
      - action: notify.alexa_media_last_called
        data:
          message: "{{ bus_data.speech }}"
          data:
            type: tts

automation:
  - alias: "EMT Buses Voice Response"
    trigger:
      - platform: state
        entity_id: input_boolean.emt_nearby_buses_trigger
        to: "on"
    action:
      - action: script.emt_nearby_buses
      - action: input_boolean.turn_off
        target:
          entity_id: input_boolean.emt_nearby_buses_trigger
```

### 2. Create Alexa Routine:
- Open Alexa app → More → Routines
- Trigger: Voice → "buses cercanos"
- Action: Smart Home → Control Device → `EMT Buses Trigger` → Turn On

### 3. Say: "Alexa, buses cercanos"

**Requirements:** [Alexa Media Player](https://github.com/alandtse/alexa_media_player) and Nabu Casa (or Alexa Smart Home skill).

## Changelog

### v2.2.1
- **Fixed authentication** — EMT API now requires `X-ClientId`/`passKey` headers for login. The integration was failing with "Invalid credentials" errors because the API changed its authentication scheme.
- **Session persistence** — Uses `requests.Session()` to maintain authentication cookies across API calls, which the EMT API requires.
- **Both response codes accepted** — Login may return code `"00"` (fresh token) or `"01"` (cached token); both are now accepted as valid.
- **UI labels updated** — Credential fields now clearly show "Client ID" and "passKey" to avoid confusion with legacy email/password credentials.

### v2.2.0
- Add Config Flow and Options Flow
- Add nearby stops detection via GPS
- Add `get_nearby_arrivals` service
- Add Alexa voice integration support
- Migrate from YAML-only to UI-based configuration

### v2.0.0
- Full rewrite with new architecture
- HACS support added
