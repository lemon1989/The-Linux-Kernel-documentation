Documentation for Kdump - The kexec-based Crash Dumping Solution

This document includes overview, setup and installation, and analysis information.
Overview

Kdump uses kexec to quickly boot to a dump-capture kernel whenever a dump of the system kernel’s memory needs to be taken (for example, when the system panics). The system kernel’s memory image is preserved across the reboot and is accessible to the dump-capture kernel.
