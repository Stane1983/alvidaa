# 43A5LE (internal 43A60LEVS_0002) — V0000.09.60A.Q0707

- **Model:** `43A5LE` (`Hisense_GetSerialType()`); internal
  `Hisense_GetModelName()` = `43A60LEVS_0002`
- **Firmware:** `V0000.09.60A.Q0707` (build 2026-07-07; AirPlay `fv`
  `p20.09.6010.260707`)
- **SoC:** `MTK9603` (`Hisense_GetChipSetName()`)
- **Region:** `EU_A` (`Hisense_GetRegion()`), country `ITA`
  (`Hisense_GetCountryCode()`)
- **VIDAA OS:** `U09.60`
- **Browser:** Chromium 111, `armv7l`, UA
  `Mozilla/5.0 (X11; Linux armv7l) AppleWebKit/537.36 (KHTML, like Gecko)
  Chrome/111.0.0.0 SurfMaster/3.0.1 Safari/537.36`; 1920×1080 @ DPR 1.5
- **`transport_protocol`:** not measured — SSDP multicast did not reach this
  host on the test LAN; only TCP 36669 (MQTT) and 7000 (AirPlay) answered
- **Reported by:** owner (this session), 2026-11-06

**Status:** launcher page and server-side Browser route verified on the owner’s
unit. Native `Appinfo.json` self-install is **closed** by the per-app
permission gate. One-shot uid-0 execution is now verified through the FaaS
fetcher → superwatch → vendor IPK installer route. A native Home candidate is
installed; visual acceptance via the TV's HOME action is unconfirmed. No shell
or boot persistence was installed.

> This guide is for the exact model **and** firmware above. The MTK9603 /
> 09.60 family is shared with the
> [43Q60SEVS guide](../../43Q60SEVS/V0000.09.60W.Q0612/README.md), and the
> [55U67LEVS guide](../../55U67LEVS/V0000.09.60A.Q0707/README.md). Root is now
> verified independently on this unit; other units and firmware remain
> separate measurements.

## Pairing — working

The 2024 client certificate is required. It is inside the official
`com.universal.remote.multi` APK as `res/El.p12` (2565 bytes,
`CN=VidaaAppAndroidV01`, valid 2024–2034), keystore password
`186e990688070325a1c4b0ce275d2388`:

```sh
unzip -p com.universal.remote.multi_1.09.05.001.0.apk res/El.p12 > El.p12
openssl pkcs12 -legacy -in El.p12 -passin pass:186e990688070325a1c4b0ce275d2388 \
  -clcerts -nokeys -out server/keys/vidaa_cert.pem
openssl pkcs12 -legacy -in El.p12 -passin pass:186e990688070325a1c4b0ce275d2388 \
  -nocerts -nodes -out server/keys/vidaa_key.pem
```

Measured on this unit:

| `TV_MAC` form | Profile | MQTT result |
| --- | --- | --- |
| `XX:XX:XX:XX:XX:XX` (uppercase, colons) | `modern` | **rc 0** |
| 12 lowercase hex chars, no colons | `modern` | rc 135 (not authorized) |
| `XX:XX:XX:XX:XX:XX` | `middle` / `standard` | rc 135 |

The PIN dialog appears once `vidaa_app_connect` is published. Submit the PIN
once; re-triggering rotates it. On success the TV returns
`authenticationcode {"result": 1}` and issues `tokenissuance` with a 2-day
access token and 30-day refresh token. Reconnecting with the issued token
enables two-way actions: `state`, `gettvstate`, `applist`, `sourcelist`, and
`launchapp` all answered on this build.

**Certificate sourcing trap:** the older `1.08.02.007.0` APK ships
`res/raw/remoteclientmobile.p12` / `res/raw/remoteclientoem.p12` instead;
the documented keystore passphrase did not open either (MAC verify error).
Use a build that ships `res/El.p12` (confirmed with `1.09.05.001.0`).

## Launcher — working, but not installable

The [server](../../../launcher/DEPLOY.md) serves the page, `/ws`, and the icon
proxy. On the TV the page renders, connects, and receives live state.

### The install gate

`HiUtils_createRequest` is present; `Hisense_installApp` and
`WebSDK_createFileRequest` are **absent**. Writing the TV's app registry is
rejected:

```
fileWrite websdk/Appinfo.json mode 6
-> {"ret":false, "code":503,
    "msg":"client request permission check error, please check appconfig",
    "sdkVersion":"1.5.0"}
```

`fileRead` on the same path works and returned the 4314-byte registry with
four entries, unchanged after the failed write. This is the same per-app
identity gate reported on [43Q60SEVS / 09.60W](../../43Q60SEVS/V0000.09.60W.Q0612/README.md)
and on the 09.60V 55A60LEVS report. **Do not expect INSTALL AS APP to
persist a tile on this firmware without root.** The launcher's file-bridge
fallback and the server-side registration keep working; the tile itself is
never created.

### The launch shape that works

A custom `appId` (`vidaa-recon`, `vidaa-home`) is silently ignored on this
build. The built-in Browser entry launches and honours an overridden URL:

```json
{"appId":"16","name":"Browser","url":"http://<server>:8770/",
 "urlType":"1","storeType":100}
```

Published to
`/remoteapp/tv/ui_service/<client_id>/actions/launchapp`.

The home watcher then works as in Lane 1: when VIDAA reports
`statetype == remote_launcher`, the server relaunches the Browser at the
launcher URL. The ~1–2 s stock-home flash is inherent to this lane. The
watcher needs the server to stay up and the TV to reach it — on this host
`firewalld` blocked TCP 8770 until it was opened explicitly.

On the live unit in this session, `/api/open` was adjusted to use the
registered Browser entry (`appId` `16`) and override its URL; the TV fetched
the served page successfully. This proves page delivery, not a native Home
replacement. The native Home install followed after capturing the stock bundle
and verifying the owner's Pi-hole OTA freeze; see finding 03.

### Native Home target fingerprint (read-only)

The active operationui package is `9.61.00-a20260727u`, branch
`VIDAA_UI_U961_IPK_MP_20260716`, with the active bundle under the `CHB`
channel at `apps/launcher/js/app.js`. Browser-context reads of that bundle
return `EACCES`. A temporary one-shot root package copied it to a readable
path, computed its hashes, then removed both the copy and package. The bundle
was 323,476 bytes, MD5
`45626039103fb60060618545dba6f0f5`, SHA-256
`5c9e0aeaa2eb5653cc8fda0fab7690ed0bacefd2f3801c8f61be9e99c41720d3` — the
same bytes as the existing 65A85Q profile’s stock bundle. The bundle exposes a
`Launcher` module with a `route` property and references `/Launcher`. This is
strong porting evidence. The live process is `SystemUI` (UID/GID 311); the
bundle is owner `305:909`, mode `0640`; the cobalt JS cache is under
`/var/local/hisenseUI/cobalt_home/.cache/cobalt`, owner `311:311`. These facts
match the existing 65A85Q switch layout. The stock-only restart dry gate
passed (SystemUI PID 952 → 12254). The Clean Home component is installed and
the live bundle hash matches the candidate. The stock bundle copy remains on
the TV; no automated rollback path was added. Captured screen remained on
 Browser initially; the owner subsequently confirmed physical HOME, standby/
 wake, and cold boot all return to the installed launcher. App launching,
 inputs, and settings remain unconfirmed. See [finding 03](findings/03-native-home-install-2026-11-18.md).

The initial live check found both update hosts reachable, despite the signed
`/OAD/AppUpgradeSwitch.json` reporting `upgradeFlag=0` and
`/var/local/nightupdate.ini` showing `dcm_update=1`. The owner then blocked the
update hostnames in Pi-hole. A TV-side recheck returned `0.0.0.0` and `::` for
both hosts, while a Google control lookup and TLS connection succeeded. This
confirms the DNS freeze; no port block was used. The subsequent native Home
install is in finding 03. Full preflight evidence: [findings/02-ota-and-native-preflight-2026-11-18.md](findings/02-ota-and-native-preflight-2026-11-18.md).

## One-shot root — working, no persistence

On 2026-11-18 the owner’s unit exposed the FaaS worker as uid/gid 314 with
supplementary group 616, and `/var/run/superwatch_socket` accepted the
length-prefixed command 85. A minimal `mtk9603` IPK with `AppId`/`AppName` and
a non-empty data archive installed through command 82. Its `postinst` read
`/proc/self/status` and wrote only the UID, GID and capability lines to a
temporary marker. The marker read back:

```text
Uid: 0 0 0 0
Gid: 0 0 0 0
CapEff: 000001ffffffffff
```

The package was immediately removed with command 83. The post-removal checks
confirmed the package registry entry, proof marker, staged IPK and temporary
FaaS module were absent. No shell, startup hook, OTA rule, or launcher file was
installed. Full session notes: [findings/01-root-2026-11-18.md](findings/01-root-2026-11-18.md).

## Reversibility

The native Home candidate is installed with the stock bundle preserved at
`/var/local/alvidaa/nativehome43a5le/original-app.js`. Temporary installer
packages and FaaS modules were removed. No boot hook or shell remains. Local
state is the `/api/open` route change and ignored pairing material under
`server/keys/`.

Teardown:

1. Stop the server (and any second instance — one MQTT session per identity).
2. `sudo firewall-cmd --remove-port=8770/tcp` if the port was opened only for
   this test.
3. `rm -rf server/keys server/venv` (keys and tokens are device credentials).
4. Press **HOME** on the TV to leave the Browser page.

The paired credential is ephemeral (2-day access / 30-day refresh). If the
TV exposes a paired-mobile-device list, remove the entry there for a
zero-trace cleanup.

## Not tested

- `file_bridge_traversal` (`../` beyond the mode base)
- `euladpp_untar` and `devtools_forge`
- app launch, input switching, and settings from the native launcher
- boot-persistent root or a root shell

Field definitions: [TESTED-MODELS.md](../../../research/TESTED-MODELS.md).
Inventory row: [`V0000.09.60A.Q0707`](../../../research/inventory.json).
