# 06 — Темір және платформа

> Зерттеу күні: **2026-09-01**. Фаза 1, research субагент.
> Тақырып: Flutter кассасының темірмен және платформалармен байланысы.
> Бұл құжатта КОД ЖОҚ — тек зерттеу мен инженерлік бағалау.

---

## 0. Әдістеме және шектеулер

### 0.1 Таңбалау тәртібі

| Белгі | Мағынасы |
|---|---|
| `[РАСТАЛҒАН]` | pub.dev пакет беті / pub.dev API (нұсқа + нақты жариялану күні), GitHub репозиторийдің шикі файлы, GitHub issue, ресми құжаттама. URL міндетті |
| `[ЖАНАМА]` | Блог, форум, дилер сайты, іздеу сниппеті, үшінші тарап мақаласы |
| `[БОЛЖАМ]` | Менің инженерлік қорытындым, тікелей дереккөз жоқ |
| `[БҰҒАТТАЛҒАН]` | Нақты құрылғы / өндірушіден SDK / KZ дилерімен байланыс керек |

### 0.2 Дереккөз сапасы — не расталды, не расталмады

**Толық расталды (бастапқы дереккөз, машиналық оқылған):**
- **pub.dev API** — `https://pub.dev/api/packages/<name>` арқылы **әр пакеттің нақты жариялану күні** (шамамен емес, `YYYY-MM-DD`), `isDiscontinued` жалауы, pubspec-тегі `flutter.plugin.platforms` тізімі.
- **pub.dev score API** — `https://pub.dev/api/packages/<name>/score` арқылы платформа тегтері, лицензия тегі, likes, 30 күндік жүктеу саны.
- **GitHub raw** — `raw.githubusercontent.com` арқылы `escpos-printer-db` дерекқорының шикі YAML/JSON файлдары толық жүктеліп, жергілікті талданды (50 принтер профилі, 188 кодтау).
- **GitHub HTML** — issue беттері (`github.com/.../issues/...`).

**Бұғатталған (егресс-прокси, 403 — қайталаудың мәні жоқ):**

| Домен | Не жоғалды |
|---|---|
| `docs.flutter.dev`, `api.flutter.dev` | Flutter supported-platforms, key-event migration ресми беттері |
| `drift.simonbinder.eu` | drift setup, encryption, isolates құжаттамасы |
| `download4.epson.biz` | Epson ESC/POS Command Reference, Character Code Tables (page 53 = KZ-1048) |
| `www.unicode.org`, `www.iana.org` | KZ-1048 mapping кестесі (0x80–0xFF → Unicode) және IANA тіркеуі |
| `docs.sunmi.com`, `sunmi.kz` | SUNMI developer порталы (dual-screen SDK, эквайринг Intent) |
| `paxtechnology.com` | PAX SDK құжаттамасы |
| `developers.google.com` | ML Kit barcode формат тізімі (pub.dev API docs арқылы айналып өттім) |
| `mike42.me`, wikipedia, stackoverflow, medium | ESC/POS code page түсіндірмелері |

**Тағы бір шектеу:** сессияның **WebSearch лимиті таусылды** (200/200, басқа субагенттермен ортақ). Зерттеудің екінші жартысы тек WebFetch + `curl` арқылы жүрді. Сол себепті Қазақстан нарығы (дилерлер, бағалар) және таразы хаттамалары ашылмады → §13.

**Ең маңызды шектеу:** ешбір нақты құрылғыда тест жүргізілген жоқ. Барлық «жұмыс істейді» деген тұжырым — құжаттамаға сүйенген. §13-ті орындамай тұрып ешнәрсені «жасалды» деп есептеуге болмайды.

### 0.3 pub.dev платформа белгілері туралы ескерту

`[РАСТАЛҒАН]` **pub.dev беттеріндегі «Platform: Android iOS Windows Linux macOS Web» белгілері жаңылыстырады.**
Мысал: `unified_esc_pos_printer` бетінде алты платформа да көрсетіледі, бірақ оның `pubspec.yaml`-ындағы `flutter.plugin.platforms` тізімінде **тек `android`, `ios`, `windows`** бар. Linux/macOS/Web тегтері Dart кодының сол платформаларда **компиляциялана алатынынан** шығады, нативті имплементация бар дегеннен емес.

Сол себепті бұл есептегі әр пакет үшін **екі бағана** беріледі:
- **«pubspec plugin platforms»** — нақты нативті имплементация бар платформалар (шындық)
- **«pub.dev тегтері»** — pub.dev көрсететін тегтер (жарнама)

---

## 1. Flutter + drift (SQLite)

### 1.1 Дәл сандар (pub.dev API, 2026-09-01)

| Пакет | Нұсқа | **Жарияланған** | Барлық нұсқа | Discontinued | Лицензия | Likes | 30к жүктеу |
|---|---|---|---|---|---|---|---|
| **drift** | 2.34.3 | **2026-07-27** | 75 (2021-10-11-ден) | жоқ | MIT | 2458 | 1 141 183 |
| **sqlite3** | 3.5.2 | **2026-08-19** | 101 (2019-11-26-дан) | жоқ | MIT | 463 | 2 314 897 |
| **sqlite3_flutter_libs** | 0.6.0+eol | **2026-02-15** | 55 | жоқ (бірақ EOL) | MIT | — | — |
| **sqlcipher_flutter_libs** | 0.7.0+eol | **2026-02-15** | 17 | жоқ (бірақ EOL) | MIT | — | — |
| **sqflite** | 2.4.3 | **2026-06-02** | 173 | жоқ | — | — | — |
| **sqlite_async** | 0.14.5 | **2026-08-27** | 46 | жоқ | — | — | — |
| **objectbox** | 5.3.2 | **2026-05-20** | 157 | жоқ | Apache-2.0 | 1577 | 150 417 |
| **isar** | 3.1.0+1 | **2023-04-25** ⛔ | 59 | жоқ (бірақ тастанды) | — | — | — |
| **isar_plus** (форк) | 1.3.9 | **2026-08-04** | 58 (2025-09-17-ден) | жоқ | — | — | — |
| **realm** | 20.2.0 | **2025-09-24** ⛔ | 54 | жоқ (бірақ EOL) | — | — | — |

`[РАСТАЛҒАН]` Барлық жол `https://pub.dev/api/packages/<name>` жауабынан алынды (`latest.version`, `latest.published`, `versions[]`, `isDiscontinued`) және `https://pub.dev/api/packages/<name>/score` (`likeCount`, `downloadCount30Days`, `tags`).

**drift тегтері** `[РАСТАЛҒАН]`: `is:flutter-favorite`, `platform:android/ios/windows/linux/macos/web`, `is:wasm-ready`, `is:dart3-compatible`, `license:mit`. Pana ұпайы **160/160**.
**objectbox тегтері** `[РАСТАЛҒАН]`: `platform:android/ios/windows/linux/macos` — **Web ЖОҚ**, `license:apache-2.0`, **160/160**.
**sqlite3 тегтері** `[РАСТАЛҒАН]`: алты платформа да, 150/160.

### 1.2 Isar және Realm — мәртебесі pub.dev-тен расталды

`[РАСТАЛҒАН]` **Isar-дың соңғы релизі — 3.1.0+1, 2023-04-25.** Бүгінгі күні (2026-09-01) бұл **3 жыл 4 ай** бұрын. Пакет `isDiscontinued: false` деп тұр (яғни автор оны ресми түрде тоқтатпаған), бірақ **3+ жыл жаңармауы — іс жүзінде тастанды**. Форк `isar_plus` 2025-09-17-де басталып, соңғы релизі **2026-08-04 (1.3.9)** — форк тірі.
<https://pub.dev/api/packages/isar> · <https://pub.dev/api/packages/isar_plus> · <https://github.com/isar/isar/issues/1689> («Isar is dead, long live Isar»)

`[РАСТАЛҒАН]` **Realm-нің соңғы релизі — 20.2.0, 2025-09-24.** `[ЖАНАМА]` MongoDB 2024-09-09-да Atlas Device Sync + Realm SDK-ларын deprecated деп жариялады, **end-of-life 2025-09-30**. Яғни соңғы релиз EOL күнінен алты күн бұрын шыққан және содан бері жаңару жоқ.
<https://pub.dev/api/packages/realm> · <https://github.com/realm/realm-js/discussions/6884> · <https://github.com/realm/realm-swift/discussions/8680>

`[БОЛЖАМ]` Екеуі де жаңа жобаға жарамайды. Isar-дың `isDiscontinued` жалауы `false` болғаны шатастырады — **pub.dev-тің «discontinued» белгісіне ғана сүйенуге болмайды, жариялану күнін қарау керек**.

### 1.3 Windows-тағы DLL мәселесі — ЖАБЫЛҒАН

`[РАСТАЛҒАН]` `sqlite3` **3.5.2** (2026-08-19) бетінде: *«Because this library uses hooks, it bundles SQLite with your application and doesn't require any external dependencies or build configuration.»*
Дайын (prebuilt) SQLite:
- Android: armv7a, aarch64, x86, x64
- iOS: arm64 (құрылғы + симулятор), x64 (симулятор)
- **Windows: aarch64, x64, x86**
<https://pub.dev/packages/sqlite3>

`[РАСТАЛҒАН]` `sqlite3_flutter_libs` **0.6.0+eol** (2026-02-15) — **ресми EOL**. Беттегі мәтін: *«This package relates to version 2.x of `package:sqlite3`, and is obsolete after upgrading»*. 0.6.0-дан бастап ол ешқандай build функциясын бермейді, тек ескі 0.5.x build скрипттерін қолданудан сақтау үшін тұр.
<https://pub.dev/packages/sqlite3_flutter_libs>

`[БОЛЖАМ]` **Практикалық қорытынды:** жаңа жобаға `sqlite3_flutter_libs`-ті **МҮЛДЕ қоспау**. `drift` + `sqlite3 ^3.5` жеткілікті. Тарихи «Windows-та sqlite3.dll табылмайды» мәселесі 2026 жылы жоқ. Бірақ бұл әлі нақты Windows машинасында тексерілмеген → §13 B6.

### 1.4 Шифрлау (SQLCipher)

`[РАСТАЛҒАН]` `sqlcipher_flutter_libs` **0.7.0+eol** (2026-02-15) — бұл да **EOL**, сол себеппен.
<https://pub.dev/packages/sqlcipher_flutter_libs>

`[РАСТАЛҒАН]` Windows-та SQLCipher-ді жинау ауыр болған: drift issue **#3395**, ашылған **2025-01-02**. CMake `FindOpenSSL` модулі OpenSSL 1.1.1 / 3.0 / 3.0.15 / 3.4.0 нұсқаларының ешқайсысында `OPENSSL_CRYPTO_LIBRARY`-ді таппайды; drift-тің ресми мысалы да сол қатемен құлайды. Issue **жабылған**, бірақ `docs` белгісімен — кодта түзетілмей, құжаттамамен «шешілген».
<https://github.com/simolus3/drift/issues/3395>

`[РАСТАЛҒАН]` `flutter_secure_storage` **11.0.0**, **2026-08-06**, pubspec plugin platforms: **android, ios, linux, macos, web, windows** — алтауы да нативті.
<https://pub.dev/api/packages/flutter_secure_storage>

`[БОЛЖАМ]` **Ұсыныс:** MVP-де БҮКІЛ БД-ны SQLCipher-мен шифрламау:
1. Құпияларды (ОФД токендері, эквайринг кілттері) `flutter_secure_storage` арқылы бөлек сақтау — ол Windows-та да нативті (DPAPI).
2. Чектердің өзгермеуін БД шифрлауымен емес, **оқиға тізбегіндегі hash chain** арқылы қамтамасыз ету (event sourcing принципіне сай, §5 CLAUDE.md).
3. SQLCipher — кейінгі фаза, Windows-та тексергеннен кейін.
→ `/docs/CONTRACTS.md`-те шешім ретінде бекітілсін.

### 1.5 Миграциялар, транзакциялар, WAL, изоляттар

`[ЖАНАМА]` drift-тің изолят моделі: бір **server isolate** барлық сұранымдарды орындап, кесте өзгерістерін broadcast жасайды; кез келген саны **client isolate** оған қосылады. Сұраным client-те құрастырылып, шикі SQL + параметрлер server-ге жіберіледі. Фондық изолят UI jank-ті азайтады, деректерді көшіру шығыны Dart VM isolate groups арқасында аз.
<https://pub.dev/documentation/drift/latest/isolate/DriftIsolate-class.html> (drift.simonbinder.eu БҰҒАТТАЛҒАН)

`[ЖАНАМА]` Балама — әр изолятта БД-ны дербес ашу; онда stream query-лер синхрондалмайды және lock-тардан қашу үшін `journal_mode=WAL` + `busy_timeout` керек.
<https://github.com/simolus3/drift/discussions/2748>

`[БОЛЖАМ]` **POS үшін ұсынылатын архитектура:**
- БД **бір фондық изолятта** (`DriftIsolate`), UI тек клиент.
- `PRAGMA journal_mode=WAL` + `PRAGMA busy_timeout=5000` міндетті.
- **Sync worker бөлек изолятта**, ол да DriftIsolate клиенті.
- Миграция: drift `MigrationStrategy` + `drift_dev schema` dump-тері; фискалдық БД болғандықтан әр миграцияға regression тест міндетті.

`[БОЛЖАМ]` **Өнімділік мәселе емес.** 1–3 жұмыс орны, күніне ~500–2000 чек — SQLite үшін түк те емес. Ең қауіпті түйін өнімділік емес, **WAL + көп изолят + идемпотенттілік дұрыстығы**.

### 1.6 §1 қорытынды

`[БОЛЖАМ]` **drift + sqlite3 3.5.x — дұрыс таңдау, өзгертудің қажеті жоқ.** Бұл жобадағы жалғыз шынымен төмен тәуекелді темірге қатысты шешім (§9.1 кестесіндегі жалғыз толық жасыл жол). ObjectBox — жалғыз шынайы балама (160/160, 1577 likes, Windows ✅), бірақ ол SQL емес — event sourcing үшін SQL қолайлырақ.

---

## 2. Термопринтерлер және ESC/POS

### 2.1 Қосылу тәсілдері және платформалық шындық

| Қосылу | Android | Windows | iOS | Ескерту |
|---|---|---|---|---|
| **Ethernet / Wi-Fi (TCP 9100)** | ✅ | ✅ | ✅ | Ең әмбебап. Тек `dart:io` Socket — плагин де қажет емес |
| **USB** | ✅ USB Host | ✅ спулер RAW | ❌ | iOS-та USB жоқ |
| **Bluetooth Classic (SPP)** | ✅ | ✅ RFCOMM | ❌ | iOS: MFi сертификаты керек |
| **BLE** | ✅ | ⚠️ бөлек форк | ✅ | Растр басу үшін тым баяу |
| **Ішкі принтер (SUNMI/PAX)** | ✅ AIDL | — | — | Тек сол темірде |

`[РАСТАЛҒАН]` Көптеген ESC/POS принтерлері әдепкіде **9100 портын** тыңдайды.
<https://pub.dev/packages/esc_pos_printer>

`[БОЛЖАМ]` **Ең маңызды инженерлік ұсыныс:** негізгі транспорт — **Ethernet/Wi-Fi (TCP 9100)**. Ол үш платформада да бірдей `Socket.connect(ip, 9100)` арқылы жұмыс істейді, ешқандай плагинді, нативті кодты, рұқсатты қажет етпейді. Бұл «hardware-agnostic» принципіне ең сай транспорт. USB мен Bluetooth — қосымша адаптерлер.

### 2.2 Windows-та ESC/POS басу — шешім табылды

`[РАСТАЛҒАН]` Windows-та ESC/POS-ты **жүйелік спулер арқылы RAW режимде** жіберуге болады. `package:win32` репозиторийінде дәл осының ресми мысалы бар: `examples/printer_raw.dart`. Тізбек:

`OpenPrinter → StartDocPrinter → StartPagePrinter → WritePrinter → EndPagePrinter → EndDocPrinter → ClosePrinter`

`datatype = 'RAW'` — драйвердің форматтау конвейерін толық айналып өтеді. Мысалда ақша жәшігін ашу командасы (`\x1b\x70\x00` = `ESC p`) EPSON термопринтеріне жіберіледі.
<https://github.com/halildurmus/win32/blob/main/examples/printer_raw.dart>

`[РАСТАЛҒАН]` `win32` **6.4.0**, **2026-08-05**. `packages/win32/lib/src/win32/winspool.g.dart` ішінде winspool API-лары генерацияланған.
<https://pub.dev/api/packages/win32>

`[БОЛЖАМ]` Демек Windows-та сұрақ «тікелей порт па, әлде драйвер ме» емес. Дұрыс жауап: **өндірушінің Windows драйверін орнатып, принтерді жүйеде тіркеп, оған RAW байт жіберу**. Бұл USB, Ethernet және COM-порт принтерлерінің үшеуіне де бір код жолын береді және Windows-та USB-ге тікелей қол жеткізудің (WinUSB/libusb драйверін ауыстыру) азабынан құтқарады.

### 2.3 Flutter пакеттері — толық аудит (pub.dev API, нақты күндермен)

| Пакет | Нұсқа | **Жарияланған** | pubspec plugin platforms | Лиц. | Likes | 30к жүктеу | Мәртебе |
|---|---|---|---|---|---|---|---|
| **unified_esc_pos_printer** | 3.4.0 | **2026-07-16** | `android, ios, windows` | BSD-3 | 9 | 1 637 | **Тірі, 160/160, ең толық** |
| **flutter_thermal_printer** | 2.2.2 | **2026-08-25** | `android, ios, macos, windows` | BSD-3 | 137 | 6 419 | Тірі, 150/160 |
| **thermal_printer_flutter** | 2.0.0+3 | **2026-06-08** | `android, ios, linux, macos, web, windows` | Apache-2.0 | 9 | 14 | Тірі, 135/160, жүктеу өте аз |
| **print_bluetooth_thermal** | 1.2.2 | **2026-05-23** | `android, ios, macos, windows` | BSD-3 | 167 | **36 730** | Тірі, 160/160 |
| **nitro_printing** | 0.0.6 | **2026-08-29** | `android, ios, linux, macos, web, windows` | MIT | 9 | 123 | Жаңа, **0.0.x — шикі** |
| **windows_escpos_engine** | 0.0.3 | **2026-07-19** | `windows` | MIT | 0 | 9 | Жаңа, **0.0.x — шикі** |
| **image** (растрлау) | 4.9.2 | **2026-08-19** | (таза Dart) | MIT | — | — | Тірі |
| **esc_pos_utils_plus** | 2.0.4 | **2024-09-01** ⚠️ | (таза Dart) | BSD-3 | 123 | **50 951** | ⚠️ **дәл 2 жыл жаңармаған** |
| **esc_pos_printer_plus** | 0.1.1 | **2025-02-11** | (таза Dart) | BSD-3 | 21 | — | Тоқтап тұр |
| **flutter_esc_pos_network** | 1.0.3 | **2024-05-03** ⚠️ | — | ⛔ **GPL-3.0** | 30 | — | Тастанды + лицензия қаупі |
| **esc_pos_printer** | 4.1.0 | **2021-09-21** ⛔ | — | BSD-3 | — | — | ⛔ **5 жыл** |
| **blue_thermal_printer** | 1.2.3 | **2022-07-19** ⛔ | `android` | — | — | — | ⛔ **4 жыл**, Android-only |
| **flutter_pos_printer_platform** | 1.4.2 | **2023-07-14** ⛔ | — | MIT | — | — | ⛔ **`isDiscontinued: true`** |

`[РАСТАЛҒАН]` Барлық күн `pub.dev/api/packages/<name>` → `latest.published`; `isDiscontinued` сол жауаптан; тегтер мен likes `pub.dev/api/packages/<name>/score`-дан.

**⛔ ЕКІ ҚАТАҢ ЕСКЕРТУ:**

1. **Лицензия.** `flutter_esc_pos_network` (және `flutter_esc_pos_network_universal` форкі) — **GPL-3.0**. Жабық кодты коммерциялық POS-қа оны қосу copyleft жұқтыру тәуекелін тудырады. **Қолданбау.** Қалғандары BSD-3 / MIT / Apache-2.0 — қауіпсіз.
2. **`esc_pos_utils_plus` — уақыт бомбасы.** Ол **2024-09-01-де** жарияланған (бүгін дәл 2 жыл), бірақ айына **50 951 рет** жүктеледі және `flutter_thermal_printer`, `thermal_printer_flutter` сияқты тірі пакеттердің **транзитивті тәуелділігі**. Яғни оны тікелей қоспасақ та, ол жобаға кіреді. → R12.

### 2.4 unified_esc_pos_printer — егжей-тегжейлі

`[РАСТАЛҒАН]` **3.4.0**, **2026-07-16**, verified publisher `elriztechnology.com`, BSD-3, pana **160/160**.
pubspec-тегі нақты нативті платформалар: **`android`, `ios`, `windows`** (Linux/macOS/Web тегтері — тек Dart компиляциясынан, §0.3 қара).

README-дегі қосылу матрицасы `[РАСТАЛҒАН]`:

| Қосылу | Android | iOS | Windows | Linux | macOS |
|---|---|---|---|---|---|
| Network (TCP/IP) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Bluetooth Classic | ✅ | ❌ | ✅ | ❌ | ❌ |
| BLE | ✅ | ✅ | ✅ | ❌ | ❌ |
| USB | ✅ (OTG) | ❌ | ✅ | ✅ (serial) | ✅ (serial) |

Мүмкіндіктер: **200+ дайын принтер профилі** code page карталарымен, QR (8 өлшем, 4 EC деңгейі), штрихкодтар (UPC-A/E, EAN-13/8, CODE39, ITF, CODABAR, CODE128), ақша жәшігі (Pin 2 / Pin 5), суретті басу (`ESC *`, `GS v 0`, `GS ( L`), және ең маңыздысы — **text rasterization**.
<https://github.com/elrizwiraswara/unified_esc_pos_printer>

`[РАСТАЛҒАН]` Репозиторийде тек **2 ашық issue** бар (#25 «having children in raster column?», 2026-07-17; #23 «PrintColumn bold style is ignored in ticket.row()», 2026-07-16). **Windows, кириллица, кодтау немесе растр өнімділігі туралы ашық issue ЖОҚ.**
<https://github.com/elrizwiraswara/unified_esc_pos_printer/issues>

`[БОЛЖАМ]` Тәуекел: **9 likes, айына 1637 жүктеу** — бұл жалғыз әзірлеушінің жеке жобасы. Сапасы жоғары (160/160), бірақ автобус факторы = 1. → R5.

### 2.5 Принтер модельдері — `escpos-printer-db` дерекқорының толық талдауы

`[РАСТАЛҒАН]` `receipt-print-hq/escpos-printer-db` дерекқорының `dist/capabilities.json` файлы толық жүктеліп (133 КБ), жергілікті талданды. Нәтиже:

- **Барлығы 50 принтер профилі**
- **RK1048 (KZ-1048, page 53) қолдайтын профильдер — 15:**
  `TM-L90, TM-P80, TM-P80-42col, TM-T20II, TM-T20II-42col, TM-T20III, TM-T70II, TM-T70II-CH, TM-T70II-CH-58mm, TM-T70II-SA, TM-T88II, TM-T88V, TM-m30III` (барлығы **Epson**), **`TEP-200M`** (vendor: EPOS), және `default`.
- **Кириллица (CP866/CP1251) бар, бірақ RK1048 ЖОҚ — 28 профиль:**
  `CT-S651` (Citizen), `ITPP047`, `KR-306`, `NT-5890K`, `NT-80-V-UL`, `OCD-100`, `OCD-300`, `P822D`, `POS-5890`, `RP-F10-58mm`, `RP-F10-80mm`, `RP326`, `RP332` (Rongta), `SP2000`, **`SRP-S300` (Bixolon)**, **`Sunmi-V2` (SUNMI!)**, `TH230`, `TH230Plus`, `TM-T70`, `TM-T88III`, `TM-T88IV`, `TSP600`, `TSP800`, `TUP500` (Star), `YHD-8390`, `ZJ-5870`, `mcPrint` (Star), `safe`.

<https://github.com/receipt-print-hq/escpos-printer-db> · `dist/capabilities.json`

**Нақты профильдер (шикі JSON-нан):**

| Профиль | Vendor | Кириллица код беттері | RK1048 | Қағаз |
|---|---|---|---|---|
| **TM-T20III** | Epson | 17:CP866, 34:CP855, 44:CP1125, 46:CP1251 | ✅ **53** | 80мм = **576 px**, 203 dpi |
| **TEP-200M** | EPOS | 17:CP866, 34:CP855, 44:CP1125, 46:CP1251 | ✅ **53** | белгісіз |
| **Sunmi-V2** | Sunmi | **17:CP866, 34:CP855, 254:CP855** | ❌ **ЖОҚ** | 57,5мм = **384 px** |
| **SRP-S300** | Bixolon | 17:CP866, 34:CP855 | ❌ **ЖОҚ** | 80мм = **640 px**, 203 dpi |
| **RP326** | Rongta | **6:CP1251, 7:CP866**, 28:CP855 | ❌ **ЖОҚ** | белгісіз |

**Осы кестеден шығатын ҮШ КРИТИКАЛЫҚ қорытынды:**

`[РАСТАЛҒАН]` **(1) SUNMI V2-нің ішкі принтерінде CP1251 де, RK1048 да ЖОҚ.** Тек CP866 және CP855. Яғни SUNMI-де қазақ әріптерін мәтіндік режимде басу **мүмкін емес** — тіпті орыс кириллицасы тек CP866 арқылы. Бұл жобаның ең маңызды өнім тәуекелдерінің бірі, өйткені SUNMI — POS сегментіндегі ең кең таралған Android темірі.

`[РАСТАЛҒАН]` **(2) Rongta-да код беті нөмірлері МҮЛДЕМ БАСҚА.** Epson-да CP866 = **17**, Rongta RP326-да CP866 = **7**, ал CP1251 = **6** (Epson-да 46). Егер кодта `ESC t 17` деп қатты жазылса, Rongta-да мүлдем басқа таңбалар шығады. **Code page нөмірі әрқашан профильден алынуы керек, ешқашан hardcode жасалмауы керек.**

`[РАСТАЛҒАН]` **(3) 80мм-де пиксель саны бірдей емес.** Epson TM-T20III → **576 px**, Bixolon SRP-S300 → **640 px** (екеуі де 203 dpi, екеуі де «80мм»). Растрлау кезінде ені **әрқашан профильден** алынуы керек, `576` деп қатты жазуға болмайды.

`[ЖАНАМА]` **XPrinter (XP-58 / XP-80)** — дерекқорда **профилі жоқ** (issue #36 «Add printer to database Xprinter XP-58» ашылған, бірақ біріктірілмеген). Өндіруші нұсқаулығы бойынша XP-80T қолдайтын код беттері: PC437, Katakana, PC850, PC860, PC863, PC865, West Europe, Greek, Hebrew, East Europe, Iran, WPC1252, **PC866**, PC852, PC858, IranII, Latvian, Arabic, **PT151 (=1251)**. **KZ-1048 ЖОҚ.**
<https://manuals.plus/xprinter/xp-80t-thermal-receipt-printer-manual> · <https://github.com/receipt-print-hq/escpos-printer-db/issues/36>

---

## 3. Қазақ әріптерін басу мәселесі (raster шешімі)

> **Есептің ең маңызды бөлімі.** Жобаның №2 дифференциациясына («толыққанды қазақ тілі») тікелей қатысты.

### 3.1 Мәселенің мәні

`[ЖАНАМА]` ESC/POS принтерлері мәтінді Unicode-пен емес, **бір байттық код беттерімен** басады: `ESC t n` командасы 128–255 диапазонын басқа 128 таңбаға ауыстырады. Бір мезетте принтерге тек 128 «қосымша» глиф қолжетімді.

Қазақ әліпбиінде орыс кириллицасынан тыс **9 әріп**: **ә ғ қ ң ө ұ ү һ і** (+ бас әріптері) — барлығы 18 глиф.

### 3.2 Код беттері бойынша талдау

| Код беті | Epson-дағы `ESC t n` | Не бар | Қазақ әріптері |
|---|---|---|---|
| **CP866** (Cyrillic #2, DOS) | 17 | Орыс кириллицасы | `[БОЛЖАМ]` **ЖОҚ** |
| **CP855** (Cyrillic) | 34 | Орыс/оңт.славян | `[БОЛЖАМ]` **ЖОҚ** |
| **CP1125** | 44 | Украин DOS | `[БОЛЖАМ]` **ЖОҚ** |
| **CP1251 / WPC1251 / PT151** | 46 | Орыс + украин/белорус/серб | `[БОЛЖАМ]` тек **і/І**; ә ғ қ ң ө ұ ү һ **ЖОҚ** |
| **CP775** (Baltic) | 33 | Тек латын | ❌ кириллица мүлдем жоқ |
| **KZ-1048 / RK1048** | **53** | CP1251-дің қазақша модификациясы | ✅ **БАРЛЫҒЫ БАР** |

`[ЖАНАМА]` **KZ-1048** (aliases: **RK1048**, **STRK1048-2002**) — ҚР Экономика және сауда министрлігінің Стандарттау комитеті жасаған, **2002-02-07**-де жарияланған ұлттық стандарт; Windows-1251-дің модификациясы; IANA-да KZ-1048 деп тіркелген.
<https://www.compart.com/en/unicode/charsets/KZ-1048> · <https://www.fileformat.info/info/charset/KZ-1048/index.htm>
(⛔ `www.iana.org` және `www.unicode.org` бұғатталған — mapping кестесі алынбады, §13 B19)

`[РАСТАЛҒАН]` ESC/POS-та KZ-1048 = **код беті 53**, командасы `ESC t 53` = `\x1B\x74\x35`. Epson құжаттамасының беті **«Page 53 [KZ-1048: Kazakhstan]»** деп аталады (⛔ бет бұғатталған, тек тақырып расталды).
<https://download4.epson.biz/sec_pubs/pos/reference_en/charcode/page_53.html>

`[РАСТАЛҒАН]` `escpos-printer-db/data/encoding.yml`-дегі **дәл жазба** (шикі файлдан оқылды, 625–626 жолдар):
```yaml
RK1048:
  iconv: RK1048
```
Салыстыру үшін, сол файлдағы басқа кириллица жазбалары:
```yaml
CP866:
  iconv: CP866
  python_encode: cp866
CP1251:
  iconv: CP1251
  python_encode: cp1251
CP775:
  iconv: CP775
  python_encode: cp775
```
**Айырма шешуші:** CP866/CP1251/CP775-те `python_encode` өрісі бар, **RK1048-де ЖОҚ — тек `iconv`.** Бұл дегеніміз RK1048 **стандартты Python (және демек Java, .NET) кодтауы емес**, ол тек GNU iconv-та бар. Файлда барлығы **188 кодтау** сипатталған.
<https://github.com/receipt-print-hq/escpos-printer-db/blob/master/data/encoding.yml>

### 3.3 Неге код беті — ЖЕТКІЛІКСІЗ шешім

`[БОЛЖАМ]` KZ-1048 бар екені жақсы, бірақ оған **сүйенуге болмайды**. Төрт себеп, әрқайсысы дереккөзбен:

1. **Темірге байлау болып қалады.** `[РАСТАЛҒАН]` 50 профильдің тек 15-і RK1048 қолдайды, олардың бәрі дерлік Epson. **SUNMI V2 — жоқ. Bixolon SRP-S300 — жоқ. Rongta — жоқ. Star — жоқ. Citizen — жоқ. XPrinter — жоқ.** «Тек Epson-да қазақша басады» дегеніміз — CLAUDE.md §7-дегі «меншікті темірге байланбау» ережесінің тікелей бұзылуы.
2. **Dart-та RK1048 кодегі жоқ.** Dart SDK-да тек `ascii`, `latin1`, `utf8`. `[РАСТАЛҒАН]` `charset_converter` **2.5.1** (**2026-07-18**) нативті жүйелік API-ларды пайдаланады: Android → Java `Charset`, iOS → CoreFoundation, Windows → `MultiByteToWideChar`, Linux → GLib iconv. `[БОЛЖАМ]` Windows-та KZ-1048 үшін жүйелік code page нөмірі жоқ, Java-да `RK1048` стандартты Charset емес (§3.2-дегі `python_encode`-тың жоқтығы соны растайды) → конверсия платформалар арасында бірдей жұмыс істемейді. Бәрібір **қолмен 128 жазбалық кесте** жазуға тура келеді.
   <https://pub.dev/packages/charset_converter>
3. **Аралас мәтін мүмкін емес.** Бір чекте қазақша тауар аты + латын брендi + ₸ болса, әр фрагментте `ESC t` ауыстыру керек — өте нәзік логика.
4. **Код беті нөмірлері әр өндірушіде әртүрлі.** `[РАСТАЛҒАН]` Rongta RP326-да CP866 = 7 (Epson-да 17), CP1251 = 6 (Epson-да 46). Мәтіндік жол профильдер дерекқорының дұрыстығына толық тәуелді.

### 3.4 ҰСЫНЫЛАТЫН ШЕШІМ: чекті сурет ретінде басу (raster)

`[РАСТАЛҒАН]` `unified_esc_pos_printer` дәл осы мәселені шешеді: мәтінді принтердің ішкі таңба кестелерімен емес, **Flutter-дің өз мәтін қозғалтқышымен** рендерлеп, суретке айналдырады. README-де CJK, араб, деванагари, тай мысал ретінде аталады. `rowRaster()` + `PrintRasterColumn` — «Flutter рендерлей алатын кез келген қаріп, жазу, `TextStyle`» үшін бағаналы кесте макетін береді.
<https://github.com/elrizwiraswara/unified_esc_pos_printer>

`[РАСТАЛҒАН]` Қағаз ені мен пиксель саны:

| Ені | Пиксель | Font A | Font B | Дереккөз |
|---|---|---|---|---|
| **58 мм** | **384 px** | 32 таңба/жол | 42 таңба/жол | unified README; `Sunmi-V2` профилі: 57,5мм = 384 px |
| **80 мм** | **576 px** | 48 таңба/жол | 64 таңба/жол | unified README; `TM-T20III` профилі: 80мм = 576 px, 203 dpi |
| **80 мм (Bixolon)** | **640 px** | — | — | ⚠️ `SRP-S300` профилі: 80мм = **640 px** |

⚠️ **Ені әрқашан профильден алынуы керек, `576` деп қатты жазуға болмайды** (§2.5 қорытынды 3).

`[РАСТАЛҒАН]` `thermal_printer_flutter` **2.0.0+3** (2026-06-08) да widget→сурет басуды **Floyd–Steinberg dithering**-пен қолдайды.
<https://pub.dev/packages/thermal_printer_flutter>

`[РАСТАЛҒАН]` Растр командалары: `GS v 0` (ескі, кең таралған) және `GS ( L` (жаңа). `unified_esc_pos_printer` екеуін де, `ESC *` (column format) режимін де қолдайды.

`[РАСТАЛҒАН]` Растрлау үшін `image` **4.9.2** (**2026-08-19**) — таза Dart, барлық платформа, изоляттарда асинхронды орындалуды қолдайды.
<https://pub.dev/api/packages/image>

### 3.5 Растрдың бағасы — көлем және жылдамдық

`[БОЛЖАМ]` Есептеу (растадылған 576/384 px санынан шығарылған арифметика, өлшенген емес):

- 80 мм, ені **576 px**, биіктігі ~1000 px (орташа 10–15 позициялы чек):
  `576 / 8 × 1000 = 72 000 байт ≈ **72 КБ**`
- 58 мм, ені **384 px**, биіктігі ~1000 px:
  `384 / 8 × 1000 = 48 000 байт ≈ **48 КБ**`
- Bixolon 80мм, **640 px**: `640 / 8 × 1000 = 80 000 байт ≈ **80 КБ**`

Мәтіндік режимде сол чек ~1–2 КБ. Яғни растр **~30–50 есе көп байт**.

`[БОЛЖАМ]` Транспорт бойынша салдары:

| Транспорт | ~Өткізу қабілеті | 72 КБ жіберу | Қорытынды |
|---|---|---|---|
| Ethernet / Wi-Fi 9100 | МБ/с | < 0,1 с | ✅ Мәселе жоқ |
| USB / Windows спулер | МБ/с | < 0,1 с | ✅ Мәселе жоқ |
| SUNMI ішкі принтер (AIDL) | жоғары | < 0,1 с | ✅ Мәселе жоқ |
| **Bluetooth SPP (RFCOMM)** | ~10–100 КБ/с | ~1–7 с | ⚠️ Байқалатын кідіріс |
| **BLE** | ~1–10 КБ/с | **7–70 с** | ⛔ **Жарамсыз** |

`[БОЛЖАМ]` Одан бөлек растр **физикалық басып шығару уақытын** да ұзартады: термобаста әр нүкте жеке қыздырылады, ал мәтіндік режимде ішкі қаріп қолданылады. Нақты айырманы өлшеу керек → §13 B3.

### 3.6 Гибридті стратегия — ҰСЫНЫС

`[БОЛЖАМ]` **Чекті бір бүтін сурет қылып басу — қате.** Дұрысы — **фрагменттік растр**:

1. **Сандар, күндер, сомалар, штрихкод, QR, латын мәтіні** → мәтіндік режим (арзан, жылдам, ішкі қаріп).
2. **Кириллица + қазақ әріптері бар жолдар ғана** → растр (`rowRaster`).
3. **Фискалдық QR** → әрқашан ESC/POS нативті QR командасы (растр емес) — сканерленуі кепілді болу үшін.
4. Профильде `RK1048` бар болса (Epson/EPOS) → опция ретінде мәтіндік қазақша режимге ауысу (жылдамдық үшін), бірақ **әдепкі — растр**.

`PrinterAdapter` интерфейсінде бұл екі мүмкіндік флагы ретінде көрсетілсін: `supportsKazakhCodePage`, `supportsRaster`, `paperWidthDots`. Бизнес-логика ешқашан «Epson па, XPrinter пе» деп сұрамайды.

`[БОЛЖАМ]` **Қаріп таңдау.** Растрлау кезінде қаріп **қосымшаға бекітілген asset** болуы міндетті — жүйелік қаріпке сүйенуге болмайды (SUNMI-дің қытайлық прошивкасында қазақ глифтері болмауы мүмкін). Кандидаттар: Noto Sans Mono, JetBrains Mono, Roboto Mono `[БОЛЖАМ]` — әрқайсысында 18 қазақ глифінің болуы және 384 px кеңдігінде оқылуы тексерілуі керек → §13 B2/B16.

### 3.7 ⚠️ ФИСКАЛ ЗЕРТТЕУШІСІМЕН ҚАЙШЫЛЫҚ НҮКТЕСІ

`[БОЛЖАМ]` Егер ОФД/фискал талаптары чек мәтінінің **нақты форматын** (белгілі бір қаріп, таңба саны, машинамен оқылатын мәтін) талап етсе, растр шешімі қайта қаралуы мүмкін. Сонымен қатар: **ҰТК (НКТ) тауар атаулары** ГБД-дан келеді және оларда қазақ әріптері болады — бұл мәселеден құтылу мүмкін емес.

---

## 4. Штрихкод және Data Matrix сканерлеу

### 4.1 HID сканерлер (пернетақта эмуляциясы) — негізгі сценарий

`[БОЛЖАМ]` Дүкен/кафе үшін **HID (keyboard wedge) сканер — 90% жағдай**: арзан, драйверсіз, USB немесе 2.4GHz донгл арқылы, Android-та да, Windows-та да жүйелік пернетақта ретінде көрінеді.

`[РАСТАЛҒАН]` ⛔ **ЕҢ БАСТЫ ТӘУЕКЕЛ:** flutter/flutter issue **#79849** — *«Flutter Windows Keyboard Issue using external barcode scanner»*.
- **Күйі: ОТКРЫТ (Open)**, ашылған **2021-04-06** — 5 жыл 5 ай бұрын
- Белгілер: **P2**, `a: desktop`, `e: device-specific`, `engine`, `platform-windows`, `team-windows`, `triaged-windows`
- Мәні: сыртқы штрихкод сканері Chrome, Word, Notepad-та және **Android-тағы дәл сол Flutter кодында** жұмыс істейді, ал **Windows-тағы Flutter қосымшасында TextField-ке дерек жетпейді**.
- **Issue-де жұмыс істейтін workaround келтірілмеген.** Мысал код `RawKeyboardListener` + `FocusNode` қолданады — бұл шешім емес, дәл проблемалы имплементация.
<https://github.com/flutter/flutter/issues/79849>

`[РАСТАЛҒАН]` `RawKeyboardListener` **deprecated** — v3.18.0-2.0.pre-дан кейін. Миграция: `RawKeyboard → HardwareKeyboard`, `RawKeyboardListener → KeyboardListener`, `RawKeyEvent → KeyEvent`.
<https://api.flutter.dev/flutter/widgets/RawKeyboardListener-class.html> · flutter#136419

`[ЖАНАМА]` Жаңа API-дың өз мәселелері бар: flutter#162305 — `KeyboardListener`/`Focus.onKeyEvent` `KeyUpEvent`-ті JS/WASM build-те дұрыс шығармайды, кейбір әзірлеушілер `RawKeyboardListener`-ге қайта оралуға мәжбүр болған. flutter#154141 — `HardwareKeyboard` тек focus node фокуста болғанда оқиға береді, глобалды тыңдау қиындаған.
<https://github.com/flutter/flutter/issues/162305> · <https://github.com/flutter/flutter/issues/154141>

`[РАСТАЛҒАН]` Дайын пакет `code_scan_listener` — `HardwareKeyboard` API-ын қолданады, бірақ **соңғы релизі 0.4.1, 2023-07-05** (3 жыл бұрын) ⚠️.
<https://pub.dev/api/packages/code_scan_listener> · <https://github.com/utamori/code_scan_listener>

`[БОЛЖАМ]` **Қосымша HID қиындықтары** (жобалық шешім керек):
- Сканер кириллица раскладкасында «типтегенде» қате символдар келуі мүмкін → сканерді сандық/латын режимге қатты бекіту немесе scan code деңгейінде оқу.
- Data Matrix кодтарында **GS (0x1D) сепараторы** болады — HID сканер оны әдетте тастап кетеді. Маркировка үшін ФАТАЛЬДІ → R4.
- «Оқу аяқталды» белгісі әдетте `Enter`, бірақ адам да Enter басады → таймингпен ажырату (сканер таңбалары ~10–30 мс аралықпен, адам ~150+ мс) `[БОЛЖАМ]`.

### 4.2 Камера арқылы сканерлеу

`[РАСТАЛҒАН]` **mobile_scanner 7.4.0**, **2026-07-20**.
pubspec `flutter.plugin.platforms`: **`android`, `ios`, `macos`, `web`** — **Windows та, Linux та ЖОҚ**.
Қозғалтқыштар: Android → CameraX/ML Kit, iOS/macOS → AVFoundation/Apple Vision, Web → ZXing/zxing-wasm. Web-те `dataMatrix` zxing-wasm және ZXing-js backend-терінде қолданылады.
<https://pub.dev/api/packages/mobile_scanner> · <https://pub.dev/packages/mobile_scanner>

`[РАСТАЛҒАН]` Репозиторийде Windows қолдауын сұрайтын ашық issue **табылмады** — яғни бұл жоспарланған функция да емес.
<https://github.com/juliansteenbakker/mobile_scanner/issues>

`[РАСТАЛҒАН]` **google_mlkit_barcode_scanning 0.16.1**, **2026-08-17**.
pubspec `flutter.plugin.platforms`: **`android`, `ios`** — басқасы жоқ. Бетте тікелей: *«Web or any other platform is not supported»*. iOS min deployment 15.5, Android minSdk 21.
<https://pub.dev/api/packages/google_mlkit_barcode_scanning>

`[РАСТАЛҒАН]` ML Kit `BarcodeFormat` enum-ында **`dataMatrix` БАР**. Толық тізім: `all, unknown, code128, code39, code93, codabar, dataMatrix, ean13, ean8, itf, qrCode, upca, upce, pdf417, aztec`.
<https://pub.dev/documentation/google_mlkit_barcode_scanning/latest/google_mlkit_barcode_scanning/BarcodeFormat.html>

`[БОЛЖАМ]` **Қорытынды: камера арқылы сканерлеу Windows-та ЖОҚ.** Екі негізгі пакеттің де pubspec-інде Windows жоқ, оны сұрайтын issue де жоқ. Windows нұсқасында камера сканері — не бар емес, не коммерциялық SDK (Dynamsoft, Scanbot). Бұл Windows нұсқасын **міндетті түрде HID сканерге тәуелді** етеді, ал HID сканер #79849 мәселесімен қауіп астында. **Екі тәуекел бір-бірін күшейтеді.**

`[БОЛЖАМ]` Одан бөлек, маркировка Data Matrix кодтары **өте ұсақ** (жиі 5–8 мм), телефон камерасының автофокусы жақын қашықтықта қиналады. Кәсіби 2D сканер (SUNMI ішкі, Zebra, Honeywell, Newland) маркировка үшін іс жүзінде міндетті.

### 4.3 SUNMI ішкі сканері

`[РАСТАЛҒАН]` **sunmi_scanner 0.0.8**, **2026-08-17**, pubspec platforms: **`android`** ғана.
Мүмкіндіктер: сервисті bind/unbind, сканерлеу оқиғаларының ағыны, қосылу күйін бақылау (connected/disconnected/failed), бағдарламалық start/stop, құрылғы моделін анықтау. Тексерілген: **SUNMI L2ks, P3H, L2s PRO**. Platform channels арқылы SUNMI Android SDK-сын орайды. MIT.
<https://pub.dev/api/packages/sunmi_scanner> · <https://github.com/FrenkyDema/sunmi_scanner>
⚠️ Нұсқасы **0.0.x** — API тұрақсыз болуы мүмкін.

`[ЖАНАМА]` SUNMI L2k/L2s — маркировка тауарларымен жұмыс үшін жасалған ТСД, ЕГАИС/ФГИС/«Честный знак» талаптарына сай. L2S сканер модулінің DataMatrix параметрлерін конфиг штрихкодтарымен баптау керек.
<https://www.cleverence.ru/hardware/mdc/sunmi/sunmi-l2k/4085/> · <https://help.mertech.ru/tsd/Mertech_Sunmi/nastroyka_L2S.html>
> ⚠️ Бұл дереккөздер **Ресей** маркировкасына қатысты. ҚР маркировкасы бөлек; темір деңгейінде айырма жоқ (Data Matrix — Data Matrix), бірақ код форматы мен ОФД хаттамасы бөлек → фискал зерттеушісі растауы керек.

`[БОЛЖАМ]` **Ұсынылатын сканер стратегиясы:**
- Негізгі: **HID сканер** (кез келген өндіруші, 2D, GS сепараторын сақтайтын режимде).
- SUNMI/PAX құрылғысында: **ішкі сканер SDK** — HID-тен сенімдірек, шикі деректер келеді.
- Резерв (шағын кофехана, планшет): **камера** (`mobile_scanner`), Android/iOS-та ғана.
- `ScannerAdapter` интерфейсі үшеуін де жасырады.

---

## 5. Ақша жәшігі (cash drawer)

### 5.1 Принтер арқылы ашу — стандартты жол

`[РАСТАЛҒАН]` ESC/POS командасы `ESC p` = `0x1B 0x70` — `win32` пакетінің ресми мысалында дәл осы байттар (`\x1b\x70\x00`) EPSON термопринтеріне RAW түрде жіберіледі.
<https://github.com/halildurmus/win32/blob/main/examples/printer_raw.dart>

`[РАСТАЛҒАН]` `unified_esc_pos_printer` жәшікті **Pin 2 (әдепкі) және Pin 5** арқылы ашуды қолдайды: `ticket.openCashDrawer()` / `ticket.openCashDrawer(pin: CashDrawer.pin5)`.
<https://github.com/elrizwiraswara/unified_esc_pos_printer>

`[РАСТАЛҒАН]` `windows_escpos_engine` (0.0.3, 2026-07-19) да ақша жәшігі командаларын қолдайды.
<https://pub.dev/packages/windows_escpos_engine>

`[БОЛЖАМ]` Физикалық схема: жәшік **RJ11/RJ12 кабельмен принтерге** қосылады (принтердің «DK» порты). Касса жәшікті **тікелей** басқармайды — ол принтерге команда жібереді, принтер 12/24 В импульс береді. Салдары:
- Ақша жәшігі — **принтерге тәуелді ресурс**, бөлек құрылғы емес.
- Принтер жоқ болса, жәшік те ашылмайды.
- `DLE DC4` (`0x10 0x14`) — балама «real-time» команда, принтер буфері бос болмаса да орындалады; кейбір принтерлерде `ESC p`-дан сенімдірек `[БОЛЖАМ]` → §13 B11.

### 5.2 Тікелей USB жәшіктер

`[БОЛЖАМ]` USB-HID/USB-serial дербес жәшіктер (мыс. APG ProUSB) бар, бірақ:
- Android: `[РАСТАЛҒАН]` `usb_serial` **0.5.2, 2024-07-12** ⚠️ (2 жыл), pubspec platforms: `android` ғана.
- Windows: `[РАСТАЛҒАН]` `flutter_libserialport` **0.6.0, 2025-08-01**, немесе `win32` FFI.
- iOS: **мүмкін емес**.
<https://pub.dev/api/packages/usb_serial> · <https://pub.dev/api/packages/flutter_libserialport>

`[БОЛЖАМ]` **Ұсыныс: MVP-де тікелей USB жәшіктерді ҚОЛДАМАУ.** Тек «принтер арқылы». Шағын кафе/дүкен сегментінде RJ11 жәшік — стандарт. `CashDrawerAdapter` интерфейсінің бір ғана имплементациясы (`PrinterCashDrawer`), кейін кеңейтуге ашық.

### 5.3 Платформалар айырмасы

| | Android | Windows | iOS |
|---|---|---|---|
| Принтер арқылы (network) | ✅ | ✅ | ✅ |
| Принтер арқылы (USB) | ✅ | ✅ спулер RAW | ❌ |
| Принтер арқылы (BT SPP) | ✅ | ✅ | ❌ |
| SUNMI ішкі жәшік порты | ✅ + **күйін оқу** | — | — |
| Тікелей USB жәшік | ⚠️ USB Host | ⚠️ serial/FFI | ❌ |

`[РАСТАЛҒАН]` `sunmi_printer_plus` **4.1.1** (**2025-09-05**, `android` ғана) жәшікті ашуды **және оның күйін оқуды** қолдайды — инкассация есебі үшін пайдалы артықшылық.
<https://pub.dev/api/packages/sunmi_printer_plus>

---

## 6. Тұтынушы дисплейі (customer display)

### 6.1 Екінші экран (Android Presentation API)

`[РАСТАЛҒАН]` **presentation_displays 1.0.0**, **2023-12-15** ⚠️ (2 жыл 9 ай бұрын).
pubspec `flutter.plugin.platforms`: `android`, `ios`. Verified publisher `smew.tech`, BSD-2.
Жұмыс принципі: Flutter widget-ті бөлек `FlutterEngine`-ге айналдырып, кэштеп, нативті Android `Presentation` диалогына береді. HDMI немесе сымсыз қосылған екінші экранға шығады. Екі жақты дерек алмасу бар.
<https://pub.dev/api/packages/presentation_displays> · <https://github.com/VNAPNIC/presentation-displays>

`[БОЛЖАМ]` Жалғыз шынайы жалпы шешім, бірақ **2 жыл 9 ай жаңармаған** — Flutter 3.3x-тегі multi-engine API өзгерістерімен үйлесімділігі күмәнді → R8.

### 6.2 SUNMI dual-screen / LCD

`[РАСТАЛҒАН]` `sunmi_printer_plus` (4.1.1, 2025-09-05, `android` ғана) құрылғының **LCD экранына сурет пен мәтін шығаруды** қолдайды.
<https://pub.dev/packages/sunmi_printer_plus>

`[БОЛЖАМ]` Екі бөлек нәрсені шатастырмау керек:
1. **SUNMI кіші LCD** (V2, T2 mini-дегі шағын экран) — `sunmi_printer_plus` арқылы, тек мәтін/сурет.
2. **SUNMI dual-screen** (D2, T2 — екі толық экран) — Android Presentation API арқылы, яғни `presentation_displays` немесе SUNMI-дің өз SDK-сы.

`[БҰҒАТТАЛҒАН]` SUNMI dual-screen SDK құжаттамасы (`docs.sunmi.com`) бұғатталған. pub.dev-те SUNMI dual-screen үшін **арнайы Flutter пакеті табылмады** — өз platform channel-ымызды жазуға тура келеді.

### 6.3 Сериялық VFD дисплейлер

`[РАСТАЛҒАН]` `escpos-printer-db`-де тұтынушы дисплейлерінің профильдері де бар: **`AF-240` (Customer Display)**, **`OCD-100`**, **`OCD-300`** — соңғы екеуінде CP866 және CP1251 бар, **RK1048 ЖОҚ**. Яғни VFD дисплейде де қазақ әріптері мәтіндік режимде шықпайды, ал VFD-де растр мүмкіндігі жоқ.
<https://github.com/receipt-print-hq/escpos-printer-db> `dist/capabilities.json`

`[БОЛЖАМ]` **MVP ұсынысы:** тұтынушы дисплейін **MVP-ден шығару** немесе ең қарапайым түрде ғана (SUNMI LCD + Android Presentation). Себептер: (а) заң талабы емес, (б) 3 бөлек имплементация (LCD/Presentation/VFD) MVP шеңберінен шығады, (в) VFD-де қазақ мәселесі шешілмейді. `CustomerDisplayAdapter` интерфейсі бекітілсін, имплементация — no-op + SUNMI.

---

## 7. Таразылар

### 7.1 Flutter экожүйесіндегі жағдай — өте нашар

`[РАСТАЛҒАН]` pub.dev-те таразыға арналған іс жүзінде **бір ғана** пакет: **digital_scale 1.2.3+1**, **2026-08-19**, **Android** ғана, ⛔ **GPL-3.0**.
Қолдайтын модельдер: **Toledo Prix 3, Elgin DP-1502, Urano (POP Light, US20/2 POP-S, US31/2 POP-S), UPX EA-20/EA-32** — бұл **бразилиялық** нарық. **CAS та, Mettler Toledo-ның еуропалық модельдері де, «Штрих» те ЖОҚ.** Қосылу: serial, USB, Bluetooth.
<https://pub.dev/api/packages/digital_scale> · <https://github.com/sergiotucano/digital_scale>

`[БОЛЖАМ]` **Қорытынды: Flutter-де таразы үшін дайын, лицензиясы жарамды шешім ЖОҚ.** Хаттаманы өзіміз жазуға тура келеді.

### 7.2 Сериялық порт

| Пакет | Нұсқа | **Жарияланған** | pubspec plugin platforms |
|---|---|---|---|
| **flutter_libserialport** | 0.6.0 | **2025-08-01** | `android, linux, macos, windows` |
| **usb_serial** | 0.5.2 | **2024-07-12** ⚠️ | `android` |

<https://pub.dev/api/packages/flutter_libserialport> · <https://pub.dev/api/packages/usb_serial>

`[БОЛЖАМ]` **Android-тағы ескерту:** `flutter_libserialport` — `libserialport` C-кітапханасының орамасы, ол `/dev/tty*` құрылғы файлдарын ашады. Android-та қарапайым қосымшаның `/dev/ttyUSB0`-ге **рұқсаты жоқ** (root немесе өндірушінің арнайы прошивкасы керек). pubspec-те `android` тұрғаны «жиналады» дегенді білдіреді, «жұмыс істейді» дегенді емес. Android-та дұрыс жол — USB Host API + FTDI/CDC драйверін user space-те іске асыру (`usb_serial`, бірақ ол 2 жыл ескі) → §13 B8.

### 7.3 Хаттамалар

`[БОЛЖАМ]` Негізгі хаттамалар (жалпы инженерлік білім, ресми сілтеме табылмады):
- **CAS** (ER Plus, AP-1): ASCII сұраныс-жауап, `ENQ`/`ACK`, салмақ өрісі + тұрақтылық белгісі.
- **Mettler Toledo**: SICS (`S` — тұрақты салмақ, `SI` — лезде).
- **«Штрих-М» / «Масса-К»**: өз екілік хаттамалары, COM немесе USB-HID.
- Ортақ: 9600/8N1 немесе 4800/7E1, салмақ ASCII мәтін ретінде.

`[БҰҒАТТАЛҒАН]` Қазақстанда іс жүзінде қандай таразылар қолданылатыны және олардың хаттама құжаттамасы тексерілмеді (WebSearch лимиті + өндіруші сайттары бұғатталған).

### 7.4 Таразы-таңбалағыш (label printing scale)

`[БОЛЖАМ]` Таразы-таңбалағыштар (CAS CL5000, «Штрих-Принт», DIGI SM) — **дербес құрылғылар**:
- Өз ішінде PLU базасын сақтайды, оны кассадан емес, өз утилитасынан/Ethernet арқылы жүктейді.
- Салмағы бар **EAN-13 штрихкодын** басады: әдетте `2` префиксі + PLU коды + салмақ/баға + checksum.
- Кассаға **тікелей қосылмайды** — касса тек этикеткадағы штрихкодты сканерлеп, префикс бойынша «бұл салмақ штрихкоды» деп таниды.

`[БОЛЖАМ]` **MVP үшін салдары маңызды әрі жеңіл:**
- Таразы-таңбалағышты **интеграциялаудың қажеті жоқ**. Керегі — кассада **салмақ штрихкодын талдау (parse) логикасы**: префикс, PLU өрісінің ұзындығы, салмақ/баға өрісінің масштабы конфигурацияланатын болуы.
- Тікелей қосылатын таразы — бөлек, күрделірек сценарий, **MVP-ден шығару керек**.
- `/docs/CONTRACTS.md`-ке `WeightBarcodeFormat` конфигурациясы ретінде кірсін.

---

## 8. SUNMI / PAX / Posiflex SDK-лары

### 8.1 SUNMI

`[РАСТАЛҒАН]` pub.dev API-дан алынған нақты деректер (2026-09-01):

| Пакет | Нұсқа | **Жарияланған** | pubspec platforms | Не істейді |
|---|---|---|---|---|
| **sunmi_printer_plus** | 4.1.1 | **2025-09-05** | `android` | Принтер (мәтін, штрихкод, QR, сурет, кесу), **ақша жәшігі + күйі**, **LCD экран** |
| **sunmi_scanner** | 0.0.8 | **2026-08-17** | `android` | Ішкі сканер, оқиға ағыны, bind/unbind. Тексерілген: L2ks, P3H, L2s PRO |
| **sunmi_kali** | 1.3.1 | **2026-06-12** | `android` | Тек принтер (Kali Alimentares компаниясы үшін) |

`sunmi_printer_plus` бетінде тікелей: **«THIS PACKAGE WILL WORK ONLY IN ANDROID!»**
<https://pub.dev/api/packages/sunmi_printer_plus> · <https://github.com/brasizza/sunmi_printer/>
<https://pub.dev/api/packages/sunmi_scanner> · <https://pub.dev/api/packages/sunmi_kali>

`[БОЛЖАМ]` SUNMI интеграциясы Android **AIDL сервистері** арқылы жүреді (ішкі принтер үшін `woyou.aidlservice.jiuiv5.IWoyouService`). Қосымша SUNMI прошивкасында тіркелген сервиске bind жасайды; SUNMI емес құрылғыда bind сәтсіз болады → адаптер соны түсініп, келесі транспортқа өтуі керек.

`[РАСТАЛҒАН]` ⚠️ **SUNMI-дің ішкі принтерінде RK1048 да, CP1251 де ЖОҚ** (§2.5). Демек SUNMI-де қазақ әріптері **тек растр арқылы** басылады.

`[БҰҒАТТАЛҒАН]` SUNMI-дің ресми developer порталы (`docs.sunmi.com`, `sunmi.com`) бұғатталған — эквайринг қосымшасын Intent арқылы шақыру, dual-screen SDK, cash drawer SDK құжаттамасы оқылмады.

### 8.2 PAX

`[РАСТАЛҒАН]` pub.dev API-дан:

| Пакет | Нұсқа | **Жарияланған** | pubspec platforms | Не істейді |
|---|---|---|---|---|
| **flutter_pax_printer_utility** | 0.1.6 | **2025-10-15** | `android` | PAX принтері **NeptuneLiteApi SDK** арқылы: мәтін, QR, сурет, кесу. **Штрихкод, divisor, принтер ақпараты — ІСКЕ АСЫРЫЛМАҒАН.** Тексерілген: **A920, A910S** |
| **nebula_flutter_plugin** | 1.0.3 | **2026-04-07** | `android` | **PAX Nebula SDK** орамасы: **Sale, Refund, Void, Settlement** + custom txn. Қосылу: **TCP (Wi-Fi), Bluetooth, USB, Cloud (MQTT)** |

<https://pub.dev/api/packages/flutter_pax_printer_utility> · <https://github.com/AuliaVailo/flutter_pax_printer_utility>
<https://pub.dev/api/packages/nebula_flutter_plugin> (homepage: `https://www.pax.com.cn/`)

`[БОЛЖАМ]` **Маңызды бақылау (эквайринг зерттеушісіне):** бұл екеуі **мүлдем бөлек архитектуралар**:
- `nebula_flutter_plugin` — касса **бөлек құрылғыда**, PAX терминалы TCP/BT/USB/MQTT арқылы **сыртқы құрылғы** ретінде басқарылады.
- `flutter_pax_printer_utility` — қосымша **PAX терминалының өзінде** (PayDroid) жұмыс істейді.
Бұл PaymentAdapter интерфейсінің дизайнына тікелей әсер етеді.

### 8.3 Posiflex

`[БҰҒАТТАЛҒАН]` Posiflex үшін pub.dev-те **бірде-бір Flutter пакеті табылмады**. `paxtechnology.com` сияқты өндіруші сайттары бұғатталған.
`[БОЛЖАМ]` Posiflex негізінен Windows POS терминалдарын шығарады. Олар — стандартты Windows машиналары, ондағы принтер/жәшік/дисплей жүйелік драйверлер арқылы жұмыс істейді, яғни арнайы SDK қажет емес: §2.2-дегі спулер RAW тәсілі жеткілікті. Тексерілуі керек.

### 8.4 Hardware-agnostic принципін сақтау — интерфейс дизайны

`[БОЛЖАМ]` CLAUDE.md §5.4: «бизнес-логикада нақты провайдер аты ЕШҚАШАН hardcode жасалмайды». Темірге қатысты бұл былай жүзеге асуы керек:

**`/packages/domain` — таза Dart, I/O жоқ.** Тек абстракциялар мен деректер:
```
ReceiptDocument   — не басу керек (домен түсінігі, ESC/POS емес)
DeviceCapability  — { supportsRaster, supportsKazakhCodePage, kazakhCodePageIndex,
                      supportsCashDrawer, supportsQr, paperWidthDots, dpi }
PrinterId, ScannerId, DrawerId
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
EscPosNetworkPrinter    (dart:io Socket 9100)   — Android/iOS/Windows
EscPosWindowsSpooler    (win32 WritePrinter)    — Windows
EscPosBluetoothPrinter  (SPP)                   — Android/Windows
SunmiInnerPrinter       (AIDL)                  — SUNMI Android
PaxInnerPrinter         (NeptuneLite)           — PAX Android
NoopPrinter             (тест/PDF)              — барлығы
```

`[БОЛЖАМ]` **Төрт қатаң ереже:**
1. **ESC/POS байттары доменге ЕШҚАШАН жетпейді.** Домен `ReceiptDocument` береді, адаптер оны байтқа айналдырады. Растр ма, мәтін бе — адаптердің шешімі.
2. **Capability-driven, brand-driven емес.** Ешқашан `if (printer is SunmiPrinter)` емес — `if (caps.supportsRaster)`.
3. **Code page нөмірі мен қағаз ені әрқашан профильден.** `[РАСТАЛҒАН]` Rongta-да CP866 = 7, Epson-да = 17; Bixolon 80мм = 640 px, Epson = 576 px (§2.5). Hardcode = сынған чек.
4. **Әр адаптер өз пакетінде, компиляция кезінде оқшауланған.** SUNMI пакеті Android-қа ғана жиналуы керек — әйтпесе Windows build құлайды (conditional import + platform-specific plugin declaration).

### 8.5 Қазақстандағы қолжетімділік

`[БҰҒАТТАЛҒАН]` SUNMI, PAX, Posiflex құрылғыларының ҚР дистрибьюторлары, бағалары, сервистік қолдауы **тексерілмеді** (WebSearch лимиті + `sunmi.kz`, `sunmi.com`, `paxtechnology.com` бұғатталған).
`[БОЛЖАМ]` Бірақ жоба принципі бойынша бұл **блокер емес**: егер архитектура шынымен hardware-agnostic болса, SUNMI/PAX адаптері — қосымша, ядро емес.

---

## 9. Платформалар салыстыру кестесі (Android / Windows / iOS)

### 9.1 Перифериялық құрылғылар матрицасы

| Құрылғы / қосылу | Android | Windows | iOS | Дәлел |
|---|---|---|---|---|
| **Локалды БД (drift/SQLite)** | ✅ | ✅ | ✅ | pub.dev: sqlite3 3.5.2 prebuilt win aarch64/x64/x86 |
| **Термопринтер — Ethernet/Wi-Fi 9100** | ✅ | ✅ | ✅ | `dart:io` Socket, плагинсіз |
| **Термопринтер — USB** | ✅ USB Host | ✅ спулер RAW | ❌ | win32 `printer_raw.dart` |
| **Термопринтер — Bluetooth SPP** | ✅ | ✅ RFCOMM | ❌ | Apple MFi талабы |
| **Термопринтер — BLE** | ✅ | ⚠️ бөлек форк | ✅ | flutter_blue_plus_winrt 0.0.20 |
| **Термопринтер — ішкі (SUNMI/PAX)** | ✅ AIDL | — | — | sunmi_printer_plus: `android` ғана |
| **Растр басу (қазақ әріптері)** | ✅ | ✅ | ✅ | unified_esc_pos_printer |
| **Ақша жәшігі — принтер арқылы** | ✅ | ✅ | ⚠️ тек network принтер | ESC p / DLE DC4 |
| **Ақша жәшігі — тікелей USB** | ⚠️ | ⚠️ | ❌ | MVP-ден тыс |
| **HID сканер (keyboard wedge)** | ✅ | ⛔ **flutter#79849 Open** | ⚠️ BT HID | github.com/flutter/flutter/issues/79849 |
| **Камера сканері (mobile_scanner)** | ✅ | ❌ **ЖОҚ** | ✅ | pubspec: android,ios,macos,web |
| **Камера сканері (ML Kit)** | ✅ | ❌ | ✅ | pubspec: android,ios |
| **Data Matrix оқу (камера)** | ✅ | ❌ | ✅ | BarcodeFormat.dataMatrix |
| **Ішкі сканер (SUNMI)** | ✅ | — | — | sunmi_scanner: `android` |
| **Тұтынушы дисплейі (Presentation)** | ✅ | ❌ | ⚠️ | presentation_displays: android,ios |
| **Тұтынушы дисплейі (SUNMI LCD)** | ✅ | — | — | sunmi_printer_plus |
| **Сериялық порт (таразы/VFD)** | ⚠️ рұқсат мәселесі | ✅ | ❌ | flutter_libserialport: android,linux,macos,windows |
| **Kiosk mode** | ✅ Lock Task | ⚠️ қолмен | ⚠️ Guided Access | kiosk_mode 0.9.0: android,ios |
| **Фондық сервис** | ✅ foreground service | ✅ | ⚠️ шектеулі | flutter_foreground_task 11.0.1: android,ios |
| **Қауіпсіз сақтау (құпиялар)** | ✅ | ✅ DPAPI | ✅ | flutter_secure_storage 11.0.0: 6 платформа |

**Жалғыз толық жасыл жол — локалды БД.** Қалғандарының бәрінде ең болмағанда бір платформа ақсайды.

### 9.2 iOS-тың шектеулері — жобаға қауіп бар ма?

`[РАСТАЛҒАН]` iOS-та Bluetooth Classic / SPP арқылы кез келген аксессуармен байланысу **мүмкін емес**. External Accessory framework тек **MFi (Made for iPhone) сертификатталған** аксессуарлармен жұмыс істейді (Apple-дің аутентификация коппроцессоры керек). Сертификатсыз құрылғыға тек **BLE** (CoreBluetooth) арқылы.
<https://developer.apple.com/documentation/externalaccessory>

`[РАСТАЛҒАН]` `flutter_blue_plus` **2.3.12** (**2026-08-10**) бетінде тікелей: **«❗ Bluetooth Classic is not supported ❗»**. pubspec platforms: `android, ios, linux, macos, web, windows`. Windows үшін іс жүзінде бөлек `flutter_blue_plus_winrt` **0.0.20** (**2026-05-08**, `windows` ғана) керек ⚠️ **0.0.x**.
<https://pub.dev/api/packages/flutter_blue_plus> · <https://pub.dev/api/packages/flutter_blue_plus_winrt>

`[БОЛЖАМ]` **iOS-тағы POS шындығы:**

iOS-та жұмыс істейді:
- ✅ Ethernet/Wi-Fi принтер (TCP 9100) — толық, растрмен қоса
- ✅ Камера сканері (`mobile_scanner`, ML Kit) — толық, Data Matrix-пен қоса
- ✅ Локалды БД, офлайн, синхрондау — толық
- ⚠️ BLE принтер — техникалық түрде, бірақ растр басу үшін тым баяу (§3.5)

iOS-та жұмыс істемейді:
- ❌ USB принтер, USB сканер, USB таразы
- ❌ Bluetooth SPP принтер (нарықтағы арзан принтерлердің көбі дәл осындай)
- ❌ SUNMI/PAX ішкі темірі
- ❌ Сериялық порт
- ❌ Ақша жәшігі (тек network принтер арқылы)
- ⚠️ HID сканер — тек Bluetooth HID пернетақта ретінде

**Қорытынды: iOS «Flutter — бір кодтан үш платформа» болжамын БҰЗБАЙДЫ, бірақ оны айтарлықтай шектейді.**
iOS-та **толық жабдықталған касса** құру мүмкін емес. Бірақ **Ethernet принтер + камера сканері** конфигурациясы толық жұмыс істейді — шағын кофехана үшін жеткілікті.

`[БОЛЖАМ]` **Стратегиялық ұсыныс:** iOS-ты **MVP-нің бірінші релизінен шығару**, бірақ кодта бұзбау:
- Сегмент — шағын кафе/дүкен; ҚР-да олар iPad емес, Android планшет/SUNMI қолданады `[БОЛЖАМ]`.
- iOS-та ешбір нақты артықшылық жоқ, ал тестілеу шығыны үлкен.
- Flutter коды бәрібір ортақ — iOS кейін салыстырмалы аз жұмыспен қосылады.
Бұл CLAUDE.md §4-тегі «Flutter (Android, iOS, Windows — бір кодтан)» тұжырымымен **ішінара қайшы** → §12.

### 9.3 Windows-тағы Flutter-дің жетілгендігі

`[БОЛЖАМ]` Windows — Flutter-дің ең әлсіз тармағы **плагин экожүйесі** тұрғысынан (engine-нің өзі тұрақты). Осы зерттеудегі нақты дәлелдер:

| Дәлел | Күйі | Дереккөз |
|---|---|---|
| `mobile_scanner` Windows қолдамайды | ⛔ расталған | pubspec: `android,ios,macos,web` |
| `google_mlkit_barcode_scanning` Windows қолдамайды | ⛔ расталған | pubspec: `android,ios` |
| `presentation_displays` Windows қолдамайды | ⛔ расталған | pubspec: `android,ios` |
| HID сканер жұмыс істемейді | ⛔ расталған, 2021-ден Open | flutter#79849 |
| `flutter_blue_plus` Windows үшін 0.0.x форк керек | ⚠️ расталған | flutter_blue_plus_winrt 0.0.20 |
| SQLCipher Windows CMake/OpenSSL | ⚠️ расталған | drift#3395 |
| `kiosk_mode` Windows қолдамайды | ⚠️ расталған | pubspec: `android,ios` |
| **drift/sqlite3 Windows толық** | ✅ расталған | sqlite3 3.5.2 prebuilt |
| **ESC/POS спулер RAW** | ✅ расталған | win32 `printer_raw.dart` |
| `flutter_secure_storage` Windows | ✅ расталған | pubspec: 6 платформа |
| `flutter_libserialport` Windows | ✅ расталған | pubspec: android,linux,macos,windows |

`[БОЛЖАМ]` Windows-та әр перифериялық құрылғы үшін **өз FFI кодымызды жазуға дайын болу керек**. Бұл MVP еңбек көлемін айтарлықтай өсіреді.

### 9.4 Android: фондық жұмыс, батарея, kiosk

`[РАСТАЛҒАН]` **kiosk_mode 0.9.0**, **2026-07-29**, pubspec platforms `android, ios`, verified publisher `mews.com` (MewsSystems/mews-flutter монорепосы), BSD-3.
Android → **Lock Task mode** (screen pinning), iOS → **Guided Access**. API: `getKioskMode()`, `watchKioskMode()`, `startKioskMode()`, `stopKioskMode()`.
<https://pub.dev/api/packages/kiosk_mode> · <https://github.com/MewsSystems/mews-flutter>

`[РАСТАЛҒАН]` **flutter_foreground_task 11.0.1**, **2026-08-20**, pubspec platforms `android, ios` — тірі әрі белсенді (major нұсқа 11).
<https://pub.dev/api/packages/flutter_foreground_task> · <https://github.com/Dev-hwang/flutter_foreground_task>

`[БОЛЖАМ]` **Android-тағы фондық жұмыс — офлайн-first үшін маңызды тәуекел:**
- ОФД-ға чек жіберу кезегі мен синхрондау **қосымша фонда тұрғанда да** жүруі керек. Android 12+ батарея оптимизациясы (Doze, App Standby) `Isolate`-ті де, `WorkManager`-ді де кідіртеді.
- Дұрыс жол: **foreground service** (тұрақты хабарландырумен) — `flutter_foreground_task` 11.0.1 тірі және дәл осы үшін.
- Ең қауіпсіз конфигурация: **kiosk mode + экран әрқашан қосулы + тұрақты қуат** — касса планшеті бәрібір қуатта тұрады, онда Doze мәселесі негізінен жойылады.
- Орнату кезінде «Battery optimization: don't optimize» сұрау керек.

`[БОЛЖАМ]` **72 сағаттық автономды режим талабына салдары:** Егер қосымша фонда өлтірілсе, чектер жоғалмайды (олар SQLite-та), бірақ **байланыс қалпына келгенде автоматты жіберу болмайды** — кассир қосымшаны ашқанда ғана. Заң (CLAUDE.md §3) «байланыс қалпына келгенде барлық чек кезекпен, ретімен жіберіледі» дейді. Бұл фискал зерттеушісімен келісілуі керек → §12.

---

## 10. Ұсынылатын Flutter пакеттерінің тізімі

> Барлық нұсқа/күн — **pub.dev API**-дан, 2026-09-01. Күндер нақты (`YYYY-MM-DD`), шамамен емес.

### 10.1 Негізгі (қабылдау ұсынылады)

| Пакет | Нұсқа | **Жарияланған** | pubspec plugin platforms | Лиц. | Не үшін |
|---|---|---|---|---|---|
| [drift](https://pub.dev/packages/drift) | 2.34.3 | 2026-07-27 | (таза Dart, 6 платформа) | MIT | Локалды БД, ORM, миграция |
| [sqlite3](https://pub.dev/packages/sqlite3) | 3.5.2 | 2026-08-19 | (таза Dart, prebuilt win/and/ios) | MIT | drift-тің нативті қабаты |
| [image](https://pub.dev/packages/image) | 4.9.2 | 2026-08-19 | (таза Dart) | MIT | Чекті растрлау, dithering |
| [unified_esc_pos_printer](https://pub.dev/packages/unified_esc_pos_printer) | 3.4.0 | 2026-07-16 | `android, ios, windows` | BSD-3 | ESC/POS + **растр мәтін** + 200+ профиль |
| [win32](https://pub.dev/packages/win32) | 6.4.0 | 2026-08-05 | (Windows FFI) | BSD-3 | Спулер RAW басу |
| [mobile_scanner](https://pub.dev/packages/mobile_scanner) | 7.4.0 | 2026-07-20 | `android, ios, macos, web` | MIT | Камера сканері (**Windows ЖОҚ**) |
| [flutter_secure_storage](https://pub.dev/packages/flutter_secure_storage) | 11.0.0 | 2026-08-06 | `android, ios, linux, macos, web, windows` | — | ОФД/эквайринг кілттері |
| [flutter_foreground_task](https://pub.dev/packages/flutter_foreground_task) | 11.0.1 | 2026-08-20 | `android, ios` | — | Фондық ОФД кезегі |
| [kiosk_mode](https://pub.dev/packages/kiosk_mode) | 0.9.0 | 2026-07-29 | `android, ios` | BSD-3 | Lock Task / Guided Access |
| [flutter_blue_plus](https://pub.dev/packages/flutter_blue_plus) | 2.3.12 | 2026-08-10 | `android, ios, linux, macos, web, windows` | BSD-3 | BLE (Windows үшін бөлек форк) |
| [charset_converter](https://pub.dev/packages/charset_converter) | 2.5.1 | 2026-07-18 | `android, ios, linux, macos, windows` | — | CP866/CP1251 (растр резерві) |

### 10.2 Шартты (тексергеннен кейін)

| Пакет | Нұсқа | **Жарияланған** | platforms | Ескерту |
|---|---|---|---|---|
| [flutter_thermal_printer](https://pub.dev/packages/flutter_thermal_printer) | 2.2.2 | 2026-08-25 | `android, ios, macos, windows` | Балама/резерв, 137 likes |
| [print_bluetooth_thermal](https://pub.dev/packages/print_bluetooth_thermal) | 1.2.2 | 2026-05-23 | `android, ios, macos, windows` | Ең көп жүктелетін (36k/ай), BT-ге маманданған |
| [sunmi_printer_plus](https://pub.dev/packages/sunmi_printer_plus) | 4.1.1 | 2025-09-05 | `android` | SUNMI адаптері |
| [sunmi_scanner](https://pub.dev/packages/sunmi_scanner) | 0.0.8 | 2026-08-17 | `android` | **0.0.x — шикі** |
| [nebula_flutter_plugin](https://pub.dev/packages/nebula_flutter_plugin) | 1.0.3 | 2026-04-07 | `android` | PAX эквайринг → **эквайринг зерттеушісіне** |
| [flutter_pax_printer_utility](https://pub.dev/packages/flutter_pax_printer_utility) | 0.1.6 | 2025-10-15 | `android` | PAX принтері, штрихкод іске асырылмаған |
| [presentation_displays](https://pub.dev/packages/presentation_displays) | 1.0.0 | **2023-12-15** ⚠️ | `android, ios` | Тұтынушы дисплейі, 2,7 жыл ескі |
| [flutter_libserialport](https://pub.dev/packages/flutter_libserialport) | 0.6.0 | 2025-08-01 | `android, linux, macos, windows` | Таразы/VFD, Android-та күмәнді |
| [nitro_printing](https://pub.dev/packages/nitro_printing) | 0.0.6 | 2026-08-29 | 6 платформа | WinSpool, бірақ **0.0.x** |
| [windows_escpos_engine](https://pub.dev/packages/windows_escpos_engine) | 0.0.3 | 2026-07-19 | `windows` | Windows RAW, **0.0.x, 0 likes** |
| [thermal_printer_flutter](https://pub.dev/packages/thermal_printer_flutter) | 2.0.0+3 | 2026-06-08 | 6 платформа | 30к жүктеу тек 14 — сирек қолданылады |

### 10.3 ⛔ ҚОЛДАНБАУ

| Пакет | Нұсқа | **Жарияланған** | Себеп |
|---|---|---|---|
| [flutter_pos_printer_platform](https://pub.dev/packages/flutter_pos_printer_platform) | 1.4.2 | 2023-07-14 | **`isDiscontinued: true`** (pub.dev API) |
| [esc_pos_printer](https://pub.dev/packages/esc_pos_printer) | 4.1.0 | **2021-09-21** | 5 жыл жаңармаған |
| [blue_thermal_printer](https://pub.dev/packages/blue_thermal_printer) | 1.2.3 | **2022-07-19** | 4 жыл, Android-only |
| [flutter_esc_pos_network](https://pub.dev/packages/flutter_esc_pos_network) | 1.0.3 | **2024-05-03** | Тастанды + **GPL-3.0** |
| [esc_pos_utils_plus](https://pub.dev/packages/esc_pos_utils_plus) | 2.0.4 | **2024-09-01** | Дәл 2 жыл жаңармаған; **транзитивті** тәуелділік |
| [esc_pos_printer_plus](https://pub.dev/packages/esc_pos_printer_plus) | 0.1.1 | 2025-02-11 | Тоқтап тұр |
| [digital_scale](https://pub.dev/packages/digital_scale) | 1.2.3+1 | 2026-08-19 | **GPL-3.0** + бразилиялық таразылар ғана |
| [usb_serial](https://pub.dev/packages/usb_serial) | 0.5.2 | **2024-07-12** | 2 жыл, Android-only |
| [code_scan_listener](https://pub.dev/packages/code_scan_listener) | 0.4.1 | **2023-07-05** | 3 жыл жаңармаған |
| [sqlite3_flutter_libs](https://pub.dev/packages/sqlite3_flutter_libs) | 0.6.0+eol | 2026-02-15 | **EOL** — қоспау |
| [sqlcipher_flutter_libs](https://pub.dev/packages/sqlcipher_flutter_libs) | 0.7.0+eol | 2026-02-15 | **EOL** — қоспау |
| [isar](https://pub.dev/packages/isar) | 3.1.0+1 | **2023-04-25** | **Тастанды** ([#1689](https://github.com/isar/isar/issues/1689)) |
| [realm](https://pub.dev/packages/realm) | 20.2.0 | **2025-09-24** | **EOL 2025-09-30** ([announcement](https://github.com/realm/realm-js/discussions/6884)) |

`[РАСТАЛҒАН]` **Транзитивті тәуелділік ескертуі:** `esc_pos_utils_plus` (2024-09-01) — `flutter_thermal_printer` мен `thermal_printer_flutter` пакеттерінің тәуелділігі, айына **50 951** жүктеу. Оны тікелей қоспасақ та, жобаға кіреді. `unified_esc_pos_printer` — өз ESC/POS генераторын жазған сияқты (200+ профилі өзінде), бұл артықшылық. `flutter pub deps` арқылы тексеру керек → R12.

---

## 11. Тәуекелдер тізімі (ықтималдық × әсер)

### 11.1 ЕҢ ЖОҒАРЫ 5 ТӘУЕКЕЛ

---

#### 🔴 R1 — Windows-та HID штрихкод сканері жұмыс істемейді

- **Не істемейді:** Windows-тағы Flutter қосымшасына HID (keyboard wedge) сканерден деректер келмейді. Сол сканер Notepad, Chrome, Word-та және **Android-тағы дәл сол Flutter кодында** жұмыс істейді.
- **Неге:** `[РАСТАЛҒАН]` Flutter-дің Windows embedder-індегі пернетақта оқиғаларын өңдеу мәселесі. flutter/flutter **#79849**, ашылған **2021-04-06**, күйі **Open**, приоритет **P2**, белгілері `platform-windows`, `team-windows`, `triaged-windows`, `engine`. **5 жыл 5 ай бойы түзетілмеген. Issue-де жұмыс істейтін workaround келтірілмеген.**
- **Ықтималдық:** **Жоғары**
- **Әсер:** **Windows тармағын тоқтатады.** Сканерсіз дүкен кассасы жоқ. Windows-та камера сканері де жоқ (R2) — сканерлеудің ЕШҚАНДАЙ жолы қалмайды.
- **Баламасы:**
  1. `KeyboardListener` / `HardwareKeyboard` (жаңа API) — issue-дегі ескі `RawKeyboardListener`-ден гөрі жұмыс істеуі мүмкін. Ең арзан тексеру.
  2. **Сканерді HID емес, USB-COM (virtual serial) режимге ауыстыру** — көп 2D сканерде мұндай режим бар, конфиг штрихкодымен қосылады. Сонда `flutter_libserialport` (Windows ✅) немесе `win32` арқылы оқимыз. **Ең сенімді балама** — әрі R4-ті (GS сепараторы) де қоса шешеді.
  3. Win32 `RegisterRawInputDevices` (WM_INPUT) арқылы FFI сканер драйверін өзіміз жазу — сканерді пернетақтадан бөлек құрылғы ретінде оқу. Ең күшті, бірақ ~1–2 апта.
  4. Ең соңғы шара: Windows-ты MVP-ден шығару.
- **Тексеру:** Windows 10/11 + кез келген USB HID 2D сканер + минималды Flutter қосымша. `KeyboardListener`, USB-COM режимі, `RegisterRawInputDevices` — үшеуін де. **1 күн, приоритет №1.**

---

#### 🔴 R2 — Windows-та камера арқылы сканерлеу мүлдем жоқ

- **Не істемейді:** Windows-та камерамен штрихкод/QR/Data Matrix оқу.
- **Неге:** `[РАСТАЛҒАН]` `mobile_scanner` 7.4.0-дің pubspec `flutter.plugin.platforms` = `android, ios, macos, web` — Windows ЖОҚ. `google_mlkit_barcode_scanning` 0.16.1 = `android, ios`. Репозиторийде Windows қолдауын сұрайтын ашық issue де табылмады — жоспарланбаған.
- **Ықтималдық:** **Расталған факт** (ықтималдық емес)
- **Әсер:** **Функцияны шектейді.** Жеке алғанда — қолайсыздық. R1-мен бірге — Windows тармағын тоқтатады.
- **Баламасы:** Windows-та камера сканерін ұсынбау (HID/USB-COM сканерге сүйену); немесе коммерциялық SDK (Dynamsoft, Scanbot) — ақылы.
- **Тексеру:** керек емес. Керегі — **өнім шешімі**: Windows-та камера сканері болмайды деп бекіту.

---

#### 🔴 R3 — Қазақ әріптері көпшілік принтерде басылмайды (SUNMI-де де!)

- **Не істемейді:** ә ғ қ ң ө ұ ү һ (8 әріп) стандартты ESC/POS мәтін режимінде басылмайды.
- **Неге:** `[РАСТАЛҒАН]` `escpos-printer-db`-дегі **50 профильдің тек 15-і** RK1048 (KZ-1048, page 53) қолдайды, олардың бәрі дерлік **Epson TM-\*** + `TEP-200M`. RK1048 ЖОҚ: **`Sunmi-V2`** (тек CP866, CP855 — CP1251 де жоқ!), **`SRP-S300`** (Bixolon), **`RP326`/`RP332`** (Rongta), `TSP600`/`TSP800`/`mcPrint` (Star), `CT-S651` (Citizen), `ZJ-5870`, `POS-5890`, `TH230` — барлығы 28 профиль. **XPrinter дерекқорда мүлдем жоқ**, өндіруші нұсқаулығы бойынша ондағы кириллица тек PC866 және PT151(1251).
  Одан бөлек `[РАСТАЛҒАН]`: `escpos-printer-db/data/encoding.yml`-де RK1048-де `python_encode` өрісі **жоқ** (CP866/CP1251-де бар) — яғни RK1048 стандартты Python/Java кодтауы емес, Dart-та да дайын кодек болмайды.
- **Ықтималдық:** **Жоғары** (расталған)
- **Әсер:** **Жобаның №2 дифференциациясын жояды.** «Толыққанды қазақ тілі» — негізгі уәде.
- **Баламасы:** ✅ **Шешім бар: чекті растр (bitmap) ретінде басу.** `[РАСТАЛҒАН]` `unified_esc_pos_printer` 3.4.0 (2026-07-16) Flutter-дің мәтін қозғалтқышымен мәтінді суретке айналдырады (`rowRaster`, `PrintRasterColumn`). Кез келген қаріп, кез келген жазу. Ені: 58мм = 384 px, 80мм = 576 px (Bixolon-да 640 px — профильден алу!). Қаріп asset ретінде жеткізіледі.
- **Қосымша тәуекел (R6):** растр ~30–50 есе көп байт (80мм × 1000px ≈ 72 КБ vs мәтіндік ~1,5 КБ). Ethernet/USB/SUNMI-де мәселе жоқ, BT SPP-де 1–7 с, **BLE-де жарамсыз**.
- **Тексеру:** Epson TM-T20III + SUNMI + XPrinter XP-80: (а) растр чектің басылу уақыты, (б) 384/576 px-те қазақ әріптерінің оқылуы, (в) таңдалған қаріпте 18 глифтің болуы. **2 күн, приоритет №2.**

---

#### 🟠 R4 — Data Matrix маркировка кодындағы GS сепараторы жоғалады

- **Не істемейді:** HID сканер маркировка Data Matrix кодын оқығанда, ішіндегі **GS (0x1D)** сепараторын тастап кетеді немесе басқа символға айналдырады. Код ОФД-ға қате форматта жіберіледі.
- **Неге:** `[БОЛЖАМ]` HID режимі — пернетақта эмуляциясы, ал 0x1D-ге сәйкес перне жоқ. Әдепкі баптауда көп сканер оны тастайды; әр өндіруші бөлек конфиг штрихкодымен қосады.
- **Ықтималдық:** **Жоғары** (әдепкі баптауда — әрқашан дерлік)
- **Әсер:** **Маркировка заң талабын бұзады** (CLAUDE.md §3). Чек ОФД-дан қайтарылуы мүмкін.
- **Баламасы:**
  1. Сканерді «GS-ті сақтап жіберу» режиміне баптау — орнату нұсқаулығына кіруі керек.
  2. **HID орнына USB-COM режимі** — сепараторлар сол күйінде келеді. R1-дің баламасымен бір шешім.
  3. SUNMI/PAX ішкі сканері — AIDL/broadcast арқылы шикі деректер `[БОЛЖАМ]`.
  4. Қосымшада **валидация**: код күтілген құрылымға сай ма — сай болмаса кассирге «сканерді баптаңыз» деген нақты хабарлама.
- **Тексеру:** нақты маркировкаланған тауар (сыра/мотор майы) + 2–3 түрлі HID сканер. `[БҰҒАТТАЛҒАН]` ҚР маркировка Data Matrix форматын фискал зерттеушісінен алу.

---

#### 🟠 R5 — ESC/POS Flutter экожүйесінің тұрақсыздығы

- **Не істемейді:** таңдалған принтер пакеті жаңармай қалады, Flutter жаңа нұсқасында сынады, немесе қажет транспортты қолдамайды.
- **Неге:** `[РАСТАЛҒАН]` Аудиттегі 13 ESC/POS пакетінің: 1-уі `isDiscontinued: true`, 4-уі 2 жылдан ескі (2021-09-21, 2022-07-19, 2024-05-03, 2024-09-01), 2-уі **0.0.x**. Ең сапалы кандидат `unified_esc_pos_printer` — 160/160 ұпай, бірақ **9 likes, айына 1637 жүктеу, жалғыз әзірлеуші** (автобус факторы = 1).
- **Ықтималдық:** **Орта**
- **Әсер:** **Қолайсыздық → функцияны шектейді.** Жобаны тоқтатпайды, бірақ форк/қайта жазу шығынын береді.
- **Баламасы:** ✅ **ESC/POS байт генераторын ӨЗІМІЗ жазу.** Іс жүзінде **шағын әрі толық құжатталған** жұмыс: ESC/POS — 30 жылдық тұрақты стандарт, чек басу үшін ~15 команда жеткілікті (`ESC @`, `ESC t`, `ESC a`, `ESC !`, `GS !`, `GS v 0`, `GS ( k` QR, `GS k` штрихкод, `GS V` кесу, `ESC p` жәшік, `DLE EOT` күй). Сыртқы пакет тек **транспорт** үшін қалады, ол ауыстыруға оңай.
  - `escpos-printer-db`-нің `capabilities.json`-ын **өз ресурсымыз ретінде бекіту** (CC-BY-4.0, коммерциялық қолдануға рұқсат) — 50 профиль, code page нөмірлері, қағаз ені. Бұл R3-тегі «Rongta-да CP866=7» мәселесін де шешеді.
  - Таза Dart генератор `/packages/hardware/escpos`-та, 100% unit-тестпен (байт-байт салыстыру — **темірсіз тестіленеді!**).
  - Транспорт: Ethernet — `dart:io` (пакетсіз); Windows USB — `win32` (Microsoft API, ешқашан жоғалмайды); Android USB/BT — жалғыз пакетке тәуелділік.
- **Тексеру:** керек емес — архитектуралық шешім. Ұсыныс: **Фаза 2-де (Іргетас) жазу.**

---

### 11.2 Қалған тәуекелдер

| # | Тәуекел | Ықт. | Әсер | Балама / тексеру |
|---|---|---|---|---|
| R6 | **Растр басу тым баяу** (BT SPP-де 1–7 с, BLE-де жарамсыз) | Орта | Қолайсыздық | Ethernet/USB — әдепкі транспорт; BT SPP-де гибридті режим; BLE принтерлерін қолдамау деп жариялау. **Тексеру:** нақты принтерде уақыт өлшеу |
| R7 | **iOS-та толық касса құру мүмкін емес** (USB жоқ, BT SPP жоқ, SUNMI/PAX жоқ) | Жоғары (расталған) | Функцияны шектейді | iOS-ты MVP v1-ден шығару; iOS-та тек Ethernet принтер + камера сканері. **Өнім шешімі** |
| R8 | **`presentation_displays` 2023-12-15-тен жаңармаған** | Орта | Қолайсыздық | Тұтынушы дисплейін MVP-ден шығару; SUNMI LCD-мен шектелу. **Тексеру:** Flutter 3.3x + HDMI экран |
| R9 | **Android-та `/dev/ttyUSB*`-ке рұқсат жоқ** → таразы жұмыс істемеуі мүмкін | Орта | Функцияны шектейді | Таразыны MVP-ден шығару (салмақ штрихкоды жеткілікті); немесе Android USB Host API + өз FFI. **Тексеру:** Android планшет + USB-serial |
| R10 | **Android батарея оптимизациясы фондық синхрондауды өлтіреді** → 72 сағаттық кезек уақытында жіберілмейді | Орта | Функцияны шектейді (**заң талабы**) | `flutter_foreground_task` 11.0.1 + kiosk mode + тұрақты қуат. **Тексеру:** Android 13/14-те 24 сағаттық тест. **Фискал зерттеушісімен келісу** |
| R11 | **SQLCipher Windows-та жиналмайды** (CMake/OpenSSL, drift#3395) | Орта | Қолайсыздық | БД-ны шифрламау; құпияларды `flutter_secure_storage`-та; тұтастықты hash chain-мен. **Тексеру:** шифрлау шешімі қабылданса ғана |
| R12 | **Транзитивті тастанды тәуелділік** (`esc_pos_utils_plus` 2024-09-01, 50k жүктеу/ай) | Орта | Қолайсыздық | Өз ESC/POS генераторымыз (R5) бұл тәуекелді де жояды. **Тексеру:** `flutter pub deps` |
| R13 | **GPL лицензиялы пакетті байқаусыз қосу** (`flutter_esc_pos_network`, `digital_scale`) | Төмен | Жобаны тоқтатады (заңды) | CI-да лицензия тексеру қадамы (allowlist: MIT/BSD/Apache). **Тексеру:** CI ережесін жазу |
| R14 | **Code page нөмірін hardcode жасау** — Rongta-да CP866=7, Epson-да 17 | Орта | Функцияны шектейді | Профиль дерекқорынан алу міндетті. **Тексеру:** код ревьюінде |
| R15 | **Қағаз енін hardcode жасау** — Bixolon 80мм = 640 px, Epson = 576 px | Орта | Қолайсыздық | Профильден алу міндетті |
| R16 | **Растрда таңдалған қаріпте қазақ глифтері жоқ** немесе 384 px-те оқылмайды | Орта | Функцияны шектейді | Қаріпті asset ретінде жеткізу; 3–4 кандидатты сынау. **Тексеру:** нақты принтерде |
| R17 | **Windows-та BT принтер** `flutter_blue_plus_winrt` **0.0.20** форкіне тәуелді | Төмен | Қолайсыздық | Windows-та BT принтерін қолдамау деп жариялау (Ethernet/USB жеткілікті) |
| R18 | **Windows спулер RAW басуда драйвер байттарды бұрмалауы** | Төмен | Қолайсыздық | «Generic / Text Only» немесе өндірушінің RAW драйверін пайдалану. **Тексеру:** Windows + USB принтер |
| R19 | **SUNMI/PAX Қазақстанда қолжетімсіз болуы** | Белгісіз | Қолайсыздық | Архитектура hardware-agnostic болса — әсер жоқ. **Тексеру:** KZ дилерлерімен байланыс |

---

## 12. Қайшылықтар мен ашық сұрақтар

### 12.1 Дереккөздер арасындағы қайшылықтар

| # | Қайшылық | Дереккөз А | Дереккөз Б | Менің бағам |
|---|---|---|---|---|
| C1 | **`print_bluetooth_thermal`-дың iOS қолдауы** | pub.dev беті: iOS ✅, «Serial Port Profile (SPP)» деп жазады, MAC мекенжайымен қосылады | Apple: SPP тек MFi аксессуарларымен; `flutter_blue_plus`: «Bluetooth Classic is not supported» | `[БОЛЖАМ]` pub.dev сипаттамасы **шатастыратын**. iOS-та ол шын мәнінде BLE қолданады немесе қолдауы толық емес. **iOS-та BT принтерге растр басу үшін сүйенбеу керек** |
| C2 | **pub.dev платформа тегтері vs pubspec** | pub.dev беті: `unified_esc_pos_printer` — 6 платформа | pubspec `flutter.plugin.platforms`: тек `android, ios, windows` | `[РАСТАЛҒАН]` **pubspec — шындық.** pub.dev тегі Dart компиляциясынан шығады. §0.3 |
| C3 | **`thermal_printer_flutter` Android USB** | README: Android USB ❌ | `flutter_thermal_printer` (басқа пакет): Android USB ✅ | Екі бөлек пакет — қайшылық емес, бірақ атаулары шатастыратындай ұқсас. Құжаттамада толық атын жазу керек |
| C4 | **`flutter_libserialport` Android қолдауы** | pubspec: `android` бар | `[БОЛЖАМ]` Android-та `/dev/tty*`-ке рұқсат жоқ | Пакет «жиналады» дегенді білдіреді, «жұмыс істейді» дегенді емес → R9 |
| C5 | **Isar-дың мәртебесі** | pub.dev API: `isDiscontinued: false` | Соңғы релиз **2023-04-25** (3 жыл 4 ай); GitHub #1689 «Isar is dead» | `[РАСТАЛҒАН]` **Жариялану күні басым.** `isDiscontinued` жалауына ғана сүйенуге болмайды |
| C6 | **CLAUDE.md §4: «Flutter — Android, iOS, Windows бір кодтан»** | Жоба құжаты | §9.2: iOS-та USB/SPP/SUNMI/PAX/жәшік жоқ; §9.3: Windows-та сканер/камера/дисплей жоқ | `[БОЛЖАМ]` Болжам **толықтай дұрыс емес**. Код ортақ, бірақ **темір мүмкіндіктері ортақ емес** |
| C7 | **58мм-дегі таңба саны** | unified README: Font A = 32, Font B = 42 | Кәдімгі ESC/POS практикасы: Font A = 32, Font B = 42 (58мм) — сай келеді | Қайшылық жоқ, бірақ **дереккөз бір ғана** — профильден алу керек |

### 12.2 Басқа зерттеушілермен қайшы келуі мүмкін тұстар

**→ Эквайринг зерттеушісіне (терминал SDK):**

1. `[РАСТАЛҒАН]` PAX үшін **екі мүлдем бөлек архитектура** бар:
   - `nebula_flutter_plugin` (1.0.3, 2026-04-07): касса **бөлек құрылғыда**, PAX терминалы TCP/BT/USB/MQTT арқылы басқарылады
   - `flutter_pax_printer_utility` (0.1.6, 2025-10-15): касса **PAX терминалының өзінде** (PayDroid)
   Бұл екеуі PaymentAdapter интерфейсіне мүлдем әртүрлі талап қояды. **Қайсысы басым — эквайринг зерттеушісі анықтауы керек.**
2. `[БҰҒАТТАЛҒАН]` SUNMI-де эквайринг қосымшасын Android **Intent** арқылы шақыру (Kaspi Pay, Halyk Pay) — `docs.sunmi.com` бұғатталғандықтан тексерілмеді. Kaspi QR интеграциясы Intent арқылы ма, әлде REST арқылы ма?
3. `[БОЛЖАМ]` Егер эквайринг «Smart POS» (терминал = касса) моделін таңдаса, **Windows тармағы мәнін жоғалтады** (PAX/SUNMI — Android). Бұл темір стратегиясына тікелей әсер етеді және R1/R2-ні автоматты түрде шешеді.

**→ Фискал зерттеушісіне (чек басу):**

1. **Ең маңызды:** `[БОЛЖАМ]` Біз чекті **сурет (растр) ретінде** басуды ұсынамыз (§3). Егер ОФД/ҚР заңы чектің нақты мәтіндік форматын, белгілі бір қаріпті немесе таңба санын талап етсе — бұл шешім қайта қаралуы керек.
2. `[РАСТАЛҒАН]` **ҰТК (НКТ) тауар атаулары** ГБД-дан келеді және оларда қазақ әріптері болады — қазақ әріптерін басу міндетті, бұл мәселеден қашу мүмкін емес.
3. `[БОЛЖАМ]` **Фискалдық QR** — біз оны ESC/POS **нативті QR командасымен** (растр емес) басуды ұсынамыз. Егер фискал талабы QR-дың нақты өлшемін/error correction деңгейін көрсетсе — соны білу керек.
4. `[БОЛЖАМ]` **72 сағаттық автономды режим + Android батарея оптимизациясы** (R10). Заң «байланыс қалпына келгенде барлық чек кезекпен, ретімен жіберіледі» дейді. Егер қосымша фонда өлтірілсе, жіберу тек қосымша ашылғанда болады. **Бұл заң талабын қанағаттандыра ма?** Жауап «жоқ» болса, foreground service міндетті.
5. `[БОЛЖАМ]` **Маркировка Data Matrix + GS сепараторы** (R4). ҚР маркировка кодының нақты құрылымы қандай, ОФД-ға қандай форматта жіберіледі?
6. `[БОЛЖАМ]` **Чек ені**: 58мм (384 px, 32 таңба/жол) және 80мм (576 px, 48 таңба/жол) — екеуін де қолдау керек пе? Барлық міндетті реквизиттер 58мм-ге сыя ма?
7. `[РАСТАЛҒАН]` Тұтынушы дисплейлерінде (`OCD-100`, `OCD-300`, `AF-240`) де **RK1048 жоқ**, ал VFD-де растр мүмкіндігі жоқ. Егер заң тұтынушы дисплейінде қазақша мәтін талап етсе — шешімі жоқ.

### 12.3 Ашық сұрақтар (өнім шешімі керек)

| # | Сұрақ | Кімге | Ұсыныс |
|---|---|---|---|
| Q1 | iOS MVP v1-ге кіре ме? | Өнім иесі | **Кірмесін** (§9.2) |
| Q2 | Windows MVP v1-ге кіре ме? | Өнім иесі | R1-ді тексергеннен кейін шешу |
| Q3 | Тұтынушы дисплейі MVP-ге кіре ме? | Өнім иесі | **Кірмесін** (§6.3) |
| Q4 | Тікелей қосылатын таразы MVP-ге кіре ме? | Өнім иесі | **Кірмесін**, тек салмақ штрихкоды (§7.4) |
| Q5 | БД шифрлануы керек пе? | Қауіпсіздік + фискал | **Жоқ**, hash chain + secure storage (§1.4) |
| Q6 | ESC/POS генераторын өзіміз жазамыз ба? | Техлид | **Иә**, Фаза 2-де (R5) |
| Q7 | Қай принтер модельдері «ресми қолдау» тізіміне кіреді? | Өнім + сату | RK1048 бар Epson + растр арқылы қалғандары |
| Q8 | `escpos-printer-db` (CC-BY-4.0) жобаға ресурс ретінде кіре ме? | Техлид + заңгер | **Иә** — CC-BY коммерциялық қолдануға рұқсат етеді, атрибуция қажет |

---

## 13. `[БҰҒАТТАЛҒАН]` тізімі

### 13.1 Нақты құрылғы керек

| # | Не тексеріледі | Керекті темір | Приор. | Уақыт |
|---|---|---|---|---|
| **B1** | **Windows-та HID сканер жұмыс істей ме** (R1) — `KeyboardListener`, USB-COM режимі, `RegisterRawInputDevices` | Windows 11 + USB 2D HID сканер | 🔴 №1 | 1 күн |
| **B2** | **Қазақ әріптері растрда оқыла ма** (R3, R16) — 384 px және 576 px | Epson TM-T20III + XPrinter XP-80 + SUNMI | 🔴 №2 | 2 күн |
| **B3** | **Растр басу уақыты** (R6) — Ethernet vs USB vs BT SPP vs BLE | Сол принтерлер + BT принтер | 🔴 №3 | 1 күн |
| B4 | **Epson-да KZ-1048 (page 53) шынымен басады ма** — `ESC t 53` + RK1048 байттары | Epson TM-T20III / TM-T88V | 🟠 | 0,5 күн |
| B5 | **SUNMI-де қазақша тек растрмен шыға ма** (§2.5, `Sunmi-V2` профилінде RK1048/CP1251 жоқ) | SUNMI V2 Pro / T2 mini | 🟠 | 0,5 күн |
| B6 | **drift + sqlite3 3.5.x Windows-та DLL-сіз жиналады ма** | Windows машина | 🟠 | 0,5 күн |
| B7 | **Windows спулер RAW басуы** (R18) — драйвер байттарды бұрмалай ма | Windows + USB термопринтер | 🟠 | 0,5 күн |
| B8 | **Data Matrix + GS сепараторы** (R4) — HID сканер оны сақтай ма | 2–3 түрлі 2D сканер + маркировкаланған тауар | 🟠 | 1 күн |
| B9 | **Android-та `flutter_libserialport`** `/dev/ttyUSB0`-ді аша ма (R9) | Android планшет + USB-serial адаптер | 🟡 | 0,5 күн |
| B10 | **Android батарея оптимизациясы** синхрондауды өлтіре ме (R10) — 24 сағаттық тест | Android 13/14 планшет | 🟡 | 2 күн (күту) |
| B11 | **`presentation_displays` жаңа Flutter-де жұмыс істей ме** (R8) | Android планшет + HDMI экран | 🟡 | 0,5 күн |
| B12 | **Ақша жәшігі** `ESC p` vs `DLE DC4` — қайсысы сенімдірек | RJ11 жәшік + принтер | 🟡 | 0,5 күн |
| B13 | **SUNMI ішкі принтер + сканер + LCD** — `sunmi_printer_plus` 4.1.1 (2025-09-05) жаңа құрылғыларда | SUNMI V2 Pro / T2 mini | 🟡 | 1 күн |
| B14 | **Bixolon (640 px!) / Rongta (CP866=7)** — профиль деректері дұрыс па | SRP-S300, RP80 | 🟢 | 0,5 күн |

### 13.2 Өндірушіден құжаттама/SDK сұрау керек

| # | Не керек | Кімнен | Себеп |
|---|---|---|---|
| B15 | **SUNMI dual-screen SDK** + Flutter wrapper бар ма | SUNMI (`docs.sunmi.com` ⛔) | Тұтынушы дисплейі |
| B16 | **SUNMI эквайринг Intent** сипаттамасы | SUNMI ⛔ | Эквайринг зерттеушісімен ортақ |
| B17 | **PAX NeptuneLite / Nebula SDK** толық құжаттамасы, NDA керек пе | PAX (`paxtechnology.com` ⛔) | PAX адаптері |
| B18 | **Posiflex** Android/Windows SDK-сы | Posiflex | pub.dev-те пакет мүлдем жоқ |
| B19 | **Epson ESC/POS Command Reference** толық PDF (page 53 кестесімен) | Epson (`download4.epson.biz` ⛔) | KZ-1048 байт кестесі |
| B20 | **KZ-1048 mapping кестесі** (0x80–0xFF → Unicode) | unicode.org / IANA (екеуі де ⛔) | Кодтауыш жазу үшін. **Балама:** GNU iconv-тан жергілікті шығару |
| B21 | **Қазақстанда қолданылатын таразылар** және хаттамалары | KZ дилерлер | §7.3 |

### 13.3 Қазақстан нарығы бойынша

| # | Не тексеріледі | Приор. |
|---|---|---|
| B22 | **ҚР-да ең көп сатылатын чек принтерлері** (XPrinter? Epson? Rongta?) | 🔴 — R3 шешімі осыған тәуелді |
| B23 | SUNMI-дің ҚР дистрибьюторы, модельдері, бағасы, кепілдігі | 🟠 |
| B24 | PAX-тың ҚР жағдайы (банктер қандай терминал таратады?) | 🟠 |
| B25 | ҚР-да қолданылатын HID сканер модельдері | 🟡 |
| B26 | Posiflex ҚР-да сатыла ма | 🟢 |

### 13.4 Осы сессияда зерттелмеген (WebSearch лимиті / бұғатталған домендер)

- Windows-та `RegisterRawInputDevices` арқылы Flutter-де HID оқудың дайын мысалы бар ма (R1 баламасы №3)
- CAS / Mettler Toledo хаттамаларының ресми құжаттамасы
- Bixolon **чек** принтерлерінің толық код беттері (тек `SRP-S300` профилі бар)
- Rongta принтерлерінің толық спецификациясы
- ҚР маркировка Data Matrix форматының спецификациясы (фискал зерттеушісінің тақырыбы)
- Dynamsoft / Scanbot Windows Flutter SDK бағасы (R2 коммерциялық баламасы)

---

## 14. Дереккөздер тізімі

> Барлығы **2026-09-01** күні қаралды. `⛔` — егресс-проксимен бұғатталған.
> **API дереккөздері** — `https://pub.dev/api/packages/<name>` және `.../score` (машиналық оқылды).

### 14.1 pub.dev (нұсқа + нақты жариялану күні + pubspec платформалары)

| Пакет | URL | Нұсқа | Жарияланған |
|---|---|---|---|
| drift | <https://pub.dev/packages/drift> | 2.34.3 | 2026-07-27 |
| sqlite3 | <https://pub.dev/packages/sqlite3> | 3.5.2 | 2026-08-19 |
| sqlite3_flutter_libs | <https://pub.dev/packages/sqlite3_flutter_libs> | 0.6.0+eol | 2026-02-15 |
| sqlcipher_flutter_libs | <https://pub.dev/packages/sqlcipher_flutter_libs> | 0.7.0+eol | 2026-02-15 |
| sqflite | <https://pub.dev/packages/sqflite> | 2.4.3 | 2026-06-02 |
| sqlite_async | <https://pub.dev/packages/sqlite_async> | 0.14.5 | 2026-08-27 |
| objectbox | <https://pub.dev/packages/objectbox> | 5.3.2 | 2026-05-20 |
| isar | <https://pub.dev/packages/isar> | 3.1.0+1 | **2023-04-25** |
| isar_plus | <https://pub.dev/packages/isar_plus> | 1.3.9 | 2026-08-04 |
| realm | <https://pub.dev/packages/realm> | 20.2.0 | **2025-09-24** |
| unified_esc_pos_printer | <https://pub.dev/packages/unified_esc_pos_printer> | 3.4.0 | 2026-07-16 |
| flutter_thermal_printer | <https://pub.dev/packages/flutter_thermal_printer> | 2.2.2 | 2026-08-25 |
| thermal_printer_flutter | <https://pub.dev/packages/thermal_printer_flutter> | 2.0.0+3 | 2026-06-08 |
| print_bluetooth_thermal | <https://pub.dev/packages/print_bluetooth_thermal> | 1.2.2 | 2026-05-23 |
| esc_pos_utils_plus | <https://pub.dev/packages/esc_pos_utils_plus> | 2.0.4 | **2024-09-01** |
| esc_pos_printer | <https://pub.dev/packages/esc_pos_printer> | 4.1.0 | **2021-09-21** |
| esc_pos_printer_plus | <https://pub.dev/packages/esc_pos_printer_plus> | 0.1.1 | 2025-02-11 |
| blue_thermal_printer | <https://pub.dev/packages/blue_thermal_printer> | 1.2.3 | **2022-07-19** |
| flutter_pos_printer_platform | <https://pub.dev/packages/flutter_pos_printer_platform> | 1.4.2 | **2023-07-14** (discontinued) |
| flutter_esc_pos_network | <https://pub.dev/packages/flutter_esc_pos_network> | 1.0.3 | **2024-05-03** (GPL) |
| nitro_printing | <https://pub.dev/packages/nitro_printing> | 0.0.6 | 2026-08-29 |
| windows_escpos_engine | <https://pub.dev/packages/windows_escpos_engine> | 0.0.3 | 2026-07-19 |
| image | <https://pub.dev/packages/image> | 4.9.2 | 2026-08-19 |
| mobile_scanner | <https://pub.dev/packages/mobile_scanner> | 7.4.0 | 2026-07-20 |
| google_mlkit_barcode_scanning | <https://pub.dev/packages/google_mlkit_barcode_scanning> | 0.16.1 | 2026-08-17 |
| — BarcodeFormat enum | <https://pub.dev/documentation/google_mlkit_barcode_scanning/latest/google_mlkit_barcode_scanning/BarcodeFormat.html> | — | — |
| charset_converter | <https://pub.dev/packages/charset_converter> | 2.5.1 | 2026-07-18 |
| charset_codec | <https://pub.dev/packages/charset_codec> | 0.1.1 | 2026-07-29 |
| flutter_blue_plus | <https://pub.dev/packages/flutter_blue_plus> | 2.3.12 | 2026-08-10 |
| flutter_blue_plus_winrt | <https://pub.dev/packages/flutter_blue_plus_winrt> | 0.0.20 | 2026-05-08 |
| win32 | <https://pub.dev/packages/win32> | 6.4.0 | 2026-08-05 |
| kiosk_mode | <https://pub.dev/packages/kiosk_mode> | 0.9.0 | 2026-07-29 |
| flutter_foreground_task | <https://pub.dev/packages/flutter_foreground_task> | 11.0.1 | 2026-08-20 |
| flutter_secure_storage | <https://pub.dev/packages/flutter_secure_storage> | 11.0.0 | 2026-08-06 |
| presentation_displays | <https://pub.dev/packages/presentation_displays> | 1.0.0 | **2023-12-15** |
| flutter_libserialport | <https://pub.dev/packages/flutter_libserialport> | 0.6.0 | 2025-08-01 |
| usb_serial | <https://pub.dev/packages/usb_serial> | 0.5.2 | **2024-07-12** |
| digital_scale | <https://pub.dev/packages/digital_scale> | 1.2.3+1 | 2026-08-19 (GPL) |
| code_scan_listener | <https://pub.dev/packages/code_scan_listener> | 0.4.1 | **2023-07-05** |
| sunmi_printer_plus | <https://pub.dev/packages/sunmi_printer_plus> | 4.1.1 | 2025-09-05 |
| sunmi_scanner | <https://pub.dev/packages/sunmi_scanner> | 0.0.8 | 2026-08-17 |
| sunmi_kali | <https://pub.dev/packages/sunmi_kali> | 1.3.1 | 2026-06-12 |
| flutter_pax_printer_utility | <https://pub.dev/packages/flutter_pax_printer_utility> | 0.1.6 | 2025-10-15 |
| nebula_flutter_plugin | <https://pub.dev/packages/nebula_flutter_plugin> | 1.0.3 | 2026-04-07 |
| drift API docs | <https://pub.dev/documentation/drift/latest/> | — | — |
| DriftIsolate | <https://pub.dev/documentation/drift/latest/isolate/DriftIsolate-class.html> | — | — |

### 14.2 GitHub (шикі файлдар және issue-лар)

**`escpos-printer-db` — негізгі темір дерекқоры (CC-BY-4.0):**
- Репозиторий — <https://github.com/receipt-print-hq/escpos-printer-db>
- `dist/capabilities.json` (133 КБ, толық жүктелді, 50 профиль талданды) — <https://raw.githubusercontent.com/receipt-print-hq/escpos-printer-db/master/dist/capabilities.json>
- `data/encoding.yml` (188 кодтау; RK1048 жазбасы 625–626 жолдар) — <https://github.com/receipt-print-hq/escpos-printer-db/blob/master/data/encoding.yml>
- `data/profile/default.yml` (`53: RK1048`, `46: CP1251`, `17: CP866`, `33: CP775`, `34: CP855`)
- Issue #36 «Add printer to database Xprinter XP-58» — <https://github.com/receipt-print-hq/escpos-printer-db/issues/36>

**Flutter engine/framework issue-лары:**
- **#79849** «Flutter Windows Keyboard Issue using external barcode scanner» — **Open**, 2021-04-06, P2, `platform-windows`/`team-windows`/`triaged-windows`/`engine` — <https://github.com/flutter/flutter/issues/79849>
- #154141 (HardwareKeyboard глобалды тыңдау) — <https://github.com/flutter/flutter/issues/154141>
- #162305 (KeyboardListener KeyUpEvent багы, JS/WASM) — <https://github.com/flutter/flutter/issues/162305>
- #136419 (RawKeyEvent deprecation) — <https://github.com/flutter/flutter/issues/136419>

**Пакет репозиторийлері:**
- `win32` RAW басу мысалы — <https://github.com/halildurmus/win32/blob/main/examples/printer_raw.dart>
- `unified_esc_pos_printer` — <https://github.com/elrizwiraswara/unified_esc_pos_printer> (2 ашық issue: #25, #23)
- `mobile_scanner` — <https://github.com/juliansteenbakker/mobile_scanner>
- `sunmi_scanner` — <https://github.com/FrenkyDema/sunmi_scanner>
- `sunmi_printer_plus` — <https://github.com/brasizza/sunmi_printer/>
- `flutter_pax_printer_utility` — <https://github.com/AuliaVailo/flutter_pax_printer_utility>
- `flutter_foreground_task` — <https://github.com/Dev-hwang/flutter_foreground_task>
- `kiosk_mode` (mews-flutter монорепосы) — <https://github.com/MewsSystems/mews-flutter>
- `presentation_displays` — <https://github.com/VNAPNIC/presentation-displays>
- `digital_scale` — <https://github.com/sergiotucano/digital_scale>
- `code_scan_listener` — <https://github.com/utamori/code_scan_listener>
- `esc_pos_utils` enums (drawer pins) — <https://github.com/andrey-ushakov/esc_pos_utils/blob/master/lib/src/enums.dart>

**БД мәртебесі:**
- drift #3395 (SQLCipher Windows OpenSSL, жабылған `docs` белгісімен) — <https://github.com/simolus3/drift/issues/3395>
- drift #2748 (MultiExecutor, WAL, изоляттар) — <https://github.com/simolus3/drift/discussions/2748>
- isar #1689 «Isar is dead, long live Isar» — <https://github.com/isar/isar/issues/1689>
- isar #1581 «Is this project still alive?» — <https://github.com/isar/isar/discussions/1581>
- realm-js #6884 (Device Sync deprecation, EOL 2025-09-30) — <https://github.com/realm/realm-js/discussions/6884>
- realm-swift #8680 «Realm is Deprecated/Dead» — <https://github.com/realm/realm-swift/discussions/8680>

### 14.3 Өндіруші / стандарт (көбі ⛔)

- Apple External Accessory (MFi) — <https://developer.apple.com/documentation/externalaccessory>
- ⛔ Epson «Page 53 [KZ-1048: Kazakhstan]» — <https://download4.epson.biz/sec_pubs/pos/reference_en/charcode/page_53.html>
- ⛔ Epson Character Code Tables — <https://download4.epson.biz/sec_pubs/pos/reference_en/charcode/supported_codepage.html>
- ⛔ Epson `ESC t` — <https://download4.epson.biz/sec_pubs/pos/reference_en/escpos/esc_lt.html>
- ⛔ Epson `ESC p` (ақша жәшігі) — <https://download4.epson.biz/sec_pubs/pos/reference_en/escpos/esc_lp.html>
- ⛔ Epson Customer Display Code Tables — <https://download4.epson.biz/sec_pubs/pos/reference_en/charcode_dm/supported_codepage_dm.html>
- Epson TM-T20 ESC/POS Quick Reference (PDF) — <http://www.novopos.ch/client/EPSON/TM-T20/TM-T20_eng_qr.pdf>
- ⛔ IANA KZ-1048 тіркеуі — <https://www.iana.org/assignments/charset-reg/KZ-1048>
- ⛔ Unicode KZ1048.TXT mapping — <https://www.unicode.org/Public/MAPPINGS/VENDORS/MISC/KZ1048.TXT>
- Compart KZ-1048 — <https://www.compart.com/en/unicode/charsets/KZ-1048>
- fileformat.info KZ-1048 — <https://www.fileformat.info/info/charset/KZ-1048/index.htm>
- ⛔ Flutter key event migration — <https://docs.flutter.dev/release/breaking-changes/key-event-migration>
- ⛔ Flutter `RawKeyboardListener` API — <https://api.flutter.dev/flutter/widgets/RawKeyboardListener-class.html>
- ⛔ SUNMI developer docs — <https://docs.sunmi.com/en/>
- ⛔ drift Isolates / Encryption — <https://drift.simonbinder.eu/isolates/> · <https://drift.simonbinder.eu/platforms/encryption/>
- Xprinter XP-80T нұсқаулығы (код беттері) — <https://manuals.plus/xprinter/xp-80t-thermal-receipt-printer-manual>
- Xprinter XP-T58 нұсқаулығы — <https://manuals.plus/m/4e2a3a70a8704029d0c80f079e49413c2ab2650213dbff9786d2c2152aeb49ea>
- Bixolon Label Printer Code Pages (PDF) — <https://www.bixolon.com/_upload/manual/Manual_Label_Printer_Code_Pages_english_V2.02.pdf>
- Diebold Nixdorf P1200 Programming Manual (PDF) — <https://www.dieboldnixdorf.com/-/media/diebold/files/retail/peripherals-en/printers/p1200-prog-manual.pdf>

### 14.4 Жанама (блог, форум, дилер)

- «The Flutter Local Database Landscape in 2026» — <https://luci-studio.com/blog/the-flutter-local-database-landscape-in-2026-a-maintenance-first-guide-fe6d267c/>
- ⛔ mike42 «How to print the characters in an ESC/POS printer code page» — <https://mike42.me/blog/2018-05-how-to-print-the-characters-in-an-esc-pos-printer-code-page>
- Честный знак — Data Matrix сканерін тексеру — <https://markirovka.ru/knowledge/fast_start/start/proverka-skanera-data-matrix-instruktsiya>
- POS-Center — сканер Data Matrix оқи ма — <https://pos-center.ru/journal/podhodit-li-skaner-shtrih-kodov-dlya-schityvaniya-data-matrix/>
- Mertech — SUNMI L2S сканерін баптау — <https://help.mertech.ru/tsd/Mertech_Sunmi/nastroyka_L2S.html>
- Клеверенс — SUNMI L2K сипаттамасы — <https://www.cleverence.ru/hardware/mdc/sunmi/sunmi-l2k/4085/>
- ⛔ Wikipedia Windows-1251 — <https://en.wikipedia.org/wiki/Windows-1251>
- DantSu ESCPOS-ThermalPrinter-Android #185 (CP866-да қытай иероглифтері) — <https://github.com/DantSu/ESCPOS-ThermalPrinter-Android/issues/185>
- QZ Tray Raw Encoding — <https://qz.io/docs/raw-encoding>
- NielsLeenheer EscPosEncoder — <https://github.com/NielsLeenheer/EscPosEncoder>

---

*Есеп соңы. Фаза 1 зерттеу материалы. Ешбір тұжырым нақты темірде тексерілмеген — §13-ті орындамай тұрып «жасалды» деп есептеуге болмайды.*
