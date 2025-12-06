```
@echo off
title ADVANCED BACKUP SYSTEM
setlocal enabledelayedexpansion

REM ===========================================
REM KONFIGURASI EMAIL
REM ===========================================
set EMAIL_TO=informatikaclasa@gmail.com
set EMAIL_FROM=shifayulia32@gmail.com
set EMAIL_PASS=urqe tiaz xdtf yhmm

REM ===========================================
REM LOKASI BACKUP
REM ===========================================
set SRC=%USERPROFILE%\Desktop\SimulasiDriveD
set DST=%USERPROFILE%\Desktop\BackupResults
mkdir "%DST%" 2>nul


:MENU
cls
echo ==============================================
echo         ADVANCED BACKUP SYSTEM (UAS)
echo ==============================================
echo 1. Selective Backup
echo 2. Incremental Backup
echo 3. Scheduled Backup
echo 4. Exit
echo ==============================================
set /p pilih=Pilih menu (1-4): 

if "%pilih%"=="1" goto SELECTIVE
if "%pilih%"=="2" goto INCREMENTAL
if "%pilih%"=="3" goto SCHEDULE
if "%pilih%"=="4" exit

goto MENU


REM ==================================================
REM 1. SELECTIVE BACKUP
REM ==================================================
:SELECTIVE
cls
echo ===== SELECTIVE BACKUP =====
echo Folder yang tersedia:
dir "%SRC%" /b /ad
echo ==============================
set /p folder=Pilih folder yang ingin di-backup (* untuk semua): 

set OUT=%DST%\Selective_%date:~0,2%-%date:~3,2%-%date:~6,4%_%time:~0,2%-%time:~3,2%
mkdir "%OUT%" >nul

echo Melakukan backup...
xcopy "%SRC%\%folder%" "%OUT%\%folder%" /E /H /C /I /Y >nul

echo Backup selesai: %OUT%

REM === KIRIM EMAIL ===
powershell -Command ^
"Send-MailMessage -To '%EMAIL_TO%' -From '%EMAIL_FROM%' -Subject 'Selective Backup selesai' -Body 'Backup folder %folder% selesai. Lokasi: %OUT%' -SmtpServer 'smtp.gmail.com' -Port 587 -UseSsl -Credential (New-Object PSCredential('%EMAIL_FROM%', (ConvertTo-SecureString '%EMAIL_PASS%' -AsPlainText -Force)))"

echo Email notifikasi terkirim!
pause
goto MENU



REM ==================================================
REM 2. INCREMENTAL BACKUP
REM ==================================================
:INCREMENTAL
cls
echo ===== INCREMENTAL BACKUP =====

set SNAP=%DST%\snapshot.txt
set NEW=%DST%\snapshot_new.txt
set OUT=%DST%\Incremental_%date:~0,2%-%date:~3,2%-%date:~6,4%_%time:~0,2%-%time:~3,2%
mkdir "%OUT%" >nul

echo Membuat snapshot terbaru...
(for /r "%SRC%" %%a in (*.*) do echo %%~za "%%a") > "%NEW%"

if not exist "%SNAP%" (
    echo Snapshot pertama. Full backup...
    xcopy "%SRC%" "%OUT%" /E /H /C /I /Y >nul
    copy "%NEW%" "%SNAP%" >nul

    powershell -Command ^
    "Send-MailMessage -To '%EMAIL_TO%' -From '%EMAIL_FROM%' -Subject 'Full Incremental Backup Pertama' -Body 'Backup penuh pertama selesai.' -SmtpServer 'smtp.gmail.com' -Port 587 -UseSsl -Credential (New-Object PSCredential('%EMAIL_FROM%', (ConvertTo-SecureString '%EMAIL_PASS%' -AsPlainText -Force)))"
    
    echo Email terkirim.
    pause
    goto MENU
)

echo File yang berubah:
for /f "tokens=1,*" %%a in (%NEW%) do (
    find "%%b" "%SNAP%" | find "%%a" >nul
    if errorlevel 1 (
        echo Updated: %%b
        copy "%%b" "%OUT%" >nul
    )
)

copy "%NEW%" "%SNAP%" >nul

echo Backup incremental selesai: %OUT%

REM === EMAIL ===
powershell -Command ^
"Send-MailMessage -To '%EMAIL_TO%' -From '%EMAIL_FROM%' -Subject 'Incremental Backup selesai' -Body 'Incremental backup selesai. Output: %OUT%' -SmtpServer 'smtp.gmail.com' -Port 587 -UseSsl -Credential (New-Object PSCredential('%EMAIL_FROM%', (ConvertTo-SecureString '%EMAIL_PASS%' -AsPlainText -Force)))"

pause
goto MENU



REM ==================================================
REM 3. SCHEDULED BACKUP
REM ==================================================
:SCHEDULE
cls
echo ===== SCHEDULED BACKUP =====
set /p menit=Backup otomatis setiap berapa menit?: 

schtasks /create /tn "AutoBackup_UAS" /sc minute /mo %menit% /tr "%~dp0BackupSystem_Advanced.bat" /f

echo Scheduled backup berhasil dibuat!

powershell -Command ^
"Send-MailMessage -To '%EMAIL_TO%' -From '%EMAIL_FROM%' -Subject 'Scheduled Backup aktif' -Body 'Backup otomatis aktif setiap %menit% menit.' -SmtpServer 'smtp.gmail.com' -Port 587 -UseSsl -Credential (New-Object PSCredential('%EMAIL_FROM%', (ConvertTo-SecureString '%EMAIL_PASS%' -AsPlainText -Force)))"

pause
goto MENU

```
