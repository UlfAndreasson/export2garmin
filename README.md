# Export 2 to Garmin Connect

Export health data from supported devices to Garmin Connect.

This project was originally created by [Robert Wójtowicz](https://github.com/RobertWojtowicz/export2garmin).

This fork adds and verifies an end-to-end macOS workflow for the **Omron HEM-7196T1 / X4 Connect AFib** blood pressure monitor.

## 1. Introduction

### 1.1 Miscale module

The Miscale module supports automatic synchronization of compatible Xiaomi/Mi body composition scales to Garmin Connect.

Supported/tested devices include:

* Mi Body Composition Scale 2 (`XMTZC05HM`);
* Xiaomi Body Composition Scale S400 (`MJTZC01YM`).

The module can export:

* Date and time;
* Weight;
* BMI;
* Body fat;
* Skeletal muscle mass;
* Bone mass;
* Body water;
* Physique rating;
* Visceral fat;
* Metabolic age;
* Heart rate for Xiaomi Body Composition Scale S400.

`miscale_backup.csv` may also contain additional values useful for analysis in Excel or other tools:

* BMR;
* Lean body mass;
* Ideal weight;
* Fat mass to ideal;
* Protein;
* Data status;
* Unix time;
* Garmin account;
* Upload date and time;
* Difference between measurement time and upload time;
* Battery level;
* Impedance.

### 1.2 Omron module

The Omron module exports stored blood pressure measurements from compatible Omron devices and uploads them to Garmin Connect.

Previously tested devices include:

* Omron M4 / HEM-7155T;
* Omron M7 / HEM-7322T Intelli IT.

This fork also adds verified support for:

* **Omron HEM-7196T1**;
* **Omron X4 Connect AFib**.

The Omron export contains:

* Measurement date and time;
* Systolic blood pressure;
* Diastolic blood pressure;
* Heart rate;
* Movement detection flag;
* Irregular heartbeat flag;
* User profile.

The local Omron import history is stored in:

```text
user/omron_backup.csv
```

### 1.3 HEM-7196T1 / X4 Connect AFib support

The HEM-7196T1 uses Omron’s modern Bluetooth Low Energy service:

```text
FE4A
```

The device differs from some older Omron models because both User 1 and User 2 measurements are stored in a shared measurement bank.

The verified record layout for this device includes:

```text
Record start address: 0x01C4
Record size:          16 bytes
User identifier:      embedded in each record
```

The driver reads the shared record bank and separates records into:

```text
tmp/omron_user1.csv
tmp/omron_user2.csv
```

The HEM-7196T1 workflow has been tested with real device data and verified all the way through Garmin Connect:

```text
Omron HEM-7196T1 / X4 Connect AFib
                ↓
Bluetooth Low Energy export
                ↓
CSV separation for User 1 and User 2
                ↓
Local import history and duplicate protection
                ↓
Garmin Connect blood pressure upload
```

### 1.4 Garmin duplicate protection

Garmin Connect accepts multiple blood pressure readings on the same day.

This fork therefore defaults to uploading all Omron readings unless filtering is explicitly enabled.

Each row in `user/omron_backup.csv` has a status:

```text
to_import  = waiting to be uploaded to Garmin Connect
imported   = successfully uploaded to Garmin Connect
failed     = upload failed and can be retried
```

After a successful Garmin upload, the script changes the row status to:

```text
imported
```

This prevents duplicate Garmin uploads when the import command is run again.

Duplicate protection has two layers:

```text
1. New Omron rows are compared against existing Unix timestamps in omron_backup.csv.
2. Already uploaded rows are marked imported and skipped on later Garmin imports.
```

### 1.5 Optional daily Omron upload filtering

Default behaviour is:

```text
omron_daily_filter=all
```

This preserves all raw Omron measurements in Garmin Connect.

Available values are:

```text
all             = upload all readings
latest          = upload the last raw reading of the day
avg_all_raw     = upload the average of all raw readings that day
first_truread   = upload the average of the first 3-reading session within 15 minutes
latest_truread  = upload the average of the latest 3-reading session within 15 minutes
```

The TruRead-related filters infer a TruRead-like session from three consecutive measurements within 15 minutes.

When no three-measurement session is found, the script falls back to the average of all raw readings from that day.

### 1.6 User module

The User module manages local integration settings and Garmin authentication.

It supports:

* Miscale configuration;
* Omron configuration;
* Garmin Connect OAuth token storage;
* Garmin Connect MFA / 2FA.

---

## 2. How the project works

The supported device modules can be run individually or together.

```text
Supported examples:
- Omron blood pressure monitor only;
- Xiaomi/Mi scale only;
- Omron plus a supported scale;
- Multiple Garmin accounts for multiple user profiles.
```

General workflow:

```text
Device measurement
        ↓
Bluetooth Low Energy readout
        ↓
CSV export
        ↓
Local history and duplicate protection
        ↓
Garmin Connect upload
```

Original project workflow diagram:

![Export 2 to Garmin Connect workflow](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/workflow.png)

### 2.1 Miscale module | Mi Body Composition Scale 2 | BLE

* The scale becomes available over Bluetooth after weighing;
* A Bluetooth adapter scans for the scale;
* Weight and impedance are processed;
* Processed values are uploaded to Garmin Connect;
* Raw and calculated data are retained in `miscale_backup.csv`.

Instructions:

* [Debian 13 / Raspberry Pi OS](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/Miscale_BLE.md)
* [Windows 11](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/all_BLE_win.md)

### 2.2 Miscale module | Mi Body Composition Scale 2 | ESP32

* ESP32 wakes periodically and scans for the scale;
* Data can be sent through MQTT;
* Data are processed and uploaded to Garmin Connect;
* Raw values are retained in `miscale_backup.csv`.

Instructions:

* [Debian 13 / Raspberry Pi OS](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/Miscale_ESP32.md)
* [Windows 11](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/Miscale_ESP32_win.md)

### 2.3 Miscale module | Xiaomi Body Composition Scale S400 | BLE

* The scale transmits briefly after a measurement;
* Bluetooth data are scanned, decrypted and parsed;
* Values are uploaded to Garmin Connect;
* Raw and calculated data are retained in `miscale_backup.csv`.

Instructions:

* [Debian 13 / Raspberry Pi OS](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/S400_BLE.md)
* [Windows 11](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/all_BLE_win.md)

### 2.4 Omron module | BLE

* Omron measurements are stored in the monitor;
* The monitor is contacted through Bluetooth Low Energy;
* Measurement data are read and written to CSV;
* New measurements are added to `user/omron_backup.csv`;
* Pending records are uploaded to Garmin Connect;
* Successful uploads are marked as `imported`.

Existing Linux and Windows instructions:

* [Debian 13 / Raspberry Pi OS](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/Omron_BLE.md)
* [Windows 11](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/all_BLE_win.md)

---

## 3. macOS support for Omron HEM-7196T1 / X4 Connect AFib

The HEM-7196T1 implementation in this fork is verified on macOS with direct Python commands.

The existing `import_data.sh` is primarily Linux-oriented and may depend on components such as:

```text
bluepy
bluetoothctl
GNU/Linux sed behaviour
Linux shell utilities
```

For macOS, use the workflow documented below.

---

## 4. Installation on a new Mac

### 4.1 Install Python

Install Python 3.10 or newer.

Check the installed version:

```bash
python3 --version
```

### 4.2 Clone this fork

```bash
mkdir -p ~/omron
cd ~/omron

git clone https://github.com/UlfAndreasson/export2garmin.git
cd export2garmin
```

### 4.3 Create a Python virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

The terminal prompt should now contain:

```text
(.venv)
```

### 4.4 Install dependencies

```bash
python -m pip install --upgrade pip
python -m pip install bleak terminaltables garminconnect
```

Verify the key dependencies:

```bash
python -c "from garminconnect import Garmin; print('garminconnect OK')"
python -c "import bleak, terminaltables; print('BLE dependencies OK')"
```

Important: when the virtual environment is active, use:

```bash
python
```

rather than `python3`.

On some Macs, `python3` may refer to a Homebrew or system Python installation outside the active virtual environment.

### 4.5 Create working directories

```bash
mkdir -p tmp _ulf
```

### 4.6 Keep health data and credentials out of Git

```bash
cat >> .git/info/exclude <<'EOF'
_ulf/
tmp/
*.log
*.csv
test
EOF
```

This keeps local health exports, logs and credential files out of normal Git status output.

---

## 5. Garmin setup

Garmin authentication data are stored locally under:

```text
user/<your-garmin-email>/
```

Example:

```text
user/your-garmin-email@example.com/garmin_tokens.json
```

Do not commit or share Garmin authentication files.

On a new Mac, the simplest option is to securely copy the Garmin token directory from the existing working Mac:

```text
user/your-garmin-email@example.com/
```

Place it in the same location in the new clone.

The local user configuration file is:

```text
user/export2garmin.cfg
```

A minimal Omron configuration looks like this:

```text
switch_temp_path=tmp/
switch_omron=on

omron_export_user1=your-garmin-email@example.com
omron_export_user2=unused@example.com

omron_export_category=eu
omron_daily_filter=all
```

Keep the configuration local:

```bash
git update-index --skip-worktree user/export2garmin.cfg
```

---

## 6. Omron Bluetooth setup on macOS

macOS uses a CoreBluetooth device UUID rather than a normal Bluetooth MAC address.

Example:

```text
F174C009-27FC-090A-D7BB-E2D2B7BDD732
```

The identifier can be different on another Mac.

### 6.1 Important Bluetooth rules

Before connecting from the Mac:

* Close the Omron Connect app on the iPhone;
* Temporarily disable Bluetooth on the iPhone if the phone may reconnect to the monitor;
* Keep the blood pressure monitor close to the Mac;
* Do not use `-p` on macOS. Explicit pairing is not supported by Bleak/CoreBluetooth in the same way as Linux.

### 6.2 Fix “Peer removed pairing information”

macOS may occasionally show:

```text
Peer removed pairing information
```

When that happens:

1. Open **System Settings → Bluetooth**.
2. Find **X4 Connect AFib**.
3. Choose **Forget This Device**.
4. Press the Bluetooth / transfer button on the Omron monitor to put it into communication mode.
5. Run the Omron export command again.

A first connection attempt may occasionally fail. Once the monitor is in transfer mode, retry the command before changing any code or configuration.

---

## 7. First-time Omron import setup

Create the local import-history file once, if it does not exist:

```bash
if [[ ! -f user/omron_backup.csv ]]; then
  cat > user/omron_backup.csv <<'EOF'
Data Status;Unix Time;Date [dd.mm.yyyy];Time [hh:mm];SYStolic [mmHg];DIAstolic [mmHg];Heart Rate [bpm];MOV;IHB;Email User;Upload Date [dd.mm.yyyy];Upload Time [hh:mm];Difference Time [s]
EOF
fi
```

This file is the local source of truth for imported Omron measurements.

---

## 8. Daily macOS workflow for User 1

The commands below export **User 1** from the Omron monitor and upload only previously unseen records to Garmin Connect.

User 2 is exported separately but is not uploaded unless explicitly configured for another Garmin account.

### 8.1 Open the project and activate the environment

```bash
cd ~/omron/export2garmin
source .venv/bin/activate
```

### 8.2 Check the active Garmin upload mode

```bash
grep -n '^omron_daily_filter=' user/export2garmin.cfg
```

Default output should be:

```text
omron_daily_filter=all
```

### 8.3 Read measurements from the Omron monitor

Before running the command:

* Ensure Omron Connect is closed on the phone;
* Ensure the monitor is close to the Mac;
* Press the Bluetooth / transfer button on the monitor when needed.

Run:

```bash
python omron/omblepy.py \
  --loggerDebug \
  -d hem-7196t1 \
  -a hci0 \
  -m YOUR-OMRON-BLE-UUID
```

Example:

```bash
python omron/omblepy.py \
  --loggerDebug \
  -d hem-7196t1 \
  -a hci0 \
  -m F174C009-27FC-090A-D7BB-E2D2B7BDD732
```

A successful run ends with lines similar to:

```text
communication finished
writing data to omron_user1.csv
writing data to omron_user2.csv
```

Verify that User 1 data exists:

```bash
ls -lh tmp/omron_user*.csv
tail -20 tmp/omron_user1.csv
```

Do not delete the previous `tmp/omron_user*.csv` files before confirming that a new BLE read has succeeded. They provide a useful fallback if a Bluetooth connection attempt fails.

### 8.4 Backup the current import history

```bash
mkdir -p _ulf

cp user/omron_backup.csv \
  "_ulf/omron_backup_before_import_$(date +%Y%m%d_%H%M%S).csv"
```

### 8.5 Add only new User 1 measurements

Replace the Garmin email address with the User 1 Garmin account.

```bash
awk -F ';' -v email="your-garmin-email@example.com" '
NR==FNR { seen[$2]=1; next }
FNR==1 { next }
!seen[$2] {
  split($3, dt, " ")
  print $1 ";" $2 ";" dt[1] ";" dt[2] ";" $4 ";" $5 ";" $6 ";" $7 ";" $8 ";" email ";;;"
}
' user/omron_backup.csv tmp/omron_user1.csv >> user/omron_backup.csv
```

This uses Unix time as the duplicate key.

Measurements already present in `user/omron_backup.csv` are not added again.

### 8.6 Review pending Garmin uploads

```bash
echo "Pending User 1 readings:"
awk -F ';' '
$1 == "to_import" {
  printf "%s %s  %s/%s  pulse %s\n", $3, $4, $5, $6, $7
}
' user/omron_backup.csv

echo
echo "Number of pending readings:"
grep -c '^to_import;' user/omron_backup.csv
```

Review this list before importing.

### 8.7 Upload pending measurements to Garmin Connect

```bash
python -B omron/omron_export.py
```

Every successful row should produce:

```text
OMRON * Upload status: OK
```

### 8.8 Confirm that the import completed

```bash
echo "Still pending:"
grep -c '^to_import;' user/omron_backup.csv

echo
echo "Latest imported records:"
tail -20 user/omron_backup.csv
```

Expected result:

```text
Still pending:
0
```

A later import run will skip all rows already marked:

```text
imported
```

---

## 9. Minimal daily command sequence

After initial setup, this is the normal workflow:

```bash
cd ~/omron/export2garmin
source .venv/bin/activate

python omron/omblepy.py \
  --loggerDebug \
  -d hem-7196t1 \
  -a hci0 \
  -m YOUR-OMRON-BLE-UUID

awk -F ';' -v email="your-garmin-email@example.com" '
NR==FNR { seen[$2]=1; next }
FNR==1 { next }
!seen[$2] {
  split($3, dt, " ")
  print $1 ";" $2 ";" dt[1] ";" dt[2] ";" $4 ";" $5 ";" $6 ";" $7 ";" $8 ";" email ";;;"
}
' user/omron_backup.csv tmp/omron_user1.csv >> user/omron_backup.csv

python -B omron/omron_export.py
```

---

## 10. Verify data by day before Garmin import

This command provides a useful per-day overview of the User 1 Omron data:

```bash
awk -F ';' '
NR > 1 {
  split($3, dt, " ")
  date = dt[1]
  count[date]++
  values[date] = values[date] "  " dt[2] " " $4 "/" $5 " p" $6
}
END {
  for (date in count)
    printf "%s: %d readings%s\n", date, count[date], values[date]
}
' tmp/omron_user1.csv | sort -t. -k3,3n -k2,2n -k1,1n
```

Example output:

```text
20.06.2026: 6 readings  07:01 123/83 p55  07:03 120/82 p55  07:04 123/79 p54  07:08 128/84 p55  07:09 127/84 p58  07:11 124/86 p55
21.06.2026: 3 readings  06:15 116/84 p68  06:16 113/79 p61  06:17 118/83 p63
```

---

## 11. Privacy and local data

This project handles sensitive health data and Garmin authentication data.

Do not commit or publicly share:

```text
user/export2garmin.cfg
user/omron_backup.csv
user/<email>/garmin_tokens.json
tmp/
_ulf/
```

The following files may contain personal blood pressure measurements:

```text
tmp/omron_user1.csv
tmp/omron_user2.csv
user/omron_backup.csv
```

---

## 12. Mobile apps

This project does not provide a mobile app.

Useful related projects for Xiaomi/Mi scales include:

* Android: [mi-scale-exporter](https://github.com/lswiderski/mi-scale-exporter)
* iOS / iPadOS: [WebBodyComposition](https://github.com/lswiderski/WebBodyComposition)

---

## 13. Synchronizing data between ecosystems

For Xiaomi/Mi body composition scales, SmartScaleConnect can synchronize Xiaomi Cloud scale data with Garmin Connect:

* [SmartScaleConnect](https://github.com/AlexxIT/SmartScaleConnect) for Linux, macOS and Windows.

---

## 14. License

This project remains licensed under the original project license.

See [LICENSE](LICENSE) for details.
