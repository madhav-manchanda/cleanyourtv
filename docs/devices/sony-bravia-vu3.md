# Sony BRAVIA VU3 / KD-55X75K — build 265-60835

**Status: device report and firmware-variant notes.** This document records the Clean Your TV app
0.3.3 report supplied for a Sony BRAVIA VU3 on 2026-10-31. It is not a separate tested debloat list.

## Device identity

| Property | Reported value |
|---|---|
| Retail model reported by app | BRAVIA VU3 |
| Android product / device | `BRAVIA_VU3_4K` |
| Corresponding retail model in the upstream device profile | Sony KD-55X75K |
| Manufacturer | Sony |
| Android | 14 (SDK 34) |
| Build | `Sony/BRAVIA_VU3/BRAVIA_VU3_4K:14/UKR1.240726.001/265-60835:user/release-keys` |
| Platform | Realtek rtd2885m family |
| Original Home | Google TV |
| Google Play services | Present |
| Report date | 2026-10-31 |

The fork's source branch did not yet contain the exact `BRAVIA_VU3_4K` example. The original
upstream project has a Sony BRAVIA VU3 / KD-55X75K profile at
[`examples/sony-bravia-vu3/`](../../examples/sony-bravia-vu3/README.md). Its tested list records
build `UKR1.240726.001/386-60842`; this report records `UKR1.240726.001/265-60835`.
The product identifier and Android release match, but the build suffixes differ. The upstream list
is therefore a close model reference, **not a verified package list for build 265-60835**.

## Package state in the supplied report

The app reported 150 system apps:

| State | Count |
|---|---:|
| Switched off by Clean Your TV | 8 |
| Off, disabled (factory state) | 1 |
| On, enabled | 6 |
| On, default | 135 |

The eight packages reported as disabled by the app are:

```text
com.android.printspooler
com.google.android.feedback
com.google.android.play.games
com.sony.dtv.multiscreendemo
com.sony.dtv.promos
com.sony.dtv.smarthelp
com.sony.dtv.sonyselect
com.sony.dtv.sonytab
```

These overlap with packages in the upstream VU3 list. The report says
`com.sony.dtv.b2b.vendorprotocol` was already disabled; do not treat it as a change made by this
tool. The six explicitly enabled packages include Sony Eco Dashboard and device-control / IoT
services. Thirteen other apps were not named, so their package identities and states remain unknown.

The upstream VU3 notes say `com.sony.dtv.applicationdisabler` can re-enable
`com.sony.dtv.ecodashboard` at boot on related Sony firmware. Keep this dependency in mind before
changing either package.

## HOME and WIZARD_WIFI

The report resolves HOME to
`com.google.android.apps.tv.launcherx/.home.HomeActivity`. It also lists
`com.google.android.tungsten.setupwraith/.RecoveryActivity` as a HOME claimant at priority 1,
followed by Realtek and Android settings fallback activities.

The app reported `Home switch: blocked WIZARD_WIFI`. This is a safety stop. Do not force a Home
switch or disable setupwraith to get around it.

The upstream VU3 profile's second unit had its saved Wi-Fi network owned by
`android.uid.system`, so setupwraith could be disabled there. That observation does not establish
the owner on this unit. Check the live network owner first:

```sh
tools/tv.py 'dumpsys wifi' | grep -o 'cname=[^ ]*'
```

If the output names setupwraith, disabling it can delete the saved Wi-Fi network and disconnect ADB.
If the owner cannot be determined, stop and keep setupwraith enabled. Follow the existing guards in
`tools/disable.sh`; do not use `FORCE_SETUPWRAITH=1` or `FORCE_HOME=1` as a workaround.

## Launcher choice and remote behaviour

The upstream VU3 profile records that Realtek routes the Netflix, YouTube, Prime Video and YouTube
Music remote buttons through Google's stock `launcherx` receiver. Disabling launcherx can therefore
break those buttons. Its tested workaround keeps launcherx enabled and redirects Home to the
CleanYourTV launcher with key filtering off; details and the exact configuration are in
[`examples/sony-bravia-vu3/README.md`](../../examples/sony-bravia-vu3/README.md).

This is the preferred option to investigate for this device, but it still requires confirmation on
build `265-60835`. Keep Google TV available as the recovery/default option until the Home redirect,
real remote buttons, reboot and restoration have all been checked on this firmware.

## Components to preserve while validating

The upstream VU3 profile identifies these as TV/input or system dependencies. Keep them enabled
unless VU3-specific evidence proves otherwise:

- `com.realtek.dtv`
- `com.realtek.tv.atv`
- `com.realtek.tv.passthrough`
- `com.realtek.advancesetting`
- `com.android.tv` (Live TV)
- `com.sony.dtv.smartui` (Sony input-selection UI)
- `com.sony.dtv.sonysystemservice`
- `com.sony.dtv.m2.devicecontrolservice`
- `com.sony.dtv.ipcontrol` and `com.sony.dtv.ipcontrolaux`
- Realtek TV/input overlays and providers
- Google Play Store and Play Services, Bluetooth, Wi-Fi, Settings, remote and keyboard services

After any small-batch test, verify HOME, Settings, Wi-Fi, Bluetooth/remote, HDMI/CEC, tuner/Live TV,
streaming apps, casting, voice, timers and firmware updates. Do not infer package safety from a
package name or from a different firmware suffix.

## Recommended workflow for build 265-60835

1. Capture a fresh `tools/tv.py --probe`, installed-package list, disabled-package list, HOME
   resolution, Wi-Fi network owner and relevant settings before changing anything.
2. Keep the original backup and exact package list. The repository's `tools/disable.sh` records
   state for `tools/restore.sh`; never uninstall packages.
3. Compare the VU3 reference list against packages actually present on this unit. Review every
   package and its dependency before applying any new disable.
4. Keep the stock launcher enabled for the first launcher test. Follow the VU3 README's
   `stock_home_redirect` approach only after the Wi-Fi/setup guard passes.
5. Disable only owner-approved optional packages in small batches. Reboot and check the real remote
   after each relevant change.
6. Record any package re-enabled, regressions, and days of normal use. Do not label this firmware
   variant as tested until it has been lived with for several days.

Related: [Sony BRAVIA VU3 reference profile](../../examples/sony-bravia-vu3/README.md),
[TV-list workflow](../workflows/tv-list.md), and [repository safety rules](../../AGENTS.md).
