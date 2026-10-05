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

**Updating from 0.3.x:** if you never changed the placement, the strip moves
to 20 dp from the edge and 20 dp above the dock by itself. A placement you set
yourself stays as it is.

**Updating from 0.4.x:** the strip is now drawn **over other apps** by
default, so the car's pull-down shortcuts panel opens over it rather than under
it. If the **Drawing method** was left on its old default, it moves once by
itself. To draw it with the accessibility service again (full opacity, above the
car's own panels), choose **Accessibility** under **Drawing method** on the
Placement page. **Recent apps** needs **Usage access** (see step 4), and
**Back** is off until you turn it on.

**Updating from 0.5.x:** your favourites stay as they are, and there are no
folders until you make one (step 5). **Usage access** for Recent apps no longer
needs a command from a computer: run the one-tap setup again, or tap **Set up**
on the **Recent apps** page.

## 2. One-tap setup

On **Overview**, run the one-tap setup. The vendor Settings app is locked down
and offers none of the screens these grants normally live on. Instead,
G700 Home opens a session to the head unit's own ADB daemon over the loopback
address 127.0.0.1, as the `shell` user. There is no PC and no cable. In that
one session it:

1. grants itself `WRITE_SECURE_SETTINGS`, so that every later boot can switch accessibility back on by itself, and so can **Keep accessibility on** when another app switches it off;
2. allows **draw over other apps** (`SYSTEM_ALERT_WINDOW`), the window the strip is drawn in by default;
3. allows **install unknown apps** (`REQUEST_INSTALL_PACKAGES`) for self-updates;
4. allows the **notification listener**, which gives media access;
5. grants **location** (fine, coarse, then background) and **notifications**;
6. grants **phone calls** and **contacts**, for the Quick contacts widget;
7. adds its **accessibility service** to the enabled list, keeping every service already there, and sets `accessibility_enabled` to 1;
8. allows **Usage access** (`GET_USAGE_STATS`), which Recent apps needs.

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

If **Usage access** is later lost, the app quietly allows it again. It does
this only with a key the car already trusts, so it never shows a prompt. You can
also tap **Set up** on the **Recent apps** page (step 4).

### Manual route (a PC with adb)

These are the same grants, typed by hand, plus usage access for Recent apps.
First read the current accessibility list, then **append** to it. Never replace it, or you will switch off other
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
adb shell appops set com.g700.home GET_USAGE_STATS allow

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
| Accessibility service | **Knowing what is in front of display 0**, so the strip shows on the home screen only and steps aside for panels. **Back**, which goes back through it, and the Recent apps swipe on the very bottom edge. **Hosting** the strip at full opacity, if **Drawing method** is set to **Accessibility**. It reads window type, bounds and package only. It never reads screen text. Its only actions are going back (when Back is on) and opening the system's recent apps (if that option is on). | The strip still hides off the home screen, from the car launcher's own page setting, but it no longer steps aside for panels. There is no Back, and the bottom-edge swipe sits just above the dock instead. |
| Draw over other apps | The window the strip is drawn in by default (**Drawing method: Over other apps**), at 80% opacity, under the car's own panels. Also the bottom-edge swipe while accessibility is off. | The strip is drawn by the accessibility service instead, if it is on. With neither, there is no strip. |
| Notification access | Listing media sessions for **Now Playing**. Android ties `getActiveSessions` to it. No notification is ever read or stored. | Now Playing says media access is off. It never nags. |
| Location (with background) | Weather and prayer times for where the car is, and saving the car's position as a Navigate place. The last position is kept, so the weather card opens on it at once while GPS is still searching. | Weather and prayer times use a fixed city that you choose, or the weather card's fallback city. Navigate places can still be found by search. |
| Notifications | The quiet status notification of the strip service, with its **Stop** action. | The strip still runs. You just don't see the notification. |
| Install unknown apps | Installing updates from inside the app. | Update by downloading from the website instead. |
| Phone calls | **Quick contacts** starts a call at once, over the phone that is connected to the car by Bluetooth. | A tap opens the dialer with the number filled in; you tap call yourself. |
| Contacts | Picking people from the phonebook the phone shares with the car, in the Quick contacts settings. Only the names, numbers and photos you pick are kept, in the app. | Type names and numbers by hand. |
| Usage access | **Recent apps**: when each app was last open, and nothing else. The one-tap setup allows it, and the app quietly restores it if it is lost. | Recent apps offers **Set up** instead of listing apps, and the bottom-edge swipe stays off. |

## 3. Arrange the strip

- **Press Home.** By default the strip shows on the car's home screen, above the dock.
- **Tap the "+" edit tile** to open **Widgets** in the manager: add, remove, reorder, resize and configure cards.
- **Long-press any card** to open that card's settings.
- **Appearance:** glass clarity (Clear, Balanced, Frosted, Solid), edge light, accent colour, glow, strip size (Compact, Standard, Large, Extra large) and reduce motion.
- **Placement:** alignment (start, centre, end), edge margin (default 20 dp) and lift above the dock (default 20 dp). While this page is open, the real strip is shown live so you can see each change.
- **Weather:** each weather card follows the car's location or a fixed city. Two cards can show two places.
  - **Car location** opens on the last place the car was and switches to the live position once GPS answers. Set a **fallback city** to use when no position has come in after 1, 5, 15 or 30 minutes.
  - **Fixed city:** search several hundred cities offline, in English or Arabic, or any city worldwide when online. **Pin here** fixes the card to where the car is now.
  - **Tap the card** for the full forecast: the next hours, ten days, air quality and dust, UV, wind, humidity, pressure, visibility, sunrise and sunset, and the moon. Tap a day to see its hours. Close it with ✕ or Back.
- **Apps card:** by default it shows just the icon on a slim card, a tap opens DisplayMirror's app grid and a long press opens your favourites launcher. In its settings choose what a tap does, and whether the name shows (the card is square then). Apps cards you set up yourself before 0.3.0 keep their settings.
  - **Open an app:** opens the app you pick, or DisplayMirror if you pick none.
  - **App and favourites:** a tap opens the app; a long press opens your favourites launcher. Turn on **Swap tap and long press** for the other way round.
  - **Favourites:** a tap opens your favourites launcher.
- **Clock:** choose **Digital** or **Analogue**. With **Show seconds** on, the small digital clock shows small seconds under the time.
- **Energy:** shows battery and fuel as percentages. The range in km is gone, because the car often leaves it empty. A Vehicle card can still show EV range when the car reports it.
- **Tyres:** temperatures appear next to the pressures when the car reports them, also while a warning is on. A low tyre is marked in a high-contrast warning colour.
- **Quick contacts:** in its settings, add people from the phonebook or type a name and number. Up to six; the card shows as many as fit its size. Tap an avatar to call. Photos are kept in the app, so they stay after restarts and drives.
- **Prayer times:** follows the car's location, or a fixed city. Choose the calculation method, Asr (Standard or Hanafi) and whether to show the Hijri date. Tap the card for the full prayer screen: the day's times, the Qibla, the Hijri month (switch to **Timetable** for the next 30 days) and, in Ramadan, Imsak and Iftar. Until the card has a place, a tap opens its settings instead.
  - The method follows the country by default: Umm al-Qura for Saudi Arabia; Gulf for the UAE, Bahrain and Oman; their own methods for Kuwait and Qatar; Egyptian for Egypt and the Levant; Tehran for Iran; Karachi for South Asia; ISNA for the US and Canada; and Muslim World League everywhere else, Iraq included.
  - Jafari is available as a manual choice.
  - **Adjust:** move the Hijri date a day or two (−2 to +2) to follow a local moon sighting, move each time by up to 30 minutes either way, or hide a time. At least one time stays on. The card, the prayer screen, the calendar and Ramadan all follow. Hold **−** or **+** to step quickly.
  - **Reminder:** on by default. The Prayer card glows softly in glass blue before a prayer (**Before the prayer**, 20 minutes, 5 to 60) and with a brighter icy glow at the prayer time (**At prayer time**, 3 minutes, 1 to 15). Switch it off, or change the times, in the card's settings under **Reminder**. Sunrise and hidden times never glow.
- **Navigate:** save places with **Use the car's location** or search. Choose Google Maps, Waze, or Automatic (the default map app). A rebuilt Google Maps, such as ReVanced, counts as Google Maps when the official app isn't installed. Tap a place on the card to start the route.
- **Launcher** page: choose the favourite apps, the grid (4–10 columns, 2–6 rows), icon size, labels and how much the screen behind is dimmed, with a live preview. The launcher itself has no title and no "Add apps" tile. Tap the pen, or hold an icon and drag it, to edit: drag to reorder, remove apps, or tap **Add apps**. Folders are in step 5.
- **Quick tour:** it opens on first start and once after updating to 0.3.0. Replay it from **About**.

## 4. Recent apps, Back and keeping accessibility on

### Recent apps

1. Open the manager's **Recent apps** page.
2. Under **Usage access**, tap **Set up**. It runs the one-tap setup in place
   and shows the result in the row. The one-tap setup has usually done this
   already. If the head unit has a Usage access screen, the page offers it too,
   and allowing **G700 Home** there works as well.
3. Choose how to open it:
   - **Swipe up from the bottom edge** (on by default): a short swipe up from
     the middle third of the bottom edge. With the accessibility service on, it
     is the very bottom edge; without it, just above the dock.
   - **Swipe up on the strip** (on by default): a swipe up on the widget cards.
   - The **Recent apps** button in the favourites launcher is always there.
   - **Use the system's recent apps** (off by default) asks the head unit for
     its own recents screen first. It needs the accessibility service. If
     nothing happens, turn it off.

In recent apps, tap a card to switch to that app, swipe a card up to close it,
or tap **Clear all**. Tap outside, swipe down or press Back to leave. A closed
app stays off the list until you use it again; **Show closed apps again** brings
them all back.

### Back

1. Make sure the accessibility service is on. The **Back** page says **Needs
   accessibility** if it isn't.
2. On the **Back** page, under **Style**, choose **Edge swipe** or **Floating
   button**. **Off** is the default.
3. **Edge swipe:** pick **Left**, **Right** or **Both** under **Edges**. Under
   **Where on the edge**, drag a handle to set the part of the edge that takes
   the swipe, or drag the middle to move it. That part glows on the screen while
   you choose. Swipe in from the edge and let go; a translucent glass bulge with a
   chevron follows your finger and changes tint once letting go will go back.
   Slide back towards the edge to cancel.
4. **Floating button:** tap it to go back. Hold it to lift it, then drag it up,
   down or to the other side; it settles against the nearer edge. Turn on **Lock
   position** so holding it doesn't move it. **Reset position** puts it back on
   the left edge, halfway down. It dims after a few seconds untouched.

Back is hidden on the car's home screen, where there is nothing to go back to.

### Keep accessibility on

Another app on the car can rewrite the list of enabled accessibility services
and drop G700 Home from it. **Keep accessibility on**, on **Overview**, is on by
default: it puts G700 Home's service back straight away and leaves every other
service as it was. The row shows how many times it has done so.

- It needs `WRITE_SECURE_SETTINGS`, which the one-tap setup grants. By hand:
  `adb shell pm grant com.g700.home android.permission.WRITE_SECURE_SETTINGS`.
- Without that grant it tries the setup's route quietly every 10 minutes, and
  only with a key the car already trusts, so it never shows a prompt.
- If the other app switches it off again each time, it waits longer between
  tries and then pauses for 30 minutes. The row says until when.

## 5. Launcher folders and app permissions

### Folders

Favourites is the launcher's first and default page. Folders are optional.

1. Open the manager's **Launcher** page and find **Folders**. Tap **New folder**
   and give it a name. Each folder's row lets you rename it, move it up or
   down, or delete it.
2. Open a folder's app picker to choose its apps.
3. In the launcher, the folders show as pills across the top, with **View all**
   first. The pills scroll sideways when there are many. **View all** shows every
   folder as a tile, with a peek at its first apps. Tap a tile to open it.

Each app lives in one place: Favourites or one folder. Adding an app to a folder
takes it out of where it was. Deleting a folder asks first, and its apps return
to Favourites.

### The app menu

Press and hold an app in the launcher:

- **Open** starts it.
- **Move to folder** moves it to Favourites or a folder. **New folder…** asks
  for a name and moves it there.
- **Uninstall** is offered for apps you installed. The system's own screen asks
  you to confirm. G700 Home never uninstalls an app silently or through adb.
- **Permissions** opens the app's permissions (below).

To reorder, tap the pencil button, or hold an icon and then drag it.

### App permissions

The **Permissions** item opens a page for that app. It lists the app's runtime
permissions (location, contacts, microphone and so on) and its special access
(such as draw over other apps), with a switch for each. **Grant all** turns on
everything that is off, as far as the head unit allows, and asks first. Some
permissions are fixed by the system or kept by the head unit, and the row says
so.

It uses the same local debugging service as the one-tap setup, so the first time
the car may show **"Allow USB debugging?"**. Tap **Allow** with **Always allow**
ticked. Everything stays on the head unit. It changes only the permissions of
the app you picked, never a car setting, and it never uninstalls an app through
adb.

## Troubleshooting by reason

The **Overview** page states whether the strip is showing and why. The rules
are checked in this order. The first one that applies decides.

| Reason | Showing? | What's going on | What to do |
|---|---|---|---|
| **Disabled** | no | The strip is switched off, or someone used **Stop** on its notification. The service is not running at all. | Switch the strip on in Overview. |
| **Preview** | yes | The manager is on **Placement**, so the strip is shown live. | Nothing. |
| **ManagerOpen** | no | The manager is open on any other page. | Press Home or close the manager. |
| **DisplayOff** | no | Display 0 reports that it is off. | It returns when the screen comes on. |
| **OwnApp** | no | G700 Home's own screen is in front: the favourites launcher, recent apps or the forecast. | Close it, or press Home. |
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
`WRITE_SECURE_SETTINGS` and **Keep accessibility on** is on. Running the
one-tap setup once gives it that permanently.

**Accessibility keeps switching itself off.** Another app is rewriting the list
of enabled services. Leave **Keep accessibility on** switched on (Overview). If
its row says it is paused, the other app switched it off again each time; it
carries on by itself after the pause.

**The strip looks a little transparent.** It is drawn over other apps, the
default, which Android caps at 80% opacity so that touches still reach the home
screen. For full opacity, switch the accessibility service on and choose
**Accessibility** under **Drawing method** on the Placement page. The car's
pull-down shortcuts panel then opens under the strip.

**Recent apps says "Allow usage access", or is empty.** Tap **Set up** on
the **Recent apps** page (step 4), or run the one-tap setup again. Only apps with an icon in the app list, used
in the last 12 hours, are shown. An app you closed stays hidden until you use
it again, or until you tap **Show closed apps again**.

**A closed app is still running.** Closing ends only background apps with
nothing keeping them running. Music, navigation and other apps with a running
service can carry on. Nothing is force-stopped.

**The bottom-edge swipe does nothing.** It needs usage access, the strip
switched on, and **Swipe up from the bottom edge** on. It steps aside while the
keyboard, a dialog or the shade is open, and while a G700 Home screen is up.
Without the accessibility service it sits just above the dock, not on the very
bottom edge.

**There is no back swipe or back button.** Choose a style on the **Back** page,
and switch the accessibility service on. Back stays hidden on the car's home
screen, and the edge swipe steps aside while the keyboard is up.

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

**An app is missing from Favourites.** Each app lives in one place. If you moved it
to a folder (or added it to one), it is in that folder: tap **View all** or the
folder's pill. Deleting a folder puts its apps back in Favourites.

**App permissions says "Allow USB debugging?".** The car asks once, on the main
screen. Tick **Always allow**, tap **Allow**, then tap **Try again**. If it says
local debugging is off, the head unit does not offer it. Use the manual route or
the one-click install from a computer to change an app's permissions instead.

**A switch on App permissions won't stay on.** The system fixes some permissions,
and some head units keep a permission as it was. The row says so. **Grant all**
reports how many it could not grant.

**The Prayer card doesn't glow.** Check **Reminder** in the card's settings. The
glow follows the prayer times as adjusted there, and sunrise and hidden times
never glow. With **Reduce motion** on, the glow stays still instead of pulsing.

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
