# EFI publication placeholder

The actual EFI is **not included** in this documentation archive. The owner has supplied a full `config.plist` text and selected hardware / Probe facts, but has not supplied the ACPI binaries, kext binaries, UTBMap contents or complete EFI archive. See [Config audit](../Docs/Config-Audit.md).

Before adding EFI here:

- duplicate the working EFI to a separate public folder;
- regenerate / sanitize populated SMBIOS identifiers (`SystemSerialNumber`, `MLB`, `SystemUUID`, `ROM`);
- remove secrets, Wi-Fi credentials, personal logs and machine-specific identifiers;
- retain only the intended `UTBMap.kext`, not timestamped backup copies;
- verify each kext in `Kernel -> Add` and remove unused duplicates;
- include OpenCore version and a changelog;
- test that the sanitized `config.sample.plist` is structurally valid.

Do not upload FileVault recovery keys, APFS Preboot data, personal certificates, private keys or account credentials.
