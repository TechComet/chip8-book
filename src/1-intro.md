# مقدمة إلى محاكاة ألعاب الفيديو باستخدام Rust

تطوير محاكي لألعاب الفيديو أصبح مشروعًا هواية شائعًا بشكل متزايد بين المطورين. يتطلب هذا المشروع معرفة بالأجهزة منخفضة المستوى، ولغات البرمجة الحديثة، وأنظمة الرسومات لإنشائه بنجاح. يعتبر هذا المشروع تعليميًا ممتازًا؛ ليس فقط لأنه يحتوي على أهداف واضحة، ولكنه أيضًا مجزٍ للغاية عندما تتمكن من تشغيل الألعاب على محاكي قمت بكتابته بنفسك. أنا ما زلت مطور محاكاة جديد نسبيًا، لكنني لم أكن لأصل إلى ما أنا عليه الآن لولا الأدلة والبرامج التعليمية الرائعة المتاحة على الإنترنت. لذلك، أردت أن أرد الجميل للمجتمع من خلال كتابة دليل يحتوي على بعض الحيل التي تعلمتها، على أمل أن يكون مفيدًا لشخص آخر.

## مقدمة عن Chip-8

نظامنا المستهدف هو [Chip-8](https://en.wikipedia.org/wiki/CHIP-8). أصبح Chip-8 بمثابة "Hello World" لتطوير المحاكاة. بينما قد تكون مغريًا للبدء بشيء أكثر إثارة مثل NES أو Game Boy، إلا أن هذه الأنظمة أكثر تعقيدًا من Chip-8. يحتوي Chip-8 على شاشة أحادية اللون بدقة 1 بت، وصوت بسيط أحادي القناة، و35 تعليمة فقط (مقارنة بحوالي 500 تعليمة في Game Boy)، ولكن سنتحدث عن ذلك لاحقًا. سيغطي هذا الدليل التفاصيل التقنية لـ Chip-8، وما هي أنظمة الأجهزة التي تحتاج إلى محاكاتها وكيفية ذلك، وكيفية التفاعل مع المستخدم. سيركز هذا الدليل على مواصفات Chip-8 الأصلية، ولن يتم تنفيذ أي من الامتدادات العديدة التي تم اقتراحها، مثل Super Chip-8 أو Chip-16 أو XO-Chip؛ حيث تم إنشاؤها بشكل مستقل عن بعضها البعض، وبالتالي تضيف ميزات متناقضة.

## المواصفات التقنية لـ Chip-8

- شاشة أحادية اللون بدقة 64x32، يتم الرسم عليها عبر أشكال (sprites) بعرض 8 بكسل وارتفاع يتراوح بين 1 و16 بكسل.

- ستة عشر سجلًا عامًا بعرض 8 بت، يُشار إليها بـ V0 حتى VF. يعمل VF أيضًا كسجل علم (flag register) لعمليات الفائض (overflow).

- عداد برنامج (program counter) بعرض 16 بت.

- سجل واحد بعرض 16 بت يُستخدم كمؤشر للوصول إلى الذاكرة، يُسمى *سجل I*.

- كمية غير موحدة من الذاكرة العشوائية (RAM)، ولكن معظم المحاكيات تخصص 4 كيلوبايت.

- مكدس (stack) بعرض 16 بت يُستخدم لاستدعاء العودة من الإجراءات الفرعية (subroutines).

- إدخال لوحة مفاتيح مكونة من 16 مفتاحًا.

- سجلين خاصين ينخفضان كل إطار ويتم تشغيلهما عند الوصول إلى الصفر:
    - مؤخر الوقت (Delay timer): يُستخدم للأحداث الزمنية في اللعبة.
    - مؤخر الصوت (Sound timer): يُستخدم لتشغيل صوت التنبيه.

## مقدمة عن Rust

يمكن كتابة المحاكيات بلغات برمجة عديدة. يستخدم هذا الدليل لغة البرمجة [Rust](https://www.rust-lang.org/)، على الرغم من أن الخطوات الموضحة هنا يمكن تطبيقها بأي لغة. تقدم Rust العديد من المزايا الرائعة؛ فهي لغة مكتوبة (compiled) تدعم منصات رئيسية ولديها مجتمع نشط من المكتبات الخارجية التي يمكن استخدامها في مشروعنا. تدعم Rust أيضًا التجميع لـ [WebAssembly](https://en.wikipedia.org/wiki/WebAssembly)، مما يسمح لنا بإعادة تجميع الكود للعمل في المتصفح مع تعديلات بسيطة. يفترض هذا الدليل أنك تفهم أساسيات لغة Rust والبرمجة بشكل عام. سأشرح الكود أثناء تقدمنا، ولكن نظرًا لأن Rust تتمتع بمنحنى تعليمي مرتفع، أوصي بقراءة والرجوع إلى [الكتاب الرسمي لـ Rust](https://doc.rust-lang.org/stable/book/title-page.html) لأي مفاهيم غير مألوفة أثناء تقدم الدليل. يفترض هذا الدليل أيضًا أنك قمت بتثبيت Rust وأنها تعمل بشكل صحيح. يرجى الرجوع إلى [تعليمات التثبيت](https://www.rust-lang.org/tools/install) لمنصتك إذا لزم الأمر.

## ما ستحتاج إليه

قبل أن تبدأ، يرجى التأكد من أن لديك أو قمت بتثبيت العناصر التالية.

### محرر نصوص

يمكن استخدام أي محرر نصوص للمشروع، ولكن هناك محرران أوصي بهما حيث يقدمان ميزات لـ Rust مثل تمييز الصيغة (syntax highlighting)، اقتراحات الكود، ودعم المصحح (debugger).

- [Visual Studio Code](https://code.visualstudio.com/) هو المحرر الذي أفضله لـ Rust، بالاشتراك مع إضافة [rust-analyzer](https://rust-analyzer.github.io/).

- بينما لا تقدم JetBrains بيئة تطوير متكاملة (IDE) مخصصة لـ Rust، إلا أن هناك إضافة لـ Rust للعديد من منتجاتها الأخرى. توفر [الإضافة](https://intellij-rust.github.io/) لـ [CLion](https://www.jetbrains.com/clion/) ميزات إضافية مثل دعم المصحح المدمج. ضع في اعتبارك أن CLion منتج مدفوع، على الرغم من أنه يقدم نسخة تجريبية لمدة 30 يومًا وفترات مجانية ممتدة للطلاب.

إذا كنت لا تفضل أيًا من هذه الخيارات، تتوفر إضافات للصيغة والاكتمال التلقائي لـ Rust للعديد من المحررات الأخرى، ويمكن تصحيح الأخطاء بسهولة باستخدام العديد من المصححات الأخرى مثل gdb.

### ROMs للاختبار

المحاكي ليس مفيدًا إذا لم يكن لديك شيء لتشغيله! تم تضمين العديد من برامج Chip-8 الشائعة مع الكود المصدري لهذا الكتاب، ويمكن أيضًا العثور عليها [هنا](https://www.zophar.net/pdroms/chip8/chip-8-games-pack.html). سيتم عرض بعض هذه الألعاب كأمثلة خلال هذا الدليل.

### أشياء أخرى

عناصر أخرى قد تكون مفيدة أثناء تقدمنا:

- يرجى تحديث معرفتك بـ [النظام الست عشري (hexadecimal)](https://en.wikipedia.org/wiki/Hexadecimal) إذا كنت لا تشعر بالراحة مع هذا المفهوم. سيتم استخدامه بشكل مكثف خلال هذا المشروع.

- ألعاب Chip-8 تكون بتنسيق ثنائي (binary)، وغالبًا ما يكون من المفيد أن تكون قادرًا على عرض القيم الست عشرية الخام أثناء تصحيح الأخطاء. عادةً لا تدعم محررات النصوص القياسية عرض الملفات بالنظام الست عشري، بل يتطلب ذلك محرر ست عشري متخصص [hex editor](https://en.wikipedia.org/wiki/Comparison_of_hex_editors). العديد منها يقدم ميزات مشابهة، لكني أفضل شخصيًا [Reverse Engineer's Hex Editor](https://github.com/solemnwarning/rehex).

لنبدأ!

\newpage
