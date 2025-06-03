---
title: Čuvanje i resetovanje state-a
---

<Intro>

State je izolovan između komponenata. React prati koji state pripada kojoj komponenti na osnovu njenog mesta u UI stablu. Možete kontrolisati kada čuvati state, a kada ga resetovati između ponovnih rendera.

</Intro>

<YouWillLearn>

* Kada React bira da li da čuva ili resetuje state
* Kako naterati React da resetuje state komponente
* Kako ključevi i tipovi utiču na to da li je state sačuvan

</YouWillLearn>

## State je povezan sa pozicijom u stablu renderovanja {/*state-is-tied-to-a-position-in-the-tree*/}

React pravi [stabla renderovanja](learn/understanding-your-ui-as-a-tree#the-render-tree) za strukturu komponenti na vašem UI-u.

Kada komponenti date state, možete pomisliti da state "živi" unutar komponente. Ali, state se zapravo čuva unutar React-a. React povezuje svaki deo state-a sa tačnom komponentom na osnovu mesta na kom se ta komponenta nalazi u stablu renderovanja.

Ovde postoji samo jedan `<Counter />` JSX tag, ali je renderovan na dve različite pozicije:

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const counter = <Counter />;
  return (
    <div>
      {counter}
      {counter}
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Dodaj jedan
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Evo kako to izgleda u obliku stabla:

<DiagramGroup>

<Diagram name="preserving_state_tree" height={248} width={395} alt="Dijagram stabla React komponenata. Korenski čvor nazvan 'div' ima dva deteta. Svako dete se naziva 'Counter' i sadrži state balon sa nazivom 'count' i vrednošću 0.">

React stablo

</Diagram>

</DiagramGroup>

**Ovo su dva odvojena brojača jer je svaki renderovan na svojoj poziciji u stablu.** Obično ne morate razmišljati o ovim pozicijama da biste koristili React, ali može biti korisno da razumete kako to funkcioniše.

U React-u, svaka komponenta na ekranu ima potpuno izolovan state. Na primer, ako renderujete dve `Counter` komponente jednu pored druge, svaka će imati svoje nezavisne `score` i `hover` state-ove.

Probajte da kliknete oba brojača i primetite da ne utiču jedan na drugog:

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  return (
    <div>
      <Counter />
      <Counter />
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Dodaj jedan
      </button>
    </div>
  );
}
```

```css
.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Kao što možete videti, kada je jedan brojač ažuriran, jedino se state za tu komponentu ažurira:


<DiagramGroup>

<Diagram name="preserving_state_increment" height={248} width={441} alt="Dijagram stabla React komponenata. Korenski čvor nazvan 'div' ima dva deteta. Levo dete se naziva 'Counter' i sadrži state balon sa nazivom 'count' i vrednošću 0. Desno dete se naziva 'Counter' i sadrži state balon sa nazivom 'count' i vrednošću 1. State balon desnog deteta je istaknut žutom bojom da bi se naznačilo da mu je vrednost ažurirana.">

Ažuriranje state-a

</Diagram>

</DiagramGroup>


React će držati state dok god istu komponentu renderujete na istoj poziciji u stablu. Da biste ovo primetili, inkrementirajte oba brojača, uklonite drugu komponentu tako što ćete odštiklirati checkbox "Renderuj drugi brojač", a onda je štikliranjem dodajte ponovo:

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [showB, setShowB] = useState(true);
  return (
    <div>
      <Counter />
      {showB && <Counter />} 
      <label>
        <input
          type="checkbox"
          checked={showB}
          onChange={e => {
            setShowB(e.target.checked)
          }}
        />
        Renderuj drugi brojač
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Dodaj jedan
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Primetite da u trenutku kada prestanete da renderujete drugi brojač, njegov state potpuno nestaje. To je zato što kad React ukloni komponentu, uklanja i njen state.

<DiagramGroup>

<Diagram name="preserving_state_remove_component" height={253} width={422} alt="Dijagram stabla React komponenata. Korenski čvor nazvan 'div' ima dva deteta. Levo dete se naziva 'Counter' i sadrži state balon sa nazivom 'count' i vrednošcu 0. Desno dete nedostaje i na njegovom mestu je žuta 'poof' slika, ističući da se komponenta briše iz stabla.">

Brisanje komponente

</Diagram>

</DiagramGroup>

Kada štiklirate "Renderuj drugi brojač", drugi `Counter` i njegov state se inicijalizuju od nule (`score = 0`) i dodaju u DOM.

<DiagramGroup>

<Diagram name="preserving_state_add_component" height={258} width={500} alt="Dijagram stabla React komponenata. Korenski čvor nazvan 'div' ima dva deteta. Levo dete se naziva 'Counter' i sadrži state balon sa nazivom 'count' i vrednošcu 0. Desno dete se naziva 'Counter' i sadrži state balon sa nazivom 'count' i vrednošcu 0. Celokupan čvor desnog deteta je istaknut žutom bojom, označavajući da je upravo dodat u stablo.">

Dodavanje komponente

</Diagram>

</DiagramGroup>

**React čuva state komponente dok god se renderuje na svojoj poziciji u UI stablu.** Ako se ukloni, ili se druga komponenta renderuje na istoj poziciji, React odbacuje njen state.

## Ista komponenta na istoj poziciji čuva state {/*same-component-at-the-same-position-preserves-state*/}

U ovom primeru postoje dva različita `<Counter />` tag-a:

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <Counter isFancy={true} /> 
      ) : (
        <Counter isFancy={false} /> 
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Koristi fensi stajling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Dodaj jedan
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.fancy {
  border: 5px solid gold;
  color: #ff6767;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Kada promenite checkbox, counter state ne bude resetovan. Nezavisno od toga da li je `isFancy` jednak `true` ili `false`, uvek ćete imati `<Counter />` kao prvo dete `div`-a koje je vraćeno iz korenske `App` komponente:

<DiagramGroup>

<Diagram name="preserving_state_same_component" height={461} width={600} alt="Dijagram sa dva dela razdvojena strelicom koja prelazi od jednog ka drugom. Svaki deo sadrži raspored komponenata sa roditeljem pod imenom 'App' koji sadrži state balon nazvan isFancy. Ova komponenta ima jedno dete nazvano 'div' koje vodi do prop balona koji sadrži isFancy (istaknuto ljubičastom bojom) i prosleđuje se jedinom detetu. Poslednje dete pod imenom 'Counter' sadrži state balon sa nazivom 'count' i vrednošcu 3 u oba dijagrama. U levom delu dijagrama ništa nije istaknuto i vrednost roditeljskog isFancy state-a je false. U desnom delu dijagrama vrednost roditeljskog isFancy state-a se promenila na true i istaknuta je žutom bojom, kao i prop balon ispod, čija isFancy vrednost se takođe promenila na true.">

Ažuriranje `App` state-a ne resetuje `Counter` zato što `Counter` ostaje na istoj poziciji

</Diagram>

</DiagramGroup>


To je ista komponenta na istoj poziciji, pa je iz React perspektive to isti brojač.

<Pitfall>

Zapamtite da je **pozicija u UI stablu--ne u JSX markup-u--ono što je React-u bitno**! Ova komponenta ima dva `return` iskaza sa različitim `<Counter />` JSX tag-ovima unutar i izvan `if`-a:

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  if (isFancy) {
    return (
      <div>
        <Counter isFancy={true} />
        <label>
          <input
            type="checkbox"
            checked={isFancy}
            onChange={e => {
              setIsFancy(e.target.checked)
            }}
          />
          Koristi fensi stajling
        </label>
      </div>
    );
  }
  return (
    <div>
      <Counter isFancy={false} />
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Koristi fensi stajling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Dodaj jedan
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.fancy {
  border: 5px solid gold;
  color: #ff6767;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Možda biste očekivali da se state resetuje kada promenite checkbox, ali to nije slučaj! To se dešava jer su **oba `<Counter />` tag-a renderovana na istoj poziciji**. React ne zna gde vi pravite uslove u vašoj funkciji. Sve što "vidi" je stablo koje vratite.

U oba slučaja, `App` komponenta vraća `<div>` sa `<Counter />` kao prvim detetom. Za React, ova dva brojača imaju iste "adrese": prvo dete prvog deteta od korena. Ovako ih React poredi između prethodnih i narednih rendera, nezavisno od strukture vaše logike.

</Pitfall>

## Različite komponente na istoj poziciji resetuju state {/*different-components-at-the-same-position-reset-state*/}

U ovom primeru, štikliranje checkbox-a će zameniti `<Counter>` sa `<p>`:

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [isPaused, setIsPaused] = useState(false);
  return (
    <div>
      {isPaused ? (
        <p>Vidimo se posle!</p> 
      ) : (
        <Counter /> 
      )}
      <label>
        <input
          type="checkbox"
          checked={isPaused}
          onChange={e => {
            setIsPaused(e.target.checked)
          }}
        />
        Uzmi pauzu
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Dodaj jedan
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Ovde menjate između _različitih_ tipova komponenti na istoj poziciji. Inicijalno, prvo dete `<div>`-a je bio `Counter`. Ali kad ste ga zamenili sa `p`, React je uklonio `Counter` iz UI stabla i uništio njegov state.

<DiagramGroup>

<Diagram name="preserving_state_diff_pt1" height={290} width={753} alt="Dijagram sa tri dela i strelicom koja prelazi između njih. Prvi deo sadrži React komponentu nazvanu 'div' sa jednim detetom nazvanim 'Counter' koji sadrži state balon sa nazivom 'count' i vrednošću 3. Srednji deo ima istog 'div' roditelja, ali je dečja komponenta sada obrisana, što je označeno žutom 'poof' slikom. Treći deo ponovo ima istog 'div' roditelja, ali sada sa novim detetom nazvanim 'p', koji je istaknut žutom.">

Kada se `Counter` zameni sa `p`, `Counter` je obrisan i `p` je dodat

</Diagram>

</DiagramGroup>

<DiagramGroup>

<Diagram name="preserving_state_diff_pt2" height={290} width={753} alt="Dijagram sa tri dela i strelicom koja prelazi između njih. Prvi deo sadrži React komponentu nazvanu 'p'. Srednji deo ima istog 'div' roditelja, ali je dečja komponenta sada obrisana, što je označeno žutom 'poof' slikom. Treći deo ponovo ima istog 'div' roditelja, ali sada sa novim detetom nazvanim 'Counter' koji sadrži state balon sa nazivom 'count' i vrednošću 0 i istaknut je žutom.">

Kada se ponovo promene, `p` je obrisan, a `Counter` je dodat

</Diagram>

</DiagramGroup>

Takođe, **kada renderujete različitu komponentu na istoj poziciji, resetuje se state od čitavog podstabla**. Da vidite kako ovo radi, inkrementirajte brojač i onda štiklirajte checkbox:

<Sandpack>

```js
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <div>
          <Counter isFancy={true} /> 
        </div>
      ) : (
        <section>
          <Counter isFancy={false} />
        </section>
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Koristi fensi stajling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Dodaj jedan
      </button>
    </div>
  );
}
```

```css
label {
  display: block;
  clear: both;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
  float: left;
}

.fancy {
  border: 5px solid gold;
  color: #ff6767;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

State `Counter`-a se resetuje kada kliknete na checkbox. Iako renderujete `Counter`, prvo dete `div`-a se promeni sa `section` na `div`. Kada je dečji `section` uklonjen iz DOM-a, celokupno stablo ispod (uključujući `Counter` i njegov state) je takođe uništeno.

<DiagramGroup>

<Diagram name="preserving_state_diff_same_pt1" height={350} width={794} alt="Dijagram sa tri dela i strelicom koja prelazi između njih. Prvi deo sadrži React komponentu nazvanu 'div' sa jednim detetom nazvanim 'section', koji ima jedno dete sa nazivom 'Counter' i sadrži state balon sa nazivom 'count' i vrednošću 3. Srednji deo ima istog 'div' roditelja, ali su dečje komponente sada obrisane, što je označeno žutom 'poof' slikom. Treći deo ponovo ima istog 'div' roditelja, ali sada sa novim detetom nazvanim 'div' i istaknutog žutom, takođe sa novim detetom nazvanim 'Counter' koji sadrži state balon sa nazivom 'count' i vrednošću 0, sve istaknuto žutom.">

Kada se `section` promeni u `div`, `section` je obrisan, a novi `div` je dodat

</Diagram>

</DiagramGroup>

<DiagramGroup>

<Diagram name="preserving_state_diff_same_pt2" height={350} width={794} alt="Dijagram sa tri dela i strelicom koja prelazi između njih. Prvi deo sadrži React komponentu nazvanu 'div' sa jednim detetom nazvanim 'div', koji ima jedno dete sa nazivom 'Counter' i sadrži state balon sa nazivom 'count' i vrednošću 0. Srednji deo ima istog 'div' roditelja, ali su dečje komponente sada obrisane, što je označeno žutom 'poof' slikom. Treći deo ponovo ima istog 'div' roditelja, ali sada sa novim detetom nazvanim 'section' i istaknutog žutom, takođe sa novim detetom nazvanim 'Counter' koji sadrži state balon sa nazivom 'count' i vrednošću 0, sve istaknuto žutom.">

Kada se ponovo promene, `div` je obrisan, a novi `section` je dodat

</Diagram>

</DiagramGroup>

Kao pravilo, **ako želite da sačuvate state između ponovnih rendera, struktura vašeg stabla mora da se "poklapa"** od jednog do drugog rendera. Ako je struktura različita, state će biti uništen jer React uništava state kada uklanja komponentu iz stabla.

<Pitfall>

Zbog ovoga ne biste trebali da ugnježdavate definicije funkcija komponenti.

Ovde je funkcija `MyTextField` komponente definisana *unutar* `MyComponent`:

<Sandpack>

```js
import { useState } from 'react';

export default function MyComponent() {
  const [counter, setCounter] = useState(0);

  function MyTextField() {
    const [text, setText] = useState('');

    return (
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
    );
  }

  return (
    <>
      <MyTextField />
      <button onClick={() => {
        setCounter(counter + 1)
      }}>Kliknuto {counter} puta</button>
    </>
  );
}
```

</Sandpack>


Svaki put kad kliknete dugme, input state nestane! To se dešava jer je *različita* `MyTextField` funkcija kreirana za svaki render `MyComponent`-a. Renderujete *različitu* komponentu na istoj poziciji, pa React resetuje sve state-ove ispod. Ovo prouzrokuje bug-ove i probleme sa performansama. Da biste izbegli ovaj problem, **uvek deklarišite funkcije komponenti na najvišem nivou i nemojte ugnježdavati njihove definicije**.

</Pitfall>

## Resetovanje state-a na istoj poziciji {/*resetting-state-at-the-same-position*/}

Po default-u, React čuva state komponente dok god ostaje na istoj poziciji. Uglavnom je to upravo ono što želite, pa ima smisla da je to default ponašanje. Ali, ponekad možete želeti da resetujete state komponente. Razmotrite ovu aplikaciju koja omogućava dvojici igrača da prate svoje poene tokom svakog poteza:

<Sandpack>

```js
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter person="Taylor" />
      ) : (
        <Counter person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Naredni igrač!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>Poeni za osobu {person}: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Dodaj jedan
      </button>
    </div>
  );
}
```

```css
h1 {
  font-size: 18px;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Trenutno, kada promenite igrača, poeni ostaju sačuvani. Dve `Counter` komponente se nalaze na istoj poziciji, pa ih React posmatra kao *isti* `Counter` čiji se `person` prop promenio.

Ali konceptualno, u ovoj aplikaciji to trebaju biti dva odvojena brojača. Iako se pojavljuju na istom mestu na UI-u, jedan brojač je za Taylor, a drugi za Sarah.

Postoje dva načina da resetujete state kada ih menjate:

1. Renderujte komponente na različitim pozicijama
2. Dajte svakoj komponenti eksplicitni identitet sa `key`


### Opcija 1: Renderovanje komponente na različitim pozicijama {/*option-1-rendering-a-component-in-different-positions*/}

Ako želite da ova dva `Counter`-a budu nezavisna, možete ih renderovati na dve različite pozicije:

<Sandpack>

```js
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA &&
        <Counter person="Taylor" />
      }
      {!isPlayerA &&
        <Counter person="Sarah" />
      }
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Naredni igrač!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>Poeni za osobu {person}: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Dodaj jedan
      </button>
    </div>
  );
}
```

```css
h1 {
  font-size: 18px;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

* Inicijalno, `isPlayerA` je `true`. To znači da prva pozicija sadrži `Counter` state, a druga je prazna.
* Kada kliknete dugme "Naredni igrač" prva pozicija se briše, a druga sada sadrži `Counter`.

<DiagramGroup>

<Diagram name="preserving_state_diff_position_p1" height={375} width={504} alt="Dijagram sa stablom React komponenata. Roditelj se naziva 'Scoreboard' i sadrži state balon sa nazivom 'isPlayerA' i vrednošću 'true'. Jedino dete, raspoređeno levo, se naziva 'Counter' i sadrži state balon sa nazivom 'count' i vrednošću 0. Celokupno levo dete je istaknuto žutom, označavajući da je dodato.">

Inicijalni state

</Diagram>

<Diagram name="preserving_state_diff_position_p2" height={375} width={504} alt="Dijagram sa stablom React komponenata. Roditelj se naziva 'Scoreboard' i sadrži state balon sa nazivom 'isPlayerA' i vrednošću 'false'. State balon je istaknut žutom, označavajući da je promenjen. Levo dete je zamenjeno sa žutom 'poof' slikom označavajući da je obrisano, a tu je novo dete sa desne strane, istaknuto žutom, označavajući da je dodato. Novo dete sa nazivom 'Counter' sadrži state balon sa nazivom 'count' i vrednošću 0.">

Kliknuto "naredno"

</Diagram>

<Diagram name="preserving_state_diff_position_p3" height={375} width={504} alt="Dijagram sa stablom React komponenata. Roditelj se naziva 'Scoreboard' i sadrži state balon sa nazivom 'isPlayerA' i vrednošću 'true'. State balon je istaknut žutom, označavajući da je promenjen. Postoji novo dete na levoj strani, istaknuto žuto, označavajući da je dodato. Novo dete sa nazivom 'Counter' sadrži state balon sa nazivom 'count' i vrednošću 0. Desno dete je zamenjeno sa žutom 'poof' slikom označavajući da je obrisano.">

Kliknuto "naredno" opet

</Diagram>

</DiagramGroup>

State svakog `Counter`-a bude uništen svaki put kad je uklonjen iz DOM-a. Zbog toga se resetuju svaki put kada kliknete dugme.

Ovo rešenje je zgodno kada imate samo par komponenti renderovanih na istom mestu. U ovom primeru, imate ih samo dve, pa nije problem renderovati ih u različitom JSX-u.

### Opcija 2: Resetovanje state-a uz key {/*option-2-resetting-state-with-a-key*/}

Postoji i drugi, više generički, način da resetujete state komponente.

Možda ste videli `key`-eve tokom [renderovanja listi](/learn/rendering-lists#keeping-list-items-in-order-with-key). Ključevi nisu samo za liste! Možete koristiti ključeve da naterate React da razlikuje bilo koje komponente. Po default-u, React koristi redosled unutar roditelja ("prvi brojač", "drugi brojač") da razlikuje komponente. Ali, ključevi vam omogućavaju da kažete React-u da to nije samo *prvi* brojač, ili *drugi* brojač, već poseban brojač--na primer, *Taylor-ov* brojač. Na ovaj način, React će znati za *Taylor-ov* brojač gde god da se pojavi u stablu!

U ovom primeru, dva `<Counter />`-a ne dele state iako se nalaze na istom mestu u JSX-u:

<Sandpack>

```js
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" />
      ) : (
        <Counter key="Sarah" person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Naredni igrač!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>Poeni za osobu {person}: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Dodaj jedan
      </button>
    </div>
  );
}
```

```css
h1 {
  font-size: 18px;
}

.counter {
  width: 100px;
  text-align: center;
  border: 1px solid gray;
  border-radius: 4px;
  padding: 20px;
  margin: 0 20px 20px 0;
}

.hover {
  background: #ffffd8;
}
```

</Sandpack>

Menjanje između Taylor i Sarah ne čuva state. To je zato što ste **im dali različite `key`-eve:**

```js
{isPlayerA ? (
  <Counter key="Taylor" person="Taylor" />
) : (
  <Counter key="Sarah" person="Sarah" />
)}
```

Specificiranje `key`-a govori React-u da koristi `key` kao deo pozicije umesto rasporeda unutar roditelja. Zbog toga, iako ih renderujete na istom mestu u JSX-u, React ih vidi kao dva različita brojača, tako da nikad neće deliti state. Svaki put kad se brojač pojavi na ekranu, njegov state je kreiran. Svaki put kad je uklonjen, njegov state je uništen. Promena između njih resetuje njihov state iznova.

<Note>

Zapamtite da ključevi nisu jedinstveni globalno. Oni samo označavaju poziciju *unutar roditelja*.

</Note>

### Resetovanje forme sa key-em {/*resetting-a-form-with-a-key*/}

Resetovanje state-a uz key je posebno korisno kada radite sa formama.

U ovoj aplikaciji za poruke, `<Chat>` komponenta sadrži state za tekstualni input:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat contact={to} />
    </div>
  )
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  selectedContact,
  contacts,
  onSelect
}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact);
            }}>
              {contact.name}
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={'Piši korisniku ' + contact.name}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>Pošalji na {contact.email}</button>
    </section>
  );
}
```

```css
.chat, .contact-list {
  float: left;
  margin-bottom: 20px;
}
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

Pokušajte da unesete nešto u input, a onda pritisnite "Alice" ili "Bob" da odaberete drugog primaoca. Primetićete da je input state sačuvan zato što je `<Chat>` renderovan na istoj poziciji u stablu.

**U mnogim aplikacijama ovo može biti željeno ponašanje, ali ne i u aplikaciji za poruke!** Ne želite da dopustite korisniku da pošalje već otkucanu poruku pogrešnoj osobi zbog slučajnog klika. Da biste ovo popravili, dodajte `key`:

```js
<Chat key={to.id} contact={to} />
```

Ovo osigurava da kada odaberete drugog primaoca, da će `Chat` komponenta biti napravljena od nule, uključujući bilo koji state u stablu ispod. React će takođe ponovo kreirati DOM elemente umesto da ih ponovo iskoristi.

Sada promena primaoca uvek briše tekstualno polje:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat key={to.id} contact={to} />
    </div>
  )
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  selectedContact,
  contacts,
  onSelect
}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact);
            }}>
              {contact.name}
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={'Piši korisniku ' + contact.name}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>Pošalji na {contact.email}</button>
    </section>
  );
}
```

```css
.chat, .contact-list {
  float: left;
  margin-bottom: 20px;
}
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

<DeepDive>

#### Čuvanje state-a za uklonjene komponente {/*preserving-state-for-removed-components*/}

U pravoj aplikaciji za poruke, verovatno ćete želeti da povratite input state kada korisnik izabere prethodnog primaoca ponovo. Postoji par načina da držite state "živim" za komponentu koja nije više vidljiva:

- Mogli biste da renderujete _sve_ chat-ove umesto samo jednog, ali da ostale sakrijete pomoću CSS-a. Chat-ovi neće biti uklonjeni iz stabla, pa će njihov lokalni state biti sačuvan. Ovo rešenje radi dobro za jednostavne UI-e. Ali, može postati dosta sporo ako su skrivena stabla velika i sadrže mnogo DOM čvorova.
- Mogli biste [podići state](/learn/sharing-state-between-components) i čuvati poruke na čekanju za svakog primaoca u roditeljskoj komponenti. Na ovaj način, nije bitno ako dečje komponente budu uklonjene, jer njihov roditelj čuva bitne informacije. Ovo je najčešće rešenje.
- Takođe možete koristiti i druge izvore informacija, kao ispomoć React state-u. Na primer, verovatno želite da sačuvate draft poruku čak iako korisnik slučajno zatvori stranicu. Da biste to implementirali, mogli biste napraviti da `Chat` komponenta inicijalizuje svoj state čitanjem iz [`localStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)-a, a takođe i da čuva draft poruke tamo.

Nevezano za to koju strategiju odaberete, chat _sa Alice_ je konceptualno drugačiji od chat-a _sa Bob-om_, tako da ima smisla zadati `key` u `<Chat>` stablo na osnovu trenutnog primaoca.

</DeepDive>

<Recap>

- React čuva state dok god se ista komponenta renderuje na istoj poziciji.
- State se ne čuva u JSX tag-ovima. Povezan je sa pozicijom u stablu na koju stavljate taj JSX.
- Možete forsirati podstablo da resetuje svoj state tako što ćete mu dati drugačiji key.
- Nemojte ugnježdavati definicije komponenti, ili ćete slučajno resetovati state.

</Recap>



<Challenges>

#### Popraviti nestajući tekst u input-u {/*fix-disappearing-input-text*/}

Ovaj primer prikazuje poruku kada kliknete dugme. Međutim, klik na dugme slučajno resetuje input. Zašto se ovo dešava? Popravite to tako da klik na dugme ne resetuje tekst u input-u.

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [showHint, setShowHint] = useState(false);
  if (showHint) {
    return (
      <div>
        <p><i>Pomoć: Vaš omiljeni grad?</i></p>
        <Form />
        <button onClick={() => {
          setShowHint(false);
        }}>Sakrij pomoć</button>
      </div>
    );
  }
  return (
    <div>
      <Form />
      <button onClick={() => {
        setShowHint(true);
      }}>Prikaži pomoć</button>
    </div>
  );
}

function Form() {
  const [text, setText] = useState('');
  return (
    <textarea
      value={text}
      onChange={e => setText(e.target.value)}
    />
  );
}
```

```css
textarea { display: block; margin: 10px 0; }
```

</Sandpack>

<Solution>

Problem je u tome što je `Form` renderovan na različitim pozicijama. U `if` grani, to je drugo dete od `<div>`-a, ali je u `else` grani prvo dete. Zbog toga se tip komponente na svakoj poziciji menja. Na prvoj poziciji se menjaju `p` i `Form`, dok se na drugoj poziciji menjaju `Form` i `button`. React resetuje state svaki put kada se tip komponente promeni.

Najlakše rešenje je da objedinite te dve grane da uvek renderuju `Form` na istoj poziciji:

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [showHint, setShowHint] = useState(false);
  return (
    <div>
      {showHint &&
        <p><i>Pomoć: Vaš omiljeni grad?</i></p>
      }
      <Form />
      {showHint ? (
        <button onClick={() => {
          setShowHint(false);
        }}>Sakrij pomoć</button>
      ) : (
        <button onClick={() => {
          setShowHint(true);
        }}>Prikaži pomoć</button>
      )}
    </div>
  );
}

function Form() {
  const [text, setText] = useState('');
  return (
    <textarea
      value={text}
      onChange={e => setText(e.target.value)}
    />
  );
}
```

```css
textarea { display: block; margin: 10px 0; }
```

</Sandpack>


Tehnički, mogli biste takođe dodati `null` pre `<Form />` u `else` grani kako bi se poklapala sa strukturom `if` grane:

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [showHint, setShowHint] = useState(false);
  if (showHint) {
    return (
      <div>
        <p><i>Pomoć: Vaš omiljeni grad?</i></p>
        <Form />
        <button onClick={() => {
          setShowHint(false);
        }}>Sakrij pomoć</button>
      </div>
    );
  }
  return (
    <div>
      {null}
      <Form />
      <button onClick={() => {
        setShowHint(true);
      }}>Prikaži pomoć</button>
    </div>
  );
}

function Form() {
  const [text, setText] = useState('');
  return (
    <textarea
      value={text}
      onChange={e => setText(e.target.value)}
    />
  );
}
```

```css
textarea { display: block; margin: 10px 0; }
```

</Sandpack>

Na ovaj način, `Form` je uvek drugo dete, pa će ostati na istoj poziciji i sačuvati svoj state. Ali, ovaj pristup je mnogo manje očigledan i uvodi rizik da će neko ukloniti taj `null`.

</Solution>

#### Zameniti dva polja u formi {/*swap-two-form-fields*/}

Ova forma vam omogućava da unesete ime i prezime. Takođe ima i checkbox koji kontroliše koje polje je prvo. Kada štiklirate checkbox, polje "Prezime" će se pojaviti pre polja "Ime".

Zamalo da radi, ali postoji bug. Ako popunite input "Ime" i štiklirate checkbox, tekst će ostati u prvom input-u (što je sada "Prezime"). Popravite ovo tako da se input tekst *takođe* pomera kada obrnete redosled.

<Hint>

Deluje da za ova polja nije dovoljna njihova pozicija unutar roditelja. Postoji li način da kažete React-u kako da poklopi state između ponovnih rendera?

</Hint>

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [reverse, setReverse] = useState(false);
  let checkbox = (
    <label>
      <input
        type="checkbox"
        checked={reverse}
        onChange={e => setReverse(e.target.checked)}
      />
      Obrni redosled
    </label>
  );
  if (reverse) {
    return (
      <>
        <Field label="Prezime" /> 
        <Field label="Ime" />
        {checkbox}
      </>
    );
  } else {
    return (
      <>
        <Field label="Ime" /> 
        <Field label="Prezime" />
        {checkbox}
      </>
    );    
  }
}

function Field({ label }) {
  const [text, setText] = useState('');
  return (
    <label>
      {label}:{' '}
      <input
        type="text"
        value={text}
        placeholder={label}
        onChange={e => setText(e.target.value)}
      />
    </label>
  );
}
```

```css
label { display: block; margin: 10px 0; }
```

</Sandpack>

<Solution>

Dodelite `key` u obe `<Field>` komponente i u `if` i u `else` granu. Ovo govori React-u kako da "poklopi" ispravan state za svaki `<Field>` čak iako se njihov redosled unutar roditelja promeni:

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [reverse, setReverse] = useState(false);
  let checkbox = (
    <label>
      <input
        type="checkbox"
        checked={reverse}
        onChange={e => setReverse(e.target.checked)}
      />
      Obrni redosled
    </label>
  );
  if (reverse) {
    return (
      <>
        <Field key="lastName" label="Prezime" /> 
        <Field key="firstName" label="Ime" />
        {checkbox}
      </>
    );
  } else {
    return (
      <>
        <Field key="firstName" label="Ime" /> 
        <Field key="lastName" label="Prezime" />
        {checkbox}
      </>
    );    
  }
}

function Field({ label }) {
  const [text, setText] = useState('');
  return (
    <label>
      {label}:{' '}
      <input
        type="text"
        value={text}
        placeholder={label}
        onChange={e => setText(e.target.value)}
      />
    </label>
  );
}
```

```css
label { display: block; margin: 10px 0; }
```

</Sandpack>

</Solution>

#### Resetovati formu sa detaljima {/*reset-a-detail-form*/}

Ovo je lista kontakata koja može da se menja. Možete menjati detalje odabranog kontakta i onda kliknuti "Sačuvaj" da ga ažurirate ili "Resetuj" da ukinete promene.

Kada izaberete drugi kontakt (na primer, Alice), state se ažurira ali forma i dalje prikazuje detalje prethodnog kontakta. Popravite ovo tako da se forma resetuje kada se odabrani kontakt promeni.

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ContactList from './ContactList.js';
import EditContact from './EditContact.js';

export default function ContactManager() {
  const [
    contacts,
    setContacts
  ] = useState(initialContacts);
  const [
    selectedId,
    setSelectedId
  ] = useState(0);
  const selectedContact = contacts.find(c =>
    c.id === selectedId
  );

  function handleSave(updatedData) {
    const nextContacts = contacts.map(c => {
      if (c.id === updatedData.id) {
        return updatedData;
      } else {
        return c;
      }
    });
    setContacts(nextContacts);
  }

  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={selectedId}
        onSelect={id => setSelectedId(id)}
      />
      <hr />
      <EditContact
        initialData={selectedContact}
        onSave={handleSave}
      />
    </div>
  )
}

const initialContacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  contacts,
  selectedId,
  onSelect
}) {
  return (
    <section>
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact.id);
            }}>
              {contact.id === selectedId ?
                <b>{contact.name}</b> :
                contact.name
              }
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/EditContact.js
import { useState } from 'react';

export default function EditContact({ initialData, onSave }) {
  const [name, setName] = useState(initialData.name);
  const [email, setEmail] = useState(initialData.email);
  return (
    <section>
      <label>
        Ime:{' '}
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        Email:{' '}
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: initialData.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        Sačuvaj
      </button>
      <button onClick={() => {
        setName(initialData.name);
        setEmail(initialData.email);
      }}>
        Resetuj
      </button>
    </section>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li { display: inline-block; }
li button {
  padding: 10px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

<Solution>

Dodelite `key={selectedId}` u `EditContact` komponentu. Na ovaj način, promena kontakata će resetovati formu:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ContactList from './ContactList.js';
import EditContact from './EditContact.js';

export default function ContactManager() {
  const [
    contacts,
    setContacts
  ] = useState(initialContacts);
  const [
    selectedId,
    setSelectedId
  ] = useState(0);
  const selectedContact = contacts.find(c =>
    c.id === selectedId
  );

  function handleSave(updatedData) {
    const nextContacts = contacts.map(c => {
      if (c.id === updatedData.id) {
        return updatedData;
      } else {
        return c;
      }
    });
    setContacts(nextContacts);
  }

  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={selectedId}
        onSelect={id => setSelectedId(id)}
      />
      <hr />
      <EditContact
        key={selectedId}
        initialData={selectedContact}
        onSave={handleSave}
      />
    </div>
  )
}

const initialContacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  contacts,
  selectedId,
  onSelect
}) {
  return (
    <section>
      <ul>
        {contacts.map(contact =>
          <li key={contact.id}>
            <button onClick={() => {
              onSelect(contact.id);
            }}>
              {contact.id === selectedId ?
                <b>{contact.name}</b> :
                contact.name
              }
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/EditContact.js
import { useState } from 'react';

export default function EditContact({ initialData, onSave }) {
  const [name, setName] = useState(initialData.name);
  const [email, setEmail] = useState(initialData.email);
  return (
    <section>
      <label>
        Ime:{' '}
        <input
          type="text"
          value={name}
          onChange={e => setName(e.target.value)}
        />
      </label>
      <label>
        Email:{' '}
        <input
          type="email"
          value={email}
          onChange={e => setEmail(e.target.value)}
        />
      </label>
      <button onClick={() => {
        const updatedData = {
          id: initialData.id,
          name: name,
          email: email
        };
        onSave(updatedData);
      }}>
        Sačuvaj
      </button>
      <button onClick={() => {
        setName(initialData.name);
        setEmail(initialData.email);
      }}>
        Resetuj
      </button>
    </section>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li { display: inline-block; }
li button {
  padding: 10px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

</Solution>

#### Ukloniti sliku dok se učitava {/*clear-an-image-while-its-loading*/}

Kada kliknete "Naredno", pretraživač počinje da učitava narednu sliku. Međutim, pošto je prikazana u istom `<img>` tag-u, po default-u ćete i dalje videti prethodnu sliku dok se naredna ne učita. Ovo može biti neželjeno ako je bitno da se tekst uvek poklapa sa slikom. Promenite tako da u trenutku kada kliknete "Naredno", prethodna slika bude odmah uklonjena.

<Hint>

Postoji li način da kažete React-a da ponovo kreira DOM umesto da ga ponovo iskoristi?

</Hint>

<Sandpack>

```js
import { useState } from 'react';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const hasNext = index < images.length - 1;

  function handleClick() {
    if (hasNext) {
      setIndex(index + 1);
    } else {
      setIndex(0);
    }
  }

  let image = images[index];
  return (
    <>
      <button onClick={handleClick}>
        Naredno
      </button>
      <h3>
        Slika {index + 1} od {images.length}
      </h3>
      <img src={image.src} />
      <p>
        {image.place}
      </p>
    </>
  );
}

let images = [{
  place: 'Penang, Malezija',
  src: 'https://i.imgur.com/FJeJR8M.jpg'
}, {
  place: 'Lisabon, Portugal',
  src: 'https://i.imgur.com/dB2LRbj.jpg'
}, {
  place: 'Bilbao, Španija',
  src: 'https://i.imgur.com/z08o2TS.jpg'
}, {
  place: 'Valparaíso, Čile',
  src: 'https://i.imgur.com/Y3utgTi.jpg'
}, {
  place: 'Švic, Švajcarska',
  src: 'https://i.imgur.com/JBbMpWY.jpg'
}, {
  place: 'Prag, Češka',
  src: 'https://i.imgur.com/QwUKKmF.jpg'
}, {
  place: 'Ljubljana, Slovenija',
  src: 'https://i.imgur.com/3aIiwfm.jpg'
}];
```

```css
img { width: 150px; height: 150px; }
```

</Sandpack>

<Solution>

Možete dodeliti `key` u `<img>` tag. Kada se taj `key` promeni, React će ponovo kreirati `<img>` DOM čvor od nule. Ovo uzrokuje kratki bljesak kad se svaka učita, pa nije nešto što biste želeli za svaku sliku u aplikaciji. Ali, ima smisla ako želite da osigurate da se slika uvek poklapa sa tekstom.

<Sandpack>

```js
import { useState } from 'react';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const hasNext = index < images.length - 1;

  function handleClick() {
    if (hasNext) {
      setIndex(index + 1);
    } else {
      setIndex(0);
    }
  }

  let image = images[index];
  return (
    <>
      <button onClick={handleClick}>
        Naredno
      </button>
      <h3>
        Slika {index + 1} od {images.length}
      </h3>
      <img key={image.src} src={image.src} />
      <p>
        {image.place}
      </p>
    </>
  );
}

let images = [{
  place: 'Penang, Malezija',
  src: 'https://i.imgur.com/FJeJR8M.jpg'
}, {
  place: 'Lisabon, Portugal',
  src: 'https://i.imgur.com/dB2LRbj.jpg'
}, {
  place: 'Bilbao, Španija',
  src: 'https://i.imgur.com/z08o2TS.jpg'
}, {
  place: 'Valparaíso, Čile',
  src: 'https://i.imgur.com/Y3utgTi.jpg'
}, {
  place: 'Švic, Švajcarska',
  src: 'https://i.imgur.com/JBbMpWY.jpg'
}, {
  place: 'Prag, Češka',
  src: 'https://i.imgur.com/QwUKKmF.jpg'
}, {
  place: 'Ljubljana, Slovenija',
  src: 'https://i.imgur.com/3aIiwfm.jpg'
}];
```

```css
img { width: 150px; height: 150px; }
```

</Sandpack>

</Solution>

#### Popraviti pogrešno postavljen state u listi {/*fix-misplaced-state-in-the-list*/}

U ovoj listi, svaki `Contact` ima state koji odlučuje da li je "Prikaži email" pritisnut za njega. Pritisnite "Prikaži email" za Alice, a onda štiklirajte checkbox "Prikaži u obrnutom redosledu". Primetićete da je _Taylor-ov_ email sada proširen, a Alice-in--koji je pomeren na dno--sklopljen.

Popravite ovo tako da je proširen state povezan sa svakim kontaktom, nevezano od odabranog rasporeda.

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Contact from './Contact.js';

export default function ContactList() {
  const [reverse, setReverse] = useState(false);

  const displayedContacts = [...contacts];
  if (reverse) {
    displayedContacts.reverse();
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={reverse}
          onChange={e => {
            setReverse(e.target.checked)
          }}
        />{' '}
        Prikaži u obrnutom redosledu
      </label>
      <ul>
        {displayedContacts.map((contact, i) =>
          <li key={i}>
            <Contact contact={contact} />
          </li>
        )}
      </ul>
    </>
  );
}

const contacts = [
  { id: 0, name: 'Alice', email: 'alice@mail.com' },
  { id: 1, name: 'Bob', email: 'bob@mail.com' },
  { id: 2, name: 'Taylor', email: 'taylor@mail.com' }
];
```

```js src/Contact.js
import { useState } from 'react';

export default function Contact({ contact }) {
  const [expanded, setExpanded] = useState(false);
  return (
    <>
      <p><b>{contact.name}</b></p>
      {expanded &&
        <p><i>{contact.email}</i></p>
      }
      <button onClick={() => {
        setExpanded(!expanded);
      }}>
        {expanded ? 'Sakrij' : 'Prikaži'} email
      </button>
    </>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li {
  margin-bottom: 20px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

<Solution>

Problem je što ovaj primer koristi indeks kao `key`:

```js
{displayedContacts.map((contact, i) =>
  <li key={i}>
```

Međutim, želite da state bude povezan sa _svakim posebnim kontaktom_.

Upotreba ID-a kontakta kao `key`-a popravlja problem:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Contact from './Contact.js';

export default function ContactList() {
  const [reverse, setReverse] = useState(false);

  const displayedContacts = [...contacts];
  if (reverse) {
    displayedContacts.reverse();
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={reverse}
          onChange={e => {
            setReverse(e.target.checked)
          }}
        />{' '}
        Prikaži u obrnutom redosledu
      </label>
      <ul>
        {displayedContacts.map(contact =>
          <li key={contact.id}>
            <Contact contact={contact} />
          </li>
        )}
      </ul>
    </>
  );
}

const contacts = [
  { id: 0, name: 'Alice', email: 'alice@mail.com' },
  { id: 1, name: 'Bob', email: 'bob@mail.com' },
  { id: 2, name: 'Taylor', email: 'taylor@mail.com' }
];
```

```js src/Contact.js
import { useState } from 'react';

export default function Contact({ contact }) {
  const [expanded, setExpanded] = useState(false);
  return (
    <>
      <p><b>{contact.name}</b></p>
      {expanded &&
        <p><i>{contact.email}</i></p>
      }
      <button onClick={() => {
        setExpanded(!expanded);
      }}>
        {expanded ? 'Sakrij' : 'Prikaži'} email
      </button>
    </>
  );
}
```

```css
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li {
  margin-bottom: 20px;
}
label {
  display: block;
  margin: 10px 0;
}
button {
  margin-right: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

State je povezan sa pozicijom u stablu. `key` vam omogućava da specificirate imenovanu poziciju umesto da se oslanjate na redosled.

</Solution>

</Challenges>
