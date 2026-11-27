# PadTV 0.7 – Android Runtime & Multimedia

PadTV je distribuce pro amd64 založená na Debianu 13. Systémové balíčky mají verzi 0.7.0 a PadBridge 0.4.0. Základ System Core, Wayland, systémové API a instalátor zůstávají zachované. Stav ověření: [validation.md](https://github.com/PadTVos/PadTV-OS/blob/main/docs/validation.md).

## Moduly a hranice

| Balíček | Odpovědnost | Umístění |
| --- | --- | --- |
| `padtv-core` | identita distribuce, účet TV, systémové API, oprávnění, základ bootloaderu | `/usr/lib/padtv-core`, `/usr/lib/padtv`, `/var/lib/padtv` |
| `padtv-session` | Wayland relace, televizní rozhraní, nastavení, integrace aplikací, Plymouth | `/usr/share/padtv`, `/etc/padtv`, jednotky systemd |
| `padtv-installer` | grafická instalace Calamares a dokončení cílového systému | `/etc/calamares`, `/usr/lib/calamares/modules/padtv-finalize` |

Identita v `/etc/os-release` a `/usr/lib/os-release` patří PadTV. Debianí soubory
jsou zachované pomocí `dpkg-divert`; odstranění `padtv-core` je obnoví.
`ID=padtv`, `ID_LIKE=debian`, `VERSION_ID=0.7`, `DEBIAN_VERSION=13`.

Uživatelská knihovna a profily webových aplikací leží v
`~/.local/share/padtv/library`; nastavení v `~/.config/padtv`. YouTube je výchozí
záznam knihovny, nikoli systémový balíček. Jeho odstranění i odstranění dat je
stejné jako u jiné webové aplikace a při dalším startu se samo nevrací.
Originální APK a jejich soukromá data spravuje experimentální PadBridge.
Oddělený Waydroid spravuje pouze APK, která uživatel výslovně instaloval do něj; nejsou součástí systémového obrazu
Androidu. Waydroid používá `/var/lib/waydroid` a uživatelská data v domovském adresáři.

## Start a zotavení

1. BIOS/UEFI spustí kernel a initramfs; Plymouth zobrazí logo, český text a animaci.
2. `padtv-account.service` připraví vyhrazený účet `padtv`.
3. `padtv-system.service` poskytne API na systémové D-Bus sběrnici.
4. `padtv.service` spustí greetd a automatickou relaci na tty7.
5. `padtv-session` spustí labwc. Nepotřebuje LightDM, Openbox, Xorg ani desktopový panel.
6. Labwc předá prostředí uživatelskému systemd a spustí `padtv-session.target`.
   `padtv-shell.service` obnovuje televizní UI po pádu.

Aplikace spuštěné z UI mají samostatné systemd scopes, takže restart UI je
neukončuje. Pád kompozitoru znamená ztrátu jeho grafických klientů; služba obnoví
celou přihlašovací relaci. Před restartem uklidí pouze starou relaci PadTV na tty7.
Po skončení automatické relace greetd požádá o restart `padtv.service`, jejíž
nový běh znovu povolí automatické přihlášení. Nenabízí přihlašovací obrazovku.

PadTV, Thunar a instalátor používají Wayland. Chromium zůstává přes Xwayland,
protože jeho nativní identifikátor webového okna nerozlišuje oddělené profily.
PadTV tak nadále hledá správné okno podle konkrétního profilu, i po restartu UI.
Xwayland slouží kompatibilitě aplikací, není to klasické desktopové prostředí.
Waydroid má vlastní vnořený kompozitor Weston s odděleným socketem `padtv-android`.
VM standardně používá renderer pixman; fyzické zařízení volí hardwarový renderer
automaticky. Explicitní `WLR_RENDERER` může nastavení VM změnit.

## Systémové API v1

Sběrnice: systémová D-Bus. Jméno a rozhraní: `org.padtv.System1`.
Cesta objektu: `/org/padtv/System1`. Výsledkem metod je JSON řetězec.

| Metoda | Účel |
| --- | --- |
| `GetInfo()` | identita, kernel, Live/perzistence, virtualizace a dostupné napájení |
| `GetDevices()` | zařízení NetworkManageru a adaptéry BlueZ; absence služby je explicitní |
| `GetServices()` | stav povolených systémových služeb |
| `Power(action)` | `poweroff`, `reboot`, `suspend` přes systemd-logind |
| `ServiceAction(name, operation)` | `network`, `bluetooth`, `android`; pouze `start`, `stop`, `restart` |

Příkazové rozhraní:

```sh
padtv-systemctl info
padtv-systemctl devices
padtv-systemctl services
padtv-systemctl service bluetooth restart
padtv-systemctl power suspend
```

API nepřijímá shellové příkazy, libovolné názvy služeb ani cesty k programům.
Změny ověřuje pomocí identity skutečného D-Bus odesílatele a polkit.
Root je oprávněný; vyhrazený uživatel `padtv` smí povolené operace jen za běhu
aktivní místní Wayland relace greetd na tty7. Kontrola čte aktuální stav logind,
protože uživatelské služby nejsou součástí přihlašovacího scope. Jiné účty toto
oprávnění nemají. Procesy téhož TV účtu sdílejí oprávnění; nejde o sandbox
oddělující vzájemně všechny linuxové aplikace.

Systémový démon má omezený zápis, vlastní stavový adresář a automatický restart.
Síť, Bluetooth, zvuk a obraz mají samostatné adaptéry v `padtv/system.py` a
`padtv/wayland_display.py`. Nastavení obrazu používá wlroots output management;
nepotvrzenou změnu vrací po 15 sekundách. Poslední potvrzené rozlišení ukládá
uživateli a obnoví jen pro stále připojený výstup a dostupný režim.

## Instalace a trvalá data

Instalátor se otevírá z **Nastavení → Systém → Nainstalovat PadTV na disk** v Live
relaci. Zápis začne až po výběru cíle, souhrnu a závěrečném potvrzení. Smazání
vybraného disku je skutečné. Před běžnou instalací zálohujte jeho obsah.
Instalátor vytváří samostatný správcovský účet; jeho heslo není vyžadované při
startu televizního prostředí. Účet `padtv` má po instalaci zamčené heslo a nemá
Live oprávnění sudo. Z cíle se odstraní Live balíčky a samotný instalátor,
vygeneruje se initramfs a nainstaluje GRUB pro BIOS nebo UEFI podle režimu startu.
UEFI zahrnuje také záložní EFI cestu. Secure Boot není zatím ověřený.

Nová šablona Live perzistence uchovává pouze `/home`, `/var/lib/padtv`,
`/var/lib/waydroid`, `/var/lib/bluetooth`, `/var/lib/NetworkManager` a
`/etc/NetworkManager/system-connections`. Systémové programy zůstávají v ISO.
Staré plné úložiště `/ union` se nemá slepě připojit k nové verzi: může překrýt
nové systémové soubory. Při přechodu se zachová záloha a přenesou se pouze
uvedená data. Běžná instalace na disk používá standardní zapisovatelný Debian
a jeho balíčkovací systém; automatický systém aktualizací celého obrazu zatím
není implementovaný.

Windows náhled používá stejné UI s výslovným režimem preview a nevolá systémové
API Windows ani Linuxu. Pro celý systém slouží QEMU nebo Live USB z výsledného ISO.

## Store a Android

`padtv_store` odděluje validaci katalogu, poskytovatele a JSON CLI od Qt UI.
`padtv/store_ui.py` provádí operace asynchronně a mění knihovnu až po ověřeném
výsledku. Webové profily a uživatelské Flatpaky jsou oddělené od APT součástí OS.
Podrobnosti zdrojů a pěti připravených repozitářů jsou v
[architektuře Store](https://github.com/PadTVos/PadTV-OS/blob/main/docs/github-and-store.md).

Waydroid zůstává samostatnou službou a APK mají vlastní instalační cestu.
Příprava ve virtuálním stroji nastavuje SwiftShader a softwarový gralloc;
fyzické stroje si ponechávají automatický výběr grafiky. Instalace konkrétního
APK neznamená certifikaci Google TV ani zaručenou podporu DRM.

Spánek využívá logind; stabilní návrat grafiky a zařízení zatím není potvrzený.
Windows VM vypíná emulované S3/S4, protože S3 resetuje prostředky VirtIO GPU.
Podrobný průběh a omezení s2idle jsou v [protokolu ověření](https://github.com/PadTVos/PadTV-OS/blob/main/docs/validation.md).

## Primární technické podklady

- [greetd v Debianu 13](https://manpages.debian.org/trixie/greetd/greetd.5.en.html)
- [Labwc: konfigurace a správa oken](https://labwc.github.io/labwc-config.5.html)
- [Calamares 3.3.14: konfigurace modulů](https://github.com/calamares/calamares/tree/v3.3.14/src/modules)
- [Chromium: identifikace linuxových oken](https://chromium.googlesource.com/chromium/src/+/HEAD/chrome/browser/shell_integration_linux.cc)
- [Waydroid ve virtuálním stroji](https://docs.waydro.id/faq/get-waydroid-to-work-through-a-vm)

## PadBridge 0.2

Samostatný balíček `/usr/lib/padbridge`, CLI `/usr/bin/padbridge`. Ověřené APK a jejich cache jsou v `~/.local/share/padbridge/packages`, soukromá data v `data/PACKAGE`, oprávnění a logy v oddělených adresářích. Waydroid používá vlastní stávající úložiště. ART je nedostupný backend; Enjarify zůstává funkční experimentální cesta. [Rozsah implementace](https://github.com/PadTVos/PadBridge/blob/main/docs/runtime-0.2.md).
