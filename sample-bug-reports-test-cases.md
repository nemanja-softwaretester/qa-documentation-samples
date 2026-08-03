# Sample Bug Reports & Test Cases

Real bug reports and test cases created based on my experience as a software tester, documenting the testing of a
fictional "industrial production-management system" with no real company name, product
name, IPs, file paths, internal ticket IDs, or proprietary feature names.

---

## Bug Report 1

**Bug name:** Health Check endpoint returns "Status: Connected" even though the PLC connection is 
interrupted

**Date submitted:** 14.05.2026
**Priority:** High
**Severity:** High —The health check incorrectly reports the PLC as 
connected, potentially masking a real production outage.
**Environment:** Staging, Google Chrome 126, Windows 11; Backend Version: 1.95, Frontend Version: 2.03 
**Tags:** Frontend, Backend

**Preconditions:**
1. Check out the local BE branch: develop
2. Check out the local FE branch: fix/plc-status-toaster
3. Log in and enable PLC connection by navigating to Preferences / Start PLC.

**Steps to reproduce:**
1. Start the application and navigate to the Home page.
2. Open the browser's Developer Tools and select the Network tab.
3. Verify that the Health Check request is sent every second.
4. While the application is running, open the database client.
5. Open the SystemPreferences table in the local database and locate the record with PlcId = 3.
6. Change the IpAddress value from 192.168.1.20 to 192.168.1.21, then save the changes.
7. Return to the application and observe the Health Check requests in the Network tab.

**Observed results:** The Health Check endpoint continuously returns 200 OK with 
Status: Connected, even though communication with the PLC has been interrupted 
due to the invalid IP address.

**Expected results:** The Health Check endpoint returns 400 Bad Request with Status: Disconnected 
and the appropriate error message: Disconnected: Invalid IP Address.
A notification toaster is displayed to inform the user that the PLC connection status has changed.

**Screenshots:** Backend log captured during testing:
```
System.InvalidOperationException: PLC is not connected.
   at DeviceSdk.Rx.DeviceConnector.EnsureConnectionValid()
   at DeviceSdk.Rx.DeviceConnector.WriteBytes(Operand operand, UInt16 startByteAddress,
      Byte[] data, UInt16 dbNo, UInt16 bytesToWrite, CancellationToken token)
   at DeviceSdk.Rx.DevicePlc.SetValue<TValue>(String variableName, TValue value,
      CancellationToken token)
   at Acme.App.DeviceDriver.Communication.DeviceCommunication.WriteToPlcAsync<T>(Int32 plcId,
      String variableName, T value) in
      C:\Users\DEV1\App_BE\src\Acme.App.DeviceDriver\Communication\DeviceCommunication.cs:line 313

```
**Comment:** Added a log entry under the Screenshots section.

**Suspected Cause:** The Backend correctly detects the lost PLC connection internally 
(see attached exception), but the Health Check endpoint continues reporting the 
previous connection state instead of propagating the updated status to the Frontend.

---

## Bug Report 2

**Bug name:** On some Liquid Records, Arrow-down button remains enabled in Restricted Mode and crashes the configuration editor
**Date submitted:** 15.10.2025
**Priority:** High
**Severity:** Medium — The application crashes during a common user interaction, blocking further work within the configuration editor
**Environment:** Staging, Mozilla Firefox 127, Windows 10; Backend Version: 1.67, Frontend Version: 1.70 
**Tags:** Frontend

**Preconditions:**
1. Log in successfully.
2. Navigate to the Liquid Records module.
3. Ensure Restricted Mode is enabled for the selected record.

**Steps to reproduce:**
1. Open the configuration editor by right-clicking a record and selecting Edit.
2. Click the Arrow Down button.

**Observed results:** The Arrow Down button remains enabled in Restricted Mode.
Clicking the button throws an unhandled exception, causing the configuration editor to crash.

**Expected results:** The Arrow Down button is disabled while Restricted Mode is enabled and cannot be clicked.

**Screenshots:** Video provided: BR-144-arrowdown-issue. Browser console:
```
TypeError: Cannot read properties of null (reading 'stepIndex')
    at handleArrowDown (StepEditor.jsx:214:18)
    at onClick (StepEditor.jsx:198:9)
    at executeDispatch (react-dom.development.js:9041:9)
    at runWithFiberInDEV (react-dom.development.js:544:16)
```
**Comment:** The issue is not consistently reproducible. Current investigation
suggests it depends on whether the parent record is opened from the record list 
or loaded directly by ID.

**Suspected Cause:** The Arrow Down button appears to remain enabled after the selected record is initialized. 
The click handler attempts to access `stepIndex` before the selected step has been initialized, 
resulting in a null reference exception.

---

## Bug Report 3

**Bug name:** For a One-Direction Cycle (Procedure #13), Parameter 57 (timed-controller flag)
sends integer value as 0 instead of null

**Date submitted:** 10.01.2025
**Priority:** Low
**Severity:** Low — The issue affects a single parameter but prevents the record from being saved successfully.
**Environment:** Production, Google Chrome 126, Windows 11; Backend Version: 1.94 , Frontend Version: 1.93 
**Tags:** Frontend

**Preconditions:**
1. Log in successfully.
2. Ensure at least one Liquid Record exists.
3. Open the record in Edit mode.
4. Open the Configure Parameters window.

**Steps to reproduce:**
1. Navigate to your Liquid Record / Automatic Parameters
2. Clear the "Timed Controller" checkbox
3. Save the changes

**Observed results:** The request payload sends 0 instead of null for Parameter 57, 
causing the save operation to fail.

Captured request payload:
```json
POST /api/liquid-records/{recordId}/parameters
{
  "functionType": "Procedure13",
  "parameter57": {
    "name": "timedController",
    "value": 0
  }
}
```
Expected `"value": null` when the checkbox is unchecked — The Frontend 
submits the default numeric value instead of clearing the parameter.

**Expected results:** Should save successfully, with `"value": null` sent when
"Timed Controller" is unchecked.

**Screenshots:** Network request captured using the browser's Network tab.

**Comment:** I verified the issue using the browser's Network tab, Postman, and Swagger.

**Suspected Cause:** The Frontend serializes the unchecked field using its 
default integer value (0) instead of null. 

---

## Bug Report 4

**Bug name:** Velocity and Cycle Time input fields are hidden instead of 
displayed as disabled in the configuration editor

**Date submitted:** 15.10.2025
**Priority:** Medium
**Severity:** Medium — The fields disappear instead of remaining visible 
in a disabled state, reducing discoverability and creating an inconsistent user experience.
**Environment:** Test, Google Chrome 126, Windows 11; Backend Version: 1.85, Frontend Version: 1.99 
**Tags:** Frontend

**Preconditions:**
1. Login successfully
2. Ensure at least one Solid Record exists
3. Ensure Velocity and Cycle Time are disabled in Preferences for the selected record

**Steps to reproduce:**
1. Open the configuration editor for the selected Solid Record.
2. Observe the Velocity and Cycle Time input fields.

**Observed results:** The Velocity and Cycle Time input fields are not displayed on the UI.

**Expected results:** The Velocity and Cycle Time input fields remain visible in a disabled
state, consistent with the intended UI behavior.

**Screenshots:** Three screenshots attached.

**Comment:** The fields should always be visible, even when disabled.

**Suspected Cause:** The Frontend appears to conditionally hide these fields instead of 
rendering them in a disabled state. The rendering logic is inconsistent with the 
intended behavior for these parameters.

---

## Bug Report 5

**Bug name:** Two-Direction Cycle Preset #14 (presetId:214) incorrectly populates 
Cycle Positive and Cycle Negative values

**Date submitted:** 15.10.2025
**Priority:** High
**Severity:** High — Incorrect Preset values are automatically applied without user 
intervention, potentially resulting in invalid production parameters.
**Environment:** Development, Google Chrome 126, Windows 11; Backend Version: 1.95, Frontend Version: 2.02 
**Tags:** Frontend

**Preconditions:**
1. Login successfully
2. Ensure default Presets are enabled in Preferences
3. Ensure at least one Liquid Record is configured as a Two-Direction Cycle

**Steps to reproduce:**
1. Open the Two-Direction Cycle parameter window for a Liquid Record
2. Select Preset #14 – Long Cycle from the Presets dropdown
3. Compare the Cycle Positive and Cycle Negative values displayed in the UI with the values stored in the database

**Observed results:** The Cycle Positive and Cycle Negative fields are automatically populated with 
values that do not correspond to the selected Preset

**Expected results:** The selected Preset populates the Cycle Positive and Cycle Negative 
fields with the correct values stored for that Preset.

**Screenshots:** Two videos attached - labeled BR-123-preset-issue-db and BR-123-preset-issue-ui.

**Comment:** The Preset values are correctly stored in and returned from the database 
(verified using GET /api/liquid-records/{recordId}/presets). The issue occurs only in the Frontend.

**Suspected Cause:** The Frontend appears to populate the fields using incorrect property mappings 
after the Preset is selected, even though the Backend returns the correct Preset values.

---

## Bug Report 6

**Bug name:** Preset "LowLevel" displays the backend value instead of the user-friendly label "Lower Level"

**Date submitted:** 15.10.2025
**Priority:** Low
**Severity:** Low — Cosmetic labeling issue that affects usability but does not impact functionality.
**Environment:** Staging, Google Chrome 126, Windows 11; Backend Version: 0.35, Frontend Version: 0.36 
**Tags:** Frontend

**Preconditions:**
1. Log in successfully
2. Ensure Default Presets are enabled in Preferences
3. Ensure at least one record exists

**Steps to reproduce:**
1. Open any record
2. Open the Preset dropdown
3. Observe the Preset labeled "LowLevel"

**Observed results:** The Preset displays "LowLevel", exposing the backend value instead of the 
user-friendly display name.
**Expected results:** The Preset displays "Lower Level", using the user-friendly label intended 
for display in the UI.

**Screenshots:** Image attached. 

**Comment:** This issue is cosmetic only and does not affect functionality.

**Suspected Cause:** The Frontend appears to display the raw enum/string value received from the Backend instead of resolving it through the application's display-name mapping or localization resource.

---

## Bug Report 7

**Bug name:** Parameter table does not update correctly with Cycle Start, Cycle Pause, and Cycle Resume values

**Date submitted:** 15.10.2025
**Priority:** Medium
**Severity:** High — The Parameter table displays outdated values after configuration changes, 
potentially misleading the user and resulting in incorrect production settings.
**Environment:** Staging, Mozilla Firefox 127, Windows 10; Backend Version: 2.05 , Frontend Version: 2.12 
**Tags:** Backend

**Preconditions:**
1. Log in successfully
2. Ensure at least one Solid Record exists with the Custom Cycle Values option enabled
3. Open the record in Edit mode

**Steps to reproduce:**
1. Change the Cycle Start, Cycle Pause, or Cycle Resume values
2. Observe the Parameter table

**Observed results:** The Parameter table continues displaying the previous values after 
the underlying parameters have been modified.

**Expected results:**The Parameter table immediately reflects the updated values after 
the parameters are changed.

**Screenshots:** Screenshots attached.

**Comment:** The values are updated correctly in the database (verified using the database client).
The issue is limited to the Frontend display.

**Suspected Cause:** The Frontend appears to update the parameter values successfully but does not refresh 
or re-bind the Parameter table after the changes are applied, resulting in stale data being displayed.

---

## Bug Report 8

**Bug name:** Partially overlapping Procedures on the same station are not blocked

**Date submitted:** —
**Priority:** High
**Severity:** High — The application allows two Procedures to be scheduled simultaneously on 
the same station, creating a scheduling conflict that can directly impact production.
**Environment:** Local branches (BE: fix/scheduling-conflict, FE: fix/procedure-scheduling), Microsoft Edge 129, Windows 11;
**Tags:** Backend, Frontend

**Preconditions:**
1. Log in successfully.
2. Ensure an existing Procedure is scheduled on Station A from 09:00–11:00.

**Steps to reproduce:**
1. Create a new Procedure on Station A with a scheduled time of 10:30–12:00, 
partially overlapping the existing Procedure.
2. Save the Procedure.

**Observed results:** The Procedure is saved successfully without displaying a scheduling conflict.

**Expected results:** The application prevents the Procedure from being saved and displays a scheduling
conflict message, consistent with the validation already implemented for fully overlapping Procedures.

**Screenshots:** Video attached, labeled BR-484-Procedure-scheduling

**Comment:** Reproducibility is not 100%. In some cases the application correctly displays a 
conflict notification, while in others the Procedure is saved successfully. During testing, 
the Backend occasionally returned an incorrect time range for the second Procedure 
(2026-07-29 11:00:00.00 – 2026-07-29 13:00:00.00 instead of 2026-07-29 10:30:00.00 – 2026-07-29 12:00:00.00), 
allowing the Frontend to accept the conflicting schedule.

**Suspected Cause:** The Backend intermittently calculates or returns an incorrect scheduling time range for partially overlapping Procedures. The Frontend also incorrectly allows the Procedure to be dragged and dropped over an overlapping procedure.

---

## Bug Report 9

**Bug name:** Backend accepts out-of-range Parameter 13 values without server-side validation

**Date submitted:** —
**Priority:** Medium
**Severity:** Low — The issue is currently only reproducible through direct API requests, 
but it allows invalid data to bypass the Frontend validation rules.
**Environment:** Test Environment API (direct API requests, Frontend bypassed)
**Tags:** Backend

**Preconditions:**
1. Valid authentication token for a test account.
2. An existing Liquid Record with configurable parameters.

**Steps to reproduce:**
1. Send a PATCH /api/liquid-records/{recordId}/{parameterId} request with parameter13.value set to -5.

**Observed results:** The API returns 200 OK, and the value is successfully persisted as -5.
**Expected results:** The API returns 400 Bad Request with a validation error indicating that Parameter 13 
only accepts whole numbers in the range 1–100. The validation should be enforced on the Backend 
regardless of any Frontend validation.

**Screenshots:** Postman request and response attached.

**Comment:** The Frontend correctly validates this field and prevents invalid values from being entered.
The issue is only reproducible by bypassing the UI and sending requests directly to the API.

**Suspected Cause:** Validation for Parameter 13 appears to exist only in the Frontend. 
The Backend does not perform range validation before persisting the incoming value, allowing invalid data to be stored.

---

## Test Case 1 — Two-Direction Cycle Configuration Window

**Test case:** Two-Direction Cycle Configuration Window

**Test description:** Verify the functionality of the Two-Direction Cycle configuration window, 
including parameter visibility, input controls, Preset functionality, and parameter table updates.

**Preconditions:**
1. Log in successfully
2. Ensure at least one Liquid Record exists
3. Open the record in Edit mode
4. Select Two-Direction Cycle from the Cycles dropdown
5. Click Configure

**Test steps:**
1. Verify the parameter name is displayed correctly
2. Verify the Start Position is valid
3. Verify the Cycle Time input field
4. Verify the Preset dropdown
5. Verify the Start checkbox functionality
6. Verify the Description field is enabled and accepts input
7. Verify the Comment text box is disabled
8. Verify the Velocity dropdown is disabled when the option is disabled in Preferences
9. Verify the Customize Preset button is disabled when no Preset is selected
10. Verify that selecting a Preset correctly populates the Start, Pause, and Resume values
11. Verify the Secondary Function checkbox
12. Verify the Start input field
13. Verify the Pause input field
14. Verify the Resume input field
15. Verify the Parameter table updates immediately when values or selections change

**Test data:** Liquid Record configured as a Two-Direction Cycle

**Expected results:** All controls are displayed correctly and function as designed.

**Actual result:** Four defects identified.

**Status:** Failed

**Linked bugs:** BR-liquid-records-004, BR-liquid-records-005, BR-liquid-records-006, BR-liquid-records-007

---

## Test Case 2 — Edit Profile (Admin User) Mandatory Field Validation

**Test case:** Edit Profile (Admin User) Mandatory Field Validation

**Test description:** Verify mandatory field validation when editing an Admin User profile.

**Preconditions:**
1. Log in successfully
2. Navigate to Profile / Edit Profile

**Test steps:**
Note: Verify that mandatory validation is implemented for each field in both the UI and the Backend (via the API).
1. Verify the Name field is required
2. Verify the Email field is required when User Type is set to Admin
3. Verify the Phone field is optional
4. Verify the User Type field is required
5. Verify the Language field is required

**Test data:** Admin User account

**Expected results:** All mandatory validations are enforced correctly in both the UI and the Backend.

**Actual result:** All validations function as expected.

**Status:** Passed

---

## Test Case 3 — Configure Parameter Window Validation

**Test case:** Configure Parameter Window Validation

**Test description:** Verify input validation for configurable parameter fields.

**Preconditions:**
1. Log in successfully
2. Navigate to Liquid Records
3. Ensure at least one Liquid Record exists

**Test steps:**
1. Open the Configure Parameters window
2. Verify that the Cycle Time field accepts only whole numbers between 1 and 99
3. Verify that the Look Ahead field accepts decimal values with up to three decimal places
4. Verify that the Look Ahead field accepts only positive values (minimum accepted value: 0.001)
5. Verify that the Start, Pause, and Resume fields accept integer values (positive and negative)


**Test data:** Multiple Liquid Records covering all available parameter configurations

**Expected results:** All validation rules are enforced correctly.

**Actual result:**  All validation rules function as expected.

**Status:** Passed

---

## Test Case 4 — Procedure Scheduling Conflict Detection

**Test case:** Procedure Scheduling Conflict Detection

**Test description:** Verify that the application prevents scheduling conflicts between Procedures assigned to the same station.

**Preconditions:**
1. Log in successfully
2. Navigate to Procedures / Live Production
3. Ensure an existing Procedure is scheduled on Station A from 09:00–11:00

**Test steps:**
1. Attempt to schedule a Procedure on Station A from 09:30–10:30 (full overlap)
2. Attempt to schedule a Procedure on Station A from 10:30–12:00 (partial overlap)
3. Attempt to schedule a Procedure on Station A from 11:00–13:00 (adjacent, no overlap)
4. Attempt to schedule a Procedure on Station B from 09:30–10:30

**Test data:** Existing Procedure scheduled on Station A from 09:00–11:00.

**Expected results:**
- Steps 1 and 2 are blocked with a scheduling conflict message.
- Step 3 is allowed.
- Step 4 is allowed.

**Actual result:** Steps 1, 3, and 4 behaved as expected. Step 2 incorrectly allowed the Procedure to be saved.

**Status:** Failed

**Linked bugs:** BR-008 - Partially overlapping Procedures on the same station are not blocked.

---

## Test Case 5 — Live Production Drag-and-Drop Functionality

**Test case:** Live Production Drag-and-Drop Functionality

**Test description:** Verify drag-and-drop functionality on the Live Production page.
 
**Preconditions:**
1. Log in successfully
2. Navigate to Procedures / Live Production

**Test steps:**
1. Verify that Procedures can be dragged between compatible stations
2. Verify that drag-and-drop is blocked for incompatible station types
3. Verify that Non-Immediate Procedures can be repositioned along the timeline
4. Verify that Immediate Procedures are automatically scheduled at the first available execution time

**Test data:** Multiple Immediate and Non-Immediate Procedures

**Expected results:** All drag-and-drop functionality behaves according to the specification.

**Actual result:** All functionality works as expected.

**Status:** Passed

---

## Test Case 6 — API Validation: Reject Invalid Parameter Payload

**Test case:** API Validation: Reject Invalid Parameter Payload

**Test description:** Verify that the Backend validates and rejects invalid parameter update 
requests submitted directly to the API.

**Preconditions:**
1. Valid authentication token for a test account
2. Ensure an existing record with configurable parameters is available

**Test steps:**
1. Send a `PUT /api/liquid-records/{recordId}/parameters` request with parameter57.value set to "abc" (string)
2. Send the same request with parameter57.value set to -5
3. Send the same request with parameter57.value explicitly set to null
4. Send the same request while omitting parameter57 from the payload

**Test data / example request (step 1):**
```json
PUT /api/liquid-records/{recordId}/parameters
{
  "parameter57": { "name": "timed-controller", "value": "abc" }
}
```

**Expected results:**
- Step 1 returns 400 Bad Request with a validation error for parameter57.
- Step 2 returns 400 Bad Request with a range validation error.
- Step 3 returns 200 OK, and the value is persisted as null.
- Step 4 returns 200 OK, and the existing value remains unchanged.

**Actual result:** All four scenarios behave as expected.

**Status:** Passed

---

