---
title: Dodavanje interaktivnosti
---

<Intro>

Neke stvari na ekranu se update-uju kao odgovor na korisnički input. Na primer, klik na galeriju slika menja aktivnu sliku. U React-u, podaci koji se menjaju tokom vremena nazivaju se *state.* Možete dodati state bilo kojoj component-i i update-ovati je po potrebi. U ovom poglavlju naučićete kako da pišete component-e koji obrađuju interakcije, update-uju svoj state i prikazuju različite output-e tokom vremena.

</Intro>

<YouWillLearn isChapter={true}>

* [Kako da rukujete event-ima koje pokrene korisnik](/learn/responding-to-events)
* [Kako da učinite da component-e „pamte” informacije koristeći state](/learn/state-a-components-memory)
* [Kako React update-uje UI u dve faze](/learn/render-and-commit)
* [Zašto se state ne update-uje odmah nakon promene](/learn/state-as-a-snapshot)
* [Kako da složite seriju state update-ova](/learn/queueing-a-series-of-state-updates)
* [Kako da update-ujete objekat u state-u](/learn/updating-objects-in-state)
* [Kako da update-ujete array u state-u](/learn/updating-arrays-in-state)

</YouWillLearn>

## Odgovaranje na event-e {/*responding-to-events*/}

React vam omogućava da dodate *event handler*-e u vaš JSX. Event handler-i su vaše sopstvene funkcije koje će se pokrenuti kao odgovor na korisničke interakcije poput klika, prelaženja mišem, fokusiranja na input-e forme i tako dalje.

Ugrađene component-e poput `<button>` podržavaju samo ugrađene browser event-e poput `onClick`. Međutim, možete kreirati i sopstvene component-e i njihovim event handler prop-ovima dati bilo koja aplikacijski specifična imena koja želite.

<Sandpack>

```js
export default function App() {
  return (
    <Toolbar
      onPlayMovie={() => alert('Puštanje!')}
      onUploadImage={() => alert('Upload-ovanje!')}
    />
  );
}

function Toolbar({ onPlayMovie, onUploadImage }) {
  return (
    <div>
      <Button onClick={onPlayMovie}>
        Pusti film
      </Button>
      <Button onClick={onUploadImage}>
        Upload-uj sliku
      </Button>
    </div>
  );
}

function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}
```

```css
button { margin-right: 10px; }
```

</Sandpack>

<LearnMore path="/learn/responding-to-events">

Pročitajte **[Odgovaranje na event-e](/learn/responding-to-events)** da biste naučili kako da dodate event handler-e.

</LearnMore>

## State: Memorija component-e {/*state-a-components-memory*/}

Component-e često moraju da promene ono što je prikazano na ekranu kao rezultat neke interakcije. Kucanje u formu treba da update-uje polje za unos, klik na "sledeće" u karuselu slika treba da promeni prikazanu sliku, klik na "kupi" stavlja proizvod u korpu. Component-e moraju da "pamte" određene stvari: trenutnu vrednost unosa, trenutnu sliku, korpu za kupovinu. U React-u, ova vrsta memorije specifična za component-e naziva se *state.*

Možete dodati state component-i pomoću Hook-a [`useState`](/reference/react/useState). *Hook-ovi* su specijalne funkcije koje omogućavaju vašim component-ama da koriste React funkcionalnosti (state je jedna od tih funkcionalnosti). Hook `useState` vam omogućava da deklarišete varijablu state-a. On prima inicijalni state i vraća par vrednosti: trenutni state i funkciju za setovanje state-a koja vam omogućava da ga update-ujete.

```js
const [index, setIndex] = useState(0);
const [showMore, setShowMore] = useState(false);
```

Here is how an image gallery uses and updates state on click:

<Sandpack>

```js
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);
  const hasNext = index < sculptureList.length - 1;

  function handleNextClick() {
    if (hasNext) {
      setIndex(index + 1);
    } else {
      setIndex(0);
    }
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleNextClick}>
        Next
      </button>
      <h2>
        <i>{sculpture.name} </i>
        by {sculpture.artist}
      </h2>
      <h3>
        ({index + 1} of {sculptureList.length})
      </h3>
      <button onClick={handleMoreClick}>
        {showMore ? 'Hide' : 'Show'} details
      </button>
      {showMore && <p>{sculpture.description}</p>}
      <img
        src={sculpture.url}
        alt={sculpture.alt}
      />
    </>
  );
}
```

```js src/data.js
export const sculptureList = [{
  name: 'Homenaje a la Neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: 'Although Colvin is predominantly known for abstract themes that allude to pre-Hispanic symbols, this gigantic sculpture, an homage to neurosurgery, is one of her most recognizable public art pieces.',
  url: 'https://i.imgur.com/Mx7dA2Y.jpg',
  alt: 'A bronze statue of two crossed hands delicately holding a human brain in their fingertips.'
}, {
  name: 'Floralis Genérica',
  artist: 'Eduardo Catalano',
  description: 'This enormous (75 ft. or 23m) silver flower is located in Buenos Aires. It is designed to move, closing its petals in the evening or when strong winds blow and opening them in the morning.',
  url: 'https://i.imgur.com/ZF6s192m.jpg',
  alt: 'A gigantic metallic flower sculpture with reflective mirror-like petals and strong stamens.'
}, {
  name: 'Eternal Presence',
  artist: 'John Woodrow Wilson',
  description: 'Wilson was known for his preoccupation with equality, social justice, as well as the essential and spiritual qualities of humankind. This massive (7ft. or 2,13m) bronze represents what he described as "a symbolic Black presence infused with a sense of universal humanity."',
  url: 'https://i.imgur.com/aTtVpES.jpg',
  alt: 'The sculpture depicting a human head seems ever-present and solemn. It radiates calm and serenity.'
}, {
  name: 'Moai',
  artist: 'Unknown Artist',
  description: 'Located on the Easter Island, there are 1,000 moai, or extant monumental statues, created by the early Rapa Nui people, which some believe represented deified ancestors.',
  url: 'https://i.imgur.com/RCwLEoQm.jpg',
  alt: 'Three monumental stone busts with the heads that are disproportionately large with somber faces.'
}, {
  name: 'Blue Nana',
  artist: 'Niki de Saint Phalle',
  description: 'The Nanas are triumphant creatures, symbols of femininity and maternity. Initially, Saint Phalle used fabric and found objects for the Nanas, and later on introduced polyester to achieve a more vibrant effect.',
  url: 'https://i.imgur.com/Sd1AgUOm.jpg',
  alt: 'A large mosaic sculpture of a whimsical dancing female figure in a colorful costume emanating joy.'
}, {
  name: 'Ultimate Form',
  artist: 'Barbara Hepworth',
  description: 'This abstract bronze sculpture is a part of The Family of Man series located at Yorkshire Sculpture Park. Hepworth chose not to create literal representations of the world but developed abstract forms inspired by people and landscapes.',
  url: 'https://i.imgur.com/2heNQDcm.jpg',
  alt: 'A tall sculpture made of three elements stacked on each other reminding of a human figure.'
}, {
  name: 'Cavaliere',
  artist: 'Lamidi Olonade Fakeye',
  description: "Descended from four generations of woodcarvers, Fakeye's work blended traditional and contemporary Yoruba themes.",
  url: 'https://i.imgur.com/wIdGuZwm.png',
  alt: 'An intricate wood sculpture of a warrior with a focused face on a horse adorned with patterns.'
}, {
  name: 'Big Bellies',
  artist: 'Alina Szapocznikow',
  description: "Szapocznikow is known for her sculptures of the fragmented body as a metaphor for the fragility and impermanence of youth and beauty. This sculpture depicts two very realistic large bellies stacked on top of each other, each around five feet (1,5m) tall.",
  url: 'https://i.imgur.com/AlHTAdDm.jpg',
  alt: 'The sculpture reminds a cascade of folds, quite different from bellies in classical sculptures.'
}, {
  name: 'Terracotta Army',
  artist: 'Unknown Artist',
  description: 'The Terracotta Army is a collection of terracotta sculptures depicting the armies of Qin Shi Huang, the first Emperor of China. The army consisted of more than 8,000 soldiers, 130 chariots with 520 horses, and 150 cavalry horses.',
  url: 'https://i.imgur.com/HMFmH6m.jpg',
  alt: '12 terracotta sculptures of solemn warriors, each with a unique facial expression and armor.'
}, {
  name: 'Lunar Landscape',
  artist: 'Louise Nevelson',
  description: 'Nevelson was known for scavenging objects from New York City debris, which she would later assemble into monumental constructions. In this one, she used disparate parts like a bedpost, juggling pin, and seat fragment, nailing and gluing them into boxes that reflect the influence of Cubism’s geometric abstraction of space and form.',
  url: 'https://i.imgur.com/rN7hY6om.jpg',
  alt: 'A black matte sculpture where the individual elements are initially indistinguishable.'
}, {
  name: 'Aureole',
  artist: 'Ranjani Shettar',
  description: 'Shettar merges the traditional and the modern, the natural and the industrial. Her art focuses on the relationship between man and nature. Her work was described as compelling both abstractly and figuratively, gravity defying, and a "fine synthesis of unlikely materials."',
  url: 'https://i.imgur.com/okTpbHhm.jpg',
  alt: 'A pale wire-like sculpture mounted on concrete wall and descending on the floor. It appears light.'
}, {
  name: 'Hippos',
  artist: 'Taipei Zoo',
  description: 'The Taipei Zoo commissioned a Hippo Square featuring submerged hippos at play.',
  url: 'https://i.imgur.com/6o5Vuyu.jpg',
  alt: 'A group of bronze hippo sculptures emerging from the sett sidewalk as if they were swimming.'
}];
```

```css
h2 { margin-top: 10px; margin-bottom: 0; }
h3 {
 margin-top: 5px;
 font-weight: normal;
 font-size: 100%;
}
img { width: 120px; height: 120px; }
button {
  display: block;
  margin-top: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

<LearnMore path="/learn/state-a-components-memory">

Pročitajte **[State: Memorija Component-e](/learn/state-a-components-memory)** kako biste naučili kako da zapamtite vrednost i update-ujete je prilikom interakcije.

</LearnMore>

## Render i commit {/*render-and-commit*/}

Pre nego što se vaše component-e prikažu na ekranu, React mora da ih renderuje. Razumevanje koraka u ovom procesu pomoći će vam da razmislite o tome kako se vaš kod izvršava i da objasnite njegovo ponašanje.

Zamislite da su vaše component-e kuvari u kuhinji, koji pripremaju ukusna jela od sastojaka. U ovom scenariju, React je konobar koji prenosi porudžbine od gostiju i donosi im njihova jela. Ovaj proces poručivanja i posluživanja UI-a ima tri koraka:

1. **Pokretanje** rendera (prenos porudžbine gosta u kuhinju)
2. **Renderovanje** component-e (priprema porudžbine u kuhinji)
3. **Commit-ovanje** u DOM (postavljanje porudžbine na sto)

<IllustrationBlock sequential>
  <Illustration caption="Pokretanje" alt="React kao konobar u restoranu, prenosi porudžbine od korisnika i dostavlja ih u Kitchen Component-u." src="/images/docs/illustrations/i_render-and-commit1.png" />
  <Illustration caption="Render" alt="Šef kuhinje za Card daje React-u svežu Card component-u." src="/images/docs/illustrations/i_render-and-commit2.png" />
  <Illustration caption="Commit" alt="React donosi Card korisniku za njegov sto." src="/images/docs/illustrations/i_render-and-commit3.png" />
</IllustrationBlock>

<LearnMore path="/learn/render-and-commit">

Pročitajte **[Render i Commit](/learn/render-and-commit)** kako biste naučili lifecycle jednog UI update-a.

</LearnMore>

## State kao snapshot {/*state-as-a-snapshot*/}

Za razliku od običnih JavaScript varijabli, React state se ponaša više kao snapshot. Njegovo postavljanje ne menja već postojeću state varijablu, već pokreće ponovni render. Ovo može biti iznenađujuće na početku!

```js
console.log(count);  // 0
setCount(count + 1); // Zatraži ponovni render sa 1
console.log(count);  // I dalje 0!
```

Ovo ponašanje vam pomaže da izbegnete suptilne greške. Evo male aplikacije za ćaskanje. Pokušajte da pogodite šta će se desiti ako prvo pritisnete "Pošalji", a *zatim* promenite primaoca na Bob. Čije ime će se pojaviti u `alert` pet sekundi kasnije?

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [to, setTo] = useState('Alice');
  const [message, setMessage] = useState('Hello');

  function handleSubmit(e) {
    e.preventDefault();
    setTimeout(() => {
      alert(`Poruka ${message} za ${to}`);
    }, 5000);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        To:{' '}
        <select
          value={to}
          onChange={e => setTo(e.target.value)}>
          <option value="Alice">Alice</option>
          <option value="Bob">Bob</option>
        </select>
      </label>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}
```

```css
label, textarea { margin-bottom: 10px; display: block; }
```

</Sandpack>

<LearnMore path="/learn/state-as-a-snapshot">

Pročitajte **[State kao snapshot](/learn/state-as-a-snapshot)** da biste saznali zašto state izgleda "fiksno" i nepromenljivo unutar event handler-a.

</LearnMore>

## Redosled serijskih ažuriranja state-a {/*queueing-a-series-of-state-updates*/}

Ova component-a ima grešku: klikom na dugme "+3" rezultat se povećava samo jednom.

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [score, setScore] = useState(0);

  function increment() {
    setScore(score + 1);
  }

  return (
    <>
      <button onClick={() => increment()}>+1</button>
      <button onClick={() => {
        increment();
        increment();
        increment();
      }}>+3</button>
      <h1>Score: {score}</h1>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
```

</Sandpack>

[State kao snapshot](/learn/state-as-a-snapshot) objašnjava zašto se ovo dešava. Postavljanje state-a traži novo render-ovanje, ali ne menja njegovu vrednost u kodu koji se već izvršava. Tako `score` nastavlja da ima vrednost `0` odmah nakon što pozovete `setScore(score + 1)`.

```js
console.log(score);  // 0
setScore(score + 1); // setScore(0 + 1);
console.log(score);  // 0
setScore(score + 1); // setScore(0 + 1);
console.log(score);  // 0
setScore(score + 1); // setScore(0 + 1);
console.log(score);  // 0
```

Možete ovo popraviti prosleđivanjem *updater funkcije* kada postavljate state. Obratite pažnju kako zamena `setScore(score + 1)` sa `setScore(s => s + 1)` rešava problem dugmeta "+3". Ovo vam omogućava da redom dodate više update-ova state-a u redosled.

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [score, setScore] = useState(0);

  function increment() {
    setScore(s => s + 1);
  }

  return (
    <>
      <button onClick={() => increment()}>+1</button>
      <button onClick={() => {
        increment();
        increment();
        increment();
      }}>+3</button>
      <h1>Score: {score}</h1>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
```

</Sandpack>

<LearnMore path="/learn/queueing-a-series-of-state-updates">

Pročitajte **[Redosled serijskih ažuriranja state-a](/learn/queueing-a-series-of-state-updates)** kako biste naučili kako da postavite redosled update-ova state-a.

</LearnMore>

## Ažuriranje objekata u state-u {/*updating-objects-in-state*/}

State može sadržati bilo koju vrstu JavaScript vrednosti, uključujući i objekte. Međutim, ne bi trebalo direktno menjati objekte i array-e koje držite u React state-u. Umesto toga, kada želite da ažurirate objekat ili array, potrebno je da kreirate novi (ili napravite kopiju postojećeg) i zatim ažurirate state kako bi koristio tu kopiju.

Obično ćete koristiti `...` spread sintaksu za kopiranje objekata i array-a koje želite da promenite. Na primer, update-ovanje ugnježdenog objekta može izgledati ovako:

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg',
      image: 'https://i.imgur.com/Sd1AgUOm.jpg',
    }
  });

  function handleNameChange(e) {
    setPerson({
      ...person,
      name: e.target.value
    });
  }

  function handleTitleChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        title: e.target.value
      }
    });
  }

  function handleCityChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        city: e.target.value
      }
    });
  }

  function handleImageChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        image: e.target.value
      }
    });
  }

  return (
    <>
      <label>
        Naziv:
        <input
          value={person.name}
          onChange={handleNameChange}
        />
      </label>
      <label>
        Naslov:
        <input
          value={person.artwork.title}
          onChange={handleTitleChange}
        />
      </label>
      <label>
        Grad:
        <input
          value={person.artwork.city}
          onChange={handleCityChange}
        />
      </label>
      <label>
        Slika:
        <input
          value={person.artwork.image}
          onChange={handleImageChange}
        />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {' napravio/la '}
        {person.name}
        <br />
        (locirano u {person.artwork.city})
      </p>
      <img
        src={person.artwork.image}
        alt={person.artwork.title}
      />
    </>
  );
}
```

```css
label { display: block; }
input { margin-left: 5px; margin-bottom: 5px; }
img { width: 200px; height: 200px; }
```

</Sandpack>

Ako kopiranje objekata u kodu postane zamorno, možete koristiti biblioteku poput [Immer-a](https://github.com/immerjs/use-immer) kako biste smanjili količinu ponavljajućeg koda:

<Sandpack>

```js
import { useImmer } from 'use-immer';

export default function Form() {
  const [person, updatePerson] = useImmer({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg',
      image: 'https://i.imgur.com/Sd1AgUOm.jpg',
    }
  });

  function handleNameChange(e) {
    updatePerson(draft => {
      draft.name = e.target.value;
    });
  }

  function handleTitleChange(e) {
    updatePerson(draft => {
      draft.artwork.title = e.target.value;
    });
  }

  function handleCityChange(e) {
    updatePerson(draft => {
      draft.artwork.city = e.target.value;
    });
  }

  function handleImageChange(e) {
    updatePerson(draft => {
      draft.artwork.image = e.target.value;
    });
  }

  return (
    <>
      <label>
        Naziv:
        <input
          value={person.name}
          onChange={handleNameChange}
        />
      </label>
      <label>
        Naslov:
        <input
          value={person.artwork.title}
          onChange={handleTitleChange}
        />
      </label>
      <label>
        Grad:
        <input
          value={person.artwork.city}
          onChange={handleCityChange}
        />
      </label>
      <label>
        Slika:
        <input
          value={person.artwork.image}
          onChange={handleImageChange}
        />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {' napravio/la '}
        {person.name}
        <br />
        (locirano u {person.artwork.city})
      </p>
      <img
        src={person.artwork.image}
        alt={person.artwork.title}
      />
    </>
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

```css
label { display: block; }
input { margin-left: 5px; margin-bottom: 5px; }
img { width: 200px; height: 200px; }
```

</Sandpack>

<LearnMore path="/learn/updating-objects-in-state">

Pročitajte **[Ažuriranje objekata u state-u](/learn/updating-objects-in-state)** kako biste naučili kako pravilno ažurirati objekte.

</LearnMore>

## Ažuriranje nizova u state-u {/*updating-arrays-in-state*/}

Nizovi su još jedan tip promenljivih JavaScript objekata koje možete čuvati u state-u i koje treba tretirati kao read-only. Kao i kod objekata, kada želite da ažurirate niz koji se nalazi u state-u, potrebno je da kreirate novi (ili napravite kopiju postojećeg), a zatim postavite state da koristi taj novi niz:

<Sandpack>

```js
import { useState } from 'react';

const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [list, setList] = useState(
    initialList
  );

  function handleToggle(artworkId, nextSeen) {
    setList(list.map(artwork => {
      if (artwork.id === artworkId) {
        return { ...artwork, seen: nextSeen };
      } else {
        return artwork;
      }
    }));
  }

  return (
    <>
      <h1>Art Bucket List</h1>
      <h2>My list of art to see:</h2>
      <ItemList
        artworks={list}
        onToggle={handleToggle} />
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

Ako pravljenje kopija nizova u kodu postane zamorno, možete koristiti biblioteku kao što je [Immer](https://github.com/immerjs/use-immer) da smanjite ponavljanje koda:

<Sandpack>

```js
import { useState } from 'react';
import { useImmer } from 'use-immer';

const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [list, updateList] = useImmer(initialList);

  function handleToggle(artworkId, nextSeen) {
    updateList(draft => {
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
        artworks={list}
        onToggle={handleToggle} />
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

<LearnMore path="/learn/updating-arrays-in-state">

Pročitajte **[Ažuriranje nizova u state-u](/learn/updating-arrays-in-state)** da biste naučili kako pravilno da ažurirate nizove.

</LearnMore>

## Šta je sledeće? {/*whats-next*/}

Pređite na [Reagovanje na Event-e](/learn/responding-to-events) da biste počeli da čitate ovo poglavlje stranicu po stranicu!

Ili, ako ste već upoznati sa ovim temama, zašto ne biste pročitali [Upravljanje state-om](/learn/managing-state)?
