# Ověření prototypu — 24. listopadu 2026

## Provedené kontroly

- Windows / Python 3.10 / PyQt6 6.9.1: všech 16 automatických testů prošlo.
- Debian 13 / Python 3.13 / distribuční PyQt6: stejných 16 testů prošlo proti
  nainstalovanému balíčku v sestavovaném systému.
- Ověřena směrová navigace, Enter a návrat domů, ovládání dialogu dálkovými klávesami,
  blokování externích akcí v náhledu, výchozí zrušení vypnutí, chybějící aplikace,
  chyba skutečného podřízeného procesu a hranice rozhraní v 720p.
- Náhledy Domov, Aplikace a Nastavení byly vykresleny a vizuálně zkontrolovány.
- Debian balíček byl sestaven pomocí `dpkg-deb`; obsah odpovídá zdrojovému kódu.
- ShellCheck: shellové skripty prošly. Konfigurace Openboxu prošla parsováním XML.
- Live-build úspěšně sestavil hybridní ISO s Debian Installerem.
- Kontrola struktury ISO potvrdila záznamy El Torito pro BIOS i UEFI a hybridní oddíly.
- Opravený obraz prošel kontrolou všech **784** položek v interním `md5sum.txt`.
- SHA256 byl znovu ověřen po zkopírování výsledného ISO do složky projektu ve Windows.
- QEMU/KVM, 4 GB RAM, 2 virtuální CPU, bez pevného disku a bez sítě: automatické
  nabootování do PadTV ověřeno zvlášť pro BIOS a UEFI (OVMF bez Secure Boot).
- V BIOS relaci stisk Enter na tlačítku Otevřít Kodi spustil Kodi 21.2 a zobrazil
  jeho úvodní obrazovku. Přehrávání tím ověřené není.

## Oprava zjištěná při bootování

Výchozí menu Debian Live čekalo na Enter. Zdrojový profil nyní obsahuje vlastní
`isolinux.cfg` a GRUB `config.cfg`, které po pěti sekundách spouštějí živý systém.
Do již sestaveného ISO byly stejné dvě konfigurace promítnuty pomocí xorriso
se zachováním bootovacích záznamů; aktualizoval se vnitřní manifest i vnější SHA256.
Bootovací zkoušky a kontrola všech souborů výše proběhly na tomto opraveném výsledku.
Budoucí sestavení z aktuálních zdrojů obsahují tyto konfigurace přímo.

## Výstupy v pracovním adresáři

```text
dist/padtv-0.1.0-amd64.iso
dist/padtv-0.1.0-amd64.iso.sha256
dist/padtv-session_0.1.0_all.deb
artifacts/padtv-home.png       náhled ve Windows
artifacts/boot-bios.png        domovská obrazovka po bootu BIOS
artifacts/boot-uefi.png        domovská obrazovka po bootu UEFI
artifacts/boot-kodi.png        úvodní obrazovka Kodi po spuštění přes Enter
artifacts/build.log           původní úspěšné sestavení live-build
artifacts/iso-integrity.log   ověření souborů opraveného ISO
```

SHA256 výsledného ISO:

```text
7834cbf8d2da1c2a78a604f6e4cb8854911ee8b9fd7812d995f509cdfb07bcc5
```

Lokální sestavení využilo oddělený Debian chroot ve WSL Ubuntu:
`/var/tmp/padtv-build/debian`. Build data jsou ponechaná pro další vývoj.
Výstupy v `dist/` a `artifacts/` nejsou zahrnuté do Gitu.

## Zatím neověřeno

Instalace na disk, skutečný mini PC, HDMI obraz/zvuk, přehrávání a akcelerace videa,
Wi-Fi, uspávání, Secure Boot, CEC/IR ovladače a streamovací služby/DRM.
Systémové nástroje a web mohou vyžadovat myš. Náhled ve Windows nespouští aplikace
ani systémové příkazy. Jde o vývojový prototyp, ne o hotovou certifikovanou TV distribuci.
