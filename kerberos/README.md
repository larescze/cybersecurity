# Kerberos

Kerberos je autentizační síťový protokol, který je určen k ověření identity v režimu klient-server. Autentizace pomocí protokolu Kerberos využívá tikety a princip jednotného přihlášení SSO (Single Sign-On). To znamená, že se v rámci domény Active Directory nemusí opakovaně provádět proces autentizace, dokud je daný tiket platný.

Kerberos využívá symetrickou kryptografii. Centrálním bodem je řadič domény Key Distribution Center (KDC), který se skládá ze dvou logicky oddělených částí: Autentizační služby (AS) pro ověření klienta a služby pro udělování tiketů TGS (Ticket-Granting Service). KDC spravuje databázi tajných klíčů uživatelů, jejichž znalostí uživatelé prokazují svou identitu. Pro komunikaci mezi stranami vygeneruje KDC klíč relace, kterým zabezpečí vzájemnou komunikaci. Bezpečnost tohoto protokolu významně závisí na vzájemné synchronizaci času protistran a krátké životnosti tiketů. Jakmile tiket vyprší, je nutné ho obnovit stejným způsobem jako při první autentizaci.

## Bezpečnost
- [Penetrační testy](tests.md)
- [Zranitelnosti](vulnerabilities.md)