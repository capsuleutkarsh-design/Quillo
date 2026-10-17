# Administrator & IT guide

Everything needed to install, run and look after Quillo in a studio.

> Quillo was called *LAN Messenger* before version 1.6.0. Upgrading keeps everything: the installers still use the old internal names (data folders, registry keys, the service task, program files), so only what you see changes.
For what the app does, see the [README](README.md).

- [1. Install the server](#1-install-the-server)
- [2. First-time setup](#2-first-time-setup)
- [3. Designations and permissions](#3-designations-and-permissions)
- [4. Install the clients](#4-install-the-clients)
- [5. Updating](#5-updating)
- [6. Running the server: service, data, backups](#6-running-the-server-service-data-backups)
- [6a. Chat backup, message history and shared files](#6a-chat-backup-message-history-and-shared-files)
- [7. Pipeline API (render farm, scripts)](#7-pipeline-api-render-farm-scripts)
- [8. Network ports](#8-network-ports)
- [9. Security and privacy](#9-security-and-privacy)
- [10. Limits](#10-limits)

---

## 1. Install the server

Pick one always-on PC (a small workstation or a VM is plenty; load-tested with 500 people online at once: 500 signed in within 2 s, ~390,000 deliveries with none lost, typical delivery 3–40 ms, server ~460 MB of RAM).
Run **`Quillo-Server-Setup-x.y.z.exe`**. It first asks how to install:

- **Install for all users** (administrator, **the default**): keep the tasks
  - **Allow the server through Windows Firewall**: needed for other PCs to connect
  - **Run as a background service**: the server starts with Windows, before anyone signs in, and starts again by
    itself 30 s after it stops unexpectedly (alternative: *start in the tray when I log in*)
- **Install for me only** (no administrator needed): installs into your user folder and runs the server in the
  system tray under your Windows account, starting when you sign in. Your mapped drives and NAS logins work as
  they are. The server runs while that account is signed in (a locked screen is fine). The first time it starts,
  Windows Firewall may ask to *Allow access* (an administrator approves this once; or IT allows TCP 5150 and
  UDP 5151).

An upgrade keeps the way the server was installed.

The setup installs to `C:\Program Files\Quillo Server` (all users) or `%LOCALAPPDATA%\Programs\Quillo Server` (just for me); an upgrade keeps the folder the earlier version used. The page **Where to keep the data** asks for:

| Folder | Default | Where it may be |
|---|---|---|
| Server data (database, settings, certificate) | `C:\ProgramData\LAN Messenger Server` (all users) or `%LOCALAPPDATA%\LAN Messenger Server` (just for me) | a **local disk** only (a database on a network share can get damaged) |
| Shared files (**the central folder**) | `<data>\files` | your **file server**: a share `\\server\share\...` |
| Database backups | `<data>\backups` | used only when the shared files are on the server PC itself (otherwise they go to `<shared files>\Quillo_Server\Backups`) |
| Server log | `<data>` | any disk, or a share |

Use `\\server\share\...` paths, not mapped letters like `Z:` (the background service can't see those). The service
runs as SYSTEM and reaches a share as the **computer account** (`DOMAIN\PCNAME$`): give that account *Modify* on
the share, or run the server in the tray instead. If a share is down, the server still starts and the dashboard
says so. Setup lists your mapped network drives by their network path, and *Browse* offers them first.
On an upgrade the page shows the current folders (the database folder is fixed); folders can also be moved in
console → *Settings*.
In service mode the data folder is readable only by Administrators and SYSTEM (in tray mode also the account that
installed it). The data folder must be a folder, not a whole drive. On uninstall setup asks whether to delete the
server data: only Quillo's own files in the data folder go (database, settings, certificate, photos, updates, and
the default `files\` and `backups\`), then the folder if nothing else is in it; files and backups kept elsewhere are
never deleted. If you keep it, the next install finds it again.

**The central folder: nothing important stays only on the server PC.** The database and the server's key stay
on the server PC's own disk. When the shared files are on your file server (or another disk), the server keeps
everything a new server PC needs next to them:

| In the shared-files folder | What | Updated |
|---|---|---|
| `Quillo_Server\Backups\messenger_2026-10-13_1400.db` (+ `config_….json`, `tls\`, `avatars\`, `about.json`) | the database (every account and chat), the settings, the certificate, the profile photos | **every hour** when something changed, and when the server stops. Kept: the newest 24, plus one a day for 14 days (*Settings → Backups*) |
| `Quillo server data\Chat backup\Rooms\<room>\2026-09.txt` | a room's messages, one text file per month | every night |
| `Quillo server data\Chat backup\People\<A + B>\2026-09.txt` | a direct chat between two people (`<name> - My space` for notes to yourself) | every night |
| `Quillo server data\User list\Users.xlsx`, `Users.csv` | everyone, with department, designation and reports-to. The .xlsx can be imported into any Quillo server | when people change, and daily |

**`Quillo_Server` is for administrators only.** The backups hold every password hash, every chat and the server's
key, so the server limits the folder to Administrators, SYSTEM and its own account when it creates it (the
Dashboard warns if it could not). Don't give artists access to it, or to `Quillo server data\Chat backup`.
If the central folder can't be reached, that hour's backup goes to `<data>\backups` on the server PC, the
Dashboard says so, and the next hour tries the central folder again.

**Servers set up with 1.12 or earlier** also kept an open copy of the database and the key in
`Quillo server data` (`messenger.db`, `tls\`, …). It is no longer updated. The Dashboard points to *Settings →
Backups → **Move data to this PC***: it makes a backup in `Quillo_Server\Backups` first, then removes the open copy,
moves the old daily backups next to the new ones, and limits the chat backup to administrators.

**A new server PC** (the old one broke, or went back): install the server and choose the same shared-files folder.
Setup finds the newest backup and offers to **restore everything** (run setup as an administrator who can open
`Quillo_Server`). The PCs keep trusting the server (same certificate) and **find it at its new address by
themselves**: when the old address stops answering, they ask the network, and follow a server only when it has the
same certificate. PCs on another subnet (a typed IP, no broadcast) need the new address typed once.
Folder: console → *Settings → Central folder*; the Dashboard shows the last backup and the user list.

## 2. First-time setup

Open **Quillo Server console** from the Start menu. It opens the console for the running service; the console
can also connect to a server on another PC (admin / IT login).

1. Sign in as **admin / admin** on the server PC itself (from any other PC, admin/admin is refused until its
   password has been changed). You must choose a new password straight away.
2. **Designations**: rename or add titles to match your studio (see below).
3. **Departments**: add your departments, and sections inside them (select a department → *Add section*).
   Tick **Chat room** for each department or section that should have its own room (see below).
4. **Users → Excel template**: an Excel file that already lists everyone, with dropdowns for department,
   section, designation and *reports to*, and columns for employee ID, birthday and joining date. Fill it in
   and use **Users → Import** (a CSV works too). You see "X new, Y changed" and any problems before anything
   is saved. New departments and sections are created; an empty cell keeps the current value, `-` clears it;
   new people without a password get a random one - the console saves a list of first passwords for you to
   hand out, and they choose their own at first sign-in. Or add people one by one with **Add user**.
5. **Settings**: the central folder, password rules, file size and clean-up, pipeline API, and **Studio file
   servers** (links in chats to these computers open with one click; links to any other computer ask first).
6. Use **Rooms → New room** for projects.
6a. **Holidays**: India's public and festival holidays are listed up to 2035. Tick the days your studio is
   closed (national days are ticked already). Festival dates follow the lunar calendar - the ones marked
   *check the date* can be a day off from your almanac, so check them each year. Add your own days, import
   an `.ics` file, or add India's list for a later year. HR (designations that can manage accounts) can do
   the same from the calendar in the app (*New → Studio holidays*).
7. **Org chart** shows who reports to whom; people without a lead appear under *Not in a reporting line*.
   When someone's reports come from two or more departments, each department gets its own branch under
   that person (a small department label with its people below). The List view's *By reporting line*
   does the same.

## 3. Designations and permissions

Every person has a **department**, a **section** (e.g. Compositing → Roto), a **designation** and a **Reports to**.
Designations carry the permissions (console → *Designations*). Since 1.6.2 every server also has a full VFX
list - Studio Head, VFX Supervisor, VFX Producer, DFX / CG / Compositing / FX / Lighting / Animation / Roto-Paint /
Matchmove / DMP supervisors, Line Producer, Production Manager and Coordinator, discipline leads, Compositor, FX
Artist, Lighting TD, Animator, Modeler, Rigger, Matte Painter, Matchmove / Roto / Paint / Prep artists, Pipeline TD,
Render Wrangler, Data I/O and more - with sensible permissions. Rename, change or delete any of them; deleted ones
do not come back. The original defaults:

| Designation | Announce to | Create rooms | Reset passwords / disable accounts | Always visible |
|---|---|---|---|---|
| Management | everyone | ✓ | | ✓ |
| HOD, Supervisor | own department (+ its sections) | ✓ | | HOD ✓ |
| HR, IT | everyone | ✓ | ✓ | ✓ |
| Production | everyone | ✓ | | ✓ |
| Lead | own section | ✓ | | |
| Senior Artist, Artist | — | ✓ | | |
| Junior Artist, Trainee | — | | | |

Anyone with people reporting to them can also announce to **My team**. A designation can be limited to
"see only own department".

**Announcements pop up and need an OK** (News → *New announcement*). Each one appears on top of everything, with
its title, text, sender and time, and stays until the person clicks **OK** (closing the window or Esc does not
count). People who are offline get it when they next sign in; every announcement also stays on the News page.
Since 1.14:
- **Urgent** (a tick in the composer): shown in red with *URGENT*, and it rings with the buzz sound.
- **Chosen people or rooms** (administrators only): *Send to → Chosen people or rooms…*, then tick people, or
  *Add a room* to tick everyone in it. Only those people get it.
- **Who clicked OK, and when:** the sender and the administrators see *OK'd by 3 of 11* under it on the News page
  (and in the console, *Announcements → Who clicked OK*), with the time of each OK. Someone who only opened it on
  the News page is listed as *opened it, no OK yet*. A PC still on an older Quillo has no OK button of its own:
  its *Got it* counts as OK.
- **A closed Quillo opens for it:** an announcement from an administrator (or the console) opens Quillo on PCs where
  it was closed but Quillo Helper is running (see section 4), and it pops up there at once. The audit log says how
  many PCs were opened. Announcements from other designations wait until the person opens Quillo.

**Departments page.** You create the departments and sections; on the Users page each person is then
picked from that list (no free typing). Nothing gets a room by itself: tick **Chat room** next to a department
or section and its room appears, with members kept in step with people's department/section. Unticking it (or
deleting the department) keeps the room and its history as a normal room — delete it on the Rooms page if it
is not needed; ticking again continues the same room. Renaming a department or section renames it for
everyone in it and renames its room. A department or section can only be deleted when nobody is in it
(deleting a department also deletes its sections). The "All Studio" room is still a switch in Settings.
When upgrading, the departments and sections people already had are added to the list (without chat rooms),
and the old automatic rooms stay as normal rooms; ticking Chat room picks the old room up again.

## 4. Install the clients

Run **`Quillo-Client-Setup-x.y.z.exe`** on every PC. It asks *Install for me only* (no administrator
rights needed; goes to `%LOCALAPPDATA%\Programs`) or *for all users* (administrator). Leave the *Server address*
page empty and the client finds the server by itself; enter the server's IP only for PCs on a different subnet /
VLAN. Add `/ALLUSERS` or `/CURRENTUSER` to a silent install to skip the question.

**Quillo starts with Windows** (the *Start Quillo when Windows starts* task is ticked by default): it opens in the
tray when the person signs in to Windows. If the server isn't up yet (both PCs just switched on), it keeps trying
every 15 s and signs in by itself as soon as the server answers (for people who ticked *Keep me signed in*). People can
switch this off in *Settings*; removing an *all users* install's entry needs an administrator.

**Closing the window keeps Quillo running** (since 1.14): the X, Alt+F4 and the taskbar's *Close window* only hide it
to the tray, so messages and buzzes still arrive. *Quit* is in the tray icon's right-click menu.

**Quillo Helper - an administrator's buzz reopens a closed Quillo** (since 1.14). The client setup also installs
`QuilloHelper.exe` next to `LANMessenger.exe` (task *Quillo Helper*, ticked by default). It is a tiny background
program with no window and no Qt (about 25 MB of memory, no CPU while waiting). It starts at Windows sign-in (the
`QuilloHelper` value under `...\CurrentVersion\Run`, like Quillo's own entry) and whenever Quillo starts. In Task
Manager it is **Quillo Helper** under *Background processes*.
- **What it does:** it signs in to the server with the password Quillo saved for that Windows user (only when *Keep me
  signed in* is ticked), over the same encryption and the same server-fingerprint check as Quillo. It never shows
  chats, never listens on a port (it only connects out to TCP 5150) and never asks a question.
- **An administrator buzzes someone whose Quillo is closed** (quit from the tray, ended in Task Manager or crashed,
  but the PC is on and they are signed in to Windows): the helper opens Quillo, which shakes and rings as usual and
  says *"Quillo reopened because <name> buzzed you"*. The administrator sees *"⚡ Buzzed ... - their Quillo was
  closed, it is opening now"*. **Only administrators' buzzes do this**; anyone else's buzz just waits in the chat.
  Every reopen is written to the audit log (*reopened Quillo with a buzz*). Direct buzzes only, not room buzzes.
  An administrator's announcement opens a closed Quillo in the same way (section 3).
- **Administrators see it coming:** such a person shows *"Quillo closed - buzz will reopen it"* instead of *last
  seen* in the chat header. Others still see a normal offline. People who chose *Invisible* always look offline.
- **After a crash** (or *End task*), the helper starts Quillo again in the tray, at most 3 times an hour. A normal
  *Quit* from the tray is respected.
- **It steps back by itself:** a wrong or changed password, a disabled account or a password reset stop it until Quillo
  saves a new sign-in (it never retries a refused password, so it can't lock anyone out); a server older than 1.14
  is asked again only every 10 minutes. Its notes go to `%LOCALAPPDATA%\LANMessenger\helper.log`.
- **To leave it out:** untick *Quillo Helper* in the setup, or `/MERGETASKS="!helper"` in a silent install. The
  uninstaller stops it and removes it and its startup entry.

**The first connection is checked.** A PC that has never talked to the server shows the server's *fingerprint* and
asks before it sends any password (another PC on the network could answer pretending to be the server). The person
compares it with the **Fingerprint** on the console Dashboard - or IT gives it to the setup with
`/FINGERPRINT=AB:CD:...`, and then the first connection is checked against it without a question (a different
server is refused with a security warning).

Silent install (GPO, PDQ Deploy, login script…):

```
Quillo-Client-Setup-1.6.0.exe /VERYSILENT /SUPPRESSMSGBOXES /NORESTART /ALLUSERS /SERVER=192.168.1.10
Quillo-Client-Setup-1.6.0.exe /VERYSILENT /SUPPRESSMSGBOXES /NORESTART /ALLUSERS /FINGERPRINT=AB:CD:...
Quillo-Client-Setup-1.6.0.exe /VERYSILENT /SUPPRESSMSGBOXES /NORESTART /ALLUSERS /MERGETASKS="autostart,!desktopicon"
```

Per-user settings live in `%APPDATA%\LANMessenger\`, the photo and preview cache in `%LOCALAPPDATA%\LANMessenger\`,
downloads in `Downloads\Quillo` (changeable in Settings; earlier installs keep `Downloads\LAN Messenger`).

**Portable use** without installing: copy a `build\dist\...` folder anywhere. A portable server keeps its data in
`server_data\` next to the exe if that folder exists (`firewall_setup.bat`, run as administrator, opens the ports).
For a preconfigured portable client, rename `client_config.example.json` to `client_config.json` and put the server
IP in it.

## 5. Updating

1. **Back up first**: console → **Settings → Backups → Back up now**.
2. **Server first:** run the new server setup over the old one. Accounts, messages, files and settings are kept;
   the database updates itself on start.
   On the server PC you can also use console → **Updates → Update this server...** and pick the new
   `Quillo-Server-Setup-x.y.z.exe`: the console closes, the setup runs and people reconnect by themselves.
3. **Then the clients:** console → **Updates → Publish client update...** and pick the new
   `Quillo-Client-Setup-x.y.z.exe` (or copy it into the server's `updates` folder yourself). Signed-in PCs are
   told straight away and show "version x.y.z is available". PCs where Quillo was installed *just for me* update
   with one click and **no administrator**; PCs installed *for all users* ask for an administrator (or run the
   setup with your deployment tool). People can also press *Settings → Check for updates*.
   The **Updates** page lists which versions the signed-in PCs run, so you can see who is behind; *Online now*
   shows it per person.

## 6. Running the server: service, data, backups

Service control (administrator command prompt, in the program folder; also `start`, `stop`, `restart`):

```
powershell -ExecutionPolicy Bypass -File service.ps1 status
```

Data folder (default `C:\ProgramData\LAN Messenger Server`; Start menu → *Server data folder* opens it):

| Item | What |
|---|---|
| `messenger.db` | accounts, messages, rooms (SQLite) |
| `files\` | shared files |
| `avatars\` | profile photos |
| `backups\` | the hourly database backups when there is no central folder (or it could not be reached) |
| `updates\` | client setups offered to the PCs |
| `tls\` | the server certificate. **Keep it**; a new one makes every PC show "server identity changed" |
| `config.json`, `server.log` | settings and log |

**Restore a backup:** console → *Settings → Backups → **Restore a backup…***: pick the time to go back to. The
server checks the backup, restarts with it (the PCs reconnect by themselves) and keeps the database it replaced as
`messenger.db.before-restore-<time>`, so a restore can be undone. Every restore is in the audit log. By hand: stop
the service, copy a `messenger_YYYY-MM-DD_HHMM.db` over `messenger.db`, **delete `messenger.db-wal` and
`messenger.db-shm`** next to it if they exist (otherwise the restore is undone), start it again.

If the server hits an unexpected error in the console window, it logs it to `server.log`, shows a short message and
keeps running. If it can't even read its settings, the note is in `%TEMP%\LANMessengerServer-startup.log`.
A hard crash (Python itself stops) leaves its trace in `server_crash.log` next to `server.log`; on a PC it is
`client_crash.log` next to `%LOCALAPPDATA%\LANMessenger\client.log`. Send these with a problem report.

The console also has **Reports** (activity per day, department, person and room, storage per user; CSV export),
the **Audit log** of every administrative action, **Online now** and the **Server log**.

## 6a. Chat backup, message history and shared files

Console → **Settings**:

| Setting | Default | What it does |
|---|---|---|
| *Write a readable chat backup every night* | on | At the backup time, new messages are appended to text files: a folder per room and per pair of people, one file per month - `Chat backup\Rooms\Falcon Comp\2026-09.txt`, `Chat backup\People\Alice Mathew + Bob Fernandes\2026-09.txt`. A renamed room keeps its folder. Replies in a thread are marked *[in the thread of ...]*. Grep-able, open in Notepad. Servers upgraded from 1.8 keep their old month folders as `Chat backup\Before 1.9.0`. |
| *Keep messages in the app for* | Forever | Messages stay in the app for good. If you choose a number of days, older messages leave the live database — **only after** they are in the chat backup. Servers upgraded from 1.8 (which used 90 days) are switched to *Forever*. |
| *Delete shared files after* | 3 days (new installs) | Files are removed from the server's file storage; the file card tells people "available until …". Point *File storage folder* at a separate (temp) drive if you like. Upgraded servers keep their old value — change it here. |
| *Delete files nobody downloaded after* | never | Extra clean-up for forgotten uploads. |
| *Allow Buzz* | on | Lets people buzz one person (shake + ring, even on Do not disturb). One buzz per 20 s per person; users can opt out in their Settings. |

**Per room:** Console → **Rooms** → *Keep files...* lets one room keep its shared files longer or shorter than the
setting above — e.g. *Forever* for a plates room, *30 days* for dailies. Direct chats always follow the server
setting. A file posted in several chats stays as long as the longest of them keeps it.

**Storage page:** Console → **Storage** shows how much space the shared files use, the free space on that disk,
and the space per person, per chat and the largest files. *Clean up now* deletes every file older than the number
of days you choose, straight away (the messages stay; the files show as expired). Every clean-up is in the audit
log.

Reminders and scheduled messages are kept on the server, so they fire (and scheduled messages are sent) even when
the person's PC is off; a scheduled message is sent as that person and fails with a visible reason if they are no
longer allowed to post there (e.g. they left the room).

*Back up chats now* writes the text backup immediately. Deleted messages written before they were deleted stay in the
text backup (it is an archive).

## 6b. Shot names, and settings that follow people

- **Shot names in chats become links** (1.10.0): `FAL_030` in a message links to everything said about that shot.
  The pattern is a regular expression in the console: *Settings → Chats → Shot names*. The default catches
  `FAL_030`, `FAL_030_0010` and `SEQ010_SH0020`; type a sample message in the box below it to see what would
  become a link. *Default* puts the standard pattern back; empty switches shot links off. PCs pick up a new
  pattern the next time they sign in.
- **Personal settings are kept on the server**, so they follow people to any PC (useful when PCs are rented or
  swapped): chats pinned to the top, focus time and whether the welcome tour was seen. They are in the database,
  so they are in the hourly backups too. The text size and the width of the chat list stay with each PC
  (they depend on its monitor).
- **UI size and Animations** (1.13.0, client *Settings*) stay with each PC, like the text size. *UI size* (90 /
  100 / 110 / 125 %) applies at the next start, on top of Windows' own scale. Quillo ignores the `QT_SCALE_FACTOR`,
  `QT_FONT_DPI` and similar settings that Nuke, Houdini, Maya or Katana set for the whole PC, so it is the same
  size on every PC; programs started from Quillo (Nuke opening a `.nk`) still get those settings. *Animations*
  (on by default) works straight away; Quillo also stays still when Windows' *Show animations* is off.

## 6c. The update tracker, leave replies (1.11.0)

- **Updates → people list**: everyone, the Quillo version they last signed in with and from which PC.
  *Only who still needs the update* narrows it; **Remind them** shows the update bar again on every signed-in PC
  that is behind. Someone who hasn't signed in since the server was updated shows as "not signed in yet".
- **Reports → Not seen in 30 days**: people who haven't signed in for a month (disable their account if they
  have left - Users → Enable / Disable).
- **Shot status** is the same for the whole studio and every change is in the Audit log ("shot status").
- **Leave auto-reply**: people write it themselves when they add leave; it answers direct messages once a day
  per person and never counts as unread.

## 7. Pipeline API (render farm, scripts)

Console → Settings → *Pipeline API*: tick *Allow*, click *New key*, save, restart the server. The API is
**HTTPS** with the server's own certificate, so the key can't be read on the network. Copy `server.crt` from the
`tls` folder in the server's data folder to the render nodes (only the certificate - **never `server.key`**), and
address the server by its computer name (the certificate is made for it). Then from any machine (Windows or Linux
render nodes):

```
curl --cacert server.crt -X POST https://SERVERPC:5152/api/message -H "Authorization: Bearer KEY" -H "Content-Type: application/json" -d "{\"to\": \"#Comp Team\", \"text\": \"FAL_020 v014 rendered\"}"
```

`to` is a username (`alice`) or a room (`#Comp Team`). `POST /api/announce` takes `title`, `text` and an optional
`department`; `GET /api/health` checks the server. Python (Deadline, Nuke, Houdini…), no extra packages:

```python
import json, ssl, urllib.request
req = urllib.request.Request("https://SERVERPC:5152/api/message",
    data=json.dumps({"to": "alice", "text": "Your render finished"}).encode(),
    headers={"Authorization": "Bearer KEY", "Content-Type": "application/json"})
urllib.request.urlopen(req, timeout=10, context=ssl.create_default_context(cafile="server.crt"))
```

Messages come from the account "Pipeline Bot" (renamable in Settings). Servers that already used the API before
this version keep plain HTTP, so existing scripts keep working; the switch is *Allow unencrypted HTTP* in the same
settings. Switch it off once your scripts use HTTPS - over plain HTTP anyone on the network can read the key.

## 8. Network ports

| Port | What |
|---|---|
| TCP 5150 | chat, files, screen sharing, console, Quillo Helper (TLS encrypted) |
| UDP 5151 | automatic server discovery |
| TCP 5152 | pipeline API (only when enabled) |

The server setup allows the server program through Windows Firewall, which covers all three.

**A port is already in use?**
- **TCP (chat) port busy** → the server does not start and says which program holds the port
  (e.g. *"used by nginx.exe (PID 4312)"*). Close that program or pick another port in Settings. Clients that find the
  server automatically follow the new port; PCs with a typed address need `IP:port`.
- **UDP (discovery) port busy** → the server runs, but the Dashboard shows *"Automatic discovery is OFF"* with the
  program holding the port. Clients can still connect by typing the server address.
- **Pipeline API port busy** → the server runs without the API; the reason is in the Server log.

## 9. Security and privacy

- **Encrypted connections (TLS).** The server makes its own certificate; each PC remembers the server's fingerprint
  (shown on the console Dashboard) and refuses to send a password to a different machine pretending to be the server.
- **Passwords:** new and changed passwords need at least 8 characters (a shorter password someone already has keeps
  working until they change it). The built-in admin/admin must be changed, and only works on the server PC itself
  until it is. Stricter rules are switches in Settings → Passwords: minimum length, letters + digits, refuse easy
  passwords, make new people choose their own password at first sign-in, expiry. Always on: a 60-second lock-out
  after 5 wrong passwords from one PC, and after 10 wrong passwords for one account from other PCs (the person's
  own PC and the server PC are never locked out by that). Remembered passwords are encrypted with Windows DPAPI.
- **Display names:** people can't take a colleague's name or username, or "Administrator", "Admin", "IT", "HR" or
  the pipeline bot's name. Screen-sharing notices show the username next to the name, and the audit log records
  the username.
- **Audit log** of users, designations, rooms, settings, password resets and photo removals.
- **Password resets.** IT/HR (designations that can manage accounts) and administrators can reset someone's
  password and could then sign in as that person, so a reset is never silent: it is in the audit log, the reset
  password works only once (the person must choose a new one at sign-in), and the person is told who reset it and
  when - at sign-in and in their *My space*, where nobody can delete it.
- **Administrators can't open people's chats in the app** (chat review was removed in 1.9.0), but see *Password
  resets* above, and whoever can open the chat backup text files in the central folder can read them: keep that
  folder for administrators only.
- **Links to other computers.** A `\\computer\share` link to a machine that is not in *Settings → Studio file
  servers* asks before Quillo even looks at it (opening it lets that computer see the person's Windows sign-in).
  Programs, scripts and shortcuts are never started from a chat: Quillo shows them in their folder instead.
- **Screen sharing** needs the consent of the person whose screen is shown - except for administrators and the
  people above them in the reporting line (their lead, the lead's lead...), who see it straight away. The person
  is always told who is watching (name and username), a red bar with *Stop* stays visible, and a view without
  asking is in the audit log.
- IT/HR can remove an inappropriate profile photo: Users → right-click → *Remove profile photo*.
- **Read receipts are always on** (1.13.0, no setting). A sender sees ticks on their own messages: ✓ the server
  has it, grey ✓✓ it has reached the other person's Quillo, blue ✓✓ they had that chat open on screen. In rooms the
  ticks count everyone who was in the room when the message was sent, and the sender can right-click → *Read by…*
  to see who has read it, who has it and who doesn't yet, with times (only the sender has *Read by…*). The times are
  kept on the server for 90 days. Older Quillo versions keep working; their messages are counted as delivered when
  the person is signed in.

## 10. Limits

- Files: 20 GB each by default; the admin can set up to 1 TB (Settings → *Max file size*). The server refuses an
  upload it has no disk space for, and interrupted transfers resume.
- Text messages: 100,000 characters (≈ 2,500 lines of Nuke script); longer text is offered as a file.
- Profile photos: 256 × 256, up to 400 KB (resized automatically).
- One server per studio; load-tested with 500 people online at once: 500 signed in within 2 s, ~390,000 deliveries with none lost, typical delivery 3–40 ms, server ~460 MB of RAM. The chat work runs on one CPU core, so around 500 *very* busy users
  delivery slows to about a second in the busiest rooms; nothing is lost.
- Screen sharing is view-only. Windows 10 / 11 only.
