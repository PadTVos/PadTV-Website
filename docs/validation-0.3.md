# Ověření PadTV 0.3 — 24. listopadu 2026

## Automatické kontroly

- Windows / Python 3.10 / PyQt6: **50 testů prošlo**.
- Debian 13 / Python 3.13 / distribuční PyQt6: **stejných 50 testů prošlo** proti nainstalovanému výslednému balíčku. Zdrojové moduly byly porovnané s projektem.
- Kontroly zahrnují náhled bez systémových změn, navigaci, návrat fokusu po odpovědi služby, klávesnici pro heslo, odinstalování aplikací, neplatné rozlišení, návrat obrazu po časovém limitu nebo uzavření vstupu, meze hlasitosti a obnovu uloženého zvuku.
- ShellCheck prošel pro sestavovací a relační skripty. Kontrolní součty souborů ve výsledném ISO prošly pro MD5 i SHA256. Windows kopie ISO odpovídá SHA256.

## Skutečné služby v Debianu

Testy proběhly v QEMU/KVM jako běžný uživatel aktivní grafické relace. Testovací terminál mimo aktivní relaci neměl oprávnění měnit síť; nebyla zavedena plošná administrátorská výjimka.

- NetworkManager: odpojení a opětovné připojení ethernetu, získání DHCP adresy.
- Wi-Fi: dva emulované adaptéry mac80211_hwsim, hostapd s WPA2 a vlastní DHCP. Vyhledání sítě, připojení správným heslem, opětovné připojení uloženým profilem, zapomenutí profilu a vypnutí/zapnutí Wi-Fi prošly přes skutečné D-Bus API.
- Bluetooth: dva virtuální HCI řadiče btvirt, skutečný BlueZ, vyhledání zařízení, potvrzení kódu agentem PadTV, stav Paired/Connected a zapomenutí zařízení prošly. Testovací protějšek neověřuje zvuk sluchátek ani konkrétní ovladač.
- PipeWire/WirePlumber: změna hlasitosti, ztlumení, výběr emulovaného výstupu a následná obnova hlasitosti 37 % po restartu prošly.
- XRandR: skutečná změna 1280×800 → 1920×1080 a návrat po 15 sekundách bez potvrzení.

## Start, prohlížeč a trvalá data

- Česká načítací obrazovka Plymouth s dodaným logem byla zachycena při bootování.
- BIOS: výsledné ISO nabootovalo automaticky do PadTV v QEMU/KVM i v nativním Windows QEMU/WHPX. Grafická relace nevyžadovala přihlašovací heslo.
- UEFI/OVMF bez Secure Boot: stejné výsledné ISO nabootovalo automaticky do PadTV.
- Čistý profil YouTube se otevřel v Chromiu na stránce souhlasu se soubory cookie bez dialogu nové klíčenky. Nebylo použito přihlášení a nebylo testováno video.
- Trvalé úložiště ext4 /run/live/persistence/vda bylo připojeno. Zkušební soubor, hlasitost a odstranění YouTube přežily restart. Výchozí šablona pro uživatele je čistá; testovací data a emulované adaptéry nejsou v distribuovaném obrazu.
- Restart z nabídky PadTV proběhl bez správcovského hesla. Vypnutí s parametrem noeject proběhlo bez výzvy k vyjmutí média nebo stisknutí Enter.
- Windows spouštěč používá nové ISO, vlastní 64GB disk a ikonu PadTV 0.3. Původní disk verze 0.2 zůstává zachovaný.

## Výstupy

- dist/padtv-0.3.0-amd64.iso
- SHA256: 17b3db45f6f84cb41cbb6053adc7e67bad72ef0766fb00eccddac8a31ccb8099
- dist/padtv-session_0.3.0_all.deb
- dist/padtv-persistence-64G.qcow2 — čistá šablona, kopírovaná jen pro nový disk.

## Co zatím není ověřeno

Fyzické Wi-Fi/Bluetooth adaptéry, WPA3, televizní HDMI zvuk, HDMI-CEC, Secure Boot, instalace na fyzický disk a aktualizace základního ISO přes existující perzistenci. Podniková Wi-Fi 802.1X a WEP nejsou ve vlastním rozhraní implementované. VM bez předaného adaptéru nabízí kabelový internet přes Windows; neukazuje neexistující Wi-Fi a Bluetooth zařízení. Samotný Windows náhled je pouze náhled.

Androidová integrace zůstává přes Waydroid. Skutečná zkouška Jellyfin Android TV je zdokumentována v [ověření 0.2](validation-0.2.md); v 0.3 nebyla opakována celá instalace Android obrazu. PadTV není certifikované Google TV, neobsahuje Google Play ani záruku kompatibility chráněného streamování. YouTube je odinstalovatelná webová aplikace.
