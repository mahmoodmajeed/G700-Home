# Setting up G700 Home on the car

This is the owner's guide: install, the one-tap setup, what each permission is
for, and what to do when the strip isn't where you expect it. Each
troubleshooting entry is keyed to the reason the manager's **Overview** page
shows.

## What you need

- A Jetour G700 with the stock head unit (Android 14).
- A way to reach the internet from the car for the download. A phone hotspot is fine.
- Optional: **DisplayMirror** (`com.example.displaymirror`). It is the app grid that the Apps card opens. If it was provisioned on this car, setup is also prompt-free (see step 2).

## 1. Install

1. On the car, open **home.g700.mahmoodmajeed.com** in a browser. The link
   **home.g700.mahmoodmajeed.com/download** starts the latest APK straight away.
   You can also take the `.apk` from
   [GitHub releases](https://github.com/mahmoodmajeed/G700-Home/releases).
2. Open `G700Home-v<version>-release.apk` and tap **Install**. If Android asks,
   allow the browser to install unknown apps.
3. Open **G700 Home**, either with the installer's **Open** button or from
   DisplayMirror's grid. If the grid doesn't show it yet, scroll to the end of
   the list to refresh it.

Later updates come from inside the app, on the **About** page. Updating keeps
your layout and settings. The app reopens on About afterwards so you
can see the new version.

## 2. One-tap setup

On **Overview**, run the one-tap setup. The vendor Settings app is locked down
and offers none of the screens these grants normally live on. Instead,
G700 Home opens a session to the head unit's own ADB daemon over the loopback
address 127.0.0.1, as the `shell` user. There is no PC and no cable. In that
one session it:

1. grants itself `WRITE_SECURE_SETTINGS`, so that every later boot can switch accessibility back on by itself;
2. allows **draw over other apps** (`SYSTEM_ALERT_WINDOW`), which is the fallback window;
3. allows **install unknown apps** (`REQUEST_INSTALL_PACKAGES`) for self-updates;
4. allows the **notification listener**, which gives media access;
5. grants **location** (fine, coarse, then background) and **notifications**;
6. adds its **accessibility service** to the enabled list, keeping every service already there, and sets `accessibility_enabled` to 1.

Each step is best-effort. Afterwards the checklist re-reads the real state, so
a refusal shows up as an unticked row rather than a false success.

| Result | What it means | What to do |
|---|---|---|
| **Done** | Accessibility is on, and the other grants were attempted. | Nothing more. Check the checklist. |
| **"Allow USB debugging?"** | adbd showed its one-time key prompt on the main screen. | Tick **Always allow from this computer**, tap **Allow**, then run the setup again. If DisplayMirror's provisioning key is on the car (`/data/local/tmp/adbkey`), G700 Home reuses it and this prompt never appears. |
| **No ADB** | Nothing answered on any candidate port. The app tries the ports in `persist.adb.tcp.port`, `service.adb.tcp.port` and `ro.adb.port`, then 5555, 55556 and 5037. | Local (wireless) debugging is off on this head unit. Use the manual route below, or switch local ADB on and retry. |
| **Failed** | The session opened, but the accessibility write did not take, or the key could not be loaded. | Retry once. If it fails again, use the manual route. |

If the app already holds `WRITE_SECURE_SETTINGS` from an earlier setup, the
button switches accessibility on directly, without ADB.

### Manual route (a PC with adb)

These are the same grants, typed by hand. First read the current accessibility
list, then **append** to it. Never replace it, or you will switch off other
services such as DisplayMirror's.

```bash
adb shell pm grant com.g700.home android.permission.WRITE_SECURE_SETTINGS
adb shell appops set com.g700.home SYSTEM_ALERT_WINDOW allow
adb shell appops set com.g700.home REQUEST_INSTALL_PACKAGES allow
adb shell cmd notification allow_listener com.g700.home/com.g700.home.media.MediaListenerService
adb shell pm grant com.g700.home android.permission.ACCESS_FINE_LOCATION
adb shell pm grant com.g700.home android.permission.ACCESS_COARSE_LOCATION
adb shell pm grant com.g700.home android.permission.ACCESS_BACKGROUND_LOCATION
adb shell pm grant com.g700.home android.permission.POST_NOTIFICATIONS

adb shell settings get secure enabled_accessibility_services
# then, with <existing> being exactly what that printed (omit "<existing>:" if it printed null):
adb shell settings put secure enabled_accessibility_services "<existing>:com.g700.home/com.g700.home.foreground.HomeAccessibilityService"
adb shell settings put secure accessibility_enabled 1
```

## What each grant does

**None of them is required.** Each missing grant takes away one thing and
nothing else.

| Grant | Used for | Without it |
|---|---|---|
| Accessibility service | **Hosting** the strip as a trusted overlay: full opacity, gaps pass touches through. **Knowing what is in front of display 0**, so the strip shows on the home screen only and steps aside for panels. It reads window type, bounds and package only. It never reads screen text and never performs actions. | The strip falls back to a normal app overlay at 80% opacity, shown everywhere, because nothing can tell it what is in front. |
| Draw over other apps | The fallback window, used when accessibility is off. | Fine while accessibility is on. With neither, there is no strip. |
| Notification access | Listing media sessions for **Now Playing**. Android ties `getActiveSessions` to it. No notification is ever read or stored. | Now Playing says media access is off. It never nags. |
| Location (with background) | Weather for where the car is. One last fix is kept. | Weather cards use a fixed city that you choose. |
| Notifications | The quiet status notification of the strip service, with its **Stop** action. | The strip still runs. You just don't see the notification. |
| Install unknown apps | Installing updates from inside the app. | Update by downloading from the website instead. |

## 3. Arrange the strip

- **Press Home.** By default the strip shows on the car's home screen, above the dock.
- **Tap the "+" edit tile** to open **Widgets** in the manager: add, remove, reorder, resize and configure cards.
- **Long-press any card** to open that card's settings.
- **Appearance:** glass clarity (Clear, Balanced, Frosted, Solid), edge light, accent colour, glow, strip size (Compact, Standard, Large, Extra large) and reduce motion.
- **Placement:** alignment (start, centre, end), edge margin (default 76 dp, which lines the first card up with the first dock icon) and lift above the dock (default 12 dp). While this page is open, the real strip is shown live so you can see each change.
- **Weather:** each weather card follows the car's location or a fixed city. Two cards can show two places.

## Troubleshooting by reason

The **Overview** page states whether the strip is showing and why. The rules
are checked in this order. The first one that applies decides.

| Reason | Showing? | What's going on | What to do |
|---|---|---|---|
| **Disabled** | no | The strip is switched off, or someone used **Stop** on its notification. The service is not running at all. | Switch the strip on in Overview. |
| **Preview** | yes | The manager is on **Placement**, so the strip is shown live. | Nothing. |
| **ManagerOpen** | no | The manager is open on any other page. | Press Home or close the manager. |
| **DisplayOff** | no | Display 0 reports that it is off. | It returns when the screen comes on. |
| **OwnApp** | no | G700 Home's own screen is in front. | Press Home. |
| **HiddenApp** | no | The app in front is on your **Never show on** list. That list beats everything below it, including **Everywhere**. | Remove the app from Never show on. |
| **Covered** | no | **Step aside** is on and something is open over the screen: the notification shade, a dialog, the keyboard, or an overlay launcher's panel such as HELM's dashboard. | Close it and the strip returns. If a panel is wrongly treated as covering, you can switch Step aside off. |
| **Always** | yes | Visibility is set to **Everywhere**. | Set it back to **Home screen** if that isn't what you want. |
| **NoWatcher** | yes | The accessibility service is off. Without it nothing can tell what is in front, so the strip shows everywhere. Because it is an app overlay, it is a little translucent (80%). | Run the one-tap setup again. Once the app holds `WRITE_SECURE_SETTINGS`, it switches accessibility back on at every boot. |
| **Unknown** | yes | The service is on but hasn't seen a window yet. This is usual for a moment after boot. | Wait a second, or open and close any app. |
| **Home** | yes | The car's home screen (or your default HOME app) is in front. | Nothing. |
| **ShownApp** | yes | The app in front is on your **Also show on** list. | Nothing. |
| **OtherApp** | no | Another app is in front and visibility is **Home screen**. | Add the app to **Also show on**, or set visibility to **Everywhere**. |

### Other symptoms

**The status says it is showing, but there is no strip.** No window could be
attached. That happens when neither accessibility nor overlay permission is
available, or the window manager was still starting. The service retries 20
times, 5 s apart. Run the one-tap setup if the checklist shows both grants
missing.

**The strip is gone after a reboot.** The strip only starts at boot if it is
switched on. Accessibility is restored at boot only if the app holds
`WRITE_SECURE_SETTINGS`. Running the one-tap setup once gives it that
permanently.

**The strip looks a little transparent.** It is running as an app overlay,
which Android caps at 80% opacity so that touches still reach the home screen.
Switch the accessibility service on, and leave the **Window** setting on
**Automatic**, which is the default.

**Taps near the strip don't reach the car or the dock.** On Android 13 and
later only the cards themselves take touches, and the glow and shadow around
them do not. If you see this on the car, please report it.

**The strip covers the dock, or floats too high.** Adjust **Height above the dock** on the
Placement page. The window follows the dock's measured top edge.

**Now Playing is empty or missing.**

- It says media access is off: grant notification access (setup step 4).
- It disappears: **Show the card** is set to **While playing**, or to **Until idle**, which leaves 2, 5, 10 or 30 minutes after the music stops. Change this in the card's settings.
- The player app doesn't publish a media session: nothing can be shown for it.

**The weather is old or missing.**

- Weather needs the internet. The last reading stays on screen and is marked stale after 3 hours.
- Without location, set a fixed city in the card's settings.

**Car figures show "—".** The car didn't report that value, or reported
something outside its plausible range. G700 Home shows "—" rather than a wrong
number. Values are read only while the strip is showing.

**The Apps card does nothing.** DisplayMirror isn't installed. Use Shortcut
cards for the apps you want instead.
