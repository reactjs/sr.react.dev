---
title: JavaScript u JSX-u sa vitičastim zagradama
---

<Intro>

JSX vam dozvoljava da pišete markup sličan HTML-u unutar JavaScript fajla, čuvajući logiku prikazivanja i sadržaja na istom mestu. Ponekad ćete želeti da dodate malo JavaScript logike ili da referencirate dinamičko svojstvo unutar tog markup-a. U ovoj situaciji možete koristiti vitičaste zagrade u vašem JSX-u da otvorite prozor ka JavaScript-u.

</Intro>

<YouWillLearn>

* Kako da prosledite stringove sa navodnicima
* Kako da referencirate JavaScript promenljivu unutar JSX-a sa vitičastim zagradama
* Kako da pozovete JavaScript funkciju unutar JSX-a sa vitičastim zagradama
* Kako da koristite JavaScript objekat unutar JSX-a sa vitičastim zagradama
</YouWillLearn>

## Prosleđivanje stringova sa navodnicima {/*passing-strings-with-quotes*/}

Kada želite da prosledite string atribut JSX-u, stavite ga u jednostruke ili dvostruke navodnike:

<Sandpack>

```js
export default function Avatar() {
  return (
    <img
      className="avatar"
      src="https://i.imgur.com/7vQD0fPs.jpg"
      alt="Gregorio Y. Zara"
    />
  );
}
```

```css
.avatar { border-radius: 50%; height: 90px; }
```

</Sandpack>

Ovde, `"https://i.imgur.com/7vQD0fPs.jpg"` i `"Gregorio Y. Zara"` se prosleđuju kao stringovi.

Ali šta ako želite da dinamički odredite `src` ili `alt` tekst? Možete **koristiti vrednost iz JavaScript-a zamenom `"` i `"` sa `{` i `}`**:

<Sandpack>

```js
export default function Avatar() {
  const avatar = 'https://i.imgur.com/7vQD0fPs.jpg';
  const description = 'Gregorio Y. Zara';
  return (
    <img
      className="avatar"
      src={avatar}
      alt={description}
    />
  );
}
```

```css
.avatar { border-radius: 50%; height: 90px; }
```

</Sandpack>

Primećujete razliku između `className="avatar"`, što određuje CSS klasu `"avatar"` koja čini sliku okruglom, i `src={avatar}` koja čita vrednost JavaScript promenljive koja se zove `avatar`. To je zato što vam vitičaste zagrade dozvoljavaju da radite sa JavaScript-om upravo u vašem markup-u!

## Koristite vitičaste zagrade: Prozor u JavaScript svet {/*using-curly-braces-a-window-into-the-javascript-world*/}

JSX je specijalan način pisanja JavaScript-a. To znači da je moguće koristiti JavaScript unutar njega - sa vitičastim zagradama `{ }`. Primer ispod prvo deklariše ime za naučnika, `name`, a zatim ga ubacuje vitičastim zagradama unutar `<h1>`:

<Sandpack>

```js
export default function TodoList() {
  const name = 'Gregorio Y. Zara';
  return (
    <h1>{name} spisak stvari za uraditi</h1>
  );
}
```

</Sandpack>

Pokušajte da promenite vrednost `name` iz `'Gregorio Y. Zara'` u `'Hedy Lamarr'`. Pogledajte kako se naslov liste menja?

Bilo koji JavaScript izraz će raditi između vitičastih zagrada, uključujući pozive funkcija kao što je `formatDate()`:

<Sandpack>

```js
const today = new Date();

function formatDate(date) {
  return new Intl.DateTimeFormat(
    'en-US',
    { weekday: 'long' }
  ).format(date);
}

export default function TodoList() {
  return (
    <h1>Lista stvari iza uraditi za {formatDate(today)}</h1>
  );
}
```

</Sandpack>

### Gde koristiti vitičaste zagrade {/*where-to-use-curly-braces*/}

Možete koristiti vitičaste zagrade samo na dva načina unutar JSX-a:

1. **Kao tekst** direktno unutar JSX oznake: `<h1>{name} lista za uraditi</h1>` radi, ali `<{tag}>Gregorio Y. Zara lista stvari za uraditi</{tag}>` neće.
2. **Kao atribute** praćen znakom `=`: `src={avatar}` će pročitati promenljivu `avatar`, ali `src="{avatar}"` će proslediti string `"{avatar}"`.

## Koristite "duple vitičaste zagrade": CSS i drugi objekti u JSX-u {/*using-double-curlies-css-and-other-objects-in-jsx*/}

Uz stringove, brojeve i druge JavaScript izraze, možete čak proslediti i objekte u JSX. Objekti se takođe označavaju vitičastim zagradama, kao `{ name: "Hedy Lamarr", inventions: 5 }`. Stoga, da biste prosledili JS objekat u JSX, morate da umotate objekat u još jedan par vitičastih zagrada: `person={{ name: "Hedy Lamarr", inventions: 5 }}`.

Možda ćete videti ovo sa inline CSS stilovima u JSX-u. React ne zahteva da koristite inline stilove (CSS klase rade odlično u većini slučajeva). Ali kada vam je potreban inline stil, prosledite objekat atributu `style`:

<Sandpack>

```js
export default function TodoList() {
  return (
    <ul style={{
      backgroundColor: 'black',
      color: 'pink'
    }}>
      <li>Unaprediti video-telefon</li>
      <li>Pripremiti predavanja iz aeronautike</li>
      <li>Raditi na motoru na alkohol</li>
    </ul>
  );
}
```

```css
body { padding: 0; margin: 0 }
ul { padding: 20px 20px 20px 40px; margin: 0; }
```

</Sandpack>

Pokušajte da promenite vrednosti `backgroundColor` i `color`.

Stvarno možete videti JavaScript objekat unutar vitičastih zagrada kada ga napišete ovako:

```js {2-5}
<ul style={
  {
    backgroundColor: 'black',
    color: 'pink'
  }
}>
```

Sledeći put kada vidite `{{` i `}}` u JSX-u, znajte da je to ništa više od objekta unutar JSX vitičastih zagrada!

<Pitfall>

Inline `style` svojstva se pišu u camelCase. Na primer, HTML `<ul style="background-color: black">` bi se napisao kao `<ul style={{ backgroundColor: 'black' }}>` u vašoj komponenti.

</Pitfall>


## Još zabave sa JavaScript objektima i vitičastim zagradama {/*more-fun-with-javascript-objects-and-curly-braces*/}

Možete da prosledite više JavaScript izraza u jedan objekat i da ih referencirate u vašem JSX-u unutar vitičastih zagrada:

<Sandpack>

```js
const person = {
  name: 'Gregorio Y. Zara',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name} lista</h1>
      <img
        className="avatar"
        src="https://i.imgur.com/7vQD0fPs.jpg"
        alt="Gregorio Y. Zara"
      />
      <ul>
        <li>Unaprediti video-telefon</li>
        <li>Pripremiti predavanja iz aeronautike</li>
        <li>Raditi na motoru na alkohol</li>
      </ul>
    </div>
  );
}
```

```css
body { padding: 0; margin: 0 }
body > div > div { padding: 20px; }
.avatar { border-radius: 50%; height: 90px; }
```

</Sandpack>

U ovom primeru JavaScript objekat `person` sadrži string `name` i objekat `theme`:

```js
const person = {
  name: 'Gregorio Y. Zara',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};
```

Komponenta može da koristi ove vrednosti iz `person` ovako:

```js
<div style={person.theme}>
  <h1>{person.name} lista</h1>
```

JSX je veoma minimalan kao jezik za šabloniranje jer vam omogućava da organizujete podatke i logiku koristeći JavaScript.

<Recap>

Sada znate skoro sve o JSX-u:

* JSX atributi unutar navodnika se prosleđuju kao stringovi.
* Vitičaste zagrade vam omogućavaju da unesete JavaScript logiku i promenljive u vaš markup.
* One rade unutar JSX oznake ili odmah nakon `=` u atributima.
* `{{` i `}}` nisu specijalna sintaksa: to je JavaScript objekat koji je umotan u JSX vitičaste zagrade.

</Recap>

<Challenges>

#### Popravite grešku {/*fix-the-mistake*/}

Ovaj kod ne radi sa greškom koja kaže `Objects are not valid as a React child`:

<Sandpack>

```js
const person = {
  name: 'Gregorio Y. Zara',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person} lista</h1>
      <img
        className="avatar"
        src="https://i.imgur.com/7vQD0fPs.jpg"
        alt="Gregorio Y. Zara"
      />
      <ul>
        <li>Unaprediti video-telefon</li>
        <li>Pripremiti predavanja iz aeronautike</li>
        <li>Raditi na motoru na alkohol</li>
      </ul>
    </div>
  );
}
```

```css
body { padding: 0; margin: 0 }
body > div > div { padding: 20px; }
.avatar { border-radius: 50%; height: 90px; }
```

</Sandpack>

Možete li da nađete problem?

<Hint>Pogledajte šta je unutar vitičastih zagrada. Da li stavljamo pravu stvar tamo?</Hint>

<Solution>

Ovo se dešava zato što ovaj primer renderuje *objekat sam po sebi* u markup umesto stringa: `<h1>{person} lista</h1>` pokušava da renderuje ceo `person` objekat! Uključivanje sirovih objekata kao tekstualnog sadržaja baca grešku jer React ne zna kako želite da ih prikažete.

Da biste popravili, zamenite `<h1>{person} lista</h1>` sa `<h1>{person.name} lista</h1>`:

<Sandpack>

```js
const person = {
  name: 'Gregorio Y. Zara',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name} lista </h1>
      <img
        className="avatar"
        src="https://i.imgur.com/7vQD0fPs.jpg"
        alt="Gregorio Y. Zara"
      />
      <ul>
        <li>Unaprediti video-telefon</li>
        <li>Pripremiti predavanja iz aeronautike</li>
        <li>Raditi na motoru na alkohol</li>
      </ul>
    </div>
  );
}
```

```css
body { padding: 0; margin: 0 }
body > div > div { padding: 20px; }
.avatar { border-radius: 50%; height: 90px; }
```

</Sandpack>

</Solution>

#### Izdvojite informacije u objekat {/*extract-information-into-an-object*/}

Izdvojite URL slike u `person` objekat.

<Sandpack>

```js
const person = {
  name: 'Gregorio Y. Zara',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name} lista</h1>
      <img
        className="avatar"
        src="https://i.imgur.com/7vQD0fPs.jpg"
        alt="Gregorio Y. Zara"
      />
      <ul>
        <li>Unaprediti video-telefon</li>
        <li>Pripremiti predavanja iz aeronautike</li>
        <li>Raditi na motoru na alkohol</li>
      </ul>
    </div>
  );
}
```

```css
body { padding: 0; margin: 0 }
body > div > div { padding: 20px; }
.avatar { border-radius: 50%; height: 90px; }
```

</Sandpack>

<Solution>

Pomerite URL slike u svojstvo `person.imageUrl` i pročitajte ga iz `<img>` oznake koristeći vitičaste zagrade:

<Sandpack>

```js
const person = {
  name: 'Gregorio Y. Zara',
  imageUrl: "https://i.imgur.com/7vQD0fPs.jpg",
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name} lista</h1>
      <img
        className="avatar"
        src={person.imageUrl}
        alt="Gregorio Y. Zara"
      />
      <ul>
        <li>Unaprediti video-telefon</li>
        <li>Pripremiti predavanja iz aeronautike</li>
        <li>Raditi na motoru na alkohol</li>
      </ul>
    </div>
  );
}
```

```css
body { padding: 0; margin: 0 }
body > div > div { padding: 20px; }
.avatar { border-radius: 50%; height: 90px; }
```

</Sandpack>

</Solution>


#### Napišite jedan izraz unutar vitičastih zagrada {/*write-one-expression-inside-curly-braces*/}

U objektu ispod, puni URL slike je podeljen na četiri dela: bazni URL, `imageId`, `imageSize` i ekstenzija fajla.

Mi želimo da URL slike kombinuje ove atribute zajedno: bazni URL (uvek `'https://i.imgur.com/'`), `imageId` (`'7vQD0fP'`), `imageSize` (`'s'`) i ekstenzija fajla (uvek `'.jpg'`). Međutim, nešto nije u redu sa načinom na koji `<img>` oznaka određuje svoj `src`.

Možete li da popravite ovo?

<Sandpack>

```js

const baseUrl = 'https://i.imgur.com/';
const person = {
  name: 'Gregorio Y. Zara',
  imageId: '7vQD0fP',
  imageSize: 's',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name} lista</h1>
      <img
        className="avatar"
        src="{baseUrl}{person.imageId}{person.imageSize}.jpg"
        alt={person.name}
      />
      <ul>
        <li>Unaprediti video-telefon</li>
        <li>Pripremiti predavanja iz aeronautike</li>
        <li>Raditi na motoru na alkohol</li>
      </ul>
    </div>
  );
}
```

```css
body { padding: 0; margin: 0 }
body > div > div { padding: 20px; }
.avatar { border-radius: 50%; }
```

</Sandpack>

Da biste proverili da li je vaša popravka uspela, pokušajte da promenite vrednost `imageSize` u `'b'`. Slika bi trebalo da se promeni nakon vaše izmene.

<Solution>

Možete napisati kao `src={baseUrl + person.imageId + person.imageSize + '.jpg'}`:

1. `{` otvara JavaScript izraz
2. `baseUrl + person.imageId + person.imageSize + '.jpg'` proizvodi ispravan URL string
3. `}` zatvara JavaScript ekspresiju

<Sandpack>

```js
const baseUrl = 'https://i.imgur.com/';
const person = {
  name: 'Gregorio Y. Zara',
  imageId: '7vQD0fP',
  imageSize: 's',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name} lista</h1>
      <img
        className="avatar"
        src={baseUrl + person.imageId + person.imageSize + '.jpg'}
        alt={person.name}
      />
      <ul>
        <li>Unaprediti video-telefon</li>
        <li>Pripremiti predavanja iz aeronautike</li>
        <li>Raditi na motoru na alkohol</li>
      </ul>
    </div>
  );
}
```

```css
body { padding: 0; margin: 0 }
body > div > div { padding: 20px; }
.avatar { border-radius: 50%; }
```

</Sandpack>

Možete pomeriti i izraz u zasebnu funkciju kao što je `getImageUrl` ispod:

<Sandpack>

```js src/App.js
import { getImageUrl } from './utils.js'

const person = {
  name: 'Gregorio Y. Zara',
  imageId: '7vQD0fP',
  imageSize: 's',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name} lista</h1>
      <img
        className="avatar"
        src={getImageUrl(person)}
        alt={person.name}
      />
      <ul>
        <li>Unaprediti video-telefon</li>
        <li>Pripremiti predavanja iz aeronautike</li>
        <li>Raditi na motoru na alkohol</li>
      </ul>
    </div>
  );
}
```

```js src/utils.js
export function getImageUrl(person) {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    person.imageSize +
    '.jpg'
  );
}
```

```css
body { padding: 0; margin: 0 }
body > div > div { padding: 20px; }
.avatar { border-radius: 50%; }
```

</Sandpack>

Promenljive i funkcije vam mogu pomoći da održite markup jednostavnim!

</Solution>

</Challenges>
