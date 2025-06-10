---
layout: default
title: AWRT – SC824ZB Pairing
---

# AWRT - Smart Relay Pairing Feature

This document outlines the functionality and requirements for the Smart Relay pairing feature with AWRT10RF thermostats and compatible relays (SC824ZB / SC812ZB).

---

## Reference Links

- **Jira Ticket**: [SAUAPP-201](https://salusinc.atlassian.net/browse/SAUAPP-201)
- **Pull Request**: [Bitbucket PR #301](https://bitbucket.org/%7B0741cce4-c632-431c-a42b-bf4ce33d34b2%7D/%7B55c24d90-7ff8-4a4e-990b-abc09bd96a14%7D/pull-requests/301)

---

## Functional Requirements

### ✅ Case 1: Display on Pairing

When the Smart Relay is **successfully paired** with an AWRT10RF thermostat, the following UI elements must be displayed:

- **Pair With** (device name or identifier)
- **Thermostat Temperature**
- **Thermostat Set Point**
- **Smart Relay State** (ON/OFF)

### 🚫 Case 2: Hidden Fields When Unpaired

If the Smart Relay is **not paired** with any AWRT10RF thermostat, the above fields must remain hidden.

### 🔁 Case 3: Smart Relay State

- The **Smart Relay State** must reflect the **real-time ON/OFF state** of the relay post successful pairing.

### ⚠️ Case 4: Exclusion of Pin Position

- The **Pin Position** attribute is **not required** for the Smart Relay.
- This property is relevant only for TRVs (Thermostatic Radiator Valves).

---

## Pairing Logic

Pairing is based on **Short ID Matching**:

- If the **Short ID** of the thermostat **matches** the **Paired Short ID** of the Smart Relay, they are considered **paired**.
- If they **do not match**, the devices are considered **unpaired**.

### 🔍 Pairing Validation

A utility function checks the pairing status and returns a **boolean** value:

- `true`: Relay is paired with a thermostat.
- `false`: Relay is not paired with any thermostat.

Ensure this logic is implemented correctly for reliable pairing state validation.

```dart

  bool checkIfRelayIsPaired(pairValue, selectedIndex) {
    SliderListItemDeviceModel? thermostatModel;
    final List<SliderListItemDeviceModel> radiantThermostats = AppDevices()
            .getCurrentIndexSlider(selectedIndex: selectedIndex)
            ?.getRadiantThermostat() ??
        [];

    if (radiantThermostats.isNotEmpty && radiantThermostats != null) {
      for (final thermostat in radiantThermostats) {
        if (thermostat.getShortId() == pairValue.toString()) {
          thermostatModel = thermostat;
        }
      }
    }
    if (thermostatModel != null) {
      return true;
    }
    return false;
  }

```

---

