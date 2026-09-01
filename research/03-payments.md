# 03 — Эквайринг және төлемдер

**Зерттеу күні:** 2026-09-01
**Автор:** research subagent (Фаза 1)
**Мәртебе:** аяқталған, бірақ **құрал шектеулері салдарынан жартылай тексерілген** (§0 қара)

---

## 0. Әдістеме және шектеулер

### 0.1 Ортаның желі саясаты (үйлестіруші агент `curl`-мен өлшеген)

**РҰҚСАТ ЕТІЛГЕН** (WebFetch жұмыс істейді):
`github.com`, `api.github.com`, `raw.githubusercontent.com`, `gist.github.com`,
`objects.githubusercontent.com`, `pub.dev`, `registry.npmjs.org`, `pypi.org`,
`files.pythonhosted.org`, `index.crates.io`, `proxy.golang.org`.

**БҰҒАТТАЛҒАН** (proxy CONNECT-ке 403 — ұйым саясаты, қайталаудың мәні жоқ):
**барлық `.kz` домендері** (`kaspi.kz`, `guide.kaspi.kz`, `halykbank.kz`,
`developer.homebank.kz`, `epayment.kz`, `ioka.kz`, `docs.ioka.kz`, `freedompay.kz`,
`jusan.kz`, `forte.kz`, `docs.fortebank.com`, `developers.tiptoppay.kz`,
`bankffin.kz`, `kaspipay.kz`, `digitalbusiness.kz`, `kz.kursiv.media`,
`tandemsoft.kz`, `arida.kz`, `pos-plugin.kz`, `1s-expert.kz`, `grprof.com`),
сондай-ақ `freedompay.kg`, `joinposter.com`, `infostart.ru`, `docs.sunmi.com`,
`blog.rospertech.com`, wikipedia, stackoverflow, medium, `*.github.io`.

### 0.2 Қолданылған құралдар және олардың күйі

| Құрал | Күйі |
|---|---|
| `WebSearch` | Жұмыс істеді, бірақ сессия лимиті (200/200) зерттеу ортасында **таусылды**. |
| `WebFetch` → GitHub / raw.githubusercontent / pub.dev | ✅ Жұмыс істеді — **жалғыз нақты бастапқы дереккөз**. |
| `WebFetch` → банк/вендор сайттары | ❌ 403. Ешбір ресми тариф не doc беті ашылмады. |
| `github.com/search` HTML | Ішінара — соңында **HTTP 429** (сағатқа бұғатталды). |
| `mcp__github__*` | Жеке репозиторийге шектелген, **қолданылмады** (үйлестіруші нұсқауы). |

### 0.3 `[РАСТАЛҒАН]` таңбасының қолданылу тәртібі және саны

Бұл есепте `[РАСТАЛҒАН]` **тек өз көзіммен оқыған файлға** қойылды:
GitHub-тағы вендордың **өз ұйымының** репозиторийі, немесе pub.dev-тегі
**verified publisher** беті. Барлық басқасы — `[ЖАНАМА]`.

Есепте екі деңгей бар:

**A. `[РАСТАЛҒАН]` — 4 тұжырым, барлығы ioka бойынша** (вендордың ӨЗ ұйымының
репозиторийі немесе pub.dev verified publisher):

| # | Тұжырым | Оқылған дереккөз |
|---|---|---|
| 1 | ioka-ның ресми GitHub ұйымы `iokadev` және 4 SDK репозиторийі | `github.com/orgs/iokadev/repositories` |
| 2 | pub.dev-те `ioka` пакеті, publisher **ioka.kz**, v1.0.2, ~3 жыл жаңармаған | `pub.dev/packages?q=ioka` |
| 3 | ioka API-дың **толық OpenAPI спецификациясы**: base URL, ~40 эндпоинт, статус enum-дары, аутентификация схемалары | `raw.githubusercontent.com/iokadev/ioka-flutter/main/lib/src/api/ioka_api.json` |
| 4 | ioka-да webhook — бірінші класты мүмкіндік (`/v2/webhooks` CRUD) | сол спецификация |

**B. `[РАСТАЛҒАН — код деңгейінде]` — 5 тұжырым.** Кодты өз көзіммен оқыдым,
бірақ репозиторий вендордың ресми ұйымына тиесілі емес немесе вендор растауы емес:

| # | Тұжырым | Оқылған файл |
|---|---|---|
| 5 | TipTopPay base URL `https://api.tiptoppay.kz/` + эндпоинттер | `tiptoppaymobile/TipTopPay-SDK-iOS` (ресми ұйым, бірақ pub.dev растауы жоқ) |
| 6 | CloudPayments KZ → TipTopPay ребрендингі | сол репозиторий |
| 7 | `api.freedompay.kz` == `api.paybox.money` (бір платформа) | `darkhan-b/arenatickets_back` — үшінші тұлға |
| 8 | SunmiPay AIDL пакет аттары | `OmniAid-DevelopmentTeam/capacitor-sunmi-pay-plugin` — үшінші тұлға |
| 9 | Flutter-PAX-Terminal плагинінің бар екені және README-і бос екені | `androiddevnotesforks/Flutter-PAX-Terminal` |

**Қалғанының бәрі — `[ЖАНАМА]`, `[ЖАНАМА, кросс-тексерілген]` немесе `[БОЛЖАМ]`.**
Соның ішінде **барлық комиссия пайыздары** және **барлық Kaspi/Halyk эндпоинт аттары**.

### 0.4 Осының салдары — нені ТЕКСЕРЕ АЛМАДЫМ

1. **Ешбір банктің тариф беті ашылмады.** §8-дегі барлық пайыз — `[ЖАНАМА]`,
   WebSearch сниппеті арқылы. **Шарт жасамас бұрын жазбаша тариф парағы міндетті.**
2. **Kaspi Smart POS интеграция PDF-і ашылмады** → нақты эндпоинт жолдары белгісіз.
3. **Halyk QR API мен ePay құжаттамасы ашылмады** → эндпоинт аттары `[ЖАНАМА]`.
4. **ForteBank, Freedom Pay, TipTopPay doc порталдары ашылмады.**
5. **МСМП (бірыңғай QR) техникалық құжаттамасы табылмады.**

### 0.5 Ойдан шығармаған нәрселерім

- Ешбір эндпоинт/өріс атын ойдан жазбадым.
- Комиссия пайызы **тек сілтемемен** және **тек `[ЖАНАМА]` таңбасымен** жазылған.
- Табылмағандар «**Таппадым**» деп ашық көрсетілген.
- Үшінші тұлғаның GitHub кодынан алынған, бірақ бірнеше тәуелсіз репозиторийде
  сәйкес келген деректер `[ЖАНАМА, кросс-тексерілген]` деп белгіленді.

---

## 1. Провайдерлер бойынша салыстыру кестесі

Белгілер: ✅ бар / ❌ жоқ / ❓ белгісіз / 🔒 шартпен (жабық)

| Провайдер | Ашық dev docs | Кассадан **терминалға** сумма жіберу | Динамикалық QR API | SoftPOS | Refund API | Sandbox |
|---|---|---|---|---|---|---|
| **Kaspi Smart POS** (жергілікті API) | 🔒 PDF, guide.kaspi.kz-та жарияланған | ✅ HTTP/LAN, порт 8080 | ✅ терминал арқылы | ✅ Kaspi Pay қосымшасы | ✅ (тек сол әдіспен) | ❓ |
| **Kaspi QR API** (сервер→сервер) | 🔒 серіктестік шарты | ❌ (терминалсыз) | ✅ `/qr/create` | — | ✅ `/payment/return` | ✅ `mtokentest.kaspi.kz` |
| **Halyk ePay 2.0** | ✅ ашық (epayment.kz / developer.homebank.kz) | ❌ (интернет-эквайринг) | ✅ QR API бөлек | ✅ Halyk Pos | ✅ `/operation/{id}/cancel` | ✅ `testepay.homebank.kz` |
| **Halyk POS терминал** | 🔒 1С драйвері арқылы ғана | ✅ 1С драйвері (Windows) | ❓ | — | ✅ | ❓ |
| **Jusan / Alatau City Bank** | ❌ (ib.jusan.kz — тек банк-клиент) | ✅ 1С драйвері (PAX A930) | ❓ | ❓ | ✅ | ❓ |
| **ForteBank** | ✅ `docs.fortebank.com` (ең сапалы) | ✅ 1С драйвері (VeriFone VX520) | ❓ | ✅ FortePOS | ✅ `/transactions/refunds` | ✅ Postman коллекциясы |
| **Freedom Pay** (ex-PayBox) | ✅ `docs.freedompay.kz` | ❌ | ✅ (QR + fiscal модулі) | ❓ | ✅ `revoke.php` | ❓ |
| **ioka** | ✅ **OpenAPI оқылды** | ❌ | ❌ (спецификацияда жоқ) | ❌ | ✅ `/refunds` | ✅ `stage-api.ioka.kz` |
| **TipTopPay** (ex-CloudPayments KZ) | ✅ `developers.tiptoppay.kz` | ❌ | ❓ | ❌ | ✅ | ✅ test Public ID |
| **PayBox.money** | ✅ (Freedom Pay-мен бір платформа) | ❌ | ❓ | ❌ | ✅ | ❓ |
| **Wooppay / WoopKassa** | ❌ Таппадым | ❌ | ❓ | ❓ | ❓ | ❓ |
| **Robokassa KZ** | ❌ **Таппадым** (іздеу лимиті таусылды) | ❓ | ❓ | ❓ | ❓ | ❓ |

---

## 2. Kaspi

Kaspi-де **бір емес, кемінде үш бөлек интеграция жолы** бар. Оларды шатастыруға болмайды —
бұл жобадағы ең жиі кездесетін қате.

| Жол | Не үшін | Кімге беріледі |
|---|---|---|
| **A. Smart POS жергілікті API** | Кассадан Kaspi терминалына сумма жіберу | Kaspi Smart POS иесі + интеграция инструкциясы |
| **B. Kaspi QR API** (server-to-server) | Терминалсыз, өз QR-ыңды генерациялау | Тек серіктестік шартымен |
| **C. Kaspi Merchant/Shop API** | Kaspi Магазин тапсырыстары (маркетплейс) | Сатушы кабинеті, `X-Auth-Token` |

**Бізге керегі — A (MVP), кейін B.** C — бұл эквайринг емес, маркетплейс.

### 2.1 A — Kaspi Smart POS жергілікті интеграция API

`[ЖАНАМА]` Іздеу индексінде Kaspi сайтындағы ашық PDF көрінеді (авторизациясыз URL),
бірақ домен 403 берді — **аша алмадым**:
- ru: `https://guide.kaspi.kz/cdn/content/pay/product/documents/Kaspi%20POS/Smart-POS-Dokymentatsia-po-integratsii.pdf`
- kk: `https://guide.kaspi.kz/cdn/content/pay/product/documents/Kaspi%20POS/Smart-POS-Integratsiyalay-zhonindegi-kyzhattama.pdf`

⚠ **WebFetch бұл PDF-ті аша алмады** (`guide.kaspi.kz` бұғатталған). Төмендегі мазмұн —
WebSearch қысқаша баяндауынан, сондықтан `[ЖАНАМА]`.

`[ЖАНАМА, кросс-тексерілген]` Архитектура:
- Транспорт: **жергілікті желі, HTTP(S)**. Smart POS-та `httpd` сервер көтеріледі,
  ол **KaspiPay қосымшасымен бірге іске қосылады**, **8080 портында** тыңдайды.
- Кассадан қосылу: Smart POS-тың **IP-адресі немесе DNS аты** бойынша.
- Kaspi **Ethernet (LAN) ұсынады** — Wi-Fi кідірістері интеграцияға зиян.
- Аутентификация: касса `register` сұрауын жібереді → Smart POS экранында
  «API-ге қатынау» сұрағы шығады → «Разрешить» → касса **`accessToken`**
  (және `revoke token`) алады. Токен әр сұрауда HTTPS Header-де жіберіледі.
- Токеннің қолданылу мерзімі — **24 сағат**, `expirationDate` өрісі беріледі,
  жаңарту `revoke` әдісі арқылы.
- API қорғанысы Smart POS-тың «Админ панелі» → «Защита интеграции» ішінде
  әр терминал үшін **жеке қосылады**.
- ⚠ **Қайшылық:** бір дереккөз «API қорғанысы SSL + токен» десе, екіншісі
  «**аутентификация жоқ, желі деңгейінде оқшаулау керек**» дейді (§11.2 қара).

`[ЖАНАМА]` Операция ағыны:
1. Касса **транзакция дайындау** сұрауын жібереді (`payment` немесе `refund`) →
   жауапта **`processId`** — интеграция операциясының бірегей идентификаторы.
2. Клиент терминалда картаны тигізеді немесе QR сканерлейді.
3. Касса `processId` бойынша **мәртебені сұрайды**.
4. Транзакциялар Smart POS жадында **24 сағат сақталады** — байланыс үзілсе,
   касса сол 24 сағат ішінде `processId` арқылы нәтижені қайта сұрай алады.
   → **Бұл дәл біздің «нәтиже белгісіз» проблемасының шешімі.**

`[ЖАНАМА]` Қатаң ереже: **қайтару төлеммен бірдей әдіспен жасалуы керек.**
QR-мен төленгенді картамен қайтаруға болмайды және керісінше.
→ Домен моделінде `PaymentMethod` чекпен бірге **міндетті түрде сақталуы керек**.

`[БОЛЖАМ]` Нақты эндпоинт жолдары (`/v2/payment`, `/v2/status` сияқты) маған **белгісіз**.
Іздеу нәтижелерінде де, GitHub-та да табылмады. **Ойдан жазбаймын.**
PDF-ті алу — §12 [БҰҒАТТАЛҒАН] тізімінде.

`[ЖАНАМА]` Терминал модельдері: **Smart POS A80 және A90** сериялары
(iiko плагині құжатынан). ⚠ Бұл PAX A80/A920 ма — тексерілмеген.

`[ЖАНАМА]` **Kaspi Касса қосылып тұрса, үшінші тарап плагині жұмыс істемейді**
(pos-plugin.kz нұсқаулығы). → Бұл бизнес тәуекел: Kaspi өз кассасын таңдағанда
біздің касса терминалға жете алмауы мүмкін.

### 2.2 B — Kaspi QR API (server-to-server)

`[ЖАНАМА, кросс-тексерілген]` — 4 тәуелсіз GitHub жобасынан алынды
(`thedakeen/kaspi-api-wrapper`, `HolySxn/KaspiQR-Wrapper`,
`altynbek132/supabase-project-kaspi`, `burcev-alex/kaspi-qr-sdk`, `RomanBronevik/kaspi-qr`).

**Base URL (тест):**
- `https://mtokentest.kaspi.kz:8543/r1/v01` — API-Key схемасы
- `https://mtokentest.kaspi.kz:8545/r3/v01` — сертификатты (enhanced) схема
- `https://qrapi-cert-ip.kaspi.kz` — PHP SDK-да көрсетілген тест хосты
- Production base URL — **Таппадым** (шартпен беріледі).

**Үш схема (сенімділік деңгейі бойынша):**

| Схема | Префикс | Аутентификация | Мүмкіндіктер |
|---|---|---|---|
| Basic / EASY | `/r1/v01` | `Api-Key` header | QR, төлем мәртебесі |
| Standard | `/r2/v01` | mTLS (`public.cer` + `private.key` + `combined_ca.pem`) | + қайтару |
| Enhanced / STRONG | `/r3/v01` | mTLS | + қашықтан төлем (remote) |

**Эндпоинттер:**

```
GET  /partner/tradepoints                  → [{ TradePointId, TradePointName }]
POST /device/register                      → { DeviceToken }
POST /device/delete
POST /qr/create                            → { QrToken, ExpireDate, QrPaymentId,
                                               PaymentMethods, QrPaymentBehaviorOptions }
POST /qr/create-link                       → { PaymentLink, ExpireDate, PaymentId, ... }
GET  /v02/payment/status/{qrPaymentId}     → { Status, TransactionId, Amount, ... }
GET  /payment/details?QrPaymentId=&DeviceToken=
                                           → { QrPaymentId, TotalAmount,
                                               AvailableReturnAmount, TransactionDate }
POST /payment/return                       → { ReturnOperationId }
POST /return/create                        → { QrToken, QrReturnId, ... }
GET  /return/status/{qrReturnId}
POST /return/operations
GET  /remote/client-info?phoneNumber=      → { ClientName }        (тек enhanced)
POST /remote/create                        → { QrPaymentId }       (тек enhanced)
POST /remote/cancel
GET  /health/ping
POST /test/payment/scan | /confirm | /scanerror | /confirmerror   (эмулятор)
```

**Сұрау денесі (`qr/create`):** `organizationBin`, `deviceToken`, `amount`, `externalId?`
**Header:** `Content-Type: application/json`, `X-Request-ID: <UUID>`
→ `X-Request-ID` — **біздің идемпотенттілік UUID-ін жіберетін орын** `[БОЛЖАМ]`
  (Kaspi оны идемпотенттілік кілті ретінде санай ма — расталмаған, §10 қара).

**Жауап конверті:** `{ StatusCode, Message, Data }`. `StatusCode: 0` = OK.

**`QrPaymentBehaviorOptions`** (Kaspi-дің өзі polling режимін айтады):
`StatusPollingInterval`, `QrCodeScanWaitTimeout`, `PaymentConfirmationTimeout`.
→ **Маңызды: Kaspi webhook бермейді, polling-ті өзі диктейді.**

**Төлем мәртебелері** (әр репозиторийде сәл өзгеше — біріктірілген тізім):
`Creating` → `QrTokenCreated` → `QrTokenScanned` → `PaymentConfirmation`/`Wait`
→ терминал: `Processed` / `Paid` / `Error` / `Expired` / `Timeout`.
`RemotePaymentCreated` — remote төлемнің аралық күйі.
⚠ Ресми enum расталмаған — `[ЖАНАМА]`.

**`PaymentMethods`:** `["Gold", "Red", "Loan"]` — яғни Kaspi Gold, Kaspi Red (несие),
Kaspi Loan (рассрочка). → Кассада «рассрочка» мүмкіндігі көрінеді.

**Қатынау:** `[ЖАНАМА]` Kaspi-мен B2B шарт, өтінім `https://kaspi.kz/webpay/partnership`,
кейбір дереккөз бойынша **IPSec VPN туннелі + статикалық IP**, енгізу **6–15 жұмыс күні**.
⚠ Бұл мәлімет үшінші тұлғаның research файлынан (§13.4) — тексеру керек.

### 2.3 C — Kaspi Shop / Merchant API (маркетплейс, бізге эквайринг емес)

`[ЖАНАМА]` Base: `https://kaspi.kz/shop/api/v2`, header `X-Auth-Token`
(сатушы кабинетінде директор генерациялайды), sandbox **жоқ** — тесттер прод-та.
Ресми gide: `https://guide.kaspi.kz/partner/ru/shop/api/general`.
Go клиенті: `github.com/abdymazhit/kaspi-merchant-api` (бейресми).

### 2.4 Kaspi Касса — бәсекелес және тәуекел

`[ЖАНАМА]` Kaspi өз фискалдық кассасын **тегін** береді
(`https://guide.kaspi.kz/partner/ru/kaspi_kassa/conditions/q2334`).
Кез келген төлем әдісіне (қолма-қол, басқа банк терминалы, **Alipay+ QR**, Kaspi Pay)
фискалдық чек шығарады, Kaspi Магазин тапсырысы автоматты чекке түседі.
→ **Бұл біздің MVP-ге тікелей бәсекелес, әрі бағасы 0 ₸.**
Дифференциацияны офлайн, техкарта, қазақ тілі және hardware-agnostic-те іздеу керек.

### 2.5 Kaspi комиссиялары — §8 қара.

---

## 3. Halyk

### 3.1 ePay 2.0 — интернет-эквайринг (ең ашық құжаттама)

`[ЖАНАМА]` Ресми порталдардың URL-дары іздеуде көрінеді, бірақ **ешқайсысы ашылмады** (403):
- `https://epayment.kz/` (ru/en/kk)
- `https://developer.homebank.kz/epay`
- Test credentials беті: `https://epayment.kz/en-US/docs/Test%20credentials`
- Mobile SDK: `https://epayment.kz/en-US/docs/mobile_sdk_documentation`

`[ЖАНАМА, кросс-тексерілген]` Хосттар (5+ тәуелсіз GitHub жобасынан):

```
OAuth (prod):    https://epay-oauth.homebank.kz/oauth2/token
OAuth (test):    https://testoauth.homebank.kz/epay2/oauth2/token
OAuth (test-2):  https://test-epay-oauth.epayment.kz/oauth2/token
API  (prod):     https://epay-api.homebank.kz
API  (test):     https://testepay.homebank.kz/api
JS lib (prod):   https://epay.homebank.kz/payform/payment-api.js
JS lib (test):   https://test-epay.epayment.kz/payform/payment-api.js
Status:          https://epay-api.homebank.kz/check-status/payment/transaction/{invoiceId}
RSA public keys: https://epay-api.homebank.kz/public.rsa
```

⚠ Бір GitHub жобасы (`itasik2/pro-cosmetics-shop-v2/docs/halyk-epay-setup.md`) тікелей
жазады: «Halyk **әртүрлі құжаттама бөлімдерінде бірнеше тест хостын** жариялаған».
→ Тест ортасын алғанда хосттарды банктен **жазбаша растату керек**.

`[ЖАНАМА, кросс-тексерілген]` Операциялар:
```
POST {oauth}/oauth2/token          grant_type=client_credentials
                                   (client_id, client_secret, terminal, scope)
POST {api}/payments/cards/auth     — криптограмма арқылы төлем (екі сатылы hold)
POST {api}/operation/{id}/charge   — растау (capture)
POST {api}/operation/{id}/cancel   — болдырмау (void/reversal)
GET  {api}/check-status/payment/transaction/{invoiceId}
```
Клиент креденшелдері: **ClientID + ClientSecret + TerminalID**.
Токен `expires_in` — бір көзде 7200 с, екіншісінде 604800 с (QR API үшін) → §11.3.

Ресми емес SDK-лар: `tungatarov/epayment` (Bitrix), `relesssar/kkb-epay2` (PHP),
`NawrasBukhari/laravel-epay`.

### 3.2 Halyk QR API — динамикалық QR (бізге ең қызықтысы)

`[ЖАНАМА]` Ресми бөлімнің URL-дары іздеуде көрінеді: `https://developer.homebank.kz/qr-api`
(ішкі беттер: `/qr-api/nachalo-raboty`, `/qr-api/avtorizaciya`, `/qr-api/spravochnik`,
`/qr-callback/qr-kod`). Сондай-ақ `https://epayment.kz/docs/QR%20by%20API`.
⚠ DNS шешілмеді — мазмұн WebSearch арқылы.

`[ЖАНАМА]` Мазмұны:
- QR **EMV QRCPS спецификациясы** бойынша генерацияланады.
- **Динамикалық QR** — әр төлемге бөлек, бір реттік.
- Қосымша payload жолын құрайды, **Reference ID** өрісіне сұрау ID-ін қояды,
  сол payload-тан QR суреті жасалады.
- Ағын: мерчант QR көрсетеді → клиент сканерлейді → қосымша **мәртебе мен сомманы
  тексереді** → төлемді қабылдағанын растайды.
- Авторизация: **OAuth 2.0**, `POST /oauth2/token`, `grant_type=client_credentials`,
  **`scope=qrapi`**. Жауап: `access_token`, `expires_in` (604800), `refresh_token`,
  `token_type: Bearer`.
- `client_id` / `client_secret` — **банктен сұрау арқылы** алынады.
- Тапсырыс құру: **`POST /merchant-api/orders`** → `order_id`.
- `/{mpan}/token` эндпоинті бар (басқа жауап құрылымымен).
- Callback/QR-код бөлімі бар → **webhook бар болуы ықтимал**, бірақ расталмады.

`[БОЛЖАМ]` Kaspi-ден айырмашылығы: Halyk QR-де callback бөлімі бар,
демек polling-ке толық тәуелді емес шығар. **Тексеру керек.**

### 3.3 Halyk Pos — SoftPOS (телефон = терминал)

`[ЖАНАМА]` Google Play пакеті (іздеу нәтижесінен, бет ашылмады): **`ru.m4bank.softpos.halyk`**
(`https://play.google.com/store/apps/details?id=ru.m4bank.softpos.halyk`),
Қырғызстан нұсқасы `ru.m4bank.softpos.halykg`.
→ Яғни Halyk SoftPOS-ты **өзі жазбаған, m4bank вендорынан алған.**

`[ЖАНАМА]` Талаптары: Android **8.1+**, NFC модулі, **интернет міндетті**.
Visa, Mastercard, UnionPay (кез келген банк), Apple Pay, Samsung Pay, Homebank Pay.
Электрондық чекті SMS/email-мен жібереді.
Ресми бет: `https://halykbank.kz/business/payment/halyk-pos`.

`[ЖАНАМА]` **Webkassa-мен интеграцияланып, чекті автоматты фискалдайды.**
→ Демек Halyk Pos-та **сыртқы кассамен байланысу нүктесі бар** — бұл біз үшін жол.

`[ЖАНАМА]` m4bank өзі айтады: SoftPOS-ты серіктестің қосымшасына
**Android SDK (plug-in-and-go)** арқылы кіріктіруге болады, немесе
**deeplink арқылы SoftPOS Android қосымшасымен app-to-app** байланыс жасауға болады
(`https://m4bank.com/sdk`). SDK Visa/Mastercard сертификатынан өткен.
→ **Бұл біздің Flutter кассамыз үшін ең нақты SoftPOS жолы.**
⚠ Deeplink схемасының нақты форматы — **Таппадым**, m4bank/Halyk-тен сұрау керек.

### 3.4 Halyk POS терминалы + касса

`[ЖАНАМА]` 1С драйвері бар (§6). Webkassa Halyk POS және Halyk Smart POS-пен
интеграцияны қолдайды (`https://promo.webkassa.kz/integrations`).
Терминалдың өз ECR-хаттамасы ашық жарияланбаған — **Таппадым**.

### 3.5 Комиссиялар — §8.

---

## 4. Jusan / Forte / Freedom

### 4.1 Jusan Bank (2025-тен бері **Alatau City Bank**)

`[ЖАНАМА]` Rebranding: Jusan → Alatau City Bank. Бұл интеграция құжаттарының
атауын да өзгертеді — келісімшартта екеуін де көрсету керек.

`[ЖАНАМА]` **Кассадан терминалға сумма жіберу — 1С драйвері арқылы бар.**
- Драйвер «1С:Библиотека подключаемого оборудования» талаптарына сай жазылған.
- Операциялар: **оплата, отмена, возврат, закрытие смены**.
- Сумма терминалға автоматты беріледі, қолмен теру қажет емес.
- Терминал моделі: **PAX A930**.
- Дереккөз: `https://1s-expert.kz/product/integratsiya-1s-s-pos-terminalami-jusan-bank/`,
  `https://grprof.com/index.php/proekty/jusanpos`.

`[ЖАНАМА]` `https://ib.jusan.kz/openapi/` — бұл **банк-клиент API**
(үзінді көшірме, төлем тапсырмасы, жалақы), эквайринг емес.
403 қайтарады, тек индекстелген.

Тарифтер — §8.

### 4.2 ForteBank

`[ЖАНАМА]` Ресми developer порталының URL-ы: `https://docs.fortebank.com/en/`
(MkDocs, ағылшынша, Postman коллекциясы:
`https://docs.fortebank.com/en/using_api/postman_collection/`).
⚠ WebFetch бұғаттады — мазмұн `[ЖАНАМА]`.

`[ЖАНАМА, кросс-тексерілген]` (research файл + тәуелсіз Go коды `thisisby/e-commerce`):
```
Base URL:  https://gateway.fortebank.com
Auth:      HTTP Basic (Shop ID + Secret Key)
Header:    X-API-Version: 3
Платформа: beGateway / eComCharge

POST /transactions/payments         — бір сатылы төлем
POST /transactions/authorizations   — екі сатылы (hold)
POST /transactions/captures
POST /transactions/voids            — болдырмау
POST /transactions/refunds          — қайтару
POST /transactions/payouts
POST /transactions/tokenizations
POST /transactions/p2p
GET  /transactions/{uid}            — мәртебе
```
Rate limit: HTTP 429 + exponential backoff. Sandbox толық, тест карталарымен.
Pay-by-link: `https://epsp.fortebank.com/products`.
Apple Pay / Google Pay / Samsung Pay қолдайды.

`[ЖАНАМА]` 1С драйвері: оплата, отмена, возврат, **итоги дня (сверка)**,
терминал **VeriFone VX520**
(`https://1s-expert.kz/product/integratsiya-1s-s-pos-terminalami-forte/`).

`[ЖАНАМА]` SoftPOS: **FortePOS** қосымшасы (NFC смартфон), банкте верификациядан өту керек.

Тарифтер — §8.

### 4.3 Freedom Pay / Freedom Bank

`[РАСТАЛҒАН — код деңгейінде]` **Freedom Pay = бұрынғы PayBox.money — бір платформа.**
Тәуелсіз кодта (`darkhan-b/arenatickets_back/app/Models/API/FreedomPayAPI.php`)
екі base URL бір-бірінің орнына жазылған:
```php
public string $url = 'https://api.freedompay.kz';
//    public string $url = 'https://api.paybox.money';
```
→ **PayBox пен Freedom Pay-ді бөлек адаптер деп жазудың қажеті жоқ.**

`[ЖАНАМА, кросс-тексерілген]` API:
```
POST {base}/init_payment.php   — төлем бастау
POST {base}/revoke.php         — болдырмау / қайтару
POST {base}/get_status.php     — мәртебе

Ортақ өрістер:  pg_merchant_id, pg_salt, pg_sig
init_payment:   pg_order_id, pg_amount, pg_description, pg_currency (KZT),
                pg_user_phone, pg_user_email,
                pg_check_url, pg_result_url, pg_success_url, pg_failure_url,
                pg_lifetime, pg_language
revoke:         pg_payment_id, pg_refund_amount
```
**Қолтаңба (`pg_sig`):** параметрлерді жалпақтау → кілт бойынша алфавит ретімен сұрыптау
→ басына скрипт атын қосу → соңына secret_key қосу → `;` арқылы біріктіру → **MD5**.
⚠ MD5 — 2026 жылға ескірген алгоритм, бірақ Freedom Pay-де осылай.

Ресми docs: `https://docs.freedompay.kz/`, `https://freedompay.kz/docs/gateway-api/pay`,
`https://freedompay.kg/doc`. Гейтвей беті: `https://freedompay.kz/banks/payment-gateway`.
`[ЖАНАМА]` Идемпотенттілік кілті құжатта аталады, бірақ өріс атын **растай алмадым**.
`[ЖАНАМА]` Фискализация модулі бар — онлайн-кассаға чек шығару.

**Freedom Bank POS терминалдары:** `https://bankffin.kz/ru/pos-terminals`,
`https://support.bankffin.kz/business/pos-terminaly-ot-freedom-bank/...`.
Кассадан терминалға сумма жіберу API-ы — **Таппадым**.

Тарифтер — §8.

---

## 5. ioka және басқа PSP-лар

### 5.1 ioka — есептегі ЕҢ ЖОҒАРЫ СЕНІМДІ дереккөз

Бұл жобадағы жалғыз провайдер, оның **толық API спецификациясын өз көзіммен оқыдым**.

`[РАСТАЛҒАН]` Ресми GitHub ұйымы `iokadev`, 4 репозиторий
(`https://github.com/orgs/iokadev/repositories`):

| Репозиторий | Не | Тіл |
|---|---|---|
| `ioka-flutter` | **ioka Flutter SDK** | Dart |
| `ioka-android` | ioka Android SDK | Kotlin |
| `ioka-ios` | ioka iOS SDK | Swift |
| `example-mobile-backend` | backend мысалы | JavaScript |

`[РАСТАЛҒАН]` pub.dev-те ресми пакет бар: **`ioka` v1.0.2**, publisher — **ioka.kz**,
MIT, ⚠ **~3 жыл жаңармаған**, 5 like / 40 pub points / 10 жүктеме.
(`https://pub.dev/packages?q=ioka`)
→ **Тәуекел: SDK іс жүзінде тасталған сияқты.** Flutter 3.x / Dart 3 үйлесімділігін
тексермей жобаға енгізуге болмайды.

`[РАСТАЛҒАН]` **Толық OpenAPI спецификациясы** репозиторийдің ішінде жатыр және оны
оқыдым: `https://raw.githubusercontent.com/iokadev/ioka-flutter/main/lib/src/api/ioka_api.json`

**Base URL:**
```
test: https://stage-api.ioka.kz
prod: https://api.ioka.kz
```

**Аутентификация (спецификациядан):** екі схема —
`API Key` (header-де, сервер жағы) және `Customer Access Token` (клиент жағы).
→ Дұрыс модель: **сервер order құрады → мобильді қосымшаға access token береді.**
Секрет кілт кассада ЕШҚАШАН сақталмайды. Бұл біздің архитектураға сай.

**Эндпоинттер (спецификациядан, толық тізім):**
```
POST   /v2/orders                                        CreateOrder
GET    /v2/orders                                        GetOrders
GET    /v2/orders/{order_id}                             GetOrderByID
POST   /v2/orders/{order_id}/capture                     CaptureOrder
POST   /v2/orders/{order_id}/cancel                      CancelOrder
POST   /v2/orders/{order_id}/refunds                     RefundOrder
GET    /v2/orders/{order_id}/refunds                     GetOrderRefunds
GET    /v2/orders/{order_id}/refunds/{refund_id}         GetRefundByID
GET    /v2/orders/{order_id}/events                      GetOrderEvents      ← аудит үшін
POST   /v2/orders/{order_id}/payments                    CreatePayment
POST   /v2/orders/{order_id}/payments/card               CreateCardPayment
GET    /v2/orders/{order_id}/payments                    GetPayments
GET    /v2/orders/{order_id}/payments/{payment_id}       GetPaymentByID      ← recovery
POST   /v2/orders/{order_id}/payments/{payment_id}/capture
POST   /v2/orders/{order_id}/payments/{payment_id}/cancel
POST   /v2/orders/{order_id}/payments/{payment_id}/refunds  CreateRefund
GET    /v2/orders/{order_id}/payments/{payment_id}/refunds  GetRefunds
POST   /v2/payment-methods                               CreatePaymentMethod
POST   /v2/payment-methods/{order_id}/apple-pay-session  StartApplePayPaymentSession
POST   /v2/webhooks   GET /v2/webhooks   GET|PATCH|DELETE /v2/webhooks/{webhook_id}
POST|GET /v2/customers      GET|DELETE /v2/customers/{customer_id}
GET    /v2/customers/{customer_id}/events
POST   /v2/customers/{customer_id}/bindings
POST|GET /v2/customers/{customer_id}/cards
GET|DELETE /v2/customers/{customer_id}/cards/{card_id}
POST|GET /v2/subscriptions  GET|POST|PATCH /v2/subscriptions/{subscription_id}
GET    /v2/subscriptions/{subscription_id}/payments
```

**Күй (status) enum-дары — спецификациядан:**
```
Order:        PENDING | PAID | CANCELLED | COMPLETED
Payment:      PENDING | AUTHORIZED | CAPTURED | CANCELLED | REJECTED
Refund:       PENDING | COMPLETED | FAILED
Subscription: ACTIVE | INACTIVE | PAUSED
Customer:     ACTIVE | INACTIVE
```
→ **Бұл — есептегі жалғыз ресми расталған статус машинасы.**
`PaymentTerminal` интерфейсіндегі `TerminalResult` осымен тексерілді:
`AUTHORIZED` vs `CAPTURED` бөлек болуы — екі сатылы (hold + capture) ағынды
интерфейс қолдауы керек дегенді растайды.

`[РАСТАЛҒАН]` **Webhook бірінші класты мүмкіндік** — CRUD эндпоинттері бар
(`/v2/webhooks`). ⚠ Kaspi QR-дан айырмашылығы: ioka-да polling міндетті емес.
`[ЖАНАМА]` Webhook оқиғасы: `PAYMENT_APPROVED`, өрістері `captured_amount`,
`refunded_amount`, `error_code`, `acquirer_reference` (docs.ioka.kz баяндауынан).

`[ЖАНАМА]` ioka-да **физикалық терминал / кассадан терминалға сумма жіберу
мүмкіндігі туралы ештеңе таппадым** — спецификацияда QR да, терминал да жоқ.
→ ioka бізге **онлайн төлем / төлем сілтемесі** үшін ғана жарайды, кассаға емес.

⚠ `example-mobile-backend` README-і бос (тек `npm install`), `index.js` 404 —
демек default branch басқа немесе файл аты өзге. Толық ағын мысалы алынбады.

Өтінім: `https://ioka.kz/#application-form`, қолдау: support@ioka.kz.

### 5.2 TipTopPay (бұрынғы CloudPayments Kazakhstan)

`[РАСТАЛҒАН — код деңгейінде]` **CloudPayments KZ → TipTopPay деп қайта аталған.**
Ресми мобильді SDK репозиторийлері (оқыдым):
- `https://github.com/tiptoppaymobile/TipTopPay-SDK-iOS`
- `https://github.com/tiptoppaymobile/TipTopPay-SDK-Android`

`[РАСТАЛҒАН — код деңгейінде]` Ресми SDK кодынан алынған base URL: **`https://api.tiptoppay.kz/`**
```
POST /payments/cards/charge      — картамен төлем (криптограмма)
POST /payments/get               — { TransactionId }
POST /payments/list              — { InvoiceId }
GET  /payments/publickey         — publicKey (Pem) + keyVersion
GET  /merchant/configuration
GET  /bins/info/{firstSix}       — BIN анықтау
```
Аутентификация: **HTTP Basic (Public ID + API Secret)**.
Мерчант кабинеті: `https://merchant.tiptoppay.kz/`.
Docs: `https://developers.tiptoppay.kz/`.
`[ЖАНАМА]` Flutter SDK `tiptoppay_sdk` pub.dev-те бар делінеді — **растай алмадым**
(GitHub-та таппадым).
`[ЖАНАМА]` Комиссия «2,5%-дан» — тек үшінші тұлғаның research файлынан,
**ресми тариф беті табылмады → сенімсіз, §8-ге енгізбедім.**
`cloudpayments.kz` әлі жұмыс істейді, ресми docs `developers.cloudpayments.ru`-ға сілтейді.

### 5.3 PayBox.money

`[РАСТАЛҒАН — код деңгейінде]` Freedom Pay-мен бір платформа (§4.3).
Ресми docs: `https://www.paybox.money/us_en/dev/payment-init`,
`https://paybox.ru/documentation/merchant-api/priemplatezhei`,
`https://paybox.ru/documentation/merchant-api/vyplaty`.
Өрістер: `pg_sig`, `pg_salt`, `pg_ps_full_amount`, `pg_to_pay`.

### 5.4 Wooppay / WoopKassa

`[ЖАНАМА]` WOOPKASSA — интернет-эквайринг шешімі, Visa/MC/Apple Pay/Google Pay,
төлем сілтемелері, выплаты. Өтінім: woopkassa.kz.
**Техникалық API құжаттамасын таппадым.**
`[ЖАНАМА]` Үшінші тұлғаның research-і «Wooppay және Processing.kz — SOAP/XML,
тек серіктестікпен, ашық құжаттама жоқ» дейді.
Ресми сайт: `https://www.wooppay.com/`, `https://wooppay.group/`.

### 5.5 Robokassa KZ

**Таппадым.** WebSearch лимиті таусылғандықтан жеке іздеу жүргізілмеді.
Келесі зерттеу итерациясында тексеру керек.

### 5.6 Бірыңғай QR (МСМП) — 2026 жылғы ЕҢ ҮЛКЕН ӨЗГЕРІС

`[ЖАНАМА]` **19.07.2026** — Қазақстанда **Межбанковская система мгновенных платежей (МСМП)**
іске қосылды. Барлық екінші деңгейлі банктер **18.07.2026-ға дейін** қосылуға міндетті болды.
- Екі сервис: **C2C** (телефон нөмірі бойынша аударым) және **C2B** (QR арқылы төлем).
- **Сатып алушы кез келген банк қосымшасымен, сатушының кез келген банк QR-ын төлей алады.**
- Kaspi де қосылды (Нацбанк мәлімдеуі бойынша 19.07.2026-дан).
- Бизнеске **жаңа жабдық қажет емес**.
- Бірінші айда 8 млн-нан астам транзакция, **210 млрд ₸**.
- Қазақстан–Қытай бірыңғай QR-ы 2026 жыл соңына жоспарланған.

Дереккөздер (барлығы 2026, WebFetch бұғатталған):
`https://tengrinews.kz/kazakhstan_news/19-iyulya-kazahstane-ofitsialno-zarabotaet-edinyiy-qr-kod-604152/`,
`https://www.zakon.kz/finansy/6521826-chto-takoe-edinyy-QRkod-i-kak-on-izmenit-zhizn-kazakhstantsev.html`,
`https://profit.kz/news/74283/...`,
`https://kz.kursiv.media/2026-08-24/fvfv-po-edinomu-qr-za-mesyac-proshlo-platezhey-na-210-mlrd-te-ge/`

`[БОЛЖАМ]` **Архитектуралық салдары:**
Егер МСМП C2B QR банк деңгейінде интероперабельді болса, кассаға **бір банкпен**
QR интеграциясы жеткілікті болуы мүмкін — Kaspi-ге бөлек, Halyk-ке бөлек QR
генерациялаудың қажеті жойылады. **Бұл интеграция көлемін бірнеше есе азайтады.**
⚠ Бірақ **мерчант тарапындағы API өзгерді ме, әлде әр банк өз API-ын сақтады ма —
БЕЛГІСІЗ.** Ұлттық төлем корпорациясынан (НПК) немесе Нацбанктен техникалық
құжаттама табылмады. **Бұл — ең жоғары приоритетті ашық сұрақ (§11.1).**

### 5.7 Пакет реестрлері мен GitHub ұйымдары бойынша тексеру нәтижесі

Ашық реестрлер — бұл ортадағы жалғыз тексерілетін дереккөз, сондықтан әр
провайдерді жеке тексердім.

| Провайдер | Ресми GitHub ұйымы | Ресми пакет реестрінде | Қорытынды |
|---|---|---|---|
| **ioka** | ✅ `iokadev` (4 repo) | ✅ pub.dev `ioka`, publisher `ioka.kz` | ⚠ бар, бірақ **ескірген** (~3 жыл) |
| **TipTopPay** | ✅ `tiptoppaymobile` (iOS + Android) | ❓ pub.dev-те **таппадым** | Мобильді SDK бар, Flutter-і жоқ |
| **PayBox** | ✅ `PayBox/paybox-authentication-sdk` (Kotlin, 2★) | ❌ | Тек auth SDK; төлем SDK-сы жоқ |
| **Freedom Pay** | ❌ **Таппадым** | ❌ | Тек қауымдастық wrapper-лері |
| **Kaspi** | ❌ **Таппадым** (тексеру аяқталмады — GitHub search 429) | ❌ | Тек бейресми wrapper-лер |
| **Halyk** | ❌ **Таппадым** | ❌ | Тек бейресми PHP/Bitrix handler-лер |
| **Jusan / Forte** | ❌ **Таппадым** | ❌ | — |
| **SUNMI** | ❌ ресми ұйым нәтижелерде көрінбеді | — | Тек қауымдастық demo/wrapper-лері |

`[БОЛЖАМ]` **Маңызды тұжырым:** біздің Flutter кассамыз үшін **бірде-бір
қазақстандық банктің ресми Dart/Flutter SDK-сы жоқ**. Барлық адаптерлерді
өзіміз жазамыз. Бұл — жоспарланатын жұмыс көлемі, тәуекел емес, бірақ
Фаза 3-те `/packages/payments` бағалауына кіруі керек.

---

## 6. 1С драйверлері мен ашық хаттамалар

### 6.1 Неге маңызды

Қазақстан банктерінің **терминал ECR-хаттамасын ашық жарияламайды**, бірақ
1С үшін драйвер жазылған. Драйвер сипаттамасы **команда сөздігін** ашады —
біздің `PaymentTerminal` интерфейсі дәл сол сөздікті қайталауы керек,
себебі банктердің терминалдары нақ соны істей алады.

### 6.2 Табылған драйверлер

| Банк | Ұсынушы | Терминал | Дереккөз |
|---|---|---|---|
| Halyk | Группа PROF | ❓ | `https://grprof.com/proekty/halykpos` |
| Kaspi | Группа PROF / Arida / 1s-expert | A80, A90 | `https://grprof.com/proekty/kaspipos`, `https://arida.kz/dev/pos/kaspi/instructions` |
| Jusan | Группа PROF / 1s-expert | **PAX A930** | `https://grprof.com/index.php/proekty/jusanpos` |
| Forte | Группа PROF / 1s-expert | **VeriFone VX520** | `https://grprof.com/proekty/fortepos` |
| Kaspi+Halyk+Jusan+Forte | Тандем Софт | — | `https://tandemsoft.kz/projects/pos/` |
| Kaspi + Halyk (1С:Розница) | infostart | — | `https://infostart.ru/1c/tools/2334511/` |
| Kaspi (iiko плагині) | pos-plugin.kz | A80, A90 | `https://pos-plugin.kz/instruction` |
| Forte (iiko плагині) | forte.pos-plugin.kz | — | `https://forte.pos-plugin.kz/instruction` |

⚠ Барлығы `.kz`/`.ru` домендер — **WebFetch бұғаттады**, мазмұн `[ЖАНАМА]`.
Евразийский банк драйвері — **Таппадым**.

### 6.3 Драйверлерден шыққан ортақ команда жиыны `[ЖАНАМА, кросс-тексерілген]`

Halyk, Jusan, Forte драйверлерінің сипаттамасы **сөзбе-сөз бірдей** тізім береді:

| 1С командасы | Мағынасы | Барлық 4 банкте бар ма |
|---|---|---|
| **Оплата** | Сумманы терминалға беру, төлем | ✅ Иә |
| **Возврат** | Қайтару (бастапқы құжат бойынша авто-іздеу) | ✅ Иә |
| **Отмена** | Болдырмау / реверс (чек нөмірін қолмен термей) | ✅ Иә |
| **Итоги дня / Закрытие смены** | Сверка итогов (settlement) | ✅ Иә |
| **ОплатаПоШтрихкоду** | — | ❌ **Таппадым** (KZ драйверлерінде аталмайды) |
| **Повтор последнего чека** | — | ❌ **Таппадым** |

`[ЖАНАМА]` Драйверлердің ортақ уәдесі: «**сумма терминалға автоматты беріледі,
терминалда қолмен теру қажет емес, төлемнен кейін терминал жауабы өңделеді**».
→ Демек **екі бағытты** хаттама (сұрау → жауап), біржақты «сумма көрсету» емес.

`[ЖАНАМА]` Қайтару кезінде драйвер «бастапқы төлем құжаты бойынша транзакцияны
автоматты табады» → терминалда **RRN / transaction reference** сақталады және
касса оны сілтей алады. → Біздің чек оқиғасында `acquirerTransactionId` міндетті.

`[БОЛЖАМ]` **Бұл драйверлердің бәрі Windows-қа арналған 1С:БПО компоненттері
(.dll / сыртқы утилита).** Flutter Android кассасынан оларды тікелей шақыру мүмкін емес.
Олар бізге **хаттаманың бар екенін дәлелдейді**, бірақ **дайын шешім бермейді**.
Бізге керегі: (а) банктің ECR-хаттамасының өзі, немесе (б) Android app-to-app.

### 6.4 ArtixPOS / Штрих-М / Frontol

`[ЖАНАМА]` Frontol Қазақстанға арналған **«ШТРИХ-М-ФР-KZ»** фискалдық регистраторын
қолдайды (`https://www.pos-shop.ru/upload/iblock/be4/.../Frontol-6.-Rukovodstvo-administratora.pdf`).
`[ЖАНАМА]` Frontol эквайринг терминалдарымен **эквайринг жүйесінің драйвері** арқылы
жұмыс істейді, әрі маңыздысы **терминал моделі емес, оның прошивкасы** —
«банкіңіздің эквайринг хаттамасын нақтылаңыз» деп жазылған
(`https://kasselect.ru/view_post.php?id=123`).
→ **Бұл бізге де қатысты: бір ғана PAX A920 әр банкте әртүрлі хаттамада болуы мүмкін.**
**ArtixPOS-тың Қазақстанға арналған эквайринг драйвері — Таппадым.**

---

## 7. Терминал SDK-лары (SUNMI, PAX, Posiflex)

### 7.1 Екі түрлі SDK-ны шатастырмау (ең маңызды тұжырым)

| Тип | Не істейді | Бізге керек пе |
|---|---|---|
| **Hardware/EMV SDK** (SunmiPay, POSLink low-level) | Картаны өзің оқисың, PIN-ді өзің аласың, EMV kernel-ді өзің жүргізесің | ❌ **ЖОҚ.** PCI PTS + банк сертификаты керек, MVP-ге мүмкін емес |
| **Semi-integrated / app-to-app** | Банк қосымшасын шақырасың, ол бәрін істейді, нәтижені қайтарады | ✅ **ИӘ.** Біздің жалғыз реалистік жол |

### 7.2 SUNMI

`[ЖАНАМА]` Ресми developer docs URL-ы: `https://docs.sunmi.com/en/documentation/smart-payment-products/p2-pro/`
(WebFetch 403 берді).

`[РАСТАЛҒАН — код деңгейінде]` SunmiPay hardware SDK-нің пакет аттарын үшінші тұлғаның
GitHub репозиторийіндегі кодтан **өз көзіммен оқыдым** (SUNMI-дің ресми растауы емес):
```
sunmi.paylib.SunmiPayKernel                      — AIDL сервиске қосылу
com.sunmi.pay.hardware.aidlv2.system.BasicOptV2
com.sunmi.pay.hardware.aidlv2.readcard.ReadCardOptV2
com.sunmi.pay.hardware.aidlv2.pinpad.PinPadOptV2
com.sunmi.pay.hardware.aidlv2.emv.EMVOptV2
com.sunmi.pay.hardware.aidlv2.security.SecurityOptV2
com.sunmi.pay.hardware.aidlv2.print.PrinterOptV2
com.sunmi.pay.hardware.aidlv2.tax.TaxOptV2
```
(SDK нұсқасы `SunmiPaySDKV2_v2.0.17_2024-10-24` — ⚠ **дереккөз 2026-дан бұрынғы**)
→ Бұл **7.1-дегі бірінші тип**. Бізге керек емес.

`[ЖАНАМА]` SUNMI semi-integrated режимдері (4 режим): **Android Service SDK (AIDL)**
— егер касса сол құрылғыда тұрса; **TCP/IP** — егер касса бөлек машина болса;
серия порты; бұлт. Ұсыныс: «касса құрылғының өзінде болса — Android Service SDK».
Дереккөз: `https://blog.rospertech.com/sunmi-semi-integrated-payment-sdk-four-modes/` (бұғатталған).

`[БОЛЖАМ]` **Қазақстанда SUNMI-ді қай банк таратады — расталмады.**
Kaspi «Smart POS A80/A90», Jusan «PAX A930», Forte «VeriFone VX520» деп аталады —
SUNMI KZ банктерінің стандартты терминалы **емес** сияқты.
SUNMI-ді біз **өз кассамыздың темірі** ретінде (принтер + сканер) қолданғанымыз дұрыс,
төлем терминалы ретінде емес.

### 7.3 PAX

`[ЖАНАМА]` **BroadPOS** — PAX-тың төлем қосымшасы (P2PE, карта деректерін шифрлайды).
Semi-integrated режимде касса PAX терминалымен **жергілікті Wi-Fi немесе бұлт арқылы
қарапайым API сұраулары** арқылы сөйлеседі.
Пакет мысалы: `com.pax.us.pay.std.broadpos.p2pe` (АҚШ нұсқасы).
`[ЖАНАМА]` **POSLink Payment SDK** — PAX барлық терминалдарға (A920, A80, E500, E600) береді.
Ресми developer guide: `https://faqs.pax.us/wp-content/uploads/2020/05/PAXSTORE-Developer-Guide-V1.08-02-04-2020-1.pdf`
⚠ **дереккөз 2020 ж. — 2026-дан әлдеқайда бұрынғы.**

`[РАСТАЛҒАН — код деңгейінде]` Flutter үшін POSLink плагині бар: `https://github.com/androiddevnotesforks/Flutter-PAX-Terminal`
— бірақ README бос (тек Flutter шаблоны), 0 жұлдыз, 2023 ж.
→ **Дайын шешім деп санауға болмайды.**

`[БОЛЖАМ]` Jusan PAX A930 таратқандықтан, POSLink жолы Jusan үшін жұмыс істеуі мүмкін.
Бірақ §6.4-тегі ескерту: **прошивка банктікі**, POSLink-ті банк қосуы керек.

### 7.4 Posiflex

**Таппадым.** Қазақстанда Posiflex төлем терминалы ретінде таралатыны туралы
дереккөз табылмады. Posiflex негізінен POS-компьютер/принтер шығарады.

### 7.5 Android app-to-app: не білеміз, не білмейміз

| Сұрақ | Жауап |
|---|---|
| Kaspi Smart POS-ты Intent арқылы шақыруға бола ма? | **Таппадым.** Kaspi құжаты **жергілікті HTTP (8080 порт)** жолын сипаттайды, Intent емес. |
| Halyk Pos (`ru.m4bank.softpos.halyk`) deeplink бар ма? | `[ЖАНАМА]` m4bank «deeplink арқылы app-to-app» дейді, **схема форматы белгісіз**. |
| Forte / Jusan Intent схемасы | **Таппадым.** |
| Нақты `Intent action` / `package` жолдары | **Ешбір банк үшін таппадым. Ойдан жазбаймын.** |

---

## 8. Комиссиялар (тек расталған сандар)

> ⚠ **БАРЛЫҚ САНДАР 2026 ЖЫЛҒЫ, БІРАҚ WebFetch БҰҒАТТАЛҒАНДЫҚТАН
> РЕСМИ ТАРИФ БЕТІН ӨЗ КӨЗІММЕН АША АЛМАДЫМ.**
> Барлығы `[ЖАНАМА]` — WebSearch баяндауы арқылы. Шарт жасамас бұрын
> банктен **жазбаша тариф парағын** алу міндетті.

### 8.1 Kaspi Pay — Smart POS `[ЖАНАМА]`

| Не | Ставка | Күшіне енген |
|---|---|---|
| **Kaspi Gold картасы**, Smart POS | **0,95%** (бұрын 1,1%) | 30.06.2026 |
| Kaspi Gold, **Apple/Google Pay** арқылы | **0,69%** (акция) → 02.10.2026-дан 0,95% | 30.06–01.10.2026 |
| Басқа банк карталары, «Остальные» санаты | **1,35%** (бұрын 1,65%) | 30.06.2026 |
| **QR** және қашықтан төлем (Kaspi POS) | **0,95%** | — |

Дереккөздер:
`https://guide.kaspi.kz/partner/ru/pos/rates/q4819`,
`https://guide.kaspi.kz/partner/ru/pos/conditions/q1901`,
`https://guide.kaspi.kz/partner/ru/pos/rates`,
`https://digitalbusiness.kz/2026-06-24/kaspi-pay-menyaet-komissiyu-za-priem-oplati-cherez-smart-pos-s-30-iyunya/`,
`https://bizmedia.kz/2026-06-24-komissiya-za-priem-oplaty-kartami-cherez-smart-pos-ot-kaspi-pay-snizhaetsya/`

### 8.2 Halyk Bank `[ЖАНАМА]`

| Не | Ставка | Күшіне енген |
|---|---|---|
| Эквайринг, басқа банк карталары | **1%** | 04.05.2026 |
| **Halyk QR** | **0,5%** | 04.05.2026 |

Дереккөздер:
`https://digitalbusiness.kz/2026-04-30/halyk-bank-snizit-tarifi-do-1-chto-eto-znachit-dlya-biznesa/`,
`https://www.zakon.kz/finansy/6516412-Halyk-Bank-snizit-tarify-za-ekvayring-do-1-chto-eto-znachit-dlya-biznesa.html`,
`https://inbusiness.kz/ru/last/halyk-bank-snizit-tarify-do-1-chto-eto-znachit-dlya-biznesa`
Ресми тариф парағы: halykbank.kz «Тарифы для юридических лиц» бөлімі.

### 8.3 Jusan / Alatau City Bank `[ЖАНАМА]`

| Не | Ставка |
|---|---|
| QR | **0,8%** |
| Jusan картасы | **1,5%** |
| Басқа банк картасы | **2,2%** |
| ЖК, жылдық айналымы 45 млн ₸-ге дейін, кез келген банк картасы | **0,9%** |

Дереккөз: `https://jusan.kz/business/tole/new`, `https://jusan.kz/business/tole`
Қызмет көрсету: айына 300 000 ₸-ден жоғары айналымда тегін, жеткізу тегін.

### 8.4 ForteBank `[ЖАНАМА]` — ⚠ дереккөздер қайшы

| Не | Ставка | Дереккөз |
|---|---|---|
| Forte карталары | **0,4%-тан** | `https://finkaz.kz/fortebank/acquiring` ⚠ **2025 ж.** |
| Forte QR (өз жүйесі) | **0,5%** | `https://finratings.kz/news/14552-...` |
| Басқа ҚР банктерінің карталары | **1,3–1,5%** (айналым мен қызмет түріне қарай) | сонда |
| POS абоненттік төлемі | ЖШС ≈ **990 ₸/ай**, ЖК ≈ **2 000 ₸/ай** | сонда |
| ЖК, жылдық түсімі 45 млн ₸-ге дейін | **0%** | сонда |

Ресми беттер: `https://business.forte.kz/ru/acquiring`, `https://business.forte.kz/ru/internet-acquiring`
⚠ Барлығы үшінші тұлға сайттарынан. **Ресми тариф расталмаған.**

### 8.5 Freedom Bank `[ЖАНАМА]`

| Не | Ставка |
|---|---|
| POS, Freedom Bank картасы | **0,8%** |
| POS, басқа банк картасы | **1,8%** |
| Онлайн-эквайринг, Freedom картасы | **1,2%** |
| Онлайн-эквайринг, басқа ЕДБ картасы | **3,5%** |

Ресми тариф PDF: `https://www.bankffin.kz/storage/docs/t7bF47s5O3J3isLWKZXFaEFUwX5J6hSyMb6HuXW3.pdf`
(«12. Тарифы по торговому эквайрингу») — ⚠ **WebFetch бұғаттады, ашылмады.**
Басқа дереккөз: `https://finkaz.kz/freedom-bank/acquiring` ⚠ **2025 ж.**

### 8.6 Ақшаның есепшотқа түсуі (settlement)

| Банк | Мерзім | Дереккөз |
|---|---|---|
| **Freedom Bank, QR** | 00:00–18:00 транзакциялары → **сол күні 18:30**; 18:00–00:00 → **00:30** | `https://support.bankffin.kz/business/pos-terminaly-ot-freedom-bank/...` `[ЖАНАМА]` |
| **ForteBank** | **күн сайын, демалыс және мереке күндерін қоса** | `https://business.forte.kz/ru/acquiring` `[ЖАНАМА]` |
| Kaspi | **Таппадым** | — |
| Halyk | **Таппадым** | — |
| Jusan | **Таппадым** | — |

### 8.7 ⚠ CLAUDE.md-дегі болжамдар ЕСКІРГЕН

`CLAUDE.md` §1-де: «QR — 0,95%, өз картасы Smart POS арқылы — 1,29%,
басқа банк картасы — 2,3%».

2026 ж. қыркүйектегі жағдай (§8.1–8.5) бұған сәйкес келмейді:
- Kaspi Gold Smart POS арқылы — **1,29% емес, 0,95%** (30.06.2026-дан).
- Басқа банк картасы Kaspi-де — **2,3% емес, 1,35%**.
- Halyk QR — **0,5%**, бұл Kaspi QR-дан (0,95%) **арзан**.
- Forte QR — **0,5%**, Jusan QR — **0,8%**.

→ **«Ең арзан әдіс = Kaspi QR» деген бастапқы болжам енді автоматты дұрыс емес.**
Ставкалар **конфигурациядан** келуі керек, кодқа жазылмауы керек (§9.5).

---

## 9. `PaymentTerminal` интерфейсінің ұсынысы

### 9.1 Негізгі принциптер

1. **«Нәтиже белгісіз» — қалыпты күй, ерекше жағдай емес.** Тайм-аут, желі үзілісі,
   кассаның қайта іске қосылуы — бәрі осыған әкеледі. Интерфейс оны **бірінші класты
   күй** ретінде қайтаруы керек, exception ретінде емес.
2. **Әр провайдер әртүрлі мүмкіндікте.** `capabilities` арқылы декларация,
   бизнес-логика соны сұрайды.
3. **Ақша — `Money` (integer тиын/тенге)**, `double` жоқ (CLAUDE.md §5).
4. **`clientOperationId` (UUID) — біздікі**, әр провайдерге өз өрісінде беріледі
   (Kaspi: `X-Request-ID` / `externalId`; Freedom: `pg_order_id`; ioka: `external_id`).
5. **Домен қабатында I/O жоқ** — бұл интерфейс `/packages/payments`-те,
   `/packages/domain`-де оның тек **нәтиже типтері** тұрады.

### 9.2 Интерфейс (Dart тәрізді псевдокод)

```dart
// packages/payments — adapter interface. Domain depends only on the result types.

/// What a concrete provider is able to do. Business logic MUST query this
/// instead of branching on provider names.
class TerminalCapabilities {
  final bool card;              // card present payment
  final bool qr;                // dynamic QR
  final bool refund;
  final bool partialRefund;
  final bool voidSameDay;       // reversal before settlement
  final bool settlement;        // day totals / сверка итогов
  final bool reprintLastReceipt;
  final bool webhook;           // false => polling only (Kaspi QR)
  final bool offlineCard;       // almost certainly false everywhere - see 9.4
  final Duration statusPollInterval;
  final Duration operationTimeout;
}

/// Provider-agnostic outcome. NOTE: `unknown` is NOT an error.
sealed class TerminalResult {}

class Approved extends TerminalResult {
  final String clientOperationId;      // our UUID, echoed back
  final String providerOperationId;    // processId / QrPaymentId / transactionId
  final String? acquirerReference;     // RRN - needed to find the txn on refund
  final Money amount;
  final PaymentMethod method;          // card | qr | installment ... MUST be stored:
                                       // Kaspi requires refund via the SAME method
  final String? cardMask;
  final String? authCode;
  final DateTime providerTimestamp;    // provider clock, not device clock
  final Map<String, String> rawFields; // for the fiscal receipt
}

class Declined extends TerminalResult {
  final String clientOperationId;
  final String providerCode;
  final String message;                // already localized? no - i18n key + params
}

class Cancelled extends TerminalResult { final String clientOperationId; }

/// The dangerous one. We do NOT know whether money moved.
/// The cashier MUST NOT be allowed to close the receipt on this result.
class Unknown extends TerminalResult {
  final String clientOperationId;
  final String? providerOperationId;   // may be null if we never got it
  final UnknownReason reason;          // timeout | networkLost | appCrash | powerLoss
  final DateTime firstAttemptAt;
}

abstract interface class PaymentTerminal {
  String get providerId;                    // 'kaspi_smartpos', 'halyk_qr', ...
  TerminalCapabilities get capabilities;

  /// Pair the cash register with the terminal. Kaspi: register -> accessToken (24h).
  Future<void> bind(TerminalBinding binding);
  Future<bool> ping();

  /// Start a payment. MUST be safe to call twice with the same clientOperationId:
  /// the second call must return the state of the first, never a second charge.
  Future<TerminalResult> pay({
    required String clientOperationId,      // idempotency key, ours
    required Money amount,
    PaymentMethodHint? hint,                // card | qr | ask-on-terminal
    String? fiscalReceiptNumber,
  });

  /// THE most important method for offline-first. Called:
  ///  - after every Unknown
  ///  - on app start, for every operation left in a non-final state
  /// Kaspi Smart POS keeps operations 24h - that is our recovery window.
  Future<TerminalResult> queryStatus({
    required String clientOperationId,
    String? providerOperationId,
  });

  /// Refund. MUST carry the original method (Kaspi: QR->QR, card->card only).
  Future<TerminalResult> refund({
    required String clientOperationId,
    required String originalProviderOperationId,
    required PaymentMethod originalMethod,
    required Money amount,                  // may be partial if capabilities allow
  });

  /// Reversal of a not-yet-settled transaction (1C: "Отмена").
  Future<TerminalResult> voidTransaction({
    required String clientOperationId,
    required String originalProviderOperationId,
  });

  /// Day totals / сверка итогов (1C: "Итоги дня"). Bound to our shift close.
  Future<SettlementReport> settle({required String clientOperationId});

  Future<void> reprintLastReceipt();        // throws UnsupportedError if !capability
}
```

### 9.3 Идемпотенттілік және «белгісіз» күйді өңдеу

```
pay(uuid) ──► Approved ──► чекті фискалдау, ауысымға жазу
         ├──► Declined ──► басқа әдіс ұсыну
         ├──► Cancelled ─► кассирге қайту
         └──► Unknown ───► ЧЕКТІ ЖАППАУ
                            │
                            ├─ жергілікті БД-ға `pending_terminal_op` оқиғасы
                            ├─ queryStatus(uuid) — backoff-пен қайталау
                            │    (2s, 5s, 10s, 30s, содан кейін 5 мин сайын, 24 сағат бойы)
                            └─ шешілмесе → кассирге «Терминалдан чекті тексеріңіз»
                               + ауысым жабылмайды (blocking reconciliation)
```

**Қатаң ережелер:**
- `Unknown` күйінде **қайта `pay()` шақыруға ТЫЙЫМ.** Тек `queryStatus()`.
- Әр `pay()` шақыруы алдында оқиға журналына `TerminalOperationStarted(uuid)`
  жазылады — қосымша қирап қалса да, қайта қосылғанда `queryStatus` жүреді.
- `clientOperationId` — **чектің UUID-і емес**, бөлек. Бір чекте бірнеше
  төлем әрекеті болуы мүмкін (сәтсіз → қайта).
- Ауысым жабу (`settle`) шешілмеген `pending_terminal_op` бар болса **бұғатталады**.

### 9.4 Офлайн: не мүмкін, не мүмкін емес

| Сценарий | Мүмкін бе | Негіз |
|---|---|---|
| Интернет жоқ, **касса ↔ терминал LAN арқылы жұмыс істейді** | ✅ **Иә** | Kaspi Smart POS API — жергілікті HTTP, 8080. Терминалдың өз GSM/Wi-Fi байланысы бөлек. |
| Терминалда да интернет жоқ, **карта төлемі** | ❌ Практикада жоқ | `[БОЛЖАМ]` EMV офлайн-авторизация теориялық бар, бірақ KZ банктері оны мерчантқа ашпайды. **Тексерілмеген — іздеу лимиті таусылды.** |
| Терминалда интернет жоқ, **QR төлемі** | ❌ **Жоқ** | `[БОЛЖАМ]` QR төлемі сатып алушының банк қосымшасы арқылы жүреді — екі жақта да интернет керек. |
| Интернет жоқ, **қолма-қол** | ✅ Иә | Фискалдық чек 72 сағат офлайн кезекте (CLAUDE.md §3). |

`[БОЛЖАМ]` **Салдары:** біздің «72 сағат автономды» уәдеміз **қолма-қол және
фискализацияға** қатысты, **карта/QR төлеміне емес**. Бұл UI-да ашық көрсетілуі керек:
интернет жоғалғанда касса «тек қолма-қол» режиміне ауысады, ал терминалдың
өз байланысы болса — карта да жұмыс істейді.
→ Бұл маркетингте абайлап тұжырымдалуы керек, әйтпесе клиентті алдағанға саяды.

### 9.5 Ең арзан төлем әдісін ұсыну логикасы

```dart
/// Rates come from CONFIG (per merchant, per bank contract), never hardcoded:
/// they changed 3 times in 2026 alone (see section 8.7).
class TariffTable {
  final Map<(String providerId, PaymentMethod, CardBrandClass), Basis> rates;
  // Basis = { int basisPoints; Money? fixedFee; DateTime validFrom; DateTime? validTo; }
}

/// Returns the ordered list to show the cashier. Cost is only ONE criterion.
List<PaymentOption> suggestOptions({
  required Money amount,
  required TariffTable tariffs,
  required Set<String> availableProviders,   // what is actually bound & online
  required bool internetAvailable,
}) {
  // 1. filter by physical availability (terminal bound? online? capability?)
  // 2. compute cost = amount * bps / 10000 (+ fixedFee), INTEGER arithmetic only
  // 3. sort by cost ascending
  // 4. BUT: never reorder against the customer's choice - the customer decides.
  //    We only pre-select the default and show the cheapest first.
}
```

**Ескертулер:**
- Комиссияны **сатып алушы емес, мерчант** төлейді → бұл кассирге көрінбейтін,
  тек **әдепкі әдісті таңдау** үшін қолданылатын оптимизация.
- Нақты ставка **картаның BIN-іне** тәуелді (өз банкі ме, басқа ма).
  BIN анықтау API-ы **тек TipTopPay-де расталған** (`GET /bins/info/{firstSix}`).
  Kaspi/Halyk-те BIN-ді касса төлемге дейін біле алмайды → **әдепкіні QR-ға қою**
  әдетте ең қауіпсіз эвристика.
- ⚠ **МСМП бірыңғай QR** (§5.6) бұл логиканы толық өзгертуі мүмкін.

### 9.6 Транспорт абстракциясы

```dart
sealed class TerminalTransport {}
class LocalHttpTransport extends TerminalTransport {   // Kaspi Smart POS: LAN, :8080
  final String host; final int port; final bool tls; final String? accessToken;
}
class AndroidAppToAppTransport extends TerminalTransport {  // SoftPOS / bank app
  final String package; final String? intentAction; final String? deeplinkScheme;
}
class CloudApiTransport extends TerminalTransport {     // Kaspi QR, Halyk QR, ioka
  final Uri baseUrl; final AuthScheme auth;             // apiKey | oauth2 | mTLS
}
class SerialTransport extends TerminalTransport {}       // Windows COM/USB (кейін)
```
→ Транспорт **интерфейсте көрінбейді**, тек адаптердің ішінде.
Бизнес-логика `PaymentTerminal`-ды ғана біледі (CLAUDE.md §5.4).

---

## 10. Әр провайдер бойынша «не белгісіз»

### Kaspi Smart POS (жергілікті API) — интерфейсті бекітпес бұрын білу керек
1. **Нақты эндпоинт жолдары және JSON өрістері.** (PDF ашылмады.)
2. Аутентификация шынымен бар ма? Бір дереккөз «SSL + token», екіншісі «жоқ» дейді.
3. HTTP па, HTTPS па? Өзін-өзі қолтаңбалаған сертификат па?
4. Идемпотенттілік: сол `processId`-ті қайта жіберсе, екінші рет ақша шешіле ме?
5. `settle` / «итоги дня» жергілікті API-де бар ма, әлде тек терминал мәзірінде ме?
6. Соңғы чекті қайта басып шығару API-ы бар ма?
7. Терминал экранындағы тілді (kk/ru) басқаруға бола ма?
8. Бір терминалға **бірнеше касса** қосыла ала ма (1–3 жұмыс орны сценарийі)?
9. Kaspi Касса қосулы болса, үшінші тарап интеграциясы шынымен бұғатталады ма?

### Kaspi QR API (server-to-server)
1. **Production base URL.**
2. `X-Request-ID` — шын идемпотенттілік кілті ме, әлде тек трассировка ма?
3. `Status` enum-ының **ресми толық тізімі**.
4. Webhook мүмкіндігі мүлдем жоқ па (әзірше тек polling көрдім)?
5. `externalId` бойынша қайта іздеу (recovery) бар ма?
6. IPSec VPN + статикалық IP шын талап па?
7. Kaspi Smart POS-тың QR-ы мен бұл QR API — бір ақша ағыны ма?

### Halyk
1. QR API-де **callback/webhook** бар ма, әлде polling ғана ма?
2. `POST /merchant-api/orders`-тан кейінгі толық ағын (мәртебе, қайтару эндпоинті).
3. Тест хосттарының **қайсысы дұрыс** (үш нұсқа кездесті).
4. Halyk POS терминалының ECR-хаттамасы 1С-тен тыс беріле ме?
5. Halyk Pos SoftPOS-тың **deeplink/Intent схемасы**.
6. m4bank SDK-ны үшінші тарап кассасына кіріктіруге Halyk рұқсат бере ме?
7. Token `expires_in`: 7200 ма, 604800 ба (қайшылық)?

### Jusan / Alatau City Bank
1. Rebranding-тен кейін API/драйвер қолдауы сақталды ма?
2. Эквайринг API мүлдем бар ма (ib.jusan.kz — банк-клиент қана)?
3. PAX A930-да POSLink қосулы ма?
4. QR API бар ма?

### ForteBank
1. `gateway.fortebank.com` — **интернет-эквайринг қана ма**, әлде физикалық терминалға да қатысты ма?
2. Sandbox base URL (research файлы «расталуы керек» деп жазады).
3. FortePOS SoftPOS-та SDK/deeplink бар ма?
4. VeriFone VX520 — ескі темір; жаңа Android терминалдарға көшу жоспары бар ма?

### Freedom Pay / Freedom Bank
1. Идемпотенттілік өрісінің **нақты аты**.
2. MD5 `pg_sig`-тен күштірек нұсқа бар ма?
3. Freedom Bank POS терминалына кассадан сумма жіберу API-ы бар ма?
4. Фискализация модулі ОФД-ға тікелей жүре ме, әлде Webkassa арқылы ма?

### ioka (OpenAPI оқылғандықтан ең аз белгісіздік)
1. **SDK әлі қолдауда ма?** pub.dev-те v1.0.2, ~3 жыл жаңармаған. Flutter 3.x /
   Dart 3-пен жұмыс істей ме — **тексеру керек** (жауап беруге болатын нәрсе,
   жобаға енгізбес бұрын `pub add ioka` сынағы жеткілікті).
2. Физикалық терминал / QR бар ма? — Спецификацияда **жоқ**, бірақ ол SDK-нің
   спецификациясы; серверлік API-де бөлек QR өнімі болуы мүмкін.
3. `Idempotency-Key` header-і бар ма? Спецификация қысқашасында көрінбеді.
4. `GetOrderEvents` — event sourcing үшін толық аудит бере ме?
5. Apple Pay бар (`apple-pay-session`), **Google Pay** спецификацияда көрінбеді.

### TipTopPay
1. Ресми тариф (2,5% — расталмаған).
2. Flutter SDK шын бар ма?
3. QR / физикалық терминал бар ма?

### МСМП / Бірыңғай QR
1. **Мерчант тарапында API өзгерді ме?**
2. C2B комиссиясы (interchange) қанша?
3. Касса бір банкпен интеграцияланып, барлық банк клиенттерін қабылдай ала ма?
4. Refund бірыңғай QR арқылы қалай жүреді?

---

## 11. Қайшылықтар мен ашық сұрақтар

### 11.1 🔴 ЕҢ ЖОҒАРЫ ПРИОРИТЕТ — МСМП бірыңғай QR
19.07.2026-дан бастап барлық банк бір QR-ды қабылдайды. **Бұл жобаның интеграция
стратегиясын түбегейлі өзгертуі мүмкін.** Техникалық құжаттама табылмады.
→ **Фаза 2-ні бастамас бұрын Нацбанк/НПК-дан анықтау керек.**
Егер бір интеграция жеткілікті болса — «Kaspi QR-ды бірінші кезекке қою»
(CLAUDE.md §1.4) деген дифференциация **мағынасын жоғалтады немесе өзгереді**.

### 11.2 Kaspi Smart POS API аутентификациясы — қайшы
- Дереккөз A: «API қорғанысы SSL (HTTPS) арқылы, касса тіркеледі, токенмен
  аутентификацияланады» (`guide.kaspi.kz` баяндауы).
- Дереккөз B: «API интеграциясында **аутентификация жоқ**, желі деңгейінде
  бөлек желіде қорғау керек» (сол PDF-тің басқа баяндауы).
→ Екеуі де бір PDF-ке сілтейді. **PDF-ті оқымай шешілмейді.**
Қауіпсіздік талабы ретінде **екеуін де болжап** жобалау керек:
токенді де қолдану + желіні де оқшаулау.

### 11.3 Halyk токенінің мерзімі
`expires_in`: ePay үшін **7200 с**, QR API үшін **604800 с** делінеді.
Мүмкін екеуі шын (бөлек сервистер), мүмкін бір дереккөз қате.

### 11.4 «Kaspi Pay-де ашық API жоқ» — қайшы
Үшінші тұлғаның research файлы (`theYahia/WWmcp`) «Kaspi Pay-де ресми ашық API
**жоқ**, тек үшінші тарап wrapper-лері бар» дейді.
Бірақ: (а) Kaspi өзі Smart POS интеграция PDF-ін ашық жариялаған;
(б) `mtokentest.kaspi.kz` тест ортасы бар және оны бірнеше жоба қолданады.
→ Дұрысы: **«ашық емес, бірақ серіктестерге бар»**. Екеуін де жазып қойдым.

### 11.5 Kaspi тест base URL — екі порт
`mtokentest.kaspi.kz:8543` (r1, Api-Key) vs `mtokentest.kaspi.kz:8545` (r3, mTLS).
Схемаға байланысты болуы мүмкін, бірақ расталмаған.

### 11.6 CLAUDE.md-дегі комиссия сандары ескірген
§8.7 қара. **CLAUDE.md §1-ді жаңарту керек.**

### 11.7 Басқа зерттеушілермен ықтимал қайшылықтар
- **Фискализация зерттеуімен:** Kaspi Касса тегін фискалдық чек береді, Halyk Pos
  Webkassa арқылы автофискалдайды. Егер фискализация зерттеуі «ОФД-мен тікелей
  интеграция» деп жазса — нарықта **банк + касса біріктірілген** шешімдер басым
  екенін ескеру керек.
- **Темір зерттеуімен:** мен «SUNMI-ді төлем терминалы ретінде емес, принтер/сканер
  ретінде алу керек» деп болжадым. Егер темір зерттеуі SUNMI P2 Pro-ны төлем
  терминалы ретінде ұсынса — қайшылық. Шешуші фактор: **қай банк қай темірді
  таратады** (жауап §7.2-де толық емес).
- **Бәсекелестер зерттеуімен:** Paloma365 «Kaspi QR + терминал + Магазин»
  интеграциясын жарнамалайды (`https://paloma365.kz/kaspi-kassa`), Poster-де
  Kaspi терминалымен интеграция бар (`https://joinposter.com/en/applications/kaspi`).
  → Kaspi интеграциясы **дифференциация емес, кіру билеті**.

---

## 12. `[БҰҒАТТАЛҒАН]` тізімі — кімге жазу, не үшін, қандай құжатпен

| # | Кімге | Не үшін | Не керек | Приоритет |
|---|---|---|---|---|
| 1 | **Kaspi Bank, серіктестік бөлімі** (`https://kaspi.kz/webpay/partnership`) | Smart POS интеграция PDF-і + QR API құжаттамасы + тест ортасы | ЖШС/ЖК тіркелуі, өтінім, B2B шарт. Мүмкін: статикалық IP + IPSec VPN | 🔴 P0 |
| 2 | **Kaspi Pay қолдауы** | Smart POS-та «Защита интеграции» қосылған терминал + `accessToken` алу | Жұмыс істеп тұрған Smart POS терминалы (мерчант аккаунт) | 🔴 P0 |
| 3 | **Halyk Bank, ePay/QR API** (`developer.homebank.kz`) | QR API үшін `client_id` + `client_secret`, `scope=qrapi` | Заңды тұлға, банкке жазбаша сұрау | 🔴 P0 |
| 4 | **Нацбанк ҚР / Ұлттық төлем корпорациясы (НПК)** | **МСМП C2B QR** техникалық құжаттамасы және мерчант интеграция тәртібі | Ресми сұрау; мүмкін банк арқылы ғана | 🔴 P0 (§11.1) |
| 5 | **Halyk Bank / m4bank** (`https://m4bank.com/sdk`) | Halyk Pos SoftPOS SDK немесе deeplink схемасы | Банкпен шарт, мүмкін NDA + m4bank лицензиясы | 🟠 P1 |
| 6 | **ForteBank** (`docs.fortebank.com`) | Sandbox Shop ID + Secret Key; терминал ECR-хаттамасы | Тіркелу; sandbox өзін-өзі қызмет көрсетуі мүмкін | 🟠 P1 |
| 7 | **Freedom Pay** (`docs.freedompay.kz`) | Merchant ID + secret key, тест ортасы | ЖШС/ЖК, өтінім | 🟠 P1 |
| 8 | **ioka** (`https://ioka.kz/#application-form`, support@ioka.kz) | API key; Flutter SDK-нің қолдау мәртебесі | Өтінім формасы | 🟡 P2 |
| 9 | **TipTopPay** (`merchant.tiptoppay.kz`) | Public ID + API Secret; тариф | Тіркелу | 🟡 P2 |
| 10 | **Jusan / Alatau City Bank** | Эквайринг API бар ма; PAX A930 хаттамасы | Банк клиенті болу | 🟡 P2 |
| 11 | **Тандем Софт / Группа PROF / Arida** | Бар драйверлердің ECR-хаттамасы (коммерциялық лицензия) | Коммерциялық шарт, NDA | 🟡 P2 (жедел жол) |
| 12 | **Webkassa** (`promo.webkassa.kz/integrations`) | API кілті + Postman коллекциясы (26+ метод) | Тіркелу, ақылы жазылым | 🟠 P1 (фискализация зерттеуімен қиылысады) |
| 13 | **PAX / SUNMI дистрибьюторы (ҚР)** | Semi-integrated режимнің қай банкте қосулы екені | Дилерлік байланыс | 🟡 P2 |

---

## 13. Дереккөздер тізімі

> Барлығы **2026-09-01** күні қаралды.
> **(F)** = WebFetch-пен тікелей оқылды (сенімді).
> **(S)** = тек WebSearch баяндауы арқылы (домен бұғатталған).
> ⚠ = дереккөз 2026 жылдан бұрынғы.

### 13.1 Kaspi
- (S) `https://guide.kaspi.kz/cdn/content/pay/product/documents/Kaspi%20POS/Smart-POS-Dokymentatsia-po-integratsii.pdf` — Smart POS интеграция құжаты (ru)
- (S) `https://guide.kaspi.kz/cdn/content/pay/product/documents/Kaspi%20POS/Smart-POS-Integratsiyalay-zhonindegi-kyzhattama.pdf` — сол, қазақша
- (S) `https://guide.kaspi.kz/partner/ru/pos/conditions/q2722` — Smart POS немен интеграцияланады
- (S) `https://guide.kaspi.kz/partner/ru/pos/conditions/q1903` — 1С:Бухгалтерия
- (S) `https://guide.kaspi.kz/partner/ru/pos/conditions/q1901` — Smart POS құны
- (S) `https://guide.kaspi.kz/partner/ru/pos/rates` , `.../rates/q4819` — комиссия төмендеуі, 30.06.2026
- (S) `https://guide.kaspi.kz/partner/ru/shop/api/general/q3193` , `q3194` — Магазин API
- (S) `https://guide.kaspi.kz/partner/ru/kaspi_kassa/conditions/q2334` , `/cashier/conditions/q2472` — Kaspi Касса
- (S) `https://business.kaspi.kz/pay/` — Kaspi Pay
- (S) `https://kaspipay.kz/documents/p5r.pdf` — техрегламент (ашылмады)
- (F) `https://github.com/thedakeen/kaspi-api-wrapper` + `mockserver-init*.json` — QR API эндпоинттері
- (F) `https://github.com/HolySxn/KaspiQR-Wrapper` — `mtokentest.kaspi.kz:8543/r1/v01`
- (F) `https://github.com/altynbek132/supabase-project-kaspi` — `:8545`, `X-Request-ID`, өрістер
- (F) `https://github.com/burcev-alex/kaspi-qr-sdk` — PHP SDK, EASY/STRONG схемалары
- (F) `https://github.com/RomanBronevik/kaspi-qr` — мәртебелер
- (F) `https://github.com/tapter-dev/kaspi-pos-automation` — мәртебелер, «Единый QR»
- (F) `https://github.com/abdymazhit/kaspi-merchant-api` — Merchant API (маркетплейс)
- (S) `https://digitalbusiness.kz/2026-06-24/kaspi-pay-menyaet-komissiyu-za-priem-oplati-cherez-smart-pos-s-30-iyunya/`
- (S) `https://bizmedia.kz/2026-06-24-komissiya-za-priem-oplaty-kartami-cherez-smart-pos-ot-kaspi-pay-snizhaetsya/`
- (S) `https://finratings.kz/news/15112-kaspi-pay-obnovil-tarify-smart-pos-dlia-predprinimatelei/`

### 13.2 Halyk
- (S) `https://epayment.kz/` , `https://epayment.kz/docs/QR%20by%20API` , `/docs/Test%20credentials` , `/docs/mobile_sdk_documentation`
- (S) `https://developer.homebank.kz/` , `/qr-api` , `/qr-api/avtorizaciya` , `/qr-api/nachalo-raboty` , `/qr-api/spravochnik` , `/qr-callback/qr-kod` , `/epay`
- (S) `https://halykbank.kz/business/payment/epay` , `/business/payment/pos-terminaly` , `/business/payment/halyk-pos`
- (S) `https://halykbank.kz/knowledge_base/entity/53` , `/54` , `/56` — POS, Halyk Pos, Halyk QR ЖС
- (F) `https://github.com/tungatarov/epayment` — ePay 2.0 Bitrix handler ⚠
- (F) `https://github.com/relesssar/kkb-epay2` , `https://github.com/NawrasBukhari/laravel-epay`
- (F) `https://github.com/itasik2/pro-cosmetics-shop-v2` (`docs/halyk-epay-setup.md`) — хосттар
- (F) `https://github.com/ShanderYO/parkpass_backend` — `/operation/{id}/charge`, `/cancel`
- (S) `https://play.google.com/store/apps/details?id=ru.m4bank.softpos.halyk` — SoftPOS пакеті
- (S) `https://m4bank.com/sdk` , `https://m4bank.com/softpos` — SDK / deeplink
- (S) `https://digitalbusiness.kz/2026-04-30/halyk-bank-snizit-tarifi-do-1-chto-eto-znachit-dlya-biznesa/`
- (S) `https://www.zakon.kz/finansy/6516412-Halyk-Bank-snizit-tarify-za-ekvayring-do-1-chto-eto-znachit-dlya-biznesa.html`
- (S) `https://inbusiness.kz/ru/last/halyk-bank-snizit-tarify-do-1-chto-eto-znachit-dlya-biznesa`

### 13.3 Jusan / Forte / Freedom / PSP
- (S) `https://jusan.kz/business/tole` , `/business/tole/new` — тарифтер
- (S) `https://1s-expert.kz/product/integratsiya-1s-s-pos-terminalami-jusan-bank/` — PAX A930
- (S) `https://grprof.com/index.php/proekty/jusanpos` , `/proekty/fortepos` , `/proekty/kaspipos` , `/proekty/halykpos`
- (S) `https://docs.fortebank.com/en/` , `/en/using_api/postman_collection/`
- (F) `https://github.com/thisisby/e-commerce` — `gateway.fortebank.com/transactions/{id}`
- (S) `https://business.forte.kz/ru/acquiring` , `/ru/internet-acquiring`
- (S) `https://finratings.kz/news/14552-kakoi-bank-predlagaet-samyi-vygodnyi-ekvairing-v-kazakhstane/`
- (S) `https://finkaz.kz/fortebank/acquiring` ⚠ 2025 , `https://finkaz.kz/freedom-bank/acquiring` ⚠ 2025
- (S) `https://docs.freedompay.kz/` , `https://freedompay.kz/docs/gateway-api/pay` , `https://freedompay.kg/doc`
- (F) `https://github.com/darkhan-b/arenatickets_back` — `api.freedompay.kz` == `api.paybox.money`, pg_* өрістер, MD5
- (S) `https://bankffin.kz/ru/pos-terminals` , `https://support.bankffin.kz/business/pos-terminaly-ot-freedom-bank/...`
- (S) `https://www.bankffin.kz/storage/docs/t7bF47s5O3J3isLWKZXFaEFUwX5J6hSyMb6HuXW3.pdf` — ресми тариф PDF (ашылмады)
- (S) `https://www.paybox.money/us_en/dev/payment-init` , `https://paybox.ru/documentation/merchant-api/...`
- (F) `https://github.com/iokadev/ioka-flutter` — **ресми** Flutter SDK ⚠ ескі
- (S) `https://docs.ioka.kz/en/ioka-api/webhooks/` , `https://ioka.kz/docs/en/ioka-api/google-pay` , `https://ioka.kz/`
- (F) `https://github.com/tiptoppaymobile/TipTopPay-SDK-iOS` , `-Android` — **ресми** SDK, `api.tiptoppay.kz`
- (S) `https://developers.tiptoppay.kz/` , `https://merchant.tiptoppay.kz/` , `https://cloudpayments.kz/`
- (S) `https://www.wooppay.com/` , `https://wooppay.group/`

**Код деңгейінде оқылған (F), ioka:**
- (F) `https://github.com/orgs/iokadev/repositories` — ресми ұйым, 4 SDK
- (F) `https://raw.githubusercontent.com/iokadev/ioka-flutter/main/lib/src/api/ioka_api.json` — **толық OpenAPI**
- (F) `https://raw.githubusercontent.com/iokadev/ioka-flutter/main/README.md`
- (F) `https://pub.dev/packages?q=ioka` — publisher `ioka.kz`, v1.0.2 ⚠ ~3 жыл
- (F) `https://github.com/iokadev/example-mobile-backend` — README бос, `index.js` 404

**Пакет реестрі / GitHub ұйымы бойынша тексеру:**
- (F) `https://github.com/search?q=paybox+money+kz+OR+freedom_pay&type=repositories` — `PayBox/paybox-authentication-sdk` (жалғыз ресми)
- (F) `https://github.com/search?q=sunmi+pay+sdk&type=repositories` — SUNMI ресми ұйымы нәтижеде жоқ
- (F) `https://pub.dev/packages?q=...` — Kaspi/Halyk/Freedom/TipTopPay үшін **бірде-бір пакет жоқ**
- ⚠ `https://github.com/search?q=kaspi+qr...` — **HTTP 429**, Kaspi бойынша GitHub-ұйым тексеруі аяқталмады

### 13.4 МСМП / бірыңғай QR (барлығы 2026)
- (S) `https://tengrinews.kz/kazakhstan_news/19-iyulya-kazahstane-ofitsialno-zarabotaet-edinyiy-qr-kod-604152/`
- (S) `https://www.zakon.kz/finansy/6521826-chto-takoe-edinyy-QRkod-i-kak-on-izmenit-zhizn-kazakhstantsev.html`
- (S) `https://www.zakon.kz/amp/finansy/6525291-kak-pokupki-po-QR-izmenyat-platezhnyy-rynok-kazakhstana.html`
- (S) `https://kapital.kz/finance/149409/v-nacbanke-nazvali-datu-zapuska-edinogo-qr-dlya-vseh-bankov.html`
- (S) `https://profit.kz/news/74283/Mezhbankovskie-platezhi-po-QR-kodu-dostupni-vsem-kazahstancam-s-19-iulya-2026-goda/`
- (S) `https://kz.kursiv.media/2026-08-24/fvfv-po-edinomu-qr-za-mesyac-proshlo-platezhey-na-210-mlrd-te-ge/`
- (S) `https://forbes.kz/articles/mezhbankovskie-platezhi-po-qr-kodu-stanut-dostupny-vsem-kazahstantsam-s-19-iyulya-0cae3a`
- (S) `https://investfuture.ru/articles/kazakhstan-i-kitay-zapustyat-ediniy-qr-...` — ҚР–Қытай QR

### 13.5 Терминалдар, 1С драйверлері, касса ПО
- (S) `https://docs.sunmi.com/en/documentation/smart-payment-products/p2-pro/` , `https://www.sunmi.com/en/p2-pro/`
- (F) `https://github.com/OmniAid-DevelopmentTeam/capacitor-sunmi-pay-plugin` — `com.sunmi.pay.hardware.aidlv2.*` ⚠
- (F) `https://github.com/munisp/agentbanking` — `SunmiPaySDKV2_v2.0.17_2024-10-24` ⚠
- (S) `https://blog.rospertech.com/sunmi-semi-integrated-payment-sdk-four-modes/`
- (S) `https://faqs.pax.us/wp-content/uploads/2020/05/PAXSTORE-Developer-Guide-V1.08-02-04-2020-1.pdf` ⚠ 2020
- (F) `https://github.com/androiddevnotesforks/Flutter-PAX-Terminal` ⚠ 2023, бос README
- (S) `https://developer.cybersource.com/docs/cybs/en-us/payworks-sdk/developer/all/na/payworks-sdk/card-readers/pax-a920.html` — BroadPOS
- (S) `https://tandemsoft.kz/projects/pos/` , `https://arida.kz/dev/pos/kaspi/instructions`
- (S) `https://pos-plugin.kz/instruction` , `https://forte.pos-plugin.kz/instruction` — iiko плагині
- (S) `https://infostart.ru/1c/tools/2334511/` — 1С:Розница + Kaspi/Halyk
- (S) `https://promo.webkassa.kz/integrations` , `https://promo.webkassa.kz/tpost/3l3zyj8x41-integratsiya-webkassa-i-terminala-kaspi`
- (S) `https://kasselect.ru/view_post.php?id=123` — Frontol + эквайринг хаттамалары
- (S) `https://www.pos-shop.ru/upload/iblock/be4/.../Frontol-6.-Rukovodstvo-administratora.pdf` — ШТРИХ-М-ФР-KZ ⚠

### 13.6 Үшінші тұлғаның зерттеу файлдары (тек лид ретінде, `[ЖАНАМА]`)
- (F) `https://github.com/theYahia/WWmcp` — `research/archive/V4/research/RESEARCH_03_KAZAKHSTAN.md`,
  `research/cis-market/central_asia_deep.md`, `research/archive/V4/KZ/DETAILED_IMPLEMENTATION_KZ.md`
  → Forte/TipTopPay/Kaspi онбординг мәліметтерінің бірінші көзі. **Тәуелсіз тексерілмеген.**

### 13.7 Бәсекелестер (контекст)
- (S) `https://paloma365.kz/kaspi-kassa` — Paloma365 + Kaspi интеграциясы
- (S) `https://joinposter.com/en/applications/kaspi` — Poster + Kaspi терминалы
- (S) `https://help.rekassa.kz/ru/re-kassa-3-0/podkliuchieniie-avtomatichieskoi-fiskalizatsii-v-kaspi-maghazinie` — re:Kassa + Kaspi

