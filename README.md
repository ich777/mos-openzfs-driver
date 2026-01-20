# MOS OpenZFS Drivers

mos-openzfs-drivers provides a **MOS plugin** for managing the OpenZFS driver.

---

## Overview

This repository contains the **MOS plugin implementation**, optional helper
functions required to integrate Coral driver management into MOS.

The plugin enables MOS to download and install OpenZFS driver on demand directly
onto the system.

Driver Source

- OpenZFS: https://github.com/openzfs/zfs


No driver binaries are included in this repository.

---

## Build & Automation

This repository includes a **GitHub Actions workflow** used to build and package
the plugin for MOS.

The build process is fully automated and produces artifacts that can be consumed
by the MOS Hub or MOS release tooling.

---

## Licensing

The contents of this repository (plugin code, scripts, and configuration)
are licensed under **GPL-3.0**.

Downloaded OpenZFS drivers remain licensed under their respective upstream licenses
and are not part of this repository.
