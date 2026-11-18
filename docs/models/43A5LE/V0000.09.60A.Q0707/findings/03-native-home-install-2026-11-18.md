# 2026-11-18 — Clean Home native component installed

Owner-authorized install on the paired live unit. The live fingerprint is
43A5LE / internal 43A60LEVS_0002, MTK9603, V0000.09.60A.Q0707; it does not
match the owner's earlier reported 43A6N/V0002 identity.

## Candidate and preflight

The target is `tv.vidaa.app.operationui`, active CHB bundle
`/APPS/pkgs/tv.vidaa.app.operationui/CHB/apps/launcher/js/app.js`. The stock
bundle was 323,476 bytes, MD5
`45626039103fb60060618545dba6f0f5`, SHA-256
`5c9e0aeaa2eb5653cc8fda0fab7690ed0bacefd2f3801c8f61be9e99c41720d3`.
The candidate came from Clean Home upstream commit
`7d79be5533d0c9ba1cc12b0d15e0d7622e2f7c52`, ported privately against the
CHB native registry and route. It uses app-list-v1 and the TV's native
app/input/settings services. Candidate bundle SHA-256:
`674d75505609c4e666180be8a6a882d56384e747cc89cda3cf5284c2c55c023e`;
MD5 `b82ea1ec601a68c18c4b027a60b668e6`.

Before writing, the owner-configured Pi-hole freeze was confirmed from the TV:
both update domains returned `0.0.0.0` and `::`, with normal Google DNS/TLS.
A stock-only SystemUI restart gate passed: PID 952 was replaced by PID 12254.

## Install and observed state

The candidate replaced the active CHB `app.js`; a post-install root readback
reported the exact candidate MD5. The original is retained on TV at
`/var/local/alvidaa/nativehome43a5le/original-app.js` with stock MD5
`45626039103fb60060618545dba6f0f5`. Existing cobalt compiled/uncompiled
caches were moved under `/var/local/alvidaa/nativehome43a5le/compiled_js` and
`uncompiled_js` before fresh cache directories were created. Atlas assets were
installed under `/var/local/hisenseUI/static/vidaa-clean-home/` and their TV
MD5s matched the local build. SystemUI restarted during package postinst.

Temporary IPKs, FaaS modules, and package registry entries were cleaned up;
the temporary fetcher was stopped. No shell, startup hook, OTA rule, protected
partition, `/OAD` staging, or persistent root service was changed. The original
bundle copy remains as a safety backup, but no automated rollback path was
installed.

A subsequent capture still showed the Browser page. The owner later confirmed
that the physical HOME key opens the installed launcher, and that both standby/
wake and a cold boot return to it successfully. App launching, input switching,
and settings actions remain unconfirmed. No claims are made for 43A6N or other
firmware.
