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
5.  **Platform Ban Risk:** Using low-level hooks to block system requests and modify API payloads on the fly always carries a non-zero risk of client instability or platform-level account bans. Use at your own risk.

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
* **Queue Jittering:** Defers outgoing messages into a local queue, dynamically inserting delays into the `scheduleDate` parameters to disrupt automated active-hour tracking.

---

## 🇷🇺 Русский (На пальцах и без пафоса)

### 1. Sticker Engine (Основная фича)
**Sticker Engine** перехватывает твою писанину прямо перед отправкой и рендерит её в прозрачный WebP-стикер. Сверху накидываются геометрические искажения и искусственная «грязь». 

Зачем это надо? Чтобы всякие спамеры, парсеры и боты-агрегаторы (типа Глаза Бога, телеграм-логов и прочей нечисти) обламывались на этапе тупого сбора текста. Чат не скроешь, но парсить его флешкой уже не выйдет.

#### Че под капотом:
* **Подмена типов:** Вместо обычного текста летит картинка. Простые логгеры, которые умеют читать только текстовые поля в базе, получают дырку от бублика.
* **Джиттер координат (микро-сдвиги):** Буквы и тени при каждом рендере смещаются на доли пикселя. Из-за этого сглаживание (anti-aliasing) отрабатывает каждый раз по-новому, и у одного и того же текста всегда будет разный хэш файла (SHA-256). Хрен ты найдешь дубликаты по хэшам.
* **Кривой холст (Skewing):** Слегка косит векторную сетку по оси X. Простые OCR-ки, которые ищут ровную базовую линию текста, сразу начинают спотыкаться.
* **Шум и помехи:** Через маску смешивания `SRC_ATOP` поверх текста кидаются случайные точки и линии. Это ломает автоматическую разрезку картинки на отдельные символы у роботов.

**Важно:** Чудес не бывает. Нормальные современные нейронки (VLM) и продвинутый софт распознают этот текст на раз-два. Вся суть затеи - заставить того, кто за тобой следит, качать тяжелые WebP вместо килобайта текста и греть свои видюхи на OCR-анализ. Бьем по карману и ресурсам парсеров.

---

### 2. AyuVeil (Затыкание дыр и паранойя)
**AyuVeil** - это набор низкоуровневых хуков (костылей, если угодно) для закручивания гаек в клиенте. Он лезет во внутренние методы приложения и фильтрует трафик к серверам.

#### Что умеет:
* **Защита от Zero-Click уязвимостей (вырезаем медиа):**
  Хукаем методы в классах `MessageObject`, `ChatMessageCell` (`setMessageObject`), `DownloadController` и `FileLoader`. Если прилетает сообщение от чужака, плагин на лету подменяет структуру `message.media` на пустую заглушку `TLRPC$TL_messageMediaEmpty`, а также вычищает кнопки (`reply_markup`) и форматирование (`entities`). Меньше парсится тяжелыми библиотеками внутри клиента - меньше шансов поймать эксплойт без клика.
* **Вырезаем WebRTC (прощай, утечка IP через звонки):**
  В `pre_request_hook` наглухо блокируются любые запросы с сигнатурами телефонии (`phone.*`, `call.*`). Звонки отвалятся совсем, зато никто не вытянет твой реальный IP-адрес через P2P-соединение. Заодно блокируются запросы WebApp (`getWebView`, `requestWebView`) и геолокация (`getLocated`), чтоб сайты внутри телеги не палили твое железо и местоположение.
* **Блокируем стук на сервера (телеметрию):**
  Режет отправку отладочных логов (`saveAppLog`), системной статы и фоновый импорт твоих контактов на сервера Дурова (`importContacts`).
* **Отправка с джиттером времени:**
  Сообщения не летят сразу, а складываются во внутренний стек и уходят с рандомной задержкой через `scheduleDate`. Сложнее будет сопоставить время твоей активности в сети.

---

## 🇨🇳 中文

### 1. Sticker Engine (核心模块)
本模块在发送阶段拦截你的文本消息，并将其渲染为透明的 WebP 格式动态贴纸。它通过将可被轻易索引的纯文本转换为媒体文件，从而大幅提高群控、监控软件大批量抓取数据的技术和硬件成本。

#### 核心技术点：
* **类型混淆 (Type Obfuscation)：** 将字符串负载替换为 WebP 图像数据，使仅能读取文本字段的简易爬虫直接抓取空值。
* **坐标抖动 (Coordinate Jitter)：** 每次渲染时，字形和阴影会在亚像素（Sub-pixel）级别发生微调。这会强制抗锯齿算法生成不同的边缘像素，即使发送完全相同的文本，生成的 WebP 文件的哈希值 (SHA-256) 也会完全不同。
* **画布倾斜 (Canvas Skewing)：** 在 X 轴上对文本进行轻微的几何倾斜，破坏基于基线（Baseline）检测的简易 OCR 引擎的识别流程。
* **对抗性噪声遮罩 (Noise Overlays)：** 采用 `SRC_ATOP` 混合模式在文本上叠加随机透明度的线条和噪点，干扰机器自动进行字符分割（Segmentation）。

**现实局限性：** 该方案无法提供绝对的安全。现代多模态大模型（VLM）和经过特定训练的深度学习 OCR 管道依然能轻松读懂这些贴纸。本工具的核心目的在于提高大众化监控的运行成本（迫使对方下载体积更大的 WebP 图片并消耗大量 GPU 算力进行 OCR 分析），而非免疫针对性的监视。

---

### 2. AyuVeil (辅助加固组件)
AyuVeil 是一个客户端加固的辅助工具，通过方法 Hook（底层拦截）和 MTProto 请求过滤来限制客户端的隐私泄露与静默网络行为。

#### 运行机制：
* **解析隔离（防范零点击漏洞）：**
  拦截 `MessageObject`、`ChatMessageCell`（`setMessageObject`）、`DownloadController` 和 `FileLoader` 等底层渲染方法。当收到来自非信任来源的非发送（接收）消息时，自动将 `media` 结构替换为空对象 `TLRPC$TL_messageMediaEmpty`，并清空 `reply_markup`（内联按钮）和 `entities`（格式化实体）。此举旨在减少客户端解析媒体文件和特殊格式时的潜在漏洞暴露面（Attack Surface）。
* **切断 VoIP（防范 WebRTC IP 泄露）：**
  在 `pre_request_hook` 中直接拦截并丢弃所有包含电话和通话签名的请求 (`phone.*` 和 `call.*`)。这将从 MTProto 协议层彻底禁用 VoIP 通话功能，从而完全规避通过 WebRTC 建立 P2P 连接时暴露真实 IP 的风险。同时，还会拦截 WebApp 初始化请求 (`getWebView` 和 `requestWebView`) 以及位置请求 (`getLocated`)，防止后台网页静默获取设备指纹。
* **屏蔽遥测与日志：**
  拦截并阻止客户端向服务器发送调试日志 (`saveAppLog`)、系统统计数据和后台联系人同步 (`importContacts`)。
* **时间抖动队列（Queue Jittering）：**
  将待发送的消息放入本地队列，自动微调并延迟 `scheduleDate` 参数。这会打乱消息发出的绝对时间，降低分析人员通过你的发言时间戳建立行为画像的精准度。

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
پلاگین **AyuVeil** یک ابزار آزمون سخت‌سازی کلاینت است که از طریق متد هوک‌ها و فیلتر کردن پکت‌های MTProto تلاش می‌کند برخی روزنه‌های نشت متادیتای سیستم را محدود کند.

#### عملکرد فنی سخت‌سازی:
* **کنترل پارسر تصاویر (کاهش ریسک آسیب‌پذیری‌های بدون کلیک):**
  با هوک کردن کلاس‌های `MessageObject`، `ChatMessageCell`، `DownloadController` و `FileLoader` در پیام‌های دریافتی ناشناس، مقدار `media` را خالی کرده و فیلدهای دکمه‌های شیشه‌ای و انتیتی‌ها را پاکسازی می‌کند تا کلاینت را از پارس کردن متادیتای مخرب احتمالی محافظت کند.
* **محدودسازی نشت آی‌پی از طریق لایه VoIP:**
  با رهگیری و مسدود کردن درخواست‌های دارای امضای تماس (`phone.*` و `call.*`)، سیگنال‌دهی WebRTC را در سطح پروتکل متوقف می‌کند تا شانس افشای IP واقعی در بسترهای همتا‌به‌همتا کاهش یابد. همچنین درخواست‌های باز شدن وب‌اپ‌ها (`getWebView` و `requestWebView`) و خدمات مکانی کلاینت بلاک می‌شوند.
* **مسدودسازی گزارش‌های خطا و آمارها:**
  درخواست‌های مربوط به ثبت و ارسال لاگ‌های عیب‌یابی کلاینت (`saveAppLog`) و همچنین همگام‌سازی ناخواسته دفترچه مخاطبان با سرور تلگرام را فیلتر می‌کند.
* **صف‌بندی پیام‌ها با تاخیر تصادفی:**
  پیام‌ها را با تاخیرهای تصادفی ریز (Jitter) در زمان ارسال بازنویسی می‌کند تا تشخیص الگوهای آنلاین بودن و رفتارهای زمانی شما برای ناظران سخت‌تر شود.

---

## Installation & Setup / Установка / 安装 / راهنمای نصب

### 1. Font Configuration / Настройка шрифта / 字体配置 / تنظیم فونت
To prevent OS-level font fingerprinting, you must supply a custom TrueType Font (`.ttf`):
1. Locate a standard `.ttf` font file (e.g., Arial, DejaVuSans, etc.).
2. Copy this file into your Telegram client's cache directory on Android:
   `Android/data/<your.client.package>/cache/`
3. In the plugin settings, enter the exact filename of the font (e.g., `myfont.ttf`). Do not leave this field blank.

---

### 2. Plugin Deployment / Как завести плагины / 插件部署 / فعال‌سازی پلاگین‌ها

#### Prerequisites / Предварительные требования / 前提条件 / پیش‌نیازها
Before loading the files, make sure the Plugin Engine is active:
* **exteraGram/AyuGram:** Go to Settings -> Client Preferences -> Plugins, and toggle the Plugin Engine ON. Enable developer/advanced settings if prompted.

#### Deployment Steps / Шаги установки / 安装步骤 / مراحل نصب
1. Download both plugin files (`sticker_engine.plugin` and `ayu_veil.plugin`).
2. Send these files to your **Saved Messages** within the modded Telegram client.
3. Tap on each file inside the chat and select **Load/Install Plugin** from the context menu.
4. Restart the client if required, then navigate to your client's plugin settings interface to customize parameters. Activate "Enable Sticker Generator" in Sticker Engine and configure the desired hardening rules in AyuVeil.
