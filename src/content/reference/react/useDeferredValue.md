---
title: useDeferredValue
---

<Intro>

`useDeferredValue` je React Hook koji vam omogućava da odložite ažuriranje dela UI-a.

```js
const deferredValue = useDeferredValue(value)
```

</Intro>

<InlineToc />

---

## Reference {/*reference*/}

### `useDeferredValue(value, initialValue?)` {/*usedeferredvalue*/}

Pozovite `useDeferredValue` na vrhu vaše komponente da dobijete odloženu verziju te vrednosti.

```js
import { useState, useDeferredValue } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  // ...
}
```

[Pogledajte još primera ispod.](#usage)

#### Parametri {/*parameters*/}

* `value`: Vrednost koju želite da odložite. Može imati bilo koji tip.
* **opcioni** `initialValue`: Vrednost koja se koristi prilikom inicijalnog rendera komponente. Ako se izostavi, `useDeferredValue` neće odložiti tokom inicijalnog rendera, jer ne postoji prethodna vrednost `value`-a koja može biti renderovana.


#### Povratne vrednosti {/*returns*/}

- `currentValue`: Tokom inicijalnog rendera, povratna odložena vrednost će biti `initialValue` ili ista kao prosleđena vrednost. Tokom ažuriranja, React će prvo pokušati ponovno renderovanje sa starom vrednošću (znači da će vratiti staru vrednost), a onda pokušati drugi ponovni render u pozadini sa novom vrednošću (znači da će vratiti ažuriranu vrednost).

#### Upozorenja {/*caveats*/}

- Kada je ažuriranje unutar Transition-a, `useDeferredValue` uvek vraća novi `value` i ne stvara odloženi render, pošto je ažuriranje već odloženo.

- Vrednosti koje prosleđujete u `useDeferredValue` bi trebale biti ili primitivne (poput stringova i brojeva) ili objekti kreirani izvan renderovanja. Ako kreirate novi objekat tokom renderovanja i odmah ga prosledite u `useDeferredValue`, biće drugačiji u svakom renderu, stvarajući nepotrebne ponovne rendere u pozadini.

- Kada `useDeferredValue` primi drugačiju vrednost (upoređenu sa [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)), kao dodatak na trenutni render (koji još uvek koristi prethodnu vrednost), zakazaće se ponovni render u pozadini sa novom vrednošću. Ponovni render u pozadini može biti prekinut: ako postoji drugo ažuriranje `value`-a, React će restartovati ponovni render u pozadini. Na primer, ako korisnik brže kuca u input nego što se tabela koja prima odloženu vrednost može ponovo renderovati, tabela će se ponovo renderovati tek kada korisnik prestane sa kucanjem.

- `useDeferredValue` je integrisan sa [`<Suspense>`-om](/reference/react/Suspense). Ako pozadinsko ažuriranje prouzrokovano novom vrednošću suspenduje UI, korisnik neće videti fallback. Videće samo staru odloženu vrednost dok se podaci ne učitaju.

- `useDeferredValue` sam od sebe ne sprečava dodatne mrežne zahteve.

- Ne postoji fiksno kašnjenje koje prouzrokuje `useDeferredValue`. Čim React završi originalni ponovni render, odmah će početi da radi na ponovnom renderu u pozadini sa novom odloženom vrednošću. Bilo koja ažuriranja prouzrokovana event-ovima (poput pisanja) će dobiti prioritet i prekinuti ponovni render u pozadini.

- Ponovni render u pozadini prouzrokovan `useDeferredValue`-om neće okinuti Effect-e dok ne bude završen. Ako se ponovni render u pozadini suspenduje, Effect-i će se pokrenuti nakon što se podaci učitaju i UI ažurira.

---

## Upotreba {/*usage*/}

### Prikazivanje zastarelog sadržaja dok se novi sadržaj učitava {/*showing-stale-content-while-fresh-content-is-loading*/}

Pozovite `useDeferredValue` na vrhu vaše komponente da odložite ažuriranje nekog dela UI-a.

```js [[1, 5, "query"], [2, 5, "deferredQuery"]]
import { useState, useDeferredValue } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  // ...
}
```

Tokom inicijalnog rendera, <CodeStep step={2}>odložena vrednost</CodeStep> će biti ista kao i <CodeStep step={1}>vrednost</CodeStep> koju ste prosledili.

Tokom ažuriranja, <CodeStep step={2}>odložena vrednost</CodeStep> će "lag-ovati" za najnovijom <CodeStep step={1}>vrednošću</CodeStep>. Konkretno, React će prvo ponovo renderovati *bez* ažuriranja odložene vrednosti, a onda pokušati da ponovo renderuje u pozadini sa novodobijenom vrednošću.

**Prođimo kroz primer da vidite kada je ovo korisno.**

<Note>

Ovaj primer očekuje da koristite izvor podataka koji podržava Suspense:

- Fetch-ovanje podataka sa framework-ovima koji podržavaju Suspense, npr. [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) i [Next.js](https://nextjs.org/docs/app/getting-started/fetching-data#with-suspense)
- Lazy-loading koda komponente sa [`lazy`](/reference/react/lazy)
- Čitanje vrednosti Promise-a sa [`use`](/reference/react/use)

[Naučite više o Suspense-u i njegovim ograničenjima.](/reference/react/Suspense)

</Note>


U ovom primeru, `SearchResults` komponenta se [suspenduje](/reference/react/Suspense#displaying-a-fallback-while-content-is-loading) dok fetch-uje rezultate pretrage. Probajte ukucati `"a"`, sačekati rezultate, a onda promeniti na `"ab"`. Rezultati za `"a"` će biti zamenjeni sa fallback-om za učitavanje.

<Sandpack>

```js src/App.js
import { Suspense, useState } from 'react';
import SearchResults from './SearchResults.js';

export default function App() {
  const [query, setQuery] = useState('');
  return (
    <>
      <label>
        Pretraži albume:
        <input value={query} onChange={e => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Učitavanje...</h2>}>
        <SearchResults query={query} />
      </Suspense>
    </>
  );
}
```

```js src/SearchResults.js
import {use} from 'react';
import { fetchData } from './data.js';

export default function SearchResults({ query }) {
  if (query === '') {
    return null;
  }
  const albums = use(fetchData(`/search?q=${query}`));
  if (albums.length === 0) {
    return <p>Nema rezultata za <i>"{query}"</i></p>;
  }
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}
```

```js src/data.js hidden
// Napomena: način fetch-ovanja podataka zavisi od
// framework-a koji koristite uz Suspense.
// Normalno, logika keširanja bi bila unutar framework-a.

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url.startsWith('/search?q=')) {
    return await getSearchResults(url.slice('/search?q='.length));
  } else {
    throw Error('Nije implementirano');
  }
}

async function getSearchResults(query) {
  // Dodaj lažno kašnjenje da bi se primetilo čekanje.
  await new Promise(resolve => {
    setTimeout(resolve, 1000);
  });

  const allAlbums = [{
    id: 13,
    title: 'Let It Be',
    year: 1970
  }, {
    id: 12,
    title: 'Abbey Road',
    year: 1969
  }, {
    id: 11,
    title: 'Yellow Submarine',
    year: 1969
  }, {
    id: 10,
    title: 'The Beatles',
    year: 1968
  }, {
    id: 9,
    title: 'Magical Mystery Tour',
    year: 1967
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];

  const lowerQuery = query.trim().toLowerCase();
  return allAlbums.filter(album => {
    const lowerTitle = album.title.toLowerCase();
    return (
      lowerTitle.startsWith(lowerQuery) ||
      lowerTitle.indexOf(' ' + lowerQuery) !== -1
    )
  });
}
```

```css
input { margin: 10px; }
```

</Sandpack>

Uobičajeni alternativni UI šablon je *odlaganje* ažuriranja liste rezultata i prikazivanje prethodnih rezultata dok novi rezultati ne budu spremni. Pozovite `useDeferredValue` da prosledite odloženu vrednost query-ja:

```js {3,11}
export default function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  return (
    <>
      <label>
        Pretraži albume:
        <input value={query} onChange={e => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Učitavanje...</h2>}>
        <SearchResults query={deferredQuery} />
      </Suspense>
    </>
  );
}
```

`query` će se odmah ažurirati, pa će input prikazati novu vrednost. Međutim, `deferredQuery` će zadržati njegovu prethodnu vrednost dok se podaci ne učitaju, pa će `SearchResults` na trenutak prikazivati zastarele rezultate.

Unestite `"a"` u primer ispod, sačekajte da se rezultati učitaju, a onda promenite na `"ab"`. Primetite kako sad, umesto Suspense fallback-a, vidite zastarelu listu rezultata dok se novi rezultati ne učitaju:

<Sandpack>

```js src/App.js
import { Suspense, useState, useDeferredValue } from 'react';
import SearchResults from './SearchResults.js';

export default function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  return (
    <>
      <label>
        Pretraži albume:
        <input value={query} onChange={e => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Učitavanje...</h2>}>
        <SearchResults query={deferredQuery} />
      </Suspense>
    </>
  );
}
```

```js src/SearchResults.js
import {use} from 'react';
import { fetchData } from './data.js';

export default function SearchResults({ query }) {
  if (query === '') {
    return null;
  }
  const albums = use(fetchData(`/search?q=${query}`));
  if (albums.length === 0) {
    return <p>Nema rezultata za <i>"{query}"</i></p>;
  }
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}
```

```js src/data.js hidden
// Napomena: način fetch-ovanja podataka zavisi od
// framework-a koji koristite uz Suspense.
// Normalno, logika keširanja bi bila unutar framework-a.

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url.startsWith('/search?q=')) {
    return await getSearchResults(url.slice('/search?q='.length));
  } else {
    throw Error('Nije implementirano');
  }
}

async function getSearchResults(query) {
  // Dodaj lažno kašnjenje da bi se primetilo čekanje.
  await new Promise(resolve => {
    setTimeout(resolve, 1000);
  });

  const allAlbums = [{
    id: 13,
    title: 'Let It Be',
    year: 1970
  }, {
    id: 12,
    title: 'Abbey Road',
    year: 1969
  }, {
    id: 11,
    title: 'Yellow Submarine',
    year: 1969
  }, {
    id: 10,
    title: 'The Beatles',
    year: 1968
  }, {
    id: 9,
    title: 'Magical Mystery Tour',
    year: 1967
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];

  const lowerQuery = query.trim().toLowerCase();
  return allAlbums.filter(album => {
    const lowerTitle = album.title.toLowerCase();
    return (
      lowerTitle.startsWith(lowerQuery) ||
      lowerTitle.indexOf(' ' + lowerQuery) !== -1
    )
  });
}
```

```css
input { margin: 10px; }
```

</Sandpack>

<DeepDive>

#### Kako odlaganje vrednosti radi ispod haube? {/*how-does-deferring-a-value-work-under-the-hood*/}

Možete zamisliti da se to dešava u dva koraka:

1. **Prvo, React ponovo renderuje sa novom `query` vrednošću (`"ab"`), ali sa starom `deferredQuery` vrednošću (još uvek `"a"`).** `deferredQuery` vrednost, koju prosleđujete u listu rezultata, je *odložena*: "lag-uje za" `query` vrednošću.

2. **U pozadini React pokušava da ponovo renderuje *i* `query` i `deferredQuery` ažuriran na `"ab"`.** Ako se ovaj ponovni render završi, React će ga prikazati na ekranu. Međutim, ako se suspenduje (rezultati za `"ab"` se još nisu učitali), React će odbaciti pokušaj renderovanja i ponovo pokušati ovaj ponovni render kad se podaci učitaju. Korisnik će videti zastarelu odloženu vrednost sve dok podaci ne budu spremni.

Odloženo "pozadinsko" renderovanje može biti prekinuto. Na primer, ako ponovo kucate u input, React će ga odbaciti i restartovati sa novom vrednošću. React će uvek koristiti poslednju dostupnu vrednost.

Primetite da i dalje postoji mrežni zahtev pri svakom pritisku tastera na tastaturi. Ono što je ovde odloženo je prikaz rezultata (dok ne budu spremni), a ne sami mrežni zahtevi. Čak iako korisnik nastavi da kuca, odgovori za svaki taster će biti keširani, pa je pritiskanje Backspace-a trenutno i ne fetch-uje podatke ponovo.

</DeepDive>

---

### Indikacija da je sadržaj zastareo {/*indicating-that-the-content-is-stale*/}

U primeru iznad, ništa nije ukazivalo da se lista rezultata za najnoviji upit još uvek učitava. Ovo može biti zbunjujuće za korisnika ako novim rezultatima treba vremena da se učitaju. Da bi korisniku bilo očiglednije da lista rezultata ne odgovara najnovijem upitu, možete dodati vizuelnu indikaciju da je prikazana zastarela lista rezultata:

```js {2}
<div style={{
  opacity: query !== deferredQuery ? 0.5 : 1,
}}>
  <SearchResults query={deferredQuery} />
</div>
```

Sa ovom promenom, čim krenete kucati, zastarela lista rezultata će postati pomalo zamagljena dok se nova lista rezultata ne učita. Takođe možete dodati i CSS transition za postepeno zamagljivanje, kao u primeru ispod:

<Sandpack>

```js src/App.js
import { Suspense, useState, useDeferredValue } from 'react';
import SearchResults from './SearchResults.js';

export default function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;
  return (
    <>
      <label>
        Pretraži albume:
        <input value={query} onChange={e => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Učitavanje...</h2>}>
        <div style={{
          opacity: isStale ? 0.5 : 1,
          transition: isStale ? 'opacity 0.2s 0.2s linear' : 'opacity 0s 0s linear'
        }}>
          <SearchResults query={deferredQuery} />
        </div>
      </Suspense>
    </>
  );
}
```

```js src/SearchResults.js
import {use} from 'react';
import { fetchData } from './data.js';

export default function SearchResults({ query }) {
  if (query === '') {
    return null;
  }
  const albums = use(fetchData(`/search?q=${query}`));
  if (albums.length === 0) {
    return <p>Nema rezultata za <i>"{query}"</i></p>;
  }
  return (
    <ul>
      {albums.map(album => (
        <li key={album.id}>
          {album.title} ({album.year})
        </li>
      ))}
    </ul>
  );
}
```

```js src/data.js hidden
// Napomena: način fetch-ovanja podataka zavisi od
// framework-a koji koristite uz Suspense.
// Normalno, logika keširanja bi bila unutar framework-a.

let cache = new Map();

export function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, getData(url));
  }
  return cache.get(url);
}

async function getData(url) {
  if (url.startsWith('/search?q=')) {
    return await getSearchResults(url.slice('/search?q='.length));
  } else {
    throw Error('Nije implementirano');
  }
}

async function getSearchResults(query) {
  // Dodaj lažno kašnjenje da bi se primetilo čekanje.
  await new Promise(resolve => {
    setTimeout(resolve, 1000);
  });

  const allAlbums = [{
    id: 13,
    title: 'Let It Be',
    year: 1970
  }, {
    id: 12,
    title: 'Abbey Road',
    year: 1969
  }, {
    id: 11,
    title: 'Yellow Submarine',
    year: 1969
  }, {
    id: 10,
    title: 'The Beatles',
    year: 1968
  }, {
    id: 9,
    title: 'Magical Mystery Tour',
    year: 1967
  }, {
    id: 8,
    title: 'Sgt. Pepper\'s Lonely Hearts Club Band',
    year: 1967
  }, {
    id: 7,
    title: 'Revolver',
    year: 1966
  }, {
    id: 6,
    title: 'Rubber Soul',
    year: 1965
  }, {
    id: 5,
    title: 'Help!',
    year: 1965
  }, {
    id: 4,
    title: 'Beatles For Sale',
    year: 1964
  }, {
    id: 3,
    title: 'A Hard Day\'s Night',
    year: 1964
  }, {
    id: 2,
    title: 'With The Beatles',
    year: 1963
  }, {
    id: 1,
    title: 'Please Please Me',
    year: 1963
  }];

  const lowerQuery = query.trim().toLowerCase();
  return allAlbums.filter(album => {
    const lowerTitle = album.title.toLowerCase();
    return (
      lowerTitle.startsWith(lowerQuery) ||
      lowerTitle.indexOf(' ' + lowerQuery) !== -1
    )
  });
}
```

```css
input { margin: 10px; }
```

</Sandpack>

---

### Odlaganje ponovnog rendera za deo UI-a {/*deferring-re-rendering-for-a-part-of-the-ui*/}

Takođe, možete primeniti `useDeferredValue` kao optimizaciju performansi. Korisno je kada je deo vašeg UI-a spor prilikom ponovnog rendera, ne postoji lak način da ga optimizujete, a želite da ga sprečite da blokira ostatak UI-a.

Zamislite da imate tekstualno polje i komponentu (poput tabele ili dugačke liste) koja se ponovo renderuje pri svakom pritisku tastera na tastaturi:

```js
function App() {
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <SlowList text={text} />
    </>
  );
}
```

Prvo, optimizujte `SlowList` da preskoči ponovne rendere kada su joj props-i jednaki. Da biste to uradili, [obmotajte je sa `memo`](/reference/react/memo#skipping-re-rendering-when-props-are-unchanged):

```js {1,3}
const SlowList = memo(function SlowList({ text }) {
  // ...
});
```

Međutim, ovo pomaže samo kada su props-i `SlowList`-a *jednaki* kao u prethodnom renderu. Problem koji sada imate je da je sporo kada su props-i *drugačiji* i kada zapravo trebate prikazati drugačiji vizuelni prikaz.

Konkretno, glavni problem sa performansama je taj da kad god nešto kucate u input, `SlowList` prima nove props-e, a ponovno renderovanje celokupnog stabla čini kucanje nezgodnim. U ovom slučaju, `useDeferredValue` vam omogućava da date prioritet ažuriranju input-a (koje mora biti brzo) u odnosu na ažuriranje liste rezultata (što je dozvoljeno da bude sporije):

```js {3,7}
function App() {
  const [text, setText] = useState('');
  const deferredText = useDeferredValue(text);
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <SlowList text={deferredText} />
    </>
  );
}
```

Ovo ne čini ponovno renderovanje `SlowList`-a bržim. Međutim, govori React-u da ponovno renderovanje liste ima manji prioritet, tako da neće blokirati unos sa tastature. Lista će "lag-ovati za" input-om i onda ga "sustići". Kao i pre, React će pokušati da ažurira listu što je pre moguće, ali neće blokirati korisnika prilikom kucanja.

<Recipes titleText="Razlika između upotrebe useDeferredValue i neoptimizovanog ponovnog rendera" titleId="examples">

#### Odloženo ponovno renderovanje liste {/*deferred-re-rendering-of-the-list*/}

U ovom primeru, svaka stavka u `SlowList` komponenti je **veštački usporena** kako biste videli da `useDeferredValue` omogućava da input ostane responzivan. Kucajte u input i primetite da kucanje deluje brzo, iako lista "lag-uje za" njim.

<Sandpack>

```js
import { useState, useDeferredValue } from 'react';
import SlowList from './SlowList.js';

export default function App() {
  const [text, setText] = useState('');
  const deferredText = useDeferredValue(text);
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <SlowList text={deferredText} />
    </>
  );
}
```

```js {expectedErrors: {'react-compiler': [19, 20]}} src/SlowList.js
import { memo } from 'react';

const SlowList = memo(function SlowList({ text }) {
  // Loguj jednom. Stvarno usporavanje je unutar SlowItem-a.
  console.log('[VEŠTAČKI SPORO] Renderovanje 250 <SlowItem />');

  let items = [];
  for (let i = 0; i < 250; i++) {
    items.push(<SlowItem key={i} text={text} />);
  }
  return (
    <ul className="items">
      {items}
    </ul>
  );
});

function SlowItem({ text }) {
  let startTime = performance.now();
  while (performance.now() - startTime < 1) {
    // Ne radi ništa 1 ms po stavci da simuliraš veoma spor kod
  }

  return (
    <li className="item">
      Tekst: {text}
    </li>
  )
}

export default SlowList;
```

```css
.items {
  padding: 0;
}

.item {
  list-style: none;
  display: block;
  height: 40px;
  padding: 5px;
  margin-top: 10px;
  border-radius: 4px;
  border: 1px solid #aaa;
}
```

</Sandpack>

<Solution />

#### Neoptimizovano ponovno renderovanje liste {/*unoptimized-re-rendering-of-the-list*/}

U ovom primeru, svaka stavka u `SlowList` komponenti je **veštački usporena**, ali nema `useDeferredValue`-a.

Primetite kako kucanje u input deluje baš nezgodno. Tako je zato što bez `useDeferredValue`-a, svaki pritisak tastera primorava celu listu da se odmah ponovo renderuje na način koji se ne može prekinuti.

<Sandpack>

```js
import { useState } from 'react';
import SlowList from './SlowList.js';

export default function App() {
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <SlowList text={text} />
    </>
  );
}
```

```js {expectedErrors: {'react-compiler': [19, 20]}} src/SlowList.js
import { memo } from 'react';

const SlowList = memo(function SlowList({ text }) {
  // Loguj jednom. Stvarno usporavanje je unutar SlowItem-a.
  console.log('[VEŠTAČKI SPORO] Renderovanje 250 <SlowItem />');

  let items = [];
  for (let i = 0; i < 250; i++) {
    items.push(<SlowItem key={i} text={text} />);
  }
  return (
    <ul className="items">
      {items}
    </ul>
  );
});

function SlowItem({ text }) {
  let startTime = performance.now();
  while (performance.now() - startTime < 1) {
    // Ne radi ništa 1 ms po stavci da simuliraš veoma spor kod
  }

  return (
    <li className="item">
      Tekst: {text}
    </li>
  )
}

export default SlowList;
```

```css
.items {
  padding: 0;
}

.item {
  list-style: none;
  display: block;
  height: 40px;
  padding: 5px;
  margin-top: 10px;
  border-radius: 4px;
  border: 1px solid #aaa;
}
```

</Sandpack>

<Solution />

</Recipes>

<Pitfall>

Ova optimizacija zahteva da je `SlowList` obmotan sa [`memo`](/reference/react/memo). To je zbog toga što kad god se `text` promeni, React mora biti sposoban da brzo ponovo renderuje roditeljsku komponentu. Tokom tog ponovnog rendera, `deferredText` i dalje ima prethodnu vrednost, pa `SlowList` može da preskoči ponovno renderovanje (props-i joj se nisu promenili). Bez [`memo`](/reference/react/memo) bi svakako morala da se ponovo renderuje, čime bi se izgubio smisao optimizacije.

</Pitfall>

<DeepDive>

#### Kako je odlaganje vrednosti različito od debouncing-a i throttling-a? {/*how-is-deferring-a-value-different-from-debouncing-and-throttling*/}

Postoje dve uobičajene tehnike optimizacije koje ste možda ranije koristili u ovakvim slučajevima:

- *Debouncing* znači da ćete čekati korisnika da prestane sa kucanjem (npr. na sekund) pre ažuriranja liste.
- *Throttling* znači da ćete ažurirati listu svako malo (npr. najviše jednom u sekundi).

Iako su ove tehnike korisne u nekim slučajevima, `useDeferredValue` je bolji za optimizaciju renderovanja jer je duboko integrisan sa samim React-om i prilagođen uređaju korisnika.

Za razliku od debouncing-a ili throttling-a, ne zahteva odabir fiksnog kašnjenja. Ako je uređaj korisnika brz (npr. moćan laptop), odloženi ponovni render se dešava gotovo odmah i neće biti primetan. Ako je uređaj korisnika spor, lista će "lag-ovati za" input-om proporcionalno u odnosu na to koliko je uređaj spor.

Takođe, za razliku od debouncing-a ili throttling-a, odloženi ponovni renderi sa `useDeferredValue` se, po default-u, mogu prekinuti. To znači da, ako je React u sred ponovnog renderovanja velike liste, a korisnik pritisne neki taster, React će odbaciti taj ponovni render, obraditi pritisak tastera, i tek onda ponovo započeti renderovanje u pozadini. U poređenju sa tim, debouncing i throttling i dalje pružaju nezgodno iskustvo jer su *blokirajući*: oni samo odlažu momenat kada renderovanje blokira pritiskanje tastera.

Ako se posao koji optimizujete ne dešava tokom renderovanja, debouncing i throttling su i dalje korisni. Na primer, mogu vam omogućiti da pravite manje mrežnih zahteva. Takođe, ove tehnike možete koristiti zajedno.

</DeepDive>
