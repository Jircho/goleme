# Goleme — marketingová landing page

## Co to je

Goleme je AI nástroj, který kadeřnickým salonům a barbershopům generuje hotové, bookovatelné weby ("popiš svůj salon → dostaneš web"). Tohle je jeho vlastní marketingová stránka (landing page).

Vznikla jako **export z Figmy** a od té doby se ručně rozšiřuje přímo v kódu — Figma soubor už neodpovídá aktuálnímu stavu, zdroj pravdy je `index.html`.

## Stack a architektura

- **Jeden soubor**: `index.html` (~3900 řádků) — obsahuje kompletně vše: `<style>` blok v `<head>`, HTML markup, a několik `<script>` bloků před `</body>`. Žádné komponenty, žádné importy.
- **Bez build procesu.** Žádný npm, webpack, bundler. Otevřít / servírovat lze rovnou.
- **Žádný framework.** Vanilla HTML/CSS/JS (JS psaný jako IIFE moduly, ES5-ish styl, žádný React/Vue).
- **Externí závislosti**: Google Fonts (Inter, Playfair Display), Font Awesome 6.4.0 z cdnjs. Font 'ITC Stone Sans II' a 'PerpetualMTPro' jsou lokální systémové fonty (`local()` fallback), takže na jiném stroji než autorově se mohou renderovat jinak.
- **Dev workflow**: soubor se testuje přes **VS Code Live Server** (port 5500, `127.0.0.1:5500`), live-reload při uložení. Žádný jiný dev server není potřeba.

## Assety

- `logo.png` — logo Goleme (rastrové, používá se v navu)
- `hair-clip.png` — dekorativní "hair clip" grafika, používá se na dvou místech: rohy hero pravého panelu (`.grid-clip`) a jako "špendlík" na testimonial kartách (`.tcard-pin`). Historicky se obrázek měnil (portrait i landscape verze) — CSS proto všude konstruuje rozměr přes `height`, ne `width`, aby to bylo robustní vůči budoucí výměně assetu.
- `barber_hero.png` — nepoužívá se aktivně v hero (hero pravý panel je čistě CSS/HTML mockup chatu, ne obrázek).
- `logo-svg.svg` — vektorová verze loga "Goleme" (5 pathů, `fill="none" stroke="black" stroke-width="5"`, viewBox `0 0 1693 471`). Přidal uživatel, má nahradit textový watermark ve footeru. **Neupravovat tento soubor** — je to zdrojový asset uživatele.
- `logo-mask.svg` — odvozená kopie výše se `stroke="white"` místo `"black"`, vytvořená speciálně pro použití jako CSS `mask-image` (viz "Neuzavřený problém" níže). Pokud se logo v budoucnu změní, je potřeba tuto kopii ručně přegenerovat se stejnou úpravou (barva obrysu → bílá).

## Design tokeny (CSS custom properties v `:root`)

```
--bg: #FFFFFF
--ink: #1D1C1B          /* hlavní tmavá barva textu */
--yellow: #fcd032       /* brand žlutá */
--yellow-dark: #D4A910
--ink-80/60/30/10/06     /* ink s různou průhledností pro hierarchii textu */
```

Brand CTA gradient (používaný konzistentně na tlačítkách, prompt kartě, footer watermarku): `linear-gradient(90deg, #FFB800, #FF5E00)`.

## Struktura stránky (sekce v pořadí)

1. **Nav** (`.nav`) — sticky, zmenšuje se a získává glass-blur po scrollu (`.is-scrolled`), sliding pill highlight za hoverovaným odkazem, "magnetická" CTA s shine efektem, scissors easter egg na hoveru loga. Na mobilu (`<900px`) se `.nav__links` + `.nav__actions` sbalí do `.nav__panel` (wrapper s `display:contents` na desktopu → na mobilu se stává dropdown menu ovládaným hamburgerem).
2. **Hero** (`.hero`) — dvousloupcový: vlevo nadpis/subtext/prompt input s animovaným rotujícím placeholderem (typewriter efekt, texty v poli `PROMPTS` v JS), vpravo mockup chatu s Golemem + plovoucí dekorativní ikony a hair-clip rohy.
3. **Logos strip** (`.logos`) — "Trusted by" logořádek.
4. **Walkthrough / Features** (`.features-section`, prefix `wt-`) — animovaná 3fázová ukázka (chat → skeleton loading → hotový web) řízená `requestAnimationFrame` smyčkou v JS (ne CSS animací), s SVG cestou a tečkou co po ní jede. Vlevo klikatelné kroky, vpravo "obrazovka" s `container-type: inline-size` (má vlastní `@container` queries pro svůj vnitřní obsah, nezávisle na viewport šířce).
5. **Showcase mosaic** (`.showcase__frame`, `.mosaic`) — 3 řady dlaždic (`.tile`), každá autoscrolluje nekonečnou smyčkou (duplikovaný obsah, `translateX(0)→translateX(-50%)`), střídavý směr po řádcích, hover na dlaždici zvětší ji a ostatní ztlumí/odsytí (`:has()` selektor).
6. **Testimonials "nástěnka"** (`.testimonials__grid`, prefix `tcard`) — CSS multi-column layout (ne grid/flex!) stylizovaný jako připíchnuté papírky s hair-clip "špendlíkem" a velkým dekorativním uvozovkovým znakem na pozadí každé karty.
7. **Footer** (`.footer`) — velký watermark na pozadí s cursor-tracked gradient reveal efektem (JS posouvá `--mx`/`--my` custom properties na `#footerHugeText` podle pozice myši, CSS to používá v `mask-image: radial-gradient(...)`). Watermark byl původně text "Goleme" (`-webkit-text-stroke` + `background-clip:text`), teď se přepisuje na vektorové logo (`logo-svg.svg`) se stejným efektem — **aktuálně nefunkční, viz "Neuzavřený problém" níže.** Dál v footeru jsou sloupce s odkazy/kontaktem/newsletterem.

## Responzivita

Desktop layout je "zdrojová pravda" — mobilní úpravy jsou výhradně v `@media (max-width: 900px)` a `@media (max-width: 480px)` blocích na konci `<style>`, aby se nikdy neovlivnil vzhled na desktopu. Klíčové mobilní vzory:
- Watermark ve footeru má velikost přes `vw` jednotky (ne pevné px), aby se vždy vešel bez ořezu na libovolné šířce.
- Hero pravý panel má na mobilu `height: auto` (ne pevnou výšku) — obsah (chat bubliny) je proměnlivě dlouhý a pevná výška ho ořezávala.

## Známé CSS pasti (ověřené debugováním, neobjevovat znovu)

1. **`column-fill: balance` (default) + `height: auto`** na `column-count` layoutu (testimonials grid) v Chromu nandá všechen obsah do prvního sloupce místo rozlití do dalších. Řešení: `column-fill: auto` + explicitní `height`.
2. **`overflow-x: hidden` na `html`/`body`** (nebo jakémkoliv předkovi) rozbíjí `position: sticky` kdekoliv v potomcích. Pokud se řeší horizontální přetečení, hledat cílenější řešení než globální `overflow-x:hidden`.
3. **Záporně posazené absolutně pozicované elementy** (např. `top: -8px`) uvnitř multi-column kontextu se můžou tiše oříznout fragmentation hranicí sloupce pro karty, co nejsou první ve sloupci. Držet dekorativní prvky uvnitř vlastního boxu (kladný offset), ne přetékající ven.
4. **CSS specificita**: kde JS/CSS potřebuje garantovaně přebít obecnější selektor (např. `.nav__links li` vs. `.nav__pill`), řešeno explicitně vyšší specificitou (`.nav__links li.nav__pill`), ne pořadím v souboru.

## Neuzavřený problém: footer logo hover efekt (START HERE)

**Stav: nefunguje, dva pokusy o opravu selhaly.** Cíl: nahradit textový watermark "Goleme" ve footeru za `logo-svg.svg`, se zachováním existujícího cursor-tracked gradient reveal efektu (viditelná gradientová verze loga jen v okruhu ~170px kolem myši, zbytek jako jemný obrys). Aktuální výsledek: **není vidět vůbec nic** — ani statická obrysová vrstva, ani hover efekt.

### Zapojené elementy (aktuální stav v `index.html`)

HTML (uvnitř `<div class="footer__huge-text" id="footerHugeText">`):
```html
<span class="footer__huge-logo footer__huge-logo--base" role="img" aria-label="Goleme"></span>
<span class="footer__huge-logo-mask" aria-hidden="true">
  <span class="footer__huge-logo-spotlight"></span>
</span>
```

CSS (hledej `.footer__huge-logo` v `<style>`):
- `.footer__huge-text` — wrapper, `position:absolute; left:60px; bottom:20px;`, žádné `right` (shrink-to-fit podle dítěte), drží `--mx`/`--my` custom properties.
- `.footer__huge-logo` — sdílená třída pro rozměry (`width: clamp(220px, 46vw, 820px); aspect-ratio: 1693/471;`) a masku (`mask-image: url('logo-mask.svg'); mask-size:contain; mask-mode:alpha;`).
- `.footer__huge-logo--base` — statická vrstva, `background: rgba(29,28,27,0.14)` maskovaná tvarem loga. **Tohle by mělo být vidět vždy, i bez hoveru — a není.**
- `.footer__huge-logo-mask` — druhý wrapper (stejná maska loga), `position:absolute; inset:0;`, uvnitř má spotlight dítě. Toggluje se `opacity` přes `.footer__huge-text.is-active`.
- `.footer__huge-logo-spotlight` — dítě uvnitř masky, `position:absolute; inset:0; background: radial-gradient(circle 170px at var(--mx) var(--my), ...)`.

JS (hledej `footerHugeText` v `<script>` blocích, sekce "FOOTER HOVER TEXT"): beze změny od verze s textem — `mousemove` na `.footer` počítá `(e.clientX - rect.left)/rect.width*100` atd. a nastavuje `--mx`/`--my` na `#footerHugeText`, plus togglue `.is-active`. Tahle část funguje spolehlivě (fungovala i s textovou verzí) a pravděpodobně **není** zdrojem bugu.

### Co bylo vyzkoušeno

1. **Pokus 1** — jedna vrstva s dvěma zkombinovanými maskami (`mask-image: url(logo.svg), radial-gradient(...)` + `mask-composite: intersect` / `-webkit-mask-composite: source-in`). Výsledek: základní vrstva neviditelná, gradientová vrstva se ukázala jako **neomezené kolečko bez ořezu do tvaru loga** — tzn. kombinace dvou masek (`mask-composite`) evidentně nefunguje spolehlivě s `url()` + `radial-gradient()` dohromady v prohlížeči, který uživatel používá.
2. **Pokus 2** — přepsáno na vnořenou strukturu s jednou maskou na elementu (`.footer__huge-logo-mask` maskuje sám sebe tvarem loga, `.footer__huge-logo-spotlight` je obyčejné dítě s pohyblivým gradientem uvnitř — žádný `mask-composite` není potřeba). Zároveň přidáno `mask-mode: alpha` s hypotézou, že prohlížeč bere `url()` SVG masku jako **luminance mask** (černá kresba = luminance 0 = "neviditelné" úplně všude, i tam kde je něco nakresleno). Pro jistotu vytvořena i `logo-mask.svg` (kopie s `stroke="white"` místo `"black"`) — bílá barva by měla fungovat správně bez ohledu na to, jestli prohlížeč použije luminance nebo alpha interpretaci. **Výsledek: pořád nic vidět, ani statická vrstva, ani hover.** Tzn. luminance/alpha teorie buď nebyla (jediná) příčina, nebo oprava nebyla aplikována správně.

### Nejpravděpodobnější podezřelý pro další session

`.footer__huge-logo-mask` má **zároveň** `inset: 0` (= `top:0; right:0; bottom:0; left:0`) **a** explicitní `width: clamp(...)` + `aspect-ratio: 1693/471`. To je CSS-přetížený (over-constrained) box: `position:absolute` element s nastaveným `left` i `right` zároveň s explicitní `width` je nejednoznačná kombinace a chování kolem `aspect-ratio` v takové situaci není napříč prohlížeči spolehlivě odladěné. **První věc k vyzkoušení**: zjednodušit na `position:absolute; top:0; left:0;` (bez `right`/`bottom`/`inset:0`) a nechat `width`+`aspect-ratio` řídit rozměr bez konkurence. Pokud se po tomhle objeví aspoň statická (base) vrstva, byl to hlavní viník.

### Další diagnostické kroky, pokud výše uvedené nepomůže

- Přes DevTools → Network ověřit, že `logo-mask.svg` (i `logo-svg.svg`) se opravdu načítá s HTTP 200, ne 404 (špatná cesta by v CSS mask-image spec měla znamenat "bez masky = celý box viditelný", což taky nesedí s pozorovaným "nic není vidět" — ale stojí za ověření).
- Izolovaný test: dát `mask-image: url('logo-mask.svg'); mask-size:contain; background:red;` na jednoduchý `<div style="width:400px;height:200px;">` mimo footer, ověřit jestli se vůbec něco maskuje. Pokud ani tohle nefunguje, problém je fundamentálnější (podpora `mask-image` s externím SVG v tomhle prostředí/prohlížeči) a je potřeba jiný přístup — např. inline `<svg>` s vlastním `<clipPath>` místo CSS `mask-image`, nebo `<img>` + `mix-blend-mode` technika místo maskování.
- Pokud CSS maskování obecně nejde spolehlivě rozchodit, záložní plán: vykreslit logo jako inline `<svg>` přímo v HTML (ne přes CSS mask), a gradientový hover efekt řešit přes SVG `<clipPath>`/`<mask>` nativně uvnitř SVG (má lepší a konzistentnější podporu napříč prohlížeči než CSS `mask-image` s externím souborem).

## Jazyk obsahu

Marketingový copy je primárně anglicky. Interaktivní walkthrough demo (fáze 1–3 vpravo v sekci "Jak to funguje") je záměrně česky, protože simuluje konkrétní use-case pro českého uživatele. Při úpravách textu zachovat toto rozdělení, pokud uživatel neřekne jinak.
