---
title: Ažuriranje nizova u state-u
---

<Intro>

Nizovi su promenljivi u JavaScript-u, ali biste trebali da ih tretirate kao immutable kada ih čuvate u state-u. Kao i kod objekata, kada želite da ažurirate niz koji se nalazi u state-u, potrebno je da kreirate novi (ili napravite kopiju postojećeg), a zatim postavite state da koristi taj novi niz.

</Intro>

<YouWillLearn>

- Kako da dodajete, uklanjate ili menjate članove niza u React state-u
- Kako da ažurirate objekat unutar niza
- Kako učiniti da se kopiranje nizova manje ponavlja uz pomoć Immer-a

</YouWillLearn>

## Ažuriranje nizova bez mutacije {/*updating-arrays-without-mutation*/}

U JavaScript-u, nizovi su samo još jedna vrsta objekata. [Kao i sa objektima](/learn/updating-objects-in-state), **trebali biste tretirati nizove u React state-u kao read-only**. To znači da ne trebate dodeljivati vrednost članovima niza poput `arr[0] = 'bird'`, a takođe ne biste trebali da koristite metode koje mutiraju niz, kao što su `push()` i `pop()`.

Umesto toga, svaki put kada želite ažurirati niz, želećete da prosledite *novi* niz u vašu state setter funkciju. Da biste to uradili, možete kreirati novi niz na osnovu originalnog niza u state-u pozivanjem metoda koje ne vrše mutaciju poput `filter()` i `map()`. Onda možete postaviti state na novi rezultujući niz.

Ovde je referentna tabela uobičajenih operacija nad nizovima. Kada koristite nizove unutar React state-a, trebate izbegavati metode u levoj koloni i, umesto toga, koristiti metode iz desne kolone:

|            | izbegavati (mutira niz)         | koristiti (vraća novi niz)                                           |
| ---------- | ----------                      | ----------                                                           |
| dodavanje  | `push`, `unshift`               | `concat`, `[...arr]` spread sintaksa ([primer](#adding-to-an-array)) |
| uklanjanje | `pop`, `shift`, `splice`        | `filter`, `slice` ([primer](#removing-from-an-array))                |
| zamena     | `splice`, `arr[i] = ...` dodela | `map` ([primer](#replacing-items-in-an-array))                       |
| sortiranje | `reverse`, `sort`               | prvo kopirati niz ([primer](#making-other-changes-to-an-array))      |

Alternativno, možete [koristiti Immer](#write-concise-update-logic-with-immer) koji vam dopušta da koristite metode iz obe kolone.

<Pitfall>

Nažalost, [`slice`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice) i [`splice`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice) se zovu slično, ali su veoma drugačiji:

* `slice` kopira niz ili deo niza.
* `splice` **mutira** niz (za ubacivanje ili uklanjanje članova).

U React-u, koristićete `slice` (ne `p`!) mnogo češće zato što ne želite da mutirate objekte i nizove u state-u. [Ažuriranje objekata](/learn/updating-objects-in-state) objašnjava šta je to mutacija i zašto nije preporučena za state.

</Pitfall>

### Dodavanje u niz {/*adding-to-an-array*/}

`push()` ce mutirati niz, a to ne želite:

<Sandpack>

```js
import { useState } from 'react';

let nextId = 0;

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState([]);

  return (
    <>
      <h1>Inspirativni skulptori:</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={() => {
        artists.push({
          id: nextId++,
          name: name,
        });
      }}>Dodaj</button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

```css
button { margin-left: 5px; }
```

</Sandpack>

Umesto toga, napravite *novi* niz koji sadrži sve postojeće članove, *ali i* novi član na kraju. Postoji više načina da to uradite, ali najlakši je upotrebom `...` [spread sintakse](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#spread_in_array_literals) za nizove:

```js
setArtists( // Zameni state
  [ // sa novim nizom
    ...artists, // koji sadrži sve stare članove
    { id: nextId++, name: name } // kao i novi član na kraju
  ]
);
```

Sada radi kako treba:

<Sandpack>

```js
import { useState } from 'react';

let nextId = 0;

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState([]);

  return (
    <>
      <h1>Inspirativni skulptori:</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={() => {
        setArtists([
          ...artists,
          { id: nextId++, name: name }
        ]);
      }}>Dodaj</button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

```css
button { margin-left: 5px; }
```

</Sandpack>

Spread sintaksa za nizove omogućava i dodavanje članova *pre* originalnih `...artists`:

```js
setArtists([
  { id: nextId++, name: name },
  ...artists // Stavi stare članove na kraj
]);
```

Na ovaj način, spread može obavljati oba posla: `push()` - dodavanje na kraj niza i `unshift()` - dodavanje na početak niza. Probajte u sandbox-u iznad!

### Uklanjanje iz niza {/*removing-from-an-array*/}

Najlakši način za uklanjanje člana iz niza je da ga *isfiltrirate*. Drugim rečima, napravićete novi niz koji ne sadrži taj član. Da biste to uradili, koristite `filter` metodu, na primer:

<Sandpack>

```js
import { useState } from 'react';

let initialArtists = [
  { id: 0, name: 'Marta Colvin Andrade' },
  { id: 1, name: 'Lamidi Olonade Fakeye'},
  { id: 2, name: 'Louise Nevelson'},
];

export default function List() {
  const [artists, setArtists] = useState(
    initialArtists
  );

  return (
    <>
      <h1>Inspirativni skulptori:</h1>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>
            {artist.name}{' '}
            <button onClick={() => {
              setArtists(
                artists.filter(a =>
                  a.id !== artist.id
                )
              );
            }}>
              Ukloni
            </button>
          </li>
        ))}
      </ul>
    </>
  );
}
```

</Sandpack>

Kliknite dugme "Ukloni" par puta i pogledajte njegov klik handler.

```js
setArtists(
  artists.filter(a => a.id !== artist.id)
);
```

Ovde, `artists.filter(a => a.id !== artist.id)` znači "napravi niz koji sadrži sve `artists` čiji ID je različit od `artist.id`". Drugim rečima, "Ukloni" dugme kod svakog umetnika će isfiltrirati _tog_ umetnika van niza, a onda zatražiti ponovni render sa rezultujućim nizom. Primetite da `filter` ne menja originalni niz.

### Transformisanje niza {/*transforming-an-array*/}

Ako želite da promenite neki ili sve članove niza, možete koristiti `map()` da kreirate **novi** niz. Funkcija koju prosledite u `map` može odlučiti šta uraditi za svaki član, na osnovu njegovih podataka ili indeksa (ili oba).

U ovom primeru, niz sadrži koordinate dva kruga i jednog kvadrata. Kada pritisnete dugme, samo će se krugovi pomeriti na dole za 50 piksela. To se radi pravljenjem novog niza upotrebom `map()` metode:

<Sandpack>

```js
import { useState } from 'react';

let initialShapes = [
  { id: 0, type: 'circle', x: 50, y: 100 },
  { id: 1, type: 'square', x: 150, y: 100 },
  { id: 2, type: 'circle', x: 250, y: 100 },
];

export default function ShapeEditor() {
  const [shapes, setShapes] = useState(
    initialShapes
  );

  function handleClick() {
    const nextShapes = shapes.map(shape => {
      if (shape.type === 'square') {
        // Nema promene
        return shape;
      } else {
        // Vrati novi krug 50px ispod
        return {
          ...shape,
          y: shape.y + 50,
        };
      }
    });
    // Ponovni render sa novim nizom
    setShapes(nextShapes);
  }

  return (
    <>
      <button onClick={handleClick}>
        Pomeri krugove dole!
      </button>
      {shapes.map(shape => (
        <div
          key={shape.id}
          style={{
          background: 'purple',
          position: 'absolute',
          left: shape.x,
          top: shape.y,
          borderRadius:
            shape.type === 'circle'
              ? '50%' : '',
          width: 20,
          height: 20,
        }} />
      ))}
    </>
  );
}
```

```css
body { height: 300px; }
```

</Sandpack>

### Zamena članova niza {/*replacing-items-in-an-array*/}

Dosta je uobičajeno da želite zameniti jedan ili više članova niza. Dodele poput `arr[0] = 'bird'` mutiraju originalni niz, tako da ćete umesto toga želeti i ovde da koristite `map`.

Da biste zamenili član, napravite novi niz sa `map`. Unutar `map` poziva, dobićete indeks člana kao drugi argument. Iskoristite ga da odlučite da li da vratite originalni član (prvi argument) ili nešto drugo:

<Sandpack>

```js
import { useState } from 'react';

let initialCounters = [
  0, 0, 0
];

export default function CounterList() {
  const [counters, setCounters] = useState(
    initialCounters
  );

  function handleIncrementClick(index) {
    const nextCounters = counters.map((c, i) => {
      if (i === index) {
        // Inkrementiraj kliknuti brojač
        return c + 1;
      } else {
        // Ostatak se ne menja
        return c;
      }
    });
    setCounters(nextCounters);
  }

  return (
    <ul>
      {counters.map((counter, i) => (
        <li key={i}>
          {counter}
          <button onClick={() => {
            handleIncrementClick(i);
          }}>+1</button>
        </li>
      ))}
    </ul>
  );
}
```

```css
button { margin: 5px; }
```

</Sandpack>

### Ubacivanje u niz {/*inserting-into-an-array*/}

Ponekad ćete želeti da ubacite član u niz na određenu poziciju koja nije ni početak ni kraj. Da biste to uradili, možete koristiti `...` spread sintaksu za nizove zajedno sa `slice()` metodom. `slice()` metoda vam omogućava da uzmete "parče" niza. Da biste ubacili član, kreiraćete niz koji sadrži parče _pre_ mesta ubacivanja, novi član i ostatak originalnog niza.

U ovom primeru, dugme "Ubaci" uvek ubacuje na indeks `1`:

<Sandpack>

```js
import { useState } from 'react';

let nextId = 3;
const initialArtists = [
  { id: 0, name: 'Marta Colvin Andrade' },
  { id: 1, name: 'Lamidi Olonade Fakeye'},
  { id: 2, name: 'Louise Nevelson'},
];

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState(
    initialArtists
  );

  function handleClick() {
    const insertAt = 1; // Može biti bilo koji indeks
    const nextArtists = [
      // Članovi pre mesta ubacivanja:
      ...artists.slice(0, insertAt),
      // New item:
      { id: nextId++, name: name },
      // Članovi nakon mesta ubacivanja:
      ...artists.slice(insertAt)
    ];
    setArtists(nextArtists);
    setName('');
  }

  return (
    <>
      <h1>Inspirativni skulptori:</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={handleClick}>
        Ubaci
      </button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

```css
button { margin-left: 5px; }
```

</Sandpack>

### Pravljenje ostalih izmena nad nizom {/*making-other-changes-to-an-array*/}

Postoje stvari koje ne možete uraditi samo sa spread sintaksom i metodama koje ne vrše mutacije poput `map()` i `filter()`. Na primer, možete želeti da obrnete ili sortirate niz. JavaScript `reverse()` i `sort()` metode mutiraju originalni niz, pa ih ne možete koristiti direktno.

**Međutim, možete prvo kopirati niz, pa onda praviti promene nad njim.**

Na primer:

<Sandpack>

```js
import { useState } from 'react';

const initialList = [
  { id: 0, title: 'Big Bellies' },
  { id: 1, title: 'Lunar Landscape' },
  { id: 2, title: 'Terracotta Army' },
];

export default function List() {
  const [list, setList] = useState(initialList);

  function handleClick() {
    const nextList = [...list];
    nextList.reverse();
    setList(nextList);
  }

  return (
    <>
      <button onClick={handleClick}>
        Obrni
      </button>
      <ul>
        {list.map(artwork => (
          <li key={artwork.id}>{artwork.title}</li>
        ))}
      </ul>
    </>
  );
}
```

</Sandpack>

Ovde prvo koristite `[...list]` spread sintaksu da napravite kopiju originalnog niza. Sad kad imate kopiju, možete koristiti metode koje mutiraju kao što su `nextList.reverse()` ili `nextList.sort()`, a možete i vršiti dodelu određenim članovima sa `nextList[0] = "something"`.

Međutim, **čak i ako kopirate niz, ne možete mutirati postojeće članove _unutar_ njega direktno**. Ovo je zbog toga što je kopija plitka--novi niz će sadržati iste članove kao i originalni. Ako izmenite objekat unutar kopiranog niza, vi mutirate postojeći state. Na primer, kod poput ovog je problematičan.

```js
const nextList = [...list];
nextList[0].seen = true; // Problem: mutira list[0]
setList(nextList);
```

Iako su `nextList` i `list` dva različita niza, **`nextList[0]` i `list[0]` pokazuju na isti objekat**. To znači da promenom `nextList[0].seen`, takođe menjate i `list[0].seen`. Ovo je mutacija state-a, što trebate izbegavati! Ovaj problem možete rešiti na sličan način kao i [ažuriranje ugnježdenih JavaScript objekata](/learn/updating-objects-in-state#updating-a-nested-object)--umesto mutacije, možete kopirati pojedinačne članove koje želite da promenite. Evo i kako.

## Ažuriranje objekata unutar nizova {/*updating-objects-inside-arrays*/}

Objekti se ne nalaze _zapravo_ "unutar" nizova. U kodu može delovati kao da su "unutar", ali svaki objekat u nizu je posebna vrednost na koju niz "pokazuje". Zbog toga trebate biti oprezni kad menjate ugnježdena polja poput `list[0]`. Lista umetničkih dela druge osobe može pokazivati na isti element niza!

**Kada ažurirate ugnježdeni state, trebate kreirati kopije od mesta gde želite ažurirati, pa nagore sve do najvišeg nivoa.** Hajde da vidimo kako to funkcioniše.

U ovom primeru, dve različite liste umetničkih dela imaju isti inicijalni state. Trebalo bi da su izolovane, ali, zbog mutacije, slučajno dele state, pa štikliranje u jednoj listi utiče i na drugu:

<Sandpack>

```js
import { useState } from 'react';

let nextId = 3;
const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [myList, setMyList] = useState(initialList);
  const [yourList, setYourList] = useState(
    initialList
  );

  function handleToggleMyList(artworkId, nextSeen) {
    const myNextList = [...myList];
    const artwork = myNextList.find(
      a => a.id === artworkId
    );
    artwork.seen = nextSeen;
    setMyList(myNextList);
  }

  function handleToggleYourList(artworkId, nextSeen) {
    const yourNextList = [...yourList];
    const artwork = yourNextList.find(
      a => a.id === artworkId
    );
    artwork.seen = nextSeen;
    setYourList(yourNextList);
  }

  return (
    <>
      <h1>Lista željenih umetnosti</h1>
      <h2>Moja lista:</h2>
      <ItemList
        artworks={myList}
        onToggle={handleToggleMyList} />
      <h2>Tvoja lista:</h2>
      <ItemList
        artworks={yourList}
        onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

</Sandpack>

Problem je u ovom kodu:

```js
const myNextList = [...myList];
const artwork = myNextList.find(a => a.id === artworkId);
artwork.seen = nextSeen; // Problem: mutira postojeći član
setMyList(myNextList);
```

Iako je sam `myNextList` niz novi, *članovi u njemu* su isti kao u originalnom `myList` nizu. Promenom `artwork.seen` menjate *originalno* umetničko delo. To umetničko delo je i u `yourList`, što prouzrokuje bug. Može biti teško razmišljati o ovakvim bug-ovima, ali, srećom, oni nestaju ako izbegavate mutiranje state-a.

**Možete koristiti `map` da zamenite stari član sa njegovom ažuriranom verzijom bez mutacije.**

```js
setMyList(myList.map(artwork => {
  if (artwork.id === artworkId) {
    // Napravi *novi* objekat sa promenama
    return { ...artwork, seen: nextSeen };
  } else {
    // Nema promena
    return artwork;
  }
}));
```

Ovde, `...` je objektna spread sintaksa kojom se [kreira kopija objekta](/learn/updating-objects-in-state#copying-objects-with-the-spread-syntax).

Ovim pristupom se ne mutira nijedan član postojećeg state-a i popravlja se bug:

<Sandpack>

```js
import { useState } from 'react';

let nextId = 3;
const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [myList, setMyList] = useState(initialList);
  const [yourList, setYourList] = useState(
    initialList
  );

  function handleToggleMyList(artworkId, nextSeen) {
    setMyList(myList.map(artwork => {
      if (artwork.id === artworkId) {
        // Napravi *novi* objekat sa promenama
        return { ...artwork, seen: nextSeen };
      } else {
        // Nema promena
        return artwork;
      }
    }));
  }

  function handleToggleYourList(artworkId, nextSeen) {
    setYourList(yourList.map(artwork => {
      if (artwork.id === artworkId) {
        // Napravi *novi* objekat sa promenama
        return { ...artwork, seen: nextSeen };
      } else {
        // Nema promena
        return artwork;
      }
    }));
  }

  return (
    <>
      <h1>Lista željenih umetnosti</h1>
      <h2>Moja lista:</h2>
      <ItemList
        artworks={myList}
        onToggle={handleToggleMyList} />
      <h2>Tvoja lista:</h2>
      <ItemList
        artworks={yourList}
        onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

</Sandpack>

Uglavnom, **trebali biste mutirati samo objekte koje ste upravo kreirali**. Ako ubacujete *novo* umetničko delo, možete ga mutirati, ali ako menjate nešto što je već u state-u, trebate napraviti kopiju.

### Pisanje koncizne logike ažuriranja sa Immer-om {/*write-concise-update-logic-with-immer*/}

Ažuriranje ugnježdenih nizova bez mutacije može postati ponovljivo. [Baš kao i sa objektima](/learn/updating-objects-in-state#write-concise-update-logic-with-immer):

- Generalno, ne biste trebali da ažurirate state za više od par nivoa dubine. Ako su vaši state objekti dosta hijerarhijski duboki, možete probati da ih [drugačije strukturirate](/learn/choosing-the-state-structure#avoid-deeply-nested-state) kako bi bili flat.
- Ako ne želite promeniti strukturu vašeg state-a, možda biste voleli da koristite [Immer](https://github.com/immerjs/use-immer) koji omogućava pisanje zgodne sintakse za mutiranje i brine o pravljenju kopija za vas.

Ovde je primer Liste željenih umetnosti napisan pomoću Immer-a:

<Sandpack>

```js
import { useState } from 'react';
import { useImmer } from 'use-immer';

let nextId = 3;
const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [myList, updateMyList] = useImmer(
    initialList
  );
  const [yourList, updateYourList] = useImmer(
    initialList
  );

  function handleToggleMyList(id, nextSeen) {
    updateMyList(draft => {
      const artwork = draft.find(a =>
        a.id === id
      );
      artwork.seen = nextSeen;
    });
  }

  function handleToggleYourList(artworkId, nextSeen) {
    updateYourList(draft => {
      const artwork = draft.find(a =>
        a.id === artworkId
      );
      artwork.seen = nextSeen;
    });
  }

  return (
    <>
      <h1>Lista željenih umetnosti</h1>
      <h2>Moja lista:</h2>
      <ItemList
        artworks={myList}
        onToggle={handleToggleMyList} />
      <h2>Tvoja lista:</h2>
      <ItemList
        artworks={yourList}
        onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

```json package.json
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "use-immer": "0.5.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

Primetite kako je sa Immer-om, **mutacija poput `artwork.seen = nextSeen` sada u redu**:

```js
updateMyTodos(draft => {
  const artwork = draft.find(a => a.id === artworkId);
  artwork.seen = nextSeen;
});
```

To se dešava zato što ne mutirate _originalni_ state, već poseban `draft` objekat koji Immer pruža. Slično tome, možete upotrebiti mutirajuće metode poput `push()` i `pop()` nad `draft` objektom.

Iza kulisa, Immer uvek konstruiše novi state od nule na osnovu promena koje ste izvršili nad `draft`-om. Ovo vaše event handler-e čini konciznijim bez mutiranja state-a.

<Recap>

- Možete držati nizove u state-u, ali ih ne smete menjati.
- Umesto da mutirate niz, napravite *novu* verziju niza i ažurirajte state da je koristi.
- Možete koristiti `[...arr, newItem]` spread sintaksu za nizove da napravite nizove sa novim članovima.
- Možete koristiti `filter()` i `map()` za kreiranje novih nizova sa filtriranim i transformisanim članovima.
- Možete koristiti Immer da vam kod bude koncizniji.

</Recap>



<Challenges>

#### Ažurirati proizvod u korpi za kupovinu {/*update-an-item-in-the-shopping-cart*/}

Popunite `handleIncreaseClick` metodu tako da pritiskanje "+" dugmeta inkrementira odgovarajući broj:

<Sandpack>

```js
import { useState } from 'react';

const initialProducts = [{
  id: 0,
  name: 'Baklava',
  count: 1,
}, {
  id: 1,
  name: 'Sir',
  count: 5,
}, {
  id: 2,
  name: 'Špagete',
  count: 2,
}];

export default function ShoppingCart() {
  const [
    products,
    setProducts
  ] = useState(initialProducts)

  function handleIncreaseClick(productId) {

  }

  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>
          {product.name}
          {' '}
          (<b>{product.count}</b>)
          <button onClick={() => {
            handleIncreaseClick(product.id);
          }}>
            +
          </button>
        </li>
      ))}
    </ul>
  );
}
```

```css
button { margin: 5px; }
```

</Sandpack>

<Solution>

Možete koristiti `map` funkciju da napravite novi niz, a onda `...` objektnu spread sintaksu da kreirate kopiju promenjenog objekta u novom nizu:

<Sandpack>

```js
import { useState } from 'react';

const initialProducts = [{
  id: 0,
  name: 'Baklava',
  count: 1,
}, {
  id: 1,
  name: 'Sir',
  count: 5,
}, {
  id: 2,
  name: 'Špagete',
  count: 2,
}];

export default function ShoppingCart() {
  const [
    products,
    setProducts
  ] = useState(initialProducts)

  function handleIncreaseClick(productId) {
    setProducts(products.map(product => {
      if (product.id === productId) {
        return {
          ...product,
          count: product.count + 1
        };
      } else {
        return product;
      }
    }))
  }

  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>
          {product.name}
          {' '}
          (<b>{product.count}</b>)
          <button onClick={() => {
            handleIncreaseClick(product.id);
          }}>
            +
          </button>
        </li>
      ))}
    </ul>
  );
}
```

```css
button { margin: 5px; }
```

</Sandpack>

</Solution>

#### Ukloniti proizvod iz korpe za kupovinu {/*remove-an-item-from-the-shopping-cart*/}

Ova korpa ima "+" dugme koje radi, ali "–" dugme ne radi ništa. Trebate dodati event handler za njega tako da pritiskom smanjite `count` odgovarajućeg proizvoda. Ako pritisnete "–" kad je brojač 1, proizvod bi automatski trebao biti uklonjen iz korpe. Osigurajte da se nikad ne prikazuje 0.

<Sandpack>

```js
import { useState } from 'react';

const initialProducts = [{
  id: 0,
  name: 'Baklava',
  count: 1,
}, {
  id: 1,
  name: 'Sir',
  count: 5,
}, {
  id: 2,
  name: 'Špagete',
  count: 2,
}];

export default function ShoppingCart() {
  const [
    products,
    setProducts
  ] = useState(initialProducts)

  function handleIncreaseClick(productId) {
    setProducts(products.map(product => {
      if (product.id === productId) {
        return {
          ...product,
          count: product.count + 1
        };
      } else {
        return product;
      }
    }))
  }

  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>
          {product.name}
          {' '}
          (<b>{product.count}</b>)
          <button onClick={() => {
            handleIncreaseClick(product.id);
          }}>
            +
          </button>
          <button>
            –
          </button>
        </li>
      ))}
    </ul>
  );
}
```

```css
button { margin: 5px; }
```

</Sandpack>

<Solution>

Možete prvo koristiti `map` za pravljenje novog niza, a onda `filter` da uklonite proizvode kojima je `count` jednak `0`:

<Sandpack>

```js
import { useState } from 'react';

const initialProducts = [{
  id: 0,
  name: 'Baklava',
  count: 1,
}, {
  id: 1,
  name: 'Sir',
  count: 5,
}, {
  id: 2,
  name: 'Špagete',
  count: 2,
}];

export default function ShoppingCart() {
  const [
    products,
    setProducts
  ] = useState(initialProducts)

  function handleIncreaseClick(productId) {
    setProducts(products.map(product => {
      if (product.id === productId) {
        return {
          ...product,
          count: product.count + 1
        };
      } else {
        return product;
      }
    }))
  }

  function handleDecreaseClick(productId) {
    let nextProducts = products.map(product => {
      if (product.id === productId) {
        return {
          ...product,
          count: product.count - 1
        };
      } else {
        return product;
      }
    });
    nextProducts = nextProducts.filter(p =>
      p.count > 0
    );
    setProducts(nextProducts)
  }

  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>
          {product.name}
          {' '}
          (<b>{product.count}</b>)
          <button onClick={() => {
            handleIncreaseClick(product.id);
          }}>
            +
          </button>
          <button onClick={() => {
            handleDecreaseClick(product.id);
          }}>
            –
          </button>
        </li>
      ))}
    </ul>
  );
}
```

```css
button { margin: 5px; }
```

</Sandpack>

</Solution>

#### Popraviti mutacije upotrebom metoda koje ne vrše mutacije {/*fix-the-mutations-using-non-mutative-methods*/}

U ovom primeru, svi event handler-i u `App.js` koriste mutaciju. Kao rezultat, izmena i brisanje todo-ova ne radi. Napišite `handleAddTodo`, `handleChangeTodo` i `handleDeleteTodo` iznova tako da koriste metode koje ne vrše mutacije:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import AddTodo from './AddTodo.js';
import TaskList from './TaskList.js';

let nextId = 3;
const initialTodos = [
  { id: 0, title: 'Kupi mleko', done: true },
  { id: 1, title: 'Pojedi tako', done: false },
  { id: 2, title: 'Skuvaj čaj', done: false },
];

export default function TaskApp() {
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAddTodo(title) {
    todos.push({
      id: nextId++,
      title: title,
      done: false
    });
  }

  function handleChangeTodo(nextTodo) {
    const todo = todos.find(t =>
      t.id === nextTodo.id
    );
    todo.title = nextTodo.title;
    todo.done = nextTodo.done;
  }

  function handleDeleteTodo(todoId) {
    const index = todos.findIndex(t =>
      t.id === todoId
    );
    todos.splice(index, 1);
  }

  return (
    <>
      <AddTodo
        onAddTodo={handleAddTodo}
      />
      <TaskList
        todos={todos}
        onChangeTodo={handleChangeTodo}
        onDeleteTodo={handleDeleteTodo}
      />
    </>
  );
}
```

```js src/AddTodo.js
import { useState } from 'react';

export default function AddTodo({ onAddTodo }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="Dodaj todo"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddTodo(title);
      }}>Dodaj</button>
    </>
  )
}
```

```js src/TaskList.js
import { useState } from 'react';

export default function TaskList({
  todos,
  onChangeTodo,
  onDeleteTodo
}) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <Task
            todo={todo}
            onChange={onChangeTodo}
            onDelete={onDeleteTodo}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ todo, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let todoContent;
  if (isEditing) {
    todoContent = (
      <>
        <input
          value={todo.title}
          onChange={e => {
            onChange({
              ...todo,
              title: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Sačuvaj
        </button>
      </>
    );
  } else {
    todoContent = (
      <>
        {todo.title}
        <button onClick={() => setIsEditing(true)}>
          Izmeni
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={e => {
          onChange({
            ...todo,
            done: e.target.checked
          });
        }}
      />
      {todoContent}
      <button onClick={() => onDelete(todo.id)}>
        Obriši
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

</Sandpack>

<Solution>

U `handleAddTodo`, možete koristiti spread sintaksu za nizove. U `handleChangeTodo`, možete kreirati novi niz sa `map`. U `handleDeleteTodo`, možete kreirati novi niz sa `filter`. Sada lista radi kako treba:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import AddTodo from './AddTodo.js';
import TaskList from './TaskList.js';

let nextId = 3;
const initialTodos = [
  { id: 0, title: 'Kupi mleko', done: true },
  { id: 1, title: 'Pojedi tako', done: false },
  { id: 2, title: 'Skuvaj čaj', done: false },
];

export default function TaskApp() {
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAddTodo(title) {
    setTodos([
      ...todos,
      {
        id: nextId++,
        title: title,
        done: false
      }
    ]);
  }

  function handleChangeTodo(nextTodo) {
    setTodos(todos.map(t => {
      if (t.id === nextTodo.id) {
        return nextTodo;
      } else {
        return t;
      }
    }));
  }

  function handleDeleteTodo(todoId) {
    setTodos(
      todos.filter(t => t.id !== todoId)
    );
  }

  return (
    <>
      <AddTodo
        onAddTodo={handleAddTodo}
      />
      <TaskList
        todos={todos}
        onChangeTodo={handleChangeTodo}
        onDeleteTodo={handleDeleteTodo}
      />
    </>
  );
}
```

```js src/AddTodo.js
import { useState } from 'react';

export default function AddTodo({ onAddTodo }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="Dodaj todo"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddTodo(title);
      }}>Dodaj</button>
    </>
  )
}
```

```js src/TaskList.js
import { useState } from 'react';

export default function TaskList({
  todos,
  onChangeTodo,
  onDeleteTodo
}) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <Task
            todo={todo}
            onChange={onChangeTodo}
            onDelete={onDeleteTodo}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ todo, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let todoContent;
  if (isEditing) {
    todoContent = (
      <>
        <input
          value={todo.title}
          onChange={e => {
            onChange({
              ...todo,
              title: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Sačuvaj
        </button>
      </>
    );
  } else {
    todoContent = (
      <>
        {todo.title}
        <button onClick={() => setIsEditing(true)}>
          Izmeni
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={e => {
          onChange({
            ...todo,
            done: e.target.checked
          });
        }}
      />
      {todoContent}
      <button onClick={() => onDelete(todo.id)}>
        Obriši
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

</Sandpack>

</Solution>


#### Popraviti mutacije koristeći Immer {/*fix-the-mutations-using-immer*/}

Ovo je isti primer kao u prethodnom izazovu. Ovog puta, popravite mutacije upotrebom Immer-a. Za vašu ugodnost, `useImmer` je već import-ovan, a vi trebate promeniti `todos` state promenljivu da ga koristi.

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { useImmer } from 'use-immer';
import AddTodo from './AddTodo.js';
import TaskList from './TaskList.js';

let nextId = 3;
const initialTodos = [
  { id: 0, title: 'Kupi mleko', done: true },
  { id: 1, title: 'Pojedi tako', done: false },
  { id: 2, title: 'Skuvaj čaj', done: false },
];

export default function TaskApp() {
  const [todos, setTodos] = useState(
    initialTodos
  );

  function handleAddTodo(title) {
    todos.push({
      id: nextId++,
      title: title,
      done: false
    });
  }

  function handleChangeTodo(nextTodo) {
    const todo = todos.find(t =>
      t.id === nextTodo.id
    );
    todo.title = nextTodo.title;
    todo.done = nextTodo.done;
  }

  function handleDeleteTodo(todoId) {
    const index = todos.findIndex(t =>
      t.id === todoId
    );
    todos.splice(index, 1);
  }

  return (
    <>
      <AddTodo
        onAddTodo={handleAddTodo}
      />
      <TaskList
        todos={todos}
        onChangeTodo={handleChangeTodo}
        onDeleteTodo={handleDeleteTodo}
      />
    </>
  );
}
```

```js src/AddTodo.js
import { useState } from 'react';

export default function AddTodo({ onAddTodo }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="Dodaj todo"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddTodo(title);
      }}>Dodaj</button>
    </>
  )
}
```

```js src/TaskList.js
import { useState } from 'react';

export default function TaskList({
  todos,
  onChangeTodo,
  onDeleteTodo
}) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <Task
            todo={todo}
            onChange={onChangeTodo}
            onDelete={onDeleteTodo}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ todo, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let todoContent;
  if (isEditing) {
    todoContent = (
      <>
        <input
          value={todo.title}
          onChange={e => {
            onChange({
              ...todo,
              title: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Sačuvaj
        </button>
      </>
    );
  } else {
    todoContent = (
      <>
        {todo.title}
        <button onClick={() => setIsEditing(true)}>
          Izmeni
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={e => {
          onChange({
            ...todo,
            done: e.target.checked
          });
        }}
      />
      {todoContent}
      <button onClick={() => onDelete(todo.id)}>
        Obriši
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

```json package.json
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "use-immer": "0.5.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

<Solution>

Sa Immer-om, možete pisati kod u stilu mutacije, dok god mutirate jedino delove `draft`-a koji vam pruža Immer. Ovde, sve mutacije su izvršene nad `draft` objektom pa kod radi:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { useImmer } from 'use-immer';
import AddTodo from './AddTodo.js';
import TaskList from './TaskList.js';

let nextId = 3;
const initialTodos = [
  { id: 0, title: 'Kupi mleko', done: true },
  { id: 1, title: 'Pojedi tako', done: false },
  { id: 2, title: 'Skuvaj čaj', done: false },
];

export default function TaskApp() {
  const [todos, updateTodos] = useImmer(
    initialTodos
  );

  function handleAddTodo(title) {
    updateTodos(draft => {
      draft.push({
        id: nextId++,
        title: title,
        done: false
      });
    });
  }

  function handleChangeTodo(nextTodo) {
    updateTodos(draft => {
      const todo = draft.find(t =>
        t.id === nextTodo.id
      );
      todo.title = nextTodo.title;
      todo.done = nextTodo.done;
    });
  }

  function handleDeleteTodo(todoId) {
    updateTodos(draft => {
      const index = draft.findIndex(t =>
        t.id === todoId
      );
      draft.splice(index, 1);
    });
  }

  return (
    <>
      <AddTodo
        onAddTodo={handleAddTodo}
      />
      <TaskList
        todos={todos}
        onChangeTodo={handleChangeTodo}
        onDeleteTodo={handleDeleteTodo}
      />
    </>
  );
}
```

```js src/AddTodo.js
import { useState } from 'react';

export default function AddTodo({ onAddTodo }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="Dodaj todo"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddTodo(title);
      }}>Dodaj</button>
    </>
  )
}
```

```js src/TaskList.js
import { useState } from 'react';

export default function TaskList({
  todos,
  onChangeTodo,
  onDeleteTodo
}) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <Task
            todo={todo}
            onChange={onChangeTodo}
            onDelete={onDeleteTodo}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ todo, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let todoContent;
  if (isEditing) {
    todoContent = (
      <>
        <input
          value={todo.title}
          onChange={e => {
            onChange({
              ...todo,
              title: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Sačuvaj
        </button>
      </>
    );
  } else {
    todoContent = (
      <>
        {todo.title}
        <button onClick={() => setIsEditing(true)}>
          Izmeni
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={e => {
          onChange({
            ...todo,
            done: e.target.checked
          });
        }}
      />
      {todoContent}
      <button onClick={() => onDelete(todo.id)}>
        Obriši
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

```json package.json
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "use-immer": "0.5.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

Takođe, možete mešati pristupe sa i bez mutacija pomoću Immer-a.

Na primer, u ovoj verziji `handleAddTodo` je implementiran mutirajući Immer-ov `draft`, dok `handleChangeTodo` i `handleDeleteTodo` koriste `map` i `filter` metode koje ne vrše mutacije:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { useImmer } from 'use-immer';
import AddTodo from './AddTodo.js';
import TaskList from './TaskList.js';

let nextId = 3;
const initialTodos = [
  { id: 0, title: 'Kupi mleko', done: true },
  { id: 1, title: 'Pojedi tako', done: false },
  { id: 2, title: 'Skuvaj čaj', done: false },
];

export default function TaskApp() {
  const [todos, updateTodos] = useImmer(
    initialTodos
  );

  function handleAddTodo(title) {
    updateTodos(draft => {
      draft.push({
        id: nextId++,
        title: title,
        done: false
      });
    });
  }

  function handleChangeTodo(nextTodo) {
    updateTodos(todos.map(todo => {
      if (todo.id === nextTodo.id) {
        return nextTodo;
      } else {
        return todo;
      }
    }));
  }

  function handleDeleteTodo(todoId) {
    updateTodos(
      todos.filter(t => t.id !== todoId)
    );
  }

  return (
    <>
      <AddTodo
        onAddTodo={handleAddTodo}
      />
      <TaskList
        todos={todos}
        onChangeTodo={handleChangeTodo}
        onDeleteTodo={handleDeleteTodo}
      />
    </>
  );
}
```

```js src/AddTodo.js
import { useState } from 'react';

export default function AddTodo({ onAddTodo }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="Dodaj todo"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddTodo(title);
      }}>Dodaj</button>
    </>
  )
}
```

```js src/TaskList.js
import { useState } from 'react';

export default function TaskList({
  todos,
  onChangeTodo,
  onDeleteTodo
}) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <Task
            todo={todo}
            onChange={onChangeTodo}
            onDelete={onDeleteTodo}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ todo, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let todoContent;
  if (isEditing) {
    todoContent = (
      <>
        <input
          value={todo.title}
          onChange={e => {
            onChange({
              ...todo,
              title: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Sačuvaj
        </button>
      </>
    );
  } else {
    todoContent = (
      <>
        {todo.title}
        <button onClick={() => setIsEditing(true)}>
          Izmeni
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={e => {
          onChange({
            ...todo,
            done: e.target.checked
          });
        }}
      />
      {todoContent}
      <button onClick={() => onDelete(todo.id)}>
        Obriši
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

```json package.json
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "use-immer": "0.5.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

Sa Immer-om, možete koristiti stil koji vam se čini najprirodnije u određenoj situaciji.

</Solution>

</Challenges>
