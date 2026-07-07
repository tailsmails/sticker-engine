# Sticker Engine / AyuVeil / Auto Delete

A privacy-conscious, client-hardening experiment for modded Telegram clients (exteraGram/AyuGram). 

*   **Sticker Engine (Core Component):** An experimental text-to-image WebP renderer designed to raise the computational barrier for automated text scrapers.
*   **AyuVeil (Auxiliary Tool):** A collection of API hooks and filters aimed at reducing some common client-side metadata leaks and telemetry.
*   **Stealth Auto Delete (Auxiliary Tool):** A smart, randomized self-destruct mechanism designed to starve passive data scrapers.

---

## LIMITATIONS & REALISTIC THREAT MODEL (MUST READ)
**This software does not offer absolute privacy or untraceable communication.** 

1.  **AI & Advanced OCR Bypass:** While the added noise and geometric distortion disrupt basic, automated OCR engines (like older Tesseract setups), they **do not** stop modern multimodal AI models (such as GPT-4o, Claude, or custom deep-learning pipelines). An adversary with sufficient GPU resources and budget can easily clean the noise and read the stickers.
2.  **Increased Cost, Not Immunity:** The primary goal of the Sticker Engine is economic: it forces scrapers to download larger media files (WebP vs. plain text) and spend CPU/GPU cycles on OCR instead of reading database strings. It makes mass, automated harvesting more expensive, but a targeted or well-funded observer will bypass this easily.
3.  **Fragility of Client Hooks:** AyuVeil relies on low-level Java method hooking. Any major update to the Telegram codebase may break these hooks silently, potentially re-enabling telemetry, VoIP, or WebApp loads without warning.
4.  **No Protection Against Device Compromise:** If your operating system is compromised (via keyloggers, screen scraping, or spyware), client-side modifications are completely ineffective.
5.  **Platform Ban Risk:** Using low-level hooks to block system requests and modify API payloads on the fly always carries a non-zero risk of client instability or platform-level account bans. Use at your own risk.
6.  **API Rate Limits (Flood Wait):** Sending a media file (WebP) for every single text message hits Telegram's API harder than standard text. Rapid-fire messaging (spamming short texts) *will* trigger temporary anti-spam limits (Flood Wait). Type longer messages, send fewer times.

---

## 🇬🇧 English

### 1. Sticker Engine (Core Module)
This module intercepts outgoing text and renders it to a transparent WebP image. It raises the technical and financial costs of mass data harvesting by converting indexable plain text into media files.

#### Key Features:
* **Type Obfuscation:** Replaces string payloads with WebP image data to bypass simple text scrapers.
* **Coordinate Jitter:** Offsets text rendering at sub-pixel levels to force unique anti-aliasing outputs, ensuring different SHA-256 hashes even for identical messages.
* **Canvas Skewing:** Skews the text on the X-axis to disrupt basic baseline-based OCR readers.
* **Adversarial Noise Overlays:** Uses `SRC_ATOP` blending to overlay random lines and dots over text bounds, disrupting machine-driven character segmentation.

### 2. AyuVeil (Auxiliary Hardening Component)
A companion component that uses method hooks and MTProto request filtering to limit client telemetry and mitigate certain attack vectors.

#### Technical Architecture:
* **Parser Isolation:** Hooks render-critical methods (`MessageObject`, `DownloadController`, etc.). It forces incoming messages (`not out`) to strip `media` payloads and `entities`, replacing them with `TLRPC$TL_messageMediaEmpty`. This reduces the rendering attack surface for zero-click exploits.
* **IP Leak Reduction:** Intercepts outgoing VoIP requests (`phone.*`, `call.*`) and WebApp initialization vectors, preventing IP harvesting through WebRTC peer-to-peer signalling.
* **Stealth Inline Queries:** Buffers inline bot queries (e.g., `@pic`). The client will *not* send your keystrokes to the server unless your text ends with a semicolon (`;`).
* **Telemetry Suppression:** Blocks `saveAppLog` requests and background contact imports (`importContacts`).
* **Queue Jittering:** Defers outgoing messages into a local queue, dynamically inserting delays into `scheduleDate` parameters to disrupt automated active-hour tracking.

### 3. Stealth Auto Delete (Data Starvation)
A background queue that deletes your sent messages locally and remotely. Unlike Telegram's native auto-delete, this works dynamically based on chat type (PV, Group, Channel) and media type.
* **The Goal:** Passive scrapers usually run on a delay to avoid API limits. By injecting random jitter into the deletion timer, this tool aims to wipe the message *before* the scraper's polling cycle reaches it.

---

## 🇷🇺 Русский (На пальцах и без пафоса)

### 1. Sticker Engine (Основная фича)
Перехватывает твою писанину перед отправкой и рендерит её в прозрачный WebP-стикер с геометрическими искажениями и искусственной «грязью». 

Зачем? Чтобы агрегаторы (типа Глаза Бога) обламывались на этапе тупого сбора текста. Чат не скроешь, но парсить его флешкой уже не выйдет.

#### Че под капотом:
* **Подмена типов:** Вместо текста летит картинка. Логгеры, читающие только текстовые поля в базе, идут лесом.
* **Джиттер координат:** Буквы при каждом рендере смещаются на доли пикселя. У одного и того же текста всегда будет разный хэш файла (SHA-256).
* **Кривой холст (Skewing):** Слегка косит сетку по оси X. Простые OCR-ки спотыкаются о кривую базовую линию.
* **Шум и помехи:** Маска `SRC_ATOP` кидает поверх текста случайные точки и линии. Это ломает автоматическую разрезку картинки на символы.

### 2. AyuVeil (Затыкание дыр и паранойя)
Набор низкоуровневых хуков для закручивания гаек в клиенте.

* **Защита от Zero-Click:** Если прилетает сообщение от чужака, плагин на лету подменяет структуру `message.media` на пустышку и вычищает кнопки. Меньше парсится - меньше шансов поймать эксплойт.
* **Вырезаем WebRTC:** Блокируются любые запросы телефонии (`phone.*`). Никто не вытянет твой IP через P2P.
* **Скрытый инлайн:** Запросы к ботам (например, `@pic`) локально блокируются. Они уйдут на сервер **только** если ты поставишь точку с запятой (`;`) в конце запроса. Защита от кейлоггинга.
* **Анти-телеметрия:** Режет отправку логов (`saveAppLog`) и фоновый импорт контактов.
* **Джиттер времени:** Сообщения уходят с рандомной задержкой через `scheduleDate`. Сложнее сопоставить время твоей активности.

### 3. Stealth Auto Delete (Голодный паек для парсеров)
Умная удалялка сообщений. Задает плавающий (рандомный) таймер на удаление в зависимости от типа чата и медиа.
* **Суть:** Парсеры работают с задержкой, чтобы не ловить лимиты от телеги. Плагин трет твое сообщение до того, как бот-шпион успеет его сграбить.

**ВНИМАНИЕ:** Не спамь короткими сообщениями. Отправка кучи стикеров вместо текста быстро триггерит Flood Wait от телеги. Пиши объемнее, отправляй реже.

---

## 🇨🇳 中文

### 1. Sticker Engine (核心模块)
在发送阶段拦截文本消息，并将其渲染为透明的 WebP 图像。通过将纯文本转换为媒体文件，大幅提高批量抓取数据的技术和硬件成本。

#### 核心技术点：
* **类型混淆：** 将字符串替换为 WebP 图像，使简单的文本爬虫抓取空值。
* **坐标抖动：** 每次渲染时字形发生亚像素级微调。即使文本相同，生成的 WebP 哈希值 (SHA-256) 也完全不同。
* **画布倾斜：** 在 X 轴上对文本进行几何倾斜，破坏基于基线检测的简易 OCR 引擎。
* **对抗性噪声：** 使用 `SRC_ATOP` 在文本上叠加随机线条和噪点，干扰机器自动字符分割。

### 2. AyuVeil (辅助加固组件)
通过方法 Hook 和 MTProto 请求过滤，限制客户端隐私泄露。

* **解析隔离 (防零点击漏洞)：** 拦截渲染底层方法。当收到非信任消息时，强制将 `media` 替换为空对象，清空内联按钮和格式，减少潜在漏洞暴露面。
* **切断 WebRTC：** 拦截所有通话签名请求 (`phone.*`)，规避 P2P 连接暴露真实 IP。
* **隐秘内联查询：** 内联机器人 (如 `@pic`) 的请求会被本地拦截，直到你在文本末尾输入分号 (`;`) 才会发送给服务器，防止键盘记录。
* **屏蔽遥测：** 阻止客户端向服务器发送调试日志和后台联系人同步。
* **时间抖动：** 将待发消息放入队列并随机延迟，打乱绝对发言时间戳。

### 3. Stealth Auto Delete (数据饥饿)
静默的随机延迟删除队列。与官方自动删除不同，它根据聊天类型和载荷动态注入随机延迟。
* **核心逻辑：** 爬虫通常需要轮询延迟以避免 API 限制。此工具旨在利用时间差，在爬虫抓取到数据前将其销毁。

**注意 (API 限制)：** 把每条短文本都当成图片发送会增加 API 负载。频繁刷屏会导致账号触发临时限制 (Flood Wait)。请合并文本，减少发送频率。

---

## 🇮🇷 فارسی

### ۱. موتور اصلی: Sticker Engine
این ماژول پیام‌های متنی شما را قبل از ارسال به تصاویر استیکر (WebP) تبدیل می‌کند. هدف، بالا بردن هزینه محاسباتی پایش اطلاعات برای سیستم‌های جمع‌آوری فله‌ای داده (ربات‌های جاسوسی تلگرامی) است.

#### ویژگی‌های فنی:
* **تغییر نوع داده:** جایگزینی متن با عکس جهت دور زدن خزنده‌های دیتابیس‌های متنی.
* **لرزش صدم پیکسلی (Jitter):** جابه‌جایی جزئی حروف در هر بار رندر. این کار باعث می‌شود حتی برای یک متن تکراری، هش فایل (SHA-256) متفاوتی تولید شود.
* **کج‌نمایی بوم (Skewing):** کج کردن متن با زاویه‌های تصادفی برای ایجاد اختلال در موتورهای OCR ساده.
* **نویزگذاری:** ترسیم خطوط تصادفی روی حروف (متد `SRC_ATOP`) برای ایجاد اختلال در بخش‌بندی کلمات توسط ماشین.

### ۲. ابزار کمکی: AyuVeil (سخت‌سازی کلاینت)
مجموعه‌ای از هوک‌ها برای مسدودسازی نشت متادیتا و تلمتری در سطح کلاینت.

* **ایزوله‌سازی پارسر تصاویر (ضد Zero-Click):** در پیام‌های دریافتی ناشناس، مقدار `media`، دکمه‌های شیشه‌ای و انتیتی‌ها پاکسازی می‌شوند تا کلاینت از پردازش متادیتای مخرب احتمالی در امان بماند.
* **مسدودسازی نشت IP:** با مسدود کردن ریکوئست‌های تماس (`phone.*`)، سیگنال‌دهی WebRTC را متوقف می‌کند تا IP واقعی لو نرود.
* **مخفی‌سازی اینلاین:** درخواست به ربات‌های اینلاین (مثل `@pic`) در پس‌زمینه مسدود است. برای ارسال درخواست، باید حتماً در انتهای متن خود از علامت نقطه‌ویرگول (`;`) استفاده کنید.
* **ضد تلمتری:** درخواست‌های ارسال لاگ عیب‌یابی کلاینت و همگام‌سازی مخاطبان با سرور را فیلتر می‌کند.
* **لرزش زمانی:** پیام‌ها با تاخیرهای تصادفی در زمان ارسال بازنویسی می‌شوند تا تشخیص الگوی آنلاین بودن شما سخت‌تر شود.

### ۳. ابزار کمکی: Stealth Auto Delete (ایجاد گرسنگی داده‌ای)
یک صف‌بندی بی‌صدا برای حذف دوطرفه پیام‌ها. برخلاف تایمر پیش‌فرض تلگرام، این ابزار بر اساس نوع چت و نوع مدیا، تاخیرهای تصادفی و متغیری اعمال می‌کند.
* **منطق کار:** ربات‌های اسکرپر برای جلوگیری از لیمیت شدن، با تاخیر دیتابیس را اسکن می‌کنند. این ابزار پیام را دقیقاً در بازه زمانیِ کوری اسکرپر (قبل از رسیدن نوبت اسکن) پاک می‌کند.

**هشدار (محدودیت Flood Wait):** تبدیل هر پیام کوتاه به فایل رسانه‌ای، فشار بیشتری به سرور تلگرام می‌آورد. ارسال رگباری پیام‌ها باعث محدودیت موقت اکانت (Flood Wait) می‌شود. پیام‌های خود را طولانی‌تر و با تعداد دفعات کمتر ارسال کنید.

---

## Installation & Setup / Установка / 安装 / راهنمای نصب

### 1. Font Configuration / Настройка шрифта / 字体配置 / تنظیم فونت
To prevent OS-level font fingerprinting, you must supply a custom TrueType Font (`.ttf`):
1. Locate a standard `.ttf` font file (e.g., Arial, DejaVuSans, etc.).
2. Copy this file into your Telegram client's cache directory on Android:
   `Android/data/<your.client.package>/cache/`
3. In the plugin settings, enter the exact filename of the font (e.g., `myfont.ttf`). Do not leave this field blank.

### 2. Plugin Deployment / Как завести плагины / 插件部署 / فعال‌سازی
**Prerequisites:** Before loading, make sure the Plugin Engine is active (exteraGram/AyuGram -> Settings -> Client Preferences -> Plugins).

1. Download the plugin files (`sticker_engine.plugin`, `ayuveil.plugin`, and `auto_delete.plugin`).
2. Send these files to your **Saved Messages**.
3. Tap on each file inside the chat and select **Load/Install Plugin** from the context menu.
4. Restart the client, navigate to your client's plugin settings, and activate/configure the desired hardening rules.
