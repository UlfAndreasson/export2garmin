# Export 2 to Garmin Connect

Export health data from supported devices to Garmin Connect.

This project was originally created by [Robert Wójtowicz](https://github.com/RobertWojtowicz/export2garmin).
The current version also includes verified support for the Omron HEM-7196T1 / X4 Connect AFib blood pressure monitor on macOS.

## 1. Introduction

### 1.1. Miscale module (once known as miscale2garmin)

* Allows fully automatic synchronization of Mi Body Composition Scale 2 (tested on XMTZC05HM) or Xiaomi Body Composition Scale S400 (tested on MJTZC01YM) directly to Garmin Connect, with the following parameters:

  * Date and Time (measurement, from device);
  * Weight (***NOTE:***** lbs is automatically converted to kg**, applies to Mi Body Composition Scale 2);
  * BMI (Body Mass Index);
  * Body Fat;
  * Skeletal Muscle Mass;
  * Bone Mass;
  * Body Water;
  * Physique Rating;
  * Visceral Fat;
  * Metabolic Age;
  * Heart rate (Xiaomi Body Composition Scale S400 only, optional upload to blood pressure section).

* `miscale_backup.csv` also contains other parameters which can be imported into Excel or other analysis tools:

  * BMR (Basal Metabolic Rate);
  * LBM (Lean Body Mass);
  * Ideal Weight;
  * Fat Mass To Ideal;
  * Protein;
  * Data Status (`to_import`, `failed`, `uploaded`);
  * Unix Time (based on Date and Time);
  * Email User (used Garmin Connect account);
  * Upload Date and Upload Time;
  * Difference Time (between measuring and uploading);
  * Battery status in V and % (ESP32 - Mi Body Composition Scale 2 only);
  * Impedance;
  * Impedance Low (Xiaomi Body Composition Scale S400 only).

* Supports multiple users with individual weight ranges and Garmin Connect accounts.

### 1.2. Omron module

The Omron module supports direct export of compatible Omron blood pressure monitors to Garmin Connect.

Tested devices include:

* Omron M4 / HEM-7155T;
* Omron M7 / HEM-7322T Intelli IT;
* Omron HEM-7196T / X4 Connect AFib.

The module exports:

* Date and Time (measurement time from device);
* DIAstolic blood pressure;
* SYStolic blood pressure;
* Heart rate.

`omron_backup.csv` also stores additional data which can be imported into Excel or other analysis tools:

* Category (***NOTE:***** EU and US classification only**);
* MOV (Movement detection);
* IHB (Irregular Heart Beat);
* Data Status (`to_import`, `failed`, `imported`);
* Unix Time;
* Email User;
* Upload Date and Upload Time;
* Difference Time between measurement and Garmin upload.

The Omron module supports two user profiles on compatible devices and can upload each profile to a separate Garmin Connect account.

### 1.3. Omron HEM-7196T1 / X4 Connect AFib support

The HEM-7196T1 / X4 Connect AFib uses Omron's modern Bluetooth Low Energy service (`FE4A`).

 tmp/omron_user1.csv tmp/omron_user2.csv   

The HEM-7196T1 workflow has been tested end-to-end with real data:

```text
Omron HEM-7196T1 / X4 Connect AFib
                ↓
Bluetooth Low Energy export
                ↓
User 1 and User 2 CSV separation
                ↓
Local Omron import history and duplicate protection
                ↓
Garmin Connect blood pressure upload
```

### 1.4. Garmin duplicate protection

The Garmin import uses the local file:

```text
user/omron_backup.csv
```

Each record has one of the following statuses:

```text
to_import  = waiting to be uploaded to Garmin Connect
imported   = successfully uploaded to Garmin Connect
failed     = upload failed and can be retried
```

After a successful Garmin upload, the record is marked as `imported`.

This prevents the same Omron measurement from being uploaded again when the import is run multiple times.

### 1.5. Optional daily Omron upload filtering

By default, all Omron measurements are uploaded to Garmin:

```text
omron_daily_filter=all
```

Optional values:

```text
all             = upload all readings
latest          = upload the last raw reading of the day
avg_all_raw     = upload the average of all raw readings that day
first_truread   = upload the average of the first 3-reading session within 15 minutes
latest_truread  = upload the average of the latest 3-reading session within 15 minutes
```

The TruRead-related filters infer a session from three consecutive measurements within 15 minutes.

If no three-reading session is found, the script falls back to the average of all raw readings for that day.

### 1.6. User module

* Enables configuration of all parameters related to Miscale and Omron integrations;
* Provides export of OAuth1 and OAuth2 tokens from Garmin Connect;
* Supports Garmin Connect MFA / 2FA.

---

## 2. How does this work

* Miscale and Omron modules can be activated individually or run together:

  * Devices can run together (Mi Body Composition Scale 2 and Omron);
  * Devices can run together but there must be a sequence, first measuring blood pressure and then weighing (Xiaomi Body Composition Scale S400 and Omron);
    This is because Xiaomi Body Composition Scale S400 requires continuous scanning. Importing data from the scale allows the workflow to proceed to the Omron module.
  * Devices can run together in parallel, but two USB Bluetooth adapters are required: one for Xiaomi Body Composition Scale S400 and one for Omron.
    This uses separate processes. See section [2.6.4.](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/about_BLE.md#264-using-two-ble-adapters-in-parallel).

* Synchronization diagram from Export 2 to Garmin Connect:

### 2.1. Miscale module | Mi Body Composition Scale 2 | BLE VERSION

* After weighing, Mi Body Composition Scale 2 is active for 15 minutes on Bluetooth transmission;
* A USB Bluetooth adapter or internal module scans BLE devices for 10 seconds to acquire data from the scale;
* Body weight and impedance data are processed by scripts;
* Processed data are sent to Garmin Connect;
* Raw and calculated data from the scale are backed up in `miscale_backup.csv`;
* This part of the project is **no longer being developed**, but it still works.

**Select your platform and go to instructions:**

* [Debian 13 | Raspberry Pi OS (based on Debian 13)](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/Miscale_BLE.md);
* [Windows 11](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/all_BLE_win.md).

### 2.2. Miscale module | Mi Body Composition Scale 2 | ESP32 VERSION

* After weighing, Mi Body Composition Scale 2 is active for 15 minutes on Bluetooth transmission;
* ESP32 operates in deep sleep and wakes every 7 minutes to scan BLE devices for 10 seconds;
* The process can be started immediately through the reset button;
* ESP32 sends acquired data through MQTT to an MQTT broker installed on the server;
* Body weight and impedance data are processed by scripts;
* Processed data are sent to Garmin Connect;
* Raw and calculated data from the scale are backed up in `miscale_backup.csv`;
* This part of the project is **no longer being developed**, but it still works.

**Select your platform and go to instructions:**

* [Debian 13 | Raspberry Pi OS (based on Debian 13)](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/Miscale_ESP32.md);
* [Windows 11](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/Miscale_ESP32_win.md).

### 2.3. Miscale module | Xiaomi Body Composition Scale S400 | BLE VERSION

* After weighing, Xiaomi Body Composition Scale S400 transmits weight data for a short period using Bluetooth;
* A USB Bluetooth adapter scans BLE devices continuously to acquire scale data;
* Data from the scale are decrypted and parsed into a readable format;
* Body weight and impedance data are processed by scripts;
* Processed data are sent to Garmin Connect;
* Raw and calculated data from the scale are backed up in `miscale_backup.csv`.

**Select your platform and go to instructions:**

* [Debian 13 | Raspberry Pi OS (based on Debian 13)](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/S400_BLE.md);
* [Windows 11](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/all_BLE_win.md).

### 2.4. Omron module | BLE VERSION

* After measuring blood pressure, compatible Omron devices allow stored measurement data to be downloaded;
* A USB Bluetooth adapter or internal Bluetooth module scans BLE devices and retrieves data from the blood pressure monitor;
* Downloading a complete history can take approximately one minute;
* Blood pressure measurements are processed by scripts;
* Processed data are sent to Garmin Connect;
* Raw and calculated device data are backed up in `omron_backup.csv`.

**Existing Linux and Windows instructions:**

* [Debian 13 | Raspberry Pi OS (based on Debian 13)](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/Omron_BLE.md);
* [Windows 11](https://github.com/RobertWojtowicz/export2garmin/blob/master/manuals/all_BLE_win.md).

### 2.5. Omron HEM-7196T1 / X4 Connect AFib on macOS

The HEM-7196T1 workflow has been verified on macOS using direct Python commands.

The existing `import_data.sh` is primarily Linux-oriented and may depend on:

```text
bluepy
bluetoothctl
GNU/Linux sed behavior
Linux shell utilities
```

For macOS, use the installation and workflow below.

---

## 3. Installation on macOS for HEM-7196T1 / X4 Connect AFib

### 3.1. Install Python

Install Python 3.10 or newer.

Check that Python is available:

```bash
python3 --version
```

### 3.2. Clone the repository

```bash
mkdir -p ~/omron
cd ~/omron

git clone https://github.com/UlfAndreasson/export2garmin.git
cd export2garmin
```

### 3.3. Create and activate a virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

The shell prompt should now include:

```text
(.venv)
```

### 3.4. Install Python dependencies

```bash
python -m pip install --upgrade pip
python -m pip install bleak terminaltables garminconnect
```

Verify installation:

```bash
python -c "from garminconnect import Garmin; print('garminconnect OK')"
python -c "import bleak, terminaltables; print('BLE dependencies OK')"
```

When the virtual environment is active, use:

```bash
python
```

rather than `python3`, as `python3` may point to another Python installation outside the virtual environment.

### 3.5. Create local directories

```bash
mkdir -p tmp _ulf
```

### 3.6. Exclude local health data, logs and credentials from Git

```bash
cat >> .git/info/exclude <<'EOF'
_ulf/
tmp/
*.log
*.csv
test
EOF
```

---

## 4. Garmin configuration

Garmin authentication tokens are stored locally under:

```text
user/<your-garmin-email>/
```

Example:

```text
user/your-garmin-email@example.com/garmin_tokens.json
```

Do not commit Garmin token files.

The local configuration file is:

```text
user/export2garmin.cfg
```

A basic Omron configuration can look like this:

```text
switch_temp_path=tmp/
switch_omron=on

omron_export_user1=your-garmin-email@example.com
omron_export_user2=unused@example.com

omron_export_category=eu
omron_daily_filter=all
```

To keep the local configuration file out of ordinary Git changes:

```bash
git update-index --skip-worktree user/export2garmin.cfg
```

---

## 5. Find the Omron BLE device identifier on macOS

On macOS, the device is identified by a CoreBluetooth UUID rather than a normal Bluetooth MAC address.

Example:

```text
F174C009-27FC-090A-D7BB-E2D2B7BDD732
```

The identifier can be different on another Mac.

Useful Bluetooth troubleshooting steps:

* Close the Omron Connect application on the phone;
* Temporarily disable Bluetooth on the phone;
* Keep the blood pressure monitor close to the Mac;
* Do not use the `-p` pairing flag on macOS. CoreBluetooth does not support explicit pairing through Bleak in the same way as Linux.

---

## 6. Daily macOS workflow for User 1

This workflow exports User 1 measurements and uploads them to the configured Garmin Connect account.

### 6.1. Activate the environment

```bash
cd ~/omron/export2garmin
source .venv/bin/activate
```

### 6.2. Read measurements from the Omron monitor

Replace the UUID with the identifier for your Mac.

```bash
find tmp -name 'omron_user*.csv' -delete

python omron/omblepy.py \
  --loggerDebug \
  -d hem-7196t1 \
  -a hci0 \
  -m YOUR-OMRON-BLE-UUID \
  > "_ulf/omron_pull_$(date +%Y%m%d_%H%M%S).log" 2>&1
```

Check the exported User 1 measurements:

```bash
tail -10 tmp/omron_user1.csv
```

User 2 measurements are exported separately:

```bash
cat tmp/omron_user2.csv
```

Do not upload User 2 data to the same Garmin account unless that is intentional.

### 6.3. Create the local Omron import-history file

Run this once if `user/omron_backup.csv` does not already exist:

```bash
if [[ ! -f user/omron_backup.csv ]]; then
  cat > user/omron_backup.csv <<'EOF'
Data Status;Unix Time;Date [dd.mm.yyyy];Time [hh:mm];SYStolic [mmHg];DIAstolic [mmHg];Heart Rate [bpm];MOV;IHB;Email User;Upload Date [dd.mm.yyyy];Upload Time [hh:mm];Difference Time [s]
EOF
fi
```

### 6.4. Add only new User 1 measurements

Replace the email address with the Garmin account used for User 1:

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

The Unix timestamp is used as the duplicate key. Measurements already present in `omron_backup.csv` are not added again.

Show the number of new pending measurements:

```bash
grep -c '^to_import;' user/omron_backup.csv
```

### 6.5. Upload pending records to Garmin Connect

```bash
python -B omron/omron_export.py
```

After successful upload, records change from:

```text
to_import
```

to:

```text
imported
```

A second run skips records that were already uploaded.

---

## 7. Minimal daily command sequence

After initial setup, the normal HEM-7196T1 workflow is:

```bash
cd ~/omron/export2garmin
source .venv/bin/activate

find tmp -name 'omron_user*.csv' -delete

python omron/omblepy.py \
  --loggerDebug \
  -d hem-7196t1 \
  -a hci0 \
  -m YOUR-OMRON-BLE-UUID \
  > "_ulf/omron_pull_$(date +%Y%m%d_%H%M%S).log" 2>&1

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

## 8. Verify Omron import status

Show records waiting for import:

```bash
grep -c '^to_import;' user/omron_backup.csv
```

Show successfully imported records:

```bash
grep -c '^imported;' user/omron_backup.csv
```

Show recently processed records:

```bash
tail -20 user/omron_backup.csv
```

---

## 9. Mobile App

I do not plan to create a mobile app, but I encourage you to use other projects for Mi Body Composition Scale / Mi Scale / S400:

* Android: https://github.com/lswiderski/mi-scale-exporter;
* iOS | iPadOS: https://github.com/lswiderski/WebBodyComposition.

---

## 10. Synchronizing data between different ecosystems

A useful project called SmartScaleConnect synchronizes scale data from Xiaomi Cloud to Garmin Connect for Mi Body Composition Scale 2 / S400:

* Linux | macOS | Windows: https://github.com/AlexxIT/SmartScaleConnect.

---

## 11. Privacy and local data

This project handles personal health data and Garmin authentication tokens.

Do not commit or share:

```text
user/export2garmin.cfg
user/omron_backup.csv
user/<email>/garmin_tokens.json
tmp/
_ulf/
```

---

## 12. License

This project remains licensed under the original project license.

See LICENSE for details.
