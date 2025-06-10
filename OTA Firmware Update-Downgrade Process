# OTA Firmware Upgrade / Downgrade Procedure

This document outlines the steps to perform an OTA Firmware **Upgrade or Downgrade** for Gateway and Coordinator devices using AWS IoT Core via MQTT topics.

---

## Prerequisites

- Access to AWS IoT Core with appropriate permissions.
- MQTT client configured to publish to AWS device shadow topics.
- Valid OTA firmware URLs for the desired firmware versions.

---

## Updating Gateway Firmware

### Steps:

1. Open the AWS IoT **Device Shadow** for the Gateway you want to update.
2. Navigate to the MQTT topic for **shadow update** (usually: `\$aws/things/{thingName}/shadow/update`).
3. Use the following JSON payload to publish the firmware URL and trigger the OTA update.

### Sample JSON Payload

```json
{
  "state": {
    "desired": {
      "000000000001": {
        "properties": {
          "ep0:sOTA:SetOTAFirmwareURL_d": "http://ec2-18-194-20-66.eu-central-1.compute.amazonaws.com/debug/OTA_Test/SAU2AG1GW/SAU2AG1-GW_020154220705.tar.gz"
        }
      }
    }
  }
}
```

### Few Gateway Firmware URLs

- **Version: 020143210405** : [http://ec2-18-194-20-66.eu-central-1.compute.amazonaws.com/debug/OTA_Test/SAU2AG1GW/SAU2AG1-GW_020143210405.tar.gz](http://ec2-18-194-20-66.eu-central-1.compute.amazonaws.com/debug/OTA_Test/SAU2AG1GW/SAU2AG1-GW_020143210405.tar.gz)
- **Version: 021700240605** : [https://pub-na.s3.us-west-2.amazonaws.com/OTA/UGs/UG600/GW/2024-0607/SAU2AG1-GW_021700240605.tar.gz](https://pub-na.s3.us-west-2.amazonaws.com/OTA/UGs/UG600/GW/2024-0607/SAU2AG1-GW_021700240605.tar.gz)
- **Version: 020154220705** : [http://ec2-18-194-20-66.eu-central-1.compute.amazonaws.com/debug/OTA_Test/SAU2AG1GW/SAU2AG1-GW_020154220705.tar.gz](http://ec2-18-194-20-66.eu-central-1.compute.amazonaws.com/debug/OTA_Test/SAU2AG1GW/SAU2AG1-GW_020154220705.tar.gz)

---

## Updating Coordinator Firmware

### Steps:

1. Open the AWS IoT **Device Shadow** for the Coordinator device.
2. Navigate to the MQTT topic for **shadow update**.
3. Publish the following JSON payload to initiate the firmware update.

### Sample JSON Payload

```json
{
  "state": {
    "desired": {
      "000000000002": {
        "properties": {
          "ep0:sOTA:SetOTAFirmwareURL_d": "http://ec2-18-194-20-66.eu-central-1.compute.amazonaws.com/firmware/UG-SYS/UG600/ZC/SAU2AG1-ZC_20221028.tar.gz"
        }
      }
    }
  }
}
```

### Few Coordinator Firmware URLs

- **Version: 20221028** : [http://ec2-18-194-20-66.eu-central-1.compute.amazonaws.com/firmware/UG-SYS/UG600/ZC/SAU2AG1-ZC_20221028.tar.gz](http://ec2-18-194-20-66.eu-central-1.compute.amazonaws.com/firmware/UG-SYS/UG600/ZC/SAU2AG1-ZC_20221028.tar.gz)
- **Version: 20231201** : [http://ec2-18-194-20-66.eu-central-1.compute.amazonaws.com/firmware/UG-SYS/UG600/ZC/SAU2AG1-ZC_20231201.tar.gz](http://ec2-18-194-20-66.eu-central-1.compute.amazonaws.com/firmware/UG-SYS/UG600/ZC/SAU2AG1-ZC_20231201.tar.gz)
- **Version: 20240531** : [https://pub-na.s3.us-west-2.amazonaws.com/OTA/UGs/UG600/ZC/SAU2AG1-ZC_20240531.tar.gz](https://pub-na.s3.us-west-2.amazonaws.com/OTA/UGs/UG600/ZC/SAU2AG1-ZC_20240531.tar.gz)

---

## Notes

- Make sure the firmware URL is publicly accessible from the IoT devices.
- After the update, monitor the AWS IoT shadow for update status.
