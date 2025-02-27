---
title: Redosled serijskih ažuriranja state-a
---

<Intro>

Postavljanje state promenljive će staviti novi render u red čekanja. Ali, ponekad želite izvršiti više operacija nad promenljivom pre zakazivanja narednog rendera. Da biste to uradili, potrebno je razumeti kako React batch-uje ažuriranja state-a.

</Intro>

<YouWillLearn>

* Šta je "batch-ovanje" i kako ga React koristi da procesira više ažuriranja state-a
* Kako da primenite više ažuriranja u nizu na jednu promenljivu state-a

</YouWillLearn>

## React batch-uje ažuriranja state-a {/*react-batches-state-updates*/}

Očekivali biste da klikom na dugme "+3" brojač bude inkrementiran tri puta jer se `setNumber(number + 1)` poziva tri puta:

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

Međutim, kao što se verovatno sećate iz prethodne sekcije, [vrednosti state-a su fiksirane za svaki render](/learn/state-as-a-snapshot#rendering-takes-a-snapshot-in-time), pa je vrednost za `number` u event handler-u prvog rendera uvek `0`, nevezano od toga koliko puta pozovete `setNumber(1)`:

```js
setNumber(0 + 1);
setNumber(0 + 1);
setNumber(0 + 1);
```

Ali, ovde postoji još jedan faktor. **React čeka da se *sav* kod u event handler-ima izvrši pre nego što procesira ažuriranja state-a.** Zato se ponovni render jedino dešava *nakon* svih `setNumber()` poziva.

Ovo vas može podsetiti na konobara koji uzima porudžbinu u restoranu. Konobar ne trči u kuhinju nakon što pomenete prvo jelo! Umesto toga, dopušta vam da završite porudžbinu, da je promenite, a čak uzima i porudžbine od drugih ljudi za stolom.

<Illustration src="/images/docs/illustrations/i_react-batching.png" alt="Elegantni kursor u restoranu poručuje više puta pomoću React-a, koji igra ulogu konobara. Nakon što više puta pozove setState(), konobar upisuje poslednje što je poručeno, kao konačnu porudžbinu." />

Ovo vam omogućava da ažurirate više state promenljivih--čak i iz različitih komponenata--bez okidanja previše [ponovnih rendera](/learn/render-and-commit#re-renders-when-state-updates). Ali, to takođe znači da UI neće biti ažuriran _sve dok_ se event handler i sav kod unutra ne izvrši. Ovo ponašanje, poznatije kao **batch-ovanje**, čini vašu React aplikaciju mnogo bržom. Takođe, izbegava se obrada zbunjujućih "polu-završenih" rendera gde su samo neke promenljive ažurirane.

**React ne radi batch-ovanje *više* namernih event-ova poput klikova**--svaki klik se obrađuje zasebno. Budite uvereni da React radi batch-ovanje samo kada je to sigurno. Ovo osigurava da se, na primer, ako prvi klik dugmeta onemogućuje formu, spreči da drugi klik uradi submit ponovo.

## Ažuriranje istog state-a više puta pre narednog rendera {/*updating-the-same-state-multiple-times-before-the-next-render*/}

Ovo je neuobičajen slučaj, ali ako biste želeli da ažurirate istu state promenljivu više puta pre narednog rendera, umesto da prosledite *narednu state vrednost* poput `setNumber(number + 1)`, možete proslediti *funkciju* koja računa naredni state na osnovu prethodnog u redu čekanja, poput `setNumber(n => n + 1)`. Ovo je način da kažete React-u da "uradi nešto sa vrednosti state-a" umesto da je samo zameni.

Probajte sada da inkrementirate brojač:

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1);
        setNumber(n => n + 1);
        setNumber(n => n + 1);
      }}>+3</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

Ovde, `n => n + 1` se naziva **updater funkcija**. Kada je prosledite u setter za state:

1. React postavlja ovu funkciju u red čekanja da bi bila procesirana nakon što se sav ostali kod u event handler-u izvrši.
2. Tokom narednog rendera, React prolazi kroz red čekanja i daje vam konačni ažurirani state.

```js
setNumber(n => n + 1);
setNumber(n => n + 1);
setNumber(n => n + 1);
```

Evo kako React tumači ove linije koda dok izvršava event handler:

1. `setNumber(n => n + 1)`: `n => n + 1` je funkcija. React je dodaje u red čekanja.
1. `setNumber(n => n + 1)`: `n => n + 1` je funkcija. React je dodaje u red čekanja.
1. `setNumber(n => n + 1)`: `n => n + 1` je funkcija. React je dodaje u red čekanja.

Kada se pozove `useState` tokom narednog rendera, React prolazi kroz red čekanja. Prethodni `number` state je bio `0`, pa je to ono što React prosleđuje u prvu updater funkciju kao argument `n`. Nakon toga, React uzima povratnu vrednost prethodne updater funkcije i prosleđuje je u narednu updater funkciju kao `n`, i tako dalje:

| ažuriranje u redu čekanja | `n` | povratna vrednost |
|-----                      |-----|-----              |
| `n => n + 1`              | `0` | `0 + 1 = 1`       |
| `n => n + 1`              | `1` | `1 + 1 = 2`       |
| `n => n + 1`              | `2` | `2 + 1 = 3`       |

React čuva `3` kao konačan rezultat i vraća ga iz `useState`.

Zato se, klikom na "+3" u primeru iznad, vrednost ispravno inkrementira za 3.

### Šta se dešava ako ažurirate state nakon što ga zamenite {/*what-happens-if-you-update-state-after-replacing-it*/}

Šta je sa ovim event handler-om? Šta mislite da će `number` biti u narednom renderu?

```js
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
}}>
```

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
      }}>Povećaj broj</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

Evo šta ovaj event handler govori React-u da uradi:

1. `setNumber(number + 5)`: `number` je `0`, znači `setNumber(0 + 5)`. React dodaje *"zameni sa `5`"* u red čekanja.
2. `setNumber(n => n + 1)`: `n => n + 1` je updater funkcija. React dodaje *tu funkciju* u red čekanja.

Tokom narednog rendera React prolazi kroz red čekanja za state:

| ažuriranje u redu čekanja | `n`                  | povratna vrednost |
|-----                      |-----                 |-----              |
| "zameni sa `5`"           | `0` (neupotrebljeno) | `5`               |
| `n => n + 1`              | `5`                  | `5 + 1 = 6`       |

React čuva `6` kao konačan rezultat i vraća ga iz `useState`.

<Note>

Možda ste primetili da `setState(5)` zapravo radi kao `setState(n => 5)`, ali `n` nije upotrebljeno!

</Note>

### Šta se dešava ako zamenite state nakon što ga ažurirate {/*what-happens-if-you-replace-state-after-updating-it*/}

Hajde da probamo još jedan primer. Šta mislite da će `number` biti u narednom renderu?

```js
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
  setNumber(42);
}}>
```

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
        setNumber(42);
      }}>Povećaj broj</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

Evo kako React tumači ove linije koda dok izvršava event handler:

1. `setNumber(number + 5)`: `number` je `0`, znači `setNumber(0 + 5)`. React dodaje *"zameni sa `5`"* u red čekanja.
2. `setNumber(n => n + 1)`: `n => n + 1` je updater funkcija. React dodaje *tu funkciju* u red čekanja.
3. `setNumber(42)`: React dodaje *"zameni sa `42`"* u red čekanja.

Tokom narednog rendera React prolazi kroz red čekanja za state:

| ažuriranje u redu čekanja | `n`                  | povratna vrednost |
|-----                      |-----                 |-----              |
| "zameni sa `5`"           | `0` (neupotrebljeno) | `5`               |
| `n => n + 1`              | `5`                  | `5 + 1 = 6`       |
| "zameni sa  `42`"         | `6` (neupotrebljeno) | `42`              |

React čuva `42` kao konačan rezultat i vraća ga iz `useState`.

Da sumiramo, evo kako možete razmišljati o onome što prosleđujete u `setNumber` setter za state:

* **Updater funkcija** (npr. `n => n + 1`) se dodaje u red čekanja.
* **Bilo koja druga vrednost** (npr. broj `5`) dodaje "zameni sa `5`" u red čekanja, ignorišući ono što je već u tom redu.

Kada se event handler izvrši, React će pokrenuti ponovni render. Tokom ponovnog rendera, React će obraditi red čekanja. Updater funkcije se izvršavaju tokom renderovanja, što znači da **updater funkcije moraju biti [čiste](/learn/keeping-components-pure)** i da samo *vrate* rezultat. Nemojte pokušavati da postavite state unutar njih ili da izvršite druge propratne efekte. U Strict Mode-u, React će izvršiti svaku updater funkciju dvaput (ali će odbaciti rezultat druge) kako bi vam pomogao da pronađete greške.

### Konvencije imenovanja {/*naming-conventions*/}

Uobičajeno je nazvati argument updater funkcije po prvim slovima odgovarajuće state promenljive:

```js
setEnabled(e => !e);
setLastName(ln => ln.reverse());
setFriendCount(fc => fc * 2);
```

Ako preferirate opširniji kod, druga konvencija je ponavljanje punog imena state promenljive, poput `setEnabled(enabled => !enabled)`, ili da koristite prefiks, npr. `setEnabled(prevEnabled => !prevEnabled)`.

<Recap>

* Postavljanje state-a ne menja promenljivu u trenutnom renderu, ali zahteva novi render.
* React procesira ažuriranje state-a nakon što su se event handler-i izvršili. Ovo se naziva batch-ovanje.
* Da biste ažurirali neki state više puta u jednom event-u, možete koristiti `setNumber(n => n + 1)` updater funkciju.

</Recap>



<Challenges>

#### Popraviti broj zahteva {/*fix-a-request-counter*/}

Radite na aplikaciji za prodaju umetnosti koja omogućava korisniku da zatraži više porudžbina umetničkih predmeta istovremeno. Svaki put kada korisnik pritisne dugme "Kupi", brojač "Na čekanju" treba da se poveća za jedan. Nakon tri sekunde, brojač "Na čekanju" treba da se smanji, a brojač "Završeno" da se uveća.

Međutim, brojač "Na čekanju" se ne ponaša kao što bi trebalo. Nakon što kliknete "Kupi", smanji se na `-1` (što ne bi trebalo da je moguće!). Ako kliknete brzo dvaput, deluje da se oba brojača ponašaju nepredvidivo.

Zašto se ovo dešava? Popravite oba brojača.

<Sandpack>

```js
import { useState } from 'react';

export default function RequestTracker() {
  const [pending, setPending] = useState(0);
  const [completed, setCompleted] = useState(0);

  async function handleClick() {
    setPending(pending + 1);
    await delay(3000);
    setPending(pending - 1);
    setCompleted(completed + 1);
  }

  return (
    <>
      <h3>
        Na čekanju: {pending}
      </h3>
      <h3>
        Završeno: {completed}
      </h3>
      <button onClick={handleClick}>
        Kupi     
      </button>
    </>
  );
}

function delay(ms) {
  return new Promise(resolve => {
    setTimeout(resolve, ms);
  });
}
```

</Sandpack>

<Solution>

Unutar `handleClick` event handler-a, vrednosti `pending` i `completed` odgovaraju onome što su bili u trenutku klik event-a. Za prvi render, `pending` je bio `0`, pa `setPending(pending - 1)` postaje `setPending(-1)`, što je pogrešno. Pošto želite da *inkrementirate* ili *dekrementirate* brojače, možete proslediti updater funkcije, umesto da ih postavite na konkretnu vrednost određenu tokom klika:

<Sandpack>

```js
import { useState } from 'react';

export default function RequestTracker() {
  const [pending, setPending] = useState(0);
  const [completed, setCompleted] = useState(0);

  async function handleClick() {
    setPending(p => p + 1);
    await delay(3000);
    setPending(p => p - 1);
    setCompleted(c => c + 1);
  }

  return (
    <>
      <h3>
        Na čekanju: {pending}
      </h3>
      <h3>
        Završeno: {completed}
      </h3>
      <button onClick={handleClick}>
        Kupi     
      </button>
    </>
  );
}

function delay(ms) {
  return new Promise(resolve => {
    setTimeout(resolve, ms);
  });
}
```

</Sandpack>

Ovo osigurava da kada inkrementirate ili dekrementirate brojač, to uradite u odnosu na *poslednji* state, umesto u odnosu na ono što je state bio u trenutku klika.

</Solution>

#### Implementirati red čekanja za state samostalno {/*implement-the-state-queue-yourself*/}

U ovom izazovu, ponovo ćete implementirati sitan deo React-a od nule! Nije tako teško kao što zvuči.

Scroll-ujte kroz sandbox prikaz. Primetite da obuhvata **četiri slučaja**. Oni odgovaraju primerima koje ste videli ranije na ovoj stranici. Vaš zadatak je da implementirate `getFinalState` funkciju tako da vraća ispravan rezultat u svim tim slučajevima. Ako je implementirate ispravno, sva četiri testa će proći.

Primate dva argumenta: `baseState` je inicijalni state (kao `0`), a `queue` je niz koji sadrži mešavinu brojeva (npr. `5`) i updater funkcija (npr. `n => n + 1`) u redosledu u kojem su dodati.

Vaš zadatak je da vratite konačni state, kao što tabele na ovoj stranici pokazuju!

<Hint>

Ako ste se zaglavili, počnite sa ovakvom strukturom:

```js
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  for (let update of queue) {
    if (typeof update === 'function') {
      // TODO: primeni updater funkciju
    } else {
      // TODO: zameni state
    }
  }

  return finalState;
}
```

Popunite nedostajuće linije!

</Hint>

<Sandpack>

```js src/processQueue.js active
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  // TODO: uradi nešto sa ovim nizom...

  return finalState;
}
```

```js src/App.js
import { getFinalState } from './processQueue.js';

function increment(n) {
  return n + 1;
}
increment.toString = () => 'n => n+1';

export default function App() {
  return (
    <>
      <TestCase
        baseState={0}
        queue={[1, 1, 1]}
        expected={1}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          increment,
          increment,
          increment
        ]}
        expected={3}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          5,
          increment,
        ]}
        expected={6}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          5,
          increment,
          42,
        ]}
        expected={42}
      />
    </>
  );
}

function TestCase({
  baseState,
  queue,
  expected
}) {
  const actual = getFinalState(baseState, queue);
  return (
    <>
      <p>Inicijalni state: <b>{baseState}</b></p>
      <p>Niz: <b>[{queue.join(', ')}]</b></p>
      <p>Očekivani rezultat: <b>{expected}</b></p>
      <p style={{
        color: actual === expected ?
          'green' :
          'red'
      }}>
        Vaš rezultat: <b>{actual}</b>
        {' '}
        ({actual === expected ?
          'tačno' :
          'netačno'
        })
      </p>
    </>
  );
}
```

</Sandpack>

<Solution>

Ovo je tačan algoritan opisan na ovoj stranici koji React koristi da izračuna konačni state:

<Sandpack>

```js src/processQueue.js active
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  for (let update of queue) {
    if (typeof update === 'function') {
      // Primeni updater funkciju.
      finalState = update(finalState);
    } else {
      // Zameni naredni state.
      finalState = update;
    }
  }

  return finalState;
}
```

```js src/App.js
import { getFinalState } from './processQueue.js';

function increment(n) {
  return n + 1;
}
increment.toString = () => 'n => n+1';

export default function App() {
  return (
    <>
      <TestCase
        baseState={0}
        queue={[1, 1, 1]}
        expected={1}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          increment,
          increment,
          increment
        ]}
        expected={3}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          5,
          increment,
        ]}
        expected={6}
      />
      <hr />
      <TestCase
        baseState={0}
        queue={[
          5,
          increment,
          42,
        ]}
        expected={42}
      />
    </>
  );
}

function TestCase({
  baseState,
  queue,
  expected
}) {
  const actual = getFinalState(baseState, queue);
  return (
    <>
      <p>Inicijalni state: <b>{baseState}</b></p>
      <p>Niz: <b>[{queue.join(', ')}]</b></p>
      <p>Očekivani rezultat: <b>{expected}</b></p>
      <p style={{
        color: actual === expected ?
          'green' :
          'red'
      }}>
        Vaš rezultat: <b>{actual}</b>
        {' '}
        ({actual === expected ?
          'tačno' :
          'netačno'
        })
      </p>
    </>
  );
}
```

</Sandpack>

Sada znate kako ovaj deo React-a radi!

</Solution>

</Challenges>
