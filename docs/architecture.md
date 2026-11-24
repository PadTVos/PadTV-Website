# Architektura PadTV 0.3 – System Core

Aktuální specifikace je v [system-core.md](system-core.md).
Rozdělení projektu na repozitáře a poskytovatele obchodu popisuje
[github-and-store.md](github-and-store.md).

Historická architektura s LightDM a Openboxem zůstává v
[architecture-0.3.1.md](architecture-0.3.1.md); nepopisuje nové vydání.

## PadBridge / 0.4

Android APK nyní mají samostatný experimentální runtime v Rustu.
[Architektura PadBridge](https://github.com/PadTVos/PadBridge/blob/main/docs/architecture.md)
odděluje správce balíčků, vykonávání DEX, sandbox a nativní API. Waydroid
zůstává oddělenou záložní cestou; data ani systémové jádro se nenahrazují.
