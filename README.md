Home Assistant Tuya Local component
===================================

The `tuya_local` component integrates 
[Goldair WiFi-enabled heaters](http://www.goldair.co.nz/product-catalogue/heating/wifi-heaters), WiFi-enabled [dehumidifiers](http://www.goldair.co.nz/product-catalogue/heating/dehumidifiers), [WiFi-enabled fans](http://www.goldair.co.nz/product-catalogue/cooling/pedestal-fans/40cm-dc-quiet-fan-with-wifi-and-remote-gcpf315) and [Kogan WiFi-enabled heaters](https://www.kogan.com/au/c/smarterhome-range/shop/heating-cooling/) into Home Assistant, enabling control of setting the following parameters via the UI and the following services:

### Climate devices
**Goldair Heaters**
* **power** (on/off)
* **mode** (Comfort, Eco, Anti-freeze)
* **target temperature** (`5`-`35` in Comfort mode, `5`-`21` in Eco mode, in °C)
* **power level** (via the swing mode setting because no appropriate HA option exists: `Auto`, `1`-`5`, `Stop`)

Current temperature is also displayed.

**Goldair Demudifiers**
* **power** (on/off)
* **mode** (Normal, Low, High, Dry clothes, Air clean)
* **target humidity** (`30`-`80`%)

Current temperature is displayed, and current humidity is available as a property. The "tank full" state is available via the **error** attribute, and if you want to you can easily surface this to a top-level entity using a [template sensor](https://www.home-assistant.io/integrations/template/).

**Goldair Fans**
* **power** (on/off)
* **mode** (Normal, Eco, Sleep)
* **fan mode** (`1`-`12`)
* **swing** (on/off)

**Kogan Heaters**
* **power** (on/off)
* **mode** (LOW/HIGH)
* **target temperature** (`16`-`30` in °C)

Current temperature is also displayed.

### Additional features
**Light** (Goldair devices)
* **LED display** (on/off)

**Lock** (Goldair heaters and dehumidifiers)
* **Child lock** (on/off)

There was previously a sensor option, however this is easily achieved using a [template sensor](https://www.home-assistant.io/integrations/template/) and therefore is no longer supported.

---

### Warning
Please note, this component has currently only been tested with the Goldair GPPH (inverter), GPDH420 (dehumidifier), and GCPF315 fan, however theoretically it should also work with GEPH and GPCV heater devices, may work with the GPDH440 dehumidifier and any other Goldair heaters, dehumidifiers or fans based on the Tuya platform.

Kogan heater support is tested with the Kogan SmarterHome 1500W Smart Panel Heater.  If you have another type of Kogan SmarterHome heater, it may or may not work with the same configuration.

---

Installation
------------
The preferred installation method is via [HACS](https://hacs.xyz/). Once you have HACS set up, simply follow the [instructions for adding a custom repository](https://hacs.xyz/docs/navigation/settings#custom-repositories) and then the integration will be available to install like any other.

You can also use [Custom Updater](https://github.com/custom-components/custom_updater). Once Custom Updater is  set up, go to the Developer Tools > Service page and call the `custom_updater.install` service with this service data:

```json
{ "element": "tuya_local" }
```

Alternatively you can copy the contents of this repository's `custom_components` directory to your `<config>/custom_components` directory, however you will not get automatic updates this way.

Configuration
-------------
You can easily configure your devices using the Integrations UI at `Home Assistant > Configuration > Integrations > +`. This is the preferred method as things will be unlikely to break as this integration is upgraded. You will need to provide your device's IP address, device ID and local key; the last two can be found using [the instructions below](#finding-your-device-id-and-local-key).

If you would rather configure using yaml, add the following lines to your `configuration.yaml` file (but bear in mind that if the configuration options change your configuration may break until you update it to match the changes):

```yaml
# Example configuration.yaml entry
tuya_local:
  - name: My heater
    host: 1.2.3.4
    device_id: <your device id>
    local_key: <your local key>
```

### Configuration variables

#### name
&nbsp;&nbsp;&nbsp;&nbsp;*(string) (Required)* Any unique for the device; required because the Tuya API doesn't provide
                                              the one you set in the app.

#### host
&nbsp;&nbsp;&nbsp;&nbsp;*(string) (Required)* IP or hostname of the device.

#### device_id
&nbsp;&nbsp;&nbsp;&nbsp;*(string) (Required)* Device ID retrieved 
                                              [as per the instructions below](#finding-your-device-id-and-local-key).

#### local_key
&nbsp;&nbsp;&nbsp;&nbsp;*(string) (Required)* Local key retrieved 
                                              [as per the instructions below](#finding-your-device-id-and-local-key).

#### type
&nbsp;&nbsp;&nbsp;&nbsp;*(string) (Optional)* The type of Tuya device. `auto` to automatically detect the device type, or if that doesn't work, select from the available options `heater`, `dehumidifier`, `fan` or `kogan_heater`.

&nbsp;&nbsp;&nbsp;&nbsp;*Default value: auto*

#### climate
&nbsp;&nbsp;&nbsp;&nbsp;*(boolean) (Optional)* Whether to surface this appliance as a climate device.

&nbsp;&nbsp;&nbsp;&nbsp;*Default value: true* 

#### display_light
&nbsp;&nbsp;&nbsp;&nbsp;*(boolean) (Optional)* Whether to surface this appliance's LED display control as a light (not supported for Kogan Heaters).

&nbsp;&nbsp;&nbsp;&nbsp;*Default value: false* 

#### child_lock
&nbsp;&nbsp;&nbsp;&nbsp;*(boolean) (Optional)* Whether to surface this appliances's child lock as a lock device (not supported for fans).

&nbsp;&nbsp;&nbsp;&nbsp;*Default value: false* 

Heater gotchas
--------------
Goldair heaters have individual target temperatures for their Comfort and Eco modes, whereas Home Assistant only supports
a single target temperature. Therefore, when you're in Comfort mode you will set the Comfort temperature (`5`-`35`), and
when you're in Eco mode you will set the Eco temperature (`5`-`21`), just like you were using the heater's own control 
panel. Bear this in mind when writing automations that change the operation mode and set a temperature at the same time: 
you must change the operation mode *before* setting the new target temperature, otherwise you will set the current 
thermostat rather than the new one. 

When switching to Anti-freeze mode, the heater will set the current power level to `1` as if you had manually chosen it.
When you switch back to other modes, you will no longer be in `Auto` and will have to set it again if this is what you
wanted. This could be worked around in code however it would require storing state that may be cleared if HA is
restarted and due to this unreliability it's probably best that you just factor it into your automations.

When child lock is enabled, the heater's display will flash with the child lock symbol (`[]`) whenever you change
something in HA. This can be confusing because it's the same behaviour as when you try to change something via the
heater's own control panel and the change is rejected due to being locked, however rest assured that the changes *are* 
taking effect.

Fan gotchas
-----------
In my experience, fans don't like to receive a lot of commands in a short period of time. They will emit two fast beeps when accepting a command, and two slow beeps when rejecting one. If you are writing automations for your fan and need to set multiple properties, you may need to put a delay of around 5 seconds or more between each.

Finding your device ID and local key 
------------------------------------
You can find these keys the same way as you would for any Tuya local integration.

* [Instructions from tuyapi](https://github.com/codetheweb/tuyapi/blob/master/docs/SETUP.md)


Next steps
----------
1. The devices need to be generalized so a new subdirectory with source code is not needed to add a new device.  Instead, device descriptors should be in a yaml file, which is referenced by the config.
2. Support for non-climate devices needs to be added.  For starters, I have some Kogan Power Monitoring Plugs that I haven't yet taken out of the box and converted to ESPHome, but this will probably need input from other users.
3. This component needs specs! Once they're written I'm considering submitting it to the HA team for inclusion in standard installations. Please report any issues and feel free to raise pull requests.

Acknowledgements
----------------
None of this would have been possible without some foundational discovery work to get me started:

* [nikrolls](https://github.com/nikrolls)'s [homeassistant-goldair-climate](https://github.com/nikrolls/homeassistant-goldair-climate) was the starting point for expanding to non-Goldair devices as well
* [TarxBoy](https://github.com/TarxBoy)'s [investigation using codetheweb/tuyapi](https://github.com/codetheweb/tuyapi/issues/31) to figure out the correlation of the cryptic DPS states 
* [sean6541](https://github.com/sean6541)'s [tuya-homeassistant](https://github.com/sean6541/tuya-homeassistant) library giving an example of integrating Tuya devices with Home Assistant
* [clach04](https://github.com/clach04)'s [python-tuya](https://github.com/clach04/python-tuya) library