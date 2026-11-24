# PadTV 0.5 / PadBridge 0.2 — ověření

Vydání je experimentální. Následující výsledky rozlišují provedené testy od
neimplementovaných částí; instalace APK není důkaz funkčnosti jejího rozhraní.
Testováno 24. listopadu 2026.

## Automatické testy

Celkem **145 různých testů**: všech původních 131 zůstalo zachováno a přibylo 14.

| Sada | Výsledek |
|---|---|
| PadTV Python / Qt | 113/113, Windows i Debian 13 (stejné testy počítané jednou) |
| PadBridge Rust | 7/7 |
| Původní podepsané APK / izolace / oprávnění | 14/14 v Linux KVM |
| Nové Java/Kotlin, split APK, framework a média | 11/11 v Linux KVM |

Nezávislé sestavení a testy PadBridge prošly také v
[GitHub CI](https://github.com/PadTVos/PadBridge/actions/runs/45379421975).

Nové APK jsou sestavené z Java/Kotlin zdrojů proti Android API, přeložené D8 do
skutečného DEX a podepsané. Testovaný běh používá původní APK, Enjarify, JVM a
povinný bubblewrap. Nejde o spuštění předem připraveného JAR místo APK.

Ověřeno: DEX 038/039 v nových Java/Kotlin APK, multidex, stejné podpisy všech
splitů, byte-for-byte uchování APK, kontrola integrity před spuštěním, odmítnutí
neúplných/duplicitních sad a nebezpečných ZIP cest, nedostupný ART bez skrytého
fallbacku, XML layout a D-pad callback, reflexe, výjimky, HandlerThread,
SharedPreferences přes opakované spuštění, skutečná SQLite transakce/rollback,
lokální Binder, služby, broadcast a privátní ContentProvider.

Mediální modul skutečně dekódoval snímky FFV1, H.264, H.265/HEVC, VP9 a AV1.
To ověřuje softwarové dekódování malých testovacích videí; **neověřuje 4K,
VA-API, zvuk, synchronizaci ani Android MediaCodec/MediaPlayer**.

## Grafické vykonávání

Ve Wayland relaci na KVM se vykreslilo rozhraní vytvořené původním Java i Kotlin
DEX. Skutečné QEMU klávesové události změnily popisek tlačítka přes APK callback.
Java případ používá kompilovaný Android XML layout a vnořený horizontální
LinearLayout. Jde o důkaz tohoto subsetu, nikoli kompatibility všech Java/Kotlin
nebo AndroidX aplikací.

- [Java XML + vstup](screenshots/padbridge-05-java-dpad.png)
- [Kotlin + vstup](screenshots/padbridge-05-kotlin-dpad.png)
- [PadTV ve Windows/WHPX](screenshots/padtv-05-windows-boot.png)
- [Samostatný vývojový náhled ve Windows](screenshots/padtv-05-windows-preview.png)
- [Store a stav kompatibility](screenshots/padtv-05-compatibility.png)

## Skutečné aplikace třetích stran

| APK z upstream GitHub Releases | Instalace | Spuštění / GUI | Přesný první blokátor po obecných opravách |
|---|---|---|---|
| Launch on Boot 1.1.1 | Ano, podpis ověřen | Ne, před vytvořením Activity | `android.view.View$OnFocusChangeListener` |
| TV Bro 3.0.7, generic/geckoExcluded | Ano, podpis ověřen | Ne, před vytvořením Activity | `android.view.Window$Callback` |
| YouTube pro Android TV | Neověřeno | Neověřeno | APK z oprávněného zdroje nebylo dostupné |
| Netflix pro Android TV | Neověřeno | Neověřeno | APK z oprávněného zdroje nebylo dostupné |

Při rozboru opravena obecná práce s přibalenými Android Support/SDK třídami,
klasifikace tříd definovaných v APK a doplněny LayoutInflater factories a
CompoundButton/CheckBox. Další stovky referencí na Android API zůstávají nepokryté;
odstranění jednoho blokátoru neznamená funkční aplikaci. Závislosti GMS/DRM nelze
z neúspěšného startu potvrdit. Kompletní seznamy API, podpisy, SHA-256, zdrojové
URL, nativní knihovny a výpisy chyb obsahuje
[databáze kompatibility](https://github.com/PadTVos/PadBridge/blob/main/compatibility/apps.json).
Žádné APK třetích stran není součástí GitHub repozitářů ani ISO.

## ISO a systém

`dist/padtv-0.5.0-amd64.iso`, 2 442 657 792 bajtů.

SHA-256: `bd7bf64c2c1cc0ca28ea032a92508505873f94afabb72307d255dbe7b7483af8`

Ověřeny oba vnitřní seznamy kontrolních součtů (782 MD5 a 782 SHA-256), Windows
kopie ISO, shoda instalovaných Python modulů, Rust binárky, JNI knihovny,
framework.jar a obou překladačových Python modulů se sestaveným zdrojem.
Zkontrolovány nabídky BIOS/UEFI, kernel a initramfs.

- BIOS: skutečný boot tohoto ISO v samostatné Windows QEMU/WHPX VM až do PadTV.
- UEFI: skutečný boot přes OVMF; bez přímého předání kernelu QEMU.
- Grafická instalace finálního ISO na nový 48GiB virtuální disk prošla:
  GPT, 512MiB EFI oddíl a ext4, Calamares zaznamenal `completion: succeeded`.
  [Dokončený instalátor](screenshots/padtv-05-install-success.png).
- Nainstalovaný systém naběhl přes UEFI přímo do PadTV bez zadávání hesla;
  ISO i původní Live datový disk byly odpojené.
  [Start z nainstalovaného disku](screenshots/padtv-05-installed-boot.png).
- Vývojový náhled Windows: testy Qt i skutečné vykreslení náhledu prošly;
  Linuxové APK v něm nejsou simulovány.
- Původní Windows VM, její QCOW2 a data nebyly nahrazeny. Konfigurace dalšího
  startu a zástupce směřují na 0.5. Běžící starší VM je nutné nejprve vypnout.
- Obsah soukromých předvoleb PadBridge na existující Live perzistenci se po
  změně ISO zachoval s totožným SHA-256.

## Co není hotové

ART, plný Android framework/WindowManager, vzdálený Binder IPC, Bionic/linker,
spouštění Android ELF, ARM64 překlad, kompletní AndroidX/Leanback,
RelativeLayout/ConstraintLayout/RecyclerView, SurfaceView/TextureView a Android
GLES/EGL nejsou funkční. Mediální Android API, PipeWire audio pro APK, VA-API,
chráněné přehrávání, Widevine, GMS a certifikace zařízení nejsou dokončené.

Přesný stav, technické překážky a další kroky jsou v
[popisu PadBridge 0.2](https://github.com/PadTVos/PadBridge/blob/main/docs/runtime-0.2.md)
a [rozboru ART](https://github.com/PadTVos/PadBridge/blob/main/docs/art-0.2-research.md).
Fyzické Wi-Fi/Bluetooth/CEC ovladače, konkrétní GPU, suspend/resume, HDMI audio
a 4K/DRM přehrávání vyžadují testy na skutečném zařízení. Stávající systémové
moduly PadTV a oddělený Waydroid zůstaly zachované, bez nového tvrzení o jejich
hardwarové kompatibilitě.
