# 2026-11-18 — OTA reachability and native Home preflight

Read-only verification on the owner’s paired unit after the owner said the OTA
freeze was applied. No launcher code or protected file was changed.

## Initial OTA check and follow-up DNS freeze

The active domain map names `ota-tv.vidaahub.com` and
`upgrade-plc-tv-eu.vidaahub.com`. From the TV:

- Both names resolved to `131.189.69.51`.
- TCP/443 connected and TLS 1.3 handshakes completed for both names. A
  Google TLS connection also succeeded as a control.
- `iptables -S OUTPUT` showed only `-P OUTPUT ACCEPT`, with no endpoint block.
- The signed `/OAD/AppUpgradeSwitch.json` reported `upgradeFlag=0` for HRV.
- `/var/local/nightupdate.ini` reported `dcm_update=1`.

At the initial check, the application switch was off but the network endpoint
freeze was not effective. The owner then added DNS blocks in Pi-hole, avoiding
port blocks that could affect unrelated applications. A repeat lookup from the
TV confirmed the freeze:

- `ota-tv.vidaahub.com` returned `0.0.0.0` and `::`.
- `upgrade-plc-tv-eu.vidaahub.com` returned `0.0.0.0` and `::`.
- The update-host TLS checks were skipped because DNS returned sinkhole
  addresses. The `www.google.com` control resolved normally and completed a
  TLS 1.3 connection.

This confirms the owner-configured DNS freeze while preserving normal DNS/TLS
for the control. No router port block was added by this session. The private
`firmware/ENDPOINTS.md` inventory was not present in this workspace.

## Native Home target facts

- Package: `tv.vidaa.app.operationui`, version `9.61.00-a20260727u`, branch
  `VIDAA_UI_U961_IPK_MP_20260716`.
- Active channel and bundle: `CHB`,
  `/APPS/pkgs/tv.vidaa.app.operationui/CHB/apps/launcher/js/app.js`.
- Stock bundle: 323,476 bytes; MD5
  `45626039103fb60060618545dba6f0f5`; SHA-256
  `5c9e0aeaa2eb5653cc8fda0fab7690ed0bacefd2f3801c8f61be9e99c41720d3`.
  This is byte-identical to the 65A85Q stock bundle.
- Bundle ownership/mode: `305:909`, `0640`.
- UI process: `SystemUI`, UID/GID 311.
- Cobalt cache: `/var/local/hisenseUI/cobalt_home/.cache/cobalt`, owner
  `311:311`; `compiled_js` and `uncompiled_js` are present.

These facts support a model-specific port of the existing CHB/SystemUI switch
layout. The stock-only dry restart subsequently passed and the Clean Home
candidate was installed; see finding 03. Screen-level acceptance remains
unconfirmed.

## Temporary probe cleanup

The read-only preflight package was uninstalled (command 83, rc 0); the report,
staged package, and FaaS module were confirmed absent. The temporary fetcher
service was explicitly stopped and replied `stopped`. The local VIDAA bridge was
then stopped. The subsequent DNS verification page and module were local
temporary files and were removed after recording the result. No OTA firewall
rule was added by this session.
