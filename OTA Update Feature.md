OTA Update Feature Overview

---

## 🔧 Feature: OTA Update  
**Ticket:** [SAUAPP-119](https://salusinc.atlassian.net/browse/SAUAPP-119)

This feature enables Over-the-Air (OTA) firmware updates for both **Coordinator** and **Gateway** devices. It includes automatic channel validation and adjustment for the coordinator if the current channel is not among the recommended values (15, 20, or 25).

---

## 🚀 Update Process Flow

1. **Update Coordinator Firmware**  
2. **Update Gateway Firmware**  
3. **Change Coordinator Channel** (if necessary)  
4. **Refresh Coordinator Data** – Initiated 10 seconds after firmware updates complete.

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

---

## 🧪 Last Pull Request for SAUAPP-119  
**Title:** Timer Connection Negative Case Handling  
**PR Link:** [PR #832](https://bitbucket.org/%7B0741cce4-c632-431c-a42b-bf4ce33d34b2%7D/%7B55c24d90-7ff8-4a4e-990b-abc09bd96a14%7D/pull-requests/832)

---

## 🔥 Hotfix: OTA UX Improvement  
**Ticket:** [SAUAPP-864](https://salusinc.atlassian.net/browse/SAUAPP-864?focusedCommentId=143761)

### 🎯 Enhancement  
This update introduces logic to **conditionally show the OTA update button in LWM mode** if the **gateway firmware version is below `021770250303`**.

**Hotfix Branch Commit:**  
[Commit #27f4aab](https://bitbucket.org/ct-bitbucket/uleeco-smart-premium-app/commits/27f4aab6323e01c7d91a20d0d9bdff2fd5ca7def)

**Additional Commit:**  
[Commit #2d9a815](https://bitbucket.org/ct-bitbucket/%7B55c24d90-7ff8-4a4e-990b-abc09bd96a14%7D/commits/2d9a8151321bd5b735712be2466ee2daeb2ef032)
