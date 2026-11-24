# Ověření PadTV 0.3.1 — 24. listopadu 2026

## Aplikace a výkon

- **63 automatických testů prošlo na Windows i Debianu 13.** Debian testoval skutečně nainstalovaný výsledný balíček; všechny moduly byly porovnané se zdroji.
- Integrační test na výsledném ISO v QEMU/KVM použil dvě skutečné aplikace Chromium s různými profily a shodným titulkem stránky. Ověřil opakované kliknutí bez dalšího okna, výběr správné aplikace, obnovení minimalizovaného okna, nalezení aplikace po vytvoření nové instance launcheru a znovuotevření zavřeného okna.
- Časovač hodin za aktivní aplikací skutečně stojí. Na aktivní ploše se aktualizuje čas na hranici minuty, místo překreslení celého okna každou sekundu.
- Srovnání 0.3.0 / 0.3.1 se stejnými obrázky, velikostí a zastavenými hodinami: medián tří sérií po 30 vykresleních byl při 1600×900 **4,04 → 3,58 ms** (−11,5 %) a při 3840×2160 **16,82 → 12,74 ms** (−24,2 %). Jde o offscreen Qt měření celé obrazovky na tomto PC, nikoliv rychlost celého OS nebo přehrávání videa. Oba snímky při 1600×900 byly pixelově shodné.
- Windows spouštěč při opakovaném spuštění zachová maximalizaci existujícího okna VM a obnoví minimalizované okno.
- Při kontrole Souborů se podařilo zopakovat SIGSEGV v `libexo-2.so.0`: stisk Home/End v prázdném ikonovém zobrazení dereferencoval koncový iterátor. Záznam z GDB byl uložen; oprava je převzatá z upstream commitu `24b902eb389eecab7b3f8aaefdda4aff7608af9d`. S balíčkem `libexo-2-0 4.20.0-1+0padtv1` prošla stejná sekvence Home, End, všechny šipky a Enter bez pádu. Výchozí uživatelské složky vytváří `xdg-user-dirs-update` při startu relace.
- Předávání zkratky Win+Home do Windows VM prošlo s `grab-on-hover=on`. Křížek hostitelského okna je vypnutý; běžné vypnutí funguje přes PadTV nebo ACPI položku Machine → Power Down. Během hledání závady před tímto nastavením došlo také k jednomu nečistému ukončení hostitelského QEMU; následující start obnovil žurnál a další vypnutí proběhlo řádně.

## Výsledné sestavení a aktualizace

- MD5 a SHA256 všech souborů ve výsledném ISO prošly. SHA256 Windows kopie ISO souhlasí.
- KVM nabootoval výsledné ISO s kopií skutečného úložiště Windows VM; nainstalovaný balíček hlásil 0.3.1. Test aplikací proběhl přímo nad tímto obrazem, bez kopírování novějšího kódu do hosta.
- Start přes UEFI/OVMF bez Secure Boot prošel do plochy bez přihlašovacího dialogu. `systemd-analyze` naměřil 10,99 s od startu kernelu po dokončení služeb (bez času firmwaru); žádné služby nebyly ve stavu failed.
- Konečné ISO s logem PadTV i v bootloaderu prošlo startem přes BIOS do plochy (KVM, 11,05 s dle systemd-analyze). Kontrolní součty znovu prošly po této změně.
- Windows QEMU/WHPX nabootovalo konečné ISO do plochy 0.3.1 s původním úložištěm. Původní emulace VGA vedla k dlouhým prodlevám při startu; s VirtIO VGA se při ověření neopakovaly. Přidané VirtIO RNG samo předchozí prodlevu nevyřešilo, zůstává jako standardní zdroj entropie pro hosta. Přesné porovnání dob startu ve Windows nebylo provedeno.
- Původní Windows VM byla řádně vypnutá přes ACPI. Její disk je zálohovaný jako `virtual-machines/PadTV-0.3/padtv-before-0.3.1.qcow2`. Aktualizace zachovává existující `padtv.qcow2`; adresář 0.3 zůstává kvůli datům stejný.
- Na ploše je zástupce **PadTV 0.3.1 - virtuální PC**. Staré ISO 0.3.0 i původní disk 0.2 zůstávají zachované.
- Konečné ISO po doplnění Exo znovu prošlo všemi MD5/SHA256 kontrolami. Samostatná VM s novou kopií původního Windows disku potvrdila verzi opravené knihovny, automatické vytvoření českých složek, žádné failed služby a ovládání prázdné složky bez pádu. Opakované otevření Souborů aktivovalo stejné X11 okno `0x00c00007`. Ve Windows/WHPX prošel start, zobrazení složek i návrat Win+Home → Soubory; VM zůstala na domovské obrazovce.

## Systémové funkce převzaté z 0.3

Podrobné výsledky jsou v [ověření 0.3](validation-0.3.md). Verze 0.3.1 nemění síťové služby, ovladače ani kernel.

- Skutečný NetworkManager: ethernet/DHCP a WPA2 připojení přes mac80211_hwsim a hostapd, opětovné připojení, zapomenutí a zapnutí/vypnutí Wi-Fi.
- Skutečný BlueZ: emulované HCI řadiče, vyhledání, potvrzení párovacího kódu, připojení a zapomenutí zařízení.
- PipeWire: hlasitost, ztlumení, výběr výstupu a obnova uložených 37 % po restartu.
- XRandR: změna rozlišení a automatický návrat po 15 sekundách bez potvrzení.
- Český Plymouth, automatické přihlášení, otevření čistého YouTube profilu bez dialogu klíčenky, restart a vypnutí bez hesla či výzvy k vyjmutí média.

## Výstupy

- `dist/padtv-0.3.1-amd64.iso`
- SHA256: `7ace61522eb146535269d2c7aa37037afc320eaaa453e87ecb54d14c8ff00f0a`
- `dist/padtv-session_0.3.1_all.deb`
- `dist/padtv-persistence-64G.qcow2` — čistá šablona pouze pro nový disk.

## Meze ověření

Fyzické Wi-Fi/Bluetooth adaptéry, WPA3, HDMI zvuk, HDMI-CEC, Secure Boot a instalace na fyzický disk zatím nejsou ověřené. Podniková Wi-Fi 802.1X a WEP nejsou ve vlastním rozhraní implementované. VM bez předaného USB adaptéru používá kabelový internet přes Windows a vlastní Wi-Fi/Bluetooth zařízení nemá. Aktualizace byla ověřena s konkrétním úložištěm tohoto PC; obecná kompatibilita libovolné překryvné vrstvy není zaručená.

Android používá Waydroid; zkouška Jellyfin Android TV je v [ověření 0.2](validation-0.2.md). Nová aktivace okna kompozitoru je pokrytá automatickým testem, ale celý Android obraz nebyl v 0.3.1 znovu instalován a testován. PadTV není certifikované Google TV a neobsahuje Google Play. YouTube zůstává odinstalovatelnou webovou aplikací; přihlášení a přehrávání videa nebyly součástí těchto testů.
