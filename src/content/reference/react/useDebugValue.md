---
title: useDebugValue
---

<Intro>

`useDebugValue` je React Hook koji vam omogućava da dodate labelu za vaš prilagođeni Hook u [React DevTools-u](/learn/react-developer-tools).

```js
useDebugValue(value, format?)
```

</Intro>

<InlineToc />

---

## Reference {/*reference*/}

### `useDebugValue(value, format?)` {/*usedebugvalue*/}

Pozovite `useDebugValue` na vrhu vašeg [prilagođenog Hook-a](/learn/reusing-logic-with-custom-hooks) da biste prikazali čitljivu debug vrednost:

```js
import { useDebugValue } from 'react';

function useOnlineStatus() {
  // ...
  useDebugValue(isOnline ? 'Online' : 'Offline');
  // ...
}
```

[Pogledajte još primera ispod.](#usage)

#### Parametri {/*parameters*/}

* `value`: Vrednost koju želite da prikažete u React DevTools-u. Može imati bilo koji tip.
* **opcioni** `format`: Funkcija za formatiranje. Kada se komponenta pregleda, React DevTools će pozvati funkciju za formatiranje sa argumentom `value`, a onda prikazati formatiranu vrednost koja je vraćena (koja može imati bilo koji tip). Ako ne specificirate funkciju za formatiranje, prikazaće se originalni `value`.

#### Povratne vrednosti {/*returns*/}

`useDebugValue` ne vraća ništa.

## Upotreba {/*usage*/}

### Dodavanje labele u prilagođeni Hook {/*adding-a-label-to-a-custom-hook*/}

Pozovite `useDebugValue` na vrhu vašeg [prilagođenog Hook-a](/learn/reusing-logic-with-custom-hooks) da biste prikazali čitljivu <CodeStep step={1}>debug vrednost</CodeStep> za [React DevTools](/learn/react-developer-tools).

```js [[1, 5, "isOnline ? 'Online' : 'Offline'"]]
import { useDebugValue } from 'react';

function useOnlineStatus() {
  // ...
  useDebugValue(isOnline ? 'Online' : 'Offline');
  // ...
}
```

Ovo daje komponentama koje pozivaju `useOnlineStatus` labelu poput `OnlineStatus: "Online"` kada ih pregledate:

![Screenshot React DevTools-a koji prikazuje debug vrednost](/images/docs/react-devtools-usedebugvalue.png)

Bez `useDebugValue` poziva, samo će podaci (u ovom primeru, `true`) biti prikazani.

<Sandpack>

```js
import { useOnlineStatus } from './useOnlineStatus.js';

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Online' : '❌ Diskonektovano'}</h1>;
}

export default function App() {
  return <StatusBar />;
}
```

```js src/useOnlineStatus.js active
import { useSyncExternalStore, useDebugValue } from 'react';

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, () => navigator.onLine, () => true);
  useDebugValue(isOnline ? 'Online' : 'Offline');
  return isOnline;
}

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
```

</Sandpack>

<Note>

Nemojte dodavati debug vrednosti u svaki prilagođeni Hook. Najkorisnije je za prilagođene Hook-ove koji su deo deljenih biblioteka i koji imaju kompleksnu unutrašnju strukturu podataka koju je teško pregledati.

</Note>

---

### Odlaganje formatiranja debug vrednosti {/*deferring-formatting-of-a-debug-value*/}

Možete proslediti i funkciju za formatiranje kao drugi argument u `useDebugValue`:

```js [[1, 1, "date", 18], [2, 1, "date.toDateString()"]]
useDebugValue(date, date => date.toDateString());
```

Vaša funkcija za formatiranje će primiti <CodeStep step={1}>debug vrednost</CodeStep> kao parametar i trebalo bi da vrati <CodeStep step={2}>formatiranu vrednost za prikazivanje</CodeStep>. Kada vaša komponenta bude pregledana, React DevTools će pozvati ovu funkciju i prikazazi njen rezultat.

Ovo vam omogućava da izbegnete izvršavanje potencijalno skupe logike formatiranja osim ako komponenta zapravo nije pregledana. Na primer, ako je `date`, vrednost Date tipa, ovo izbegava pozivanje `toDateString()` za svaki render.
