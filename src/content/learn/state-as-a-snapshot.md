---
title: State kao snapshot
---

<Intro>

State promenljive mogu ličiti na obične JavaScript promenljive koje možete čitati i menjati. Međutim, state se ponaša više kao snapshot. Njegovo postavljanje ne menja već postojeću state promenljivu, već pokreće ponovni render.

</Intro>

<YouWillLearn>

* Kako postavljanje state-a pokreće ponovne rendere
* Kada i kako se state ažurira
* Zašto se state ne ažurira čim ga postavite
* Kako event handler-i pristupaju "snapshot-u" state-a

</YouWillLearn>

## Postavljanje state-a pokreće rendere {/*setting-state-triggers-renders*/}

Možda biste pomislili da se korisnički interfejs menja odmah nakon korisničkog event-a poput klika. U React-u, stvari funkcionišu malo drugačije od toga. Na prethodnoj stranici videli ste da [postavljanje state-a traži ponovni render](/learn/render-and-commit#step-1-trigger-a-render) od React-a. Ovo znači da trebate *ažurirati state* kako bi interfejs odreagovao na event.

U ovom primeru, kada pritisnete "Pošalji", `setIsSent(true)` kaže React-u da ponovo renderuje UI:

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState('Ćao!');
  if (isSent) {
    return <h1>Vaša poruka je na putu!</h1>
  }
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      setIsSent(true);
      sendMessage(message);
    }}>
      <textarea
        placeholder="Poruka"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Pošalji</button>
    </form>
  );
}

function sendMessage(message) {
  // ...
}
```

```css
label, textarea { margin-bottom: 10px; display: block; }
```

</Sandpack>

Evo šta se desi kada kliknete na dugme:

1. Izvrši se `onSubmit` event handler.
2. `setIsSent(true)` postavi `isSent` na `true` i postavi novi render u red čekanja.
3. React ponovo renderuje komponentu u skladu sa novom vrednošću za `isSent`.

Hajde da detaljnije pogledamo vezu između state-a i renderovanja.

## Renderovanje uzima snapshot u vremenu {/*rendering-takes-a-snapshot-in-time*/}

["Renderovanje"](/learn/render-and-commit#step-2-react-renders-your-components) znači da React poziva vašu komponentu, koja je funkcija. JSX koji se vrati iz te funkcije je kao snapshot UI-a u vremenu. Svi props-i, event handler-i i lokalne promenljive su izračunati **na osnosu state-a u trenutku rendera**.

Za razliku od fotografije ili filmskog kadra, UI "snapshot" koji vraćate je interaktivan. Uključena je logika poput event handler-a koja specificira šta će se desiti kao rezultat input-a. React ažurira prikaz da odgovara tom snapshot-u i povezuje event handler-e. Kao rezultat, pritiskom na dugme pokrenuće se handler za klik u vašem JSX-u.

Kada React ponovo renderuje komponentu:

1. React poziva vašu funkciju ponovo.
2. Funkcija vraća novi JSX snapshot.
3. React onda ažurira prikaz ekrana da odgovara snapshot-u koji je funkcija vratila.

<IllustrationBlock sequential>
    <Illustration caption="React izvršava funkciju" src="/images/docs/illustrations/i_render1.png" />
    <Illustration caption="Računanje snapshot-a" src="/images/docs/illustrations/i_render2.png" />
    <Illustration caption="Ažuriranje DOM stabla" src="/images/docs/illustrations/i_render3.png" />
</IllustrationBlock>

Kao memorija komponente, state nije kao obična promenljiva koja nestane nakon što se funkcija završi. State zapravo "živi" unutar samog React-a--kao na polici!--izvan vaše funkcije. Kada React poziva vašu komponentu, daje vam snapshot state-a za taj određeni render. Vaša komponenta vrati snapshot UI-a sa novim setom props-a i event handler-a unutar JSX-a, gde su svi izračunati **na osnosu state-a iz tog rendera**!

<IllustrationBlock sequential>
  <Illustration caption="Kažete React-u da ažurira state" src="/images/docs/illustrations/i_state-snapshot1.png" />
  <Illustration caption="React ažurira state vrednost" src="/images/docs/illustrations/i_state-snapshot2.png" />
  <Illustration caption="React prosleđuje snapshot state-a u komponentu" src="/images/docs/illustrations/i_state-snapshot3.png" />
</IllustrationBlock>

Evo malog eksperimenta da vam pokažemo kako ovo radi. U ovom primeru, mogli biste očekivati da će klik na "+3" dugme povećati brojač tri puta pošto se `setNumber(number + 1)` poziva tri puta.

Pogledajte šta će se desiti klikom na "+3" dugme:

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

Primetite da se `number` povećao jednom po kliku!

**Postavljanje state-a ga menja samo za *naredni* render.** Tokom prvog rendera, `number` je bio `0`. Zato je, u `onClick` handler-u *tog rendera*, vrednost za `number` i dalje `0` nakon što je `setNumber(number + 1)` pozvan:

```js
<button onClick={() => {
  setNumber(number + 1);
  setNumber(number + 1);
  setNumber(number + 1);
}}>+3</button>
```

Evo šta klik handler ovog dugmeta govori React-u da uradi:

1. `setNumber(number + 1)`: `number` je `0` znači `setNumber(0 + 1)`.
    - React priprema da promeni `number` na `1` u narednom renderu.
2. `setNumber(number + 1)`: `number` je `0` znači `setNumber(0 + 1)`.
    - React priprema da promeni `number` na `1` u narednom renderu.
3. `setNumber(number + 1)`: `number` je `0` znači `setNumber(0 + 1)`.
    - React priprema da promeni `number` na `1` u narednom renderu.

Iako ste pozvali `setNumber(number + 1)` tri puta, u event handler-u *ovog rendera* `number` je uvek `0`, pa ste postavili state na `1` tri puta. Zato, nakon što se event handler završi, React ponovo renderuje komponentu sa vrednošću `1` za `number`, umesto sa vrednošću `3`.

Ovo možete zamisliti i zamenom state promenljivih sa njenim vrednostima. Pošto je `number` state promenljiva jednako `0` za *ovaj render*, njen event handler izgleda ovako:

```js
<button onClick={() => {
  setNumber(0 + 1);
  setNumber(0 + 1);
  setNumber(0 + 1);
}}>+3</button>
```

Za naredni render, `number` je `1`, pa klik handler *tog rendera* izgleda ovako:

```js
<button onClick={() => {
  setNumber(1 + 1);
  setNumber(1 + 1);
  setNumber(1 + 1);
}}>+3</button>
```

Zbog toga će ponovno kliktanje na dugme postaviti brojač na `2`, pa na `3` i tako dalje.

## State tokom vremena {/*state-over-time*/}

Pa, to je bilo zabavno. Probajte pogoditi šta će prikazati ovaj alert:

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
        alert(number);
      }}>+5</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

Ako iskoristite metod zamene od ranije, možete pogoditi da alert prikazuje "0":

```js
setNumber(0 + 5);
alert(0);
```

Ali šta ako postavite tajmer za alert koji će se okinuti _nakon_ što se komponenta ponovo renderuje? Da li će biti "0" ili "5"? Pogodite!

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
        setTimeout(() => {
          alert(number);
        }, 3000);
      }}>+5</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

Iznenađeni? Ako iskoristite metod zamene, videćete da se "snapshot" state-a prosleđuje u alert.

```js
setNumber(0 + 5);
setTimeout(() => {
  alert(0);
}, 3000);
```

State sačuvan u React-u se možda promenio u trenutku izvršavanja alert-a, ali je bio zakazan koristeći snapshot state-a u trenutku kada je korisnik interagovao sa njim!

**Vrednost state promenljive se nikad ne menja unutar rendera**, čak iako je kod u event handler-u asinhron. Unutar `onClick`-a *tog rendera*, vrednost `number` nastavlja da bude `0` i nakon što je `setNumber(number + 5)` pozvano. Ta vrednost je "fiksirana" kada je React "napravio snapshot" UI-a pozivajući vašu komponentu.

Evo primera kako to smanjuje mogućnost grešaka u tajmingu u event handler-ima. Ispod je forma koja šalje poruku sa pet sekundi zakašnjenja. Zamislite ovaj scenario:

1. Pritisnete "Pošalji" i šaljete "Zdravo" osobi Alice.
2. Pre nego što istekne pet sekundi, promenite vrednost u polju "Za" na "Bob".

Šta očekujete da će `alert` prikazati? Da li će prikazati "Rekli ste Zdravo osobi Alice"? Ili će prikazati "Rekli ste Zdravo osobi Bob"? Pokušajte da pogodite na osnovu onoga što znate, a onda isprobajte:

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [to, setTo] = useState('Alice');
  const [message, setMessage] = useState('Zdravo');

  function handleSubmit(e) {
    e.preventDefault();
    setTimeout(() => {
      alert(`Rekli ste ${message} osobi ${to}`);
    }, 5000);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Za:{' '}
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
      <button type="submit">Pošalji</button>
    </form>
  );
}
```

```css
label, textarea { margin-bottom: 10px; display: block; }
```

</Sandpack>

**React zadržava state vrednosti "fiksiranim" unutar event handler-a jednog rendera.** Ne trebate brinuti da li se state promenio tokom izvršavanja koda.

Ali, šta ako želite da pročitate najnoviji state pre ponovnog rendera? Želećete da koristite [funkciju za ažuriranje state-a](/learn/queueing-a-series-of-state-updates) koju smo pokrili na narednoj stranici!

<Recap>

* Postavljanje state-a traži novi render.
* React čuva state izvan komponente, kao da je na polici.
* Kada pozovete `useState`, React vam daje snapshot state-a *za taj render*.
* Promenljive i event handler-i ne "preživljavaju" ponovne rendere. Svaki render ima svoje event handler-e.
* Svaki render (i funkcije unutar njega) će uvek "videti" snapshot state-a koji je React dao za *taj* render.
* Možete zamisliti zamenu state-a u event handler-ima slično kao što zamišljate renderovan JSX.
* Event handler-i kreirani u prošlosti imaju state vrednosti iz rendera u kom su kreirani.

</Recap>



<Challenges>

#### Implementirati semafor {/*implement-a-traffic-light*/}

Ovde je komponenta semafora za pešake koja menja svetlo kada se klikne dugme:

<Sandpack>

```js
import { useState } from 'react';

export default function TrafficLight() {
  const [walk, setWalk] = useState(true);

  function handleClick() {
    setWalk(!walk);
  }

  return (
    <>
      <button onClick={handleClick}>
        Promeni na {walk ? 'Stani' : 'Hodaj'}
      </button>
      <h1 style={{
        color: walk ? 'darkgreen' : 'darkred'
      }}>
        {walk ? 'Hodaj' : 'Stani'}
      </h1>
    </>
  );
}
```

```css
h1 { margin-top: 20px; }
```

</Sandpack>

Dodajte `alert` u klik handler. Kada je svetlo zeleno i kaže "Hodaj", klikom na dugme treba se prikazati "Naredno je Stani". Kada je svetlo crveno i kaže "Stani", klikom na dugme treba se prikazati "Naredno je Hodaj".

Ima li razlike ako postavite `alert` pre ili posle `setWalk` poziva?

<Solution>

Vaš `alert` bi trebao ovako da izgleda:

<Sandpack>

```js
import { useState } from 'react';

export default function TrafficLight() {
  const [walk, setWalk] = useState(true);

  function handleClick() {
    setWalk(!walk);
    alert(walk ? 'Naredno je Stani' : 'Naredno je Hodaj');
  }

  return (
    <>
      <button onClick={handleClick}>
        Promeni na {walk ? 'Stani' : 'Hodaj'}
      </button>
      <h1 style={{
        color: walk ? 'darkgreen' : 'darkred'
      }}>
        {walk ? 'Hodaj' : 'Stani'}
      </h1>
    </>
  );
}
```

```css
h1 { margin-top: 20px; }
```

</Sandpack>

Nema razlike ako ga postavite pre ili posle `setWalk` poziva. Vrednost `walk`-a u tom renderu je fiksirana. Pozivanje `setWalk` će je promeniti samo za *naredni* render, ali neće uticati na event handler iz prethodnog rendera.

Ova linija može delovati kontraintuitivno na prvu loptu:

```js
alert(walk ? 'Naredno je Stani' : 'Naredno je Hodaj');
```

Ali ima smisla ako je ovako čitate: "Ako semafor pokazuje 'Hodaj', poruka treba biti 'Naredno je Stani'". `walk` promenljiva unutar event handler-a odgovara vrednosti `walk`-a za taj render i ne menja se.

Možete potvrditi da je ovo tačno primenom metode zamene. Kada je `walk` jednako `true`, onda je:

```js
<button onClick={() => {
  setWalk(false);
  alert('Naredno je Stani');
}}>
  Promeni na Stani
</button>
<h1 style={{color: 'darkgreen'}}>
  Hodaj
</h1>
```

Klikom na "Promeni na Stani" stavljate render sa `walk` koji iznosi `false` u red čekanja i prikazujete "Naredno je Stani".

</Solution>

</Challenges>
