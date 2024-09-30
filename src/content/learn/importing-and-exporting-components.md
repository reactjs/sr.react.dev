---
title: Import-ovanje i export-ovanje komponenata
---

<Intro>

Magija komponenata leži u njihovoj upotrebljivosti: možete kreirati komponente koje se sastoje iz drugih komponenata. Međutim, ugnježdavanjem novih i novih komponenata, često ima smisla započeti njihovu podelu u različite fajlove. Ovo vam omogućava da fajlove lakše skenirate i koristite komponente na više mesta.

</Intro>

<YouWillLearn>

* Šta je to fajl root komponente
* Kako da import-ujete i export-ujete komponentu
* Kada da koristite default i imenovane import-e i export-e
* Kako da import-ujete i export-ujete više komponenata iz jednog fajla
* Kako da podelite komponente u više fajlova

</YouWillLearn>

## Fajl root komponente {/*the-root-component-file*/}

U [Vaša prva komponenta](/learn/your-first-component), napravili ste `Profile` komponentu i `Gallery` komponentu koja je renderuje:

<Sandpack>

```js
function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3As.jpg"
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

One trenutno žive u **fajlu root komponente**, koji se u ovom primeru zove `App.js`. U zavisnosti od vaših podešavanja, root komponenta može biti i u drugom fajlu. Ako koristite neki framework sa fajl rutiranjem, kao što je Next.js, vaša root komponenta će biti drugačija na svakoj stranici.

## Export-ovanje i import-ovanje komponente {/*exporting-and-importing-a-component*/}

Šta ako želite da promenite landing stranicu i prikažete listu naučnih knjiga na njoj? Ili da postavite sve profile na neko drugo mesto? Ima smisla pomeriti `Gallery` i `Profile` izvan fajla root komponente. To će im omogućiti da budu modularnije i reusable. Komponentu možete pomeriti u tri koraka:

1. **Napravite** novi JS fajl gde ćete smestiti komponente.
2. **Export-ujte** vašu funkciju komponente iz tog fajla (koristeći ili [default](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/export#using_the_default_export) ili [imenovane](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/export#using_named_exports) export-e).
3. **Import-ujte** ih u fajl gde ćete ih koristiti (pomoću odgovarajuće tehnike import-ovanja [default](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/import#importing_defaults) ili [imenovanih](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/import#import_a_single_export_from_a_module) export-a).

Sada su i `Profile` i `Gallery` pomereni iz `App.js` u fajl pod imenom `Gallery.js`. Sada možete promeniti `App.js` da import-uje `Gallery` iz `Gallery.js`:

<Sandpack>

```js src/App.js
import Gallery from './Gallery.js';

export default function App() {
  return (
    <Gallery />
  );
}
```

```js src/Gallery.js
function Profile() {
  return (
    <img
      src="https://i.imgur.com/QIrZWGIs.jpg"
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
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

Primetite da je primer sada razdvojen na dva fajla:

1. `Gallery.js`:
     - Definiše `Profile` komponentu koja se koristi samo u okviru istog fajla i nije export-ovana.
     - Export-uje `Gallery` komponentu kao **default export**.
2. `App.js`:
     - Import-uje `Gallery` kao **default import** iz `Gallery.js`.
     - Export-uje root `App` komponentu kao **default export**.


<Note>

Možete se susresti sa fajlovima koji ne koriste `.js` ekstenziju:

```js 
import Gallery from './Gallery';
```

I `'./Gallery.js'` i `'./Gallery'` će raditi u React-u, ali je prvi način bliži tome kako [native ES moduli](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules) rade.

</Note>

<DeepDive>

#### Default vs imenovani export-i {/*default-vs-named-exports*/}

Postoje dva primarna načina za export-ovanje vrednosti u JavaScript-u: default export-i i imenovani export-i. Naši primeri su do sad koristili samo default export-e. Možete koristiti jedan ili oba načina u istom fajlu. **Fajl ne sme imati više od jednog _default_ export-a, ali može imati koliko god želite _imenovanih_ export-a.**

![Default i imenovani export-i](/images/docs/illustrations/i_import-export.svg)

Način na koji export-ujete komponentu diktira način za njeno import-ovanje. Dobićete grešku ako pokušate da import-ujete default export na isti način kao i imenovani export! Ova tabela vam može pomoći da bolje razumete:

| Sintaksa    | Export iskaz                          | Import iskaz                            |
| ----------- | -----------                           | -----------                             |
| Default     | `export default function Button() {}` | `import Button from './Button.js';`     |
| Imenovano   | `export function Button() {}`         | `import { Button } from './Button.js';` |

Kada koristite _default_ import, možete staviti bilo koje ime nakon `import`-a. Na primer, možete napisati `import Banana from './Button.js'`, ali ćete i dalje dobiti isti default export. Nasuprot tome, sa imenovanim import-ima, ime mora da se poklapa na obe strane. Zato se i nazivaju _imenovani_ import-i!

**Ljudi često koriste default export-e ako fajl export-uje samo jednu komponentu, a imenovane export-e ako fajl export-uje više komponenata i vrednosti.** Koji god stil kodiranja da preferirate, uvek dajte smislena imena funkcijama komponenata i fajlovima u kojima se nalaze. Komponente bez imena poput `export default () => {}` ne bi trebale da se koriste jer otežavaju debug-ovanje.

</DeepDive>

## Export-ovanje i import-ovanje više komponenata iz istog fajla {/*exporting-and-importing-multiple-components-from-the-same-file*/}

Šta ako želite prikazati samo jedan `Profile` umesto galerije? Možete export-ovati `Profile` komponentu takođe. Ali `Gallery.js` već ima *default* export, a ne možete imati _dva_ default export-a. Možete kreirati novi fajl sa default export-om, ili možete dodati *imenovani* export za `Profile`. **Fajl može imati samo jedan default export, ali može imati bezbroj imenovanih export-a!**

<Note>

Da bi izbegli zabunu između default i imenovanih export-a, neki timovi odluče da se drže jednog načina (default ili imenovani), ili izbegavaju upotrebu oba u istom fajlu. Koristite ono što vam najviše odgovara!

</Note>

Prvo, **export-ujte** `Profile` iz `Gallery.js` upotrebom imenovanog export-a (bez ključne reči `default`):

```js
export function Profile() {
  // ...
}
```

Nakon toga, **import-ujte** `Profile` iz `Gallery.js` u `App.js` upotrebom imenovanog import-a (sa vitičastim zagradama):

```js
import { Profile } from './Gallery.js';
```

Na kraju, **renderujte** `<Profile />` u `App` komponenti:

```js
export default function App() {
  return <Profile />;
}
```

Sada `Gallery.js` sadrži dva export-a: default `Gallery` export i imenovani `Profile` export. `App.js` ih oba import-uje. Probajte da menjate `<Profile />` i `<Gallery />` naizmenično:

<Sandpack>

```js src/App.js
import Gallery from './Gallery.js';
import { Profile } from './Gallery.js';

export default function App() {
  return (
    <Profile />
  );
}
```

```js src/Gallery.js
export function Profile() {
  return (
    <img
      src="https://i.imgur.com/QIrZWGIs.jpg"
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
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

Sada koristite kombinaciju default i imenovanih export-a:

* `Gallery.js`:
  - Export-uje `Profile` komponentu kao **imenovani export pod imenom `Profile`**.
  - Export-uje `Gallery` komponentu kao **default export**.
* `App.js`:
  - Import-uje `Profile` kao **imenovani import pod imenom `Profile`** iz `Gallery.js`.
  - Import-uje `Gallery` kao **default import** iz `Gallery.js`.
  - Export-uje root `App` komponentu kao **default export**.

<Recap>

Na ovoj stranici ste naučili:

* Šta je to fajl root komponente
* Kako da import-ujete i export-ujete komponentu
* Kada i kako da koristite default i imenovane import-e i export-e
* Kako da export-ujete više komponenata iz istog fajla

</Recap>

<Challenges>

#### Delite komponente dalje {/*split-the-components-further*/}

Trenutno, `Gallery.js` export-uje i `Profile` i `Gallery`, što je malo zbunjujuće.

Pomerite `Profile` komponentu u zaseban fajl `Profile.js`, a nakon toga izmenite `App` komponentu da renderuje i `<Profile />` i `<Gallery />` komponente jednu za drugom.

Možete koristiti i default i imenovani export za `Profile`, ali se potrudite da iskoristite odgovarajuću import sintaksu i u `App.js` i u `Gallery.js`! Možete se osloniti na već spomenutu tabelu:

| Sintaksa    | Export iskaz                          | Import iskaz                            |
| ----------- | -----------                           | -----------                             |
| Default     | `export default function Button() {}` | `import Button from './Button.js';`     |
| Imenovano   | `export function Button() {}`         | `import { Button } from './Button.js';` |

<Hint>

Ne zaboravite da import-ujete komponente na mestu gde ih koristite. I `Gallery` koristi `Profile`, zar ne?

</Hint>

<Sandpack>

```js src/App.js
import Gallery from './Gallery.js';
import { Profile } from './Gallery.js';

export default function App() {
  return (
    <div>
      <Profile />
    </div>
  );
}
```

```js src/Gallery.js active
// Pomerite me u Profile.js!
export function Profile() {
  return (
    <img
      src="https://i.imgur.com/QIrZWGIs.jpg"
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

```js src/Profile.js
```

```css
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

Nakon što uspete sa jednim tipom export-a, napravite da radi i sa drugim.

<Solution>

Ovo je rešenje sa imenovanim export-ima:

<Sandpack>

```js src/App.js
import Gallery from './Gallery.js';
import { Profile } from './Profile.js';

export default function App() {
  return (
    <div>
      <Profile />
      <Gallery />
    </div>
  );
}
```

```js src/Gallery.js
import { Profile } from './Profile.js';

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

```js src/Profile.js
export function Profile() {
  return (
    <img
      src="https://i.imgur.com/QIrZWGIs.jpg"
      alt="Alan L. Hart"
    />
  );
}
```

```css
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

Ovo je rešenje sa default export-ima:

<Sandpack>

```js src/App.js
import Gallery from './Gallery.js';
import Profile from './Profile.js';

export default function App() {
  return (
    <div>
      <Profile />
      <Gallery />
    </div>
  );
}
```

```js src/Gallery.js
import Profile from './Profile.js';

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

```js src/Profile.js
export default function Profile() {
  return (
    <img
      src="https://i.imgur.com/QIrZWGIs.jpg"
      alt="Alan L. Hart"
    />
  );
}
```

```css
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

</Solution>

</Challenges>
