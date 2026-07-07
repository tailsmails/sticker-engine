# Sticker Engine & AyuVeil

A privacy-conscious, client-hardening experiment for modded Telegram clients (exteraGram/AyuGram).

*   **Sticker Engine (Core Component):** An experimental text-to-image WebP renderer designed to raise the computational barrier for automated text scrapers.
*   **AyuVeil (Auxiliary Tool):** A collection of API hooks and filters aimed at reducing some common client-side metadata leaks and telemetry.

---

## LIMITATIONS & REALISTIC THREAT MODEL (MUST READ)
**This software does not offer absolute privacy or untraceable communication.** 

1.  **AI & Advanced OCR Bypass:** While the added noise and geometric distortion disrupt basic, automated OCR engines (like older Tesseract setups), they **do not** stop modern multimodal AI models (such as GPT-4o, Claude, or custom deep-learning pipelines). An adversary with sufficient GPU resources and budget can easily clean the noise and read the stickers.
2.  **Increased Cost, Not Immunity:** The primary goal of the Sticker Engine is economic: it forces scrapers to download larger media files (WebP vs. plain text) and spend CPU/GPU cycles on OCR instead of reading database strings. It makes mass, automated harvesting more expensive, but a targeted or well-funded observer will bypass this easily.
3.  **Fragility of Client Hooks:** AyuVeil relies on low-level Java method hooking. Any major update to the Telegram codebase may break these hooks silently, potentially re-enabling telemetry, VoIP, or WebApp loads without warning.
4.  **No Protection Against Device Compromise:** If your operating system is compromised (via keyloggers, screen scraping, or spyware), client-side modifications are completely ineffective.

---

## 🇷🇺 Русский (Native Technical & Realistic Style)

### 1. Sticker Engine (Основной модуль)
**Sticker Engine** перехватывает исходящие текстовые сообщения на этапе отправки и рендерит их в прозрачные стикеры формата WebP, добавляя контролируемые геометрические искажения и искусственный шум. 

Инструмент создавался как эксперимент по увеличению стоимости массового автоматического сбора данных (dragnet surveillance). Он не скрывает факт общения, но усложняет парсинг текста ботами-агрегаторами (такими как Глаз Бога, Telelog и др.).

#### Как это работает на техническом уровне:
* **Замена типа данных:** Вместо строки передается медиа-объект. Простые логгеры, собирающие только текстовое поле сообщений, упираются в пустые значения.
* **Микро-сдвиги рендеринга (Jitter):** Координаты глифов и теней смещаются на случайные доли пикселя при каждом вызове. Алгоритмы сглаживания (anti-aliasing) формируют уникальные граничные пиксели, что изменяет итоговый хэш файла (SHA-256) при повторной отправке одного и того же текста.
* **Искажение геометрии холста:** Слегка наклоняет векторную сетку по оси X, что снижает эффективность алгоритмов поиска базовой линии (baseline detection) в простых OCR-системах.
* **Искусственные помехи:** Через маску смешивания `SRC_ATOP` поверх текста накладываются линии и точки случайной прозрачности. Это ухудшает автоматическую сегментацию символов.

**Ограничение метода:** Современные мультимодальные нейросети (VLM) и продвинутые конвейеры обработки изображений способны распознать этот текст. Использование стикеров лишь заставляет наблюдателя тратить больше трафика (скачивание WebP вместо текста) и вычислительных мощностей GPU на OCR-анализ.

---

### 2. AyuVeil (Вспомогательный модуль харднинга)
**AyuVeil** (ранее AyuPlus) - это экспериментальный набор низкоуровневых правил для ужесточения (hardening) конфигурации клиента. Он работает через перехват внутренних методов и фильтрацию некоторых запросов API.

#### Основные механизмы:
* **Попытка фильтрации входящих структур (Защита от Zero-Click):**
  Через хуки классов `MessageObject`, `ChatMessageCell` (`setMessageObject`), `DownloadController` и `FileLoader` плагин заменяет структуру `message.media` на пустой объект `TLRPC$TL_messageMediaEmpty` для входящих сообщений от недоверенных источников, а также очищает списки `reply_markup` и `entities`. Это снижает вероятность срабатывания уязвимостей парсинга медиа-библиотек внутри клиента.
* **Снижение вероятности утечки IP через WebRTC:**
  В `pre_request_hook` блокируются запросы с сигнатурами телефонии (`phone.*`, `call.*`). Это отключает протокол VoIP на уровне MTProto, минимизируя риск раскрытия реального IP-адреса через P2P-соединения. Также блокируются запросы WebView (`getWebView`, `requestWebView`) и запросы геопозиции (`getLocated`).
* **Подавление встроенной телеметрии:**
  Перехватывает и отклоняет отправку отладочных логов (`saveAppLog`), системной статистики и фоновую синхронизацию списка контактов (`importContacts`).
* **Очередь с микро-задержками (Jitter):**
  Сохраняет сообщения во внутренний JSON-стек и отправляет их с задержкой, пересчитывая `scheduleDate` с добавлением случайного смещения. Это затрудняет статистический анализ времени вашей активности.

---

## 🇬🇧 English

### 1. Sticker Engine (Core Module)
This module intercepts outgoing text and renders it to a transparent WebP image. It raises the technical and financial costs of mass data harvesting by converting indexable plain text into media files.

#### Key Features:
* **Type Obfuscation:** Replaces string payloads with WebP image data to bypass simple text scrapers.
* **Coordinate Jitter:** Offsets text rendering at sub-pixel levels to force unique anti-aliasing outputs, ensuring different SHA-256 hashes even for identical messages.
* **Canvas Skewing:** Skews the text on the X-axis to disrupt basic baseline-based OCR readers.
* **Adversarial Noise Overlays:** Uses `SRC_ATOP` blending to overlay random lines and dots over text bounds, disrupting machine-driven character segmentation.

---

### 2. AyuVeil (Auxiliary Hardening Component)
AyuVeil is a companion component that uses method hooks and MTProto request filtering to limit client telemetry and mitigate certain attack vectors.

#### Technical Architecture:
* **Parser Isolation:** Hooks render-critical methods in `MessageObject`, `ChatMessageCell`, `DownloadController`, and `FileLoader`. It forces incoming messages (`not out`) to strip `media` payloads (replacing them with `TLRPC$TL_messageMediaEmpty`), `reply_markup` configurations, and `entities`. This reduces the rendering attack surface.
* **IP Leak Reduction:** Intercepts and drops outgoing VoIP requests (`phone.*`, `call.*`) and WebApp initialization vectors (`getWebView`, `requestWebView`), preventing IP harvesting through WebRTC peer-to-peer signalling or silent background browser loads.
* **Telemetry Suppression:** Blocks `saveAppLog` requests and background contact imports (`importContacts`).
* **Queue Jittering:** defers outgoing messages into a local queue, dynamically inserting delays into the `scheduleDate` parameters to disrupt automated active-hour tracking.

---

## 🇮🇷 فارسی

### ۱. موتور اصلی: Sticker Engine
این ماژول پیام‌های متنی ارسالی شما را به تصاویر استیکر شفاف با فرمت WebP تبدیل می‌کند. هدف اصلی این بخش، بالا بردن هزینه محاسباتی پایش اطلاعات برای سیستم‌های جمع‌آوری فله‌ای داده (مانند ابزارهای عمومی OSINT) است.

#### ویژگی‌های فنی:
* **تغییر نوع داده:** جایگزینی داده‌های متنی با استیکرهای WebP جهت به خطا انداختن خزنده‌های متنی ساده.
* **لرزش صدم پیکسلی (Jitter):** جابه‌جایی جزئی حروف و سایه‌ها در هر بار رندر برای تغییر بایت‌های تصویر خروجی و تولید هش فایل (SHA-256) متفاوت در هر ارسال.
* **کج‌نمایی بوم (Skewing):** کج کردن متن با زاویه‌های تصادفی افقی برای ایجاد اختلال در سیستم‌های تراز افقی متون در OCRها.
* **نویزگذاری روی حروف:** ترسیم خطوط و نقطه‌های تصادفی با شفافیت متغیر روی سطح حروف چت (با متد `SRC_ATOP`) برای ایجاد اختلال در بخش‌بندی تصاویر در موتورهای پردازش تصویر.

**توجه واقعی:** این روش امنیت ۱۰۰٪ ایجاد نمی‌کند. ابزارهای هوش مصنوعی مالتی‌مودال جدید یا خزنده‌های مجهز به فیلترهای پیش‌پردازش تصویر با صرف هزینه محاسباتی و زمان بیشتر، همچنان قادر به خواندن این متون خواهند بود. این ابزار صرفاً پایش ارزان و فله‌ای را متوقف می‌کند.

---

### ۲. ابزار کمکی: AyuVeil (کاهش سطح آسیب‌پذیری و تلمتری)
پلاگین **AyuVeil** (نسخه قبلی AyuPlus) یک ابزار آزمون سخت‌سازی کلاینت است که از طریق متد هوک‌ها و فیلتر کردن پکت‌های MTProto تلاش می‌کند برخی روزنه‌های نشت متادیتای سیستم را محدود کند.

#### عملکرد فنی سخت‌سازی:
* **کنترل پارسر تصاویر (کاهش ریسک آسیب‌پذیری‌های بدون کلیک):**
  با هوک کردن کلاس‌های `MessageObject`، `ChatMessageCell`، `DownloadController` و `FileLoader` در پیام‌های دریافتی ناشناس، مقدار `media` را خالی کرده و فیلدهای دکمه‌های شیشه‌ای و انتیتی‌ها را پاکسازی می‌کند تا کلاینت را از پارس کردن متادیتای مخرب احتمالی محافظت کند.
* **محدودسازی نشت آی‌پی از طریق لایه VoIP:**
  با رهگیری و مسدود کردن درخواست‌های دارای امضای تماس (`phone.*` و `call.*`)، سیگنال‌دهی WebRTC را در سطح پروتکل متوقف می‌کند تا شانس افشای IP واقعی در بسترهای همتا‌به‌همتا کاهش یابد. همچنین درخواست‌های باز شدن وب‌اپ‌ها (`getWebView`) و خدمات مکانی کلاینت بلاک می‌شوند.
* **مسدودسازی گزارش‌های خطا و آمارها:**
  درخواست‌های مربوط به ثبت و ارسال لاگ‌های عیب‌یابی کلاینت (`saveAppLog`) و همچنین همگام‌سازی ناخواسته دفترچه مخاطبان با سرور تلگرام را فیلتر می‌کند.
* **صف‌بندی پیام‌ها با تاخیر تصادفی:**
  پیام‌ها را با تاخیرهای تصادفی ریز (Jitter) در زمان ارسال بازنویسی می‌کند تا تشخیص الگوهای آنلاین بودن و رفتارهای زمانی شما برای ناظران سخت‌تر شود.

---

## Installation & Setup / Инструкция по установке

### 1. Font Configuration / Настройка шрифта
To prevent OS-level font fingerprinting, you must supply a custom TrueType Font (`.ttf`):
1. Locate a standard `.ttf` font file (e.g., Arial, DejaVuSans, etc.).
2. Copy this file into your Telegram client's cache directory on Android:
   `Android/data/<your.client.package>/cache/`
3. In the plugin settings, enter the exact filename of the font (e.g., `myfont.ttf`). Do not leave this field blank.

### 2. Plugin Deployment / Установка плагинов
1. Download both Python plugin files (`sticker_engine.py` and `ayu_veil.py`).
2. Upload the files to your saved messages inside modded Telegram client (e.g., exteraGram / AyuGram) and load them.
3. Configure the settings for both plugins as desired. Turn on the "Enable Sticker Generator" in Sticker Engine settings and activate the hardening rules in AyuVeil.
