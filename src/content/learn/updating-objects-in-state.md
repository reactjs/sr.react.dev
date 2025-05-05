---
title: Ažuriranje objekata u state-u
---

<Intro>

State može sadržati bilo koju vrstu JavaScript vrednosti, uključujući i objekte. Međutim, ne bi trebalo direktno menjati objekte koje držite u React state-u. Umesto toga, kada želite da ažurirate objekat, potrebno je da kreirate novi (ili napravite kopiju postojećeg) i zatim ažurirate state kako bi koristio tu kopiju.

</Intro>

<YouWillLearn>

- Kako korektno ažurirati objekat u React state-u
- Kako ažurirati ugnježdeni objekat bez mutiranja
- Šta je immutability i kako da je ne prekršite
- Kako učiniti da se kopiranje objekta manje ponavlja uz pomoć Immer-a

</YouWillLearn>

## Šta je to mutacija? {/*whats-a-mutation*/}

Bilo koju vrstu JavaScript vrednosti možete držati u state-u.

```js
const [x, setX] = useState(0);
```

Do sad ste radili sa brojevima, stringovima i boolean vrednostima. Te vrste JavaScript vrednosti su "immutable", što znači nepromenljivo ili "read-only". Možete pokrenuti ponovni render da _zamenite_ vrednost:

```js
setX(5);
```

State `x` se promenilo sa `0` na `5`, ali _sam broj `0`_ se nije promenio. Nije moguće napraviti bilo kakve promene nad ugrađenim primitivnim vrednostima poput brojeva, stringova ili boolean-a u JavaScript-u.

Razmotrite objekat u state-u:

```js
const [position, setPosition] = useState({ x: 0, y: 0 });
```

Tehnički, moguće je promeniti sadržaj _samog objekta_. **Ovo se naziva mutacija:**

```js
position.x = 5;
```

Međutim, iako su objekti u React state-u tehnički mutable (promenljivi), trebalo bi da ih tretirate **kao da** su immutable--poput brojeva, boolean-a i stringova. Umesto da ih mutirate, uvek ih trebate zameniti.

## Tretiranje state-a kao read-only {/*treat-state-as-read-only*/}

Drugim rečima, trebate **bilo koji JavaScript objekat koji držite u state-u da tretirate kao da je read-only**.

Ovaj primer u state-u drži objekat koji predstavlja trenutnu poziciju kursora. Crvena tačka bi trebala da se pomera kada kliknete ili pomerate kursor preko preview oblasti. Ali, tačka ostaje na svojoj inicijalnoj poziciji:

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
        position.x = e.clientX;
        position.y = e.clientY;
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
  );
}
```

```css
body { margin: 0; padding: 0; height: 250px; }
```

</Sandpack>

Problem je u ovom delu koda.

```js
onPointerMove={e => {
  position.x = e.clientX;
  position.y = e.clientY;
}}
```

Ovaj kod menja objekat dodeljen `position` promenljivoj iz [prethodnog rendera](/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time). Ali, bez upotrebe state setter funkcije, React nema predstavu da se objekat promenio. Zato React ne radi ništa. Ovo je kao da pokušate promeniti porudžbinu nakon što završite obrok. Iako mutacija state-a može raditi u nekim slučajevima, ne preporučujemo takav pristup. Trebate tretirati state vrednost koju imate u renderu kao read-only.

Da biste zapravo [pokrenuli ponovni render](/learn/state-as-a-snapshot#setting-state-triggers-renders) u ovom slučaju, **kreirajte *novi* objekat i prosledite ga u state setter funkciju**:

```js
onPointerMove={e => {
  setPosition({
    x: e.clientX,
    y: e.clientY
  });
}}
```

Sa `setPosition` React-u govorite sledeće:

* Zameni `position` sa novim objektom
* I renderuj komponentu ponovo

Primetite da crvena tačka sad prati vaš kursor kada ga pomerate kroz preview oblast:

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
  );
}
```

```css
body { margin: 0; padding: 0; height: 250px; }
```

</Sandpack>

<DeepDive>

#### Lokalna mutacija je u redu {/*local-mutation-is-fine*/}

Kod poput ovog je problematičan jer menja *postojeći* objekat u state-u:

```js
position.x = e.clientX;
position.y = e.clientY;
```

Ali, ovakav kod je **apsolutno u redu** jer mutirate novi objekat koji ste *upravo kreirali*:

```js
const nextPosition = {};
nextPosition.x = e.clientX;
nextPosition.y = e.clientY;
setPosition(nextPosition);
```

U suštini, potpuno je jednako ovome:

```js
setPosition({
  x: e.clientX,
  y: e.clientY
});
```

Mutacija je jedino problematična kada menjate *postojeće* objekte koji su već u state-u. Mutiranje objekta koji ste upravo kreirali je u redu zato što *ga ništa ne referencira još uvek*. Njegovom promenom ne možete slučajno uticati na nešto što zavisi od njega. Ovo se naziva "lokalna mutacija". Možete koristiti lokalnu mutaciju i [tokom renderovanja](/learn/keeping-components-pure#local-mutation-your-components-little-secret). Veoma zgodno i potpuno u redu!

</DeepDive>  

## Kopiranje objekata sa spread sintaksom {/*copying-objects-with-the-spread-syntax*/}

U prethodnom primeru, `position` objekat se uvek kreirao iznova na osnovu trenutne pozicije kursora. Ali, često ćete želeti da uključite *postojeće* podatke u novi objekat koji kreirate. Na primer, možete želeti da ažurirate *samo jedno* polje u formi, a da ostavite trenutne vrednosti za ostala polja.

Ova input polja ne rade jer `onChange` handler-i mutiraju state:

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com'
  });

  function handleFirstNameChange(e) {
    person.firstName = e.target.value;
  }

  function handleLastNameChange(e) {
    person.lastName = e.target.value;
  }

  function handleEmailChange(e) {
    person.email = e.target.value;
  }

  return (
    <>
      <label>
        Ime:
        <input
          value={person.firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Prezime:
        <input
          value={person.lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <label>
        Email:
        <input
          value={person.email}
          onChange={handleEmailChange}
        />
      </label>
      <p>
        {person.firstName}{' '}
        {person.lastName}{' '}
        ({person.email})
      </p>
    </>
  );
}
```

```css
label { display: block; }
input { margin-left: 5px; margin-bottom: 5px; }
```

</Sandpack>

Na primer, ova linija mutira state iz prethodnog rendera:

```js
person.firstName = e.target.value;
```

Pouzdan način da dobijete željeno ponašanje je da kreirate novi objekat i prosledite ga u `setPerson`. Ali, ovde želite i da **kopirate postojeće podatke u njega**, jer se samo jedno polje promenilo:

```js
setPerson({
  firstName: e.target.value, // Novo ime iz input-a
  lastName: person.lastName,
  email: person.email
});
```

Možete koristiti `...` [objektnu spread](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#spread_in_object_literals) sintaksu kako ne biste morali da kopirate svako polje pojedinačno.

```js
setPerson({
  ...person, // Kopiraj stara polja
  firstName: e.target.value // Override-uj ovo jedno
});
```

Sada forma radi! 

Primetite da niste deklarisali posebnu state promenljivu za svako input polje. Za velike forme, grupisanje svih podataka u objekat je veoma zgodno--dok god ga ažurirate kako treba!

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com'
  });

  function handleFirstNameChange(e) {
    setPerson({
      ...person,
      firstName: e.target.value
    });
  }

  function handleLastNameChange(e) {
    setPerson({
      ...person,
      lastName: e.target.value
    });
  }

  function handleEmailChange(e) {
    setPerson({
      ...person,
      email: e.target.value
    });
  }

  return (
    <>
      <label>
        Ime:
        <input
          value={person.firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Prezime:
        <input
          value={person.lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <label>
        Email:
        <input
          value={person.email}
          onChange={handleEmailChange}
        />
      </label>
      <p>
        {person.firstName}{' '}
        {person.lastName}{' '}
        ({person.email})
      </p>
    </>
  );
}
```

```css
label { display: block; }
input { margin-left: 5px; margin-bottom: 5px; }
```

</Sandpack>

Primetite da je `...` spread sintaksa "plitka"--kopira polja samo na prvom nivou dubine. Ovo je čini brzom, ali, takođe znači da ako želite ažurirati ugnježdeno polje, moraćete je koristiti više od jednom.

<DeepDive>

#### Upotreba jednog event handler-a za više polja {/*using-a-single-event-handler-for-multiple-fields*/}

Možete koristiti `[` i `]` zagrade unutar definicije objekta da specificirate polje sa dinamičkim imenom. Ovo je isti primer, ali sa jednim event handler-om umesto tri različita:

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com'
  });

  function handleChange(e) {
    setPerson({
      ...person,
      [e.target.name]: e.target.value
    });
  }

  return (
    <>
      <label>
        Ime:
        <input
          name="firstName"
          value={person.firstName}
          onChange={handleChange}
        />
      </label>
      <label>
        Prezime:
        <input
          name="lastName"
          value={person.lastName}
          onChange={handleChange}
        />
      </label>
      <label>
        Email:
        <input
          name="email"
          value={person.email}
          onChange={handleChange}
        />
      </label>
      <p>
        {person.firstName}{' '}
        {person.lastName}{' '}
        ({person.email})
      </p>
    </>
  );
}
```

```css
label { display: block; }
input { margin-left: 5px; margin-bottom: 5px; }
```

</Sandpack>

Ovde, `e.target.name` predstavlja `name` polje zadato u `<input>` DOM elementu.

</DeepDive>

## Ažuriranje ugnježdenog objekta {/*updating-a-nested-object*/}

Pogledajte ovu strukturu ugnježdenog objekta:

```js
const [person, setPerson] = useState({
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://i.imgur.com/Sd1AgUOm.jpg',
  }
});
```

Ako želite ažurirati `person.artwork.city`, jasno je kako to uraditi kroz mutaciju:

```js
person.artwork.city = 'New Delhi';
```

Ali, u React-u, tretirate state kao da je immutable! Da biste promenili `city`, prvo trebate napraviti novi `artwork` objekat (popunjen podacima iz prethodnog), a onda napraviti novi `person` objekat koji pokazuje na novi `artwork`:

```js
const nextArtwork = { ...person.artwork, city: 'New Delhi' };
const nextPerson = { ...person, artwork: nextArtwork };
setPerson(nextPerson);
```

Ili, napisano u jednom pozivu funkcije:

```js
setPerson({
  ...person, // Kopiraj ostala polja
  artwork: { // ali zameni artwork
    ...person.artwork, // sa svim istim poljima
    city: 'New Delhi' // ali sa gradom New Delhi!
  }
});
```

Ovo postaje dugačko, ali radi dobro u mnogim slučajevima:

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

<DeepDive>

#### Objekti nisu zapravo ugnježdeni {/*objects-are-not-really-nested*/}

Ovaj objekat u kodu deluje kao "ugnježden":

```js
let obj = {
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://i.imgur.com/Sd1AgUOm.jpg',
  }
};
```

Međutim, "ugnježdavanje" je netačan način za razmišljanje o tome kako se objekti ponašaju. Kad se kod izvršava, ne postoji nešto što se naziva "ugnježden" objekat. Zapravo gledate u dva različita objekta:

```js
let obj1 = {
  title: 'Blue Nana',
  city: 'Hamburg',
  image: 'https://i.imgur.com/Sd1AgUOm.jpg',
};

let obj2 = {
  name: 'Niki de Saint Phalle',
  artwork: obj1
};
```

Objekat `obj1` nije "unutar" `obj2`. Na primer, `obj3` bi mogao da "pokazuje" na `obj1` takođe:

```js
let obj1 = {
  title: 'Blue Nana',
  city: 'Hamburg',
  image: 'https://i.imgur.com/Sd1AgUOm.jpg',
};

let obj2 = {
  name: 'Niki de Saint Phalle',
  artwork: obj1
};

let obj3 = {
  name: 'Copycat',
  artwork: obj1
};
```

Ako biste mutirali `obj3.artwork.city`, to bi uticalo na `obj2.artwork.city` i `obj1.city` takođe. To se dešava zato što su `obj3.artwork`, `obj2.artwork` i `obj1` isti objekat. Ovo je teško da se primeti kada mislite o objektima kao da su "ugnježdeni". Umesto toga, to su odvojeni objekti koji "pokazuju" jedni na druge preko svojih polja.

</DeepDive>  

### Pisanje koncizne logike ažuriranja sa Immer-om {/*write-concise-update-logic-with-immer*/}

Ako je vaš state duboko ugnježden, možete razmisliti da ga [flatten-ujete](/learn/choosing-the-state-structure#avoid-deeply-nested-state). Ali, ako ne želite promeniti strukturu vašeg state-a, možda biste voleli prečicu za ugnježdene spread-ove. [Immer](https://github.com/immerjs/use-immer) je popularna biblioteka koja omogućava pisanje zgodne sintakse za mutiranje i brine o pravljenju kopija za vas. Sa Immer-om, kod koji pišete deluje kao da "krši pravila" i mutira objekat:

```js
updatePerson(draft => {
  draft.artwork.city = 'Lagos';
});
```

Ali, za razliku od obične mutacije, ne menja prethodni state!

<DeepDive>

#### Kako Immer radi? {/*how-does-immer-work*/}

`draft`, promenljiva koju Immer uvodi, je poseban objekat koji se naziva [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy), koji "beleži" šta radite sa njim. To je razlog zašto ga slobodno možete mutirati kako želite! Ispod haube, Immer prepoznaje koji delovi `draft`-a su promenjeni i proizvodi potpuno novi objekat koji sadrži vaše promene.

</DeepDive>

Da biste isprobali Immer:

1. Pokrenite `npm install use-immer` da dodate Immer kao zavisnost
2. Onda, zamenite `import { useState } from 'react'` sa `import { useImmer } from 'use-immer'`

Ovo je primer od gore konvertovan u Immer:

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

Primetite kako su event handler-i postali dosta koncizniji. Možete mešati `useState` i `useImmer` u jednoj komponenti koliko god želite. Immer je sjajan način za pisanje konciznih handler-a za ažuriranje, posebno ako imate ugnježdeni state, a kopiranje objekata vodi ka dupliranju koda.

<DeepDive>

#### Zašto mutiranje state-a nije preporučeno u React-u? {/*why-is-mutating-state-not-recommended-in-react*/}

Postoji par razloga:

* **Debug-ovanje:** Ako koristite `console.log` i ne mutirate state, stari logovi neće biti pretrpani sa novijim izmenama state-a. Tako da možete jasno videti kako se state promenio između rendera.
* **Optimizacije:** Uobičajene React-ove [strategije optimizacije](/reference/react/memo) se oslanjaju na preskakanje posla ako su prethodni props-i ili state isti kao i novi. Ako nikada ne mutirate state, veoma se brzo proveri da li je bilo promena. Ako je `prevObj === obj`, možete biti sigurni da se ništa unutar njih nije promenilo.
* **Nove funkcionalnosti:** Nove React funkcionalnosti koje pravimo se zasnivaju na tome da se state [tretira kao snapshot](/learn/state-as-a-snapshot). Ako mutirate prethodne verzije state-a, to vas može sprečiti da koristite nove funkcionalnosti.
* **Promene zahteva:** Neke funkcionalnosti, poput Undo/Redo, prikaza istorije promena ili dopuštanja korisniku da povrati formu na prethodne vrednosti, su jednostavnije kada ništa nije mutirano. Zato što možete čuvati prethodne kopije state-a u memoriji i koristiti ih kad vam je potrebno. Ako započnete sa pristupom mutacija, biće teže kasnije dodati ovakve funkcionalnosti.
* **Jednostavnija implementacija:** React ne treba da radi ništa specijalno sa vašim objektima, jer se ne oslanja na mutaciju. Ne treba da otima njihova polja, da ih obmotava u Proxy-je ili da radi druge stvari tokom inicijalizacije kao što to mnoga "reaktivna" rešenja rade. Baš zbog ovoga vam React dopušta da stavite bilo kakav objekat u state--nebitno koliko velik--bez dodatnih problema sa performansama i ispravnošću.

U praksi, često ćete moći da "se izvučete" sa mutiranjem state-a u React-u, ali vam ozbiljno savetujemo da to ne radite kako biste mogli da koristite nove React funkcionalnosti napravljene sa ovim pristupom na umu. Budući saradnici, a možda i vi u budućnosti, će vam biti zahvalni!

</DeepDive>

<Recap>

* Tretirajte svaki state u React-u kao immutable.
* Kada držite objekte u state-u, njihovim mutiranjem nećete pokrenuti rendere i izmenićete state u "snapshot-ovima" prethodnog rendera.
* Umesto mutiranja objekta, kreirajte *novu* verziju i pokrenite ponovni render postavljanjem state-a na tu novu verziju.
* Možete koristiti `{...obj, something: 'newValue'}` objektnu spread sintaksu za kreiranje kopija objekata.
* Spread sintaksa je plitka: kopira samo jedan nivo u dubinu.
* Za ažuriranje ugnježdenog objekta morate kreirati kopije sve do mesta koje ažurirate.
* Za smanjivanje ponavljajućeg koda tokom kopiranja, koristite Immer.

</Recap>



<Challenges>

#### Popraviti netačna ažuriranja state-a {/*fix-incorrect-state-updates*/}

Ova forma ima par bug-ova. Kliknite dugme koje povećava rezultat par puta. Primetite da se ne povećava. Onda, promenite ime i primetite da se rezultat "uskladio" sa vašim izmenama. Konačno, promenite prezime i primetite da je rezultat potpuno nestao.

Vaš zadatak je da popravite ove bug-ove. Kad ih popravite, objasnite zašto se svaki od njih desio.

<Sandpack>

```js
import { useState } from 'react';

export default function Scoreboard() {
  const [player, setPlayer] = useState({
    firstName: 'Ranjani',
    lastName: 'Shettar',
    score: 10,
  });

  function handlePlusClick() {
    player.score++;
  }

  function handleFirstNameChange(e) {
    setPlayer({
      ...player,
      firstName: e.target.value,
    });
  }

  function handleLastNameChange(e) {
    setPlayer({
      lastName: e.target.value
    });
  }

  return (
    <>
      <label>
        Rezultat: <b>{player.score}</b>
        {' '}
        <button onClick={handlePlusClick}>
          +1
        </button>
      </label>
      <label>
        Ime:
        <input
          value={player.firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Prezime:
        <input
          value={player.lastName}
          onChange={handleLastNameChange}
        />
      </label>
    </>
  );
}
```

```css
label { display: block; margin-bottom: 10px; }
input { margin-left: 5px; margin-bottom: 5px; }
```

</Sandpack>

<Solution>

Ovo je verzija gde su oba bug-a popravljena:

<Sandpack>

```js
import { useState } from 'react';

export default function Scoreboard() {
  const [player, setPlayer] = useState({
    firstName: 'Ranjani',
    lastName: 'Shettar',
    score: 10,
  });

  function handlePlusClick() {
    setPlayer({
      ...player,
      score: player.score + 1,
    });
  }

  function handleFirstNameChange(e) {
    setPlayer({
      ...player,
      firstName: e.target.value,
    });
  }

  function handleLastNameChange(e) {
    setPlayer({
      ...player,
      lastName: e.target.value
    });
  }

  return (
    <>
      <label>
        Rezultat: <b>{player.score}</b>
        {' '}
        <button onClick={handlePlusClick}>
          +1
        </button>
      </label>
      <label>
        Ime:
        <input
          value={player.firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Prezime:
        <input
          value={player.lastName}
          onChange={handleLastNameChange}
        />
      </label>
    </>
  );
}
```

```css
label { display: block; }
input { margin-left: 5px; margin-bottom: 5px; }
```

</Sandpack>

Problem sa `handlePlusClick` je u tome što je menjao `player` objekat. Kao rezultat, React nije znao da postoji razlog za ponovni render, pa nije ažurirao rezultat na ekranu. Zato se, kada ste promenili ime, state ažurirao, pokrenuvši ponovni render koji je _takođe_ ažurirao i rezultat na ekranu.

Problem sa `handleLastNameChange` je u tome što nije kopirao postojeća `...player` polja u novi objekat. Zbog toga se rezultat izgubio kada ste promenili prezime.

</Solution>

#### Pronaći i popraviti mutaciju {/*find-and-fix-the-mutation*/}

Ovde je kutija za prevlačenje na statičnoj pozadini. Možete promeniti boju kutije pomoću select input-a.

Ali, tu je bug. Ako prvo pomerite kutiju, a onda promenite boju, pozadina (koja ne bi trebalo da se pomera!) će "skočiti" na poziciju kutije. Ovo ne bi trebalo da se dešava: `position` prop od `Background`-a je postavljen na `initialPosition`, što je `{ x: 0, y: 0 }`. Zašto se pozadina pomera nakon promene boje?

Pronađite bug i popravite ga.

<Hint>

Ako se nešto neočekivano promeni, postoji mutacija. Pronađite mutaciju u `App.js` i popravite je.

</Hint>

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Background from './Background.js';
import Box from './Box.js';

const initialPosition = {
  x: 0,
  y: 0
};

export default function Canvas() {
  const [shape, setShape] = useState({
    color: 'orange',
    position: initialPosition
  });

  function handleMove(dx, dy) {
    shape.position.x += dx;
    shape.position.y += dy;
  }

  function handleColorChange(e) {
    setShape({
      ...shape,
      color: e.target.value
    });
  }

  return (
    <>
      <select
        value={shape.color}
        onChange={handleColorChange}
      >
        <option value="orange">orange</option>
        <option value="lightpink">lightpink</option>
        <option value="aliceblue">aliceblue</option>
      </select>
      <Background
        position={initialPosition}
      />
      <Box
        color={shape.color}
        position={shape.position}
        onMove={handleMove}
      >
        Pomeri me!
      </Box>
    </>
  );
}
```

```js src/Box.js
import { useState } from 'react';

export default function Box({
  children,
  color,
  position,
  onMove
}) {
  const [
    lastCoordinates,
    setLastCoordinates
  ] = useState(null);

  function handlePointerDown(e) {
    e.target.setPointerCapture(e.pointerId);
    setLastCoordinates({
      x: e.clientX,
      y: e.clientY,
    });
  }

  function handlePointerMove(e) {
    if (lastCoordinates) {
      setLastCoordinates({
        x: e.clientX,
        y: e.clientY,
      });
      const dx = e.clientX - lastCoordinates.x;
      const dy = e.clientY - lastCoordinates.y;
      onMove(dx, dy);
    }
  }

  function handlePointerUp(e) {
    setLastCoordinates(null);
  }

  return (
    <div
      onPointerDown={handlePointerDown}
      onPointerMove={handlePointerMove}
      onPointerUp={handlePointerUp}
      style={{
        width: 100,
        height: 100,
        cursor: 'grab',
        backgroundColor: color,
        position: 'absolute',
        border: '1px solid black',
        display: 'flex',
        justifyContent: 'center',
        alignItems: 'center',
        transform: `translate(
          ${position.x}px,
          ${position.y}px
        )`,
      }}
    >{children}</div>
  );
}
```

```js src/Background.js
export default function Background({
  position
}) {
  return (
    <div style={{
      position: 'absolute',
      transform: `translate(
        ${position.x}px,
        ${position.y}px
      )`,
      width: 250,
      height: 250,
      backgroundColor: 'rgba(200, 200, 0, 0.2)',
    }} />
  );
};
```

```css
body { height: 280px; }
select { margin-bottom: 10px; }
```

</Sandpack>

<Solution>

Problem je bila mutacija u `handleMove`. Mutirao se `shape.position`, a to je isti objekat na koji `initialPosition` pokazuje. Zbog ovoga su se i kutija i pozadina pomerili. (To je mutacija, pa se izmena ne prikaže na ekranu pre nepovezanog ažuriranja--izmena boje--pokreće ponovni render.)

Rešenje je uklanjanje mutacije iz `handleMove` i upotreba spread sintakse za pravljenje shape-a. Primetite da je `+=` mutacija, pa morate koristiti običnu `+` operaciju.

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Background from './Background.js';
import Box from './Box.js';

const initialPosition = {
  x: 0,
  y: 0
};

export default function Canvas() {
  const [shape, setShape] = useState({
    color: 'orange',
    position: initialPosition
  });

  function handleMove(dx, dy) {
    setShape({
      ...shape,
      position: {
        x: shape.position.x + dx,
        y: shape.position.y + dy,
      }
    });
  }

  function handleColorChange(e) {
    setShape({
      ...shape,
      color: e.target.value
    });
  }

  return (
    <>
      <select
        value={shape.color}
        onChange={handleColorChange}
      >
        <option value="orange">orange</option>
        <option value="lightpink">lightpink</option>
        <option value="aliceblue">aliceblue</option>
      </select>
      <Background
        position={initialPosition}
      />
      <Box
        color={shape.color}
        position={shape.position}
        onMove={handleMove}
      >
        Pomeri me!
      </Box>
    </>
  );
}
```

```js src/Box.js
import { useState } from 'react';

export default function Box({
  children,
  color,
  position,
  onMove
}) {
  const [
    lastCoordinates,
    setLastCoordinates
  ] = useState(null);

  function handlePointerDown(e) {
    e.target.setPointerCapture(e.pointerId);
    setLastCoordinates({
      x: e.clientX,
      y: e.clientY,
    });
  }

  function handlePointerMove(e) {
    if (lastCoordinates) {
      setLastCoordinates({
        x: e.clientX,
        y: e.clientY,
      });
      const dx = e.clientX - lastCoordinates.x;
      const dy = e.clientY - lastCoordinates.y;
      onMove(dx, dy);
    }
  }

  function handlePointerUp(e) {
    setLastCoordinates(null);
  }

  return (
    <div
      onPointerDown={handlePointerDown}
      onPointerMove={handlePointerMove}
      onPointerUp={handlePointerUp}
      style={{
        width: 100,
        height: 100,
        cursor: 'grab',
        backgroundColor: color,
        position: 'absolute',
        border: '1px solid black',
        display: 'flex',
        justifyContent: 'center',
        alignItems: 'center',
        transform: `translate(
          ${position.x}px,
          ${position.y}px
        )`,
      }}
    >{children}</div>
  );
}
```

```js src/Background.js
export default function Background({
  position
}) {
  return (
    <div style={{
      position: 'absolute',
      transform: `translate(
        ${position.x}px,
        ${position.y}px
      )`,
      width: 250,
      height: 250,
      backgroundColor: 'rgba(200, 200, 0, 0.2)',
    }} />
  );
};
```

```css
body { height: 280px; }
select { margin-bottom: 10px; }
```

</Sandpack>

</Solution>

#### Ažurirati objekat sa Immer-om {/*update-an-object-with-immer*/}

Ovo je isti bug-oviti primer iz prethodnog izazova. Ovog puta, popravite mutaciju pomoću Immer-a. Za vašu ugodnost, `useImmer` je već import-ovan, a vi trebate promeniti `shape` state promenljivu da ga koristi.

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { useImmer } from 'use-immer';
import Background from './Background.js';
import Box from './Box.js';

const initialPosition = {
  x: 0,
  y: 0
};

export default function Canvas() {
  const [shape, setShape] = useState({
    color: 'orange',
    position: initialPosition
  });

  function handleMove(dx, dy) {
    shape.position.x += dx;
    shape.position.y += dy;
  }

  function handleColorChange(e) {
    setShape({
      ...shape,
      color: e.target.value
    });
  }

  return (
    <>
      <select
        value={shape.color}
        onChange={handleColorChange}
      >
        <option value="orange">orange</option>
        <option value="lightpink">lightpink</option>
        <option value="aliceblue">aliceblue</option>
      </select>
      <Background
        position={initialPosition}
      />
      <Box
        color={shape.color}
        position={shape.position}
        onMove={handleMove}
      >
        Pomeri me!
      </Box>
    </>
  );
}
```

```js src/Box.js
import { useState } from 'react';

export default function Box({
  children,
  color,
  position,
  onMove
}) {
  const [
    lastCoordinates,
    setLastCoordinates
  ] = useState(null);

  function handlePointerDown(e) {
    e.target.setPointerCapture(e.pointerId);
    setLastCoordinates({
      x: e.clientX,
      y: e.clientY,
    });
  }

  function handlePointerMove(e) {
    if (lastCoordinates) {
      setLastCoordinates({
        x: e.clientX,
        y: e.clientY,
      });
      const dx = e.clientX - lastCoordinates.x;
      const dy = e.clientY - lastCoordinates.y;
      onMove(dx, dy);
    }
  }

  function handlePointerUp(e) {
    setLastCoordinates(null);
  }

  return (
    <div
      onPointerDown={handlePointerDown}
      onPointerMove={handlePointerMove}
      onPointerUp={handlePointerUp}
      style={{
        width: 100,
        height: 100,
        cursor: 'grab',
        backgroundColor: color,
        position: 'absolute',
        border: '1px solid black',
        display: 'flex',
        justifyContent: 'center',
        alignItems: 'center',
        transform: `translate(
          ${position.x}px,
          ${position.y}px
        )`,
      }}
    >{children}</div>
  );
}
```

```js src/Background.js
export default function Background({
  position
}) {
  return (
    <div style={{
      position: 'absolute',
      transform: `translate(
        ${position.x}px,
        ${position.y}px
      )`,
      width: 250,
      height: 250,
      backgroundColor: 'rgba(200, 200, 0, 0.2)',
    }} />
  );
};
```

```css
body { height: 280px; }
select { margin-bottom: 10px; }
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

Ovo je rešenje napisano pomoću Immer-a. Primetite da su event handler-i napisani u stilu mutacije, ali se bug ne dešava. Zato što, ispod haube, Immer nikada ne mutira već postojeće objekte.

<Sandpack>

```js src/App.js
import { useImmer } from 'use-immer';
import Background from './Background.js';
import Box from './Box.js';

const initialPosition = {
  x: 0,
  y: 0
};

export default function Canvas() {
  const [shape, updateShape] = useImmer({
    color: 'orange',
    position: initialPosition
  });

  function handleMove(dx, dy) {
    updateShape(draft => {
      draft.position.x += dx;
      draft.position.y += dy;
    });
  }

  function handleColorChange(e) {
    updateShape(draft => {
      draft.color = e.target.value;
    });
  }

  return (
    <>
      <select
        value={shape.color}
        onChange={handleColorChange}
      >
        <option value="orange">orange</option>
        <option value="lightpink">lightpink</option>
        <option value="aliceblue">aliceblue</option>
      </select>
      <Background
        position={initialPosition}
      />
      <Box
        color={shape.color}
        position={shape.position}
        onMove={handleMove}
      >
        Pomeri me!
      </Box>
    </>
  );
}
```

```js src/Box.js
import { useState } from 'react';

export default function Box({
  children,
  color,
  position,
  onMove
}) {
  const [
    lastCoordinates,
    setLastCoordinates
  ] = useState(null);

  function handlePointerDown(e) {
    e.target.setPointerCapture(e.pointerId);
    setLastCoordinates({
      x: e.clientX,
      y: e.clientY,
    });
  }

  function handlePointerMove(e) {
    if (lastCoordinates) {
      setLastCoordinates({
        x: e.clientX,
        y: e.clientY,
      });
      const dx = e.clientX - lastCoordinates.x;
      const dy = e.clientY - lastCoordinates.y;
      onMove(dx, dy);
    }
  }

  function handlePointerUp(e) {
    setLastCoordinates(null);
  }

  return (
    <div
      onPointerDown={handlePointerDown}
      onPointerMove={handlePointerMove}
      onPointerUp={handlePointerUp}
      style={{
        width: 100,
        height: 100,
        cursor: 'grab',
        backgroundColor: color,
        position: 'absolute',
        border: '1px solid black',
        display: 'flex',
        justifyContent: 'center',
        alignItems: 'center',
        transform: `translate(
          ${position.x}px,
          ${position.y}px
        )`,
      }}
    >{children}</div>
  );
}
```

```js src/Background.js
export default function Background({
  position
}) {
  return (
    <div style={{
      position: 'absolute',
      transform: `translate(
        ${position.x}px,
        ${position.y}px
      )`,
      width: 250,
      height: 250,
      backgroundColor: 'rgba(200, 200, 0, 0.2)',
    }} />
  );
};
```

```css
body { height: 280px; }
select { margin-bottom: 10px; }
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

</Solution>

</Challenges>
