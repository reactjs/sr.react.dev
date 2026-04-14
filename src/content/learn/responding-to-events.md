---
title: Odgovaranje na event-e
---

<Intro>

React vam omogućava da dodate *event handler*-e u vaš JSX. Event handler-i su vaše sopstvene funkcije koje će se pokrenuti kao odgovor na korisničke interakcije poput klika, prelaženja mišem, fokusiranja na input-e forme i tako dalje.

</Intro>

<YouWillLearn>

* Različite načine da napišete event handler
* Kako proslediti logiku za obrađivanje event-ova iz roditeljske komponente
* Kako se event-i propagiraju i kako ih zaustaviti

</YouWillLearn>

## Dodavanje event handler-a {/*adding-event-handlers*/}

Da biste dodali event handler, prvo ćete definisati funkciju, a onda je [proslediti kao prop](/learn/passing-props-to-a-component) u odgovarajući JSX tag. Na primer, ovde je dugme koje još uvek ništa ne radi:

<Sandpack>

```js
export default function Button() {
  return (
    <button>
      Ne radim ništa
    </button>
  );
}
```

</Sandpack>

Možete učiniti da se prikaže poruka kad ga korisnik klikne prateći ova tri koraka:

1. Deklarišite funkciju pod imenom `handleClick` *unutar* `Button` komponente.
2. Implementirajte logiku unutar te funkcije (koristite `alert` da prikažete poruku).
3. Dodajte `onClick={handleClick}` u `<button>` JSX.

<Sandpack>

```js
export default function Button() {
  function handleClick() {
    alert('Kliknuli ste me!');
  }

  return (
    <button onClick={handleClick}>
      Klikni me
    </button>
  );
}
```

```css
button { margin-right: 10px; }
```

</Sandpack>

Definisali ste `handleClick` funkciju i [prosledili ste je kao prop](/learn/passing-props-to-a-component) u `<button>`. `handleClick` je **event handler**. Event handler funkcije:

* Su uglavnom definisane *unutar* vaših komponenata.
* Imaju imena koja počinju sa `handle`, nakon čega sledi ime event-a.

Po konvenciji, praksa je da imena event handler-a počinju sa `handle`, nakon čega sledi ime event-a. Često ćete videti `onClick={handleClick}`, `onMouseEnter={handleMouseEnter}` i slično.

Alternativno, možete definisati event handler inline u JSX-u:

```jsx
<button onClick={function handleClick() {
  alert('Kliknuli ste me!');
}}>
```

Ili konciznije, upotrebom arrow funkcije:

```jsx
<button onClick={() => {
  alert('Kliknuli ste me!');
}}>
```

Svi ovi načini su ekvivalentni. Inline event handler-i su zgodni za kratke funkcije.

<Pitfall>

Funkcije prosleđene u event handler-e moraju biti prosleđene, a ne pozvane. Na primer:

| prosleđivanje funkcije (ispravno) | pozivanje funkcije (neispravno)    |
| --------------------------------  | ---------------------------------- |
| `<button onClick={handleClick}>`  | `<button onClick={handleClick()}>` |

Razlika je veoma suptilna. U prvom primeru, `handleClick` funkcija je prosleđena kao `onClick` event handler. To govori React-u da je upamti i pozove samo kada korisnik klikne dugme.

U drugom primeru, `()` na kraju `handleClick()` poziva funkciju *istog momenta* tokom [renderovanja](/learn/render-and-commit), bez bilo kakvih klikova. Tako je zbog toga što JavaScript sve u [JSX-u unutar `{` i `}`](/learn/javascript-in-jsx-with-curly-braces) izvršava odmah.

Kada napišete kod inline, ista zamka će se prikazati, samo na drugi način:

| prosleđivanje funkcije (ispravno)       | pozivanje funkcije (neispravno)   |
| --------------------------------------- | --------------------------------- |
| `<button onClick={() => alert('...')}>` | `<button onClick={alert('...')}>` |


Prosleđivanje inline koda na ovaj način neće okinuti funkciju na klik—okinuće je svaki put kad se komponenta renderuje:

```jsx
// Ovaj alert se okida kad se komponenta renderuje, ne kad se klikne!
<button onClick={alert('Kliknuli ste me!')}>
```

Ako želite definisati event handler inline, obmotajte ga u anonimnu funkciju poput ove:

```jsx
<button onClick={() => alert('Kliknuli ste me!')}>
```

Umesto da se kod izvršava na svaki render, ovo kreira funkciju koja će biti pozvana kasnije.

U oba slučaja, želećete da prosledite funkciju:

* `<button onClick={handleClick}>` prosleđuje `handleClick` funkciju.
* `<button onClick={() => alert('...')}>` prosleđuje `() => alert('...')` funkciju.

[Pročitajte više o arrow funkcijama.](https://javascript.info/arrow-functions-basics)

</Pitfall>

### Čitanje props-a u event handler-ima {/*reading-props-in-event-handlers*/}

Pošto su event handler-i deklarisani unutar komponente, imaju pristup props-ima komponente. Ovde je dugme koje, kad je kliknuto, prikazuje alert sa `message` prop-om:

<Sandpack>

```js
function AlertButton({ message, children }) {
  return (
    <button onClick={() => alert(message)}>
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <AlertButton message="Puštanje!">
        Pusti film
      </AlertButton>
      <AlertButton message="Upload-ovanje!">
        Upload-uj sliku
      </AlertButton>
    </div>
  );
}
```

```css
button { margin-right: 10px; }
```

</Sandpack>

Ovo omogućava tim dugmićima da prikažu različite poruke. Probajte da promenite poruke koje su im prosleđene.

### Prosleđivanje event handler-a kao prop-a {/*passing-event-handlers-as-props*/}

Često ćete želeti da roditeljska komponenta specificira dečji event handler. Razmotrite dugmiće: u zavisnosti od toga gde koristite `Button` komponentu, možete poželeti da izvršite različite funkcije—jedna da pušta film, druga da upload-uje sliku.

Da biste to uradili, prosledite prop koji komponenta primi od roditelja kao event handler:

<Sandpack>

```js
function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}

function PlayButton({ movieName }) {
  function handlePlayClick() {
    alert(`Puštanje ${movieName}!`);
  }

  return (
    <Button onClick={handlePlayClick}>
      Pusti "{movieName}"
    </Button>
  );
}

function UploadButton() {
  return (
    <Button onClick={() => alert('Upload-ovanje!')}>
      Upload-uj sliku
    </Button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <PlayButton movieName="Kikina služba za dostavu" />
      <UploadButton />
    </div>
  );
}
```

```css
button { margin-right: 10px; }
```

</Sandpack>

Ovde, `Toolbar` komponenta renderuje `PlayButton` i `UploadButton`:

- `PlayButton` prosleđuje `handlePlayClick` kao `onClick` prop u `Button` komponentu.
- `UploadButton` prosleđuje `() => alert('Upload-ovanje!')` kao `onClick` prop u `Button` komponentu.

Konačno, vaša `Button` komponenta prima prop po imenu `onClick`. Taj prop direktno prosleđuje u ugrađeni `<button>` tag sa `onClick={onClick}`. Ovo govori React-u da pozove prosleđenu funkciju na klik.

Ako koristite [sistem dizajna](https://uxdesign.cc/everything-you-need-to-know-about-design-systems-54b109851969), uobičajeno je da komponente poput dugmića sadrže stajling, ali da ne specificiraju ponašanje. Umesto toga, komponente poput `PlayButton` i `UploadButton` će im proslediti event handler-e.

### Imenovanje event handler props-a {/*naming-event-handler-props*/}

Ugrađene komponente kao što su `<button>` i `<div>` podržavaju samo [imena event-ova u pretraživaču](/reference/react-dom/components/common#common-props) poput `onClick`. Međutim, kada pravite sopstvene komponente, vaše event handler props-e možete imenovati kako god želite.

Po konvenciji, event handler props-i trebaju započeti sa `on`, nakon čega sledi veliko slovo.

Na primer, `onClick` prop u `Button` komponenti može biti nazvan `onSmash`:

<Sandpack>

```js
function Button({ onSmash, children }) {
  return (
    <button onClick={onSmash}>
      {children}
    </button>
  );
}

export default function App() {
  return (
    <div>
      <Button onSmash={() => alert('Puštanje!')}>
        Pusti film
      </Button>
      <Button onSmash={() => alert('Upload-ovanje!')}>
        Upload-uj sliku
      </Button>
    </div>
  );
}
```

```css
button { margin-right: 10px; }
```

</Sandpack>

U ovom primeru, `<button onClick={onSmash}>` prikazuje da ugrađeni `<button>` (malim slovima) i dalje zahteva prop sa imenom `onClick`, ali ime prop-a koji prima vaša custom `Button` komponenta je na vama!

Kada komponenta podržava više interakcija, možete imenovati event handler props-e na osnovu koncepata specifičnih za aplikaciju. Na primer, ova `Toolbar` komponenta prima `onPlayMovie` i `onUploadImage` event handler-e:

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

Primetite da `App` komponenta ne mora da zna *šta* će `Toolbar` uraditi sa `onPlayMovie` i `onUploadImage`. To je implementacijski detalj u `Toolbar`-u. Takođe, `Toolbar` ih prosleđuje kao `onClick` handler-e u svoje `Button`-e, ali ih kasnije može okinuti i za prečicu na tastaturi. Imenovanje event handler props-a na osnovu koncepata specifičnih za aplikaciju poput `onPlayMovie` vam daje fleksibilnost da kasnije menjate kako se koriste.

<Note>

Postarajte se da koristite odgovarajuće HTML tag-ove za event handler-e. Na primer, za klikove, koristite [`<button onClick={handleClick}>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button) umesto `<div onClick={handleClick}>`. Korišćenjem `<button>`-a u pretraživaču omogućavate ugrađena ponašanja pretraživača poput navigacije pomoću tastature. Ako vam se ne sviđa default stajling dugmeta u pretraživaču i želite da liči na link ili drugi UI element, to možete učititi pomoću CSS-a. [Naučite više o pisanju pristupačnog markup-a](https://developer.mozilla.org/en-US/docs/Learn/Accessibility/HTML).

</Note>

## Propagacija event-ova {/*event-propagation*/}

Event handler-i će takođe uhvatiti event-e iz svake dečje komponente. Kažemo da se event "ponaša kao mehurić" ili "propagira" uz stablo: počinje tamo gde se event desio, a onda kreće uz stablo.

Ovaj `<div>` sadrži dva dugmeta. I `<div>`, *ali i* svako dugme, imaju svoj `onClick` handler. Šta mislite koji handler-i će se okinuti kada kliknete na dugme?

<Sandpack>

```js
export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('Kliknuli ste na toolbar!');
    }}>
      <button onClick={() => alert('Puštanje!')}>
        Pusti film
      </button>
      <button onClick={() => alert('Upload-ovanje!')}>
        Upload-uj sliku
      </button>
    </div>
  );
}
```

```css
.Toolbar {
  background: #aaa;
  padding: 5px;
}
button { margin: 5px; }
```

</Sandpack>

Ako kliknete na neko dugme, prvo će se izvršiti njegov `onClick`, a nakon toga i `onClick` od roditeljskog `<div>`-a. Znači, dve poruke će se prikazati. Ako kliknete samo na toolbar, samo će se izvršiti `onClick` na `<div>`-u.

<Pitfall>

U React-u, svi event-i se propagiraju osim `onScroll`, koji radi samo za JSX tag na kom je definisan.

</Pitfall>

### Zaustavljanje propagacije {/*stopping-propagation*/}

Event handler-i primaju **event objekat** kao jedini argument. Po konvenciji, često se naziva `e`, što označava "event". Možete koristiti ovaj objekat da čitate informacije o event-u.

Ovaj event objekat vam takođe omogućava da zaustavite propagaciju. Ako želite da sprečite event da dospe do roditeljskih komponenata, morate pozvati `e.stopPropagation()` kao što to radi `Button` komponenta:

<Sandpack>

```js
function Button({ onClick, children }) {
  return (
    <button onClick={e => {
      e.stopPropagation();
      onClick();
    }}>
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('Kliknuli ste na toolbar!');
    }}>
      <Button onClick={() => alert('Puštanje!')}>
        Pusti film
      </Button>
      <Button onClick={() => alert('Upload-ovanje!')}>
        Upload-uj sliku
      </Button>
    </div>
  );
}
```

```css
.Toolbar {
  background: #aaa;
  padding: 5px;
}
button { margin: 5px; }
```

</Sandpack>

Kada kliknete na dugme:

1. React poziva `onClick` handler prosleđen u `<button>`.
2. Taj handler, definisan u `Button`-u, radi sledeće:
   * Poziva `e.stopPropagation()`, sprečavajući da se event propagira dalje.
   * Poziva `onClick` funkciju, koja je prop prosleđen iz `Toolbar` komponente.
3. Ta funkcija, definisana u `Toolbar` komponenti, prikazuje poruku specifičnu za dugme.
4. Pošto je propagacija zaustavljena, `onClick` handler u roditeljskom `<div>`-u se *ne* pokreće.

Kao rezultat od `e.stopPropagation()`, klik na dugmiće prikazuje samo jednu poruku (iz `<button>`-a) umesto dve (iz `<button>`-a i iz roditeljskog toolbar `<div>`-a). Kliktanje dugmeta nije isto kao kliktanje okružujućeg toolbar-a, tako da zaustavljanje propagacije ima smisla za ovaj UI.

<DeepDive>

#### Hvatanje (capture) phase event-ova {/*capture-phase-events*/}

U retkim slučajevima, možete poželeti da uhvatite sve event-e iz dečjih elemenata, *čak iako su zaustavili propagaciju*. Na primer, želite da logujete svaki klik za analitiku, nevezano sa logikom propagacije. To možete uraditi dodavanjem `Capture` na kraju imena event-a:

```js
<div onClickCapture={() => { /* ovo se izvršava prvo */ }}>
  <button onClick={e => e.stopPropagation()} />
  <button onClick={e => e.stopPropagation()} />
</div>
```

Svaki event se propagira u tri faze:

1. Putuje na dole, pozivajući sve `onClickCapture` handler-e.
2. Pokreće `onClick` handler na kliknutom elementu. 
3. Putuje na gore, pozivajući sve `onClick` handler-e.

Capture event-i su korisni za rutere i analitiku, ali ih verovatno nećete koristiti u kodu aplikacije.

</DeepDive>

### Prosleđivanje handler-a kao alternative za propagaciju {/*passing-handlers-as-alternative-to-propagation*/}

Primetite da ovaj klik handler pokreće liniju koda, _a onda_ poziva `onClick` prop prosleđen iz roditelja:

```js {4,5}
function Button({ onClick, children }) {
  return (
    <button onClick={e => {
      e.stopPropagation();
      onClick();
    }}>
      {children}
    </button>
  );
}
```

Takođe, možete dodati još koda u ovaj handler pre poziva roditeljskog `onClick` event handler-a. Ovaj šablon pruža *alternativu* propagaciji. Omogućava dečjoj komponenti da obradi event, dopuštajući roditeljskoj komponenti da specificira i neko dodatno ponašanje. Za razliku od propagacije, ovo nije automatsko. Međutim, benefit ovog šablona je da jasno možete ispratiti sav kod koji se izvršava kao rezultat nekog event-a.

Ako se oslanjate na propagaciju i teško vam je da ispratite koji handler-i se izvršavaju i zašto, probajte ovaj pristup.

### Sprečavanje default ponašanja {/*preventing-default-behavior*/}

Neki event-i u pretraživačima imaju default ponašanje. Na primer, `<form>` submit event, koji se okida kada se klikne dugme unutar forme, ponovo će učitati celu stranicu po default-u:

<Sandpack>

```js
export default function Signup() {
  return (
    <form onSubmit={() => alert('Submit-ovanje!')}>
      <input />
      <button>Pošalji</button>
    </form>
  );
}
```

```css
button { margin-left: 5px; }
```

</Sandpack>

Možete pozvati `e.preventDefault()` nad event objektom kako biste zaustavili ovo:

<Sandpack>

```js
export default function Signup() {
  return (
    <form onSubmit={e => {
      e.preventDefault();
      alert('Submit-ovanje!');
    }}>
      <input />
      <button>Pošalji</button>
    </form>
  );
}
```

```css
button { margin-left: 5px; }
```

</Sandpack>

Nemojte pomešati `e.stopPropagation()` i `e.preventDefault()`. Korisni su, ali nisu povezani:

* [`e.stopPropagation()`](https://developer.mozilla.org/docs/Web/API/Event/stopPropagation) zaustavlja okidanje event handler-a povezanih sa tag-ovima iznad u hijerarhiji.
* [`e.preventDefault()`](https://developer.mozilla.org/docs/Web/API/Event/preventDefault) sprečava default ponašanje pretraživača za par event-ova koji ga imaju.

## Da li event handler-i smeju imati propratne efekte? {/*can-event-handlers-have-side-effects*/}

Apsolutno! Event handler-i su najbolje mesto za propratne efekte.

Za razliku od funkcija za renderovanje, event handler-i ne moraju biti [čisti](/learn/keeping-components-pure), tako da su odlično mesto za *promenu* nečega—na primer, promena input vrednosti kao rezultat kucanja, ili promena liste kao odgovor na klik dugmeta. Međutim, da biste izmenili neku informaciju, prvo vam je potreban način da je čuvate. U React-u, to se dešava kroz [state, memoriju komponente](/learn/state-a-components-memory). Naučićete sve o tome na narednoj stranici.

<Recap>

* Možete obrađivati event-e prosleđivanjem funkcije kao prop-a u elemente poput `<button>`-a.
* Event handler-i se moraju proslediti, **a ne pozvati**! `onClick={handleClick}`, ne `onClick={handleClick()}`.
* Možete definisati event handler funkciju zasebno ili inline.
* Event handler-i su definisani unutar komponente kako bi mogli pristupiti props-ima.
* Možete deklarisati event handler u roditelju i proslediti ga kao prop detetu.
* Možete definisati sopstvene event handler props-e sa imenima na osnovu koncepata specifičnih za aplikaciju.
* Event-i se propagiraju nagore. Pozovite `e.stopPropagation()` nad prvim argumentom kako biste to sprečili.
* Event-i mogu imati neželjeno default ponašanje u pretraživaču. Pozovite `e.preventDefault()` da to sprečite.
* Eksplicitno pozivanje event handler prop-a iz dečjeg handler-a je dobra alternativa za propagaciju.

</Recap>



<Challenges>

#### Popraviti event handler {/*fix-an-event-handler*/}

Klikom na dugme bi se trebala promeniti pozadina stranice između bele i crne. Međutim, kad kliknete ništa se ne dešava. Popravite problem. (Ne brinite za logiku unutar `handleClick`—taj deo je u redu.)

<Sandpack>

```js {expectedErrors: {'react-compiler': [5, 7]}}
export default function LightSwitch() {
  function handleClick() {
    let bodyStyle = document.body.style;
    if (bodyStyle.backgroundColor === 'black') {
      bodyStyle.backgroundColor = 'white';
    } else {
      bodyStyle.backgroundColor = 'black';
    }
  }

  return (
    <button onClick={handleClick()}>
      Promeni pozadinu
    </button>
  );
}
```

</Sandpack>

<Solution>

Problem je u tome što `<button onClick={handleClick()}>` _poziva_ `handleClick` funkciju tokom renderovanja umesto da je _prosleđuje_. Uklanjanje `()` iz poziva `<button onClick={handleClick}>` rešava problem:

<Sandpack>

```js
export default function LightSwitch() {
  function handleClick() {
    let bodyStyle = document.body.style;
    if (bodyStyle.backgroundColor === 'black') {
      bodyStyle.backgroundColor = 'white';
    } else {
      bodyStyle.backgroundColor = 'black';
    }
  }

  return (
    <button onClick={handleClick}>
      Promeni pozadinu
    </button>
  );
}
```

</Sandpack>

Alternativno, možete obmotati poziv u drugu funkciju, na primer `<button onClick={() => handleClick()}>`:

<Sandpack>

```js
export default function LightSwitch() {
  function handleClick() {
    let bodyStyle = document.body.style;
    if (bodyStyle.backgroundColor === 'black') {
      bodyStyle.backgroundColor = 'white';
    } else {
      bodyStyle.backgroundColor = 'black';
    }
  }

  return (
    <button onClick={() => handleClick()}>
      Promeni pozadinu
    </button>
  );
}
```

</Sandpack>

</Solution>

#### Povezati event-e {/*wire-up-the-events*/}

Ova `ColorSwitch` komponenta renderuje dugme. Trebala bi da promeni boju na stranici. Povežite je sa `onChangeColor` event handler prop-om koji dobija od roditelja, kako biste klikom na dugme menjali boju.

Nakon što to uradite, primetite da klikom na dugme povećavate i brojač klikova na stranici. Vaš kolega koji je napisao roditeljsku komponentu insistira da `onChangeColor` ne povećava nijedan brojač. Šta se još može dogoditi? Popravite ovo tako da klik na dugme *jedino* menja boju, a _ne_ povećava brojač.

<Sandpack>

```js src/ColorSwitch.js active
export default function ColorSwitch({
  onChangeColor
}) {
  return (
    <button>
      Promeni boju
    </button>
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import ColorSwitch from './ColorSwitch.js';

export default function App() {
  const [clicks, setClicks] = useState(0);

  function handleClickOutside() {
    setClicks(c => c + 1);
  }

  function getRandomLightColor() {
    let r = 150 + Math.round(100 * Math.random());
    let g = 150 + Math.round(100 * Math.random());
    let b = 150 + Math.round(100 * Math.random());
    return `rgb(${r}, ${g}, ${b})`;
  }

  function handleChangeColor() {
    let bodyStyle = document.body.style;
    bodyStyle.backgroundColor = getRandomLightColor();
  }

  return (
    <div style={{ width: '100%', height: '100%' }} onClick={handleClickOutside}>
      <ColorSwitch onChangeColor={handleChangeColor} />
      <br />
      <br />
      <h2>Klikova na stranici: {clicks}</h2>
    </div>
  );
}
```

</Sandpack>

<Solution>

Prvo, morate dodati event handler, `<button onClick={onChangeColor}>`.

Međutim, to uvodi problem povećavanja brojača. Ako `onChangeColor` to ne radi, kao što vaš kolega insistira, problem je u tome što se event propagira nagore, a neki handler iznad to radi. Da biste rešili problem morate zaustaviti propagaciju. Ali, ne zaboravite da i dalje trebate pozvati `onChangeColor`.

<Sandpack>

```js src/ColorSwitch.js active
export default function ColorSwitch({
  onChangeColor
}) {
  return (
    <button onClick={e => {
      e.stopPropagation();
      onChangeColor();
    }}>
      Promeni boju
    </button>
  );
}
```

```js src/App.js hidden
import { useState } from 'react';
import ColorSwitch from './ColorSwitch.js';

export default function App() {
  const [clicks, setClicks] = useState(0);

  function handleClickOutside() {
    setClicks(c => c + 1);
  }

  function getRandomLightColor() {
    let r = 150 + Math.round(100 * Math.random());
    let g = 150 + Math.round(100 * Math.random());
    let b = 150 + Math.round(100 * Math.random());
    return `rgb(${r}, ${g}, ${b})`;
  }

  function handleChangeColor() {
    let bodyStyle = document.body.style;
    bodyStyle.backgroundColor = getRandomLightColor();
  }

  return (
    <div style={{ width: '100%', height: '100%' }} onClick={handleClickOutside}>
      <ColorSwitch onChangeColor={handleChangeColor} />
      <br />
      <br />
      <h2>Klikova na stranici: {clicks}</h2>
    </div>
  );
}
```

</Sandpack>

</Solution>

</Challenges>
