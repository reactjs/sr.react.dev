---
title: useId
---

<Intro>

`useId` je React Hook za generisanje jedinstvenih ID-eva koji se mogu proslediti atributima pristupačnosti.

```js
const id = useId()
```

</Intro>

<InlineToc />

---

## Reference {/*reference*/}

### `useId()` {/*useid*/}

Pozovite `useId` na vrhu vaše komponente da biste generisali jedinstveni ID:

```js
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  // ...
```

[Pogledajte još primera ispod.](#usage)

#### Parametri {/*parameters*/}

`useId` ne prima nikakve parametre.

#### Povratne vrednosti {/*returns*/}

`useId` vraća jedinstveni ID string asociran za specifični `useId` poziv u specifičnoj komponenti.

#### Upozorenja {/*caveats*/}

* `useId` je Hook, pa ga možete pozvati samo **na vrhu vaše komponente** ili vaših Hook-ova. Ne možete ga pozvati unutar petlji i uslova. Ako vam je to potrebno, izdvojite novu komponentu i pomerite state u nju.

* `useId` **ne bi trebao da se koristi za generisanje keš ključeva** za [use()](/reference/react/use). ID je stabilan kada je komponenta montirana, ali se može promeniti tokom renderovanja. Trebalo bi generisati keš ključeve na osnovu vaših podataka.

* `useId` **ne bi trebao da se koristi za generisanje ključeva** u listi. [Ključevi trebaju biti generisani na osnovu vaših podataka.](/learn/rendering-lists#where-to-get-your-key)

* `useId` se trenutno ne može koristiti u [asinhronim Server Components](/reference/rsc/server-components#async-components-with-server-components).

---

## Upotreba {/*usage*/}

<Pitfall>

**Nemojte pozivati `useId` da biste generisali ključeve u listi.** [Ključevi trebaju biti generisani na osnovu vaših podataka.](/learn/rendering-lists#where-to-get-your-key)

</Pitfall>

### Generisanje jedinstvenih ID-eva za atribute pristupačnosti {/*generating-unique-ids-for-accessibility-attributes*/}

Pozovite `useId` na vrhu vaše komponente da biste generisali jedinstveni ID:

```js [[1, 4, "passwordHintId"]]
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  // ...
```

Onda možete proslediti <CodeStep step={1}>generisani ID</CodeStep> u različite atribute:

```js [[1, 2, "passwordHintId"], [1, 3, "passwordHintId"]]
<>
  <input type="password" aria-describedby={passwordHintId} />
  <p id={passwordHintId}>
</>
```

**Prođimo kroz primer da vidimo kada je ovo korisno.**

[HTML atributi pristupačnosti](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA) kao što je [`aria-describedby`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-describedby) vam omogućavaju da specificirate dva tag-a koja su povezana. Na primer, možete specificirati da je element (kao što je input) opisan drugim elementom (kao što je paragraph).

U običnom HTML-u napisali biste ovako nešto:

```html {5,8}
<label>
  Šifra:
  <input
    type="password"
    aria-describedby="password-hint"
  />
</label>
<p id="password-hint">
  Šifra treba da sadrži barem 18 karaktera
</p>
```

Međutim, hardkodirani ID-evi poput ovih nisu dobra praksa u React-u. Komponenta može biti renderovana više od jednom na stranici--ali ID-evi moraju biti jedinstveni! Umesto da hardkodirate ID, generišite jedinstveni ID sa `useId`:

```js {4,11,14}
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  return (
    <>
      <label>
        Šifra:
        <input
          type="password"
          aria-describedby={passwordHintId}
        />
      </label>
      <p id={passwordHintId}>
        Šifra treba da sadrži barem 18 karaktera
      </p>
    </>
  );
}
```

Sada, čak iako se `PasswordField` pojavi više puta na ekranu, generisani ID-evi se neće preklapati.

<Sandpack>

```js
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  return (
    <>
      <label>
        Šifra:
        <input
          type="password"
          aria-describedby={passwordHintId}
        />
      </label>
      <p id={passwordHintId}>
        Šifra treba da sadrži barem 18 karaktera
      </p>
    </>
  );
}

export default function App() {
  return (
    <>
      <h2>Unesi šifru</h2>
      <PasswordField />
      <h2>Potvrdi šifru</h2>
      <PasswordField />
    </>
  );
}
```

```css
input { margin: 5px; }
```

</Sandpack>

[Pogledajte ovaj video](https://www.youtube.com/watch?v=0dNzNcuEuOo) da biste uočili razliku u korisničkom iskustvu sa pomoćnim tehnologijama.

<Pitfall>

Sa [renderovanjem na serveru](/reference/react-dom/server), **`useId` zahteva identično stablo komponente na serveru i na klijentu**. Ako se stabla koja renderujete na serveru i klijentu ne poklapaju, ni generisani ID-evi se neće poklapati.

</Pitfall>

<DeepDive>

#### Zašto je useId bolji od inkrementalnog brojača? {/*why-is-useid-better-than-an-incrementing-counter*/}

Možda se pitate zašto je `useId` bolji od inkrementiranja globalne promenljive poput `nextId++`.

Primarna korist od `useId` je da React garantuje da radi sa [renderovanjem na serveru](/reference/react-dom/server). Tokom renderovanja na serveru, vaše komponente generišu HTML izlaz. Kasnije, na klijentu, [hidratacijom](/reference/react-dom/client/hydrateRoot) se povezuju vaši event handler-i sa generisanim HTML-om. Da bi hidratacija radila, klijentski izlaz se mora poklapati sa serverskim HTML-om.

Ovo je teško garantovati sa inkrementalnim brojačem jer se redosled u kom se Client Components hidratizuju možda neće poklapati sa redosledom u kom je emitovan HTML sa servera. Pozivanjem `useId` osiguravate da će hidratacija raditi i da će se izlaz servera poklapati sa klijentskim.

Unutar React-a, `useId` je generisan na osnovu "roditeljske putanje" pozivajuće komponente. Zbog toga, ako se klijentsko i serversko stablo poklapaju, "roditeljska putanja" će se poklapati bez obzira na redosled.

</DeepDive>

---

### Generisanje ID-eva za više povezanih elemenata {/*generating-ids-for-several-related-elements*/}

Ako vam je potrebno da napravite ID-eve za više povezanih elemenata, možete pozvati `useId` da generišete deljeni prefiks za njih:

<Sandpack>

```js
import { useId } from 'react';

export default function Form() {
  const id = useId();
  return (
    <form>
      <label htmlFor={id + '-firstName'}>Ime:</label>
      <input id={id + '-firstName'} type="text" />
      <hr />
      <label htmlFor={id + '-lastName'}>Prezime:</label>
      <input id={id + '-lastName'} type="text" />
    </form>
  );
}
```

```css
input { margin: 5px; }
```

</Sandpack>

Ovo vam omogućava da izbegnete pozivanje `useId`-a za svaki pojedinačni element kojem je potreban jedinstveni ID.

---

### Specificiranje deljenog prefiksa za sve generisane ID-eve {/*specifying-a-shared-prefix-for-all-generated-ids*/}

Ako renderujete više nezavisnih React aplikacija na jednog stranici, prosledite `identifierPrefix` u vaše [`createRoot`](/reference/react-dom/client/createRoot#parameters) ili [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) pozive. Ovo osigurava da se ID-evi generisani u dve različite aplikacije nikad neće preklapati jer će svaki identifikator generisan sa `useId` početi sa različitim prefiksom koji ste specificirali.

<Sandpack>

```html public/index.html
<!DOCTYPE html>
<html>
  <head><title>My app</title></head>
  <body>
    <div id="root1"></div>
    <div id="root2"></div>
  </body>
</html>
```

```js
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  console.log('Generisani identifikator:', passwordHintId)
  return (
    <>
      <label>
        Šifra:
        <input
          type="password"
          aria-describedby={passwordHintId}
        />
      </label>
      <p id={passwordHintId}>
        Šifra treba da sadrži barem 18 karaktera
      </p>
    </>
  );
}

export default function App() {
  return (
    <>
      <h2>Unesi šifru</h2>
      <PasswordField />
    </>
  );
}
```

```js src/index.js active
import { createRoot } from 'react-dom/client';
import App from './App.js';
import './styles.css';

const root1 = createRoot(document.getElementById('root1'), {
  identifierPrefix: 'my-first-app-'
});
root1.render(<App />);

const root2 = createRoot(document.getElementById('root2'), {
  identifierPrefix: 'my-second-app-'
});
root2.render(<App />);
```

```css
#root1 {
  border: 5px solid blue;
  padding: 10px;
  margin: 5px;
}

#root2 {
  border: 5px solid green;
  padding: 10px;
  margin: 5px;
}

input { margin: 5px; }
```

</Sandpack>

---

### Upotreba istog ID prefiksa na klijentu i serveru {/*using-the-same-id-prefix-on-the-client-and-the-server*/}

Ako [renderujete više nezavisnih React aplikacija na istoj stranici](#specifying-a-shared-prefix-for-all-generated-ids), a neke od tih aplikacija su renderovane na serveru, postarajte se da `identifierPrefix` koji prosleđujete u [`hydrateRoot`](/reference/react-dom/client/hydrateRoot) poziv na klijentskoj strani bude isti kao i `identifierPrefix` koji prosleđujete u [serverske API-je](/reference/react-dom/server) poput [`renderToPipeableStream`](/reference/react-dom/server/renderToPipeableStream).

```js
// Server
import { renderToPipeableStream } from 'react-dom/server';

const { pipe } = renderToPipeableStream(
  <App />,
  { identifierPrefix: 'react-app1' }
);
```

```js
// Klijent
import { hydrateRoot } from 'react-dom/client';

const domNode = document.getElementById('root');
const root = hydrateRoot(
  domNode,
  reactNode,
  { identifierPrefix: 'react-app1' }
);
```

Nije potrebno da prosleđujete `identifierPrefix` ako imate samo jednu React aplikaciju na stranici.
