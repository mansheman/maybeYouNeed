2026-02-04 17:44

Status:

Tags:[[eWPTX]] [[Authentication]] [[OAuth]]
###### Prasyarat: [[Common OAuth Attacks]]
# OAuth Attack Scenario 2

Skenario ini menggambarkan bagaimana kelemahan OAuth bisa di-chain dengan isu lain sampai dampaknya terasa nyata.

Contoh rantai (high level):

- OAuth parameter reflected XSS
- weak `X-Frame-Options` configuration
- browser autocomplete not disabled

Hasil:

- attacker can potentially extract credentials or sensitive tokens.

Catatan:

- perlakukan endpoint OAuth seperti permukaan web lain: validasi/sanitasi input dan harden security headers.
