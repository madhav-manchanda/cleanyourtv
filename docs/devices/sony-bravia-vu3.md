# Sony BRAVIA VU3 (BRAVIA_VU3_4K)

**Status: diagnostic report only. This is not a tested debloat list.** The information below comes
from Clean Your TV app 0.3.3's report, read on 2026-10-31. No packages have been disabled by this
repository change, and the report does not establish that any candidate can be safely disabled for
several days.

## Device identity

| Property | Reported value |
|---|---|
| Manufacturer / brand | Sony |
| Retail model | BRAVIA VU3 |
| Android device | `BRAVIA_VU3_4K` |
| Android | 14 (SDK 34) |
| Firmware build | `Sony/BRAVIA_VU3/BRAVIA_VU3_4K:14/UKR1.240726.001/265-60835:user/release-keys` |
| Platform family | Realtek (also reported by the related VU31 profile) |
| Original Home | Google TV |
| Google Play services | Present |
| Report date | 2026-10-31 |

The repository already has a tested-for-observation Sony BRAVIA VU31 4K list at
[`examples/sony-bravia-vu31/`](../../examples/sony-bravia-vu31/debloat.txt). That device reports
the same Android release and firmware build prefix (`UKR1.240726.001`) and is also a Realtek
Google TV 14 model. It is a useful comparison, **not proof that the VU3 and VU31 are interchangeable**:
the device identifiers differ (`BRAVIA_VU3_4K` versus `BRAVIA_VU31_4K`). Do not apply the VU31
list wholesale to a VU3 without checking package presence, dependencies, inputs and behaviour on
this exact set.

## Package state in the supplied report

The report says there are 150 system apps:

| State | Count | Meaning |
|---|---:|---|
| Switched off by Clean Your TV | 8 | Already disabled by the phone app |
| Off, disabled | 1 | Factory-disabled package |
| On, enabled | 6 | Explicitly enabled |
| On, default | 135 | Default state |

The eight packages reported as switched off by the app are:

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

The package reported as already disabled by the TV is
`com.sony.dtv.b2b.vendorprotocol`. Do not assume that this was disabled by CleanYourTV or add it
to a restore list as though this tool changed it. The report also says 13 other apps were not named;
their identities and states are unknown.

The explicitly enabled packages include `com.sony.dtv.ecodashboard`,
`com.sony.dtv.m2.devicecontrolservice`, `com.sony.dtv.m2.seeds.auth`,
`com.sony.dtv.m2.seeds.iot` and `com.sony.dtv.m2.seeds.iot.remotestart`. Their presence is
not evidence that they are bloat. In particular, the related VU31 notes report that
`com.sony.dtv.applicationdisabler` can re-enable Eco Dashboard at boot; investigate that
relationship before changing either package.

## Home and setup-wizard observations

The report resolves HOME to:

```text
com.google.android.apps.tv.launcherx/.home.HomeActivity
```

It also reports these HOME claimants:

- `com.google.android.tungsten.setupwraith/.RecoveryActivity` (priority 1)
- `com.realtek.advancesetting/.FallbackHome` (priority -999)
- `com.android.tv.settings/.system.FallbackHome` (priority -1000)

The app reports `Home switch: blocked WIZARD_WIFI`. Treat this as a safety stop, not an invitation
to force the launcher change. The related VU31 notes explain that disabling
`com.google.android.tungsten.setupwraith` can remove saved Wi-Fi credentials when that package
owns the saved network. Before any Home handover or setup-wizard change, inspect the live Wi-Fi
owner and let the repository's existing Home/setup guards decide whether the operation can proceed.

Useful read-only checks, after ADB is connected and authorized:

```sh
tools/tv.py 'dumpsys wifi' | grep -o 'cname=[^ ]*'
tools/tv.py 'cmd package resolve-activity --brief -a android.intent.action.MAIN -c android.intent.category.HOME'
tools/tv.py 'cmd package query-activities --brief -a android.intent.action.MAIN -c android.intent.category.HOME'
tools/tv.py 'pm list packages -d'
```

Do not set `FORCE_SETUPWRAITH=1` or `FORCE_HOME=1` merely to get past the app's warning. Use
those existing escape hatches only under the conditions documented by `AGENTS.md` and
`tools/disable.sh`, with the owner's informed agreement and a verified recovery route.

## Realtek / Sony functionality to preserve while investigating

The related VU31 profile reports that the following components participate in TV, tuner, HDMI/AV,
input-menu, remote or Sony system behaviour. Treat them as **protected pending VU3-specific
verification**, not as debloat candidates:

- `com.realtek.dtv`
- `com.realtek.tv.atv`
- `com.realtek.tv.passthrough`
- `com.realtek.advancesetting` (also owns fallback and, on the related VU31, remote app-button handling)
- `com.android.tv` (Live Channels / Live TV on the related VU31)
- `com.sony.dtv.smartui` (Sony input-selection UI on the related VU31)
- `com.sony.dtv.sonysystemservice`
- `com.sony.dtv.m2.devicecontrolservice`
- `com.sony.dtv.ipcontrol` and `com.sony.dtv.ipcontrolaux`
- Realtek TV/input overlays and providers

The exact VU3 report lists these packages as enabled/default, but package state alone does not prove
what a feature depends on. Confirm HDMI, CEC, tuner/Live TV, Bluetooth/remote, Wi-Fi, AirPlay,
Chromecast/casting, voice input, timers and firmware updates after any future small-batch test.

## Balanced-cleaning plan for this exact report

1. Keep the eight already-disabled packages recorded above as **observed current state**, not as a
   validated VU3 recommendation. Ask the owner which features they use before retaining each change.
2. Keep `com.sony.dtv.b2b.vendorprotocol` in the baseline as factory-disabled; do not claim this
   tool disabled it.
3. Do not add a new `examples/sony-bravia-vu3/debloat.txt` yet. Per
   [the TV-list workflow](../workflows/tv-list.md), a folder under `examples/` requires a list
   whose packages were actually disabled and lived with for several days.
4. Make a complete backup before testing: installed packages, disabled packages, build properties,
   relevant settings, HOME resolution and Wi-Fi network owner. Keep the backup and the exact list
   used for the matching `tools/restore.sh` invocation.
5. Test only a small, owner-approved batch of optional packages at a time. Verify after each batch
   and after a cold reboot: HOME, Settings, Wi-Fi, Bluetooth/remote, HDMI/CEC, tuner/Live TV,
   streaming apps, casting, voice, timers and updates.
6. Record packages re-enabled, failures, reboot behaviour and the number of days lived with the
   changes. Only then promote the result to a tested example list.

## Launcher choice

The report shows Google TV's `launcherx` is currently the Home owner and that the phone app blocked
the switch because of `WIZARD_WIFI`. Keep Google TV as the safe default until the setup-wizard and
Wi-Fi ownership checks pass. If a custom launcher is later selected, install and verify it as Home
before disabling the stock launcher; retain the stock launcher as a selectable recovery option.
The repository's launcher settings can open Android's default-home picker on devices that expose it.

## Related reference

- [Sony BRAVIA VU31 4K tested notes/list](../../examples/sony-bravia-vu31/debloat.txt) — same
  Android 14 / `UKR1.240726.001` family and Realtek platform, but a different device identifier.
- [Workflow: add or update a TV list](../workflows/tv-list.md).
- [Repository safety rules](../../AGENTS.md).
