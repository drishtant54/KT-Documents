## OTA Update Feature Overview

## 🔧 Feature: OTA Update  
**Ticket:** [SAUAPP-119](https://salusinc.atlassian.net/browse/SAUAPP-119)

This feature enables OTA Firmware updates for both **Coordinator**, **Gateway** and change **Coordinator Channel ID**. It includes automatic channel validation and adjustment for the coordinator if the current channel is not among the recommended values (15, 20, or 25).

---

## 🚀 Update Process Flow

1. **Update Coordinator Firmware**  
2. **Update Gateway Firmware**  
3. **Change Coordinator Channel** (if necessary)  
4. **Refresh Coordinator Data** – Initiated 10 seconds after firmware updates complete.
   
Below function helps to call the update API of coordinator , gateway also changes the channel of the coordinator if the channel is not 15, 20 or 25.

```dart

Future<void> checkAndUpdateFirmware(SliderListItemDeviceModel item) async {
    GatewayModel gatewayModel = GatewayModel.fromJson(item.shadow ?? {});
    bool gatewayNeedsUpdate = false;
    bool coordinatorNeedsUpdate = false;
    bool coordinatorNeedsChannelChange = false;

    if (gatewayModel != null) {
      CoordinatorModel coordinatorModel =
          CoordinatorModel.fromJson(item?.coordinatorDevice?.shadow ?? {});

      if (coordinatorModel != null) {
        _statusController.add(FirmwareUpdateStatus.checking);

        // Check Coordinator Firmware Update
        coordinatorNeedsUpdate =
            await FirmwareUpdaterService().checkAndUpdateFirmware(
          item: item!.coordinatorDevice!,
          sOtaModel: coordinatorModel!.reported!.model!.properties!.ep0SOta!,
          firmwareVersion: coordinatorModel
                  ?.reported?.model?.properties?.ep0SZdo?.firmwareVersion ??
              "",
        );

        logger.i(
            "[GatewayUpdate] COORDINATOR FIRMWARE UPDATE RESULT:- $coordinatorNeedsUpdate");

        // Check if Coordinator Channel Needs Update
        if (coordinatorModel?.reported?.model?.properties?.ep0SCoord
                ?.needChannelChange() ??
            false) {
          coordinatorNeedsChannelChange = coordinatorModel
                  ?.reported?.model?.properties?.ep0SCoord
                  ?.needChannelChange() ??
              false;
          Map<String, dynamic> publishChannelChangeCommand = coordinatorModel
                  ?.reported?.model?.properties?.ep0SCoord
                  ?.publishSetZigbeeCommandDToUpdateChannel() ??
              {};
          if (publishChannelChangeCommand.isNotEmpty) {
            await Future.delayed(Duration(seconds: 5));
            await coordinatorBloc.publish(publishChannelChangeCommand,
                item.coordinatorDevice!.deviceCode!);
            await refershCoordinator(coordinatorModel, item);
          }
        }
      }

      // Check Gateway Firmware Update
      gatewayNeedsUpdate =
          await FirmwareUpdaterService().checkAndUpdateFirmware(
        item: item,
        sOtaModel: gatewayModel!.reported!.model!.properties!.ep0SOta!,
        firmwareVersion: gatewayModel?.reported?.model?.properties?.ep0SGateway
                ?.getGatewaySoftwareVersion() ??
            "",
      );

      logger.i(
          "[GatewayUpdate] GATEWAY FIRMWARE UPDATE RESULT:- $gatewayNeedsUpdate");

      if (coordinatorNeedsUpdate ||
          gatewayNeedsUpdate ||
          coordinatorNeedsChannelChange) {
        await getShadow(item, screenMode: ScreenMode.gatewayUpdate);

        if (!(gatewayModel.reported?.isConnected() == true)) {
          _statusController.add(FirmwareUpdateStatus.failed);
        }
        _statusController.add(FirmwareUpdateStatus.downloading);

        await Future.delayed(const Duration(seconds: 70), () async {
          await getShadow(item, screenMode: ScreenMode.gatewayUpdate);
        });

        if (!(gatewayModel.reported?.isConnected() == true)) {
          _statusController.add(FirmwareUpdateStatus.failed);
        }
        _statusController.add(FirmwareUpdateStatus.installing);

        await Future.delayed(const Duration(seconds: 70), () async {
          await getShadow(item, screenMode: ScreenMode.gatewayUpdate);
        });
        if (!(gatewayModel.reported?.isConnected() == true)) {
          _statusController.add(FirmwareUpdateStatus.failed);
        }
        _statusController.add(FirmwareUpdateStatus.rebooting);

        await Future.delayed(const Duration(seconds: 55), () async {
          await getShadow(item, screenMode: ScreenMode.gatewayUpdate);
        });
        if (!(gatewayModel.reported?.isConnected() == true)) {
          _statusController.add(FirmwareUpdateStatus.failed);
        }
        _statusController.add(FirmwareUpdateStatus.validating);

        await Future.delayed(const Duration(seconds: 45), () async {
          await getShadow(item, screenMode: ScreenMode.gatewayUpdate);
        });
        if (!(gatewayModel.reported?.isConnected() == true)) {
          _statusController.add(FirmwareUpdateStatus.failed);
        }
        _statusController.add(FirmwareUpdateStatus.success);
      } else {
        if (!(gatewayModel.reported?.isConnected() == true)) {
          _statusController.add(FirmwareUpdateStatus.failed);
        }
        _statusController.add(FirmwareUpdateStatus.alreadyUptoDate);
      }
    }
  }


```
Refresh All Data for coordinator after 10 sec delay to update all the properties in shadow.

```dart

  // Refresh All Data for coordinator after 10 sec delay
  Future<void> refershCoordinator(
      CoordinatorModel coordinatorModel, SliderListItemDeviceModel item) async {
    await Future.delayed(Duration(seconds: 10));
    Map<String, dynamic> publishRefreshCommand = coordinatorModel
            ?.reported?.model?.properties?.ep0SZdo
            ?.publishRefreshD() ??
        {};
    if (publishRefreshCommand.isNotEmpty) {
      await coordinatorBloc.publish(
        publishRefreshCommand,
        item.coordinatorDevice!.deviceCode,
      );
    }
  }

```
---

## 🖥 Gateway Update UI  
**File:** `gateway_update.dart`  
This file contains logic to render different UI states during the firmware update process.

```dart
Widget _getScreenForStatus(int status) {
  if (status == -1) return _buildFailedScreen();
  else if (status == 0) return _buildCheckingScreen();
  else if (status == 1) return _buildDownloadingScreen(currentStep: 1, currentText: "Downloading \nUpdate");
  else if (status == 2) return _buildDownloadingScreen(currentStep: 2, currentText: "Installing \nUpdate");
  else if (status == 3) return _buildDownloadingScreen(currentStep: 3, currentText: "Rebooting \nGateway");
  else if (status == 6) return _buildDownloadingScreen(currentStep: 4, currentText: "Validating \nUpdate");
  else if (status == 4) return _buildDownloadingCompleteScreen();
  else if (status == 5) return _buildSuccessScreen();
  else return _buildCheckingScreen();
}
```

---

## 🔄 Connection State Handling  
During gateway firmware updates, the device reboots, which can lead to false-negative connection states. To address this:

- If the connection is lost, the system **rechecks the connection after 30 seconds**.
- If still unsuccessful, it triggers the **Gateway Update Failed** UI.

```dart

  Future<bool> _checkConnectionWithDelay() async {
    bool isConnected = widget.item.isConnected(gatewayUpdate: true) ?? false;
    if (!isConnected) {
      await Future.delayed(const Duration(seconds: 30));
      isConnected = widget.item.isConnected(gatewayUpdate: true) ?? false;
    }
    return isConnected;
  }

```
---

### 🧪 Last Pull Request for SAUAPP-119  
**Title:** Timer Connection Negative Case Handling  
**PR Link:** [PR #832](https://bitbucket.org/%7B0741cce4-c632-431c-a42b-bf4ce33d34b2%7D/%7B55c24d90-7ff8-4a4e-990b-abc09bd96a14%7D/pull-requests/832)

---

### 🔥 Hotfix: OTA UX Improvement  
**Ticket:** [SAUAPP-864](https://salusinc.atlassian.net/browse/SAUAPP-864?focusedCommentId=143761)

### 🎯 Enhancement  
This update introduces logic to **conditionally show the OTA update button in LWM mode** if the **gateway firmware version is below `021770250303`**.

**Hotfix Branch Commit:**  
[Commit #27f4aab](https://bitbucket.org/ct-bitbucket/uleeco-smart-premium-app/commits/27f4aab6323e01c7d91a20d0d9bdff2fd5ca7def)

**Additional Commit:**  
[Commit #2d9a815](https://bitbucket.org/ct-bitbucket/%7B55c24d90-7ff8-4a4e-990b-abc09bd96a14%7D/commits/2d9a8151321bd5b735712be2466ee2daeb2ef032)
