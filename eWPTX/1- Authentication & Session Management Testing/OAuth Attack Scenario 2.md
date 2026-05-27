2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[OAuth]]
###### Prerequisites: [[Common OAuth Attacks]]
# OAuth Attack Scenario 2

This scenario illustrates how OAuth-related weaknesses can be chained with other issues.

Example chain (high level):

- OAuth parameter reflected XSS
- weak `X-Frame-Options` configuration
- browser autocomplete not disabled

Result:

- attacker can potentially extract credentials or sensitive tokens.

Notes:

- treat OAuth endpoints like any other web surface: validate/sanitize and harden headers.
