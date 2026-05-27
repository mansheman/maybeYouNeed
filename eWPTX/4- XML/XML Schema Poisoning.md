2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prerequisites: [[XML DTD (Document Type Definition)]]
# XML Schema Poisoning

## Overview

Schema poisoning refers to manipulating schemas/validation assumptions so that malicious or unexpected XML passes validation.

Impact:

- validation bypass
- injection of unexpected data that affects app logic

Mitigation (high level):

- use strict, pinned schemas
- don’t fetch schemas from untrusted locations
- enforce validation and parser security limits
