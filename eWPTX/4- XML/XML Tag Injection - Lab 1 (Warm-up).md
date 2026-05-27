2026-02-04 17:55

Status:

Tags: [[eWPTX]] [[XML]]
###### Prerequisites: [[XML Tag Injection - Overview]]
# XML Tag Injection - Lab 1 (Warm-up)

Context: training/lab environment.

Goal: become a “leet” member when default users are “looser”.

Notes captured from lab write-up:

- the XML structure is visible in a client-side function (e.g., `WSregister__old`)
- testing showed `username` was the injectable parameter
- crafted input allowed inserting a new `<user>` element with a different role

(Keep this note as a lab reference; expand with screenshots/requests only for your authorized lab.)
