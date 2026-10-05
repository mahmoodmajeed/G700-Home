# Setting up G700 Home on the car

This is the owner's guide: install, the one-tap setup, what each permission is
for, and what to do when the strip isn't where you expect it. Each
troubleshooting entry is keyed to the reason the manager's **Overview** page
shows.

## What you need

- A Jetour G700 with the stock head unit (Android 14).
- For the one-click install from a computer: Chrome or Edge on Windows, macOS, Linux or ChromeOS, and a USB-A to USB-A **data** cable.
- For the install on the car: a way to reach the internet from the car for the download. A phone hotspot is fine.
- Optional: **DisplayMirror** (`com.example.displaymirror`). It is the app grid that the Apps card opens. If it was provisioned on this car, setup is also prompt-free (see step 2).

## 1. Install

### From a computer, in one click

This also does the whole setup of step 2.

1. Park, and keep the car switched on.
2. Turn on ADB (USB debugging) in the car's engineering menu.
3. Connect the car's upper USB-A port on the driver's side to a computer with
   a USB-A to USB-A **data** cable. Charge-only cables don't work. Chrome on an
   Android phone also works, through an OTG adapter.
4. Close anything else that uses ADB, such as Android Studio, scrcpy or an
   `adb` window. Only one program can talk to the car at a time.
5. Open **[home.g700.mahmoodmajeed.com/install](https://home.g700.mahmoodmajeed.com/install)**
   in **Chrome** or **Edge**. Other browsers can't reach USB devices.
6. Press **Connect to your car and install** and pick the car in the browser's
   list.
7. The car asks **"Allow USB debugging?"**. Tick **Always allow** and tap
   **Allow**.
8. Wait for the ticks: installing, permissions, settings. When it says done,
   G700 Home opens on the car.

If the car doesn't appear in the list, check the cable and the port. On
Windows, the car's ADB interface needs the WinUSB driver (Google USB Driver).
If the page says the device is busy, another ADB program still holds it: close
it, or run `adb kill-server`.

### On the car

1. On the car, open **home.g700.mahmoodmajeed.com** in a browser. The link
   **home.g700.mahmoodmajeed.com/download** starts the latest APK straight away.
   You can also take the `.apk` from
   [GitHub releases](https://github.com/mahmoodmajeed/G700-Home/releases/latest).
2. Open `G700Home-v<version>-release.apk` and tap **Install**. If Android asks,
   allow the browser to install unknown apps.
3. Open **G700 Home**, either with the installer's **Open** button or from
   DisplayMirror's grid. If the grid doesn't show it yet, scroll to the end of
   the list to refresh it.
4. Run the one-tap setup (step 2).

### Updates

Later updates come from inside the app, on the **About** page. Updating keeps
your layout and settings. The app reopens on About afterwards so you
can see the new version.

**Updating from 0.2.x:** Overview shows the setup as not finished, because the
new Quick contacts widget needs phone access. Run the one-tap setup once more.
The quick tour also opens once after this update.

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
6. grants **phone calls** and **contacts**, for the Quick contacts widget;
7. adds its **accessibility service** to the enabled list, keeping every service already there, and sets `accessibility_enabled` to 1.

Each step is best-effort. Afterwards the checklist re-reads the real state, so
a refusal shows up as an unticked row rather than a false success.

| Result | What it means | What to do |
|---|---|---|
| **Done** | Accessibility is on, and the other grants were attempted. | Nothing more. Check the checklist. |
| **"Allow USB debugging?"** | adbd showed its one-time key prompt on the main screen. | Tick **Always allow from this computer**, tap **Allow**, then run the setup again. If DisplayMirror's provisioning key is on the car (`/data/local/tmp/adbkey`), G700 Home reuses it and this prompt never appears. |
| **No ADB** | Nothing answered on any candidate port. The app tries the ports in `persist.adb.tcp.port`, `service.adb.tcp.port` and `ro.adb.port`, then 5555, 55556 and 5037. | Local (wireless) debugging is off on this head unit. Use the one-click install from a computer, or the manual route below, or switch local ADB on and retry. |
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
adb shell pm grant com.g700.home android.permission.CALL_PHONE
adb shell pm grant com.g700.home android.permission.READ_CONTACTS

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
| Accessibility service | **Hosting** the strip as a trusted overlay: full opacity, gaps pass touches through. **Knowing what is in front of display 0**, so the strip shows on the home screen only and steps aside for panels. It reads window type, bounds and package only. It never reads screen text and never performs actions. | The strip falls back to a normal app overlay at 80% opacity. It still hides off the home screen, from the car launcher's own page setting, but it no longer steps aside for panels. |
| Draw over other apps | The fallback window, used when accessibility is off, or when **Drawing method** is set to **Over other apps**. | Fine while accessibility is on. With neither, there is no strip. |
| Notification access | Listing media sessions for **Now Playing**. Android ties `getActiveSessions` to it. No notification is ever read or stored. | Now Playing says media access is off. It never nags. |
| Location (with background) | Weather and prayer times for where the car is, and saving the car's position as a Navigate place. The last position is kept, so the weather card opens on it at once while GPS is still searching. | Weather and prayer times use a fixed city that you choose, or the weather card's fallback city. Navigate places can still be found by search. |
| Notifications | The quiet status notification of the strip service, with its **Stop** action. | The strip still runs. You just don't see the notification. |
| Install unknown apps | Installing updates from inside the app. | Update by downloading from the website instead. |
| Phone calls | **Quick contacts** starts a call at once, over the phone that is connected to the car by Bluetooth. | A tap opens the dialer with the number filled in; you tap call yourself. |
| Contacts | Picking people from the phonebook the phone shares with the car, in the Quick contacts settings. Only the names and numbers you pick are kept. | Type names and numbers by hand. |

## 3. Arrange the strip

- **Press Home.** By default the strip shows on the car's home screen, above the dock.
- **Tap the "+" edit tile** to open **Widgets** in the manager: add, remove, reorder, resize and configure cards.
- **Long-press any card** to open that card's settings.
- **Appearance:** glass clarity (Clear, Balanced, Frosted, Solid), edge light, accent colour, glow, strip size (Compact, Standard, Large, Extra large) and reduce motion.
- **Placement:** alignment (start, centre, end), edge margin (default 16 dp) and lift above the dock (default 16 dp). While this page is open, the real strip is shown live so you can see each change.
- **Weather:** each weather card follows the car's location or a fixed city. Two cards can show two places.
  - **Car location** opens on the last place the car was and switches to the live position once GPS answers. Set a **fallback city** to use when no position has come in after 1, 5, 15 or 30 minutes.
  - **Fixed city:** search several hundred cities offline, in English or Arabic, or any city worldwide when online. **Pin here** fixes the card to where the car is now.
  - **Tap the card** for the full forecast: the next hours, ten days, air quality and dust, UV, wind, humidity, pressure, visibility, sunrise and sunset, and the moon. Tap a day to see its hours. Close it with ✕ or Back.
- **Apps card:** by default it shows just the icon, a tap opens DisplayMirror's app grid and a long press opens your favourites launcher. In its settings choose what a tap does, and whether the name shows. Apps cards you set up yourself before 0.3.0 keep their settings.
  - **Open an app:** opens the app you pick, or DisplayMirror if you pick none.
  - **App and favourites:** a tap opens the app; a long press opens your favourites launcher.
  - **Favourites:** a tap opens your favourites launcher.
- **Clock:** choose **Digital** or **Analogue**.
- **Energy:** shows battery and fuel as percentages. The range in km is gone, because the car often leaves it empty. A Vehicle card can still show EV range when the car reports it.
- **Tyres:** temperatures appear next to the pressures when the car reports them, also while a warning is on. A low tyre is marked in a high-contrast warning colour.
- **Quick contacts:** in its settings, add people from the phonebook or type a name and number. Up to six; the card shows as many as fit its size. Tap an avatar to call.
- **Prayer times:** follows the car's location, or a fixed city. Choose the calculation method, Asr (Standard or Hanafi) and whether to show the Hijri date.
  - The method follows the country by default: Umm al-Qura for Saudi Arabia; Gulf for the UAE, Bahrain and Oman; their own methods for Kuwait and Qatar; Egyptian for Egypt and the Levant; Tehran for Iran; Karachi for South Asia; ISNA for the US and Canada; and Muslim World League everywhere else, Iraq included.
  - Jafari is available as a manual choice.
- **Navigate:** save places with **Use the car's location** or search. Choose Google Maps, Waze, or Automatic (the default map app). Tap a place on the card to start the route.
- **Launcher** page: choose the favourite apps, the grid (4–10 columns, 2–6 rows), icon size, labels and how much the screen behind is dimmed, with a live preview. The launcher itself has no title and no "Add apps" tile. Tap the pen (or press and hold an icon) to edit: drag to reorder, remove apps, or tap **Add apps**.
- **Quick tour:** it opens on first start and once after updating to 0.3.0. Replay it from **About**.

## Troubleshooting by reason

The **Overview** page states whether the strip is showing and why. The rules
are checked in this order. The first one that applies decides.

| Reason | Showing? | What's going on | What to do |
|---|---|---|---|
| **Disabled** | no | The strip is switched off, or someone used **Stop** on its notification. The service is not running at all. | Switch the strip on in Overview. |
| **Preview** | yes | The manager is on **Placement**, so the strip is shown live. | Nothing. |
| **ManagerOpen** | no | The manager is open on any other page. | Press Home or close the manager. |
| **DisplayOff** | no | Display 0 reports that it is off. | It returns when the screen comes on. |
| **OwnApp** | no | G700 Home's own screen is in front: the favourites launcher or the forecast. | Close it, or press Home. |
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

**The strip covers the dock, or floats too high.** Adjust **Height above the
dock** on the Placement page. The window follows the dock's measured top edge.

**Overview says the setup isn't finished after an update.** A new widget needs
a permission the earlier setup didn't grant. After updating from 0.2.x, this is
phone access for Quick contacts. Run the one-tap setup once more.

**Now Playing is empty or missing.**

- It says media access is off: grant notification access (setup step 4).
- It disappears: **Show the card** is set to **While playing**, or to **Until idle**, which leaves 2, 5, 10 or 30 minutes after the music stops. Change this in the card's settings.
- The player app doesn't publish a media session: nothing can be shown for it.

**The weather is old or missing.**

- Weather needs the internet. The last reading stays on screen and is marked stale after 3 hours. The forecast says how old its data is.
- **"Last known location"** means GPS hasn't answered since the car started, so the card shows where the car last was. It switches over by itself once a position comes in. The weather settings show which source is in use and which location providers are on.
- If GPS takes long, set a **fallback city** in the card's settings. Without location access the fallback applies at once.
- Or set a **fixed city** in the card's settings.

**Car figures show "—".** The car didn't report that value, or reported
something outside its plausible range. G700 Home shows "—" rather than a wrong
number. Values are read only while the strip is showing.

**The Apps card does nothing or says "Not installed".** The app it opens isn't
installed: DisplayMirror by default. Pick another app in the card's settings,
or switch it to **Favourites**.

**The favourites launcher is empty.** Tap **Add apps**, or choose favourites on
the manager's **Launcher** page.

**Quick contacts opens the dialer instead of calling.** The app doesn't have
the phone-call permission yet. Run the one-tap setup on **Overview** again; it
now includes phone calls and contacts.

**The phonebook list is empty.** The phone hasn't shared its contacts with the
car. Allow contact sharing for the car in the phone's Bluetooth settings, or
type the number by hand.

**Tyre temperatures are missing.** Some cars report pressures but no
temperatures. When none of the four wheels reports one, the card shows
pressures only.

**Navigate does nothing.** No map app is installed on the head unit, or the
one chosen in the card's settings was removed. Choose **Automatic**, or install
Google Maps or Waze.

**There is no climate (A/C) widget.** G700 Home only ever reads car data and
never changes a car setting, so it can't switch the A/C.

**Show the quick tour again.** Open **About** and tap **Show the quick tour**.

**The launcher isn't blurred.** Android turns off blur behind windows in
battery saver, during some video, and on builds where the maker switches it
off. The launcher then dims the screen without the blur. Choose **Dense** on
the Launcher page if the names are hard to read.
