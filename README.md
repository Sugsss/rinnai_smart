# Rinnai Smart for Home Assistant

Support for [Rinnai Smart Water Heater (E86) monitoring and control device](https://www.rinnai.com.cn/productDetail-646a2c67-29d6-4aff-8ff2-167cc1d33367.html) for Home Assistant.

## WARNING

* **RINNAI DOESN'T PROVIDE ANY OFFICIALLY SUPPORTED API, THUS THEIR CHANGES MAY BREAK HASS INTEGRATIONS AT ANY TIME.**
* **USE IT AS YOUR OWN RISK. YOUR ACCOUNT MAY BE BANNED.**
* **THIS INTEGRATION IS IN ALPHA STATE. ONCE CONNECTION LOST, RECONFIGURE THE DEVICE AGAIN.**

## IMPORTANT NOTES

* **THIS LIBRARY ONLY WORKS ON RUS-R16E86FBF.**

### Features

- water heater:
    * set operating temperature (&deg;C)
    * set operating mode
- set recirculation mode
- set recirculation reservation
- set temporary recirculation
- multiple Rinnai devices

![](./screenshot.png)

## Installation

#### Versions

The 'main' branch of this custom component is considered unstable, alpha quality and not guaranteed to work.
Please make sure to use one of the official release branches when installing using HACS, see [what has changed in each version](https://github.com/catro/rinnai_smart/releases).

#### With HACS

[![Open your Home Assistant instance and open a repository inside the Home Assistant Community Store.](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=catro&repository=rinnai_smart&category=integration)

#### Manual
1. Copy the `rinnai_smart` directory from `custom_components` in this repository and place inside your Home Assistant's `custom_components` directory.
2. Restart Home Assistant
3. Follow the instructions in the `Setup` section

> [!WARNING]
> If installing manually, in order to be alerted about new releases, you will need to subscribe to releases from this repository.

# Setup
[![Open your Home Assistant instance and start setting up a new integration.](https://my.home-assistant.io/badges/config_flow_start.svg)](https://my.home-assistant.io/redirect/config_flow_start/?domain=rinnai)

> [!Tip]
> If you are unable to use the button above, follow the steps below:
> 1. Navigate to the Home Assistant Integrations page `(Settings --> Devices & Services)`
> 2. Click the `+ ADD INTEGRATION` button in the lower right-hand corner
> 3. Search for `Rinnai Smart`
## Device and entity selection

Setup validates your Rinnai Smart account, then shows two selection steps:

1. **Devices:** choose heaters by their account name and MAC address. New setups
   default to **Include** with all currently listed devices checked. Uncheck any
   device you do not want to import. Later account additions are not automatically
   imported in this mode.
2. **Entities:** choose individual functions for each selected heater, such as
   its water heater entity, burning sensor, or recirculation switch. This step
   defaults to **Exclude** with no exclusions, importing all functions of the
   selected devices.

Both steps support **Include** (only checked items) and **Exclude** (all items
except those checked). An empty Include list imports nothing; an empty Exclude
list imports everything. Exclude mode also permits future items, subject to the
other filter. Entity choices are per device and use stable unique IDs, so renaming
an entity does not change its selection. Filtering a separate mode selector does
not remove the operating-mode feature of the water heater entity.

Use **Settings → Devices & services → Rinnai Smart → Configure** to change the
selection later without entering credentials again. Save both steps to apply the
changes; canceling leaves the previous selection intact. Saved choices temporarily
absent from the account are retained and displayed by ID. The integration reloads
on save and closes its previous MQTT connection before starting another.

Existing entries without filter options retain the original import-all behavior.
Unselected devices (or devices with no selected entities) receive no state-detail
requests or MQTT subscriptions. The cloud account's device list is still needed
for discovery. An intentionally empty selection is valid and does not start MQTT.

Previously created entities that are excluded become **disabled by the
integration**. Their registry records, IDs, custom names, and device records are
retained. Selecting them again restores them; entities you manually disabled remain
disabled. Existing dashboard cards or automations referencing an excluded entity
must be adjusted separately.

## Development tests

With Python 3.14, install `requirements_test.txt` into a virtual environment, then
run `python -m pytest -q --disable-socket --allow-unix-socket`. The tests use
synthetic device data and mocked HTTP/MQTT transports, including Home Assistant's
flow manager, platform setup, entity registry, and options reload. They do not
contact Rinnai or control a physical heater.

## Home Assistant 2026.11 compatibility (this fork only)

This fork (`Sugsss/rinnai_smart`) is upstream `0.0.7` plus one fix:

- **Problem:** HA Core 2026.11 removed the `PRECISION_WHOLE` re-export from
  `homeassistant.components.water_heater`. Importing it there made
  `water_heater.py` fail at import time, aborting the whole config entry
  (`Error setting up entry ... for rinnai`) and leaving every Rinnai entity
  unavailable.
- **Fix:** import `PRECISION_WHOLE` from `homeassistant.const`, where it still
  exists (`1`). This is backward compatible with older HA versions, which only
  re-exported the same constant.
- **Verified** on HAOS 2026.11.0: after a core restart the config entry loads,
  the `water_heater` platform imports, and all 7 registered `rinnai` entities
  are enabled.
- **Upstream:** [PR #19](https://github.com/catro/rinnai_smart/pull/19)
  (an independently verified alternative to the earlier
  [PR #18](https://github.com/catro/rinnai_smart/pull/18)). Once either is
  merged, prefer the upstream release again.

### Installing this build

The directory in this repository is `custom_components/rinnai_smart`, while the
manifest `domain` is `rinnai`. Home Assistant indexes custom integrations by the
manifest domain, so an existing "Rinnai Smart" / 林内智家 config entry still
matches. **Never keep both** `custom_components/rinnai` and
`custom_components/rinnai_smart` on the same machine: both declare domain
`rinnai`. Remove/rename the old directory before installing, or simply keep the
already-patched `custom_components/rinnai` in place.

For HACS, add this repository as a custom *Integration* repository and install
the `0.0.7-ha2026.11` release. `main` contains the same patched build.

### Known follow-ups (not changed here)

- HA 2026.11 warns that `_attr_temperature_unit` and `target_temperature` are
  deprecated and stop working in 2027.11 (use `native_*` instead).
- A blocking `import_module` warning is logged during platform forwarding.
