# TCAT DUT Preparation Instructions

This guide is a practical checklist for preparing an OpenThread-based device under test (DUT) for TCAT testing. It covers the device and lab-side preparation that can be performed before a test session; it does not replace the current Thread Group certification program requirements, test plan, or Authorized Test Lab (ATL) instructions.

> **Scope:** OpenThread's CLI TCAT example currently documents TCAT over BLE. Confirm that the test plan, DUT firmware, and test harness use the same TCAT transport and feature scope. Do not assume that this guide or a successful smoke test demonstrates certification compliance.

## 1. Confirm the test scope

Before preparing hardware, record and confirm:

- Product name, hardware revision, SoC/radio, and DUT serial number.
- OpenThread release/commit and platform SDK/toolchain versions used to build the image.
- Thread specification version and TCAT feature/test scope requested by the certification plan.
- Required TCAT transport and supported interfaces on the DUT. OpenThread's documented CLI example is BLE-only.
- Required test harness, topology, test cases, and DUT control method with the ATL or test owner.
- Whether the test session expects a factory-fresh device, a pre-provisioned device, or a specific operational Thread state.

Use the current Thread Group/ATL-provided test plan, harness release, topology files, and device-specific requirements as authoritative. Do not modify controlled test inputs unless the lab explicitly instructs you to.

## 2. Hardware and bench setup

Prepare the following, as applicable to the product and lab setup:

- DUT and any required carrier board, antenna, power supply, and programming/debug probe.
- A reliable USB/UART or other supported control connection, with the correct cable, adapter, and host drivers.
- A host computer with the required test harness and approved DUT-control software installed.
- BLE-capable hardware and host support if the selected TCAT test cases use BLE.
- Any required Thread reference devices or RF equipment specified by the test plan.

Place the DUT in the required physical orientation and antenna configuration. Keep the bench configuration repeatable; avoid unapproved changes to antennas, RF paths, shielding, or nearby radios. Record board connections and host port assignments so the setup can be reproduced.

## 3. Build and flash the DUT firmware

1. Start from the release, commit, and platform SDK required by the test plan. Preserve the source revision and build configuration used for the test image.
2. Enable the TCAT/BLE-Secure functionality using the configuration mechanism supported by the selected OpenThread platform and application. Feature options and prerequisites are platform-specific; verify them against that platform's build documentation rather than copying configuration values from another product.
3. Build the production-equivalent DUT image with the required test hooks and interfaces. Avoid development-only behavior that changes protocol operation unless the test plan explicitly requires it.
4. Save the build command, relevant configuration, compiler/SDK versions, image filename, and cryptographic hash in the test record.
5. Flash the image using the vendor-supported programming procedure. Confirm the flash operation succeeds, reset or power-cycle as required, and capture the boot/version output.

Do not include real production secrets, private keys, or sensitive customer data in shared logs or this document. Use test credentials and certificate material provisioned for the test environment.

## 4. Prepare device identity and credentials

Before connecting the DUT to the harness:

- Provision the test device certificate(s), trust material, and any other TCAT credentials required by the selected test cases.
- Confirm the certificate/identity selection expected by the test plan. OpenThread's CLI documentation provides `tcat certid` to display the selected certificate ID and `tcat certid <id>` to select an ID where supported.
- Set the TCAT Device ID and advertised IDs only when the test plan requires particular values. IDs are device- or ecosystem-specific; use assigned test values, not the sample values in CLI documentation.
- Record which test identity is installed and selected, without recording private key contents.
- If a test requires factory reset or credential cleanup between cases, follow the lab procedure and verify the resulting state before proceeding.

## 5. Start and smoke-test TCAT

Connect to the DUT's supported CLI/control interface. The exact connection command depends on the platform. First check whether the TCAT commands are present:

```text
> tcat help
```

The OpenThread TCAT CLI example documents `start`, `stop`, `active`, `standby`, `certid`, `devid`, and `advid` commands. If `tcat` or a required subcommand is unavailable, stop and check the firmware feature configuration and platform support.

Start the TCAT agent:

```text
> tcat start
Done
```

When applicable, verify the expected certificate selection and IDs using the documented read commands:

```text
> tcat certid
> tcat devid
> tcat advid
```

Then use the approved test client or harness to confirm that the DUT is discoverable/reachable over the transport required by the test plan and that the expected TCAT connection can be established. This is a setup smoke test only; do not substitute it for certification test cases or alter the required test sequence.

If the plan requires TCAT to remain available but inactive until a test step, use the documented `tcat standby` / `tcat active` behavior as directed by the test procedure. To end the session, use `tcat stop` if required; this stops the agent and advertisements. Confirm the DUT is in the expected state before handing it to the next test.

For exact CLI behavior and argument formats, refer to the [OpenThread TCAT CLI reference](src/cli/README_TCAT.md). The reference notes that its current example implements TCAT over BLE only.

## 6. Pre-test readiness checklist

- [ ] DUT model, hardware revision, serial number, and test owner are recorded.
- [ ] Required test plan, test harness version, topology, and TCAT scope are confirmed with the ATL/test owner.
- [ ] Firmware matches the requested source revision and feature configuration; image hash is recorded.
- [ ] DUT is flashed, boots reliably, and exposes the expected control interface.
- [ ] Required BLE/Thread radios, antennas, cables, power, and reference devices are connected as specified.
- [ ] Test certificates and identity are provisioned; selected certificate/ID has been checked.
- [ ] TCAT CLI commands are available and `tcat start` succeeds, if CLI control is part of the setup.
- [ ] The approved client/harness can perform the required pre-test connectivity check.
- [ ] Logs are enabled as permitted, timestamps are usable, and secrets are excluded.
- [ ] Device state and cleanup/reset procedure between tests are understood.

## 7. Troubleshooting notes

| Symptom | Checks |
| --- | --- |
| `tcat` command is unavailable | Confirm that the correct image is running and that TCAT support is enabled and included for this platform/application. Check the build configuration and firmware version. |
| `tcat start` fails or no advertisement is observed | Check platform BLE support, radio/antenna setup, power, host-side BLE adapter, and whether the TCAT agent is already in a different state. Verify against the platform and test-plan instructions. |
| Client discovers the DUT but cannot complete the expected test connection | Check the selected test identity/certificate, credential provisioning, test-client configuration, and transport/test scope. Do not expose private keys in logs. |
| DUT control connection is unstable | Check cable, host port/driver, baud or interface settings from the board documentation, and power stability. Capture the failure and exact DUT/build details. |

When troubleshooting, preserve the original test configuration and collect only the logs/artifacts permitted by the lab. If behavior differs from the test plan, consult the ATL/test owner before changing firmware, credentials, topology, or harness settings.

## References

- [OpenThread TCAT CLI reference](src/cli/README_TCAT.md)
- [OpenThread CLI reference](src/cli/README.md)
- [OpenThread certification automation setup](https://openthread.io/certification/automation-setup)
- [Thread Group certification information](https://www.threadgroup.org/Certification)

Thread Group certification requirements and lab procedures may change. Confirm the latest official materials and lab-specific instructions before each certification run.
