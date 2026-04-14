---
title: Vaša prva komponenta
---

<Intro>

*Komponente* su jedan od glavnih koncepata u React-u. One predstavljaju osnovu pomoću koje pravite UI, što ih čini savršenim mestom za početak vaše React avanture!

</Intro>

<YouWillLearn>

* Šta je to komponenta
* Koju ulogu komponente igraju u React aplikaciji
* Kako da napišete vašu prvu React komponentu

</YouWillLearn>

## Komponente: Blokovi za pravljenje UI-a {/*components-ui-building-blocks*/}

HTML nam omogućava da na web-u kreiramo bogato struktuirane dokumente pomoću ugrađenog skupa tag-ova poput `<h1>` ili `<li>`:

```html
<article>
  <h1>Moja prva komponenta</h1>
  <ol>
    <li>Komponente: Blokovi za pravljenje UI-a</li>
    <li>Definisanje komponente</li>
    <li>Upotreba komponente</li>
  </ol>
</article>
```

Ovaj markup predstavlja ovaj članak `<article>`, njegov naslov `<h1>` i (skraćenu) tabelu sa sadržajem kao ordered list-u `<ol>`. Markup poput ovog, u kombinaciji sa CSS-om za stilizovanje i JavaScript-om za interaktivnost, stoji iza svakog sidebar-a, avatar-a, modal-a, dropdown-a, odnosno iza svakog dela UI-a kojeg vidite na web-u.

React vam omogućava da kombinujete markup, CSS i JavaScript u custom "komponente", **reusable UI elemente za vašu aplikaciju**. Kod za tabelu sadržaja koji ste videli gore može biti pretvoren u `<TableOfContents />` komponentu koju možete renderovati na svakoj stranici. Ispod haube, i dalje će se koristiti isti HTML tag-ovi poput `<article>`, `<h1>`, itd.

Kao i sa HTML tag-ovima, komponente možete sastavljati, praviti im redosled i ugnježdavati ih kako bi dizajnirali cele stranice. Na primer, stranica za dokumentaciju koju upravo čitate je napravljena pomoću React komponenata:

```js
<PageLayout>
  <NavigationHeader>
    <SearchBar />
    <Link to="/docs">Dokumentacija</Link>
  </NavigationHeader>
  <Sidebar />
  <PageContent>
    <TableOfContents />
    <DocumentationText />
  </PageContent>
</PageLayout>
```

Kako vaš projekat raste, uvidećete da dobar deo dizajna može biti sastavljen upotrebom komponenata koje ste već napisali. To može ubrzati vaš razvoj. Tabela sadržaja od gore može biti dodata bilo gde pomoću `<TableOfContents />`! Možete čak i započeti projekat velikom brzinom uz pomoć ogromnog broja komponenata dostupnih u React-ovoj open source zajednici poput [Chakra UI](https://chakra-ui.com/) i [Material UI](https://material-ui.com/).

## Definisanje komponente {/*defining-a-component*/}

Tradicionalno, developeri su tokom pravljenja web stranica prvo kreirali sadržaj, a nakon toga su ga obogatili interakcijom pomoću JavaScript-a. Ovo je radilo dobro dok je interakcija bila poželjna na web-u. Danas je ona očekivana na većini sajtova i u svim aplikacijama. React stavlja interakciju na prvo mesto, iako još uvek koristi istu tehnologiju: **React komponenta je JavaScript funkcija koju možete _obogatiti sa markup-om_**. Ovako to izgleda (možete menjati primer ispod):

<Sandpack>

```js
export default function Profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/MK3eW3Am.jpg"
      alt="Katherine Johnson"
    />
  )
}
```

```css
img { height: 200px; }
```

</Sandpack>

A ovako možete napraviti komponentu:

### Korak 1: Export-ovati komponentu {/*step-1-export-the-component*/}

`export default` prefiks je [standardna JavaScript sintaksa](https://developer.mozilla.org/docs/web/javascript/reference/statements/export) (nije specifično za React). Omogućava vam da obeležite glavnu funkciju u fajlu kako biste je mogli import-ovati u drugim fajlovima. (Više o import-ovanju u [Import-ovanje i export-ovanje komponenata](/learn/importing-and-exporting-components)!)

### Korak 2: Definisati funkciju {/*step-2-define-the-function*/}

Pomoću `function Profile() { }` definišete JavaScript funkciju čije je ime `Profile`.

<Pitfall>

React komponente su obične JavaScript funkcije, ali **njihova imena moraju da počnu sa velikim slovom** ili neće raditi!

</Pitfall>

### Korak 3: Dodati markup {/*step-3-add-markup*/}

Komponenta vraća `<img />` tag sa `src` i `alt` atributima. `<img />` je napisan kao HTML, ali je, ispod haube, zapravo JavaScript! Ova sintaksa se naziva [JSX](/learn/writing-markup-with-jsx) i omogućava vam da ugradite markup unutar JavaScript-a.

Return iskazi mogu biti napisani u jednoj liniji, kao u ovoj komponenti:

```js
return <img src="https://react.dev/images/docs/scientists/MK3eW3As.jpg" alt="Katherine Johnson" />;
```

Ako vaš markup nije u istoj liniji kao i ključna reč `return`, morate koristiti zagrade:

```js
return (
  <div>
    <img src="https://react.dev/images/docs/scientists/MK3eW3As.jpg" alt="Katherine Johnson" />
  </div>
);
```

<Pitfall>

Bez zagrada, sav kod koji je napisan u linijama ispod `return`-a [biće ignorisan](https://stackoverflow.com/questions/2846283/what-are-the-rules-for-javascripts-automatic-semicolon-insertion-asi)!

</Pitfall>

## Upotreba komponente {/*using-a-component*/}

Kada ste definisali vašu `Profile` komponentu, možete je ugnjezditi unutar ostalih komponenata. Na primer, možete export-ovati `Gallery` komponentu koja koristi više `Profile` komponenata:

<Sandpack>

```js
function Profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/MK3eW3As.jpg"
      alt="Katherine Johnson"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Zadivljujući naučnici</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```css
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

### Šta pretraživač vidi {/*what-the-browser-sees*/}

Primetite razliku u veličini slova:

* `<section>` je napisano malim slovima, pa React zna da se to odnosi na HTML tag.
* `<Profile />` počinje velikim slovom `P`, pa React zna da želimo koristiti našu komponentu po imenu `Profile`.

`Profile` sadrži još više HTML-a: `<img />`. Na kraju, ovo je ono što pretraživač vidi:

```html
<section>
  <h1>Zadivljujući naučnici</h1>
  <img src="https://react.dev/images/docs/scientists/MK3eW3As.jpg" alt="Katherine Johnson" />
  <img src="https://react.dev/images/docs/scientists/MK3eW3As.jpg" alt="Katherine Johnson" />
  <img src="https://react.dev/images/docs/scientists/MK3eW3As.jpg" alt="Katherine Johnson" />
</section>
```

### Ugnježdavanje i organizacija komponenata {/*nesting-and-organizing-components*/}

Komponente su obične JavaScript funkcije, tako da možete imati više komponenata u jednom fajlu. Ovo je zgodno kada su komponente relativno male ili su usko povezane međusobno. Ako fajl postane prenatrpan, uvek možete pomeriti `Profile` u poseban fajl. Naučićete kako se to radi ubrzo na [stranici o import-ovanju](/learn/importing-and-exporting-components).

Pošto su `Profile` komponente renderovane unutar `Gallery`-a (i to više puta), možemo reći da je `Gallery` **roditeljska komponenta** koja renderuje svaki `Profile` kao "dete". Ovo je deo React-ove magije: komponentu definišete jednom, a onda je koristite koliko god i gde god želite.

<Pitfall>

Komponente mogu renderovati druge komponente, ali **nikad ne smete ugnježdavati njihove definicije**:

```js {2-5}
export default function Gallery() {
  // 🔴 Nikada nemojte definisati komponentu unutar druge komponente!
  function Profile() {
    // ...
  }
  // ...
}
```

Snippet iznad je [veoma spor i prouzrokuje bug-ove](/learn/preserving-and-resetting-state#different-components-at-the-same-position-reset-state). Umesto toga, definišite svaku komponentu na najvišem nivou unutar fajla:

```js {5-8}
export default function Gallery() {
  // ...
}

// ✅ Definišite komponente na najvišem nivou unutar fajla
function Profile() {
  // ...
}
```

Kada detetu (child) komponenti trebaju podaci od roditelja (parent), [prosledite ih preko props-a](/learn/passing-props-to-a-component) umesto da ugnježdavate definicije.

</Pitfall>

<DeepDive>

#### Komponente nemaju granice {/*components-all-the-way-down*/}

Vaša React aplikacija počinje sa "root" komponentom. Obično je ona kreirana kada započnete novi projekat. Na primer, ako koristite [CodeSandbox](https://codesandbox.io/) ili [Next.js](https://nextjs.org/) framework, root komponenta je definisana u `pages/index.js`. U ovim primerima ste export-ovali root komponente.

Većina React aplikacija koristi komponente svuda. To znači da nećete koristiti komponente samo za reusable delove poput dugmića, već i za veće, kao što su sidebar-ovi, liste, pa čak i cele stranice! Komponente su zgodan način za organizaciju UI koda i markup-a, iako su neke se od njih koriste samo na jednom mestu.

[React-based framework-ovi](/learn/creating-a-react-app) odlaze korak dalje. Umesto da koristite prazan HTML fajl i da pustite React-u "da preuzme" upravljanje stranicom uz pomoć JavaScript-a, oni *takođe* automatski generišu HTML na osnovu vaših React komponenata. Na ovaj način vaša aplikacija može prikazati neki sadržaj pre neko što se JavaScript kod učita.

Međutim, veliki broj sajtova koristi React samo za [dodavanje interaktivnosti na postojeće HTML stranice](/learn/add-react-to-an-existing-project#using-react-for-a-part-of-your-existing-page). Oni imaju više root komponenata umesto jedne za čitavu stranicu. Možete koristiti React koliko god mnogo, ili malo, da vam treba.

</DeepDive>

<Recap>

Upravo ste dobili prvi utisak o React-u! Hajde da rezimiramo par ključnih stvari.

* React vam omogućava da kreirate komponente, **reusable UI elemente za vašu aplikaciju**.
* U React aplikaciji, svaki deo UI-a je komponenta.
* React komponente su obične JavaScript funkcije osim što:

  1. Njihova imena počinju sa velikim slovom.
  2. One vraćaju JSX markup.

</Recap>

<Challenges>

#### Export-ovati komponentu {/*export-the-component*/}

Ovaj sandbox ne radi jer root komponenta export-ovana:

<Sandpack>

```js
function Profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/lICfvbD.jpg"
      alt="Aklilu Lemma"
    />
  );
}
```

```css
img { height: 181px; }
```

</Sandpack>

Pokušajte da ga popravite pre nego što pogledate rešenje!

<Solution>

Dodajte `export default` ispred definicije funkcije:

<Sandpack>

```js
export default function Profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/lICfvbD.jpg"
      alt="Aklilu Lemma"
    />
  );
}
```

```css
img { height: 181px; }
```

</Sandpack>

Možda se pitate zašto dodavanje `export` ključne reči nije dovoljno za ispravku ovog primera. Možete naučiti razliku između `export` i `export default` u [Import-ovanje i export-ovanje komponenata](/learn/importing-and-exporting-components).

</Solution>

#### Popraviti return iskaz {/*fix-the-return-statement*/}

Nešto nije u redu sa `return` iskazom. Možete li ga popraviti?

<Hint>

Možete dobiti "Unexpected token" grešku dok pokušavate da rešite problem. U tom slučaju, proverite da li se tačka-zarez nalazi *izvan* zatvarajuće zagrade. Ostavljanjem tačke-zareza unutar `return ( )` dobićete tu grešku.

</Hint>


<Sandpack>

```js
export default function Profile() {
  return
    <img src="https://react.dev/images/docs/scientists/jA8hHMpm.jpg" alt="Katsuko Saruhashi" />;
}
```

```css
img { height: 180px; }
```

</Sandpack>

<Solution>

Možete popraviti ovu komponentu pomeranjem return iskaza u jednu liniju:

<Sandpack>

```js
export default function Profile() {
  return <img src="https://react.dev/images/docs/scientists/jA8hHMpm.jpg" alt="Katsuko Saruhashi" />;
}
```

```css
img { height: 180px; }
```

</Sandpack>

Ili tako što ćete dodati zagrade oko JSX markup-a odmah nakon `return`-a:

<Sandpack>

```js
export default function Profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/jA8hHMpm.jpg"
      alt="Katsuko Saruhashi"
    />
  );
}
```

```css
img { height: 180px; }
```

</Sandpack>

</Solution>

#### Uočiti grešku {/*spot-the-mistake*/}

Nešto nije u redu u vezi definicijom i upotrebom `Profile` komponente. Uočavate li grešku? (Pokušajte da se setite kako React razlikuje komponente i obične HTML tag-ove!)

<Sandpack>

```js
function profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/QIrZWGIs.jpg"
      alt="Alan L. Hart"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Zadivljujući naučnici</h1>
      <profile />
      <profile />
      <profile />
    </section>
  );
}
```

```css
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

<Solution>

Imena React komponenata moraju početi velikim slovom.

Promenite `function profile()` u `function Profile()`, a onda promenite svaki `<profile />` u `<Profile />`:

<Sandpack>

```js
function Profile() {
  return (
    <img
      src="https://react.dev/images/docs/scientists/QIrZWGIs.jpg"
      alt="Alan L. Hart"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Zadivljujući naučnici</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```css
img { margin: 0 10px 10px 0; }
```

</Sandpack>

</Solution>

#### Vaša sopstvena komponenta {/*your-own-component*/}

Napišite komponentu od nule. Možete joj dati bilo koje validno ime i vratiti bilo kakav markup. Ako nemate ideju, možete napisati `Congratulations` komponentu koja prikazuje `<h1>Dobar posao!</h1>`. Ne zaboravite da je export-ujete!

<Sandpack>

```js
// Napišite vašu komponentu ispod!

```

</Sandpack>

<Solution>

<Sandpack>

```js
export default function Congratulations() {
  return (
    <h1>Dobar posao!</h1>
  );
}
```

</Sandpack>

</Solution>

</Challenges>
