# مقدمة إلى WebAssembly

سنتحدث في هذا القسم عن كيفية تحويل المحاكي الذي أنشأناه لتشغيله في متصفح الويب باستخدام تقنية جديدة نسبيًا تسمى *WebAssembly*. أشجعك على [قراءة المزيد](https://en.wikipedia.org/wiki/WebAssembly) عن WebAssembly. إنها صيغة لتحويل البرامج إلى ملف تنفيذي ثنائي، مشابه في النطاق لملف .exe، ولكنها مخصصة للتشغيل داخل متصفح الويب. يتم دعمها من قبل جميع المتصفحات الرئيسية، وهي معيار متعدد الشركات يتم تطويره بينها. هذا يعني أنه بدلًا من الاضطرار إلى كتابة كود الويب باستخدام JavaScript أو لغات أخرى مخصصة للويب، يمكنك كتابته بأي لغة تدعم تحويل ملفات .wasm ولا يزال بإمكانك التشغيل في المتصفح. في وقت كتابة هذا الدليل، تعد C وC++ وRust اللغات الرئيسية التي تدعمها، ولحسن الحظ بالنسبة لنا.

## الإعداد

بينما يمكننا التحويل يدويًا إلى أهداف WebAssembly في Rust، تم تطوير مجموعة من الأدوات المساعدة تسمى [wasm-pack](https://github.com/rustwasm/wasm-pack) لتسمح لنا بتحويل البرامج إلى WebAssembly بسهولة دون الحاجة إلى إضافة الأهداف والتبعيات يدويًا. ستحتاج إلى تثبيتها عبر:

```
$ cargo install wasm-pack
```

إذا كنت تستخدم Windows، قد يفشل التثبيت عند crate `openssl-sys` [راجع هذا الرابط](https://github.com/rustwasm/wasm-pack/issues/1108) وسيتعين عليك تنزيلها يدويًا من [هنا](https://rustwasm.github.io/wasm-pack/).

كما ذكرت سابقًا، سنحتاج إلى إجراء تعديل بسيط على وحدة `chip8_core` الخاصة بنا للسماح لها بالتحويل بشكل صحيح إلى هدف `wasm`. يستخدم Rust نظامًا يسمى `wasm-bindgen` لإنشاء روابط تعمل مع WebAssembly. كل الكود الذي نستخدمه من `std` جاهز بالفعل، ولكننا نستخدم أيضًا crate `rand` في الواجهة الخلفية، وهي غير مهيأة للعمل بشكل صحيح. لحسن الحظ، تدعم هذه الوظيفة، نحتاج فقط إلى تمكينها. في ملف `chip8_core/Cargo.toml` نحتاج إلى تغيير:

```toml
[dependencies]
rand = "^0.7.3"
```

إلى:

```toml
[dependencies]
rand = { version = "^0.7.3", features = ["wasm-bindgen"] }
```

كل ما يفعله هذا هو تحديد أننا سنحتاج إلى تضمين ميزة `wasm-bindgen` في crate `rand` عند التحويل، مما يسمح لها بالعمل بشكل صحيح في ملف WebAssembly الثنائي.

ملاحظة: بين وقت كتابة كود هذا الدليل وإنهاء الكتابة، تم تحديث crate `rand` إلى الإصدار 0.8. من بين التغييرات الأخرى، تمت إزالة ميزة `wasm-bindgen`. إذا كنت ترغب في استخدام أحدث إصدار من crate `rand`، يبدو أن دعم WebAssembly تم نقله إلى crate منفصل. نظرًا لأننا نستخدم فقط الوظيفة العشوائية الأساسية، لم أشعر بالحاجة إلى الترقية إلى الإصدار 0.8، ولكن إذا كنت ترغب في ذلك، يبدو أن التكامل الإضافي مطلوب.

هذه هي المرة الأخيرة التي ستحتاج فيها إلى تعديل وحدة `chip8_core` الخاصة بك، كل شيء آخر سيتم في الواجهة الأمامية الجديدة. لنقم بإعداد ذلك الآن. أولاً، لننشئ وحدة Rust جديدة عبر:

```
$ cargo init wasm --lib
```

قد تبدو هذه الأوامر مألوفة، حيث ستقوم بإنشاء مكتبة Rust جديدة تسمى `wasm`. تمامًا مثل `desktop`، سنحتاج إلى تعديل `wasm/Cargo.toml` للإشارة إلى مكان وجود `chip8_core`.

```toml
[dependencies]
chip8_core = { path = "../chip8_core" }
```

الآن، الفرق الكبير بين `desktop` وواجهة `wasm` الجديدة هو أن `desktop` كان مشروعًا تنفيذيًا، حيث كان يحتوي على `main.rs` والذي كنا نحمله ونشغله. `wasm` لن تحتوي على ذلك، فهي مخصصة للتحويل إلى ملف .wasm والذي سنحمله في صفحة ويب. ستكون صفحة الويب هي الواجهة الأمامية، لذا دعنا نضيف بعض القوالب الأساسية لصفحة HTML، فقط لبدء العمل. أنشئ مجلدًا جديدًا يسمى `web` لحفظ الكود المخصص لصفحة الويب، ثم أنشئ `web/index.html` وأضف قالب HTML الأساسي.

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Chip-8 Emulator</title>
        <meta charset="utf-8">
    </head>
    <body>
        <h1>My Chip-8 Emulator</h1>
    </body>
</html>
```

سنضيف المزيد لاحقًا، ولكن هذا يكفي حاليًا. لن يعمل برنامج الويب الخاص بنا إذا قمت ببساطة بفتح الملف في متصفح الويب، ستحتاج إلى بدء خادم ويب أولاً. إذا كان لديك Python 3 مثبتًا، وهو موجود في جميع أجهزة Mac الحديثة والعديد من توزيعات Linux، يمكنك ببساطة بدء خادم ويب عبر:

```
$ python3 -m http.server
```

انتقل إلى `localhost` في متصفح الويب الخاص بك. إذا قمت بتشغيل هذا في مجلد `web`، يجب أن ترى صفحة `index.html` معروضة. لقد حاولت العثور على طريقة بسيطة ومدمجة لبدء خادم ويب محلي على Windows، ولم أجد واحدة. أنا شخصيًا أستخدم Python 3، ولكنك مرحب باستخدام أي خدمة مشابهة أخرى، مثل `npm` أو حتى بعض إضافات Visual Studio Code. لا يهم أي منها، طالما يمكنها استضافة صفحة ويب محلية.

## تعريف واجهة برمجة تطبيقات WebAssembly

لدينا `chip8_core` جاهزة بالفعل، ولكننا نفتقد الآن جميع الوظائف التي أضفناها إلى `desktop`. تحميل ملف، التعامل مع ضغطات المفاتيح، إخبارها بالتنفيذ، إلخ. من ناحية أخرى، لدينا صفحة ويب (ستعمل) باستخدام JavaScript، والتي تحتاج إلى التعامل مع مدخلات المستخدم وعرض العناصر. وحدة `wasm` الخاصة بنا هي ما يقع في المنتصف. ستأخذ المدخلات من JavaScript وتحولها إلى أنواع البيانات المطلوبة من قبل `chip8_core`.

الأهم من ذلك، نحتاج أيضًا إلى إنشاء كائن `chip8_core::Emu` والحفاظ عليه في النطاق طوال فترة عمل صفحة الويب.

للبدء، دعنا نضمّن بعض الحزم الخارجية التي سنحتاجها للسماح لـ Rust بالتفاعل مع JavaScript. افتح `wasm/Cargo.toml` وأضف التبعيات التالية:

```toml
[dependencies]
chip8_core = { path = "../chip8_core" }
js-sys = "^0.3.46"
wasm-bindgen = "^0.2.69"

[dependencies.web-sys]
version = "^0.3.46"
features = []
```

ستلاحظ أننا نتعامل مع `web-sys` بشكل مختلف عن التبعيات الأخرى. تم تنظيم هذه الحزمة بطريقة تجعلنا بدلًا من الحصول على كل ما تحتويه ببساطة عن طريق تضمينها في `Cargo.toml`، نحتاج أيضًا إلى تحديد "ميزات" إضافية تأتي مع الحزمة، ولكنها غير متاحة بشكل افتراضي. ابق هذا الملف مفتوحًا، حيث سنضيف إلى ميزات `web_sys` قريبًا.

نظرًا لأن هذه الحزمة ستتفاعل مع لغة أخرى، نحتاج إلى تحديد كيفية التواصل بينهما. دون الخوض في التفاصيل، يمكن لـ Rust استخدام ABI لغة C للتواصل بسهولة مع اللغات الأخرى التي تدعمه، وسيُبسّط بشكل كبير ملف wasm الثنائي الخاص بنا للقيام بذلك. لذا، سنحتاج إلى إخبار `cargo` باستخدامه. أضف هذا أيضًا في `wasm/Cargo.toml`:

```toml
[lib]
crate-type = ["cdylib"]
```

ممتاز. الآن إلى `wasm/src/lib.rs`. لنقم بإنشاء struct سيحتوي على كائن `Emu` الخاص بنا بالإضافة إلى جميع وظائف الواجهة الأمامية التي نحتاجها للتفاعل مع JavaScript والتشغيل. سنحتاج أيضًا إلى تضمين جميع العناصر العامة من `chip8_core`.

```rust
use chip8_core::*;
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub struct EmuWasm {
    chip8: Emu,
}
```

لاحظ العلامة `#[wasm_bindgen]`، والتي تخبر المترجم أن هذا الـ struct يحتاج إلى التهيئة لـ WebAssembly. أي دالة أو struct سيتم استدعاؤها من داخل JavaScript سيحتاج إلى وجودها. دعنا نحدد المُنشئ أيضًا.

```rust
use chip8_core::*;
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub struct EmuWasm {
    chip8: Emu,
}

#[wasm_bindgen]
impl EmuWasm {
    #[wasm_bindgen(constructor)]
    pub fn new() -> EmuWasm {
        EmuWasm {
            chip8: Emu::new(),
        }
    }
}
```

بسيط جدًا. أهم شيء يجب ملاحظته هو أن الدالة `new` تتطلب تضمين `constructor` الخاص حتى يعرف المترجم ما نحاول القيام به.

الآن لدينا struct يحتوي على كائن محاكاة `chip8` الأساسي. هنا، سننفذ نفس الطرق التي احتجناها في الواجهة الأمامية لـ `desktop`، مثل تمرير ضغطات/إفلاتات المفاتيح إلى النواة، تحميل ملف، والتنفيذ. لنبدأ بتنفيذ CPU والموقتات، حيث أنها الأسهل.

```rust
#[wasm_bindgen]
impl EmuWasm {
    // -- الكود غير المتغير محذوف --

    #[wasm_bindgen]
    pub fn tick(&mut self) {
        self.chip8.tick();
    }

    #[wasm_bindgen]
    pub fn tick_timers(&mut self) {
        self.chip8.tick_timers();
    }
}
```

هذا كل شيء، هذه مجرد أغلفة رقيقة لاستدعاء الدوال المقابلة في `chip8_core`. هذه الدوال لا تأخذ أي مدخلات، لذا لا يوجد شيء معقد عليها سوى التنفيذ.

تذكر دالة `reset` التي أنشأناها في `chip8_core`، ولكننا لم نستخدمها أبدًا؟ حسنًا، سنستخدمها الآن. ستكون هذه مجرد غلاف مثل الدوال السابقة.

```rust
#[wasm_bindgen]
pub fn reset(&mut self) {
    self.chip8.reset();
}
```

ضغط المفاتيح هو أول هذه الدوال التي ستنحرف عما تم في `desktop`. يعمل هذا بطريقة مشابهة لما فعلناه في `desktop`، ولكن بدلًا من أخذ ضغطة مفتاح SDL، سنحتاج إلى قبول واحدة من JavaScript. لقد وعدت بإضافة بعض ميزات `web-sys`، لذا لنفعل ذلك الآن. عد إلى `wasm/Cargo.toml` وأضف ميزة `KeyboardEvent`.

```toml
[dependencies.web-sys]
version = "^0.3.46"
features = [
    "KeyboardEvent"
]
```

```rust
use web_sys::KeyboardEvent;

// -- الكود غير المتغير محذوف --

impl EmuWasm {
    // -- الكود غير المتغير محذوف --

    #[wasm_bindgen]
    pub fn keypress(&mut self, evt: KeyboardEvent, pressed: bool) {
        let key = evt.key();
        if let Some(k) = key2btn(&key) {
            self.chip8.keypress(k, pressed);
        }
    }
}

fn key2btn(key: &str) -> Option<usize> {
    match key {
        "1" => Some(0x1),
        "2" => Some(0x2),
        "3" => Some(0x3),
        "4" => Some(0xC),
        "q" => Some(0x4),
        "w" => Some(0x5),
        "e" => Some(0x6),
        "r" => Some(0xD),
        "a" => Some(0x7),
        "s" => Some(0x8),
        "d" => Some(0x9),
        "f" => Some(0xE),
        "z" => Some(0xA),
        "x" => Some(0x0),
        "c" => Some(0xB),
        "v" => Some(0xF),
        _ =>   None,
    }
}
```

هذا مشابه جدًا لتنفيذنا في `desktop`، إلا أننا سنأخذ حدث `KeyboardEvent` من JavaScript، والذي سيؤدي إلى سلسلة نصية لتحليلها. لاحظ أن سلاسل المفاتيح حساسة لحالة الأحرف، لذا حافظ على كل شيء بأحرف صغيرة إلا إذا كنت تريد من اللاعبين الضغط على Shift كثيرًا.

قصة مشابهة تنتظرنا عند تحميل لعبة، ستتبع نمطًا مشابهًا، إلا أننا سنحتاج إلى استقبال ومعالجة كائن JavaScript.

```rust
use js_sys::Uint8Array;
// -- الكود غير المتغير محذوف --

impl EmuWasm {
    // -- الكود غير المتغير محذوف --

    #[wasm_bindgen]
    pub fn load_game(&mut self, data: Uint8Array) {
        self.chip8.load(&data.to_vec());
    }
}
```

الشيء الوحيد المتبقي هو دالة الرسم الفعلية على الشاشة. سأقوم بإنشاء دالة فارغة هنا، ولكننا سنؤجل تنفيذها حاليًا، بدلًا من ذلك سنوجه انتباهنا مرة أخرى إلى صفحة الويب، ونبدأ العمل من الاتجاه الآخر.

```rust
impl EmuWasm {
    // -- الكود غير المتغير محذوف --

    #[wasm_bindgen]
    pub fn draw_screen(&mut self, scale: usize) {
        // TODO
    }
}
```

سنعود إلى هنا بمجرد إعداد JavaScript الخاص بنا ومعرفة كيفية الرسم بالضبط.

## إنشاء وظائف الواجهة الأمامية

حان الوقت للتعمق في JavaScript. أولاً، دعنا نضيف بعض العناصر الإضافية إلى صفحة الويب البسيطة جدًا. عندما أنشأنا المحاكي للتشغيل على جهاز كمبيوتر، استخدمنا SDL لإنشاء نافذة للرسم عليها. بالنسبة لصفحة الويب، سنستخدم عنصرًا يوفره لنا HTML5 يسمى *canvas*. سنقوم أيضًا بالإشارة إلى صفحة الويب الخاصة بنا إلى النص البرمجي JS (غير الموجود حاليًا).

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Chip-8 Emulator</title>
        <meta charset="utf-8">
    </head>
    <body>
        <h1>My Chip-8 Emulator</h1>
        <label for="fileinput">Upload a Chip-8 game: </label>
        <input type="file" id="fileinput" autocomplete="off"/>
        <br/>
        <canvas id="canvas">If you see this message, then your browser doesn't support HTML5</canvas>
    </body>
    <script type="module" src="index.js"></script>
</html>
```

أضفنا هنا ثلاثة أشياء، أولاً زرًا يسمح للمستخدمين باختيار لعبة Chip-8 لتشغيلها عند النقر عليه. ثانيًا، عنصر `canvas`، والذي يتضمن رسالة قصيرة لأي مستخدمين غير محظوظين لديهم متصفح قديم. أخيرًا أخبرنا صفحة الويب بتحميل النص البرمجي `index.js` الذي سنقوم بإنشائه. لاحظ أنه في وقت كتابة هذا الدليل، من أجل تحميل ملف .wasm عبر JavaScript، تحتاج إلى تحديد أنه من نوع `module`.

الآن، لنقم بإنشاء `index.js` ونحدد بعض العناصر التي سنحتاجها. أولاً، نحتاج إلى إخبار JavaScript بتحميل وظائف WebAssembly. الآن، لن نقوم بتحميلها مباشرة هنا. عندما نستخدم `wasm-pack` للتحويل، سيتم إنشاء ليس فقط ملف .wasm الخاص بنا، ولكن أيضًا "صمغ" JavaScript الذي سيلف كل دالة قمنا بتعريفها حول دالة JavaScript يمكننا استخدامها هنا.

```js
import init, * as wasm from "./wasm.js"
```

هذا يستورد جميع وظائفنا، بالإضافة إلى دالة خاصة `init` سيحتاج إلى استدعائها أولاً قبل أن نتمكن من استخدام أي شيء من `wasm`.

لنقم بتعريف بعض الثوابت وإجراء بعض الإعدادات الأساسية الآن.

```js
import init, * as wasm from "./wasm.js"

const WIDTH = 64
const HEIGHT = 32
const SCALE = 15
const TICKS_PER_FRAME = 10
let anim_frame = 0

const canvas = document.getElementById("canvas")
canvas.width = WIDTH * SCALE
canvas.height = HEIGHT * SCALE

const ctx = canvas.getContext("2d")
ctx.fillStyle = "black"
ctx.fillRect(0, 0, WIDTH * SCALE, HEIGHT * SCALE)

const input = document.getElementById("fileinput")
```

كل هذا سيبدو مألوفًا من بناء `desktop` الخاص بنا. نحن نحصل على لوحة HTML ونضبط حجمها إلى أبعاد شاشة Chip-8 الخاصة بنا، بالإضافة إلى تكبيرها قليلاً (لا تتردد في تعديل هذا وفقًا لتفضيلاتك).

لنقم بإنشاء وظيفة رئيسية `run` والتي ستحمّل كائن `EmuWasm` الخاص بنا وتتعامل مع المحاكاة الرئيسية.

```js
async function run() {
    await init()
    let chip8 = new wasm.EmuWasm()

    document.addEventListener("keydown", function(evt) {
        chip8.keypress(evt, true)
    })

    document.addEventListener("keyup", function(evt) {
        chip8.keypress(evt, false)
    })

    input.addEventListener("change", function(evt) {
        // التعامل مع تحميل الملف
    }, false)
}

run().catch(console.error)
```

هنا، قمنا باستدعاء الوظيفة الإلزامية `init` التي تخبر متصفحنا بتهيئة ثنائي WebAssembly قبل أن نستخدمه. ثم نقوم بإنشاء محاكي الخلفية الخاص بنا عن طريق إنشاء كائن جديد `EmuWasm`.

سنقوم الآن بالتعامل مع تحميل ملف عند الضغط على الزر.

```js
input.addEventListener("change", function(evt) {
    // إيقاف عرض اللعبة السابقة، إذا كانت موجودة
    if (anim_frame != 0) {
        window.cancelAnimationFrame(anim_frame)
    }

    let file = evt.target.files[0]
    if (!file) {
        alert("فشل في قراءة الملف")
        return
    }

    // تحميل اللعبة كـ Uint8Array، إرسالها إلى .wasm، بدء الحلقة الرئيسية
    let fr = new FileReader()
    fr.onload = function(e) {
        let buffer = fr.result
        const rom = new Uint8Array(buffer)
        chip8.reset()
        chip8.load_game(rom)
        mainloop(chip8)
    }
    fr.readAsArrayBuffer(file)
}, false)

function mainloop(chip8) {
}
```

تضيف هذه الوظيفة مستمع حدث إلى زر `input` الخاص بنا والذي يتم تشغيله عند النقر عليه. استخدم واجهة `desktop` الأمامية SDL لإدارة الرسم في النافذة، وكذلك للتأكد من أننا نعمل بسرعة 60 إطارًا في الثانية. الميزة المماثلة للوحات هي "إطارات الرسوم المتحركة". في أي وقت نريد عرض شيء ما على اللوحة، نطلب من النافذة تحريك إطار، وسوف تنتظر حتى ينقضي الوقت الصحيح لضمان أداء 60 إطارًا في الثانية. سنرى كيف يعمل هذا في لحظة، ولكن الآن، نحتاج إلى إخبار برنامجنا أنه إذا كنا نحمّل لعبة جديدة، نحتاج إلى إيقاف الرسوم المتحركة السابقة. سنقوم أيضًا بإعادة تعيين المحاكي قبل تحميل ROM، للتأكد من أن كل شيء كما بدأ، دون الحاجة إلى إعادة تحميل صفحة الويب.

بعد ذلك، ننظر إلى الملف الذي أشار إليه المستخدم. لا نحتاج إلى التحقق مما إذا كان برنامج Chip-8 فعليًا، ولكننا نحتاج إلى التأكد من أنه ملف من نوع ما. ثم نقوم بقراءته وتمريره إلى الخلفية عبر كائن `EmuWasm` الخاص بنا. بمجرد تحميل اللعبة، يمكننا القفز إلى حلقة المحاكاة الرئيسية!

```js
function mainloop(chip8) {
    // الرسم فقط كل بضع دورات
    for (let i = 0; i < TICKS_PER_FRAME; i++) {
        chip8.tick()
    }
    chip8.tick_timers()

    // مسح اللوحة قبل الرسم
    ctx.fillStyle = "black"
    ctx.fillRect(0, 0, WIDTH * SCALE, HEIGHT * SCALE)
    // إعادة تعيين لون الرسم إلى الأبيض قبل عرض الإطار
    ctx.fillStyle = "white"
    chip8.draw_screen(SCALE)

    anim_frame = window.requestAnimationFrame(() => {
        mainloop(chip8)
    })
}
```

يجب أن يبدو هذا مشابهًا جدًا لما فعلناه لواجهة `desktop` الأمامية. نقوم بالتنفيذ عدة مرات قبل مسح اللوحة وإخبار كائن `EmuWasm` الخاص بنا برسم الإطار الحالي على اللوحة. هنا نخبر النافذة أننا نرغب في عرض إطار، ونحتفظ بمعرفها إذا احتجنا إلى إلغائه أعلاه. سوف ينتظر `requestAnimationFrame` لضمان أداء 60 إطارًا في الثانية، ثم يعيد تشغيل `mainloop` عندما يحين الوقت، ويبدأ العملية من جديد.

## تجميع ثنائي WebAssembly الخاص بنا

قبل أن نذهب أبعد من ذلك، دعنا نحاول بناء كود Rust الخاص بنا ونتأكد من أنه يمكن تحميله بواسطة صفحة الويب دون مشاكل. سيتعامل `wasm-pack` مع تجميع ثنائي .wasm، ولكننا نحتاج أيضًا إلى تحديد أننا لا نرغب في استخدام أي أنظمة حزم ويب مثل `npm`. للبناء، قم بتغيير الدلائل إلى مجلد `wasm` وقم بتشغيل:

```
$ wasm-pack build --target web
```

بمجرد اكتماله، سيتم بناء الأهداف في دليل جديد `pkg`. هناك عدة عناصر هنا، ولكن الوحيدة التي نحتاجها هي `wasm_bg.wasm` و `wasm.js`. `wasm_bg.wasm` هو مزيج من حزمتي `wasm` و `chip8_core` Rust المترجمة في واحدة، و `wasm.js` هو "الغراء" JavaScript الذي أدرجناه سابقًا. إنه في الغالب أغلفة حول API الذي حددناه في `wasm` بالإضافة إلى بعض كود التهيئة. إنه في الواقع قابل للقراءة إلى حد ما، لذا فإن الأمر يستحق النظر إلى ما يفعله.

تشغيل الصفحة في خادم ويب محلي يجب أن يسمح لك باختيار وتحميل لعبة دون ظهور أي تحذيرات في وحدة تحكم المتصفح. ومع ذلك، لم نكتب وظيفة عرض الشاشة بعد، لذا دعنا ننهي ذلك حتى نتمكن من رؤية لعبتنا تعمل بالفعل.

## الرسم على اللوحة

هذه هي الخطوة الأخيرة، العرض على الشاشة. لقد أنشأنا وظيفة `draw_screen` فارغة في كائن `EmuWasm` الخاص بنا، ونستدعيها في الوقت المناسب، ولكنها حاليًا لا تفعل أي شيء. الآن، هناك طريقتان يمكننا التعامل مع هذا. يمكننا إما تمرير إطار العرض إلى JavaScript وعرضه، أو يمكننا الحصول على لوحتنا في ثنائي `EmuWasm` الخاص بنا وعرضها في Rust. أي من الطريقتين ستكون جيدة، ولكن شخصيًا وجدت أن التعامل مع العرض في Rust أسهل.

لقد استخدمنا الحزمة `web_sys` للتعامل مع أحداث `KeyboardEvent` في JavaScript في Rust، ولكن لديها وظائف لإدارة العديد من عناصر JavaScript الأخرى. مرة أخرى، تلك التي نرغب في استخدامها تحتاج إلى تعريفها كسمات في `wasm/Cargo.toml`.

```toml
[dependencies.web-sys]
version = "^0.3.46"
features = [
    "CanvasRenderingContext2d",
    "Document",
    "Element",
    "HtmlCanvasElement",
    "ImageData",
    "KeyboardEvent",
    "Window"
]
```

هذه نظرة عامة على خطواتنا التالية. من أجل العرض على لوحة HTML5، تحتاج إلى الحصول على كائن اللوحة و*سياقها* وهو الكائن الذي يتم استدعاء وظائف الرسم عليه. نظرًا لأن ثنائي WebAssembly الخاص بنا تم تحميله بواسطة صفحة الويب الخاصة بنا، فإنه يمكنه الوصول إلى جميع عناصرها تمامًا كما يفعل نص JS. سنقوم بتغيير المُنشئ `new` للحصول على النافذة الحالية، اللوحة، والسياق كما تفعل في JavaScript.

```rust
use wasm_bindgen::JsCast;
use web_sys::{CanvasRenderingContext2d, HtmlCanvasElement, KeyboardEvent};

#[wasm_bindgen]
pub struct EmuWasm {
    chip8: Emu,
    ctx: CanvasRenderingContext2d,
}

#[wasm_bindgen]
impl EmuWasm {
    #[wasm_bindgen(constructor)]
    pub fn new() -> Result<EmuWasm, JsValue> {
        let chip8 = Emu::new();

        let document = web_sys::window().unwrap().document().unwrap();
        let canvas = document.get_element_by_id("canvas").unwrap();
        let canvas: HtmlCanvasElement = canvas
            .dyn_into::<HtmlCanvasElement>()
            .map_err(|_| ())
            .unwrap();

        let ctx = canvas.get_context("2d")
            .unwrap().unwrap()
            .dyn_into::<CanvasRenderingContext2d>()
            .unwrap();

        Ok(EmuWasm{chip8, ctx})
    }

    // -- كود غير متغير محذوف --
}
```

يجب أن يبدو هذا مألوفًا لأولئك الذين قاموا ببرمجة JavaScript من قبل. نحن نحصل على لوحة النافذة الحالية ونحصل على سياقها ثنائي الأبعاد، والذي يتم حفظه كمتغير عضو في بنية `EmuWasm` الخاصة بنا. الآن بعد أن أصبح لدينا سياق فعلي للرسم عليه، يمكننا تحديث وظيفة `draw_screen` للرسم عليه.

```rust
#[wasm_bindgen]
pub fn draw_screen(&mut self, scale: usize) {
    let disp = self.chip8.get_display();
    for i in 0..(SCREEN_WIDTH * SCREEN_HEIGHT) {
        if disp[i] {
            let x = i % SCREEN_WIDTH;
            let y = i / SCREEN_WIDTH;
            self.ctx.fill_rect(
                (x * scale) as f64,
                (y * scale) as f64,
                scale as f64,
                scale as f64
            );
        }
    }
}
```

نحصل على مخزن عرض من `chip8_core` الخاص بنا ونكرر عبر كل بكسل. إذا تم تعيينه، نرسمه مكبرًا إلى القيمة التي تم تمريرها من واجهتنا الأمامية. لا تنس أننا قمنا بالفعل بمسح اللوحة إلى الأسود وتعيين لون الرسم إلى الأبيض قبل استدعاء `draw_screen`، لذا لا يحتاج إلى القيام بذلك هنا.

هذا كل شيء! التنفيذ انتهى. كل ما تبقى هو بناؤه وتجربته بأنفسنا.

أعد البناء عن طريق الانتقال إلى دليل `wasm` وتشغيل:

```
$ wasm-pack build --target web
$ mv pkg/wasm_bg.wasm ../web
$ mv pkg/wasm.js ../web
```

الآن ابدأ خادم الويب الخاص بك واختر لعبة. إذا سار كل شيء على ما يرام، يجب أن تكون قادرًا على لعب ألعاب Chip-8 في المتصفح بنفس جودة سطح المكتب!

\newpage
