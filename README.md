# Tutorial Cek Informasi OS, Build OS, RAM, dan CPU Windows Menggunakan CMD

Dokumen ini digunakan sebagai panduan bagi pegawai untuk mengambil informasi perangkat Windows melalui **Command Prompt (CMD)**, lalu memasukkan hasilnya ke **Google Form**.

> **Link Google Form:** `ISI_LINK_GOOGLE_FORM_DI_SINI`
>
> Catatan untuk admin: ganti teks `ISI_LINK_GOOGLE_FORM_DI_SINI` dengan link Google Form yang sebenarnya sebelum panduan ini dibagikan ke pegawai.

---

## 1. Informasi yang Akan Diambil

Pegawai akan mengambil informasi berikut dari komputer/laptop Windows:

- Nama OS, contoh: `Microsoft Windows 11 Pro`
- Display Version, contoh: `25H2`
- Build OS lengkap sampai angka terakhir, contoh: `26200.8485`
- RAM, contoh: `16,127 MB`
- CPU sampai informasi GHz, contoh: `Intel(R) Core(TM) i7-12700H @ 2.30 GHz`

Contoh hasil akhir:

```text
OS: Microsoft Windows 11 Pro | Display Version: 25H2 | Version: 26200.8485 | RAM: 16,127 MB | CPU: Intel(R) Core(TM) i7-12700H @ 2.30 GHz
```

---

## 2. Cara Membuka Command Prompt

1. Tekan tombol **Windows** pada keyboard.
2. Ketik:

   ```text
   cmd
   ```

3. Klik **Command Prompt**.
4. Tidak wajib menggunakan **Run as administrator**. CMD biasa sudah cukup.

---

## 3. Aktifkan Mode CMD yang Dibutuhkan

Di jendela Command Prompt, ketik perintah berikut lalu tekan **Enter**:

```cmd
cmd /v:on
```

Setelah itu, Command Prompt akan tetap terbuka. Lanjutkan ke langkah berikutnya.

---

## 4. Jalankan Perintah Cek Spesifikasi

Salin semua perintah di bawah ini, lalu paste ke Command Prompt dan tekan **Enter**.

```cmd
powershell -NoProfile -Command "$cv=Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion'; $os=Get-CimInstance Win32_OperatingSystem; $cpu=Get-CimInstance Win32_Processor | Select-Object -First 1; 'OS: {0} | Display Version: {1} | Version: 10.0.{2}.{3} | RAM: {4} GB | CPU: {5} @ {6} GHz' -f $os.Caption,$cv.DisplayVersion,$cv.CurrentBuildNumber,$cv.UBR,[math]::Round($os.TotalVisibleMemorySize/1MB,2),$cpu.Name,[math]::Round($cpu.MaxClockSpeed/1000,2)"
```

---

## 5. Salin Hasil Output

Setelah perintah dijalankan, akan muncul hasil seperti ini:

```text
OS: Microsoft Windows 11 Pro | Display Version: 25H2 | Version: 26200.8485 | RAM: 16,127 MB | CPU: Intel(R) Core(TM) i7-12700H @ 2.30 GHz
```

Blok teks hasil tersebut, lalu salin dengan salah satu cara berikut:

1. Blok teks hasil output menggunakan mouse.
2. Tekan **Ctrl + C**.
3. Simpan sementara di Notepad jika diperlukan.

Yang perlu disalin hanya baris hasil akhirnya, yaitu baris yang diawali dengan:

```text
OS:
```

---

## 6. Masukkan Data ke Google Form

1. Buka link Google Form berikut:

   ```text
   ISI_LINK_GOOGLE_FORM_DI_SINI
   ```

2. Isi identitas sesuai kolom yang tersedia, misalnya:

   - Nama Pegawai
   - Unit Kerja
   - Nomor HP atau email, jika diminta
   - Nama perangkat, jika diminta

3. Pada kolom informasi perangkat atau spesifikasi perangkat, paste hasil dari CMD.

   Contoh:

   ```text
   OS: Microsoft Windows 11 Pro | Display Version: 25H2 | Version: 26200.8485 | RAM: 16,127 MB | CPU: Intel(R) Core(TM) i7-12700H @ 2.30 GHz
   ```

4. Periksa kembali data yang sudah dimasukkan.
5. Klik **Submit** atau **Kirim**.

---

## 7. Alternatif: Membuat File `.bat`

Jika pegawai kesulitan menyalin banyak baris perintah, admin dapat membuat file batch agar pegawai cukup menjalankan satu file.

### 7.1 Buat File BAT

1. Buka **Notepad**.
2. Salin isi script berikut:

```bat
@echo off
setlocal EnableDelayedExpansion

for /f "tokens=2 delims=:" %%A in ('systeminfo ^| findstr /B /C:"OS Name" /C:"Nama OS"') do set "os=%%A"
set "os=!os:~1!"

for /f "tokens=3" %%A in ('reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v DisplayVersion 2^>nul ^| findstr /i "DisplayVersion"') do set "display=%%A"
if not defined display for /f "tokens=3" %%A in ('reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v ReleaseId 2^>nul ^| findstr /i "ReleaseId"') do set "display=%%A"

for /f "tokens=3" %%A in ('reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v CurrentBuildNumber ^| findstr /i "CurrentBuildNumber"') do set "build=%%A"
for /f "tokens=3" %%A in ('reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v UBR ^| findstr /i "UBR"') do set "ubr=%%A"
set /a "ubr=!ubr!"

for /f "tokens=2 delims=:" %%A in ('systeminfo ^| findstr /I /C:"Total Physical Memory" /C:"Memori Fisik Total"') do set "ram=%%A"
set "ram=!ram:~1!"

for /f "tokens=2,*" %%A in ('reg query "HKLM\HARDWARE\DESCRIPTION\System\CentralProcessor\0" /v ProcessorNameString ^| findstr /i "ProcessorNameString"') do set "cpu=%%B"
for /f "tokens=3" %%A in ('reg query "HKLM\HARDWARE\DESCRIPTION\System\CentralProcessor\0" /v "~MHz" ^| findstr /i "MHz"') do set "mhz=%%A"
set /a "mhz=!mhz!"
set /a "ghz=!mhz!/1000"
set /a "dec=(!mhz!%%1000)/10"
set "dec=00!dec!"
set "dec=!dec:~-2!"

echo(!cpu! | findstr /i "GHz" >nul && set "cpufinal=!cpu!" || set "cpufinal=!cpu! @ !ghz!.!dec! GHz"

echo.
echo Salin hasil di bawah ini ke Google Form:
echo ------------------------------------------------------------
echo OS: !os! ^| Display Version: !display! ^| Version: !build!.!ubr! ^| RAM: !ram! ^| CPU: !cpufinal!
echo ------------------------------------------------------------
echo.
pause
```

3. Klik **File > Save As**.
4. Pada bagian **Save as type**, pilih **All Files**.
5. Simpan dengan nama:

   ```text
   cek-spesifikasi-windows.bat
   ```

6. Jalankan file tersebut dengan double-click.
7. Salin hasil yang muncul, lalu masukkan ke Google Form.

---

## 8. Kendala yang Mungkin Muncul

### 8.1 `wmic is not recognized`

Tidak masalah. Tutorial ini **tidak menggunakan WMIC**, karena WMIC sudah tidak tersedia di beberapa versi Windows terbaru.

### 8.2 `Missing operand`

Biasanya terjadi jika perintah untuk CMD langsung dan file `.bat` tertukar.

Perbedaannya:

```text
CMD langsung : %1000
File .bat    : %%1000
```

Gunakan perintah sesuai bagian tutorial yang dipakai.

### 8.3 Display Version kosong

Jika bagian `Display Version` kosong, lanjutkan pengisian Form dengan hasil yang muncul. Build OS tetap bisa digunakan sebagai informasi utama.

Contoh:

```text
OS: Microsoft Windows 11 Pro | Display Version:  | Version: 26200.8485 | RAM: 16,127 MB | CPU: Intel(R) Core(TM) i7-12700H @ 2.30 GHz
```

### 8.4 RAM tampil dalam MB, bukan GB

Tidak masalah. Masukkan sesuai hasil yang muncul di CMD.

Contoh:

```text
RAM: 16,127 MB
```

---

## 9. Catatan Keamanan

Perintah ini hanya mengambil informasi umum perangkat, yaitu OS, versi OS, RAM, dan CPU.

Perintah ini **tidak mengambil**:

- Password
- File pribadi
- Data browser
- Nomor serial perangkat
- IP address
- Informasi akun pribadi

---

## 10. Format Jawaban yang Benar di Google Form

Gunakan format satu baris seperti berikut:

```text
OS: [Nama OS] | Display Version: [Versi Display] | Version: [Build.UBR] | RAM: [Total RAM] | CPU: [Nama CPU @ GHz]
```

Contoh:

```text
OS: Microsoft Windows 11 Pro | Display Version: 25H2 | Version: 26200.8485 | RAM: 16,127 MB | CPU: Intel(R) Core(TM) i7-12700H @ 2.30 GHz
```
