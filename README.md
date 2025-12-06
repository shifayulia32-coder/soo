```
echo MEMULAI PROSES DATA RECOVERY...
echo ================================
 
set sumber=%USERPROFILE%\Desktop\simulasidriveD
set tujuan=%USERPROFILE%\Desktop\SimulasiDrive_C
 
:: Membuat folder backup dengan timestamp sederhana
set backup_folder=DataRecovery_%date:/=-%_%time::=-%
mkdir "%tujuan%\%backup_folder%"
 
echo Sumber  : %sumber%
echo Tujuan : %tujuan%\%backup_folder%
echo.
 
echo Proses menyalin data...
xcopy "%sumber%" "%tujuan%\%backup_folder%" /E /H /C /I /Y > "%tujuan%\%backup_folder%\recovery_log.txt"
 
echo ================================
echo RECOVERY SELESAI
echo Log tersimpan di recovery_log.txt
pause
```
