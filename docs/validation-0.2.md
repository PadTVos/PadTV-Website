# Ověření PadTV 0.2 — 24. listopadu 2026

## Rozhraní a aplikace

- Windows / Python 3.10 / PyQt6 6.9.1: **35 automatických testů prošlo**.
- Debian 13 / Python 3.13 / distribuční PyQt6: **stejných 35 testů prošlo**
  proti nainstalovanému výslednému balíčku.
- Ověřena navigace, modalita, hranice ovládacích prvků v 720p, bezpečný náhled,
  chybové stavy procesů a zrušení napájení i odinstalování.
- YouTube je první uživatelská aplikace. Test ověřuje odstranění jejího manifestu
  i vlastního profilu, zachování ostatních dat a nepřidání po restartu.
- Skutečná cesta klávesami v Qt: Delete → dvě šipky → Enter odstranila YouTube;
  nově otevřené okno jej již neobsahovalo. Snímky dialogu a výsledku jsou uložené.
- Stejná operace v nabootovaném Linuxu odstranila i skutečně použitý profil
  Chromia. Po ukončení a opětovném spuštění procesu PadTV zůstal YouTube odebraný.
- Pozadí a logo jsou bajtově shodné s dodanými PNG. Domov, správa aplikací
  a stránka Androidu byly vizuálně zkontrolované.
- Zástupce na ploše Windows odkazuje na místní `.venv`, spouští bezpečný náhled
  a používá ikonu odvozenou z dodaného loga.

## ISO a Debian

- Hybridní ISO obsahuje BIOS i UEFI záznamy El Torito; automatický start za 5 sekund.
- Vnitřní manifest obsahuje **782 souborů**; kontrola obsahu sestaveného obrazu prošla.
- Nainstalované zdrojové moduly, root helper, Leanback XML a obrázky odpovídají
  zdrojům projektu. ShellCheck prošel pro sestavovací i relační skripty.
- QEMU/KVM, 4 GB RAM, 2 CPU, bez disku a sítě: nabootování do PadTV ověřeno
  pro BIOS a UEFI/OVMF bez Secure Boot.
- V samostatné síťové VM se YouTube otevřel v Chromiu až na stránku souhlasu
  se soubory cookie. Nebylo použito přihlášení a nebylo zkoušeno přehrávání videa.
  První spuštění v autologin živé relaci zobrazilo dialog nové klíčenky;
  zrušení tohoto dialogu umožnilo pokračovat bez přihlášení.

Balíčky ve výsledném systému:

| Součást | Verze |
| --- | --- |
| PadTV | 0.2.0 |
| Waydroid | 1.6.3+ds-2~bpo13+1 |
| Weston | 14.0.2-1 |
| aapt | 1:14~beta1-3 |
| Linux | 6.12.118+deb13-amd64 |

## Skutečná zkouška Androidu

QEMU/KVM, 16 GB RAM, 4 CPU, síť NAT, Debian kernel z ISO. Waydroid byl připraven
pomocí dodaného helperu; stáhl oficiální VANILLA systém
`lineage-20.0-20260927-VANILLA-waydroid_x86_64` a vendor obraz. Pro emulovanou
grafiku této VM bylo podle dokumentace Waydroidu nastaveno softwarové vykreslování
(`ro.hardware.gralloc=default`, `ro.hardware.egl=swiftshader`). Produkční profil
se do softwarového vykreslování automaticky nepřepíná.

- Weston v X11 relaci a Waydroid session se spustily přes dodané systemd jednotky.
- Oficiální **Jellyfin pro Android TV 0.20.0** se nainstaloval z APK s x86_64 knihovnami.
- Android potvrdil umístění `/data/app/.../org.jellyfin.androidtv.../base.apk`;
  nešlo o systémovou aplikaci.
- Spuštění zobrazilo skutečnou obrazovku Jellyfinu pro zadání serveru.
- Běžné potvrzení odinstalování v Androidu odstranilo APK. Následná nezávislá
  kontrola přes `pm list packages` a `pm path` již balíček nenalezla.

Zkouška odhalila a vedla k opravám tří věcí: předčasného načtení Binderu bez
všech zařízení, chybného považování opakovaných pokusů při startu za konečnou
chybu a použití seznamu aplikací před dokončením bootu Androidu. Při chybě přímé
odinstalace (`PendingIntent` v této verzi obrazu) se použije běžný dialog Androidu,
který je výslovně zobrazen ve Waydroid okně. Aplikace se označí za odebranou až
po ověření seznamu v naběhlém Androidu.

Opravy jsou ve zdrojích i novém Debian balíčku. Systémový souborový obraz v ISO
byl po těchto zkouškách znovu zabalen z opraveného balíčku; xorriso zachovalo
bootovací záznamy a byly přepočítány vnitřní MD5/SHA256 manifesty i vnější SHA256.

APK získané z [oficiálního vydání Jellyfinu](https://github.com/jellyfin/jellyfin-androidtv/releases/tag/v0.20.0)
mělo SHA256 odpovídající metadatům vydání:

```text
dfcc665195bed4ea087dca63df6f77497acc1185c4b15776b336d80eeaeb7fa6
```

## Výstupy

```text
dist/padtv-0.2.0-amd64.iso
dist/padtv-0.2.0-amd64.iso.sha256
dist/padtv-session_0.2.0_all.deb
artifacts/padtv-0.2-home.png
artifacts/padtv-0.2-manage.png
artifacts/padtv-0.2-android.png
artifacts/youtube-uninstall-dialog.png
artifacts/youtube-after-restart.png
artifacts/youtube-linux-removed.png
artifacts/boot-020-bios.png
artifacts/boot-020-uefi.png
artifacts/youtube-web.png
artifacts/jellyfin-android.png
artifacts/tests-020-debian.log
artifacts/android-final-test.log
artifacts/iso-020-integrity.log
```

SHA256 aktuálního ISO je v sousedním souboru `.iso.sha256`.
Sestavení používá oddělený Debian chroot `/var/tmp/padtv-build/debian` ve WSL Ubuntu.
Výstupy a pomocná testovací data v `dist/` a `artifacts/` jsou mimo Git.
Záznam původní verze je zachován v `docs/validation-0.1.md`.

## Virtuální PC na Windows

Dodatečně ověřeno spuštění celého ISO v nativním **QEMU 11.1.0 pro Windows**
s akcelerací **WHPX**, 16 GB RAM, 4 vCPU a grafickým oknem GTK. Síť NAT a výstup
DirectSound jsou připojené; samotné přehrávání zvuku tím ještě ověřené není.
Virtuální disk má kapacitu 64 GiB a při vytvoření zabíral přibližně 193 KiB.
PadTV naběhl až na domovskou obrazovku. Vznikl zástupce
`PadTV 0.2 - virtuální PC` na ploše a spouštěč brání souběžnému otevření dvou VM
nad týmž virtuálním diskem. Soubor disku, stav a logy jsou mimo Git.
Nejde o instalaci na tento disk: výchozí spuštění používá Live ISO bez perzistence.

## Meze ověření

**YouTube je předinstalovaná webová aplikace, nikoliv oficiální Android TV APK.**
To není součástí obrazu a nebylo zde zkoušeno. Google Play a Google služby nejsou
přibalené; PadTV není certifikované Google TV. Jellyfin se spustil, ale bez
uživatelova serveru nebylo ověřeno přihlášení, knihovna ani přehrávání. Android
obraz používá běžné prostředí Waydroidu s deklarací Leanback, ne kompletní TV ROM;
Jellyfin může zobrazit upozornění na typ zařízení.

Dále není ověřena instalace na disk, fyzický mini PC, HDMI zvuk, hardwarová
akcelerace videa, Wi-Fi, uspávání, Secure Boot, CEC/IR ani streamovací DRM.
Živé ISO bez perzistence po restartu zahodí všechny změny včetně odinstalování
předinstalovaných aplikací. Náhled Windows spouštění aplikací a Androidu simuluje.
