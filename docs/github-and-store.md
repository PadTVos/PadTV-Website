# Organizace PadTV a Store

Organizace: [PadTVos](https://github.com/PadTVos). Lokální příprava je v adresáři
`repositories/`; její obsah vzniká skriptem `scripts/prepare-repositories.py`.
Publikace používá přihlášený účet se správcovským oprávněním organizace.
Všech šest repozitářů je veřejných v organizaci PadTVos.

| Veřejný repozitář | Obsah a hranice |
| --- | --- |
| [PadTV-OS](https://github.com/PadTVos/PadTV-OS) | distribuce, systémové API, Wayland, instalátor, sestavení ISO, integrace UI |
| [PadBridge](https://github.com/PadTVos/PadBridge) | experimentální Android runtime, framework a databáze kompatibility |
| [PadTV-Updates](https://github.com/PadTVos/PadTV-Updates) | metadata vydání, SHA-256, příprava odděleného podpisu; žádné soukromé klíče |
| [PadTV-Store](https://github.com/PadTVos/PadTV-Store) | `padtv_store`: validátor katalogu, poskytovatelé, JSON CLI; integrační Qt frontend |
| [PadTV-Apps](https://github.com/PadTVos/PadTV-Apps) | katalog JSON, jeho schéma a kontrola změn |
| [PadTV-Website](https://github.com/PadTVos/PadTV-Website) | uživatelská a vývojová dokumentace, podklady pro budoucí web |

OS má pro sestavení 0.6 lokální kopii Store a katalogu. Každé vydání tak používá
konkrétní ověřenou verzi; start televize není závislý na dostupnosti GitHubu.
Další vývoj může kopie nahradit verzovanými balíčky, bez změny systémového API.
ISO, virtuální disky, profily aplikací, Wi-Fi údaje a uživatelská data se do
Git historie nekopírují. Testovací ISO 0.6 je v GitHub Releases ve dvou částech
kvůli limitu 2 GiB na soubor; spojovací skript ověřuje SHA-256 částí i celého ISO.

## Poskytovatelé aplikací

| Zdroj | Stav v 0.6 | Instalace a data |
| --- | --- | --- |
| Web Apps | implementováno | manifest a samostatný profil Chromium v uživatelské knihovně |
| Flathub | implementován poskytovatel | `flatpak --user`, data v `~/.var/app`, ověření výsledku instalace |
| Google Play | oficiální TV katalog a předání instalace | Waydroid GAPPS, uživatelské přihlášení, ověření přes skutečný inventář Androidu |
| Debian APT | skutečný inventář systémových aplikací | správu systému zajišťuje APT; Store neodstraňuje základní balíčky |
| GitHub | funkční stahování vlastních APK z Releases | připnutý hash, velikost a podpis, instalace přes PadBridge |
| PadTV Repository | rozhraní zdroje vyhrazeno, nepřipojeno | komunitní balíčky po nastavení důvěryhodného podepsaného repozitáře |

Katalog není vykonatelný skript. Obsahuje identifikátory, HTTPS adresy a metadata.
Nepovoluje shellové příkazy ani automatické přidání libovolného zdroje balíčků.
Flathub používá oficiální `.flatpakrepo` s ověřováním podpisů; již existující
vzdálený zdroj se stejným názvem a jinou adresou je odmítnut.

Instalace nativních aplikací běží na pozadí. UI označí aplikaci za nainstalovanou
až po úspěšném dokončení a ověření přes Flatpak nebo inventář Waydroidu.
Otevření Google Play nevytváří instalovanou aplikaci. Vypnutý Android znamená
neznámý stav, nikoli odinstalování; existující záznamy se zachovají.
YouTube zůstává webovou uživatelskou aplikací, předinstalovanou jen při prvním
vytvoření knihovny. Odinstalování přežije restart i aktualizaci UI.

Webová aplikace není oficiální Android TV aplikace. Dostupnost přehrávání,
předplatného, DRM a ovládání dálkovým ovladačem závisí na konkrétní službě.
Android APK se nadále instalují samostatně přes Waydroid. Google Play poskytuje
oficiální `com.google.android.youtube.tv` a `com.netflix.ninja`; PadTV nekopíruje
jejich APK do repozitářů a nepoužívá neoficiální zrcadla.

`GooglePlayProvider` komunikuje přes JSON CLI `padtv-android`. Store tak nemá
přímou závislost na Python modulech UI. Příprava GAPPS je samostatná operace
s pevným výběrem VANILLA/GAPPS a oficiálními OTA adresami. Google Play může
vyžadovat registraci zařízení. Dostupnost aplikací a přehrávání DRM nejsou
garantované; registrace Waydroidu nepředstavuje Netflix certifikaci.

## Založení a zveřejnění

1. V přihlášeném GitHubu založte organizaci: <https://github.com/organizations/plan>.
2. Připojte GitHub a ověřte vlastnictví zvoleného názvu organizace.
3. `python scripts/prepare-repositories.py` vytvoří obsah pěti lokálních Git repozitářů.
4. Prohlédněte jejich soubory. `scripts/publish-repositories.ps1` ve výchozím
   režimu pouze vypíše plán; s `-Publish -Organization <skutečný-název>` vytvoří
   veřejné repozitáře přes přihlášené GitHub CLI. Před existujícími repozitáři
   a nečistým exportem se zastaví; nic nepřepisuje silou.

Příprava aktualizačních metadat není automatický aktualizátor systému. Živé USB
se aktualizuje výměnou ISO při zachování oddělené datové perzistence. Instalace
na disk používá Debian APT. Automatické přepínání obrazů a rollback OS jsou další
vývojová fáze; žádný nepodepsaný vzdálený manifest se nyní automaticky neaplikuje.

## Technické zdroje

- [Vytvoření organizace GitHub](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/creating-a-new-organization-from-scratch)
- [Uživatelské instalace Flathub](https://docs.flathub.org/docs/for-users/user-vs-system-install)
- [Příkazy Flatpak](https://docs.flatpak.org/en/latest/flatpak-command-reference.html)
- [YouTube pro Android TV v Google Play](https://play.google.com/store/apps/details?id=com.google.android.youtube.tv)
- [Netflix pro Android TV v Google Play](https://play.google.com/store/apps/details?id=com.netflix.ninja)
- [Waydroid a Google Play](https://docs.waydro.id/usage/install-and-run-android-applications)
- [Registrace Waydroidu u Google](https://docs.waydro.id/faq/google-play-certification)
- [Netflix a certifikace zařízení](https://help.netflix.com/en/node/57688)

## PadBridge / PadTV 0.4

Šestý repozitář: [PadBridge](https://github.com/PadTVos/PadBridge). Místní APK se
instalují tlačítkem ve Store, podpis se ověřuje před instalací. PadBridge má vlastní
datový prostor a typ manifestu; nemigruje ani nemaže Waydroid. Oficiální položky
YouTube TV / Netflix stále otevírají Google Play v záložním Waydroidu. Samotné
otevření obchodu nikdy nevytvoří záznam o úspěšné instalaci.

## Vydání 0.5

Store nabízí PadBridge jako preferovanou experimentální cestu pro místní APK, split APK a `.apks`. Waydroid a Google Play mají oddělená data. Každá položka katalogu obsahuje stav kompatibility. Ve správě PadBridge aplikace je diagnostika API, nativních knihoven a posledního běhu. Přesný stav je v [ověření 0.5](https://github.com/PadTVos/PadTV-OS/blob/main/docs/validation-0.5.md).

## Vydání 0.6

GitHub provider nyní skutečně stahuje APK z konkrétního vydání. V katalogu je
[PadBridge Runtime Probe](https://github.com/PadTVos/PadBridge/releases/tag/store-test-1),
vlastní testovací aplikace s původním Java DEX a XML. Pevná HTTPS adresa, velikost,
SHA-256 a otisk podpisu jsou součástí katalogu vydaného se systémem. Přesměrování
je omezené na GitHub servery; před instalací se ověřuje také package ID a skutečný
výsledek instalace. Odinstalace zachová soukromá data. Vzdálený katalog sám nemůže
podstrčit nový důvěryhodný hash. Není spouštěn stažený shellový instalátor.

Detaily aplikací zobrazují pět stavů kompatibility a konkrétní důvod omezení.
Podporované místní kontejnery zahrnují APKM. Vzhled, logo a pozadí jsou zachované,
ikony pocházejí z dodané sady PadTV-V6. Úvodní reklamní slogany byly odstraněné.
YouTube/Netflix APK dodané pro soukromé testy nejsou publikované; Google Play
zůstává odděleným zdrojem a vyžaduje kompatibilní Android prostředí.
