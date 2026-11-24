# Architektura PadTV 0.3.1

```text
BIOS / UEFI
  -> Linux + Debian 13 + český Plymouth
  -> systemd -> LightDM -> relace padtv-session
                         -> Openbox (okna a globální klávesy)
                         -> lxpolkit (autorizace nastavení)
                         -> PadTV / Python + Qt 6
                              -> Kodi
                              -> Chromium / YouTube / web
                              -> knihovna uživatelských aplikací
                              -> padtv-android (JSON CLI)
                                   -> Weston v okně X11 / Wayland
                                   -> Waydroid / běžné Android APK
                              -> Thunar / disky a soubory
                              -> vlastní Qt nastavení / JSON worker
                                   -> NetworkManager / BlueZ (D-Bus)
                                   -> PipeWire / WirePlumber / XRandR
```

PadTV je uživatelská relace nad existujícím Debianem. Používá jeho kernel, správu
balíčků a ovladače; nový kernel ani vlastní balíčkovací systém není potřeba.

Nastavení používá samostatné krátce žijící procesy `padtv.system`, které komunikují
s Qt přes JSON na standardních vstupech a výstupech. Hesla Wi-Fi nejsou v argumentech
procesů ani v protokolech. Připojení patří aktuálnímu uživateli; úzké pravidlo
Polkit povoluje ovládání sítě pouze aktivní místní relaci skupiny `padtv`.
BlueZ agent je registrovaný jen pro konkrétní operaci a ověřuje vybrané zařízení.
Párovací kód potvrzuje uživatel v PadTV. XRandR worker obnoví předchozí rozlišení
po 15 sekundách bez potvrzení, i při uzavření vstupního kanálu.

Hlasitost, ztlumení a výstup se ukládají do `~/.config/padtv/audio.json` a při další
relaci je obnovuje `padtv-audio-restore.service`. Používá je i ovládání multimediálními
klávesami. Windows VM má vlastní 64GB QCOW2 s ext4 označeným `persistence` a konfigurací
`/ union`; launcher kopíruje prázdnou šablonu pouze pro nový disk. ISO samo žádný disk
neformátuje. Obnovu po aktualizaci celého základního ISO je potřeba navrhnout zvlášť.

Rozhraní používá virtuální plochu 1600 × 900, která se přepočítává podle rozlišení
obrazovky. Ovládací prvky jsou skutečná Qt tlačítka s přístupnými názvy. Šipky vybírají
další položku podle směru a polohy. Každá stránka má předvídatelnou počáteční volbu.
Čas se čte z místního systému. Nezobrazuje se smyšlený stav Wi-Fi ani knihovna médií.
Pozadí a logo jsou dodané PNG soubory v balíčku; rozhraní nepotřebuje jejich síťové stažení.

## Životní cyklus aplikací

`AppLibrary` ukládá validované JSON manifesty do
`$XDG_DATA_HOME/padtv/library/apps` (výchozí `~/.local/share`). YouTube je
uživatelský manifest, nikoliv systémová akce. Při vytvoření profilu se přidá jednou;
značka `defaults-v1.done` přežije odinstalování i aktualizaci rozhraní. Výslovné
opětovné přidání je dostupné ve správě aplikací. Webová aplikace má samostatný profil
Chromia v `library/profiles/<id>`, který lze odstranit spolu s aplikací.

Android manifest se odstraní až po úspěšné odinstalaci skutečného balíčku.
Instalace i odinstalace čekají na potvrzení výsledku v seznamu aplikací, protože
Android PackageInstaller pracuje asynchronně. Prosté skrytí dlaždice se nevydává
za odinstalování. Na živém médiu bez perzistence jsou uživatelská data dočasná.
Pokud přímé odebrání Waydroidem selže, otevře se standardní odinstalační dialog
Androidu přes `ACTION_DELETE`; jeho potvrzení nevyžaduje root oprávnění.
Před prací se seznamem aplikací se čeká na `sys.boot_completed=1`.

## Android

Waydroid pochází z Debian `trixie-backports`. Root helper pro první přípravu
má pevné operace: načtení binder modulu, oficiální obraz VANILLA, deklaraci
Leanback pro vyhledávání TV aktivit a zapnutí služby kontejneru. Vlastní
Android obraz ani Google služby se nepřibalují. Uživatelské APK nejsou
součástí systémového obrazu Androidu.

Relace PadTV je X11. Dvě uživatelské systemd jednotky spouštějí v této relaci
Weston s X11 backendem a kiosk shellem a Waydroid session. Mají společný životní
cyklus; ukončení PadTV zastaví obě. Grafická relace poskytuje prostředí
`DISPLAY`/`XAUTHORITY`, socket má název `padtv-android`. GUI volá samostatný
proces s JSON odpovědí, takže stažení či instalace neblokuje ovládání rozhraní.

APK se před instalací kontroluje nástrojem `aapt`; identifikátory balíčků jsou
validované a ARM-only knihovny se odmítnou. Podpora Leanback neznamená certifikaci
Google TV, dostupnost Google Play nebo kompatibilitu DRM služeb.

Aplikace běží pod přihlášeným uživatelem. Příkazy tvoří pevný seznam argumentů pro
`QProcess`, nikdy shellový řetězec. Chromium používá svůj standardní sandbox.
Napájení jde přes `systemctl` a standardní oprávnění logind/polkit. Dialog vždy
začíná na volbě Zpět. Aplikace nepřidává vlastní pravidlo pro neomezená root oprávnění.
Živá relace používá standardní administrátorská oprávnění Debian Live.

`--preview`, Windows a `--screenshot` vypínají všechny externí akce. Skutečné
spouštění aplikací vyžaduje Linux bez režimu náhledu. Chybějící aplikace i neúspěšné
spuštění mají zprávu v rozhraní. Globální Win + Home aktivuje okno PadTV přes wmctrl.

Před spuštěním aplikace samostatný proces `padtv.windows` vyhledá její existující
okno. Webové aplikace rozlišuje podle přesného profilu Chromia, nativní aplikace
podle známé třídy okna nebo rodičovských procesů. Titulek stránky není identita.
Aktivace přes EWMH obnoví i minimalizované okno; opakované kliknutí během hledání
nevytvoří další proces. Chromium bez otevřeného okna dostane požadavek na nové.
Hledání i čekání na první okno běží mimo hlavní vlákno rozhraní.

Pozadí, přechod a logo jsou uložené v jedné mezipaměti pro aktuální velikost,
hustotu pixelů a stránku. Hodiny obnovují pouze svůj obdélník na hranici minuty
a jejich časovač stojí, když je aktivní jiná aplikace. Změna velikosti nebo návrat
na plochu obnoví příslušný obraz a čas.

Balíček obsahuje rozhraní i relaci, takže není závislý na dočasném zdrojovém adresáři.
Při instalaci registruje `padtv-session` v `update-alternatives`, při odstranění ji
odregistruje. LightDM má jako výchozí relaci PadTV. Živého uživatele a jeho
automatické přihlášení vytváří `live-config`; jednotka `padtv-account` doplňuje
potřebné skupiny a vytvoří televizní účet i mimo Live systém. Správcovský účet
instalátoru je samostatný. Identifikace Debianu v `/etc/os-release` zůstává zachovaná.

První profil cílí na amd64 a otevřené grafické ovladače. Na konkrétním zařízení je
nutné ověřit akceleraci videa, HDMI zvuk, uspávání a ovladače. HDMI-CEC,
průvodce trvalým USB úložištěm a automatické aktualizace jsou další vývojové etapy.
