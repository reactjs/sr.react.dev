---
title: Evakuacioni izlazi
---

<Intro>

Neke od vaših komponenata će možda trebati da kontrolišu i da se sinhronizuju sa sistemima izvan React-a. Na primer, možda ćete morati da fokusirate input uz pomoć API-ja pretraživača, da pustite i pauzirate video plejer koji nije implementiran u React-u ili da se povežete i slušate poruke sa udaljenog servera. U ovom poglavlju naučićete evakuacione izlaze koji vam omogućavaju da "izađete iz" React-a i povežete se na eksterne sisteme. Većina logike u vašoj aplikaciji, kao i tok podataka, ne bi trebalo da se oslanjaju na ove funkcionalnosti.

</Intro>

<YouWillLearn isChapter={true}>

* [Kako da "upamtite" informaciju bez ponovnog renderovanja](/learn/referencing-values-with-refs)
* [Kako da pristupite DOM elementima kojima upravlja React](/learn/manipulating-the-dom-with-refs)
* [Kako da sinhronizujete komponente sa eksternim sistemima](/learn/synchronizing-with-effects)
* [Kako da uklonite nepotrebne Effect-e iz vaših komponenti](/learn/you-might-not-need-an-effect)
* [Kako se životni ciklus Effect-a razlikuje od životnog ciklusa komponente](/learn/lifecycle-of-reactive-effects)
* [Kako da sprečite da neke vrednosti ponovo pokrenu Effect-e](/learn/separating-events-from-effects)
* [Kako da učinite da se vaš Effect ređe pokreće](/learn/removing-effect-dependencies)
* [Kako da delite logiku među komponentama](/learn/reusing-logic-with-custom-hooks)

</YouWillLearn>

## Referenciranje vrednosti sa ref-ovima {/*referencing-values-with-refs*/}

Kada želite da komponenta "upamti" neku informaciju, ali ne želite da ta informacija [pokrene nove rendere](/learn/render-and-commit), možete koristiti *ref*:

```js
const ref = useRef(0);
```

Kao i state, React čuva ref-ove između rendera. Međutim, postavljanje state-a ponovo renderuje komponentu. Promena ref-a to ne radi! Možete pristupiti trenutnoj vrednosti tog ref-a kroz `ref.current` polje.

<Sandpack>

```js
import { useRef } from 'react';

export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert('Kliknuli ste ' + ref.current + ' puta!');
  }

  return (
    <button onClick={handleClick}>
      Klikni me!
    </button>
  );
}
```

</Sandpack>

Ref je kao tajni džep vaše komponente koji React ne prati. Na primer, možete koristiti ref-ove da čuvate [timeout ID-eve](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout#return_value), [DOM elemente](https://developer.mozilla.org/en-US/docs/Web/API/Element) i ostale objekte koji ne utiču na izlaz renderovanja komponente.

<LearnMore path="/learn/referencing-values-with-refs">

Pročitajte **[Referenciranje vrednosti sa Ref-ovima](/learn/referencing-values-with-refs)** da naučite kako da koristite ref-ove da upamtite informaciju.

</LearnMore>

## Manipulisanje DOM-om sa ref-ovima {/*manipulating-the-dom-with-refs*/}

React automatski ažurira DOM kako bi odgovarao izlazu renderovanja, tako da vaše komponente neće često morati da manipulišu DOM-om. Međutim, ponekad vam može biti potreban pristup DOM elementima kojima upravlja React—na primer, da biste se fokusirali na čvor, scroll-ovali na njega ili izmerili njegovu veličinu i poziciju. Ne postoji ugrađeni način da to uradite u React-u, pa će vam biti potreban ref na DOM čvor. Na primer, klik na dugme će fokusirati input upotrebom ref-a:

<Sandpack>

```js
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Fokusiraj input
      </button>
    </>
  );
}
```

</Sandpack>

<LearnMore path="/learn/manipulating-the-dom-with-refs">

Pročitajte **[Manipulisanje DOM-om sa Ref-ovima](/learn/manipulating-the-dom-with-refs)** da naučite kako da pristupite DOM elementima kojima upravlja React.

</LearnMore>

## Sinhronizacija sa Effect-ima {/*synchronizing-with-effects*/}

Neke komponente trebaju da se sinhronizuju sa eksternim sistemima. Na primer, želite da kontrolišete komponente koje nisu pisane u React-u na osnovu React state-a, da uspostavite konekciju sa serverom ili da pošaljete analitički log kad se komponenta pojavi na ekranu. Za razliku od event handler-a, koji vam omogućavaju da rukujete određenim event-ima, *Effect-i* vam omogućavaju da izvršite kod nakon rendera. Koristite ih da sinhronizujete vaše komponente sa sistemom izvan React-a.

Kliknite na Pusti/Pauziraj nekoliko puta da vidite kako video plejer ostaje sinhronizovan sa vrednošću `isPlaying` prop-a:

<Sandpack>

```js
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }, [isPlaying]);

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pauziraj' : 'Pusti'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

```css
button { display: block; margin-bottom: 20px; }
video { width: 250px; }
```

</Sandpack>

Mnogi Effect-i takođe i "čiste" za sobom. Na primer, Effect koji uspostavlja konekciju sa serverom za dopisivanje bi trebao da vrati *cleanup funkciju* koja govori React-u kako da diskonektuje vašu komponentu sa tog servera:

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, []);
  return <h1>Dobro došli u čet!</h1>;
}
```

```js src/chat.js
export function createConnection() {
  // Stvarna implementacija bi se zapravo konektovala na server
  return {
    connect() {
      console.log('✅ Konektovanje...');
    },
    disconnect() {
      console.log('❌ Diskonektovano.');
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
```

</Sandpack>

U toku razvoja, React će odmah pokrenuti i očistiti vaš Effect jedan dodatni put. Zbog toga vidite poruku `"✅ Konektovanje..."` dvaput. Ovo osigurava da ne zaboravite da implementirate cleanup funkciju.

<LearnMore path="/learn/synchronizing-with-effects">

Pročitajte **[Sinhronizacija sa Effect-ima](/learn/synchronizing-with-effects)** da biste naučili kako da sinhronizujete komponente sa eksternim sistemima.

</LearnMore>

## Možda vam neće trebati Effect {/*you-might-not-need-an-effect*/}

Effect-i su evakuacioni izlaz u React paradigmi. Omogućavaju vam da "izađete iz" iz React-a i sinhronizujete vaše komponente sa nekim eksternim sistemom. Ako eksterni sistem ne postoji (na primer, ako želite da ažurirate state komponente kada se neki props-i ili state promene), neće vam trebati Effect. Uklanjanje nepotrebnih Effect-a će vaš kod učiniti lakšim za praćenje, bržim za pokretanje i biće manje podložan greškama.

Ovo su dva uobičajena slučaja u kojima vam ne trebaju Effect-i:
- **Ne trebaju vam Effect-i da transformišete podatke za renderovanje.**
- **Ne trebaju vam Effect-i da rukujete korisničkim event-ima.**

Na primer, ne treba vam Effect da prilagodite neki state na osnovu drugog state-a:

```js {expectedErrors: {'react-compiler': [8]}} {5-9}
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // 🔴 Izbegavati: suvišan state i nepotreban Effect
  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
  // ...
}
```

Umesto toga izračunajte koliko god možete tokom renderovanja:

```js {4-5}
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // ✅ Dobro: izračunato tokom renderovanja
  const fullName = firstName + ' ' + lastName;
  // ...
}
```

Međutim, Effect-i su vam *potrebni* da se sinhronizujete sa eksternim sistemima.

<LearnMore path="/learn/you-might-not-need-an-effect">

Pročitajte **[Možda vam neće trebati Effect](/learn/you-might-not-need-an-effect)** da znate kako da uklonite nepotrebne Effect-e.

</LearnMore>

## Životni ciklus reaktivnih effect-a {/*lifecycle-of-reactive-effects*/}

Effect-i imaju drugačiji životni ciklus od komponenata. Komponente se mogu montirati, ažurirati i demontirati. Effect može uraditi samo dve stvari: da započne sinhronizaciju nečega i da kasnije prekine tu sinhronizaciju. Ovaj ciklus se može dogoditi više puta ako vaš Effect zavisi od props-a i state-a koji se vremenom menjaju.

Ovaj Effect zavisi od vrednosti `roomId` prop-a. Props-i su *reaktivne vrednosti*, što znači da se mogu promeniti pri ponovnom renderu. Primetite da se Effect *ponovo sinhronizuje* (i ponovo konektuje na server) ako se `roomId` promeni:

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Dobro došli u sobu {roomId}!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('opšte');
  return (
    <>
      <label>
        Odaberi sobu za dopisivanje:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="opšte">opšte</option>
          <option value="putovanje">putovanje</option>
          <option value="muzika">muzika</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // Stvarna implementacija bi se zapravo konektovala na server
  return {
    connect() {
      console.log('✅ Konektovanje na sobu "' + roomId + '" na adresi ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Diskonektovano iz sobe "' + roomId + '" na adresi ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

React vam pruža linter pravilo za proveru da li ste ispravno specificirali zavisnosti Effect-a. Ako zaboravite da specificirate `roomId` u listi zavisnosti u primeru iznad, linter će automatski pronaći taj bug.

<LearnMore path="/learn/lifecycle-of-reactive-effects">

Pročitajte **[Životni ciklus reaktivnih Effect-a](/learn/lifecycle-of-reactive-effects)** da biste naučili kako se životni ciklus Effect-a razlikuje od životnog ciklusa komponente.

</LearnMore>

## Odvajanje event-ova od Effect-a {/*separating-events-from-effects*/}

<Wip>

Ova sekcija opisuje **eksperimentalni API koji još uvek nije deo** stabilne verzije React-a.

</Wip>

Event handler-i se ponovo pokreću samo kada ponovo izvršite istu interakciju. Za razliku od event handler-a, Effect-i se ponovo sinhronizuju kada se bilo koja vrednost koju čitaju, poput props-a ili state-a, razlikuje u odnosu na prethodni render. Ponekad želite kombinaciju ta dva ponašanja: Effect koji se ponovo pokreće kao odgovor na neke vrednosti, ali ne i na neke druge.

Sav kod unutar Effect-a je *reaktivan*. Pokrenuće se ponovo ako se neka reaktivna vrednost promenila zbog ponovnog rendera. Na primer, ovaj Effect će se ponovo konektovati na čet ako se `roomId` ili `theme` promene:

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js
import { useState, useEffect } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('Konektovano!', theme);
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, theme]);

  return <h1>Dobro došli u sobu {roomId}!</h1>
}

export default function App() {
  const [roomId, setRoomId] = useState('opšte');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        Odaberi sobu za dopisivanje:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="opšte">opšte</option>
          <option value="putovanje">putovanje</option>
          <option value="muzika">muzika</option>
        </select>
      </label>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Koristi tamnu temu
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        theme={isDark ? 'dark' : 'light'} 
      />
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // Stvarna implementacija bi se zapravo konektovala na server
  let connectedCallback;
  let timeout;
  return {
    connect() {
      timeout = setTimeout(() => {
        if (connectedCallback) {
          connectedCallback();
        }
      }, 100);
    },
    on(event, callback) {
      if (connectedCallback) {
        throw Error('Ne možete dodati handler dvaput.');
      }
      if (event !== 'connected') {
        throw Error('Samo je "connected" event podržan.');
      }
      connectedCallback = callback;
    },
    disconnect() {
      clearTimeout(timeout);
    }
  };
}
```

```js src/notifications.js
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme) {
  Toastify({
    text: message,
    duration: 2000,
    gravity: 'top',
    position: 'right',
    style: {
      background: theme === 'dark' ? 'black' : 'white',
      color: theme === 'dark' ? 'white' : 'black',
    },
  }).showToast();
}
```

```css
label { display: block; margin-top: 10px; }
```

</Sandpack>

Ovo nije idealno. Želite da se ponovo konektujete na čet samo ako se `roomId` promeni. Promenom `theme` vrednosti ne bi trebalo da se ponovo konektujete na čet! Pomerite kod koji čita `theme` izvan vašeg Effect-a u *Effect Event*:

<Sandpack>

```json package.json hidden
{
  "dependencies": {
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

```js
import { useState, useEffect } from 'react';
import { useEffectEvent } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Konektovano!', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Dobro došli u sobu {roomId}!</h1>
}

export default function App() {
  const [roomId, setRoomId] = useState('opšte');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        Odaberi sobu za dopisivanje:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="opšte">opšte</option>
          <option value="putovanje">putovanje</option>
          <option value="muzika">muzika</option>
        </select>
      </label>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Koristi tamnu temu
      </label>
      <hr />
      <ChatRoom
        roomId={roomId}
        theme={isDark ? 'dark' : 'light'} 
      />
    </>
  );
}
```

```js src/chat.js
export function createConnection(serverUrl, roomId) {
  // Stvarna implementacija bi se zapravo konektovala na server
  let connectedCallback;
  let timeout;
  return {
    connect() {
      timeout = setTimeout(() => {
        if (connectedCallback) {
          connectedCallback();
        }
      }, 100);
    },
    on(event, callback) {
      if (connectedCallback) {
        throw Error('Ne možete dodati handler dvaput.');
      }
      if (event !== 'connected') {
        throw Error('Samo je "connected" event podržan.');
      }
      connectedCallback = callback;
    },
    disconnect() {
      clearTimeout(timeout);
    }
  };
}
```

```js src/notifications.js hidden
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme) {
  Toastify({
    text: message,
    duration: 2000,
    gravity: 'top',
    position: 'right',
    style: {
      background: theme === 'dark' ? 'black' : 'white',
      color: theme === 'dark' ? 'white' : 'black',
    },
  }).showToast();
}
```

```css
label { display: block; margin-top: 10px; }
```

</Sandpack>

Kod unutar Effect Event-ova nije reaktivan, pa promena `theme` vrednosti ne pokreće ponovnu konekciju u Effect-u.

<LearnMore path="/learn/separating-events-from-effects">

Pročitajte **[Odvajanje Event-ova od Effect-a](/learn/separating-events-from-effects)** da biste naučili kako da sprečite da neke vrednosti ponovo pokrenu Effect-e.

</LearnMore>

## Uklanjanje zavisnosti Effect-a {/*removing-effect-dependencies*/}

Kada pišete Effect, linter će verifikovati da li ste uključili svaku reaktivnu vrednost (poput props-a i state-a) koju Effect čita u listi zavisnosti vašeg Effect-a. Ovo osigurava da vaš Effect ostane sinhronizovan sa poslednjim props-ima i state-om vaše komponente. Nepotrebne zavisnosti mogu prouzrokovati da se vaš Effect pokreće previše često ili da čak naprave beskonačnu petlju. Način njihovog uklanjanja zavisi od slučaja.

Na primer, ovaj Effect zavisi od `options` objekta koji se ponovo kreira svaki put kada izmenite input:

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);

  return (
    <>
      <h1>Dobro došli u sobu {roomId}!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('opšte');
  return (
    <>
      <label>
        Odaberi sobu za dopisivanje:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="opšte">opšte</option>
          <option value="putovanje">putovanje</option>
          <option value="muzika">muzika</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/chat.js
export function createConnection({ serverUrl, roomId }) {
  // Stvarna implementacija bi se zapravo konektovala na server
  return {
    connect() {
      console.log('✅ Konektovanje na sobu "' + roomId + '" na adresi ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Diskonektovano iz sobe "' + roomId + '" na adresi ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Ne želite da se ponovo konektujete na čet svaki put kada pišete poruku u taj čet. Da biste popravili ovaj problem, pomerite pravljenje `options` objekta unutar Effect-a tako da Effect zavisi samo od `roomId` stringa:

<Sandpack>

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return (
    <>
      <h1>Dobro došli u sobu {roomId}!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('opšte');
  return (
    <>
      <label>
        Odaberi sobu za dopisivanje:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="opšte">opšte</option>
          <option value="putovanje">putovanje</option>
          <option value="muzika">muzika</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js src/chat.js
export function createConnection({ serverUrl, roomId }) {
  // Stvarna implementacija bi se zapravo konektovala na server
  return {
    connect() {
      console.log('✅ Konektovanje na sobu "' + roomId + '" na adresi ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Diskonektovano iz sobe "' + roomId + '" na adresi ' + serverUrl);
    }
  };
}
```

```css
input { display: block; margin-bottom: 20px; }
button { margin-left: 10px; }
```

</Sandpack>

Primetite da izmenu liste zavisnosti niste započeli uklanjanjem `options` zavisnosti. To bi bilo pogrešno. Umesto toga, promenili ste okolni kod tako da je ta zavisnost postala *nepotrebna*. Zamislite listu zavisnosti kao listu svih reaktivnih vrednosti koje se koriste u kodu vašeg Effect-a. Ne birate namerno šta da stavite u tu listu. Ta lista opisuje vaš kod. Da biste promenili listu zavisnosti, promenite kod.

<LearnMore path="/learn/removing-effect-dependencies">

Pročitajte **[Uklanjanje zavisnosti Effect-a](/learn/removing-effect-dependencies)** da biste saznali kako da učinite da se vaš Effect ređe pokreće.

</LearnMore>

## Upotreba reusable logike sa prilagođenim Hook-ovima {/*reusing-logic-with-custom-hooks*/}

React dolazi sa ugrađenim Hook-ovima poput `useState`, `useContext` i `useEffect`. Ponekad ćete poželeti da postoji Hook sa konkretnijom svrhom: na primer, za fetch-ovanje podataka, za praćenje da li je korisnik online ili za konekciju na sobu za dopisivanje. Da biste ovo uradili, možete napraviti sopstvene Hook-ove za potrebe vaše aplikacije.

U ovom primeru, `usePointerPosition` prilagođeni Hook prati poziciju kursora, dok `useDelayedValue` prilagođeni Hook vraća vrednost koja "lag-uje" određeni broj milisekundi za vrednošću koju ste prosledili. Pomerajte kursor po sandbox-u da biste videli pokretni trag tačaka koji prati kursor:

<Sandpack>

```js
import { usePointerPosition } from './usePointerPosition.js';
import { useDelayedValue } from './useDelayedValue.js';

export default function Canvas() {
  const pos1 = usePointerPosition();
  const pos2 = useDelayedValue(pos1, 100);
  const pos3 = useDelayedValue(pos2, 200);
  const pos4 = useDelayedValue(pos3, 100);
  const pos5 = useDelayedValue(pos4, 50);
  return (
    <>
      <Dot position={pos1} opacity={1} />
      <Dot position={pos2} opacity={0.8} />
      <Dot position={pos3} opacity={0.6} />
      <Dot position={pos4} opacity={0.4} />
      <Dot position={pos5} opacity={0.2} />
    </>
  );
}

function Dot({ position, opacity }) {
  return (
    <div style={{
      position: 'absolute',
      backgroundColor: 'pink',
      borderRadius: '50%',
      opacity,
      transform: `translate(${position.x}px, ${position.y}px)`,
      pointerEvents: 'none',
      left: -20,
      top: -20,
      width: 40,
      height: 40,
    }} />
  );
}
```

```js src/usePointerPosition.js
import { useState, useEffect } from 'react';

export function usePointerPosition() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  useEffect(() => {
    function handleMove(e) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
  }, []);
  return position;
}
```

```js src/useDelayedValue.js
import { useState, useEffect } from 'react';

export function useDelayedValue(value, delay) {
  const [delayedValue, setDelayedValue] = useState(value);

  useEffect(() => {
    setTimeout(() => {
      setDelayedValue(value);
    }, delay);
  }, [value, delay]);

  return delayedValue;
}
```

```css
body { min-height: 300px; }
```

</Sandpack>

Možete napraviti prilagođene Hook-ove, sastaviti ih zajedno, proslediti podatke kroz njih i reuse-ovati ih između komponenata. Kako vaša aplikacija raste, pisaćete manje Effect-a ručno jer ćete moći da reuse-ujete prilagođene Hook-ove koje ste već napisali. Takođe, postoji mnogo odličnih prilagođenih Hook-ova koje održava React zajednica.

<LearnMore path="/learn/reusing-logic-with-custom-hooks">

Pročitajte **[Upotreba reusable logike sa prilagođenim Hook-ovima](/learn/reusing-logic-with-custom-hooks)** kako biste naučili da delite logiku između komponenata.

</LearnMore>

## Šta je sledeće? {/*whats-next*/}

Pređite na [Referenciranje vrednosti sa Ref-ovima](/learn/referencing-values-with-refs) da biste počeli da čitate ovo poglavlje stranicu po stranicu!
