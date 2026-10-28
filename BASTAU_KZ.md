# Esep QR сайтын өз Firebase жобаңызда іске қосу

Бұл архивте сайттың толық бастапқы коды және алғашқы жариялауға дайын `dist` бумасы бар. Сайт ChatGPT жүйесіне тәуелсіз жұмыс істеуге бейімделген.

## 1. Архивте не бар?

- `dist/` — Firebase Hosting-ке жариялауға дайын сайт.
- `src/` — өзгертуге болатын React және TypeScript коды.
- `public/firebase-config.json` — өз Firebase жобаңыздың веб баптаулары.
- `firebase.json` — Hosting, Firestore және Storage конфигурациясы.
- `firestore.rules`, `storage.rules` — әр мұғалімге өз деректерін ғана ашатын ережелер.
- `scripts/setup.mjs` — Firebase баптауларын сұрап, файлдарға жазатын құрал.
- `.github/workflows/` — Firebase Hosting пен GitHub Pages үшін жариялау үлгілері.
- `tests/` — QR, балл шектері, жетіспейтін беттер және ЖИ жауабын тексеру тесттері.

## 2. Firebase Console ішінде жобаны дайындау

1. https://console.firebase.google.com сайтына Google есептік жазбаңызбен кіріңіз.
2. **Create a project / Add project** арқылы жоба жасаңыз. `Project ID` мәнін сақтап алыңыз.
3. **Project settings → General → Your apps → Web (`</>`)** арқылы веб қосымша тіркеңіз.
4. Firebase берген `firebaseConfig` объектісінен `apiKey`, `authDomain`, `projectId`, `storageBucket`, `messagingSenderId`, `appId` мәндерін алыңыз.
5. **Authentication → Sign-in method → Google** қызметін қосыңыз. Жобаның қолдау email мекенжайын таңдаңыз.
6. **Authentication → Settings → Authorized domains** ішінде `PROJECT_ID.web.app` доменін тексеріңіз. GitHub Pages қолдансаңыз, `GITHUB_LOGIN.github.io` доменін де қосыңыз.
7. **Firestore Database → Create database** арқылы `(default)` деректер қорын жасаңыз. Бастапқы қауіпсіз режимді таңдаңыз; дайын ережелер кейін CLI арқылы жарияланады.
8. **Storage → Get started** арқылы файл қоймасын жасаңыз. Console көрсеткен нақты bucket атауын алыңыз: жаңа жобада әдетте `PROJECT_ID.firebasestorage.app`, ескі жобада `PROJECT_ID.appspot.com` болуы мүмкін.

**Ескерілетін нақты шығын:** Cloud Storage for Firebase қолдану үшін Blaze жоспары қажет. ЖИ сұраулары мен ресурстарды пайдалану бөлек есептелуі мүмкін. Қызметтерді өз жобаңызда қосу және төлем жоспарын өзгерту осы архивті жүктеумен автоматты түрде орындалмайды.

Firebase веб конфигурациясы браузерде көрінетін баптау болып саналады. Оған Gemini құпия API кілтін немесе service-account JSON кілтін қоспаңыз. Деректерді `firestore.rules`, `storage.rules` және App Check қорғайды.

## 3. Ең оңай жариялау: Firebase Console → Cloud Shell

Firebase Console ішінен Cloud Shell ашуға болады. Ресми нұсқаулыққа сәйкес, Cloud Shell ішінде Firebase CLI алдын ала орнатылған. Оны өз компьютеріңізде қолдансаңыз, Node.js 22.13+ және Firebase CLI орнатыңыз:

```bash
npm install -g firebase-tools
```

1. Архивті Cloud Shell-ге жүктеп, ашыңыз. Командаларды `package.json` тұрған `esep-qr-firebase` бумасында орындаңыз.
2. Дайын `dist` бумасы архивте бар болғандықтан, алғашқы жариялау үшін жобаның npm тәуелділіктерін орнату міндетті емес.
3. Баптауды іске қосыңыз:

```bash
node scripts/setup.mjs
```

Бұл құрал 2-бөлімдегі Firebase Web config мәндерін сұрайды. Ол `public/firebase-config.json`, бар болса `dist/firebase-config.json` және `.firebaserc` файлдарын толтырады. ЖИ қызметін әлі баптамаған болсаңыз, тиісті сұраққа **жоқ** деп жауап беріңіз.

4. Firebase есептік жазбаңызға кіріп, жариялаңыз:

```bash
firebase login
firebase deploy --only hosting,firestore:rules,firestore:indexes,storage
```

5. Жұмыс суреттерін браузерде тек мұғалімнің рұқсатымен жүктеу үшін Storage CORS баптауын орындаңыз. `BUCKET_NAME` орнына өз қоймаңыздың нақты атауын жазыңыз:

```bash
gsutil cors set cors.json gs://BUCKET_NAME
```

`cors.json` файлындағы `origin: ["*"]` өзі деректерді жарияламайды. Файлға рұқсатты Storage Rules тексереді. Қосымшада ұзақ өмір сүретін ашық download-token сілтемелері жасалмайды.

Жариялау сәтті аяқталса, Firebase CLI нақты сайт URL-ын көрсетеді. Әдеттегі мекенжайы: `https://PROJECT_ID.web.app`.

## 4. Кодты өзгерту және қайта жариялау

```bash
npm install
npm test
npm run build
firebase deploy --only hosting,firestore:rules,firestore:indexes,storage
```

`npm run build` сайтты қайтадан `dist/` бумасына жинайды. Баптау файлын `public/firebase-config.json` ішінде өзгертіңіз: келесі жинақта ол `dist/` ішіне көшіріледі.

Жергілікті өңдеу:

```bash
npm run dev
```

Жергілікті кіру үшін Firebase Authentication ішіндегі Authorized domains тізіміне `localhost` қосу қажет болуы мүмкін. App Check үшін ресми debug provider нұсқаулығын қолданыңыз; production сайтында debug token қолданбаңыз.

## 5. GitHub-та сақтау

1. https://github.com/new арқылы өз есептік жазбаңызда бос репозиторий жасаңыз. Атауына, мысалы, `esep-qr-firebase` таңдауға болады.
2. **Add file → Upload files** арқылы архивтің ішіндегі файлдарды жүктеңіз. `package.json` репозиторийдің түбірінде тұрсын. `.github` бумасын да қосыңыз.
3. `node_modules` бумасын, service-account кілттерін, `.env` файлдарын және нақты оқушы жұмыстарын GitHub-қа жүктемеңіз.
4. Git арқылы сақтау ыңғайлы болса, өз репозиторий URL-ын қолданып орындаңыз:

```bash
git init
git add .
git commit -m "Add Esep QR Firebase app"
git branch -M main
git remote add origin https://github.com/YOUR_LOGIN/esep-qr-firebase.git
git push -u origin main
```

Бұл архив ешбір GitHub репозиторийіне автоматты түрде жіберілген жоқ. Сақтайтын репозиторийді өзіңіз таңдайсыз.

### GitHub-тан Firebase Hosting-ке автоматты жариялау

Ең жеңіл ресми тәсіл:

```bash
firebase init hosting:github
```

Firebase CLI репозиторийді сұрап, қажетті service account, GitHub secret және workflow файлдарын дайындайды. Build командасын сұраса, `npm install && npm run build` енгізіңіз. Hosting public directory ретінде `dist` қолданылады.

Архивтегі дайын `.github/workflows/firebase-hosting.yml` үлгісін қолдануды таңдасаңыз:

- GitHub **Settings → Secrets and variables → Actions → Variables** ішінде `FIREBASE_PROJECT_ID` мәнін енгізіңіз.
- Firebase Hosting-ке жариялауға рұқсаты бар service-account JSON-ды `FIREBASE_SERVICE_ACCOUNT` атты **Secret** ретінде тіркеңіз. JSON-ды кодқа қоспаңыз.
- `public/firebase-config.json` өз жобаңызға толтырылған болсын.
- Алғашқы Firestore/Storage ережелерін CLI арқылы жариялаңыз. Дайын workflow кейін тек Hosting-ті жаңартады.
- Одан кейін `main` тармағына push сайттың жаңа нұсқасын жариялайды.

CLI жасаған workflow мен берілген үлгінің бірін таңдаңыз. Екеуін бірдей баптасаңыз, қосарланған жариялау болуы мүмкін.

## 6. GitHub Pages нұсқасы

GitHub Pages сайт интерфейсін орналастыра алады. Мұғалімнің кіруі, деректер қоры, файлдар және ЖИ байланысы бәрібір өз Firebase жобаңыз арқылы жұмыс істейді.

1. Firebase жобаңызды және ережелеріңізді алдыңғы қадамдармен баптаңыз.
2. GitHub **Settings → Pages → Source → GitHub Actions** таңдаңыз.
3. **Actions → GitHub Pages (optional) → Run workflow** іске қосыңыз.
4. Workflow репозиторий атауына сәйкес `/REPOSITORY_NAME/` жолын өзі орнатады.
5. Firebase Authentication және App Check ішінде GitHub Pages доменін қосыңыз.

Берілген Pages workflow қарапайым project site үшін арналған. `YOUR_LOGIN.github.io` атты арнайы user-site репозиторийін қолдансаңыз, workflow ішіндегі `BASE_PATH` мәнін `/` етіп өзгертіңіз.

## 7. Қолжазбаны ЖИ арқылы оқуды қосу

Бұл кодта Firebase AI Logic арқылы Gemini-ге қосылатын нақты интеграция бар. Жоба бапталмайынша ЖИ батырмасы белсенді болмайды.

1. Өз Firebase Console жобаңызда **Firebase AI Logic** бөлімін ашыңыз.
2. **Gemini Developer API** провайдерін қосу қадамдарын орындаңыз. Gemini құпия кілтін сайттың браузер кодына енгізбеңіз.
3. **App Check** ішінде веб қосымшаңызды **reCAPTCHA Enterprise** арқылы тіркеңіз. Сайт доменін дұрыс көрсетіңіз.
4. Ашық reCAPTCHA site key мәнін `public/firebase-config.json` ішіндегі `appCheckSiteKey` жолына енгізіңіз.
5. Конфигурацияда `ai.enabled` мәнін `true` етіңіз. `ai.model` ішінде өз жобаңызда қолжетімді модельді көрсетіңіз. Берілген бастапқы мән — `gemini-3.8-flash`.
6. App Check метрикаларын тексеріп, AI Logic үшін enforcement қосыңыз. Өз жобаңызда сұрау квоталарын баптаңыз.
7. Сайтты қайта жинап, жариялаңыз.
8. **Нәтижелер → Тексеру → ЖИ-мен оқу** арқылы алдымен шартты, аты-жөнсіз жұмыс парағын сынаңыз.

ЖИ оқушының жазуын, формулаларын және шешу жолын талдап, әр критерийге балл жобасын ұсынады. Түсініксіз жауап үшін балл бос қалады. Ұсынысты қолдану жұмысты автоматты түрде бекітпейді. Мұғалім барлық критерийді және геометриялық дәлелдеуді қарап, қорытындыны өзі бекітеді.

## 8. Алғашқы сынақ

1. Google арқылы кіріңіз.
2. **Сынақ үлгілерін ашу** батырмасын басыңыз.
3. Бір үлгінің **QR-парақтар** батырмасын ашып, бір оқушының парағын PDF ретінде сақтаңыз немесе қағазға шығарыңыз.
4. Жауабын жазып, толық парақты түсіріңіз.
5. **Парақтарды қабылдау** ішінде фотосын не сканерленген PDF жүктеңіз.
6. QR оқушыны дұрыс таңдағанын тексеріп, сақтаңыз.
7. **Нәтижелер → Тексеру** ішінде әр критерийге балл қойыңыз немесе бапталған ЖИ ұсынысын қараңыз.
8. Балдарды бекітіп, **Excel үшін CSV** файлын жүктеңіз.

## Тексерілгені және шектеулері

- TypeScript және production жинағы тексерілді.
- QR-кодтың қалыпты және бұрылған суреттен оқылуы, толық емес жұмыстар, балл шектері, CSV және ЖИ жауабын валидациялау тесттері бар.
- Нақты Firebase жобаңыз, Google кіруі, Storage Rules және нақты ЖИ сұрауы осы ортада сыналған жоқ: өз жобаңыздың конфигурациясын енгізгеннен кейін 8-бөлімдегі сынақты орындау қажет.
- Басқарылатын браузер құралы болмағандықтан, толық визуалды браузер сынағы жасалмады.
- Оқушыға жеке кіру, электрондық журналмен синхрондау, төлем қабылдау және жазылым басқару бұл нұсқаға кірмейді.

## Ресми нұсқаулықтар

- https://firebase.google.com/docs/hosting/quickstart
- https://firebase.google.com/docs/hosting/github-integration
- https://firebase.google.com/docs/storage/web/start
- https://firebase.google.com/docs/storage/web/download-files
- https://firebase.google.com/docs/ai-logic/get-started
- https://firebase.google.com/docs/web/alt-setup
