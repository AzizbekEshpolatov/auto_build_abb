Bu loyiha Flutter dasturchilari uchun Android ilovani `.aab` formatda build qilish jarayonini avtomatlashtirish uchun yozilgan. Har safar `versionCode` ni qo‘lda yangilab, keyin `.aab` faylni build qilib, so‘ngra Play Console’ga yuklash vaqtingizni oladi. Endi esa siz terminalda faqat `deploy` deb yozish orqali bu ishlarning barchasini avtomatik bajarasiz: versiya avtomatik +1 bo‘ladi, ilova build qilinadi va tayyor `.aab` fayl joylashuvi ko‘rsatiladi. Bu PowerShell script orqali amalga oshadi. Skript `android/local.properties` faylidan `flutter.versionCode` qiymatini topadi, uni +1 qiladi va faylni yangilaydi. So‘ng `gradlew.bat bundleRelease` buyrug‘i orqali build qilinadi. Siz faqat `deploy` deb yozasiz — qolgan ishlarni skript hal qiladi. Buni ishlatish uchun faqat 1 marta sozlash kifoya: PowerShell profiling faylingizga `deploy` funksiyasini yozasiz. Shu bilan har safar `deploy` komandasini ishlatishingiz mumkin bo‘ladi. Quyidagi bosqichlarni bajaring:

1. `deploy.ps1` faylini loyihangizning asosiy papkasiga joylashtiring (masalan, `D:\manzara-mobile`)

2. PowerShell terminalida quyidagilarni yozing:  
   `notepad $PROFILE`  
   ochilgan faylga shuni yozing:
```
function deploy {
   powershell -ExecutionPolicy Bypass -File "$PWD\deploy.ps1"
}
```

3. Faylni saqlang va terminalni qayta oching.

4. Yangi fayl yarating: ``deploy.ps1`` (bu Windows PowerShell skript fayli).
5. Ichiga quyidagi kodni yozing (bu ishlab turgan, testdan o‘tgan versiya):
```
$localPropsPath = "android\local.properties"
$lines = Get-Content $localPropsPath

$currentVersionCodeLine = $lines | Where-Object { $_ -match "flutter.versionCode" }
$currentVersionCode = ($currentVersionCodeLine -split "=")[1].Trim()
$newVersionCode = [int]$currentVersionCode + 1

$newLines = @()
foreach ($line in $lines) {
    if ($line -match "flutter.versionCode") {
        $newLines += "flutter.versionCode=$newVersionCode"
    } else {
        $newLines += $line
    }
}

$newLines | Set-Content $localPropsPath

Push-Location android
cmd /c "gradlew.bat bundleRelease"
Pop-Location

Write-Host "flutter.versionCode $newVersionCode ga oshirildi!"
Write-Host @"
.aab fayl: android\app\build\outputs\bundle\release\app-release.aab
"@
```


6. Faylni saqlang. PowerShell terminalni yoki Android Studio’ni qayta ishga tushiring.

Endi siz terminalda `deploy` deb yozsangiz, quyidagi ishlar avtomatik bo‘ladi:
- `flutter.versionCode` avtomatik oshadi
- `local.properties` faylga yoziladi
- `gradlew.bat` orqali `.aab` fayl build qilinadi
- build fayl manzili ko‘rsatiladi

.aab fayl: `android/app/build/outputs/bundle/release/app-release.aab`

---

Bu usul sizga har safar qo‘lda versiya o‘zgartirish va build yozishdan qutqaradi. Siz vaqt tejaysiz, xatolik ehtimoli kamayadi. Keyinchalik bu tizimga `fastlane` qo‘shib, Play Store’ga avtomatik yuklashni ham amalga oshirishingiz mumkin, kengaytirib chiqsa boladi bemalol.


