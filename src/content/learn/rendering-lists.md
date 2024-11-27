---
title: Renderovanje listi
---

<Intro>

Često ćete želeti da prikažete više sličnih komponenata iz kolekcije podataka. Možete koristiti [JavaScript metode za nizove](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array#) kako bi manipulisali nizovima podataka. U ovom članku, koristićete [`filter()`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) i [`map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) da uz React filtrirate i transformišete vaš niz podataka u niz komponenata.

</Intro>

<YouWillLearn>

* Kako da renderujete komponente na osnovu niza uz pomoć JavaScript-ovog `map()`-a
* Kako da renderujete samo specifične komponente uz pomoć JavaScript-ovog `filter()`-a
* Kada i zašto da koristite React ključeve

</YouWillLearn>

## Renderovanje podataka iz nizova {/*rendering-data-from-arrays*/}

Recimo da imate sledeću listu.

```js
<ul>
  <li>Creola Katherine Johnson: matematičar</li>
  <li>Mario José Molina-Pasquel Henríquez: hemičar</li>
  <li>Mohammad Abdus Salam: fizičar</li>
  <li>Percy Lavon Julian: hemičar</li>
  <li>Subrahmanyan Chandrasekhar: astrofizičar</li>
</ul>
```

Jedina razlika u stavkama liste je njihov sadržaj, njihovi podaci. Često ćete imati potrebu da prikažete nekoliko instanci iste komponente sa drugačijim podacima kada pravite interfejse: od liste komentara do galerije profilnih slika. U takvim situacijama, možete čuvati podatke u JavaScript objektima i nizovima i upotrebiti metode kao što su [`map()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) i [`filter()`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) da od njih renderujete listu komponenata.

Evo kratkog primera kako da generišete listu stavki na osnovu niza:

1. **Pomerite** podatke u niz:

```js
const people = [
  'Creola Katherine Johnson: matematičar',
  'Mario José Molina-Pasquel Henríquez: hemičar',
  'Mohammad Abdus Salam: fizičar',
  'Percy Lavon Julian: hemičar',
  'Subrahmanyan Chandrasekhar: astrofizičar'
];
```

2. **Mapirajte** `people` članove u novi niz JSX čvorova, `listItems`:

```js
const listItems = people.map(person => <li>{person}</li>);
```

3. **Vratite** `listItems` iz vaše komponente umotano u `<ul>`:

```js
return <ul>{listItems}</ul>;
```

Evo rezultata:

<Sandpack>

```js
const people = [
  'Creola Katherine Johnson: matematičar',
  'Mario José Molina-Pasquel Henríquez: hemičar',
  'Mohammad Abdus Salam: fizičar',
  'Percy Lavon Julian: hemičar',
  'Subrahmanyan Chandrasekhar: astrofizičar'
];

export default function List() {
  const listItems = people.map(person =>
    <li>{person}</li>
  );
  return <ul>{listItems}</ul>;
}
```

```css
li { margin-bottom: 10px; }
```

</Sandpack>

Primetite da sandbox iznad prikazuje grešku u konzoli:

<ConsoleBlock level="error">

Warning: Each child in a list should have a unique "key" prop.

</ConsoleBlock>

<ConsoleBlock level="error">

Upozorenje: Svako dete u listi bi trebalo da ima jedinstven "key" prop.

</ConsoleBlock>

Naučićete kako da popravite grešku malo kasnije u ovom članku. Pre toga, hajde da malo strukturiramo podatke.

## Filtriranje niza stavki {/*filtering-arrays-of-items*/}

Ovi podaci mogu dodatno biti strukturirani.

```js
const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'matematičar',
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'hemičar',
}, {
  id: 2,
  name: 'Mohammad Abdus Salam',
  profession: 'fizičar',
}, {
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'hemičar',
}, {
  id: 4,
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrofizičar',
}];
```

Recimo da želite pronaći način da prikažete samo ljude čija je profesija `'hemičar'`. Možete koristiti JavaScript-ovu `filter()` metodu da vratite samo te ljude. Ova metoda prima niz stavki, propušta ih kroz “test” (funkcija koja vraća `true` ili `false`), i vraća novi niz samo onih stavki koje su prošle test (vraćeno je `true`).

Želite samo stavke gde je `profession` jednako `'hemičar'`. Ta "test" funkcija u ovom slučaju izgleda `(person) => person.profession === 'hemičar'`. Evo kako to možete uraditi:

1. **Napravite** novi niz samo “hemičara”, `chemists`, pozivanjem `filter()` nad `people` i filtriranjem `person.profession === 'hemičar'`:

```js
const chemists = people.filter(person =>
  person.profession === 'hemičar'
);
```

2. Sada **mapirajte** kroz `chemists`:

```js {1,13}
const listItems = chemists.map(person =>
  <li>
     <img
       src={getImageUrl(person)}
       alt={person.name}
     />
     <p>
       <b>{person.name}:</b>
       {' ' + person.profession + ' '}
       poznat je zbog {person.accomplishment}
     </p>
  </li>
);
```

3. Na kraju, **vratite** `listItems` iz vaše komponente:

```js
return <ul>{listItems}</ul>;
```

<Sandpack>

```js src/App.js
import { people } from './data.js';
import { getImageUrl } from './utils.js';

export default function List() {
  const chemists = people.filter(person =>
    person.profession === 'hemičar'
  );
  const listItems = chemists.map(person =>
    <li>
      <img
        src={getImageUrl(person)}
        alt={person.name}
      />
      <p>
        <b>{person.name}:</b>
        {' ' + person.profession + ' '}
        poznat je zbog {person.accomplishment}
      </p>
    </li>
  );
  return <ul>{listItems}</ul>;
}
```

```js src/data.js
export const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'matematičar',
  accomplishment: 'formula za svemirske letove',
  imageId: 'MK3eW3A'
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'hemičar',
  accomplishment: 'otkriće Arktičke rupe u ozonu',
  imageId: 'mynHUSa'
}, {
  id: 2,
  name: 'Mohammad Abdus Salam',
  profession: 'fizičar',
  accomplishment: 'teorija o elektromagnetizmu',
  imageId: 'bE7W1ji'
}, {
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'hemičar',
  accomplishment: 'pionirski kortizon, steroidi i pilule za kontrolu rađanja',
  imageId: 'IOjWm71'
}, {
  id: 4,
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrofizičar',
  accomplishment: 'računanje mase belog patuljka',
  imageId: 'lrWQx8l'
}];
```

```js src/utils.js
export function getImageUrl(person) {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    's.jpg'
  );
}
```

```css
ul { list-style-type: none; padding: 0px 10px; }
li { 
  margin-bottom: 10px; 
  display: grid; 
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
img { width: 100px; height: 100px; border-radius: 50%; }
```

</Sandpack>

<Pitfall>

Arrow funkcije implicitno vraćaju izraz desno od `=>`, tako da vam ne treba `return` iskaz:

```js
const listItems = chemists.map(person =>
  <li>...</li> // Implicitno vraćanje!
);
```

Međutim, **morate napisati `return` eksplicitno ako se nakon `=>` nalazi `{` vitičasta zagrada**!

```js
const listItems = chemists.map(person => { // Vitičasta zagrada
  return <li>...</li>;
});
```

Za arrow funkcije koje sadrže `=> {` se kaže da imaju ["blok telo"](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions#function_body). Omogućavaju vam da napišete više od jedne linije koda, ali *morate da* napišete `return` iskaz samostalno. Ako ga zaboravite, ništa neće biti vraćeno!

</Pitfall>

## Čuvanje redosleda stavki u listi sa `key` {/*keeping-list-items-in-order-with-key*/}

Primetite da svi sandbox-ovi iznad prikazuju grešku u konzoli:

<ConsoleBlock level="error">

Warning: Each child in a list should have a unique "key" prop.

</ConsoleBlock>

<ConsoleBlock level="error">

Upozorenje: Svako dete u listi bi trebalo da ima jedinstven "key" prop.

</ConsoleBlock>

Svakom članu niza morate dodeliti `key` -- string ili broj koji ga jedinstveno identifikuje u tom nizu:

```js
<li key={person.id}>...</li>
```

<Note>

JSX elementima unutar `map()` poziva su uvek potrebni ključevi!

</Note>

Ključevi govore React-u koji član niza odgovara kojoj komponenti, kako bi mogao kasnije da ih poveže. Ovo postaje značajno ako članovi niza mogu da se pomeraju (npr. prilikom sortiranja), ubacuju, ili brišu. Dobro odabran `key` pomaže React-u da zaključi šta se zapravo dogodilo, kako bi ispravno mogao da ažurira DOM stablo.

Umesto da generišete ključeve u hodu, trebalo bi da ih uključite u vaše podatke:

<Sandpack>

```js src/App.js
import { people } from './data.js';
import { getImageUrl } from './utils.js';

export default function List() {
  const listItems = people.map(person =>
    <li key={person.id}>
      <img
        src={getImageUrl(person)}
        alt={person.name}
      />
      <p>
        <b>{person.name}</b>
          {' ' + person.profession + ' '}
          poznat je zbog {person.accomplishment}
      </p>
    </li>
  );
  return <ul>{listItems}</ul>;
}
```

```js src/data.js active
export const people = [{
  id: 0, // Koristi se u JSX kao ključ
  name: 'Creola Katherine Johnson',
  profession: 'matematičar',
  accomplishment: 'formula za svemirske letove',
  imageId: 'MK3eW3A'
}, {
  id: 1, // Koristi se u JSX kao ključ
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'hemičar',
  accomplishment: 'otkriće Arktičke rupe u ozonu',
  imageId: 'mynHUSa'
}, {
  id: 2, // Koristi se u JSX kao ključ
  name: 'Mohammad Abdus Salam',
  profession: 'fizičar',
  accomplishment: 'teorija o elektromagnetizmu',
  imageId: 'bE7W1ji'
}, {
  id: 3, // Koristi se u JSX kao ključ
  name: 'Percy Lavon Julian',
  profession: 'hemičar',
  accomplishment: 'pionirski kortizon, steroidi i pilule za kontrolu rađanja',
  imageId: 'IOjWm71'
}, {
  id: 4, // Koristi se u JSX kao ključ
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrofizičar',
  accomplishment: 'računanje mase belog patuljka',
  imageId: 'lrWQx8l'
}];
```

```js src/utils.js
export function getImageUrl(person) {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    's.jpg'
  );
}
```

```css
ul { list-style-type: none; padding: 0px 10px; }
li { 
  margin-bottom: 10px; 
  display: grid; 
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
img { width: 100px; height: 100px; border-radius: 50%; }
```

</Sandpack>

<DeepDive>

#### Prikazivanje nekoliko DOM čvorova za svaku stavku u listi {/*displaying-several-dom-nodes-for-each-list-item*/}

Šta raditi kad svaka stavka u listi treba renderovati ne jedan, već nekoliko DOM čvorova?

Kratka [`<>...</>` Fragment](/reference/react/Fragment) sintaksa vam ne dopušta da prosledite ključ, tako da ih morate grupisati u jedan `<div>`, ili prosto upotrebiti malo dužu i [eksplicitniju `<Fragment>` sintaksu](/reference/react/Fragment#rendering-a-list-of-fragments):

```js
import { Fragment } from 'react';

// ...

const listItems = people.map(person =>
  <Fragment key={person.id}>
    <h1>{person.name}</h1>
    <p>{person.bio}</p>
  </Fragment>
);
```

Fragment-i nestaju iz DOM-a, tako da ćete dobiti listu od `<h1>`, `<p>`, `<h1>`, `<p>`, i tako dalje.

</DeepDive>

### Gde dobiti `key` {/*where-to-get-your-key*/}

Različiti izvori podataka pružaju različite ključeve:

* **Podaci iz baze podataka:** Ako podaci dolaze iz baze podataka, možete koristiti ključeve ili ID-eve iz baze podataka, koji su po prirodi jedinstveni.
* **Lokalno generisani podaci:** Ako su vam podaci generisani i čuvani lokalno (npr. beleške u aplikaciji za zabeleške), koristite inkrementalni brojač, [`crypto.randomUUID()`](https://developer.mozilla.org/en-US/docs/Web/API/Crypto/randomUUID) ili pakete poput [`uuid`](https://www.npmjs.com/package/uuid) kada kreirate stavke.

### Pravila ključeva {/*rules-of-keys*/}

* **Ključevi moraju biti jedinstveni između „sestrinskih” stavki.** Međutim, u redu je koristiti iste ključeve za JSX čvorove u _različitim_ nizovima.
* **Ključevi se ne smeju menjati** ili će im se svrha obesmisliti! Nemojte ih generisati tokom renderovanja.

### Zašto su React-u potrebni ključevi? {/*why-does-react-need-keys*/}

Zamislite da fajlovi na vašem desktop-u nemaju imena. Umesto toga, referencirali bi ih po njihovom redosledu -- prvi fajl, drugi fajl, i tako dalje. Mogli biste se navići, ali jednom kada obrišete fajl, postalo bi zbunjujuće. Drugi fajl bi postao prvi fajl, treći fajl bi postao drugi fajl, i tako dalje.

Imena fajlova u folderu i JSX ključevi imaju sličnu ulogu. Omogućavaju nam da jedinstveno identifikujemo stavku među „sestrinskim” stavkama. Dobro odabran ključ pruža više informacija od puke pozicije u nizu. Čak iako se _pozicija_ promeni zbog promene redosleda, `key` omogućava React-u da identifikuje stavku tokom njenog životnog veka.

<Pitfall>

Možete biti u iskušenju da koristite indeks člana niza kao njegov ključ. U suštini, to je ono što će React koristiti ako ne specificirate `key`. Ali, redosled u kojem renderujete stavke će se menjati tokom vremena ako se neka stavka ubaci, obriše, ili se promeni redosled niza. Indeks kao ključ često dovodi do suptilnih i zbunjujućih bug-ova.

Slično tome, nemojte generisati ključeve u hodu, npr. pomoću `key={Math.random()}`. Ovo će učiniti da se ključevi ne podudaraju između renderovanja, što znači da će sve vaše komponente, kao i DOM, biti ponovo kreirane svaki put. Ne samo što je sporo, već ćete izgubiti bilo koji korisnički input unutar stavke liste. Umesto toga, koristite stabilan ID baziran na podacima.

Obratite pažnju da vaše komponente neće primiti `key` kao prop. Njega sam React koristi kao nagoveštaj. Ako je vašoj komponenti potreban ID, morate ga proslediti kao poseban prop: `<Profile key={id} userId={id} />`.

</Pitfall>

<Recap>

Na ovoj stranici naučili ste:

* Kako da premestite podatke iz komponenti u strukture podataka poput nizova i objekata.
* Kako da generišete setove sličnih komponenata pomoću JavaScript-ovog `map()`-a.
* Kako da kreirate nizove filtriranih stavki pomoću JavaScript-ovog `filter()`-a.
* Zašto i kako da postavite `key` za svaku komponentu u kolekciji, kako bi React mogao da prati svaku od njih, čak iako im se pozicija ili podaci promene.

</Recap>



<Challenges>

#### Podeliti listu na dva dela {/*splitting-a-list-in-two*/}

Ovaj primer prikazuje listu svih ljudi.

Promenite ga da prikazuje dve odvojene liste jednu za drugom: **Hemičari** i **Svi ostali**. Kao i ranije, možete odrediti da li je osoba hemičar sledećim uslovom `person.profession === 'hemičar'`.

<Sandpack>

```js src/App.js
import { people } from './data.js';
import { getImageUrl } from './utils.js';

export default function List() {
  const listItems = people.map(person =>
    <li key={person.id}>
      <img
        src={getImageUrl(person)}
        alt={person.name}
      />
      <p>
        <b>{person.name}:</b>
        {' ' + person.profession + ' '}
        poznat je zbog {person.accomplishment}
      </p>
    </li>
  );
  return (
    <article>
      <h1>Naučnici</h1>
      <ul>{listItems}</ul>
    </article>
  );
}
```

```js src/data.js
export const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'matematičar',
  accomplishment: 'formula za svemirske letove',
  imageId: 'MK3eW3A'
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'hemičar',
  accomplishment: 'otkriće Arktičke rupe u ozonu',
  imageId: 'mynHUSa'
}, {
  id: 2,
  name: 'Mohammad Abdus Salam',
  profession: 'fizičar',
  accomplishment: 'teorija o elektromagnetizmu',
  imageId: 'bE7W1ji'
}, {
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'hemičar',
  accomplishment: 'pionirski kortizon, steroidi i pilule za kontrolu rađanja',
  imageId: 'IOjWm71'
}, {
  id: 4,
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrofizičar',
  accomplishment: 'računanje mase belog patuljka',
  imageId: 'lrWQx8l'
}];
```

```js src/utils.js
export function getImageUrl(person) {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    's.jpg'
  );
}
```

```css
ul { list-style-type: none; padding: 0px 10px; }
li {
  margin-bottom: 10px;
  display: grid;
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
img { width: 100px; height: 100px; border-radius: 50%; }
```

</Sandpack>

<Solution>

Možete koristiti `filter()` dvaput, kreirajući dva odvojena niza, a nakon toga pozvati `map` nad oba:

<Sandpack>

```js src/App.js
import { people } from './data.js';
import { getImageUrl } from './utils.js';

export default function List() {
  const chemists = people.filter(person =>
    person.profession === 'hemičar'
  );
  const everyoneElse = people.filter(person =>
    person.profession !== 'hemičar'
  );
  return (
    <article>
      <h1>Naučnici</h1>
      <h2>Hemičari</h2>
      <ul>
        {chemists.map(person =>
          <li key={person.id}>
            <img
              src={getImageUrl(person)}
              alt={person.name}
            />
            <p>
              <b>{person.name}:</b>
              {' ' + person.profession + ' '}
              poznat je zbog {person.accomplishment}
            </p>
          </li>
        )}
      </ul>
      <h2>Svi ostali</h2>
      <ul>
        {everyoneElse.map(person =>
          <li key={person.id}>
            <img
              src={getImageUrl(person)}
              alt={person.name}
            />
            <p>
              <b>{person.name}:</b>
              {' ' + person.profession + ' '}
              poznat je zbog {person.accomplishment}
            </p>
          </li>
        )}
      </ul>
    </article>
  );
}
```

```js src/data.js
export const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'matematičar',
  accomplishment: 'formula za svemirske letove',
  imageId: 'MK3eW3A'
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'hemičar',
  accomplishment: 'otkriće Arktičke rupe u ozonu',
  imageId: 'mynHUSa'
}, {
  id: 2,
  name: 'Mohammad Abdus Salam',
  profession: 'fizičar',
  accomplishment: 'teorija o elektromagnetizmu',
  imageId: 'bE7W1ji'
}, {
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'hemičar',
  accomplishment: 'pionirski kortizon, steroidi i pilule za kontrolu rađanja',
  imageId: 'IOjWm71'
}, {
  id: 4,
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrofizičar',
  accomplishment: 'računanje mase belog patuljka',
  imageId: 'lrWQx8l'
}];
```

```js src/utils.js
export function getImageUrl(person) {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    's.jpg'
  );
}
```

```css
ul { list-style-type: none; padding: 0px 10px; }
li {
  margin-bottom: 10px;
  display: grid;
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
img { width: 100px; height: 100px; border-radius: 50%; }
```

</Sandpack>

U ovom rešenju, `map` pozivi su smešteni direktno unutar roditeljskih `<ul>` elemenata, ali možete uvesti promenljive za njih ako vam to deluje čitljivije.

I dalje postoji duplirani kod između renderovanih listi. Možete ići dalje i izdvojiti ponavljajuće delove u `<ListSection>` komponentu:

<Sandpack>

```js src/App.js
import { people } from './data.js';
import { getImageUrl } from './utils.js';

function ListSection({ title, people }) {
  return (
    <>
      <h2>{title}</h2>
      <ul>
        {people.map(person =>
          <li key={person.id}>
            <img
              src={getImageUrl(person)}
              alt={person.name}
            />
            <p>
              <b>{person.name}:</b>
              {' ' + person.profession + ' '}
              poznat je zbog {person.accomplishment}
            </p>
          </li>
        )}
      </ul>
    </>
  );
}

export default function List() {
  const chemists = people.filter(person =>
    person.profession === 'hemičar'
  );
  const everyoneElse = people.filter(person =>
    person.profession !== 'hemičar'
  );
  return (
    <article>
      <h1>Naučnici</h1>
      <ListSection
        title="Hemičari"
        people={chemists}
      />
      <ListSection
        title="Svi ostali"
        people={everyoneElse}
      />
    </article>
  );
}
```

```js src/data.js
export const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'matematičar',
  accomplishment: 'formula za svemirske letove',
  imageId: 'MK3eW3A'
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'hemičar',
  accomplishment: 'otkriće Arktičke rupe u ozonu',
  imageId: 'mynHUSa'
}, {
  id: 2,
  name: 'Mohammad Abdus Salam',
  profession: 'fizičar',
  accomplishment: 'teorija o elektromagnetizmu',
  imageId: 'bE7W1ji'
}, {
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'hemičar',
  accomplishment: 'pionirski kortizon, steroidi i pilule za kontrolu rađanja',
  imageId: 'IOjWm71'
}, {
  id: 4,
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrofizičar',
  accomplishment: 'računanje mase belog patuljka',
  imageId: 'lrWQx8l'
}];
```

```js src/utils.js
export function getImageUrl(person) {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    's.jpg'
  );
}
```

```css
ul { list-style-type: none; padding: 0px 10px; }
li {
  margin-bottom: 10px;
  display: grid;
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
img { width: 100px; height: 100px; border-radius: 50%; }
```

</Sandpack>

Veoma pažljiv čitalac može primetiti da sa dva `filter` poziva proveravamo profesiju svake osobe dvaput. Proveravanje polja je veoma brzo, pa je u ovom primeru u redu. Ako je vaša logika komplikovanija od toga, možete zameniti `filter` pozive sa petljom koja ručno pravi nizove i proverava svaku osobu jednom.

U suštini, ako se `people` nikad ne menja, možete pomeriti ovaj kod izvan komponente. Iz perspektive React-a, bitno je samo da mu na kraju date niz JSX čvorova. Njega ne zanima kako vi dobijate taj niz:

<Sandpack>

```js src/App.js
import { people } from './data.js';
import { getImageUrl } from './utils.js';

let chemists = [];
let everyoneElse = [];
people.forEach(person => {
  if (person.profession === 'hemičar') {
    chemists.push(person);
  } else {
    everyoneElse.push(person);
  }
});

function ListSection({ title, people }) {
  return (
    <>
      <h2>{title}</h2>
      <ul>
        {people.map(person =>
          <li key={person.id}>
            <img
              src={getImageUrl(person)}
              alt={person.name}
            />
            <p>
              <b>{person.name}:</b>
              {' ' + person.profession + ' '}
              poznat je zbog {person.accomplishment}
            </p>
          </li>
        )}
      </ul>
    </>
  );
}

export default function List() {
  return (
    <article>
      <h1>Naučnici</h1>
      <ListSection
        title="Hemičari"
        people={chemists}
      />
      <ListSection
        title="Svi ostali"
        people={everyoneElse}
      />
    </article>
  );
}
```

```js src/data.js
export const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'matematičar',
  accomplishment: 'formula za svemirske letove',
  imageId: 'MK3eW3A'
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'hemičar',
  accomplishment: 'otkriće Arktičke rupe u ozonu',
  imageId: 'mynHUSa'
}, {
  id: 2,
  name: 'Mohammad Abdus Salam',
  profession: 'fizičar',
  accomplishment: 'teorija o elektromagnetizmu',
  imageId: 'bE7W1ji'
}, {
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'hemičar',
  accomplishment: 'pionirski kortizon, steroidi i pilule za kontrolu rađanja',
  imageId: 'IOjWm71'
}, {
  id: 4,
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrofizičar',
  accomplishment: 'računanje mase belog patuljka',
  imageId: 'lrWQx8l'
}];
```

```js src/utils.js
export function getImageUrl(person) {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    's.jpg'
  );
}
```

```css
ul { list-style-type: none; padding: 0px 10px; }
li {
  margin-bottom: 10px;
  display: grid;
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
img { width: 100px; height: 100px; border-radius: 50%; }
```

</Sandpack>

</Solution>

#### Ugnježdene liste u jednoj komponenti {/*nested-lists-in-one-component*/}

Napravite listu recepata od ovog niza! Za svaki recept u nizu, prikažite mu ime kao `<h2>` i listu sastojaka u `<ul>`-u.

<Hint>

Ovo će zahtevati da ugnjezdite dva različita `map` poziva.

</Hint>

<Sandpack>

```js src/App.js
import { recipes } from './data.js';

export default function RecipeList() {
  return (
    <div>
      <h1>Recepti</h1>
    </div>
  );
}
```

```js src/data.js
export const recipes = [{
  id: 'greek-salad',
  name: 'Grčka salata',
  ingredients: ['paradajz', 'krastavci', 'crni luk', 'masline', 'feta']
}, {
  id: 'hawaiian-pizza',
  name: 'Havajska pica',
  ingredients: ['kora za picu', 'sos za picu', 'mocarela', 'šunka', 'ananas']
}, {
  id: 'hummus',
  name: 'Humus',
  ingredients: ['leblebija', 'maslinovo ulje', 'čen belog luka', 'limun', 'tahini']
}];
```

</Sandpack>

<Solution>

Evo jednog načina kako to možete uraditi:

<Sandpack>

```js src/App.js
import { recipes } from './data.js';

export default function RecipeList() {
  return (
    <div>
      <h1>Recepti</h1>
      {recipes.map(recipe =>
        <div key={recipe.id}>
          <h2>{recipe.name}</h2>
          <ul>
            {recipe.ingredients.map(ingredient =>
              <li key={ingredient}>
                {ingredient}
              </li>
            )}
          </ul>
        </div>
      )}
    </div>
  );
}
```

```js src/data.js
export const recipes = [{
  id: 'greek-salad',
  name: 'Grčka salata',
  ingredients: ['paradajz', 'krastavci', 'crni luk', 'masline', 'feta']
}, {
  id: 'hawaiian-pizza',
  name: 'Havajska pica',
  ingredients: ['kora za picu', 'sos za picu', 'mocarela', 'šunka', 'ananas']
}, {
  id: 'hummus',
  name: 'Humus',
  ingredients: ['leblebija', 'maslinovo ulje', 'čen belog luka', 'limun', 'tahini']
}];
```

</Sandpack>

Svaka stavka u `recipes` već uključuje `id` polje, tako da se to koristi kao `key` u spoljašnjoj petlji. Ne postoji ID koji možete koristiti da prolazite kroz sastojke. Međutim, razumno je pretpostaviti da isti sastojak neće biti dvaput nabrojan u jednom receptu, tako da njegovo ime može služiti kao `key`. Alternativno, možete promeniti strukturu podataka i dodati ID-eve, ili koristiti indeks kao `key` (uz upozorenje da ne možete sigurno menjati redosled sastojaka).

</Solution>

#### Izdvojiti stavku liste u komponentu {/*extracting-a-list-item-component*/}

Ova `RecipeList` komponenta sadrži dva ugnježdena `map` poziva. Da biste je pojednostavili, izdvojite `Recipe` komponentu iz nje koja će primiti `id`, `name` i `ingredients` props-e. Gde ćete smestiti spoljašni `key` i zašto?

<Sandpack>

```js src/App.js
import { recipes } from './data.js';

export default function RecipeList() {
  return (
    <div>
      <h1>Recepti</h1>
      {recipes.map(recipe =>
        <div key={recipe.id}>
          <h2>{recipe.name}</h2>
          <ul>
            {recipe.ingredients.map(ingredient =>
              <li key={ingredient}>
                {ingredient}
              </li>
            )}
          </ul>
        </div>
      )}
    </div>
  );
}
```

```js src/data.js
export const recipes = [{
  id: 'greek-salad',
  name: 'Grčka salata',
  ingredients: ['paradajz', 'krastavci', 'crni luk', 'masline', 'feta']
}, {
  id: 'hawaiian-pizza',
  name: 'Havajska pica',
  ingredients: ['kora za picu', 'sos za picu', 'mocarela', 'šunka', 'ananas']
}, {
  id: 'hummus',
  name: 'Humus',
  ingredients: ['leblebija', 'maslinovo ulje', 'čen belog luka', 'limun', 'tahini']
}];
```

</Sandpack>

<Solution>

Možete kopirati i nalepiti JSX iz spoljašnjeg `map`-a u novu `Recipe` komponentu i vratiti taj JSX. Onda, možete promeniti `recipe.name` u `name`, `recipe.id` u `id`, i tako dalje, prosleđujući ih kao props u `Recipe`:

<Sandpack>

```js
import { recipes } from './data.js';

function Recipe({ id, name, ingredients }) {
  return (
    <div>
      <h2>{name}</h2>
      <ul>
        {ingredients.map(ingredient =>
          <li key={ingredient}>
            {ingredient}
          </li>
        )}
      </ul>
    </div>
  );
}

export default function RecipeList() {
  return (
    <div>
      <h1>Recepti</h1>
      {recipes.map(recipe =>
        <Recipe {...recipe} key={recipe.id} />
      )}
    </div>
  );
}
```

```js src/data.js
export const recipes = [{
  id: 'greek-salad',
  name: 'Grčka salata',
  ingredients: ['paradajz', 'krastavci', 'crni luk', 'masline', 'feta']
}, {
  id: 'hawaiian-pizza',
  name: 'Havajska pica',
  ingredients: ['kora za picu', 'sos za picu', 'mocarela', 'šunka', 'ananas']
}, {
  id: 'hummus',
  name: 'Humus',
  ingredients: ['leblebija', 'maslinovo ulje', 'čen belog luka', 'limun', 'tahini']
}];
```

</Sandpack>

Ovde, `<Recipe {...recipe} key={recipe.id} />` je sintaksna skraćenica za "prosleđivanje svih polja `recipe` objekta kao props u `Recipe` komponentu". Možete napisati i svaki prop eksplicitno: `<Recipe id={recipe.id} name={recipe.name} ingredients={recipe.ingredients} key={recipe.id} />`.

**Primetite da je `key` specificiran na samom `<Recipe>` umesto na root `<div>` elementu vraćenom iz `Recipe`.** Razlog tome je zato što je ovaj `key` direktno potreban u kontekstu okružujućeg niza. Ranije ste imali niz `<div>`-ova pa je svakom od njih bio potreban `key`, ali sada imate niz `<Recipe>`-ova. Drugim rečima, kada izdvajate komponentu, ne zaboravite da `key` ostavite izvan JSX-a koji kopirate i nalepite.

</Solution>

#### Lista sa separatorom {/*list-with-a-separator*/}

Ovaj primer renderuje poznatu haiku koju je napisao Tachibana Hokushi, gde je svaka linija obmotana `<p>` tag-om. Vaš posao je da ubacite `<hr />` separator između paragrafa. Rezultat treba da izgleda ovako:

```js
<article>
  <p>I write, erase, rewrite</p>
  <hr />
  <p>Erase again, and then</p>
  <hr />
  <p>A poppy blooms.</p>
</article>
```

Haiku sadrži samo tri linije, ali vaše rešenje treba raditi za bilo koji broj linija. Zapazite da se `<hr />` elementi pojavljuju samo *između* `<p>` elemenata, ali ne na početku i kraju!

<Sandpack>

```js
const poem = {
  lines: [
    'I write, erase, rewrite',
    'Erase again, and then',
    'A poppy blooms.'
  ]
};

export default function Poem() {
  return (
    <article>
      {poem.lines.map((line, index) =>
        <p key={index}>
          {line}
        </p>
      )}
    </article>
  );
}
```

```css
body {
  text-align: center;
}
p {
  font-family: Georgia, serif;
  font-size: 20px;
  font-style: italic;
}
hr {
  margin: 0 120px 0 120px;
  border: 1px dashed #45c3d8;
}
```

</Sandpack>

(Ovo je redak primer gde je indeks kao ključ prihvatljiv, jer se redosled linija u pesmama nikad ne menja.)

<Hint>

Moraćete ili da pretvorite `map` u petlju, ili da koristite Fragment.

</Hint>

<Solution>

Možete ručno napisati petlju ubacivanjem `<hr />` i `<p>...</p>` u izlazni niz:

<Sandpack>

```js
const poem = {
  lines: [
    'I write, erase, rewrite',
    'Erase again, and then',
    'A poppy blooms.'
  ]
};

export default function Poem() {
  let output = [];

  // Popunite izlazni niz
  poem.lines.forEach((line, i) => {
    output.push(
      <hr key={i + '-separator'} />
    );
    output.push(
      <p key={i + '-text'}>
        {line}
      </p>
    );
  });
  // Obrišite prvi <hr />
  output.shift();

  return (
    <article>
      {output}
    </article>
  );
}
```

```css
body {
  text-align: center;
}
p {
  font-family: Georgia, serif;
  font-size: 20px;
  font-style: italic;
}
hr {
  margin: 0 120px 0 120px;
  border: 1px dashed #45c3d8;
}
```

</Sandpack>

Upotreba indeksa originalne linije za `key` više ne radi jer su i separatori i paragrafi sada u istom nizu. Međutim, svakom od njih možete dati jedinstven ključ upotrebom sufiksa, npr. `key={i + '-text'}`.

Alternativno, možete renderovati kolekciju Fragment-a koji sadrže `<hr />` i `<p>...</p>`. Međutim, `<>...</>` sintaksna skraćenica ne podržava prosleđivanje ključeva, pa morate napisati `<Fragment>` eksplicitno:

<Sandpack>

```js
import { Fragment } from 'react';

const poem = {
  lines: [
    'I write, erase, rewrite',
    'Erase again, and then',
    'A poppy blooms.'
  ]
};

export default function Poem() {
  return (
    <article>
      {poem.lines.map((line, i) =>
        <Fragment key={i}>
          {i > 0 && <hr />}
          <p>{line}</p>
        </Fragment>
      )}
    </article>
  );
}
```

```css
body {
  text-align: center;
}
p {
  font-family: Georgia, serif;
  font-size: 20px;
  font-style: italic;
}
hr {
  margin: 0 120px 0 120px;
  border: 1px dashed #45c3d8;
}
```

</Sandpack>

Upamtite, Fragment-i (češće napisani kao `<> </>`) vam omogućavaju da grupišete JSX čvorove bez dodavanja `<div>`-ova!

</Solution>

</Challenges>
