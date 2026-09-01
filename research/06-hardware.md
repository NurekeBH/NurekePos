# 06 — Темір және платформа

> Зерттеу күні: **2026-09-01**. Автор: research субагент (Фаза 1).
> Тақырып: Flutter кассасының темірмен және платформалармен байланысы.
> Бұл құжатта КОД ЖОҚ — тек зерттеу мен инженерлік бағалау.

---

## 0. Әдістеме және шектеулер

**Таңбалау тәртібі:**

| Белгі | Мағынасы |
|---|---|
| `[РАСТАЛҒАН]` | Ресми құжаттама, pub.dev пакет беті (нұсқа + күн), өндіруші SDK docs, GitHub репозиторий/issue |
| `[ЖАНАМА]` | Блог, форум, StackOverflow, дилер сайты, үшінші тарап мақаласы |
| `[БОЛЖАМ]` | Менің инженерлік қорытындым, тікелей дереккөз жоқ |
| `[БҰҒАТТАЛҒАН]` | Нақты құрылғы / өндірушіден SDK / NDA / KZ дилерімен байланыс керек |

**Әдістеме.** pub.dev пакет беттері, GitHub кодын іздеу (`escpos-printer-db`, `win32`), өндірушілердің беттері WebFetch арқылы ашылды. Барлық нұсқа нөмірлері мен «соңғы жаңарту» күндері pub.dev беттерінен 2026-09-01 жағдайы бойынша алынды.

**Шектеулер — мұны есеп оқығанда есте ұстау керек:**

1. **Session-дегі WebSearch лимиті таусылды** (200/200) — зерттеудің екінші жартысы тек тікелей WebFetch арқылы жүрді. Сол себепті кейбір тақырыптар (Қазақстандағы дилерлер, CAS таразы хаттамасы) толық ашылмады, олар §13-те `[БҰҒАТТАЛҒАН]` ретінде тұр.
2. **Келесі домендер егресс-проксимен бұғатталған, WebFetch сәтсіз болды:**
   - `drift.simonbinder.eu` — drift-тің ресми құжаттамасы (setup, encryption, isolates беттері АШЫЛМАДЫ)
   - `download4.epson.biz` — Epson ESC/POS command reference және Character Code Tables (page_53 = KZ-1048 беті АШЫЛМАДЫ)
   - `www.iana.org`, `www.unicode.org` — KZ-1048 тіркеуі және mapping кестесі АШЫЛМАДЫ
   - `docs.sunmi.com` — SUNMI developer порталы АШЫЛМАДЫ
   - `docs.flutter.dev` — supported-platforms беті АШЫЛМАДЫ
   - `developers.google.com` — ML Kit barcode формат тізімі АШЫЛМАДЫ (pub.dev API docs арқылы айналып өттім)
   - `mike42.me` — ESC/POS code page блогы АШЫЛМАДЫ
   - `sunmi.kz` — АШЫЛМАДЫ
3. **Ешбір нақты құрылғыда тест жүргізілген жоқ.** Барлық «жұмыс істейді» деген тұжырым — құжаттамаға сүйенген, өмірде тексерілмеген.
4. pub.dev «X ай бұрын жарияланды» деп көрсетеді, нақты күн бермейді — сол себепті күндер шамамен (мыс. «46 күн бұрын» ≈ 2026-07-17).

---

## 1. Flutter + drift (SQLite)

### 1.1 drift — ағымдағы жағдай

`[РАСТАЛҒАН]` **drift 2.34.3**, жарияланған ≈35 күн бұрын (≈2026-07-28).
Платформалар: **Android, iOS, Linux, macOS, Web, Windows** — алтауы да.
Flutter Favorite белгісі бар, verified publisher `simonbinder.eu`. Тәуелділік: `sqlite3: ^3.4.0`.
<https://pub.dev/packages/drift>

`[РАСТАЛҒАН]` **sqlite3 3.5.2**, жарияланған ≈12 күн бұрын (≈2026-08-20).
Ең маңызды өзгеріс: 3.x нұсқасы **Dart hooks (native assets)** механизмін қолданады және SQLite-ті қосымшамен бірге өзі жинайды: *«Because this library uses hooks, it bundles SQLite with your application and doesn't require any external dependencies or build configuration.»*
Дайын (prebuilt) SQLite нұсқалары:
- Android: armv7a, aarch64, x86, x64
- iOS: arm64 (құрылғы + симулятор), x64 (симулятор)
- **Windows: aarch64, x64, x86**
<https://pub.dev/packages/sqlite3>

### 1.2 Windows-тағы DLL мәселесі — ЖАБЫЛҒАН

`[РАСТАЛҒАН]` Тарихи түрде Flutter + Windows-та `sqlite3.dll`-ді қолмен жинау/бумаға салу керек болатын. Қазір ол мәселе жоқ:

**sqlite3_flutter_libs 0.6.0+eol** (≈6 ай бұрын) — пакет **ресми түрде тоқтатылған (EOL)**. Беттегі мәтін: *«This package relates to version 2.x of `package:sqlite3`, and is obsolete after upgrading»*. 0.6.0-дан бастап ол ешқандай build функциясын бермейді, тек ескі 0.5.x build скрипттерін қолданудан сақтау үшін тұр.
<https://pub.dev/packages/sqlite3_flutter_libs>

`[БОЛЖАМ]` **Практикалық қорытынды:** жаңа жобада `sqlite3_flutter_libs`-ті МҮЛДЕ қоспау керек. `drift` + `sqlite3` ^3.5 жеткілікті, Windows-та DLL мәселесі болмауы тиіс. Бірақ бұл әлі нақты Windows машинасында тексерілмеген → §13 қара.

### 1.3 Шифрлау (SQLCipher)

`[РАСТАЛҒАН]` **sqlcipher_flutter_libs 0.7.0+eol** (≈6 ай бұрын) — бұл да **EOL**. Бет: *«obsolete after upgrading»* to `package:sqlite3` 3.x.
<https://pub.dev/packages/sqlcipher_flutter_libs>

`[ЖАНАМА]` drift 2.32.0-дан бастап `sqlcipher_flutter_libs`-ке тәуелділік қажет емес (іздеу нәтижесінде табылған drift changelog/docs сілтемесі).

`[РАСТАЛҒАН]` Windows-та SQLCipher-ді жинау тарихи түрде ауыр болған: drift issue #3395 (ашылған 2025-01-02) — CMake `FindOpenSSL` модулі OpenSSL 1.1.1 / 3.0 / 3.4 нұсқаларының ешқайсысында `OPENSSL_CRYPTO_LIBRARY`-ді таппайды, тіпті drift-тің ресми мысалы да сол қатемен құлайды. Issue **жабылған**, бірақ `docs` белгісімен — яғни кодта түзетілмей, құжаттамамен шешілген.
<https://github.com/simolus3/drift/issues/3395>

`[БОЛЖАМ]` **Ұсыныс:** MVP-де БҮКІЛ БД-ны SQLCipher-мен шифрламау. Себебі: (а) Windows build тәуекелі жоғары, (б) фискалдық заң талабы «БД шифрлансын» демейді — ол чектің өзгермеуін (event sourcing + hash chain) талап етеді. Оның орнына:
- Құпия деректерді (ОФД токендері, эквайринг кілттері) **бөлек** сақтау: `flutter_secure_storage` (Android Keystore / iOS Keychain / Windows DPAPI).
- Чектердің тұтастығын БД шифрлауымен емес, **оқиға тізбегіндегі hash chain** арқылы қамтамасыз ету.
- SQLCipher-ді кейінгі фазада, Windows-та тексергеннен кейін ғана қосу.
→ Бұл `/docs/CONTRACTS.md`-те шешім ретінде бекітілуі керек.

### 1.4 Миграциялар, транзакциялар, WAL, изоляттар

`[ЖАНАМА]` drift-тің изолят моделі: бір **server isolate** барлық сұранымдарды орындайды және кесте өзгерістерін broadcast жасайды; кез келген саны **client isolate** оған қосылады. Сұраным client-те құрастырылып, шикі SQL + параметрлер server-ге жіберіледі. Фондық изолятта жүргізу UI jank-ті азайтады, бірақ деректерді көшіру қосымша шығын береді (Dart VM isolate groups арқасында ол шығын аз).
<https://drift.simonbinder.eu/isolates/> (бет БҰҒАТТАЛҒАН, мазмұн іздеу сниппетінен)
<https://pub.dev/documentation/drift/latest/isolate/DriftIsolate-class.html>

`[ЖАНАМА]` Балама тәсіл — әр изолятта БД-ны дербес ашу; бірақ онда stream query-лер синхрондалмайды және lock-тардан қашу үшін `journal_mode=WAL` + `busy_timeout` pragma-ларын қосу керек.
<https://github.com/simolus3/drift/discussions/2748>

`[БОЛЖАМ]` **POS үшін ұсынылатын архитектура:**
- БД-ны **бір фондық изолятта** ұстау (`DriftIsolate`), UI тек клиент.
- `PRAGMA journal_mode=WAL` + `PRAGMA busy_timeout=5000` міндетті — синхрондау мен фискалдық жіберу кезекте параллель жүреді.
- **Sync worker-ді бөлек изолятта** ұстау, сол изолят та DriftIsolate клиенті болады.
- Миграциялар: drift-тің `MigrationStrategy` + `schema dump` тестілері (drift `drift_dev schema` командасы) — фискалдық БД болғандықтан әр миграцияға regression тест міндетті.

`[БОЛЖАМ]` **Өнімділік.** 1–3 жұмыс орны, күніне ~500–2000 чек — SQLite үшін бұл түк те емес. Өнімділік мәселесі емес; ЕҢ БАСТЫ тәуекел — **WAL + көп изолят + идемпотенттілік** түйіні, ол өнімділік емес, дұрыстық мәселесі.

### 1.5 Баламалар — салыстыру

| Пакет | Нұсқа | Соңғы жаңарту | Windows | Мәртебе | Дереккөз |
|---|---|---|---|---|---|
| **drift** | 2.34.3 | ≈35 күн бұрын | ✅ | Тірі, Flutter Favorite | [pub](https://pub.dev/packages/drift) |
| **sqlite3** (drift астында) | 3.5.2 | ≈12 күн бұрын | ✅ (aarch64/x64/x86) | Тірі | [pub](https://pub.dev/packages/sqlite3) |
| **sqflite** | 2.4.3 | ≈3 ай бұрын | ❌ (тек `sqflite_common_ffi` арқылы) | Тірі | [pub](https://pub.dev/packages/sqflite) |
| **sqlite_async** | 0.14.5 | ≈4 күн бұрын | ✅ | Тірі (PowerSync) | [pub](https://pub.dev/packages/sqlite_async) |
| **ObjectBox** | 5.3.2 | ≈3 ай бұрын | ✅ (Web ❌) | Тірі, коммерциялық | [pub](https://pub.dev/packages/objectbox) |
| **Isar** | — | — | — | ⚠️ **ТАСТАНДЫ** | төменде |
| **Realm** | — | — | — | ⛔ **EOL 2025-09-30** | төменде |

`[РАСТАЛҒАН]` **Isar — тастанды.** Түпнұсқа авторы жобаны тастаған; GitHub-та «Isar is dead, long live Isar» деген issue бар. Қауымдастық форкі `isar_plus` бар.
<https://github.com/isar/isar/issues/1689> · <https://github.com/isar/isar/discussions/1581> · <https://pub.dev/packages/isar_plus>

`[РАСТАЛҒАН]` **Realm — EOL.** MongoDB 2024-09-09-да Atlas Device Sync + Realm SDK-ларын deprecated деп жариялады; **end-of-life 2025-09-30**, яғни бүгінгі күні (2026-09-01) ол қолдаудан толық шыққан. Sync-сіз клиенттік бөлігі ашық код күйінде қалды, бірақ MongoDB қолдамайды.
<https://github.com/realm/realm-js/discussions/6884>

`[ЖАНАМА]` 2026 жылғы «Flutter local DB landscape» шолуы: Isar-ды миграция объектісі деп қарауды, әдепкі таңдау ретінде drift-ті ұсынады.
<https://luci-studio.com/blog/the-flutter-local-database-landscape-in-2026-a-maintenance-first-guide-fe6d267c/>

### 1.6 §1 бойынша қорытынды

`[БОЛЖАМ]` **drift + sqlite3 3.5.x — дұрыс таңдау, өзгертудің қажеті жоқ.** Ол жобадағы жалғыз шынымен төмен тәуекелді темір-байланысты шешім. ObjectBox — жалғыз шынайы балама, бірақ ол SQL емес, event sourcing үшін SQL қолайлырақ және ObjectBox коммерциялық лицензиясы бар. `sqlite_async`-ты бөлек қолданудың қажеті жоқ — оның WAL/concurrency идеяларын drift үстінен өзіміз құрамыз.

---

## 2. Термопринтерлер және ESC/POS

### 2.1 Қосылу тәсілдері және олардың платформалық шындығы

| Қосылу | Android | Windows | iOS | Ескерту |
|---|---|---|---|---|
| **Ethernet / Wi-Fi (TCP 9100)** | ✅ | ✅ | ✅ | Ең әмбебап, ең сенімді. Тек `dart:io` Socket керек — плагин де қажет емес |
| **USB** | ✅ (USB Host/OTG) | ✅ (спулер, RAW) | ❌ | iOS-та USB жоқ |
| **Bluetooth Classic (SPP)** | ✅ | ✅ (RFCOMM) | ❌ | iOS-та MFi сертификатысыз мүмкін емес |
| **BLE** | ✅ | ⚠️ (WinRT) | ✅ | Баяу, чек басу үшін нашар |
| **Ішкі принтер (SUNMI/PAX)** | ✅ (AIDL) | — | — | Тек сол темірде |

`[РАСТАЛҒАН]` Көптеген ESC/POS принтерлері әдепкіде **9100 портын** тыңдайды.
<https://pub.dev/packages/esc_pos_printer>

`[БОЛЖАМ]` **Ең маңызды инженерлік ұсыныс:** негізгі транспорт ретінде **Ethernet/Wi-Fi (TCP 9100)** таңдау керек. Себебі — ол үш платформада да бірдей `Socket.connect(ip, 9100)` арқылы жұмыс істейді, ешқандай плагинді, ешқандай нативті кодты, ешқандай рұқсатты қажет етпейді. Бұл «hardware-agnostic» принципіне ең сай транспорт. USB мен Bluetooth — қосымша адаптерлер.

### 2.2 Windows-та ESC/POS басу — шешім табылды

`[РАСТАЛҒАН]` Windows-та ESC/POS-ты **жүйелік спулер арқылы RAW режимде** жіберуге болады. `package:win32` репозиторийінде дәл осының ресми мысалы бар: `examples/printer_raw.dart`. Шақыру тізбегі:

`OpenPrinter → StartDocPrinter → StartPagePrinter → WritePrinter → EndPagePrinter → EndDocPrinter → ClosePrinter`

datatype ретінде `'RAW'` беріледі — бұл драйвердің форматтау конвейерін толық айналып өтеді. Мысалда тіпті ақша жәшігін ашу командасы (`\x1b\x70\x00` = ESC p) EPSON термопринтеріне жіберіледі.
<https://github.com/halildurmus/win32/blob/main/examples/printer_raw.dart>
`[РАСТАЛҒАН]` `win32` **6.4.0**, ≈26 күн бұрын, `packages/win32/lib/src/win32/winspool.g.dart` ішінде winspool API-лары бар.
<https://pub.dev/packages/win32>

`[БОЛЖАМ]` Демек Windows-та сұрақ «тікелей порт па, әлде драйвер ме» емес. Дұрыс жауап: **өндірушінің Windows драйверін орнатып, принтерді жүйеде тіркеп, оған RAW байт жіберу**. Бұл USB, Ethernet және COM-порт принтерлерінің үшеуі үшін де бір код жолын береді және Windows-та USB-ге тікелей қол жеткізудің (WinUSB/libusb драйверін ауыстыру) азабынан құтқарады.

### 2.3 Flutter пакеттері — толық аудит

> ⚠️ **ТАСТАНДЫ (2 жылдан бері жаңармаған)** деп белгіленгендерін өндірісте қолданбау керек.

| Пакет | Нұсқа | Соңғы жаңарту | Android | iOS | Windows | macOS | Linux | Лицензия | Мәртебе |
|---|---|---|---|---|---|---|---|---|---|
| **unified_esc_pos_printer** | 3.4.0 | ≈46 күн бұрын | ✅ | ✅ | ✅ | ✅ | ✅ | BSD-3 | **Тірі, ең толық** |
| **flutter_thermal_printer** | 2.2.2 | ≈6 күн бұрын | ✅ | ✅ | ✅ | ✅ | ❌ | BSD-3 | Тірі |
| **thermal_printer_flutter** | 2.0.0+3 | ≈2 ай бұрын | ✅ | ✅ | ✅ | ✅ | ⚠️ тек network | Apache-2.0 | Тірі |
| **print_bluetooth_thermal** | 1.2.2 | ≈3 ай бұрын | ✅ | ✅ | ✅ | ✅ | ❌ | — | Тірі |
| **nitro_printing** | 0.0.6 | ≈2 күн бұрын | ✅ | ✅ | ✅ | ✅ | ✅ | MIT | Жаңа, шикі (0.0.x) |
| **windows_escpos_engine** | 0.0.3 | ≈43 күн бұрын | ❌ | ❌ | ✅ | ❌ | ❌ | MIT | Жаңа, шикі (0.0.x) |
| **image** (растрлау үшін) | 4.9.2 | ≈13 күн бұрын | ✅ | ✅ | ✅ | ✅ | ✅ | MIT | Тірі, таза Dart |
| **esc_pos_utils_plus** | 2.0.4 | **≈24 ай бұрын** | ✅ | ✅ | ✅ | ✅ | ✅ | BSD-3 | ⚠️ **ТАСТАНДЫ шегінде** |
| **esc_pos_printer_plus** | 0.1.1 | ≈18 ай бұрын | ✅ | ✅ | ✅ | ✅ | ✅ | BSD-3 | Тоқтап тұр |
| **flutter_esc_pos_network** | 1.0.3 | **≈2 жыл бұрын** | ✅ | ✅ | ✅ | ✅ | ✅ | ⛔ **GPL-3.0** | ⚠️ ТАСТАНДЫ + лицензия қаупі |
| **esc_pos_printer** | 4.1.0 | **≈4 жыл бұрын** | ✅ | ✅ | ✅ | ✅ | ✅ | BSD-3 | ⛔ ТАСТАНДЫ |
| **blue_thermal_printer** | 1.2.3 | **≈4 жыл бұрын** | ✅ | ❌ | ❌ | ❌ | ❌ | — | ⛔ ТАСТАНДЫ, Android-only |
| **flutter_pos_printer_platform** | 1.4.2 | **≈3 жыл бұрын** | ✅ | ⚠️ BLE ғана | ⚠️ USB+network | ❌ | ❌ | MIT | ⛔ **DISCONTINUED** (pub.dev белгісі) |

Дереккөздер: [unified](https://pub.dev/packages/unified_esc_pos_printer) · [flutter_thermal_printer](https://pub.dev/packages/flutter_thermal_printer) · [thermal_printer_flutter](https://pub.dev/packages/thermal_printer_flutter) · [print_bluetooth_thermal](https://pub.dev/packages/print_bluetooth_thermal) · [nitro_printing](https://pub.dev/packages/nitro_printing) · [windows_escpos_engine](https://pub.dev/packages/windows_escpos_engine) · [image](https://pub.dev/packages/image) · [esc_pos_utils_plus](https://pub.dev/packages/esc_pos_utils_plus) · [esc_pos_printer_plus](https://pub.dev/packages/esc_pos_printer_plus) · [flutter_esc_pos_network](https://pub.dev/packages/flutter_esc_pos_network) · [esc_pos_printer](https://pub.dev/packages/esc_pos_printer) · [blue_thermal_printer](https://pub.dev/packages/blue_thermal_printer) · [flutter_pos_printer_platform](https://pub.dev/packages/flutter_pos_printer_platform)

**⛔ ЛИЦЕНЗИЯ ЕСКЕРТУІ:** `flutter_esc_pos_network` (және оның `flutter_esc_pos_network_universal` форкі) **GPL-3.0** лицензиясымен. Жабық кодты коммерциялық POS-қа оны қосу лицензиялық жұқтыру (copyleft) тәуекелін тудырады. Қолданбау керек. Қалғандары BSD/MIT/Apache — қауіпсіз.

### 2.4 unified_esc_pos_printer — егжей-тегжейлі

`[РАСТАЛҒАН]` **3.4.0**, ≈46 күн бұрын, verified publisher `elriztechnology.com`, BSD-3.
Қосылу матрицасы (README-ден):

| Қосылу | Android | iOS | Windows | Linux | macOS |
|---|---|---|---|---|---|
| Network (TCP/IP) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Bluetooth Classic | ✅ | ❌ | ✅ | ❌ | ❌ |
| BLE | ✅ | ✅ | ✅ | ❌ | ❌ |
| USB | ✅ (OTG) | ❌ | ✅ | ✅ (serial) | ✅ (serial) |

Мүмкіндіктер: **200+ дайын принтер профилі** (code page карталарымен), QR (8 өлшем, 4 error-correction деңгейі), штрихкодтар (UPC-A/E, EAN-13/8, CODE39, ITF, CODABAR, CODE128), ақша жәшігі (Pin 2 / Pin 5), суретті басу (ESC*, GS v 0, GS(L)), **және ең маңыздысы — text rasterization**.
<https://pub.dev/packages/unified_esc_pos_printer> · <https://github.com/elrizwiraswara/unified_esc_pos_printer>

### 2.5 Модельдер бойынша

`[РАСТАЛҒАН]` **Epson TM-T20II / TM-T20III / TM-T88II / TM-T88V / TM-T70II / TM-m30III / TM-L90** — `escpos-printer-db` профильдерінде **53: RK1048 (Kazakhstan)** код беті тіркелген. Бұл — қазақ әріптерін тікелей басуды қолдайтын жалғыз растадылған отбасы.
<https://github.com/receipt-print-hq/escpos-printer-db> (профиль файлдары `data/profile/TM-T20II.yml` т.б.)

`[ЖАНАМА]` **XPrinter XP-80T** қолдайтын код беттері: PC437, Katakana, PC850, PC860, PC863, PC865, West Europe, Greek, Hebrew, East Europe, Iran, WPC1252, **PC866 (Cyrillic#2)**, PC852, PC858, IranII, Latvian, Arabic, **PT151 (=1251)**. **KZ-1048 ЖОҚ.**
<https://manuals.plus/xprinter/xp-80t-thermal-receipt-printer-manual>

`[ЖАНАМА]` **XPrinter XP-58** ұқсас тізім: PC437, Katakana, PC860, PC863, PC865, ..., **PC866**, PC852, PC858, ..., **PT151(1251)**, PC747, WPC1257, Vietnam, PC864, Uigur, WPC1255, Thai. **KZ-1048 ЖОҚ.**
<https://manuals.plus/m/4e2a3a70a8704029d0c80f079e49413c2ab2650213dbff9786d2c2152aeb49ea>

`[БҰҒАТТАЛҒАН]` **Bixolon** (SRP-330II, SRP-Q300) және **Rongta** (RP58/RP80) чек принтерлерінің KZ-1048 қолдауы тексерілмеді. Bixolon-нан табылған жалғыз құжат — **этикетка** принтерлерінің код беттері, чек принтерінікі емес.
<https://www.bixolon.com/_upload/manual/Manual_Label_Printer_Code_Pages_english_V2.02.pdf>

---

## 3. Қазақ әріптерін басу мәселесі (raster шешімі)

> **Бұл — есептің ең маңызды бөлімі.** Ол жобаның №2 дифференциациясына («толыққанды қазақ тілі») тікелей қатысты.

### 3.1 Мәселенің мәні

`[ЖАНАМА]` ESC/POS принтерлері мәтінді Unicode-пен емес, **бір байттық код беттерімен** басады: `ESC t n` командасы 128–255 диапазонын басқа 128 таңбаға ауыстырады. Яғни бір мезетте принтерге тек 128 «қосымша» глиф қолжетімді.
<https://mike42.me/blog/2018-05-how-to-print-the-characters-in-an-esc-pos-printer-code-page> (бет БҰҒАТТАЛҒАН, мазмұн іздеу сниппетінен)

Қазақ әліпбиінде орыс кириллицасынан тыс **9 әріп** бар: **ә ғ қ ң ө ұ ү һ і** (+ бас әріптері **Ә Ғ Қ Ң Ө Ұ Ү Һ І**) — барлығы 18 глиф.

### 3.2 Код беттері бойынша талдау

| Код беті | ESC t n | Не бар | Қазақ әріптері |
|---|---|---|---|
| **CP866** (Cyrillic #2) | 17 | Орыс кириллицасы (DOS) | `[БОЛЖАМ]` ЖОҚ (тек орыс алфавиті) |
| **CP1251 / WPC1251 / PT151** | 46 | Орыс + украин/белорус/серб | `[БОЛЖАМ]` тек **і/І** бар (0xB3/0xB2); ә ғ қ ң ө ұ ү һ **ЖОҚ** |
| **CP855** (Cyrillic) | 34 | Орыс/оңт.славян | `[БОЛЖАМ]` ЖОҚ |
| **CP775** (Baltic) | 33 | Латын, Балтық | `[БОЛЖАМ]` мүлдем кириллица жоқ — **қолдануға болмайды** |
| **KZ-1048 / RK1048** | **53** | CP1251-дің қазақша модификациясы | ✅ **БАРЛЫҒЫ БАР** |

`[РАСТАЛҒАН]` **KZ-1048** (aliases: **RK1048**, **STRK1048-2002**) — Қазақстан Республикасы Экономика және сауда министрлігінің Стандарттау комитеті жасаған, 2002-02-07-де жарияланған ұлттық стандарт. Windows-1251-дің модификациясы. IANA-да KZ-1048 деп тіркелген.
<https://www.iana.org/assignments/charset-reg/KZ-1048> (бет БҰҒАТТАЛҒАН) · <https://www.compart.com/en/unicode/charsets/KZ-1048> · <https://www.fileformat.info/info/charset/KZ-1048/index.htm>

`[РАСТАЛҒАН]` ESC/POS-та KZ-1048 = **код беті 53**, командасы `ESC t 53` = `\x1B\x74\x35`.
<https://download4.epson.biz/sec_pubs/pos/reference_en/charcode/page_53.html> — беттің тақырыбы **«Page 53 [KZ-1048: Kazakhstan]»** (бет БҰҒАТТАЛҒАН, тек тақырып расталды)

`[РАСТАЛҒАН]` `escpos-printer-db/data/encoding.yml` ішінде:
```yaml
RK1048:
  iconv: RK1048
```
Ал `data/profile/TM-T70II.yml` ішінде тікелей комментарий бар: `53: RK1048  # Kazakhstan`.
<https://github.com/receipt-print-hq/escpos-printer-db/blob/master/data/encoding.yml>

### 3.3 Неге код беті — ЖЕТКІЛІКСІЗ шешім

`[БОЛЖАМ]` KZ-1048 бар екені жақсы жаңалық, бірақ оған **сүйенуге болмайды**. Үш себеп:

1. **Темірге байлау болып қалады.** KZ-1048 — тек Epson TM отбасында расталған. Қазақстанда ең көп сатылатын арзан принтерлер (XPrinter, Rongta, GPrinter) — оларда KZ-1048 ЖОҚ (§2.5). Жоба принципі «меншікті темірге ЕШҚАШАН байланбаймыз» — ал «тек Epson-да қазақша басады» дегеніміз дәл сол байлау.
2. **Dart-та RK1048 кодегі жоқ.** Dart SDK-да тек `ascii`, `latin1`, `utf8` бар. `charset_converter` (2.5.1, ≈44 күн бұрын) нативті жүйелік API-ларды пайдаланады: Android → Java `Charset`, Windows → `MultiByteToWideChar`, Linux → GLib iconv. `[БОЛЖАМ]` Windows-та KZ-1048 үшін жүйелік code page нөмірі жоқ, Java-да да `RK1048` стандартты Charset емес → конверсия платформалар арасында бірдей жұмыс істемейді. Яғни бәрібір **қолмен 128 жазбалық кесте** жазуға тура келеді.
   <https://pub.dev/packages/charset_converter>
3. **Аралас мәтін мүмкін емес.** Бір чекте қазақша тауар аты + орысша/латынша бренд аты + ₸ белгісі болса, әр фрагментте `ESC t` ауыстыру керек — бұл өте нәзік логика.

### 3.4 ҰСЫНЫЛАТЫН ШЕШІМ: чекті сурет ретінде басу (raster)

`[РАСТАЛҒАН]` `unified_esc_pos_printer` дәл осы мәселені шешу үшін **text rasterization** мүмкіндігін береді: мәтінді принтердің ішкі таңба кестелерімен емес, **Flutter-дің өз мәтін қозғалтқышымен** рендерлеп, суретке айналдырады. Құжаттамада CJK, араб, деванагари, тай жазулары мысал ретінде аталады. Сонымен қатар `rowRaster()` + `PrintRasterColumn` — «Flutter рендерлей алатын кез келген қаріп, жазу, TextStyle» үшін бағаналы кесте макетін береді.
<https://github.com/elrizwiraswara/unified_esc_pos_printer>

`[РАСТАЛҒАН]` Қағаз ені мен пиксель саны (сол құжаттамадан):

| Ені | Пиксель (dots) | Font A | Font B |
|---|---|---|---|
| **58 мм** | **384 px** | 32 таңба/жол | 42 таңба/жол |
| **80 мм** | **576 px** | 48 таңба/жол | 64 таңба/жол |

`[РАСТАЛҒАН]` `thermal_printer_flutter` (2.0.0+3) да widget-ті суретке айналдырып басуды қолдайды, Floyd–Steinberg dithering-пен.
<https://pub.dev/packages/thermal_printer_flutter>

`[РАСТАЛҒАН]` Растрлау командалары: `GS v 0` (ескі, кең таралған) және `GS ( L` (жаңа). `unified_esc_pos_printer` екеуін де қолдайды, автоматты өлшем өзгертумен. `esc_pos_utils_plus` үш функцияны береді: column format (`ESC *`) және екі bit raster формат.

### 3.5 Растрдың бағасы — көлем және жылдамдық

`[БОЛЖАМ]` Есептеу (растадылған 576/384 px санынан шығарылған арифметика, өлшенген емес):

- 80 мм чек, ені **576 px**, биіктігі ~1000 px (орташа 10–15 позициялы чек):
  `576 / 8 × 1000 = 72 000 байт ≈ 72 КБ`
- 58 мм чек, ені **384 px**, биіктігі ~1000 px:
  `384 / 8 × 1000 = 48 000 байт ≈ 48 КБ`

Салыстыру үшін мәтіндік режимде сол чек ~1–2 КБ. Яғни **растр ~30–50 есе көп байт**.

`[БОЛЖАМ]` Транспорт бойынша салдары:

| Транспорт | ~Өткізу қабілеті | 72 КБ жіберу | Қорытынды |
|---|---|---|---|
| Ethernet / Wi-Fi 9100 | МБ/с деңгейінде | < 0,1 с | ✅ Мәселе жоқ |
| USB / Windows спулер | МБ/с | < 0,1 с | ✅ Мәселе жоқ |
| SUNMI ішкі принтер (AIDL) | жоғары | < 0,1 с | ✅ Мәселе жоқ |
| **Bluetooth SPP (RFCOMM)** | ~10–100 КБ/с | ~1–7 с | ⚠️ Байқалатын кідіріс |
| **BLE** | ~1–10 КБ/с | **7–70 с** | ⛔ **Жарамсыз** |

`[БОЛЖАМ]` Одан бөлек, растр басу **физикалық басып шығару уақытын** да ұзартады: термобасқа әр нүкте жеке қыздырылады, ал мәтіндік режимде ішкі қаріп қолданылады. Нақты айырманы өлшеу керек → §13.

### 3.6 Гибридті стратегия — ҰСЫНЫС

`[БОЛЖАМ]` **Чекті бір бүтін сурет қылып басу — қате.** Ол ең қымбат вариант. Дұрысы — **фрагменттік растр**:

1. **Сандар, күндер, сомалар, штрихкод, QR, латын мәтіні** → мәтіндік режим (арзан, жылдам, ішкі қаріп).
2. **Кириллица + қазақ әріптері бар жолдар ғана** → растр (`rowRaster`).
3. **Фискалдық QR** → әрқашан ESC/POS-тың нативті QR командасы (растр емес) — сканерленуі кепілді болу үшін.
4. Принтер профилінде `RK1048` бар болса (Epson) → опция ретінде мәтіндік қазақша режимге ауысу (жылдамдық үшін), бірақ **әдепкі — растр**.

Бұл `PrinterAdapter` интерфейсінде екі мүмкіндік флагы ретінде көрсетілуі керек:
`supportsKazakhCodePage` және `supportsRaster`. Бизнес-логика ешқашан «Epson па, XPrinter пе» деп сұрамайды.

`[БОЛЖАМ]` **Қаріп таңдау.** Flutter-де қазақ әріптерін толық қамтитын, чектің 384/576 px кеңдігінде оқылатын моноширинді қаріп керек. Растрлау кезінде қаріпті **қосымшаға бекітілген** (bundled asset) қылу міндетті — жүйелік қаріпке сүйенуге болмайды (Android нұсқалары арасында әртүрлі, SUNMI-дің қытайлық прошивкасында қазақ глифтері болмауы мүмкін). Кандидаттар: Noto Sans Mono, JetBrains Mono, Roboto Mono — үшеуінде де кеңейтілген кириллица бар `[БОЛЖАМ]`, бірақ ә/ғ/қ/ң/ө/ұ/ү/һ глифтерінің әрқайсысы нақты тексерілуі керек → §13.

### 3.7 ⚠️ ФИСКАЛ ЗЕРТТЕУШІСІМЕН ҚАЙШЫЛЫҚ НҮКТЕСІ

`[БОЛЖАМ]` Егер ОФД/фискал талаптары чек мәтінінің **нақты форматын** (белгілі бір қаріп, белгілі бір таңба саны, машинамен оқылатын мәтін) талап етсе, растр шешімі қайта қаралуы мүмкін. Сонымен қатар: **ҰТК (НКТ) тауар атаулары** ГБД-дан келеді және оларда қазақ әріптері болады — яғни бұл мәселеден құтылу мүмкін емес. Бұл сұрақ фискал зерттеушісіне жіберілуі керек.

---

## 4. Штрихкод және Data Matrix сканерлеу

### 4.1 HID сканерлер (пернетақта эмуляциясы) — негізгі сценарий

`[БОЛЖАМ]` Дүкен/кафе үшін **HID (keyboard wedge) режиміндегі сканер — 90% жағдай**: арзан, драйверсіз, USB немесе 2.4GHz донгл арқылы, Android-та да, Windows-та да жүйелік пернетақта ретінде көрінеді.

`[ЖАНАМА]` Flutter-де оны TextField-сіз оқу: `KeyboardListener` (жаңа `HardwareKeyboard` API-ына негізделген) немесе `Focus.onKeyEvent`. Дайын пакет: `code_scan_listener` — ол дәл `HardwareKeyboard` API-ын қолданады.
<https://github.com/utamori/code_scan_listener>

`[РАСТАЛҒАН]` `RawKeyboardListener` **deprecated** — v3.18.0-2.0.pre-дан кейін. Миграция: `RawKeyboard → HardwareKeyboard`, `RawKeyboardListener → KeyboardListener`, `RawKeyEvent → KeyEvent`.
<https://docs.flutter.dev/release/breaking-changes/key-event-migration> (домен бұғатталған, бірақ URL мен мазмұн іздеуден расталды) · <https://api.flutter.dev/flutter/widgets/RawKeyboardListener-class.html>

`[РАСТАЛҒАН]` ⛔ **ЕҢ БАСТЫ ТӘУЕКЕЛ:** flutter/flutter issue **#79849** — *«Flutter Windows Keyboard Issue using external barcode scanner»*. Сыртқы штрихкод сканері Chrome, Word, Notepad-та тамаша жұмыс істейді, Android-тағы Flutter қосымшасында да жұмыс істейді, ал **Windows-тағы Flutter қосымшасында деректер келмейді**. Issue **2021-04-06-да ашылған, әлі АШЫҚ**, приоритеті **P2**, Windows командасы triaged жасаған. Бұл — Flutter-дің Windows пернетақта оқиғаларын өңдеуіндегі мәселе, сканердің темірінде емес.
<https://github.com/flutter/flutter/issues/79849>

`[БОЛЖАМ]` Бұл — **жобаның Windows тармағы үшін ең қауіпті нақты факт**. Егер Windows-та HID сканер жұмыс істемесе, Windows POS нұсқасының мәні жоғалады (дүкенде сканерсіз касса болмайды). Тексеру №1 приоритет.

`[ЖАНАМА]` Тағы бір белгілі қиындық: `HardwareKeyboard` оқиғалары focus node фокуста болғанда ғана шығады, ал ескі `RawKeyboard` глобалды тыңдай алатын — глобалды шорткаттар/сканер оқуын ұйымдастыру қиындады (flutter#154141).
<https://github.com/flutter/flutter/issues/154141>

`[БОЛЖАМ]` **Қосымша HID қиындықтары** (жобалық шешім керек):
- Сканер сандық емес пернетақта раскладкасында «типтегенде», кириллица режимінде қате символдар келуі мүмкін → сканерді **сандық/латын режимге** қатты бекіту немесе scan code деңгейінде оқу.
- Data Matrix кодтарында **GS (0x1D) сепараторы** болады — HID сканер оны әдетте тастап кетеді немесе басқа символмен ауыстырады. Маркировка коды үшін бұл ФАТАЛЬДІ. Сканерді «GS-ті сақтап жіберетін» режимге баптау керек (әр өндірушіде бөлек конфиг штрихкоды).
- Сканердің «оқу аяқталды» белгісі — әдетте `Enter`. Бірақ адам да Enter басады → таймингпен ажырату керек (сканер таңбалары ~10–30 мс аралықпен келеді, адам ~150+ мс).

### 4.2 Камера арқылы сканерлеу

`[РАСТАЛҒАН]` **mobile_scanner 7.4.0**, ≈42 күн бұрын:
- ✅ Android, ✅ iOS, ✅ macOS, ✅ Web
- ❌ **Windows**, ❌ Linux
Қозғалтқыштар: Android → CameraX/ML Kit, iOS/macOS → AVFoundation/Apple Vision, Web → ZXing/zxing-wasm.
Web-те `dataMatrix` zxing-wasm және ZXing-js backend-терінде қолдайды.
<https://pub.dev/packages/mobile_scanner>

`[РАСТАЛҒАН]` **google_mlkit_barcode_scanning 0.16.1**, ≈14 күн бұрын: **тек Android және iOS**. Бетте тікелей жазылған: *«Web or any other platform is not supported»*. iOS min deployment target 15.5, Android minSdk 21.
<https://pub.dev/packages/google_mlkit_barcode_scanning>

`[РАСТАЛҒАН]` ML Kit `BarcodeFormat` enum-ында **`dataMatrix` БАР**. Толық тізім: all, unknown, code128, code39, code93, codabar, **dataMatrix**, ean13, ean8, itf, qrCode, upca, upce, pdf417, aztec.
<https://pub.dev/documentation/google_mlkit_barcode_scanning/latest/google_mlkit_barcode_scanning/BarcodeFormat.html>

`[БОЛЖАМ]` **Қорытынды: камера арқылы сканерлеу Windows-та ЖОҚ.** Екі негізгі пакеттің де Windows қолдауы жоқ. Windows нұсқасында камера сканері — не бар емес, не үшінші тарап шешімі (Dynamsoft, Scanbot — коммерциялық, ақылы). Бұл Windows нұсқасын **міндетті түрде HID сканерге тәуелді** етеді, ал HID сканер #79849 мәселесімен қауіп астында. Екі тәуекел бір-бірін күшейтеді.

`[БОЛЖАМ]` Одан бөлек, камерамен **Data Matrix** оқу теориялық түрде мүмкін болғанымен, маркировка кодтары **өте ұсақ** (жиі 5–8 мм) және телефон камерасының автофокусы жақын қашықтықта қиналады. Кәсіби 2D сканер (SUNMI ішкі, Zebra, Honeywell, Newland) — маркировка үшін іс жүзінде міндетті.

### 4.3 Data Matrix және маркировка

`[ЖАНАМА]` Маркировка (ЧЗ/Data Matrix) жүйесімен жұмыс істеу үшін сканер **міндетті түрде 2D** болуы және Data Matrix оқуы керек.
<https://markirovka.ru/knowledge/fast_start/start/proverka-skanera-data-matrix-instruktsiya> · <https://pos-center.ru/journal/podhodit-li-skaner-shtrih-kodov-dlya-schityvaniya-data-matrix/>

`[ЖАНАМА]` **SUNMI L2k / L2s** — маркировка тауарларымен жұмыс үшін жасалған ТСД, ЕГАИС / ФГИС / «Честный знак» талаптарына сай. L2S сканер модулінің DataMatrix параметрлерін конфиг штрихкодтарымен баптау керек.
<https://www.cleverence.ru/hardware/mdc/sunmi/sunmi-l2k/4085/> · <https://help.mertech.ru/tsd/Mertech_Sunmi/nastroyka_L2S.html>
> Ескерту: бұл дереккөздер **Ресей** маркировка жүйесіне (Честный знак) қатысты. Қазақстан маркировкасы бөлек. Темір деңгейінде айырма жоқ (Data Matrix — Data Matrix), бірақ код форматы мен ОФД хаттамасы бөлек → фискал зерттеушісі растауы керек.

`[БОЛЖАМ]` **Ұсынылатын сканер стратегиясы:**
- Негізгі: **HID сканер** (кез келген өндіруші, 2D, Data Matrix + GS сепараторын сақтайтын режимде).
- SUNMI/PAX құрылғысында: **ішкі сканер SDK** (broadcast/AIDL) — HID-тен сенімдірек, GS сепараторын дұрыс береді.
- Резерв (шағын кофехана, планшет қана): **камера** (`mobile_scanner`), Android/iOS-та ғана.
- `ScannerAdapter` интерфейсі үшеуін де жасырады.

`[РАСТАЛҒАН]` **sunmi_scanner 0.0.8**, ≈14 күн бұрын, **тек Android**. Мүмкіндіктер: сервисті bind/unbind, сканерлеу оқиғаларының ағыны, қосылу күйін бақылау, бағдарламалық start/stop. Тексерілген құрылғылар: **SUNMI L2ks, P3H, L2s PRO**.
<https://pub.dev/packages/sunmi_scanner>

---

## 5. Ақша жәшігі (cash drawer)

### 5.1 Принтер арқылы ашу — стандартты жол

`[РАСТАЛҒАН]` ESC/POS командасы `ESC p` = `0x1B 0x70` — `win32` пакетінің ресми мысалында дәл осы байттар (`\x1b\x70\x00`) EPSON термопринтеріне RAW түрде жіберіледі.
<https://github.com/halildurmus/win32/blob/main/examples/printer_raw.dart>

`[РАСТАЛҒАН]` `unified_esc_pos_printer` ақша жәшігін ашуды **Pin 2 (әдепкі) және Pin 5** арқылы қолдайды: `ticket.openCashDrawer()` / `ticket.openCashDrawer(pin: CashDrawer.pin5)`.
<https://github.com/elrizwiraswara/unified_esc_pos_printer>

`[РАСТАЛҒАН]` `esc_pos_utils_plus`-та да drawer pin enum-ы бар (`enums.dart` ішінде «drawer pins» enum-ы расталды).
<https://github.com/andrey-ushakov/esc_pos_utils/blob/master/lib/src/enums.dart>

`[БОЛЖАМ]` Физикалық схема: жәшік **RJ11/RJ12 кабельмен принтерге** қосылады (принтердің артындағы «DK» порты). Касса жәшікті **тікелей** басқармайды — ол принтерге команда жібереді, принтер 12/24 В импульс береді. Демек:
- Ақша жәшігі **принтерге тәуелді ресурс**, бөлек құрылғы емес.
- Принтер жоқ болса (тек чек экранға/PDF), жәшік те ашылмайды.
- `DLE DC4` (`0x10 0x14`) — балама, «real-time» команда, принтер буфері бос болмаса да орындалады. Бұл кейбір принтерлерде `ESC p`-дан сенімдірек `[БОЛЖАМ]`.

### 5.2 Тікелей USB жәшіктер

`[БОЛЖАМ]` USB-HID немесе USB-serial интерфейсі бар дербес ақша жәшіктері (мыс. APG «USB ProUSB») бар, бірақ:
- Android-та: `usb_serial` (0.5.2, **≈2 жыл бұрын**, тек Android) немесе қолмен USB Host API — `[РАСТАЛҒАН]` <https://pub.dev/packages/usb_serial>
- Windows-та: `flutter_libserialport` (0.6.0, ≈13 ай бұрын) немесе `win32` FFI
- iOS-та: **мүмкін емес**

`[БОЛЖАМ]` **Ұсыныс: MVP-де тікелей USB жәшіктерді ҚОЛДАМАУ.** Тек «принтер арқылы» сценарий. Себебі: шағын кафе/дүкен сегментінде RJ11 жәшік — стандарт, ал USB жәшік — сирек әрі қымбат. Бұл `CashDrawerAdapter` интерфейсінің бір ғана имплементациясы (`PrinterCashDrawer`) деген сөз, кейін кеңейтуге ашық.

### 5.3 Android vs Windows айырмасы

| | Android | Windows | iOS |
|---|---|---|---|
| Принтер арқылы (network) | ✅ | ✅ | ✅ |
| Принтер арқылы (USB) | ✅ | ✅ (спулер RAW) | ❌ |
| Принтер арқылы (BT SPP) | ✅ | ✅ | ❌ |
| SUNMI ішкі жәшік порты | ✅ (`sunmi_printer_plus`) | — | — |
| Тікелей USB жәшік | ⚠️ USB Host | ⚠️ serial/FFI | ❌ |

`[РАСТАЛҒАН]` `sunmi_printer_plus` (4.1.1) ақша жәшігін ашуды **және оның күйін оқуды** қолдайды — SUNMI-де жәшік күйін білу артықшылығы бар (инкассация есебі үшін пайдалы).
<https://pub.dev/packages/sunmi_printer_plus>

---

## 6. Тұтынушы дисплейі (customer display)

### 6.1 Екінші экран (Android Presentation API)

`[РАСТАЛҒАН]` **presentation_displays 1.0.0**, жарияланған **≈2 жыл бұрын** ⚠️ (тастанды шегінде). Платформалар: Android, iOS. Verified publisher `smew.tech`, BSD-2.
Жұмыс принципі: Flutter widget-ті бөлек `FlutterEngine`-ге айналдырып, кэштеп, оны нативті Android `Presentation` диалогына береді. HDMI немесе сымсыз қосылған екінші экранға шығады. Негізгі және екінші экран арасында екі жақты дерек алмасу бар. Android-та бөлек entry point қолдайды.
<https://pub.dev/packages/presentation_displays>

`[БОЛЖАМ]` Бұл жалғыз шынайы жалпы шешім, бірақ ол **2 жыл жаңармаған** — Flutter 3.3x-тегі multi-engine API өзгерістерімен үйлесімділігі күмәнді. Тәуекел: орташа.

### 6.2 SUNMI dual-screen

`[РАСТАЛҒАН]` `sunmi_printer_plus` (4.1.1, ≈12 ай бұрын, **тек Android**) құрылғының **LCD экранына сурет пен мәтін шығаруды** қолдайды.
<https://pub.dev/packages/sunmi_printer_plus>

`[БОЛЖАМ]` Мұнда екі бөлек нәрсені шатастырмау керек:
1. **SUNMI кіші LCD** (мыс. V2, T2 mini-дегі бір жолды/шағын экран) — `sunmi_printer_plus` арқылы қолжетімді, тек мәтін/сурет.
2. **SUNMI dual-screen** (D2, T2 сияқты екі толық экранды құрылғылар) — бұл Android Presentation API-ы арқылы жүреді, яғни `presentation_displays` немесе SUNMI-дің өз SDK-сы керек.

`[БҰҒАТТАЛҒАН]` SUNMI-дің ресми dual-screen SDK құжаттамасы (`docs.sunmi.com`) егресс-проксимен бұғатталған. Flutter wrapper пакеті pub.dev іздеуінде **табылмады** — SUNMI dual-screen үшін дайын Flutter пакеті жоқ сияқты. Бұл өз platform channel-ымызды жазуды талап етеді.

### 6.3 Сериялық VFD дисплейлер

`[БОЛЖАМ]` Классикалық VFD (мыс. Epson DM-D110, Posiflex PD-320) — COM/USB-serial арқылы, көбіне ESC/POS-қа ұқсас қарапайым командалармен (2×20 таңба). Flutter-де дайын пакет жоқ; `flutter_libserialport` (Windows/Linux/macOS/Android) арқылы қолмен жазуға болады. Epson өзінің customer display-лары үшін бөлек «Customer Display Character Code Tables» құжатын жүргізеді (`download4.epson.biz/.../charcode_dm/`) — бет бұғатталған.

`[БОЛЖАМ]` **MVP ұсынысы:** Тұтынушы дисплейі **MVP-ге кірмеуі керек** немесе ең қарапайым түрде ғана — SUNMI LCD + Android Presentation. Себебі: шағын кафе/кофеханада тұтынушы дисплейі заң талабы емес, ал ол үшін 3 бөлек имплементация (LCD/Presentation/VFD) жазу — MVP шеңберінен шығу. `CustomerDisplayAdapter` интерфейсі бекітілсін, имплементация — no-op + SUNMI.

---

## 7. Таразылар

### 7.1 Flutter экожүйесіндегі жағдай — өте нашар

`[РАСТАЛҒАН]` pub.dev-те таразыға арналған іс жүзінде **бір ғана** пакет бар: **digital_scale 1.2.3+1**, ≈12 күн бұрын, **тек Android**, GPL-3.0 ⚠️.
Қолдайтын модельдер: **Toledo Prix 3, Elgin DP-1502, Urano (POP Light, US20/2 POP-S, US31/2 POP-S), UPX EA-20/EA-32**. Қосылу: serial, USB немесе Bluetooth.
**CAS та, Mettler Toledo-ның еуропалық модельдері де, «Штрих» те тізімде ЖОҚ** — бұл бразилиялық нарыққа арналған пакет.
<https://pub.dev/packages/digital_scale>

> ⛔ Лицензия: **GPL-3.0** — коммерциялық жабық POS-қа қосуға болмайды.

`[БОЛЖАМ]` **Қорытынды: Flutter-де таразы үшін дайын шешім ЖОҚ.** Хаттаманы өзіміз жазуға тура келеді.

### 7.2 Сериялық порт

| Пакет | Нұсқа | Соңғы жаңарту | Android | Windows | Linux | macOS | iOS |
|---|---|---|---|---|---|---|---|
| **flutter_libserialport** | 0.6.0 | ≈13 ай бұрын | ✅* | ✅ | ✅ | ✅ | ❌ |
| **usb_serial** | 0.5.2 | **≈2 жыл бұрын** ⚠️ | ✅ | ❌ | ❌ | ❌ | ❌ |

<https://pub.dev/packages/flutter_libserialport> · <https://pub.dev/packages/usb_serial>

`[БОЛЖАМ]` `*` **Android-тағы ескерту:** `flutter_libserialport` — `libserialport` C-кітапханасының орамасы, ол `/dev/tty*` құрылғы файлдарын ашады. Android-та қарапайым қосымшаның `/dev/ttyUSB0`-ге қол жеткізуге **рұқсаты жоқ** (root немесе өндірушінің арнайы прошивкасы керек). Android-та USB-serial үшін дұрыс жол — Android USB Host API + FTDI/CDC драйверін user space-те іске асыру, яғни `usb_serial` (бірақ ол 2 жыл жаңармаған). Бұл нақты құрылғыда тексерілуі керек → §13.

### 7.3 Хаттамалар

`[БОЛЖАМ]` Негізгі хаттамалар (жалпы инженерлік білім, ресми сілтеме табылмады):
- **CAS** (CAS ER Plus, AP-1): ASCII негізіндегі сұраныс-жауап, `ENQ`/`ACK`, салмақ өрісі + тұрақтылық белгісі.
- **Mettler Toledo**: SICS хаттамасы (`S` — тұрақты салмақ сұрау, `SI` — лезде).
- **«Штрих-М» / «Масса-К»**: өз екілік хаттамалары, көбіне COM/USB-HID.
- Барлығына ортақ: 9600/8N1 немесе 4800/7E1, салмақ ASCII мәтін ретінде.

`[БҰҒАТТАЛҒАН]` Қазақстанда іс жүзінде қандай таразылар қолданылатыны және олардың хаттама құжаттамасы — тексерілмеді (WebSearch лимиті). Дилерден datasheet сұрау керек.

### 7.4 Таразы-таңбалағыш (label printing scale)

`[БОЛЖАМ]` Таразы-таңбалағыштар (CAS CL5000, «Штрих-Принт», DIGI SM) — **дербес құрылғылар**. Олар:
- Өз ішінде тауар базасын (PLU) сақтайды, оны кассадан емес, өз утилитасынан немесе Ethernet арқылы жүктейді.
- Салмағы бар **EAN-13 штрихкодын** басады: әдетте `2` префиксі + PLU коды + салмақ/баға + checksum.
- Кассаға **тікелей қосылмайды** — касса тек басылған этикеткадағы штрихкодты сканерлейді және префикс бойынша «бұл салмақ штрихкоды» деп таниды.

`[БОЛЖАМ]` **MVP үшін салдары маңызды әрі жеңіл:**
- Таразы-таңбалағышты **интеграциялаудың қажеті жоқ**. Керегі — кассада **салмақ штрихкодын талдау (parse) логикасы**: префикс, PLU өрісінің ұзындығы, салмақ/баға өрісінің масштабы конфигурацияланатын болуы.
- Тікелей қосылатын таразы (кассаға сым арқылы) — бөлек, күрделірек сценарий. **MVP-ден шығару керек** (шағын кафе/кофеханада таразы жоқ, шағын дүкенде — таразы-таңбалағыш).
- Бұл `/docs/CONTRACTS.md`-ке `WeightBarcodeFormat` конфигурациясы ретінде кіруі керек.

---

## 8. SUNMI / PAX / Posiflex SDK-лары

### 8.1 SUNMI

`[РАСТАЛҒАН]` pub.dev-тегі SUNMI пакеттері (2026-09-01):

| Пакет | Нұсқа | Соңғы жаңарту | Платформа | Не істейді |
|---|---|---|---|---|
| **sunmi_printer_plus** | 4.1.1 | ≈12 ай бұрын | **Android ғана** | Принтер (мәтін, штрихкод, QR, сурет, кесу), **ақша жәшігі + күйі**, **LCD экран** |
| **sunmi_scanner** | 0.0.8 | ≈14 күн бұрын | **Android ғана** | Ішкі сканер, оқиға ағыны, bind/unbind |
| **sunmi_kali** | 1.3.1 | ≈2 ай бұрын | **Android ғана** | Тек принтер (Kali Alimentares компаниясының қажеттігі үшін жасалған) |
| sunmi_citybank / sunmi_citybank_v2 | 1.0.1 / 1.0.0 | ≈6 / ≈3 күн бұрын | Android | Банк интеграциясы (бізге қатысы жоқ) |
| sunmi_mtb | 1.0.4 | ≈25 күн бұрын | Android | — |

`sunmi_printer_plus` бетінде тікелей жазылған: **«THIS PACKAGE WILL WORK ONLY IN ANDROID!»**
<https://pub.dev/packages/sunmi_printer_plus> · <https://pub.dev/packages/sunmi_scanner> · <https://pub.dev/packages/sunmi_kali>

`[БОЛЖАМ]` SUNMI интеграциясы Android **AIDL сервистері** арқылы жүреді (`woyou.aidlservice.jiuiv5.IWoyouService` — ішкі принтер). Бұл дегеніміз: қосымша SUNMI прошивкасында тіркелген сервиске bind жасайды. SUNMI емес құрылғыда bind сәтсіз болады → адаптер соны түсініп, келесі транспортқа өтуі керек.

`[БҰҒАТТАЛҒАН]` SUNMI-дің ресми developer порталы (`docs.sunmi.com`) бұғатталған — эквайринг қосымшасын Intent арқылы шақыру, dual-screen SDK, cash drawer SDK құжаттамасы оқылмады.

### 8.2 PAX

`[РАСТАЛҒАН]` pub.dev-те PAX пакеттері:

| Пакет | Нұсқа | Соңғы жаңарту | Платформа | Не істейді |
|---|---|---|---|---|
| **flutter_pax_printer_utility** | 0.1.6 | ≈10 ай бұрын | **Android ғана** | PAX принтері **NeptuneLiteApi SDK** арқылы: мәтін, QR, сурет, кесу. **Штрихкод, divisor, принтер ақпараты — ІСКЕ АСЫРЫЛМАҒАН.** Тексерілген: **A920, A910S** |
| **nebula_flutter_plugin** | 1.0.3 | ≈4 ай бұрын | **Android** | **PAX Nebula SDK** орамасы: **Sale, Refund, Void, Settlement** + custom txn. Қосылу: **TCP (Wi-Fi), Bluetooth, USB, Cloud (MQTT)** |
| pax_player | — | — | Android | Карта оқу, штрихкод/QR сканерлеу |
| pos_printer_unity | — | — | Android | Бірнеше бренд бір API астында |

<https://pub.dev/packages/flutter_pax_printer_utility> · <https://pub.dev/packages/nebula_flutter_plugin>

`[БОЛЖАМ]` **Маңызды бақылау (эквайринг зерттеушісіне):** `nebula_flutter_plugin` — Nebula SDK арқылы PAX терминалын **сыртқы құрылғы** ретінде басқаруға мүмкіндік береді (TCP/BT/USB арқылы). Бұл «касса планшетте, терминал бөлек» сценарийі үшін өте маңызды. Ал `flutter_pax_printer_utility` — қосымшаның **PAX терминалының өзінде** жұмыс істеуін болжайды (PayDroid). Бұл екеуі мүлдем бөлек архитектуралар және эквайринг адаптерінің дизайнына тікелей әсер етеді.

### 8.3 Posiflex

`[БҰҒАТТАЛҒАН]` Posiflex үшін pub.dev-те **бірде-бір Flutter пакеті табылмады**. Posiflex негізінен Windows POS терминалдарын шығарады (кейбір Android модельдері бар). `[БОЛЖАМ]` Posiflex-тің Windows терминалдары — стандартты Windows машиналары, ондағы принтер/жәшік/дисплей жүйелік драйверлер арқылы жұмыс істейді, яғни арнайы SDK қажет емес: §2.2-дегі спулер RAW тәсілі жеткілікті. Бұл нақты құрылғыда тексерілуі керек.

### 8.4 Hardware-agnostic принципін сақтау — интерфейс дизайны

`[БОЛЖАМ]` Жоба принципі: «бизнес-логикада нақты провайдер аты ЕШҚАШАН hardcode жасалмайды». Темірге қатысты бұл былай жүзеге асуы керек:

**`/packages/domain` — таза Dart, I/O жоқ.** Мұнда тек **абстракциялар мен деректер** тұрады:
```
ReceiptDocument   — не басу керек (домен түсінігі, ESC/POS емес)
DeviceCapability  — { supportsRaster, supportsKazakhCodePage,
                      supportsCashDrawer, supportsQr, paperWidthDots }
PrinterId, ScannerId, DrawerId — идентификаторлар
```

**Порт интерфейстері** (домен қабатында, имплементациясыз):
```
ReceiptPrinterPort  : print(ReceiptDocument), capabilities, status
BarcodeScannerPort  : Stream<ScanResult>
CashDrawerPort      : open(), Stream<DrawerState>
CustomerDisplayPort : show(DisplayView)
ScalePort           : Stream<WeightReading>
```

**Адаптерлер** (`/packages/hardware` — жаңа пакет ұсынылады, `/packages/domain`-нен тыс):
```
EscPosNetworkPrinter   (Socket 9100)      — Android/iOS/Windows
EscPosUsbWindowsPrinter (win32 spooler)   — Windows
EscPosBluetoothPrinter (SPP)              — Android/Windows
SunmiInnerPrinter      (AIDL)             — SUNMI Android
PaxInnerPrinter        (NeptuneLite)      — PAX Android
NoopPrinter            (тест/PDF)         — барлығы
```

`[БОЛЖАМ]` **Үш қатаң ереже:**
1. **ESC/POS байттары доменге ЕШҚАШАН жетпейді.** Домен `ReceiptDocument` береді, адаптер оны байтқа айналдырады. Растр ма, мәтін бе — адаптердің шешімі.
2. **Capability-driven, brand-driven емес.** Код ешқашан `if (printer is SunmiPrinter)` деп жазбайды; ол `if (caps.supportsRaster)` деп жазады.
3. **Әр адаптер өз пакетінде, компиляция кезінде оқшауланған.** SUNMI пакеті Android-қа ғана жиналуы керек — әйтпесе Windows build құлайды (Flutter-де бұл conditional import + platform-specific plugin declaration арқылы).

### 8.5 Қазақстандағы қолжетімділік

`[БҰҒАТТАЛҒАН]` SUNMI, PAX, Posiflex құрылғыларының Қазақстандағы ресми дистрибьюторлары, бағалары және сервистік қолдауы **тексерілмеді** — WebSearch лимиті таусылды, `sunmi.kz` домені бұғатталған. Бұл коммерциялық шешім үшін маңызды: SUNMI-ге сүйенген адаптер жазып, кейін оны Қазақстанда сатып алу мүмкін болмаса — бос жұмыс.
`[БОЛЖАМ]` Бірақ жоба принципі бойынша бұл **блокер емес**: егер архитектура шынымен hardware-agnostic болса, SUNMI адаптері — қосымша, ядро емес.

---

## 9. Платформалар салыстыру кестесі (Android / Windows / iOS)

### 9.1 Перифериялық құрылғылар матрицасы

| Құрылғы / қосылу | Android | Windows | iOS | Ескерту |
|---|---|---|---|---|
| **Термопринтер — Ethernet/Wi-Fi (9100)** | ✅ | ✅ | ✅ | Таза `dart:io` Socket, плагинсіз |
| **Термопринтер — USB** | ✅ USB Host | ✅ спулер RAW | ❌ | iOS-та USB жоқ |
| **Термопринтер — Bluetooth SPP** | ✅ | ✅ RFCOMM | ❌ | iOS: MFi керек |
| **Термопринтер — BLE** | ✅ | ⚠️ WinRT | ✅ | Растр үшін тым баяу |
| **Термопринтер — ішкі (SUNMI/PAX)** | ✅ AIDL | — | — | Тек сол темірде |
| **Растр басу (қазақ әріптері)** | ✅ | ✅ | ✅ | Барлық транспортта жұмыс істейді |
| **Ақша жәшігі — принтер арқылы** | ✅ | ✅ | ⚠️ тек network принтер | |
| **Ақша жәшігі — тікелей USB** | ⚠️ | ⚠️ | ❌ | MVP-ден тыс |
| **HID сканер (keyboard wedge)** | ✅ | ⛔ **flutter#79849** | ⚠️ BT HID | Windows — ең үлкен тәуекел |
| **Камера сканері (`mobile_scanner`)** | ✅ | ❌ **ЖОҚ** | ✅ | Windows қолдауы жоқ |
| **Камера сканері (ML Kit)** | ✅ | ❌ | ✅ | Тек Android/iOS |
| **Data Matrix оқу (камера)** | ✅ ML Kit | ❌ | ✅ | Ұсақ кодтарда сапасы төмен |
| **Ішкі сканер (SUNMI)** | ✅ | — | — | `sunmi_scanner` |
| **Тұтынушы дисплейі (Presentation)** | ✅ | ❌ | ⚠️ | `presentation_displays` (2 жыл ескі) |
| **Тұтынушы дисплейі (SUNMI LCD)** | ✅ | — | — | `sunmi_printer_plus` |
| **Сериялық порт (таразы/VFD)** | ⚠️ рұқсат мәселесі | ✅ | ❌ | `flutter_libserialport` |
| **Kiosk mode** | ✅ Lock Task | ⚠️ қолмен | ⚠️ Guided Access | `kiosk_mode` 0.9.0 |
| **Локалды БД (drift/SQLite)** | ✅ | ✅ | ✅ | Жалғыз толық көк аймақ |

### 9.2 iOS-тың шектеулері — жобаға қауіп бар ма?

`[РАСТАЛҒАН]` iOS-та Bluetooth Classic / SPP арқылы кез келген аксессуармен байланысу **мүмкін емес**. External Accessory framework тек **MFi (Made for iPhone) сертификатталған** аксессуарлармен жұмыс істейді — оларда Apple-дің аутентификация коппроцессоры болуы керек. Сертификатсыз құрылғыға тек **BLE** (CoreBluetooth) арқылы қосылуға болады.
<https://developer.apple.com/documentation/externalaccessory>

`[РАСТАЛҒАН]` `flutter_blue_plus` (2.3.12, ≈22 күн бұрын) бетінде тікелей: **«❗ Bluetooth Classic is not supported ❗»**. Тек BLE Central рөлі. Windows үшін бөлек `flutter_blue_plus_winrt` пакеті керек.
<https://pub.dev/packages/flutter_blue_plus>

`[БОЛЖАМ]` **iOS-тағы POS шындығы:**

iOS-та жұмыс істейтін нәрсе:
- ✅ Ethernet/Wi-Fi принтер (TCP 9100) — толық
- ✅ Камера сканері (`mobile_scanner`, ML Kit) — толық
- ✅ BLE принтер — техникалық түрде, бірақ растр басу үшін тым баяу
- ✅ Локалды БД, офлайн, синхрондау — толық

iOS-та жұмыс істемейтін нәрсе:
- ❌ USB принтер, USB сканер, USB таразы
- ❌ Bluetooth SPP принтер (нарықтағы арзан принтерлердің көбі дәл осындай)
- ❌ SUNMI/PAX ішкі темірі
- ❌ Сериялық порт
- ⚠️ HID сканер — тек Bluetooth HID пернетақта ретінде

**Қорытынды: iOS «Flutter — бір кодтан үш платформа» болжамын БҰЗБАЙДЫ, бірақ оны шектейді.**
iOS-та **толық жабдықталған касса** құру мүмкін емес. Бірақ iOS-та **Ethernet принтер + камера сканері** конфигурациясы толық жұмыс істейді — бұл шағын кофехана үшін жеткілікті.

`[БОЛЖАМ]` **Стратегиялық ұсыныс:** iOS-ты **MVP-нің бірінші релизінен шығару**, бірақ кодта оны бұзбау. Себептер:
- Сегмент — шағын кафе/дүкен, Қазақстанда олар iPad емес, Android планшет/SUNMI қолданады `[БОЛЖАМ]`.
- iOS-та ешбір нақты артықшылық жоқ, ал тестілеу шығыны үлкен (нақты iPad + принтер керек).
- Apple App Store-да фискалдық қосымшаның қосымша модерация тәуекелі бар `[БОЛЖАМ]`.
- Flutter коды бәрібір ортақ — iOS кейін 2 апталық жұмыспен қосылады.
Бұл CLAUDE.md-дегі «Flutter (Android, iOS, Windows — бір кодтан)» тұжырымымен **ішінара қайшы келеді** → §12.

### 9.3 Windows-тағы Flutter-дің жетілгендігі

`[БОЛЖАМ]` Windows — Flutter-дің ең әлсіз тармағы **плагин экожүйесі** тұрғысынан (Flutter engine-нің өзі тұрақты). Осы зерттеуде табылған нақты дәлелдер:
- ⛔ `mobile_scanner` — Windows қолдауы **жоқ** (расталған)
- ⛔ `google_mlkit_barcode_scanning` — Windows қолдауы **жоқ** (расталған)
- ⛔ `presentation_displays` — Windows қолдауы **жоқ** (расталған)
- ⛔ HID сканер — flutter#79849, 2021-ден ашық (расталған)
- ⚠️ `flutter_blue_plus` — Windows үшін бөлек форк керек (расталған)
- ⚠️ SQLCipher — Windows CMake/OpenSSL мәселесі (расталған, docs арқылы жабылған)
- ✅ drift/sqlite3 — Windows толық қолдау (расталған)
- ✅ ESC/POS спулер RAW — `win32` мысалы бар (расталған)

`[БОЛЖАМ]` Windows-та әр перифериялық құрылғы үшін **өз FFI кодымызды жазуға дайын болу керек**. Бұл MVP еңбек көлемін айтарлықтай өсіреді.

### 9.4 Android: фондық жұмыс, батарея, kiosk

`[РАСТАЛҒАН]` **kiosk_mode 0.9.0**, ≈33 күн бұрын, Android + iOS, verified publisher `mews.com`, BSD-3.
Android → **Lock Task mode** (screen pinning), iOS → **Guided Access**. API: `getKioskMode()`, `watchKioskMode()`, `startKioskMode()`, `stopKioskMode()`.
<https://pub.dev/packages/kiosk_mode>

`[БОЛЖАМ]` **Android-тағы фондық жұмыс — офлайн-first үшін маңызды тәуекел:**
- ОФД-ға чек жіберу кезегі мен синхрондау **қосымша фонда тұрғанда да** жүруі керек. Android 12+ батарея оптимизациясы (Doze, App Standby, background restrictions) `Isolate`-ті де, `WorkManager`-ді де кідіртеді.
- Дұрыс жол: **foreground service** (тұрақты хабарландырумен) + `WorkManager` резерв ретінде.
- Flutter-де: `flutter_foreground_task` немесе `flutter_background_service` `[БОЛЖАМ]` — бұл пакеттердің ағымдағы мәртебесі осы зерттеуде тексерілмеді (WebSearch лимиті) → §13.
- Ең қауіпсіз конфигурация: **kiosk mode + экран әрқашан қосулы + қуат көзінде** — касса планшеті бәрібір тұрақты қуатта тұрады. Онда Doze мәселесі негізінен жойылады.
- Қосымшаны «Battery optimization: don't optimize» тізіміне қосуды орнату кезінде сұрау керек.

`[БОЛЖАМ]` **72 сағаттық автономды режим талабына салдары:** Егер қосымша фонда өлтірілсе, чектер жоғалмайды (олар SQLite-та), бірақ **байланыс қалпына келгенде автоматты жіберу болмайды** — кассир қосымшаны ашқанда ғана. Заң «байланыс қалпына келгенде барлық чек кезекпен жіберіледі» дейді. Бұл фискал зерттеушісімен келісілуі керек: «қосымша ашылғанда жіберу» жеткілікті ме, әлде фондық сервис міндетті ме → §12.

---

## 10. Ұсынылатын Flutter пакеттерінің тізімі

> Барлық нұсқа/күн 2026-09-01 жағдайы бойынша pub.dev-тен.

### 10.1 Негізгі (қабылдау ұсынылады)

| Пакет | Нұсқа | Соңғы жаңарту | Платформалар | Лиц. | Не үшін |
|---|---|---|---|---|---|
| [drift](https://pub.dev/packages/drift) | 2.34.3 | ≈35 күн | And/iOS/Win/mac/Lin/Web | MIT | Локалды БД, ORM, миграция |
| [sqlite3](https://pub.dev/packages/sqlite3) | 3.5.2 | ≈12 күн | And/iOS/Win/mac/Lin/Web | MIT | drift-тің нативті қабаты |
| [image](https://pub.dev/packages/image) | 4.9.2 | ≈13 күн | барлығы | MIT | Чекті растрлау, dithering |
| [unified_esc_pos_printer](https://pub.dev/packages/unified_esc_pos_printer) | 3.4.0 | ≈46 күн | And/iOS/Win/Lin/mac | BSD-3 | ESC/POS + **растр мәтін** + 200+ профиль |
| [win32](https://pub.dev/packages/win32) | 6.4.0 | ≈26 күн | Windows | BSD-3 | Спулер RAW басу, FFI |
| [mobile_scanner](https://pub.dev/packages/mobile_scanner) | 7.4.0 | ≈42 күн | And/iOS/mac/Web | MIT | Камера сканері (Windows ЖОҚ) |
| [flutter_blue_plus](https://pub.dev/packages/flutter_blue_plus) | 2.3.12 | ≈22 күн | And/iOS/Lin/mac/Web | BSD-3 | BLE (Windows үшін бөлек форк) |
| [kiosk_mode](https://pub.dev/packages/kiosk_mode) | 0.9.0 | ≈33 күн | And/iOS | BSD-3 | Lock Task / Guided Access |
| [charset_converter](https://pub.dev/packages/charset_converter) | 2.5.1 | ≈44 күн | And/iOS/Win/mac/Lin | — | CP866/CP1251 конверсиясы (растр резерві) |

### 10.2 Шартты (тексергеннен кейін)

| Пакет | Нұсқа | Соңғы жаңарту | Ескерту |
|---|---|---|---|
| [flutter_thermal_printer](https://pub.dev/packages/flutter_thermal_printer) | 2.2.2 | ≈6 күн | Балама/резерв, Linux жоқ |
| [sunmi_printer_plus](https://pub.dev/packages/sunmi_printer_plus) | 4.1.1 | ≈12 ай | Android-only, SUNMI адаптері үшін |
| [sunmi_scanner](https://pub.dev/packages/sunmi_scanner) | 0.0.8 | ≈14 күн | Android-only, 0.0.x — шикі |
| [nebula_flutter_plugin](https://pub.dev/packages/nebula_flutter_plugin) | 1.0.3 | ≈4 ай | PAX эквайринг — **эквайринг зерттеушісіне** |
| [flutter_pax_printer_utility](https://pub.dev/packages/flutter_pax_printer_utility) | 0.1.6 | ≈10 ай | PAX принтері, штрихкод іске асырылмаған |
| [presentation_displays](https://pub.dev/packages/presentation_displays) | 1.0.0 | **≈2 жыл** ⚠️ | Тұтынушы дисплейі, ескі |
| [flutter_libserialport](https://pub.dev/packages/flutter_libserialport) | 0.6.0 | ≈13 ай | Таразы/VFD, Android-та күмәнді |
| [nitro_printing](https://pub.dev/packages/nitro_printing) | 0.0.6 | ≈2 күн | 6 платформа, WinSpool, бірақ **0.0.x** |
| [windows_escpos_engine](https://pub.dev/packages/windows_escpos_engine) | 0.0.3 | ≈43 күн | Windows RAW, бірақ **0.0.x** |

### 10.3 ⛔ ҚОЛДАНБАУ (тастанды / лицензия / тоқтатылған)

| Пакет | Нұсқа | Соңғы жаңарту | Себеп |
|---|---|---|---|
| [flutter_pos_printer_platform](https://pub.dev/packages/flutter_pos_printer_platform) | 1.4.2 | ≈3 жыл | **DISCONTINUED** (pub.dev белгісі) |
| [esc_pos_printer](https://pub.dev/packages/esc_pos_printer) | 4.1.0 | **≈4 жыл** | Тастанды |
| [blue_thermal_printer](https://pub.dev/packages/blue_thermal_printer) | 1.2.3 | **≈4 жыл** | Тастанды, Android-only |
| [flutter_esc_pos_network](https://pub.dev/packages/flutter_esc_pos_network) | 1.0.3 | **≈2 жыл** | Тастанды + **GPL-3.0** |
| [esc_pos_utils_plus](https://pub.dev/packages/esc_pos_utils_plus) | 2.0.4 | **≈24 ай** | Тастанды шегінде (тікелей емес, транзитивті болуы мүмкін) |
| [digital_scale](https://pub.dev/packages/digital_scale) | 1.2.3+1 | ≈12 күн | **GPL-3.0** + бразилиялық таразылар ғана |
| [sqlite3_flutter_libs](https://pub.dev/packages/sqlite3_flutter_libs) | 0.6.0+eol | ≈6 ай | **EOL** — қоспау |
| [sqlcipher_flutter_libs](https://pub.dev/packages/sqlcipher_flutter_libs) | 0.7.0+eol | ≈6 ай | **EOL** — қоспау |
| [usb_serial](https://pub.dev/packages/usb_serial) | 0.5.2 | **≈2 жыл** | Тастанды, Android-only |
| Isar | — | — | **Тастанды** ([issue #1689](https://github.com/isar/isar/issues/1689)) |
| Realm | — | — | **EOL 2025-09-30** ([announcement](https://github.com/realm/realm-js/discussions/6884)) |

`[БОЛЖАМ]` **Транзитивті тәуелділік ескертуі:** `esc_pos_utils_plus` (24 ай ескі) `flutter_thermal_printer` және `thermal_printer_flutter` пакеттерінің тәуелділігі. Яғни оны тікелей қоспасақ та, ол жобаға кіреді. `unified_esc_pos_printer` — өз ESC/POS генераторын жазған сияқты, бұл артықшылық. Тексеру керек.

---

## 11. Тәуекелдер тізімі (ықтималдық × әсер)

### 11.1 ЕҢ ЖОҒАРЫ 5 ТӘУЕКЕЛ

---

#### 🔴 R1 — Windows-та HID штрихкод сканері жұмыс істемейді

- **Не істемейді:** Windows-тағы Flutter қосымшасына HID (keyboard wedge) сканерден деректер келмейді. Сол сканер Notepad, Chrome, Word-та және Android-тағы дәл сол Flutter кодында жұмыс істейді.
- **Неге:** Flutter-дің Windows embedder-індегі пернетақта оқиғаларын өңдеу мәселесі. `[РАСТАЛҒАН]` flutter/flutter **#79849**, ашылған **2021-04-06**, әлі **АШЫҚ**, приоритет **P2**, Windows командасы triaged жасаған. 5 жылдан бері түзетілмеген.
- **Ықтималдық:** **Жоғары** (issue ашық және ескі)
- **Әсер:** **Функцияны шектейді → Windows тармағын тоқтатады**. Сканерсіз дүкен кассасы жоқ. Windows-та камера сканері де жоқ (R2) — яғни Windows-та сканерлеудің ЕШҚАНДАЙ жолы қалмайды.
- **Баламасы:**
  1. `KeyboardListener` / `HardwareKeyboard` (жаңа API) issue-дегі ескі `RawKeyboard`-тан гөрі жұмыс істеуі мүмкін — тексеру керек.
  2. Сканерді **HID емес, USB-COM (virtual serial) режимге** ауыстыру — көп 2D сканерде мұндай режим бар, конфиг штрихкодымен қосылады. Сонда `flutter_libserialport`/`win32` арқылы оқимыз. **Бұл ең сенімді балама.**
  3. Windows-та Win32 `RegisterRawInputDevices` (WM_INPUT) арқылы FFI сканер драйверін өзіміз жазу — сканерді пернетақтадан бөлек құрылғы ретінде оқу. Ең күшті шешім, бірақ ~1–2 апта жұмыс.
  4. Ең соңғы шара: Windows-ты MVP-ден шығару.
- **Тексеру үшін не керек:** Windows 10/11 машинасы + кез келген USB HID 2D сканер + минималды Flutter қосымша. `KeyboardListener` мен `RegisterRawInputDevices` екеуін де сынау. **1 күндік жұмыс, ең жоғары приоритет.**

---

#### 🔴 R2 — Windows-та камера арқылы сканерлеу мүлдем жоқ

- **Не істемейді:** Windows-та камерамен штрихкод/QR/Data Matrix оқу.
- **Неге:** `[РАСТАЛҒАН]` `mobile_scanner` 7.4.0 — Windows ❌, Linux ❌. `google_mlkit_barcode_scanning` 0.16.1 — тек Android/iOS. Экожүйеде тегін Windows баламасы жоқ.
- **Ықтималдық:** **Жоғары** (расталған факт, «ықтималдық» емес — бұл шындық)
- **Әсер:** **Функцияны шектейді.** Жеке алғанда — қолайсыздық (HID сканер бәрібір керек). Бірақ R1-мен бірге — Windows тармағын тоқтатады.
- **Баламасы:** Windows-та камера сканерін ұсынбау (HID/USB-COM сканерге сүйену); немесе коммерциялық SDK (Dynamsoft, Scanbot) — ақылы, лицензия шығыны.
- **Тексеру үшін не керек:** Тексерудің қажеті жоқ — расталған. Керегі — **өнім шешімі**: Windows-та камера сканері болмайды деп бекіту.

---

#### 🔴 R3 — Қазақ әріптері арзан принтерлерде басылмайды

- **Не істемейді:** ә ғ қ ң ө ұ ү һ (8 әріп) стандартты ESC/POS мәтін режимінде басылмайды.
- **Неге:** `[РАСТАЛҒАН]` Оларды қамтитын жалғыз ESC/POS код беті — **KZ-1048 / RK1048, page 53**, ол `escpos-printer-db`-де тек **Epson TM** отбасы профильдерінде тіркелген. `[ЖАНАМА]` XPrinter XP-58/XP-80 (Қазақстанда ең көп таралған арзан принтер) тек **PC866** және **PT151 (=CP1251)** береді — оларда бұл әріптер жоқ. CP1251-де тек **і/І** бар. CP775-те кириллица мүлдем жоқ.
- **Ықтималдық:** **Жоғары**
- **Әсер:** **Функцияны шектейді → жобаның №2 дифференциациясын жояды.** «Толыққанды қазақ тілі» — жобаның негізгі уәдесі; чекте қазақша дұрыс басылмаса, ол уәде орындалмайды.
- **Баламасы:** ✅ **Шешім бар: чекті растр (bitmap) ретінде басу.** `[РАСТАЛҒАН]` `unified_esc_pos_printer` 3.4.0 Flutter-дің мәтін қозғалтқышымен мәтінді суретке айналдырады (`rowRaster`, `PrintRasterColumn`). Кез келген қаріп, кез келген жазу. Ені: 58мм = 384 px, 80мм = 576 px. Қаріпті қосымшамен бірге жеткізу керек.
- **Қосымша тәуекел:** растр **~30–50 есе көп байт** жібереді (`[БОЛЖАМ]`: 80мм × 1000px ≈ 72 КБ, мәтіндік режимде ~1,5 КБ). Ethernet/USB-те мәселе жоқ, Bluetooth SPP-де ~1–7 с, **BLE-де жарамсыз**.
- **Тексеру үшін не керек:** Epson TM-T20III + XPrinter XP-80 + Rongta RP80 үшеуінде де: (а) растр чектің басылу уақыты, (б) 384/576 px-те қазақ әріптерінің оқылуы, (в) таңдалған қаріпте 18 глифтің болуы. **2 күндік жұмыс, приоритет №2.**

---

#### 🟠 R4 — Data Matrix маркировка кодындағы GS сепараторы жоғалады

- **Не істемейді:** HID сканер маркировка Data Matrix кодын оқығанда, ішіндегі **GS (0x1D)** топ сепараторын тастап кетеді немесе басқа символға айналдырады. Нәтижесінде код ОФД-ға дұрыс емес форматта жіберіледі.
- **Неге:** `[БОЛЖАМ]` HID (keyboard wedge) режимі — пернетақта эмуляциясы, ал 0x1D-ге сәйкес пернетақта пернесі жоқ. Әдепкі баптауда көп сканер оны жай тастап кетеді. Әр өндіруші оны бөлек конфиг штрихкодымен қосады.
- **Ықтималдық:** **Жоғары** (әдепкі баптауда — әрқашан дерлік)
- **Әсер:** **Функцияны шектейді → маркировка талабын бұзады** (заң талабы!). Чек ОФД-дан қайтарылуы мүмкін.
- **Баламасы:**
  1. Сканерді «GS-ті сақтап жіберу» режиміне баптау (орнату нұсқаулығына кіруі керек, әр модель үшін бөлек конфиг штрихкоды).
  2. HID орнына USB-COM режимі — сепараторлар сол күйінде келеді.
  3. SUNMI/PAX ішкі сканері — AIDL/broadcast арқылы шикі деректер келеді, сепаратор сақталады `[БОЛЖАМ]`.
  4. Қосымшада **валидация**: маркировка коды күтілген құрылымға сай ма — сай болмаса кассирге «сканерді баптаңыз» деп нақты хабарлама.
- **Тексеру үшін не керек:** Нақты маркировкаланған тауар (сыра/мотор майы) + 2–3 түрлі HID сканер. Фискал зерттеушісінен ҚР маркировка кодының нақты құрылымын алу.
- `[БҰҒАТТАЛҒАН]` ҚР маркировка Data Matrix форматы (ЕАЭО/ЧЗ форматына сай ма?) — фискал зерттеушісі растауы керек.

---

#### 🟠 R5 — ESC/POS Flutter экожүйесінің тұрақсыздығы

- **Не істемейді:** таңдалған принтер пакеті жарты жылдан кейін жаңармай қалады, Flutter жаңа нұсқасында сынады, немесе қажет транспортты қолдамайды.
- **Неге:** `[РАСТАЛҒАН]` Аудиттегі 13 ESC/POS пакетінің **6-уы 18 айдан ескі**, оның 4-уі 2–4 жыл жаңармаған, 1-уі ресми **DISCONTINUED**. Ең сапалы кандидат `unified_esc_pos_printer` — жалғыз әзірлеушінің жеке жобасы (v3.4.0, ≈46 күн). Ең жаңалары (`nitro_printing` 0.0.6, `windows_escpos_engine` 0.0.3) — **0.0.x**, өндіріске жарамсыз.
- **Ықтималдық:** **Орта**
- **Әсер:** **Қолайсыздық → функцияны шектейді.** Жобаны тоқтатпайды, бірақ форк жасау/қайта жазу шығынын береді.
- **Баламасы:** ✅ **ESC/POS байт генераторын ӨЗІМІЗ жазу.** Бұл — қорқынышты естілгенімен, іс жүзінде **шағын әрі толық құжатталған** жұмыс: ESC/POS — 30 жылдық тұрақты стандарт, чек басу үшін ~15 команда жеткілікті (`ESC @`, `ESC t`, `ESC a`, `ESC !`, `GS !`, `GS v 0`, `GS ( k` (QR), `GS k` (штрихкод), `GS V` (кесу), `ESC p` (жәшік), `DLE EOT` (күй)). Сыртқы пакет тек **транспорт** (Socket, USB, BT, спулер) үшін қалады, ол әлдеқайда ауыстыруға оңай.
  - Таза Dart генератор `/packages/hardware/escpos`-та тұрады, 100% unit-тестпен жабылады (байт-байт салыстыру — темірсіз тестіленеді!).
  - Транспорт: Ethernet — `dart:io` (пакетсіз); Windows USB — `win32` (Microsoft API, ешқашан жоғалмайды); Android USB/BT — жалғыз пакетке тәуелділік.
- **Тексеру үшін не керек:** Тексерудің қажеті жоқ — бұл архитектуралық шешім. Ұсыныс: **ESC/POS генераторын өз кодымызда жазу, Фаза 2-де** (Іргетас фазасы, өйткені ол `/packages/domain`-ге көрші).

---

### 11.2 Қалған тәуекелдер

| # | Тәуекел | Ықт. | Әсер | Балама / тексеру |
|---|---|---|---|---|
| R6 | **Растр басу тым баяу** (Bluetooth SPP-де 1–7 с, BLE-де жарамсыз) | Орта | Қолайсыздық | Ethernet/USB-ті әдепкі транспорт қылу; BT SPP-де мәтіндік+растр гибридін қолдану; BLE принтерлерін қолдамау деп жариялау. **Тексеру:** нақты принтерде уақыт өлшеу |
| R7 | **iOS-та толық касса құру мүмкін емес** (USB жоқ, BT SPP жоқ, SUNMI/PAX жоқ) | Жоғары (расталған) | Функцияны шектейді | iOS-ты MVP v1-ден шығару; iOS-та тек Ethernet принтер + камера сканері конфигурациясын қолдау. **Тексеру керек емес — өнім шешімі** |
| R8 | **`presentation_displays` 2 жыл ескі**, жаңа Flutter-де сынуы мүмкін | Орта | Қолайсыздық | Тұтынушы дисплейін MVP-ден шығару; SUNMI LCD-мен шектелу. **Тексеру:** Flutter 3.3x + нақты HDMI экран |
| R9 | **Android-та `/dev/ttyUSB*`-ке рұқсат жоқ** → `flutter_libserialport` таразы үшін жұмыс істемеуі мүмкін | Орта | Функцияны шектейді | Таразыны MVP-ден шығару (таразы-таңбалағыш штрихкоды жеткілікті); немесе Android USB Host API-мен өз FFI-ымыз. **Тексеру:** Android планшет + USB-serial таразы |
| R10 | **Android батарея оптимизациясы фондық синхрондауды өлтіреді** → 72 сағаттық кезек уақытында жіберілмейді | Орта | Функцияны шектейді (заң талабы!) | Foreground service + kiosk mode + тұрақты қуат; «battery optimization» ерекшелігін орнатуда сұрау. **Тексеру:** Android 13/14 планшетінде 24 сағаттық тест. **Фискал зерттеушісімен келісу керек** |
| R11 | **SQLCipher Windows-та жиналмайды** (CMake/OpenSSL) | Орта | Қолайсыздық | БД-ны шифрламау; құпияларды `flutter_secure_storage`-та ұстау; тұтастықты hash chain-мен қамтамасыз ету. **Тексеру:** егер шифрлау шешімі қабылданса ғана |
| R12 | **Транзитивті тастанды тәуелділік** (`esc_pos_utils_plus` 24 ай ескі, көп пакеттің ішінде отыр) | Орта | Қолайсыздық | Өз ESC/POS генераторымыз (R5) бұл тәуекелді де жояды. **Тексеру:** `flutter pub deps` |
| R13 | **GPL лицензиялы пакетті байқаусыз қосу** (`flutter_esc_pos_network`, `digital_scale`) | Төмен | Жобаны тоқтатады (заңды тұрғыда) | CI-да лицензия тексеру қадамын қосу (`flutter pub deps` + allowlist). **Тексеру:** CI ережесін жазу |
| R14 | **Windows-та Bluetooth принтер** `flutter_blue_plus_winrt` бөлек форкіне тәуелді | Төмен | Қолайсыздық | Windows-та BT принтерін қолдамау деп жариялау (Ethernet/USB жеткілікті) |
| R15 | **SUNMI/PAX Қазақстанда қолжетімсіз болуы** | Белгісіз | Қолайсыздық | Архитектура hardware-agnostic болса — әсер жоқ. **Тексеру:** KZ дилерлерімен байланыс |
| R16 | **Растрда таңдалған қаріпте қазақ глифтері жоқ** немесе 384 px-те оқылмайды | Орта | Функцияны шектейді | Қаріпті asset ретінде жеткізу; 3–4 кандидатты сынау. **Тексеру:** нақты принтерде басу |
| R17 | **Windows спулер RAW басуда драйвер мәтінді бұрмалауы** (кейбір драйверлер RAW-ды да өңдейді) | Төмен | Қолайсыздық | «Generic / Text Only» немесе өндірушінің «RAW» драйверін пайдалану. **Тексеру:** Windows + USB принтер |

---

## 12. Қайшылықтар мен ашық сұрақтар

### 12.1 Дереккөздер арасындағы қайшылықтар

| # | Қайшылық | Дереккөз А | Дереккөз Б | Менің бағам |
|---|---|---|---|---|
| C1 | **`print_bluetooth_thermal`-дың iOS қолдауы** | pub.dev беті: iOS ✅ қолдайды, «Bluetooth Serial Port Profile (SPP)» деп жазады, MAC мекенжайымен қосылады | Apple: iOS-та SPP тек MFi аксессуарларымен мүмкін; `flutter_blue_plus`: «Bluetooth Classic is not supported» | `[БОЛЖАМ]` pub.dev сипаттамасы **шатастыратын**. iOS-та ол шын мәнінде BLE қолданады немесе iOS қолдауы толық емес. **Растр басу үшін iOS-та BT принтерге сүйенбеу керек.** |
| C2 | **`sqlite3_flutter_libs` нұсқасы** | Бірінші фетч: 0.6.0+eol | Екінші фетч (sqlcipher бетінен): 0.7.0+eol | Екеуі де EOL — мәні өзгермейді. Пакетті мүлдем қоспаймыз |
| C3 | **`thermal_printer_flutter` Android USB** | Пакет README-сі: Android USB ❌ | `flutter_thermal_printer` (басқа пакет): Android USB ✅ | Екі бөлек пакет — қайшылық емес, бірақ атаулары шатастыратындай ұқсас. Құжаттамада толық атын жазу керек |
| C4 | **`flutter_libserialport` Android қолдауы** | pub.dev: Android ✅ | `[БОЛЖАМ]` Android-та `/dev/tty*`-ге рұқсат жоқ | Пакет «жиналады» дегенді білдіреді, «жұмыс істейді» дегенді емес. → R9 |
| C5 | **CLAUDE.md: «Flutter — Android, iOS, Windows бір кодтан»** | Жоба құжаты | §9.2: iOS-та USB/SPP/SUNMI/PAX жоқ | `[БОЛЖАМ]` Болжам **толықтай дұрыс емес**. Код ортақ, бірақ **темір мүмкіндіктері ортақ емес**. iOS-та шектеулі касса ғана болады |

### 12.2 Басқа зерттеушілермен қайшы келуі мүмкін тұстар

**→ Эквайринг зерттеушісіне (терминал SDK):**

1. `[РАСТАЛҒАН]` PAX үшін екі мүлдем бөлек архитектура бар:
   - `nebula_flutter_plugin` (1.0.3): касса **бөлек құрылғыда**, PAX терминалы TCP/BT/USB/MQTT арқылы басқарылады
   - `flutter_pax_printer_utility` (0.1.6): касса **PAX терминалының өзінде** (PayDroid) жұмыс істейді
   Бұл екеуі PaymentAdapter интерфейсіне мүлдем әртүрлі талап қояды. **Қайсысы басым болатынын эквайринг зерттеушісі анықтауы керек.**
2. `[БҰҒАТТАЛҒАН]` SUNMI-де эквайринг қосымшасын Android **Intent** арқылы шақыру (Kaspi Pay, Halyk Pay) — `docs.sunmi.com` бұғатталғандықтан тексерілмеді. Kaspi QR интеграциясы Intent арқылы ма, әлде REST арқылы ма — эквайринг зерттеушісінің тақырыбы.
3. `[БОЛЖАМ]` Егер эквайринг «Smart POS» (терминал = касса) моделін таңдаса, онда **Windows тармағы мәнін жоғалтады** (PAX/SUNMI — Android). Бұл темір стратегиясына тікелей әсер етеді.

**→ Фискал зерттеушісіне (чек басу):**

1. **Ең маңызды:** `[БОЛЖАМ]` Біз чекті **сурет (растр) ретінде** басуды ұсынамыз (қазақ әріптері үшін). Егер ОФД/ҚР заңы чектің нақты мәтіндік форматын, белгілі бір қаріпті немесе таңба санын талап етсе — бұл шешім қайта қаралуы керек.
2. `[РАСТАЛҒАН]` **ҰТК (НКТ) тауар атаулары** ГБД-дан келеді және оларда қазақ әріптері болады — яғни қазақ әріптерін басу мәселесінен қашу мүмкін емес, ол міндетті.
3. `[БОЛЖАМ]` **Фискалдық QR** — біз оны ESC/POS-тың **нативті QR командасымен** (растр емес) басуды ұсынамыз, сканерленуі кепілді болу үшін. Егер фискал талабы QR-дың нақты өлшемін/error correction деңгейін көрсетсе — соны білу керек.
4. `[БОЛЖАМ]` **72 сағаттық автономды режим + Android батарея оптимизациясы** (R10). Заң «байланыс қалпына келгенде барлық чек жіберіледі» дейді. Егер қосымша фонда өлтірілсе, жіберу тек қосымша ашылғанда болады. **Бұл заң талабын қанағаттандыра ма?** — фискал зерттеушісі жауап беруі керек. Жауап «жоқ» болса, foreground service міндетті болады.
5. `[БОЛЖАМ]` **Маркировка Data Matrix + GS сепараторы** (R4). ҚР маркировка кодының нақты құрылымы қандай, ОФД-ға қандай форматта жіберіледі — сканер баптауы соған тәуелді.
6. `[БОЛЖАМ]` **Чек ені**: 58мм (384 px) және 80мм (576 px) — екеуін де қолдау керек пе, әлде 80мм жеткілікті ме? Барлық міндетті реквизиттер 58мм-ге (32 таңба/жол) сыя ма? Бұл фискал талаптарынан шығады.

### 12.3 Ашық сұрақтар (өнім шешімі керек)

| # | Сұрақ | Кімге |
|---|---|---|
| Q1 | iOS MVP v1-ге кіре ме? (§9.2 бойынша — кірмеуі ұсынылады) | Өнім иесі |
| Q2 | Windows MVP v1-ге кіре ме? (R1+R2 шешілмесе — қауіпті) | Өнім иесі |
| Q3 | Тұтынушы дисплейі MVP-ге кіре ме? (кірмеуі ұсынылады) | Өнім иесі |
| Q4 | Тікелей қосылатын таразы MVP-ге кіре ме? (кірмеуі ұсынылады, тек салмақ штрихкоды) | Өнім иесі |
| Q5 | БД шифрлануы керек пе? (R11 — Windows-та қиын) | Қауіпсіздік + фискал |
| Q6 | ESC/POS генераторын өзіміз жазамыз ба? (R5 — иә деп ұсынылады) | Техлид |
| Q7 | Қай принтер модельдері «ресми қолдау» тізіміне кіреді? | Өнім + сату |

---

## 13. `[БҰҒАТТАЛҒАН]` тізімі

Бұлар зерттеумен шешілмейді — нақты темір, өндірушіден SDK, немесе KZ нарығымен байланыс керек.

### 13.1 Нақты құрылғы керек (сатып алу / қарызға алу)

| # | Не тексеріледі | Керекті темір | Приоритет | Бағаланған уақыт |
|---|---|---|---|---|
| B1 | **Windows-та HID сканер жұмыс істей ме** (R1) — `KeyboardListener` және `RegisterRawInputDevices` | Windows 11 ноут + USB 2D HID сканер | 🔴 №1 | 1 күн |
| B2 | **Қазақ әріптері растрда оқыла ма** (R3, R16) — 384 px және 576 px-те | Epson TM-T20III + XPrinter XP-80 + XP-58 | 🔴 №2 | 2 күн |
| B3 | **Растр басу уақыты** (R6) — Ethernet vs USB vs BT SPP vs BLE | Сол принтерлер + BT принтер | 🔴 №3 | 1 күн |
| B4 | **Epson-да KZ-1048 (page 53) шынымен басады ма** — `ESC t 53` + RK1048 байттары | Epson TM-T20III немесе TM-T88V | 🟠 | 0,5 күн |
| B5 | **Windows спулер RAW басуы** (R17) — драйвер байттарды бұрмалай ма | Windows + USB термопринтер | 🟠 | 0,5 күн |
| B6 | **drift + sqlite3 3.5.x Windows-та DLL-сіз жиналады ма** (§1.2) | Windows машина | 🟠 | 0,5 күн |
| B7 | **Data Matrix + GS сепараторы** (R4) — HID сканер оны сақтай ма | 2–3 түрлі 2D сканер + маркировкаланған тауар | 🟠 | 1 күн |
| B8 | **Android-та `flutter_libserialport`** `/dev/ttyUSB0`-ді аша ма (R9) | Android планшет + USB-serial адаптер | 🟡 | 0,5 күн |
| B9 | **Android батарея оптимизациясы** синхрондауды өлтіре ме (R10) — 24 сағаттық тест | Android 13/14 планшет | 🟡 | 2 күн (күту) |
| B10 | **`presentation_displays` жаңа Flutter-де жұмыс істей ме** (R8) | Android планшет + HDMI экран | 🟡 | 0,5 күн |
| B11 | **Ақша жәшігі** `ESC p` vs `DLE DC4` — қайсысы сенімдірек | RJ11 жәшік + принтер | 🟡 | 0,5 күн |
| B12 | **SUNMI ішкі принтер + сканер + LCD** — `sunmi_printer_plus` 4.1.1 жаңа құрылғыларда | SUNMI V2 Pro немесе T2 mini | 🟡 | 1 күн |
| B13 | **Bixolon / Rongta** KZ-1048 қолдай ма | SRP-330II, RP80 | 🟢 | 0,5 күн |

### 13.2 Өндірушіден құжаттама/SDK сұрау керек

| # | Не керек | Кімнен | Себеп |
|---|---|---|---|
| B14 | **SUNMI dual-screen SDK** құжаттамасы + Flutter wrapper бар ма | SUNMI (`docs.sunmi.com` бұғатталған) | Тұтынушы дисплейі |
| B15 | **SUNMI эквайринг Intent** сипаттамасы | SUNMI | Эквайринг зерттеушісімен ортақ |
| B16 | **PAX NeptuneLite / Nebula SDK** толық құжаттамасы, NDA керек пе | PAX (әдетте NDA талап етеді `[БОЛЖАМ]`) | PAX адаптері |
| B17 | **Posiflex** Android/Windows терминалдарының SDK-сы | Posiflex | Flutter пакеті мүлдем жоқ |
| B18 | **Epson ESC/POS Command Reference** толық PDF (page 53 кестесімен) | Epson (`download4.epson.biz` бұғатталған) | KZ-1048 байт кестесі |
| B19 | **KZ-1048 mapping кестесі** (0x80–0xFF → Unicode) | unicode.org / IANA (екеуі де бұғатталған) | Кодтауыш жазу үшін |
| B20 | **Қазақстанда қолданылатын таразылар** және олардың хаттамалары | KZ дилерлер | §7.3 |

### 13.3 Қазақстан нарығы бойынша

| # | Не тексеріледі | Приоритет |
|---|---|---|
| B21 | SUNMI-дің ҚР-дағы ресми дистрибьюторы, модельдері, бағасы, кепілдігі | 🟠 |
| B22 | PAX-тың ҚР-дағы жағдайы (банктер қандай терминал таратады?) | 🟠 |
| B23 | ҚР-да ең көп сатылатын чек принтерлері (XPrinter? Epson? Rongta?) | 🔴 — R3 шешімі осыған тәуелді |
| B24 | ҚР-да қолданылатын HID сканер модельдері | 🟡 |
| B25 | Posiflex ҚР-да сатыла ма | 🟢 |

### 13.4 Осы сессияда зерттелмеген (WebSearch лимиті)

- `flutter_foreground_task` / `flutter_background_service` пакеттерінің ағымдағы мәртебесі (R10 үшін керек)
- `flutter_secure_storage`-тың Windows (DPAPI) қолдауының жағдайы
- Bixolon чек принтерлерінің код беттері
- Rongta принтерлерінің толық спецификациясы
- CAS / Mettler Toledo хаттамаларының ресми құжаттамасы
- Windows-та `RegisterRawInputDevices` арқылы Flutter-де HID оқудың дайын мысалы бар ма
- ҚР маркировка Data Matrix форматының спецификациясы (фискал зерттеушісінің тақырыбы)

---

## 14. Дереккөздер тізімі

> Барлығы 2026-09-01 күні қаралды. `⛔` — егресс-проксимен бұғатталған, тек тақырып/іздеу сниппеті бойынша.

### 14.1 pub.dev пакет беттері

| Пакет | URL | Нұсқа | Соңғы жаңарту |
|---|---|---|---|
| drift | <https://pub.dev/packages/drift> | 2.34.3 | ≈35 күн |
| sqlite3 | <https://pub.dev/packages/sqlite3> | 3.5.2 | ≈12 күн |
| sqlite3_flutter_libs | <https://pub.dev/packages/sqlite3_flutter_libs> | 0.6.0+eol | ≈6 ай |
| sqlcipher_flutter_libs | <https://pub.dev/packages/sqlcipher_flutter_libs> | 0.7.0+eol | ≈6 ай |
| sqflite | <https://pub.dev/packages/sqflite> | 2.4.3 | ≈3 ай |
| sqlite_async | <https://pub.dev/packages/sqlite_async> | 0.14.5 | ≈4 күн |
| objectbox | <https://pub.dev/packages/objectbox> | 5.3.2 | ≈3 ай |
| isar_plus | <https://pub.dev/packages/isar_plus> | — | — |
| unified_esc_pos_printer | <https://pub.dev/packages/unified_esc_pos_printer> | 3.4.0 | ≈46 күн |
| flutter_thermal_printer | <https://pub.dev/packages/flutter_thermal_printer> | 2.2.2 | ≈6 күн |
| thermal_printer_flutter | <https://pub.dev/packages/thermal_printer_flutter> | 2.0.0+3 | ≈2 ай |
| print_bluetooth_thermal | <https://pub.dev/packages/print_bluetooth_thermal> | 1.2.2 | ≈3 ай |
| esc_pos_utils_plus | <https://pub.dev/packages/esc_pos_utils_plus> | 2.0.4 | ≈24 ай ⚠️ |
| esc_pos_printer | <https://pub.dev/packages/esc_pos_printer> | 4.1.0 | ≈4 жыл ⛔ |
| esc_pos_printer_plus | <https://pub.dev/packages/esc_pos_printer_plus> | 0.1.1 | ≈18 ай |
| blue_thermal_printer | <https://pub.dev/packages/blue_thermal_printer> | 1.2.3 | ≈4 жыл ⛔ |
| flutter_pos_printer_platform | <https://pub.dev/packages/flutter_pos_printer_platform> | 1.4.2 | ≈3 жыл ⛔ DISCONTINUED |
| flutter_esc_pos_network | <https://pub.dev/packages/flutter_esc_pos_network> | 1.0.3 | ≈2 жыл ⛔ GPL |
| nitro_printing | <https://pub.dev/packages/nitro_printing> | 0.0.6 | ≈2 күн |
| windows_escpos_engine | <https://pub.dev/packages/windows_escpos_engine> | 0.0.3 | ≈43 күн |
| image | <https://pub.dev/packages/image> | 4.9.2 | ≈13 күн |
| mobile_scanner | <https://pub.dev/packages/mobile_scanner> | 7.4.0 | ≈42 күн |
| google_mlkit_barcode_scanning | <https://pub.dev/packages/google_mlkit_barcode_scanning> | 0.16.1 | ≈14 күн |
| — BarcodeFormat enum | <https://pub.dev/documentation/google_mlkit_barcode_scanning/latest/google_mlkit_barcode_scanning/BarcodeFormat.html> | — | — |
| charset_converter | <https://pub.dev/packages/charset_converter> | 2.5.1 | ≈44 күн |
| charset_codec | <https://pub.dev/packages/charset_codec> | 0.1.1 | ≈34 күн |
| flutter_blue_plus | <https://pub.dev/packages/flutter_blue_plus> | 2.3.12 | ≈22 күн |
| win32 | <https://pub.dev/packages/win32> | 6.4.0 | ≈26 күн |
| kiosk_mode | <https://pub.dev/packages/kiosk_mode> | 0.9.0 | ≈33 күн |
| presentation_displays | <https://pub.dev/packages/presentation_displays> | 1.0.0 | ≈2 жыл ⚠️ |
| flutter_libserialport | <https://pub.dev/packages/flutter_libserialport> | 0.6.0 | ≈13 ай |
| usb_serial | <https://pub.dev/packages/usb_serial> | 0.5.2 | ≈2 жыл ⚠️ |
| digital_scale | <https://pub.dev/packages/digital_scale> | 1.2.3+1 | ≈12 күн (GPL) |
| sunmi_printer_plus | <https://pub.dev/packages/sunmi_printer_plus> | 4.1.1 | ≈12 ай |
| sunmi_scanner | <https://pub.dev/packages/sunmi_scanner> | 0.0.8 | ≈14 күн |
| sunmi_kali | <https://pub.dev/packages/sunmi_kali> | 1.3.1 | ≈2 ай |
| flutter_pax_printer_utility | <https://pub.dev/packages/flutter_pax_printer_utility> | 0.1.6 | ≈10 ай |
| nebula_flutter_plugin | <https://pub.dev/packages/nebula_flutter_plugin> | 1.0.3 | ≈4 ай |
| drift API docs | <https://pub.dev/documentation/drift/latest/> | — | — |
| DriftIsolate | <https://pub.dev/documentation/drift/latest/isolate/DriftIsolate-class.html> | — | — |

### 14.2 GitHub

- `escpos-printer-db` — <https://github.com/receipt-print-hq/escpos-printer-db>
  - `data/encoding.yml` (RK1048 бар) — <https://github.com/receipt-print-hq/escpos-printer-db/blob/master/data/encoding.yml>
  - `data/profile/TM-T70II.yml` (`53: RK1048 # Kazakhstan`), сондай-ақ TM-T20II, TM-T20III, TM-T88II, TM-T88V, TM-T70II-SA, TM-m30III, TM-L90, default.yml
- `win32` RAW басу мысалы — <https://github.com/halildurmus/win32/blob/main/examples/printer_raw.dart>
- `unified_esc_pos_printer` — <https://github.com/elrizwiraswara/unified_esc_pos_printer>
- flutter/flutter **#79849** (Windows HID сканер, АШЫҚ) — <https://github.com/flutter/flutter/issues/79849>
- flutter/flutter **#154141** (HardwareKeyboard глобалды тыңдау) — <https://github.com/flutter/flutter/issues/154141>
- flutter/flutter **#162305** (KeyboardListener KeyUpEvent багы, web) — <https://github.com/flutter/flutter/issues/162305>
- flutter/flutter **#136419** (RawKeyEvent deprecation) — <https://github.com/flutter/flutter/issues/136419>
- drift **#3395** (SQLCipher Windows OpenSSL) — <https://github.com/simolus3/drift/issues/3395>
- drift **#2748** (MultiExecutor, WAL, изоляттар) — <https://github.com/simolus3/drift/discussions/2748>
- isar **#1689** «Isar is dead» — <https://github.com/isar/isar/issues/1689>
- isar **#1581** «Is this project still alive?» — <https://github.com/isar/isar/discussions/1581>
- realm-js **#6884** (Device Sync deprecation, EOL 2025-09-30) — <https://github.com/realm/realm-js/discussions/6884>
- realm-swift **#8680** «Realm is Deprecated/Dead» — <https://github.com/realm/realm-swift/discussions/8680>
- `code_scan_listener` (HardwareKeyboard HID) — <https://github.com/utamori/code_scan_listener>
- `esc_pos_utils` enums (drawer pins) — <https://github.com/andrey-ushakov/esc_pos_utils/blob/master/lib/src/enums.dart>

### 14.3 Өндіруші / стандарт құжаттамасы

- ⛔ Epson «Page 53 [KZ-1048: Kazakhstan]» — <https://download4.epson.biz/sec_pubs/pos/reference_en/charcode/page_53.html>
- ⛔ Epson Character Code Tables — <https://download4.epson.biz/sec_pubs/pos/reference_en/charcode/supported_codepage.html>
- ⛔ Epson `ESC t` — <https://download4.epson.biz/sec_pubs/pos/reference_en/escpos/esc_lt.html>
- ⛔ Epson `ESC p` (ақша жәшігі) — <https://download4.epson.biz/sec_pubs/pos/reference_en/escpos/esc_lp.html>
- ⛔ Epson TM-T20III supported commands — <https://download4.epson.biz/sec_pubs/pos/reference_en/escpos/tmt20iii.html>
- Epson TM-T20 ESC/POS Quick Reference (PDF) — <http://www.novopos.ch/client/EPSON/TM-T20/TM-T20_eng_qr.pdf>
- ⛔ IANA KZ-1048 тіркеуі — <https://www.iana.org/assignments/charset-reg/KZ-1048>
- ⛔ Unicode KZ1048.TXT mapping — <https://www.unicode.org/Public/MAPPINGS/VENDORS/MISC/KZ1048.TXT>
- Compart KZ-1048 — <https://www.compart.com/en/unicode/charsets/KZ-1048>
- fileformat.info KZ-1048 — <https://www.fileformat.info/info/charset/KZ-1048/index.htm>
- Apple External Accessory (MFi) — <https://developer.apple.com/documentation/externalaccessory>
- ⛔ Flutter key event migration — <https://docs.flutter.dev/release/breaking-changes/key-event-migration>
- Flutter `RawKeyboardListener` API — <https://api.flutter.dev/flutter/widgets/RawKeyboardListener-class.html>
- ⛔ SUNMI developer docs — <https://docs.sunmi.com/en/>
- Xprinter XP-80T нұсқаулығы (код беттері) — <https://manuals.plus/xprinter/xp-80t-thermal-receipt-printer-manual>
- Xprinter XP-T58 нұсқаулығы — <https://manuals.plus/m/4e2a3a70a8704029d0c80f079e49413c2ab2650213dbff9786d2c2152aeb49ea>
- Bixolon Label Printer Code Pages (PDF) — <https://www.bixolon.com/_upload/manual/Manual_Label_Printer_Code_Pages_english_V2.02.pdf>
- Diebold Nixdorf P1200 Programming Manual (PDF) — <https://www.dieboldnixdorf.com/-/media/diebold/files/retail/peripherals-en/printers/p1200-prog-manual.pdf>

### 14.4 Жанама (блог, форум, дилер)

- ⛔ mike42 «How to print the characters in an ESC/POS printer code page» — <https://mike42.me/blog/2018-05-how-to-print-the-characters-in-an-esc-pos-printer-code-page>
- «The Flutter Local Database Landscape in 2026» — <https://luci-studio.com/blog/the-flutter-local-database-landscape-in-2026-a-maintenance-first-guide-fe6d267c/>
- ⛔ drift Isolates құжаттамасы — <https://drift.simonbinder.eu/isolates/>
- ⛔ drift Encryption құжаттамасы — <https://drift.simonbinder.eu/platforms/encryption/>
- Честный знак — Data Matrix сканерін тексеру — <https://markirovka.ru/knowledge/fast_start/start/proverka-skanera-data-matrix-instruktsiya>
- POS-Center — сканер Data Matrix оқи ма — <https://pos-center.ru/journal/podhodit-li-skaner-shtrih-kodov-dlya-schityvaniya-data-matrix/>
- Mertech — SUNMI L2S сканерін баптау — <https://help.mertech.ru/tsd/Mertech_Sunmi/nastroyka_L2S.html>
- Клеверенс — SUNMI L2K сипаттамасы — <https://www.cleverence.ru/hardware/mdc/sunmi/sunmi-l2k/4085/>
- Wikipedia Windows-1251 — <https://en.wikipedia.org/wiki/Windows-1251>
- Wikipedia Kazakh Short U (Ұұ, KZ-1048-те 0xA2/0xA3) — <https://en.wikipedia.org/wiki/Kazakh_Short_U>
- DantSu ESCPOS-ThermalPrinter-Android issue #185 (CP866-да қытай иероглифтері) — <https://github.com/DantSu/ESCPOS-ThermalPrinter-Android/issues/185>
- escpos-printer-db issue #36 (XPrinter XP-58 профилін қосу) — <https://github.com/receipt-print-hq/escpos-printer-db/issues/36>
- QZ Tray Raw Encoding — <https://qz.io/docs/raw-encoding>

---

*Есеп соңы. Фаза 1 зерттеу материалы. Ешбір тұжырым нақты темірде тексерілмеген — §13-ті орындамай тұрып «жасалды» деп есептеуге болмайды.*
