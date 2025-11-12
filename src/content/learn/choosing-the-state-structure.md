---
title: Odabir strukture state-a
---

<Intro>

Pravilno strukturiranje state-a može napraviti razliku između komponente koju je lako menjati i debug-ovati, i one koja je stalan izvor bug-ova. Ovde se nalazi par saveta koje trebate razmotriti kada strukturirate state.

</Intro>

<YouWillLearn>

* Kada da koristite jednu ili više state promenljivih
* Šta izbegavati prilikom organizacije state-a
* Kako popraviti česte probleme sa strukturom state-a

</YouWillLearn>

## Principi za strukturiranje state-a {/*principles-for-structuring-state*/}

Kada pišete komponentu koja sadrži neki state, moraćete da odlučite koliko state promenljivih da koristite i kakav oblik podataka u njima vam je potreban. Iako je moguće da napišete pravilan program sa neoptimizovanom strukturom state-a, postoji par principa koji vas mogu uputiti da donesete bolje odluke:

1. **Grupisati povezane state-ove.** Ako uvek istovremeno ažurirate dve ili više state promenljivih, pokušajte da ih spojite u jednu state promenljivu.
2. **Izbegavati kontradikcije u state-u.** Kada je state strukturiran tako da više delova state-a budu kontradiktorni i "neodgovarajući" jedni drugima, ostavljate prostora za greške. Pokušajte ovo da izbegnete.
3. **Izbegavati suvišan state.** Ako, tokom renderovanja, neku informaciju možete izračunati na osnovu props-a komponente ili već postojeće state promenljive, ne biste trebali da stavite tu informaciju u state komponente.
4. **Izbegavati dupliranje u state-u.** Kada su isti podaci duplirani u više state promenljivih, ili unutar ugnježdenih objekata, teško je sinhronizovati ih. Smanjite dupliranje kada to možete.
5. **Izbegavati duboko ugnježedeni state.** Hijerarhijski dubok state nije zgodan za ažuriranje. Kada je moguće, preferirajte da strukturirate state da bude flat.

Cilj iza ovih principa je da *napravite da se state lako ažurira bez uvođenja grešaka*. Uklanjanje suvišnih i dupliranih podataka iz state-a pomaže da svi njegovi delovi ostanu sinhronizovani. Ovo je slično onome kako inženjeri baza podataka žele da ["normalizuju" strukturu baze podataka](https://docs.microsoft.com/en-us/office/troubleshoot/access/database-normalization-description) i smanje šanse za bug-ove. Da parafraziramo Alberta Ajnštajna, **"Napravite vaš state što jednostavnijim--ali ne jednostavnijim od toga."**

Hajde da vidimo primenu ovih principa na delu.

## Grupisati povezane state-ove {/*group-related-state*/}

Ponekad se možete dvoumiti između upotrebe jedne ili više state promenljivih.

Da li koristiti ovo?

```js
const [x, setX] = useState(0);
const [y, setY] = useState(0);
```

Ili ovo?

```js
const [position, setPosition] = useState({ x: 0, y: 0 });
```

Tehnički, možete koristiti oba pristupa. Ali, **ako se dve state promenljive uvek menjaju zajedno, može biti dobra ideja da ih grupišete u jednu state promenljivu**. Tako nećete zaboravati da ih držite sinhronizovane, kao u ovom primeru gde pomeranje kursora ažurira obe koordinate crvene tačke:

<Sandpack>

```js
import { useState } from 'react';

export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });
  return (
    <div
      onPointerMove={e => {
        setPosition({
          x: e.clientX,
          y: e.clientY
        });
      }}
      style={{
        position: 'relative',
        width: '100vw',
        height: '100vh',
      }}>
      <div style={{
        position: 'absolute',
        backgroundColor: 'red',
        borderRadius: '50%',
        transform: `translate(${position.x}px, ${position.y}px)`,
        left: -10,
        top: -10,
        width: 20,
        height: 20,
      }} />
    </div>
  )
}
```

```css
body { margin: 0; padding: 0; height: 250px; }
```

</Sandpack>

Drugi slučaj gde možete grupisati podatke u objekat ili niz je kada ne znate koliko state-ova vam treba. Na primer, korisno je kada imate formu gde korisnik može uneti polja po sopstvenoj želji.

<Pitfall>

Ako vam je state promenljiva objekat, zapamtite da [ne možete ažurirati samo jedno polje u njemu](/learn/updating-objects-in-state) bez eksplicitnog kopiranja ostalih polja. Na primer, ne možete napisati `setPosition({ x: 100 })` u primeru iznad jer nemate `y` polje uopšte! Umesto toga, ako želite da postavite samo `x`, napisaćete `setPosition({ ...position, x: 100 })`, ili ćete ih podeliti u dve state promenljive i napisati `setX(100)`.

</Pitfall>

## Izbegavati kontradikcije u state-u {/*avoid-contradictions-in-state*/}

Ovde je feedback forma za hotel sa `isSending` i `isSent` state promenljivama:

<Sandpack>

```js
import { useState } from 'react';

export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  const [isSent, setIsSent] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsSending(true);
    await sendMessage(text);
    setIsSending(false);
    setIsSent(true);
  }

  if (isSent) {
    return <h1>Hvala za feedback!</h1>
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>Kako vam je bilo u The Prancing Pony hotelu?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button
        disabled={isSending}
        type="submit"
      >
        Pošalji
      </button>
      {isSending && <p>Slanje...</p>}
    </form>
  );
}

// Pretvaraj se da šalješ poruku.
function sendMessage(text) {
  return new Promise(resolve => {
    setTimeout(resolve, 2000);
  });
}
```

</Sandpack>

Iako ovaj kod radi, ostavlja prostor za "nemoguća" stanja. Na primer, ako zaboravite da pozovete `setIsSent` i `setIsSending` zajedno, možete završiti u situaciji gde su i `isSending` i `isSent` postavljeni na `true` istovremeno. Što je vaša komponenta kompleksnija, teže je razumeti šta se desilo.

**Pošto `isSending` i `isSent` nikad ne bi trebali da budu `true` istovremeno, bolje je zameniti ih sa jednom `status` state promenljivom koja može imati jedno od *tri* validna stanja:** `'typing'` (inicijalno), `'sending'` i `'sent'`:

<Sandpack>

```js
import { useState } from 'react';

export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [status, setStatus] = useState('typing');

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('sending');
    await sendMessage(text);
    setStatus('sent');
  }

  const isSending = status === 'sending';
  const isSent = status === 'sent';

  if (isSent) {
    return <h1>Hvala za feedback!</h1>
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>Kako vam je bilo u The Prancing Pony hotelu?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button
        disabled={isSending}
        type="submit"
      >
        Pošalji
      </button>
      {isSending && <p>Slanje...</p>}
    </form>
  );
}

// Pretvaraj se da šalješ poruku.
function sendMessage(text) {
  return new Promise(resolve => {
    setTimeout(resolve, 2000);
  });
}
```

</Sandpack>

Možete deklarisati konstante da poboljšate čitljivost:

```js
const isSending = status === 'sending';
const isSent = status === 'sent';
```

Ali, one nisu state promenljive, pa ne morate brinuti o tome da li će biti sinhronizovane.

## Izbegavati suvišan state {/*avoid-redundant-state*/}

Ako, tokom renderovanja, neku informaciju možete izračunati na osnovu props-a komponente ili već postojeće state promenljive, **ne biste** trebali da stavite tu informaciju u state komponente.

Na primer, pogledajte ovu formu. Radi, ali, da li možete pronaći neki suvišan state u njoj?

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState('');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + ' ' + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + ' ' + e.target.value);
  }

  return (
    <>
      <h2>Prijavite se</h2>
      <label>
        Ime:{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Prezime:{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        Vaša karta će biti izdata na ime: <b>{fullName}</b>
      </p>
    </>
  );
}
```

```css
label { display: block; margin-bottom: 5px; }
```

</Sandpack>

Ova forma ima tri state promenljive: `firstName`, `lastName` i `fullName`. Međutim, `fullName` je suvišno. **Uvek možete izračunati `fullName` pomoću `firstName` i `lastName` tokom rendera, pa je uklonite iz state-a.**

Evo kako to da uradite:

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  const fullName = firstName + ' ' + lastName;

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }

  return (
    <>
      <h2>Prijavite se</h2>
      <label>
        Ime:{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Prezime:{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        Vaša karta će biti izdata na ime: <b>{fullName}</b>
      </p>
    </>
  );
}
```

```css
label { display: block; margin-bottom: 5px; }
```

</Sandpack>

Ovde, `fullName` *nije* state promenljiva. Umesto toga, računa se tokom rendera:

```js
const fullName = firstName + ' ' + lastName;
```

Kao rezultat, handler-i za promenu ne moraju ništa posebno da rade da bi je ažurirali. Kada pozovete `setFirstName` ili `setLastName`, pokrećete ponovni render, a onda će naredni `fullName` biti izračunat na osnovu najnovijih podataka.

<DeepDive>

#### Ne preslikavajte props-e u state {/*don-t-mirror-props-in-state*/}

Uobičajen primer suvišnog state-a je ovakav kod:

```js
function Message({ messageColor }) {
  const [color, setColor] = useState(messageColor);
```

Ovde, `color` state promenljiva je inicijalizovana na `messageColor` prop. Problem je u tome što **ako roditeljska komponenta prosledi drugu vrednost za `messageColor` kasnije (na primer, `'red'` umesto `'blue'`), `color` *state promenljiva* neće biti ažurirana**! State je inicijalizovan samo tokom prvog rendera.

Zato "preslikavanje" nekog prop-a u state promenljivu može dovesti do zabune. Umesto toga, koristite `messageColor` prop direktno u kodu. Ako želite da mu date kraće ime, koristite konstantu:

```js
function Message({ messageColor }) {
  const color = messageColor;
```

Na ovaj način neće ostati nesinhronzovan sa prop-om prosleđenim iz roditeljske komponente.

"Preslikavanje" props-a u state ima smisla jedino ako *želite* da ignorišete sve promene tog prop-a. Po konvenciji, nazovite prop tako da počinje sa `initial` ili `default` da ukažete na to da će nove vrednosti biti ignorisane:

```js
function Message({ initialColor }) {
  // `color` state promenljiva drži *prvu* vrednost `initialColor`-a.
  // Naredne promene `initialColor` prop-a su ignorisane.
  const [color, setColor] = useState(initialColor);
```

</DeepDive>

## Izbegavati dupliranje u state-u {/*avoid-duplication-in-state*/}

Ova komponenta vam omogućava da odaberete jednu grickalicu za put od nekoliko ponuđenih:

<Sandpack>

```js
import { useState } from 'react';

const initialItems = [
  { title: 'perece', id: 0 },
  { title: 'hrskave morske alge', id: 1 },
  { title: 'musli pločica', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(
    items[0]
  );

  return (
    <>
      <h2>Koja je vaša grickalica za put?</h2>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.title}
            {' '}
            <button onClick={() => {
              setSelectedItem(item);
            }}>Izaberi</button>
          </li>
        ))}
      </ul>
      <p>Izabrali ste {selectedItem.title}.</p>
    </>
  );
}
```

```css
button { margin-top: 10px; }
```

</Sandpack>

Trenutno se čuva izabrana stavka kao objekat u `selectedItem` state promenljivoj. Međutim, ovo nije dobro: **sadržaj `selectedItem`-a je isti objekat koji se nalazi unutar stavke u `items` listi**. Ovo znači da je informacija o stavki duplirana na dva mesta.

Zašto je ovo problem? Hajde da dodamo izmenu stavki:

<Sandpack>

```js
import { useState } from 'react';

const initialItems = [
  { title: 'perece', id: 0 },
  { title: 'hrskave morske alge', id: 1 },
  { title: 'musli pločica', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(
    items[0]
  );

  function handleItemChange(id, e) {
    setItems(items.map(item => {
      if (item.id === id) {
        return {
          ...item,
          title: e.target.value,
        };
      } else {
        return item;
      }
    }));
  }

  return (
    <>
      <h2>Koja je vaša grickalica za put?</h2> 
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={e => {
                handleItemChange(item.id, e)
              }}
            />
            {' '}
            <button onClick={() => {
              setSelectedItem(item);
            }}>Izaberi</button>
          </li>
        ))}
      </ul>
      <p>Izabrali ste {selectedItem.title}.</p>
    </>
  );
}
```

```css
button { margin-top: 10px; }
```

</Sandpack>

Primetite da ako prvo kliknete "Izaberi", a *onda* izmenite stavku, **input se ažurira, ali labela na dnu ne prikazuje unete promene**. To je zato što imate dupliran state, a zaboravili ste da ažurirate `selectedItem`.

Iako možete ažurirati i `selectedItem` takođe, lakše je ukloniti dupliranje. U ovom primeru, umesto `selectedItem` objekta (koji kreira dupliranje sa objektima unutar `items` liste), vi čuvate `selectedId` u state-u, a *onda* `selectedItem` dobijate pretraživanjem `items`-a preko tog ID-a:

<Sandpack>

```js
import { useState } from 'react';

const initialItems = [
  { title: 'perece', id: 0 },
  { title: 'hrskave morske alge', id: 1 },
  { title: 'musli pločica', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedId, setSelectedId] = useState(0);

  const selectedItem = items.find(item =>
    item.id === selectedId
  );

  function handleItemChange(id, e) {
    setItems(items.map(item => {
      if (item.id === id) {
        return {
          ...item,
          title: e.target.value,
        };
      } else {
        return item;
      }
    }));
  }

  return (
    <>
      <h2>Koja je vaša grickalica za put?</h2>
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={e => {
                handleItemChange(item.id, e)
              }}
            />
            {' '}
            <button onClick={() => {
              setSelectedId(item.id);
            }}>Izaberi</button>
          </li>
        ))}
      </ul>
      <p>Izabrali ste {selectedItem.title}.</p>
    </>
  );
}
```

```css
button { margin-top: 10px; }
```

</Sandpack>

State koji je dupliran izgleda ovako:

* `items = [{ id: 0, title: 'perece'}, ...]`
* `selectedItem = {id: 0, title: 'perece'}`

Ali, nakon promene, sada izgleda ovako:

* `items = [{ id: 0, title: 'perece'}, ...]`
* `selectedId = 0`

Dupliranja nema, a ostaje samo obavezan state!

Sada, ako izmenite *izabranu* stavku, poruka ispod će se odmah ažurirati. To se dešava jer `setItems` pokreće ponovni render, a `items.find(...)` će pronaći stavku sa ažuriranim nazivom. Nije vam potrebno da čuvate *izabranu stavku* u state-u, zato što je jedino *izabrani ID* obavezan. Ostatak može biti izračunat tokom rendera.

## Izbegavati duboko ugnježedeni state {/*avoid-deeply-nested-state*/}

Zamislite plan putovanja koji sadrži planete, kontinente i države. Možete biti u iskušenju da strukturirate state upotrebom ugnježdenih objekata i nizova, kao u ovom primeru:

<Sandpack>

```js
import { useState } from 'react';
import { initialTravelPlan } from './places.js';

function PlaceTree({ place }) {
  const childPlaces = place.childPlaces;
  return (
    <li>
      {place.title}
      {childPlaces.length > 0 && (
        <ol>
          {childPlaces.map(place => (
            <PlaceTree key={place.id} place={place} />
          ))}
        </ol>
      )}
    </li>
  );
}

export default function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);
  const planets = plan.childPlaces;
  return (
    <>
      <h2>Mesta za posetu</h2>
      <ol>
        {planets.map(place => (
          <PlaceTree key={place.id} place={place} />
        ))}
      </ol>
    </>
  );
}
```

```js src/places.js active
export const initialTravelPlan = {
  id: 0,
  title: '(Root)',
  childPlaces: [{
    id: 1,
    title: 'Zemlja',
    childPlaces: [{
      id: 2,
      title: 'Afrika',
      childPlaces: [{
        id: 3,
        title: 'Bocvana',
        childPlaces: []
      }, {
        id: 4,
        title: 'Egipat',
        childPlaces: []
      }, {
        id: 5,
        title: 'Kenija',
        childPlaces: []
      }, {
        id: 6,
        title: 'Madagaskar',
        childPlaces: []
      }, {
        id: 7,
        title: 'Maroko',
        childPlaces: []
      }, {
        id: 8,
        title: 'Nigerija',
        childPlaces: []
      }, {
        id: 9,
        title: 'Južna Afrika',
        childPlaces: []
      }]
    }, {
      id: 10,
      title: 'Amerika',
      childPlaces: [{
        id: 11,
        title: 'Argentina',
        childPlaces: []
      }, {
        id: 12,
        title: 'Brazil',
        childPlaces: []
      }, {
        id: 13,
        title: 'Barbados',
        childPlaces: []
      }, {
        id: 14,
        title: 'Kanada',
        childPlaces: []
      }, {
        id: 15,
        title: 'Jamajka',
        childPlaces: []
      }, {
        id: 16,
        title: 'Meksiko',
        childPlaces: []
      }, {
        id: 17,
        title: 'Trinidad i Tobago',
        childPlaces: []
      }, {
        id: 18,
        title: 'Venecuela',
        childPlaces: []
      }]
    }, {
      id: 19,
      title: 'Azija',
      childPlaces: [{
        id: 20,
        title: 'Kina',
        childPlaces: []
      }, {
        id: 21,
        title: 'Indija',
        childPlaces: []
      }, {
        id: 22,
        title: 'Singapur',
        childPlaces: []
      }, {
        id: 23,
        title: 'Južna Koreja',
        childPlaces: []
      }, {
        id: 24,
        title: 'Tajland',
        childPlaces: []
      }, {
        id: 25,
        title: 'Vijetnam',
        childPlaces: []
      }]
    }, {
      id: 26,
      title: 'Evropa',
      childPlaces: [{
        id: 27,
        title: 'Hrvatska',
        childPlaces: [],
      }, {
        id: 28,
        title: 'Francuska',
        childPlaces: [],
      }, {
        id: 29,
        title: 'Nemačka',
        childPlaces: [],
      }, {
        id: 30,
        title: 'Italija',
        childPlaces: [],
      }, {
        id: 31,
        title: 'Portugal',
        childPlaces: [],
      }, {
        id: 32,
        title: 'Španija',
        childPlaces: [],
      }, {
        id: 33,
        title: 'Turska',
        childPlaces: [],
      }]
    }, {
      id: 34,
      title: 'Okeanija',
      childPlaces: [{
        id: 35,
        title: 'Australija',
        childPlaces: [],
      }, {
        id: 36,
        title: 'Bora Bora (Francuska Polinezija)',
        childPlaces: [],
      }, {
        id: 37,
        title: 'Uskršnje ostrvo (Čile)',
        childPlaces: [],
      }, {
        id: 38,
        title: 'Fidži',
        childPlaces: [],
      }, {
        id: 39,
        title: 'Havaji (SAD)',
        childPlaces: [],
      }, {
        id: 40,
        title: 'Novi Zeland',
        childPlaces: [],
      }, {
        id: 41,
        title: 'Vanuatu',
        childPlaces: [],
      }]
    }]
  }, {
    id: 42,
    title: 'Mesec',
    childPlaces: [{
      id: 43,
      title: 'Rheita krater',
      childPlaces: []
    }, {
      id: 44,
      title: 'Piccolomini krater',
      childPlaces: []
    }, {
      id: 45,
      title: 'Tihov krater',
      childPlaces: []
    }]
  }, {
    id: 46,
    title: 'Mars',
    childPlaces: [{
      id: 47,
      title: 'Corn Town',
      childPlaces: []
    }, {
      id: 48,
      title: 'Green Hill',
      childPlaces: []      
    }]
  }]
};
```

</Sandpack>

Recimo da želite dodati dugme za brisanje mesta koje ste već posetili. Kako biste to uradili? [Ažuriranje ugnježdenog state-a](/learn/updating-objects-in-state#updating-a-nested-object) podrazumeva pravljenje kopija svih objekata od mesta koje se promenilo na gore. Brisanje duboko ugnježdenog mesta bi zahtevalo kopiranje celokupnog lanca roditeljskih objekata. Takav kod može biti veoma opširan.

**Ako je state previše ugnježden da bi se lako ažurirao, razmotrite da ga napravite da bude "flat".** Ovde je jedan način da restrukturirate ove podatke. Umesto strukture nalik na stablo gde svaki `place` ima niz *dečjih mesta*, možete napraviti da svako mesto čuva niz *ID-eva dečjih mesta*. Onda napravite mapiranje od ID-a mesta do odgovarajućeg mesta.

Ovo restrukturiranje podataka vas može podsetiti na tabelu u bazi podataka:

<Sandpack>

```js
import { useState } from 'react';
import { initialTravelPlan } from './places.js';

function PlaceTree({ id, placesById }) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      {childIds.length > 0 && (
        <ol>
          {childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              placesById={placesById}
            />
          ))}
        </ol>
      )}
    </li>
  );
}

export default function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);
  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Mesta za posetu</h2>
      <ol>
        {planetIds.map(id => (
          <PlaceTree
            key={id}
            id={id}
            placesById={plan}
          />
        ))}
      </ol>
    </>
  );
}
```

```js src/places.js active
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 42, 46],
  },
  1: {
    id: 1,
    title: 'Zemlja',
    childIds: [2, 10, 19, 26, 34]
  },
  2: {
    id: 2,
    title: 'Afrika',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  }, 
  3: {
    id: 3,
    title: 'Bocvana',
    childIds: []
  },
  4: {
    id: 4,
    title: 'Egipat',
    childIds: []
  },
  5: {
    id: 5,
    title: 'Kenija',
    childIds: []
  },
  6: {
    id: 6,
    title: 'Madagaskar',
    childIds: []
  }, 
  7: {
    id: 7,
    title: 'Maroko',
    childIds: []
  },
  8: {
    id: 8,
    title: 'Nigerija',
    childIds: []
  },
  9: {
    id: 9,
    title: 'Južna Afrika',
    childIds: []
  },
  10: {
    id: 10,
    title: 'Amerika',
    childIds: [11, 12, 13, 14, 15, 16, 17, 18],   
  },
  11: {
    id: 11,
    title: 'Argentina',
    childIds: []
  },
  12: {
    id: 12,
    title: 'Brazil',
    childIds: []
  },
  13: {
    id: 13,
    title: 'Barbados',
    childIds: []
  }, 
  14: {
    id: 14,
    title: 'Kanada',
    childIds: []
  },
  15: {
    id: 15,
    title: 'Jamajka',
    childIds: []
  },
  16: {
    id: 16,
    title: 'Meksiko',
    childIds: []
  },
  17: {
    id: 17,
    title: 'Trinidad i Tobago',
    childIds: []
  },
  18: {
    id: 18,
    title: 'Venecuela',
    childIds: []
  },
  19: {
    id: 19,
    title: 'Azija',
    childIds: [20, 21, 22, 23, 24, 25],   
  },
  20: {
    id: 20,
    title: 'Kina',
    childIds: []
  },
  21: {
    id: 21,
    title: 'Indija',
    childIds: []
  },
  22: {
    id: 22,
    title: 'Singapur',
    childIds: []
  },
  23: {
    id: 23,
    title: 'Južna Koreja',
    childIds: []
  },
  24: {
    id: 24,
    title: 'Tajland',
    childIds: []
  },
  25: {
    id: 25,
    title: 'Vijetnam',
    childIds: []
  },
  26: {
    id: 26,
    title: 'Evropa',
    childIds: [27, 28, 29, 30, 31, 32, 33],   
  },
  27: {
    id: 27,
    title: 'Hrvatska',
    childIds: []
  },
  28: {
    id: 28,
    title: 'Francuska',
    childIds: []
  },
  29: {
    id: 29,
    title: 'Nemačka',
    childIds: []
  },
  30: {
    id: 30,
    title: 'Italija',
    childIds: []
  },
  31: {
    id: 31,
    title: 'Portugal',
    childIds: []
  },
  32: {
    id: 32,
    title: 'Španija',
    childIds: []
  },
  33: {
    id: 33,
    title: 'Turska',
    childIds: []
  },
  34: {
    id: 34,
    title: 'Okeanija',
    childIds: [35, 36, 37, 38, 39, 40, 41],   
  },
  35: {
    id: 35,
    title: 'Australija',
    childIds: []
  },
  36: {
    id: 36,
    title: 'Bora Bora (Francuska Polinezija)',
    childIds: []
  },
  37: {
    id: 37,
    title: 'Uskršnje ostrvo (Čile)',
    childIds: []
  },
  38: {
    id: 38,
    title: 'Fidži',
    childIds: []
  },
  39: {
    id: 40,
    title: 'Havaji (SAD)',
    childIds: []
  },
  40: {
    id: 40,
    title: 'Novi Zeland',
    childIds: []
  },
  41: {
    id: 41,
    title: 'Vanuatu',
    childIds: []
  },
  42: {
    id: 42,
    title: 'Mesec',
    childIds: [43, 44, 45]
  },
  43: {
    id: 43,
    title: 'Rheita krater',
    childIds: []
  },
  44: {
    id: 44,
    title: 'Piccolomini krater',
    childIds: []
  },
  45: {
    id: 45,
    title: 'Tihov krater',
    childIds: []
  },
  46: {
    id: 46,
    title: 'Mars',
    childIds: [47, 48]
  },
  47: {
    id: 47,
    title: 'Corn Town',
    childIds: []
  },
  48: {
    id: 48,
    title: 'Green Hill',
    childIds: []
  }
};
```

</Sandpack>

**Sada, kada je state "flat" (poznato i pod nazivom "normalizovano"), ažuriranje ugnježdenih stavki postaje lakše.**

Da biste uklonili mesto, potrebno je ažurirati dva nivoa state-a:

- Ažurirana verzija njegovog *roditeljskog* mesta treba da isključi uklonjeni ID iz `childIds` niza.
- Ažurirana verzija root "tabela" objekta treba da uključi ažuriranu verziju roditeljskog mesta.

Ovde je primer kako to možete uraditi:

<Sandpack>

```js
import { useState } from 'react';
import { initialTravelPlan } from './places.js';

export default function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);

  function handleComplete(parentId, childId) {
    const parent = plan[parentId];
    // Kreiraj novu verziju roditeljskog mesta
    // koji ne sadrži ovaj ID deteta.
    const nextParent = {
      ...parent,
      childIds: parent.childIds
        .filter(id => id !== childId)
    };
    // Ažuriraj root state objekat...
    setPlan({
      ...plan,
      // ...tako da ima ažuriranog roditelja.
      [parentId]: nextParent
    });
  }

  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Mesta za posetu</h2>
      <ol>
        {planetIds.map(id => (
          <PlaceTree
            key={id}
            id={id}
            parentId={0}
            placesById={plan}
            onComplete={handleComplete}
          />
        ))}
      </ol>
    </>
  );
}

function PlaceTree({ id, parentId, placesById, onComplete }) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      <button onClick={() => {
        onComplete(parentId, id);
      }}>
        Kompetiraj
      </button>
      {childIds.length > 0 &&
        <ol>
          {childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              parentId={id}
              placesById={placesById}
              onComplete={onComplete}
            />
          ))}
        </ol>
      }
    </li>
  );
}
```

```js src/places.js
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 42, 46],
  },
  1: {
    id: 1,
    title: 'Zemlja',
    childIds: [2, 10, 19, 26, 34]
  },
  2: {
    id: 2,
    title: 'Afrika',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  }, 
  3: {
    id: 3,
    title: 'Bocvana',
    childIds: []
  },
  4: {
    id: 4,
    title: 'Egipat',
    childIds: []
  },
  5: {
    id: 5,
    title: 'Kenija',
    childIds: []
  },
  6: {
    id: 6,
    title: 'Madagaskar',
    childIds: []
  }, 
  7: {
    id: 7,
    title: 'Maroko',
    childIds: []
  },
  8: {
    id: 8,
    title: 'Nigerija',
    childIds: []
  },
  9: {
    id: 9,
    title: 'Južna Afrika',
    childIds: []
  },
  10: {
    id: 10,
    title: 'Amerika',
    childIds: [11, 12, 13, 14, 15, 16, 17, 18],   
  },
  11: {
    id: 11,
    title: 'Argentina',
    childIds: []
  },
  12: {
    id: 12,
    title: 'Brazil',
    childIds: []
  },
  13: {
    id: 13,
    title: 'Barbados',
    childIds: []
  }, 
  14: {
    id: 14,
    title: 'Kanada',
    childIds: []
  },
  15: {
    id: 15,
    title: 'Jamajka',
    childIds: []
  },
  16: {
    id: 16,
    title: 'Meksiko',
    childIds: []
  },
  17: {
    id: 17,
    title: 'Trinidad i Tobago',
    childIds: []
  },
  18: {
    id: 18,
    title: 'Venecuela',
    childIds: []
  },
  19: {
    id: 19,
    title: 'Azija',
    childIds: [20, 21, 22, 23, 24, 25],   
  },
  20: {
    id: 20,
    title: 'Kina',
    childIds: []
  },
  21: {
    id: 21,
    title: 'Indija',
    childIds: []
  },
  22: {
    id: 22,
    title: 'Singapur',
    childIds: []
  },
  23: {
    id: 23,
    title: 'Južna Koreja',
    childIds: []
  },
  24: {
    id: 24,
    title: 'Tajland',
    childIds: []
  },
  25: {
    id: 25,
    title: 'Vijetnam',
    childIds: []
  },
  26: {
    id: 26,
    title: 'Evropa',
    childIds: [27, 28, 29, 30, 31, 32, 33],   
  },
  27: {
    id: 27,
    title: 'Hrvatska',
    childIds: []
  },
  28: {
    id: 28,
    title: 'Francuska',
    childIds: []
  },
  29: {
    id: 29,
    title: 'Nemačka',
    childIds: []
  },
  30: {
    id: 30,
    title: 'Italija',
    childIds: []
  },
  31: {
    id: 31,
    title: 'Portugal',
    childIds: []
  },
  32: {
    id: 32,
    title: 'Španija',
    childIds: []
  },
  33: {
    id: 33,
    title: 'Turska',
    childIds: []
  },
  34: {
    id: 34,
    title: 'Okeanija',
    childIds: [35, 36, 37, 38, 39, 40, 41],   
  },
  35: {
    id: 35,
    title: 'Australija',
    childIds: []
  },
  36: {
    id: 36,
    title: 'Bora Bora (Francuska Polinezija)',
    childIds: []
  },
  37: {
    id: 37,
    title: 'Uskršnje ostrvo (Čile)',
    childIds: []
  },
  38: {
    id: 38,
    title: 'Fidži',
    childIds: []
  },
  39: {
    id: 39,
    title: 'Havaji (SAD)',
    childIds: []
  },
  40: {
    id: 40,
    title: 'Novi Zeland',
    childIds: []
  },
  41: {
    id: 41,
    title: 'Vanuatu',
    childIds: []
  },
  42: {
    id: 42,
    title: 'Mesec',
    childIds: [43, 44, 45]
  },
  43: {
    id: 43,
    title: 'Rheita krater',
    childIds: []
  },
  44: {
    id: 44,
    title: 'Piccolomini krater',
    childIds: []
  },
  45: {
    id: 45,
    title: 'Tihov krater',
    childIds: []
  },
  46: {
    id: 46,
    title: 'Mars',
    childIds: [47, 48]
  },
  47: {
    id: 47,
    title: 'Corn Town',
    childIds: []
  },
  48: {
    id: 48,
    title: 'Green Hill',
    childIds: []
  }
};
```

```css
button { margin: 10px; }
```

</Sandpack>

Možete ugnjezditi state koliko god želite, ali pravljenje "flat" state-a može rešiti dosta problema. Čini da ažuriranje state-a bude lakše, a takođe i osigurava da nemate dupliranje u različitim delovima ugnježdenog objekta.

<DeepDive>

#### Poboljšanje upotrebe memorije {/*improving-memory-usage*/}

Idealno, takođe bi uklanjali obrisane stavke (i njihovu decu!) iz "tabela" objekta da biste poboljšali upotrebu memorije. Ova verzija to radi. Takođe [koristi Immer](/learn/updating-objects-in-state#write-concise-update-logic-with-immer) kako bi učinila logiku ažuriranja konciznijom.

<Sandpack>

```js
import { useImmer } from 'use-immer';
import { initialTravelPlan } from './places.js';

export default function TravelPlan() {
  const [plan, updatePlan] = useImmer(initialTravelPlan);

  function handleComplete(parentId, childId) {
    updatePlan(draft => {
      // Ukloni iz ID-eva roditeljskog mesta.
      const parent = draft[parentId];
      parent.childIds = parent.childIds
        .filter(id => id !== childId);

      // Zaboravi ovo mesto i sva njegova podstabla.
      deleteAllChildren(childId);
      function deleteAllChildren(id) {
        const place = draft[id];
        place.childIds.forEach(deleteAllChildren);
        delete draft[id];
      }
    });
  }

  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Mesta za posetu</h2>
      <ol>
        {planetIds.map(id => (
          <PlaceTree
            key={id}
            id={id}
            parentId={0}
            placesById={plan}
            onComplete={handleComplete}
          />
        ))}
      </ol>
    </>
  );
}

function PlaceTree({ id, parentId, placesById, onComplete }) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      <button onClick={() => {
        onComplete(parentId, id);
      }}>
        Kompetiraj
      </button>
      {childIds.length > 0 &&
        <ol>
          {childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              parentId={id}
              placesById={placesById}
              onComplete={onComplete}
            />
          ))}
        </ol>
      }
    </li>
  );
}
```

```js src/places.js
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 42, 46],
  },
  1: {
    id: 1,
    title: 'Zemlja',
    childIds: [2, 10, 19, 26, 34]
  },
  2: {
    id: 2,
    title: 'Afrika',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  }, 
  3: {
    id: 3,
    title: 'Bocvana',
    childIds: []
  },
  4: {
    id: 4,
    title: 'Egipat',
    childIds: []
  },
  5: {
    id: 5,
    title: 'Kenija',
    childIds: []
  },
  6: {
    id: 6,
    title: 'Madagaskar',
    childIds: []
  }, 
  7: {
    id: 7,
    title: 'Maroko',
    childIds: []
  },
  8: {
    id: 8,
    title: 'Nigerija',
    childIds: []
  },
  9: {
    id: 9,
    title: 'Južna Afrika',
    childIds: []
  },
  10: {
    id: 10,
    title: 'Amerika',
    childIds: [11, 12, 13, 14, 15, 16, 17, 18],   
  },
  11: {
    id: 11,
    title: 'Argentina',
    childIds: []
  },
  12: {
    id: 12,
    title: 'Brazil',
    childIds: []
  },
  13: {
    id: 13,
    title: 'Barbados',
    childIds: []
  }, 
  14: {
    id: 14,
    title: 'Kanada',
    childIds: []
  },
  15: {
    id: 15,
    title: 'Jamajka',
    childIds: []
  },
  16: {
    id: 16,
    title: 'Meksiko',
    childIds: []
  },
  17: {
    id: 17,
    title: 'Trinidad i Tobago',
    childIds: []
  },
  18: {
    id: 18,
    title: 'Venecuela',
    childIds: []
  },
  19: {
    id: 19,
    title: 'Azija',
    childIds: [20, 21, 22, 23, 24, 25,],   
  },
  20: {
    id: 20,
    title: 'Kina',
    childIds: []
  },
  21: {
    id: 21,
    title: 'Indija',
    childIds: []
  },
  22: {
    id: 22,
    title: 'Singapur',
    childIds: []
  },
  23: {
    id: 23,
    title: 'Južna Koreja',
    childIds: []
  },
  24: {
    id: 24,
    title: 'Tajland',
    childIds: []
  },
  25: {
    id: 25,
    title: 'Vijetnam',
    childIds: []
  },
  26: {
    id: 26,
    title: 'Evropa',
    childIds: [27, 28, 29, 30, 31, 32, 33],   
  },
  27: {
    id: 27,
    title: 'Hrvatska',
    childIds: []
  },
  28: {
    id: 28,
    title: 'Francuska',
    childIds: []
  },
  29: {
    id: 29,
    title: 'Nemačka',
    childIds: []
  },
  30: {
    id: 30,
    title: 'Italija',
    childIds: []
  },
  31: {
    id: 31,
    title: 'Portugal',
    childIds: []
  },
  32: {
    id: 32,
    title: 'Španija',
    childIds: []
  },
  33: {
    id: 33,
    title: 'Turska',
    childIds: []
  },
  34: {
    id: 34,
    title: 'Okeanija',
    childIds: [35, 36, 37, 38, 39, 40, 41],   
  },
  35: {
    id: 35,
    title: 'Australija',
    childIds: []
  },
  36: {
    id: 36,
    title: 'Bora Bora (Francuska Polinezija)',
    childIds: []
  },
  37: {
    id: 37,
    title: 'Uskršnje ostrvo (Čile)',
    childIds: []
  },
  38: {
    id: 38,
    title: 'Fidži',
    childIds: []
  },
  39: {
    id: 39,
    title: 'Havaji (SAD)',
    childIds: []
  },
  40: {
    id: 40,
    title: 'Novi Zeland',
    childIds: []
  },
  41: {
    id: 41,
    title: 'Vanuatu',
    childIds: []
  },
  42: {
    id: 42,
    title: 'Mesec',
    childIds: [43, 44, 45]
  },
  43: {
    id: 43,
    title: 'Rheita krater',
    childIds: []
  },
  44: {
    id: 44,
    title: 'Piccolomini krater',
    childIds: []
  },
  45: {
    id: 45,
    title: 'Tihov krater',
    childIds: []
  },
  46: {
    id: 46,
    title: 'Mars',
    childIds: [47, 48]
  },
  47: {
    id: 47,
    title: 'Corn Town',
    childIds: []
  },
  48: {
    id: 48,
    title: 'Green Hill',
    childIds: []
  }
};
```

```css
button { margin: 10px; }
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

</DeepDive>

Ponekad, možete smanjiti ugnježdenost state-a pomeranjem nekih ugnježdenih objekata u dečje komponente. Ovo radi dobro za kratkotrajne UI state-ove koji ne moraju biti čuvani, kao što je podatak da li se prelazi mišem iznad neke stavke.

<Recap>

* Ako se dve state promenljive uvek istovremeno ažuriraju, pokušajte da ih spojite u jednu.
* Pažljivo birajte state promenljive da biste izbegli kreiranje "nemogućih" stanja.
* Strukturirajte state na način koji smanjuje šanse za pravljenje grešaka prilikom ažuriranja.
* Izbegavajte suvišan i dupliran state kako ne biste morali da ih sinhronizujete.
* Ne stavljajte props *unutar* state-a osim ako specifično želite da sprečite ažuriranja.
* Za UI šablone poput odabira, u state-u čuvajte ID ili indeks umesto celog objekta.
* Ako je ažuriranje duboko ugnježdenog state-a komplikovano, probajte da ga flatten-ujete.

</Recap>

<Challenges>

#### Popraviti komponentu koja se ne ažurira {/*fix-a-component-thats-not-updating*/}

Ova `Clock` komponenta prima dva props-a: `color` i `time`. Kada izaberete drugu boju iz dropdown-a, `Clock` komponenta primi drugačiji `color` prop iz svoje roditeljske komponente. Međutim, iz nekog razloga, prikazana boja se ne ažurira. Zašto? Popravite problem.

<Sandpack>

```js src/Clock.js active
import { useState } from 'react';

export default function Clock(props) {
  const [color, setColor] = useState(props.color);
  return (
    <h1 style={{ color: color }}>
      {props.time}
    </h1>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  const [color, setColor] = useState('lightcoral');
  return (
    <div>
      <p>
        Izaberi boju:{' '}
        <select value={color} onChange={e => setColor(e.target.value)}>
          <option value="lightcoral">lightcoral</option>
          <option value="midnightblue">midnightblue</option>
          <option value="rebeccapurple">rebeccapurple</option>
        </select>
      </p>
      <Clock color={color} time={time.toLocaleTimeString()} />
    </div>
  );
}
```

</Sandpack>

<Solution>

Problem je u tome što ova komponenta ima `color` state inicijalizovan sa inicijalnom vrednošću `color` prop-a. Ali, kada se `color` prop promeni, to ne utiče na state promenljivu! Tako da one postaju nesinhronzovane. Da biste popravili problem, potpuno uklonite state promenljivu i koristite `color` prop direktno.

<Sandpack>

```js src/Clock.js active
import { useState } from 'react';

export default function Clock(props) {
  return (
    <h1 style={{ color: props.color }}>
      {props.time}
    </h1>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  const [color, setColor] = useState('lightcoral');
  return (
    <div>
      <p>
        Izaberi boju:{' '}
        <select value={color} onChange={e => setColor(e.target.value)}>
          <option value="lightcoral">lightcoral</option>
          <option value="midnightblue">midnightblue</option>
          <option value="rebeccapurple">rebeccapurple</option>
        </select>
      </p>
      <Clock color={color} time={time.toLocaleTimeString()} />
    </div>
  );
}
```

</Sandpack>

Ili, upotrebom sintakse dekonstruisanja:

<Sandpack>

```js src/Clock.js active
import { useState } from 'react';

export default function Clock({ color, time }) {
  return (
    <h1 style={{ color: color }}>
      {time}
    </h1>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  const [color, setColor] = useState('lightcoral');
  return (
    <div>
      <p>
        Izaberi boju:{' '}
        <select value={color} onChange={e => setColor(e.target.value)}>
          <option value="lightcoral">lightcoral</option>
          <option value="midnightblue">midnightblue</option>
          <option value="rebeccapurple">rebeccapurple</option>
        </select>
      </p>
      <Clock color={color} time={time.toLocaleTimeString()} />
    </div>
  );
}
```

</Sandpack>

</Solution>

#### Popraviti listu za pakovanje {/*fix-a-broken-packing-list*/}

Ova lista za pakovanje ima footer koji prikazuje koliko stavki je spakovano i koliko ukupno stavki postoji. Na prvu deluje da radi, ali je bug-ovita. Na primer, ako odaberete stavku kao spakovanu, a onda je obrišete, brojač neće biti ažuriran korektno. Popravite brojač tako da je uvek tačan.

<Hint>

Da li je neki state u ovom primeru suvišan?

</Hint>

<Sandpack>

```js src/App.js
import { useState } from 'react';
import AddItem from './AddItem.js';
import PackingList from './PackingList.js';

let nextId = 3;
const initialItems = [
  { id: 0, title: 'Tople čarape', packed: true },
  { id: 1, title: 'Dnevnik putovanja', packed: false },
  { id: 2, title: 'Vodene bojice', packed: false },
];

export default function TravelPlan() {
  const [items, setItems] = useState(initialItems);
  const [total, setTotal] = useState(3);
  const [packed, setPacked] = useState(1);

  function handleAddItem(title) {
    setTotal(total + 1);
    setItems([
      ...items,
      {
        id: nextId++,
        title: title,
        packed: false
      }
    ]);
  }

  function handleChangeItem(nextItem) {
    if (nextItem.packed) {
      setPacked(packed + 1);
    } else {
      setPacked(packed - 1);
    }
    setItems(items.map(item => {
      if (item.id === nextItem.id) {
        return nextItem;
      } else {
        return item;
      }
    }));
  }

  function handleDeleteItem(itemId) {
    setTotal(total - 1);
    setItems(
      items.filter(item => item.id !== itemId)
    );
  }

  return (
    <>  
      <AddItem
        onAddItem={handleAddItem}
      />
      <PackingList
        items={items}
        onChangeItem={handleChangeItem}
        onDeleteItem={handleDeleteItem}
      />
      <hr />
      <b>Spakovano {packed} od {total}!</b>
    </>
  );
}
```

```js src/AddItem.js hidden
import { useState } from 'react';

export default function AddItem({ onAddItem }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="Dodaj stavku"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddItem(title);
      }}>Dodaj</button>
    </>
  )
}
```

```js src/PackingList.js hidden
import { useState } from 'react';

export default function PackingList({
  items,
  onChangeItem,
  onDeleteItem
}) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          <label>
            <input
              type="checkbox"
              checked={item.packed}
              onChange={e => {
                onChangeItem({
                  ...item,
                  packed: e.target.checked
                });
              }}
            />
            {' '}
            {item.title}
          </label>
          <button onClick={() => onDeleteItem(item.id)}>
            Obriši
          </button>
        </li>
      ))}
    </ul>
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

Iako možete pažljivo da promenite svaki event handler da pravilno ažurira `total` i `packed` brojače, izvorni problem je to što te state promenljive uopšte postoje. One su suvišne jer uvek možete izračunati broj stavki (spakovanih i ukupnih) na osnovu `items` niza. Uklonite suvišan state da popravite bug:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import AddItem from './AddItem.js';
import PackingList from './PackingList.js';

let nextId = 3;
const initialItems = [
  { id: 0, title: 'Tople čarape', packed: true },
  { id: 1, title: 'Dnevnik putovanja', packed: false },
  { id: 2, title: 'Vodene bojice', packed: false },
];

export default function TravelPlan() {
  const [items, setItems] = useState(initialItems);

  const total = items.length;
  const packed = items
    .filter(item => item.packed)
    .length;

  function handleAddItem(title) {
    setItems([
      ...items,
      {
        id: nextId++,
        title: title,
        packed: false
      }
    ]);
  }

  function handleChangeItem(nextItem) {
    setItems(items.map(item => {
      if (item.id === nextItem.id) {
        return nextItem;
      } else {
        return item;
      }
    }));
  }

  function handleDeleteItem(itemId) {
    setItems(
      items.filter(item => item.id !== itemId)
    );
  }

  return (
    <>  
      <AddItem
        onAddItem={handleAddItem}
      />
      <PackingList
        items={items}
        onChangeItem={handleChangeItem}
        onDeleteItem={handleDeleteItem}
      />
      <hr />
      <b>Spakovano {packed} od {total}!</b>
    </>
  );
}
```

```js src/AddItem.js hidden
import { useState } from 'react';

export default function AddItem({ onAddItem }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="Dodaj stavku"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddItem(title);
      }}>Dodaj</button>
    </>
  )
}
```

```js src/PackingList.js hidden
import { useState } from 'react';

export default function PackingList({
  items,
  onChangeItem,
  onDeleteItem
}) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          <label>
            <input
              type="checkbox"
              checked={item.packed}
              onChange={e => {
                onChangeItem({
                  ...item,
                  packed: e.target.checked
                });
              }}
            />
            {' '}
            {item.title}
          </label>
          <button onClick={() => onDeleteItem(item.id)}>
            Obriši
          </button>
        </li>
      ))}
    </ul>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

</Sandpack>

Primetite kako event handler-i brinu jedino o pozivanju `setItems` nakon ove promene. Brojači stavki se sada računaju tokom narednog rendera na osnovu `items`-a, tako da su uvek tačni.

</Solution>

#### Popraviti nestajući odabir {/*fix-the-disappearing-selection*/}

Ovde je lista `letters` u state-u. Kada pismo dobije fokus, ili pređete mišem preko njega, postaje istaknuto. Trenutno istaknuto pismo se čuva u `highlightedLetter` state promenljivoj. Možete da "lajkujete" ili "uklonite lajk" za svako pismo, što ažurira `letters` niz u state-u.

Ovaj kod radi, ali postoji mali UI glitch. Kada kliknete "Lajkuj" ili "Ukloni lajk", isticanje nestane za trenutak. Međutim, ponovo se pojavi čim pomerite miša ili odaberete drugo pismo pomoću tastature. Zašto se ovo dešava? Popravite ovo tako da isticanje ne nestaje nakon klika na dugme.

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { initialLetters } from './data.js';
import Letter from './Letter.js';

export default function MailClient() {
  const [letters, setLetters] = useState(initialLetters);
  const [highlightedLetter, setHighlightedLetter] = useState(null);

  function handleHover(letter) {
    setHighlightedLetter(letter);
  }

  function handleStar(starred) {
    setLetters(letters.map(letter => {
      if (letter.id === starred.id) {
        return {
          ...letter,
          isStarred: !letter.isStarred
        };
      } else {
        return letter;
      }
    }));
  }

  return (
    <>
      <h2>Inbox</h2>
      <ul>
        {letters.map(letter => (
          <Letter
            key={letter.id}
            letter={letter}
            isHighlighted={
              letter === highlightedLetter
            }
            onHover={handleHover}
            onToggleStar={handleStar}
          />
        ))}
      </ul>
    </>
  );
}
```

```js src/Letter.js
export default function Letter({
  letter,
  isHighlighted,
  onHover,
  onToggleStar,
}) {
  return (
    <li
      className={
        isHighlighted ? 'highlighted' : ''
      }
      onFocus={() => {
        onHover(letter);        
      }}
      onPointerMove={() => {
        onHover(letter);
      }}
    >
      <button onClick={() => {
        onToggleStar(letter);
      }}>
        {letter.isStarred ? 'Ukloni lajk' : 'Lajkuj'}
      </button>
      {letter.subject}
    </li>
  )
}
```

```js src/data.js
export const initialLetters = [{
  id: 0,
  subject: 'Spremni za avanturu?',
  isStarred: true,
}, {
  id: 1,
  subject: 'Vreme za prijavu!',
  isStarred: false,
}, {
  id: 2,
  subject: 'Festival počinje za samo SEDAM dana!',
  isStarred: false,
}];
```

```css
button { margin: 5px; }
li { border-radius: 5px; }
.highlighted { background: #d2eaff; }
```

</Sandpack>

<Solution>

Problem je u tome što čuvate objekat pisma u `highlightedLetter`. Ali, takođe, čuvate istu informaciju u `letters` nizu. Tako da vaš state ima dupliranje! Kada ažurirate `letters` niz nakon klika na dugme, pravite novi objekat pisma koji se razlikuje od `highlightedLetter`. Zbog toga `highlightedLetter === letter` provera postaje `false`, a isticanje nestaje. Ponovo se pojavljuje naredni put kada pozovete `setHighlightedLetter` nakon što pomerite miša.

Da biste popravili problem, uklonite dupliranje iz state-a. Umesto da čuvate *samo pismo* na dva mesta, čuvajte umesto toga `highlightedId`. Onda možete proveriti `isHighlighted` za svako pismo pomoću `letter.id === highlightedId`, što će raditi čak iako se `letter` objekat promeni nakon poslednjeg rendera.

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { initialLetters } from './data.js';
import Letter from './Letter.js';

export default function MailClient() {
  const [letters, setLetters] = useState(initialLetters);
  const [highlightedId, setHighlightedId ] = useState(null);

  function handleHover(letterId) {
    setHighlightedId(letterId);
  }

  function handleStar(starredId) {
    setLetters(letters.map(letter => {
      if (letter.id === starredId) {
        return {
          ...letter,
          isStarred: !letter.isStarred
        };
      } else {
        return letter;
      }
    }));
  }

  return (
    <>
      <h2>Inbox</h2>
      <ul>
        {letters.map(letter => (
          <Letter
            key={letter.id}
            letter={letter}
            isHighlighted={
              letter.id === highlightedId
            }
            onHover={handleHover}
            onToggleStar={handleStar}
          />
        ))}
      </ul>
    </>
  );
}
```

```js src/Letter.js
export default function Letter({
  letter,
  isHighlighted,
  onHover,
  onToggleStar,
}) {
  return (
    <li
      className={
        isHighlighted ? 'highlighted' : ''
      }
      onFocus={() => {
        onHover(letter.id);        
      }}
      onPointerMove={() => {
        onHover(letter.id);
      }}
    >
      <button onClick={() => {
        onToggleStar(letter.id);
      }}>
        {letter.isStarred ? 'Ukloni lajk' : 'Lajkuj'}
      </button>
      {letter.subject}
    </li>
  )
}
```

```js src/data.js
export const initialLetters = [{
  id: 0,
  subject: 'Spremni za avanturu?',
  isStarred: true,
}, {
  id: 1,
  subject: 'Vreme za prijavu!',
  isStarred: false,
}, {
  id: 2,
  subject: 'Festival počinje za samo SEDAM dana!',
  isStarred: false,
}];
```

```css
button { margin: 5px; }
li { border-radius: 5px; }
.highlighted { background: #d2eaff; }
```

</Sandpack>

</Solution>

#### Implementirati višestruki izbor {/*implement-multiple-selection*/}

U ovom primeru, svaki `Letter` ima `isSelected` prop i `onToggle` handler koji označava da li je izabrano ili ne. Ovo radi, ali state se čuva kao `selectedId` (ili `null` ili ID), što znači da samo jedno pismo može biti izabrano.

Promenite strukturu state-a da podržava višestruki izbor. (Kako to strukturirati? Razmislite o ovome pre pisanja koda.) Svaki checkbox bi trebao da bude nezavistan od ostalih. Klik na izabrano pismo bi trebao da poništi odabir. Konačno, footer treba prikazivati tačan broj izabranih stavki.

<Hint>

Umesto jednog izabranog ID-a, poželećete da čuvate niz ili [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set) izabranih ID-eva u state-u.

</Hint>

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { letters } from './data.js';
import Letter from './Letter.js';

export default function MailClient() {
  const [selectedId, setSelectedId] = useState(null);

  // TODO: dozvoli višestruki izbor
  const selectedCount = 1;

  function handleToggle(toggledId) {
    // TODO: dozvoli višestruki izbor
    setSelectedId(toggledId);
  }

  return (
    <>
      <h2>Inbox</h2>
      <ul>
        {letters.map(letter => (
          <Letter
            key={letter.id}
            letter={letter}
            isSelected={
              // TODO: dozvoli višestruki izbor
              letter.id === selectedId
            }
            onToggle={handleToggle}
          />
        ))}
        <hr />
        <p>
          <b>
            Izabrali ste {selectedCount} pisama
          </b>
        </p>
      </ul>
    </>
  );
}
```

```js src/Letter.js
export default function Letter({
  letter,
  onToggle,
  isSelected,
}) {
  return (
    <li className={
      isSelected ? 'selected' : ''
    }>
      <label>
        <input
          type="checkbox"
          checked={isSelected}
          onChange={() => {
            onToggle(letter.id);
          }}
        />
        {letter.subject}
      </label>
    </li>
  )
}
```

```js src/data.js
export const letters = [{
  id: 0,
  subject: 'Spremni za avanturu?',
  isStarred: true,
}, {
  id: 1,
  subject: 'Vreme za prijavu!',
  isStarred: false,
}, {
  id: 2,
  subject: 'Festival počinje za samo SEDAM dana!',
  isStarred: false,
}];
```

```css
input { margin: 5px; }
li { border-radius: 5px; }
label { width: 100%; padding: 5px; display: inline-block; }
.selected { background: #d2eaff; }
```

</Sandpack>

<Solution>

Umesto jednog `selectedId`, čuvajte `selectedIds` *niz* u state-u. Na primer, ako izaberete prvo i poslednje pismo, niz će sadržati `[0, 2]`. Kada ništa nije izabrano, to će biti prazan `[]` niz:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { letters } from './data.js';
import Letter from './Letter.js';

export default function MailClient() {
  const [selectedIds, setSelectedIds] = useState([]);

  const selectedCount = selectedIds.length;

  function handleToggle(toggledId) {
    // Da li je trenutno izabrano?
    if (selectedIds.includes(toggledId)) {
      // Onda ukloni ovaj ID iz niza.
      setSelectedIds(selectedIds.filter(id =>
        id !== toggledId
      ));
    } else {
      // U suprotnom, dodaj ovaj ID u niz.
      setSelectedIds([
        ...selectedIds,
        toggledId
      ]);
    }
  }

  return (
    <>
      <h2>Inbox</h2>
      <ul>
        {letters.map(letter => (
          <Letter
            key={letter.id}
            letter={letter}
            isSelected={
              selectedIds.includes(letter.id)
            }
            onToggle={handleToggle}
          />
        ))}
        <hr />
        <p>
          <b>
            Izabrali ste {selectedCount} pisama
          </b>
        </p>
      </ul>
    </>
  );
}
```

```js src/Letter.js
export default function Letter({
  letter,
  onToggle,
  isSelected,
}) {
  return (
    <li className={
      isSelected ? 'selected' : ''
    }>
      <label>
        <input
          type="checkbox"
          checked={isSelected}
          onChange={() => {
            onToggle(letter.id);
          }}
        />
        {letter.subject}
      </label>
    </li>
  )
}
```

```js src/data.js
export const letters = [{
  id: 0,
  subject: 'Spremni za avanturu?',
  isStarred: true,
}, {
  id: 1,
  subject: 'Vreme za prijavu!',
  isStarred: false,
}, {
  id: 2,
  subject: 'Festival počinje za samo SEDAM dana!',
  isStarred: false,
}];
```

```css
input { margin: 5px; }
li { border-radius: 5px; }
label { width: 100%; padding: 5px; display: inline-block; }
.selected { background: #d2eaff; }
```

</Sandpack>

Jedna sitna mana upotrebe nizova je da za svaku stavku pozivate `selectedIds.includes(letter.id)` da biste proverili da li je izabrana. Ako je niz veoma velik, ovo može izazvati problem sa performansama, jer pretraga nizova sa [`includes()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/includes) traje linearno, a vi radite pretragu za svaku pojedinačnu stavku.

Da ovo popravite, možete, umesto toga, čuvati [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set) u state-u, koji pruža brzu [`has()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set/has) operaciju:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { letters } from './data.js';
import Letter from './Letter.js';

export default function MailClient() {
  const [selectedIds, setSelectedIds] = useState(
    new Set()
  );

  const selectedCount = selectedIds.size;

  function handleToggle(toggledId) {
    // Napravi kopiju (da izbegneš mutaciju).
    const nextIds = new Set(selectedIds);
    if (nextIds.has(toggledId)) {
      nextIds.delete(toggledId);
    } else {
      nextIds.add(toggledId);
    }
    setSelectedIds(nextIds);
  }

  return (
    <>
      <h2>Inbox</h2>
      <ul>
        {letters.map(letter => (
          <Letter
            key={letter.id}
            letter={letter}
            isSelected={
              selectedIds.has(letter.id)
            }
            onToggle={handleToggle}
          />
        ))}
        <hr />
        <p>
          <b>
            Izabrali ste {selectedCount} pisama
          </b>
        </p>
      </ul>
    </>
  );
}
```

```js src/Letter.js
export default function Letter({
  letter,
  onToggle,
  isSelected,
}) {
  return (
    <li className={
      isSelected ? 'selected' : ''
    }>
      <label>
        <input
          type="checkbox"
          checked={isSelected}
          onChange={() => {
            onToggle(letter.id);
          }}
        />
        {letter.subject}
      </label>
    </li>
  )
}
```

```js src/data.js
export const letters = [{
  id: 0,
  subject: 'Spremni za avanturu?',
  isStarred: true,
}, {
  id: 1,
  subject: 'Vreme za prijavu!',
  isStarred: false,
}, {
  id: 2,
  subject: 'Festival počinje za samo SEDAM dana!',
  isStarred: false,
}];
```

```css
input { margin: 5px; }
li { border-radius: 5px; }
label { width: 100%; padding: 5px; display: inline-block; }
.selected { background: #d2eaff; }
```

</Sandpack>

Sada svaka stavka koristi `selectedIds.has(letter.id)` proveru koja je veoma brza.

Imajte na umu da [ne biste trebali da mutirate objekte u state-u](/learn/updating-objects-in-state), a da to, takođe, uključuje i Set-ove. Zbog ovoga `handleToggle` funkcija prvo pravi *kopiju* Set-a, a tek onda ažurira tu kopiju.

</Solution>

</Challenges>
