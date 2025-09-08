---
title: 'Tutorijal: Iks-Oks'
---

<Intro>

Tokom ovog tutorijala napravićete jednostavnu igru iks-oks. Ovaj tutorijal ne zahteva prethodno znanje o React-u. Tehnike koje ćete naučiti su fundamentalne za pravljenje bilo koje React aplikacije, a potpuno razumevanje ovog tutorijala će vam pružiti duboko razumevanje React-a.

</Intro>

<Note>

Ovaj tutorijal je dizajniran za one koji preferiraju da ***uče kroz praksu*** i žele brzo da probaju napraviti nešto konkretno. Ukoliko više volite da učite svaki koncept korak po korak, za početak uzmite [Opisivanje UI-a](/learn/describing-the-ui)

</Note>

Tutorijal je podeljen u nekoliko sekcija:

- [Setup tutorijala](#setup-for-the-tutorial) pružiće vam ***početnu tačku*** za praćenje tutorijala.
- [Pregled](#overview) će vas naučiti ***osnovama*** React-a: component-ama, props-ima i state-u.
- [Završavanje igre](#completing-the-game) će vas naučiti ***najčešćim tehnikama*** u radu sa React-om.
- [Dodavanje putovanja kroz vreme](#adding-time-travel) pružiće vam ***dublji uvid*** u jedinstvene prednosti React-a.

### Šta pravite? {/*what-are-you-building*/}

U ovom tutorijalu napravićete interaktivnu igru iks-oks koristeći React.

Ovde možete videti kako će izgledati gotov projekat:

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({ value, onSquareClick }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

function Board({ xIsNext, squares, onPlay }) {
  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    onPlay(nextSquares);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = 'Pobednik: ' + winner;
  } else {
    status = 'Sledeći igrač : ' + (xIsNext ? 'X' : 'O');
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

export default function Game() {
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const [currentMove, setCurrentMove] = useState(0);
  const xIsNext = currentMove % 2 === 0;
  const currentSquares = history[currentMove];

  function handlePlay(nextSquares) {
    const nextHistory = [...history.slice(0, currentMove + 1), nextSquares];
    setHistory(nextHistory);
    setCurrentMove(nextHistory.length - 1);
  }

  function jumpTo(nextMove) {
    setCurrentMove(nextMove);
  }

  const moves = history.map((squares, move) => {
    let description;
    if (move > 0) {
      description = 'Prebacite se na potez #' + move;
    } else {
      description = 'Prebacite se na početak';
    }
    return (
      <li key={move}>
        <button onClick={() => jumpTo(move)}>{description}</button>
      </li>
    );
  });

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
      </div>
      <div className="game-info">
        <ol>{moves}</ol>
      </div>
    </div>
  );
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

Ako vam kod još uvek nije jasan, ili ako niste upoznati sa sintaksom, ne brinite! Cilj ovog tutorijala je da vam pomogne da razumete React i njegovu sintaksu.

Preporučujemo vam da se prvo poigrate sa završenom igrom pre nego što nastavite sa tutorijalom. Jedna od funkcionalnosti koju ćete primetiti je numerisana lista sa desne strane table igre. Ova lista vam pruža istoriju svih poteza koji su se odigrali tokom igre, i update-je se kako igra napreduje.

Nakon što se upoznate sa završenom igrom, nastavite sa čitanjem. Sledeći korak je da vas postavimo u poziciju da počnete sa pravljenjem igre.

## Setup tutorijala {/*setup-for-the-tutorial*/}

U live code editor ispod, kliknite na ***Fork*** u gornjem desnom uglu kako bi otvorili editor u novom tabu koristeći CodeSandbox. CodeSandbox vam omogućava da pišete kod direktno u browseru i pregledate kako će aplikacija izgledati korisnicima. Novi tab bi trebalo da prikaže prazan kvadrat i početni kod za ovaj tutorijal.

<Sandpack>

```js src/App.js
export default function Square() {
  return <button className="square">X</button>;
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

<Note>

Takođe možete pratiti ovaj vodič koristeći vaše lokalno okruženje. Da biste to uradili, potrebno je da:

1. Instalirate [Node.js](https://nodejs.org/en/)
1. U CodeSandbox tab-u koju ste ranije otvorili, kliknite dugme u gornjem levom uglu da otvorite meni, a zatim izaberite **Download Sandbox** u tom meniju kako biste preuzeli arhivu fajlova lokalno
1. Otpakujte arhivu, zatim otvorite terminal i `cd` do direktorijuma u koji ste otpakovali fajlove
1. Instalirajte zavisnosti (dependancies) sa `npm install`
1. Pokrenite `npm start` da biste startovali lokalni server i pratite uputstva da biste videli kako kod radi u browseru

Ukoliko se u ovom procesu zaglavite, nemojte dozvoliti da vas to zaustavi! Pratite uputstva online i pokušajte ponovo docnije da postavite lokalno okruženje.

</Note>

## Pregled {/*overview*/}

Sada kada ste postavili okruženje, hajde da napravimo pregled React-a!

### Pregled starter koda {/*inspecting-the-starter-code*/}

U CodeSandbox-u ćete videti tri glavne sekcije:

![CodeSandbox sa starter kodom](../images/tutorial/react-starter-code-codesandbox.png)

<<<<<<< HEAD
1. Sekcija *Files* sa listom fajlova kao što su `App.js`, `index.js`, `styles.css` i folder pod nazivom `public`
1. *code editor* gde ćete videti source kod odabranog fajla
1. Sekcija *browser* gde ćete videti kako će kod koji ste napisali biti prikazan
=======
1. The _Files_ section with a list of files like `App.js`, `index.js`, `styles.css` in `src` folder and a folder called `public`
1. The _code editor_ where you'll see the source code of your selected file
1. The _browser_ section where you'll see how the code you've written will be displayed
>>>>>>> d34c6a2c6fa49fc6f64b07aa4fa979d86d41c4e8

Fajl `App.js` bi trebalo da bude izabran u sekciji *Files*. Sadržaj tog fajla u *code editor* bi trebalo da izgleda ovako:

```jsx
export default function Square() {
  return <button className="square">X</button>;
}
```

Sekcija *browser* bi trebalo da prikazuje kvadrat sa X u njemu ovako:

![kvadrat sa X](../images/tutorial/x-filled-square.png)

Sada, hajde da pogledamo fajlove u starter kodu.

#### `App.js` {/*appjs*/}

Kod u fajlu `App.js` kreira *component-u*. U React-u, component-a je deo višekratnog koda koji predstavlja deo korisničkog interfejsa. Component-e se koriste za renderovanje, upravljanje i update-ovanje elemenata korisničkog interfejsa u vašoj aplikaciji. Hajde da analiziramo component-u liniju po liniju kako bismo uočili šta se dešava:

```js {1}
export default function Square() {
  return <button className="square">X</button>;
}
```

Prva linija definiše funkciju pod nazivom `Square`. JavaScript ključna reč `export` omogućava da ova funkcija bude dostupna van ovog fajla. Ključna reč `default` označava da je to glavna funkcija u vašem fajlu koju će drugi fajlovi koristiti.

```js {2}
export default function Square() {
  return <button className="square">X</button>;
}
```

Druga linija *return*-uje dugme. JavaScript ključna reč `return` znači da se ono što dolazi posle nje vraća kao vrednost pozivaocu funkcije. `<button>` je *JSX element*. JSX element je kombinacija JavaScript koda i HTML oznaka koja opisuje šta želite da prikažete. `className="square"` je svojstvo dugmeta ili *prop* koje CSS-u govori kako da stilizuje dugme. `X` je tekst koji se prikazuje unutar dugmeta, a `</button>` zatvara JSX element, označavajući da bilo koji sadržaj nakon toga ne treba da bude postavljen unutar dugmeta.

#### `styles.css` {/*stylescss*/}

Kliknite na fajl pod nazivom `styles.css` u sekciji *Files* u CodeSandbox-u. Ovaj fajl definiše stilove za vašu React aplikaciju. Prva dva *CSS selektora* (`*` i `body`) definišu stil za veće delove vaše aplikacije, dok selektor `.square` definiše stil za bilo koju component-u gde je svojstvo `className` postavljeno na `square`. U vašem kodu, to bi odgovaralo dugmetu iz component-e Square u fajlu `App.js`.

#### `index.js` {/*indexjs*/}

Kliknite na fajl pod nazivom `index.js` u sekciji *Files* u CodeSandbox-u. Nećete editovati ovaj fajl tokom tutorijala, ali on predstavlja sponu između component-e koju ste kreirali u fajlu `App.js` i web browsera.

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';
```

Linije 1-5 spajaju sve potrebne delove zajedno:

- React
- React-ov library za komunikaciju sa web browserima (React DOM)
- stilove za vaše component-e
- component-u koju ste kreirali u `App.js`.

Ostatak fajla spaja sve delove i ubacuje finalni proizvod u `index.html` u folderu `public`.

### Kreiranje table {/*building-the-board*/}

Vratimo se na `App.js`. Ovdee ćete provesti ostatak tutorijala.

Trenutno je tabla samo jedan kvadrat, ali vam je neophodno devet! Ukoliko pokušate samo da kopirate i nalepite vaš kvadrat kako biste napravili dva kvadrata ovako:

```js {2}
export default function Square() {
  return <button className="square">X</button><button className="square">X</button>;
}
```

Dobićete ovu grešku:

<ConsoleBlock level="error">

/src/App.js: Adjacent JSX elements must be wrapped in an enclosing tag. Did you want a JSX Fragment `<>...</>`?

</ConsoleBlock>

React component-e moraju da vraćaju jedinstven JSX element, a ne više susednih JSX elemenata poput dva dugmeta. Da biste to ispravili, možete koristiti *Fragmente* (`<>` i `</>`) kako biste obuhvatili više susednih JSX elemenata ovako:

```js {3-6}
export default function Square() {
  return (
    <>
      <button className="square">X</button>
      <button className="square">X</button>
    </>
  );
}
```

Sada bi trebalo da vidite:

![dva kvadrata sa X](../images/tutorial/two-x-filled-squares.png)

Odlično! Sada samo treba nekoliko puta da kopirate i nalepite kako biste dodali devet kvadrata i...

![devet kvadrata sa X u liniji](../images/tutorial/nine-x-filled-squares.png)

Oh ne! Svi kvadrati su u jednoj liniji, a ne u mreži kakva vam je potrebna za našu tablu. Da biste to ispravili, moraćete da grupišete kvadrate u redove koristeći `div` i dodate nekoliko CSS klasa. Usput, dodelićete svakom kvadratu broj kako biste bili sigurni gde je svaki kvadrat prikazan.

U fajlu `App.js`, update-ujete component-u `Square` da izgleda ovako:

```js {3-19}
export default function Square() {
  return (
    <>
      <div className="board-row">
        <button className="square">1</button>
        <button className="square">2</button>
        <button className="square">3</button>
      </div>
      <div className="board-row">
        <button className="square">4</button>
        <button className="square">5</button>
        <button className="square">6</button>
      </div>
      <div className="board-row">
        <button className="square">7</button>
        <button className="square">8</button>
        <button className="square">9</button>
      </div>
    </>
  );
}
```

CSS definisan u fajlu `styles.css` stilizuje div-ove sa `className` postavljenim na `board-row`. Sada kada ste grupisali svoje component-e u redove sa stilizovanim `div`-ovima, imate vašu iks-oks tablu:

![iks-oks tabla ispunjena brojevima od 1 do 9](../images/tutorial/number-filled-board.png)

Ali sada imate problem. Vaša component-a pod nazivom `Square` više zapravo nije kvadrat. Hajde da to ispravimo promenom imena u `Board`:

```js {1}
export default function Board() {
  //...
}
```

U ovom trenutku vaš kod bi trebalo da izgleda ovako:

<Sandpack>

```js
export default function Board() {
  return (
    <>
      <div className="board-row">
        <button className="square">1</button>
        <button className="square">2</button>
        <button className="square">3</button>
      </div>
      <div className="board-row">
        <button className="square">4</button>
        <button className="square">5</button>
        <button className="square">6</button>
      </div>
      <div className="board-row">
        <button className="square">7</button>
        <button className="square">8</button>
        <button className="square">9</button>
      </div>
    </>
  );
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

<Note>

Psssst... To je previše za kucanje! U redu je da kopirate i nalepite kod sa ove stranice. Međutim, ako ste raspoloženi za mali izazov, preporučujemo da kopirate samo onaj kod koji ste bar jednom već ručno otkucali.

</Note>

### Prosleđivanje podataka putem props-a {/*passing-data-through-props*/}

Sledeće, želećete da promenite vrednost kvadrata iz praznog u "X" kada korisnik klikne na kvadrat. Na osnovu toga kako ste do sada izgradili tablu, morali biste da kopirate i nalepite kod koji update-uje kvadrat devet puta (jednom za svaki kvadrat koji imate)! Umesto kopiranja i lepljenja, React-ova component-na arhitektura vam omogućava da kreirate ponovo iskoristivu component-u kako biste izbegli nered i dupliran kod.

Prvo, kopiraćete liniju koja definiše vaš prvi kvadrat (`<button className="square">1</button>`) iz component-e `Board` u novu component-u `Square`:

```js {1-3}
function Square() {
  return <button className="square">1</button>;
}

export default function Board() {
  // ...
}
```

Zatim ćete update-ovati Board component-u da renderuje ovu `Square` component-u koristeći JSX sintaksu:

```js {5-19}
// ...
export default function Board() {
  return (
    <>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
    </>
  );
}
```

Obratite pažnju na to da, za razliku od browser-ovih `div`-ova, vaše component-e `Board` i `Square` moraju počinjati velikim slovom.

Hajde da pogledamo:

![tabla popunjena jedinicama](../images/tutorial/board-filled-with-ones.png)

Oh ne! Izgubili ste numerisane kvadrate koje ste imali ranije. Sada svaki kvadrat prikazuje "1". Da biste to ispravili, koristićete *props* za prosleđivanje vrednosti koju svaki kvadrat treba da ima iz parent component-e (`Board`) ka njenom child-u (`Square`).

Update-ujte component-u `Square` da koristi `value` prop koji ćete proslediti iz component-e `Board`:

```js {1}
function Square({ value }) {
  return <button className="square">1</button>;
}
```

`function Square({ value })` označava da Square component-a može primiti prop pod nazivom `value`.

Sada želite da prikažete taj `value` umesto `1` unutar svakog kvadrata. Pokušajte to da uradite ovako:

```js {2}
function Square({ value }) {
  return <button className="square">value</button>;
}
```

Ups, ovo nije ono što ste želeli:

![tabla popunjena sa "value"](../images/tutorial/board-filled-with-value.png)

Želeli ste da renderujete JavaScript promenljivu pod nazivom `value` iz vaše component-e, a ne reč "value". Da biste "pobegli u JavaScript" iz JSX-a, potrebno je da koristite vitičaste (kovrdžave) zagrade. Dodajte vitičaste zagrade oko `value` u JSX-u ovako:

```js {2}
function Square({ value }) {
  return <button className="square">{value}</button>;
}
```

Za sada, trebalo bi da vidite praznu tablu:

![prazna tabla](../images/tutorial/empty-board.png)

To je zato što component-a `Board` još uvek nije prosledila `value` prop svakoj component-i `Square` koju renderuje. Da biste to ispravili, dodaćete `value` prop svakoj component-i `Square` koju renderuje component-a `Board`:

```js {5-7,10-12,15-17}
export default function Board() {
  return (
    <>
      <div className="board-row">
        <Square value="1" />
        <Square value="2" />
        <Square value="3" />
      </div>
      <div className="board-row">
        <Square value="4" />
        <Square value="5" />
        <Square value="6" />
      </div>
      <div className="board-row">
        <Square value="7" />
        <Square value="8" />
        <Square value="9" />
      </div>
    </>
  );
}
```

Sada bi trebalo ponovo da vidite mrežu sa brojevima:

![iks-oks tabla ispunjena brojevima od 1 do 9](../images/tutorial/number-filled-board.png)

Vaš update-ovani kod bi trebalo da izgleda ovako:

<Sandpack>

```js src/App.js
function Square({ value }) {
  return <button className="square">{value}</button>;
}

export default function Board() {
  return (
    <>
      <div className="board-row">
        <Square value="1" />
        <Square value="2" />
        <Square value="3" />
      </div>
      <div className="board-row">
        <Square value="4" />
        <Square value="5" />
        <Square value="6" />
      </div>
      <div className="board-row">
        <Square value="7" />
        <Square value="8" />
        <Square value="9" />
      </div>
    </>
  );
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

### Izrada interaktivne component-e {/*making-an-interactive-component*/}

Hajde da ispunimo `Square` component-u sa `X` kada kliknete na nju. Deklarišite funkciju pod nazivom `handleClick` unutar component-e `Square`. Zatim, dodajte `onClick` u props dugmeta koje se vraća iz JSX elementa u `Square` component-i:

```js {2-4,9}
function Square({ value }) {
  function handleClick() {
    console.log('clicked!');
  }

  return (
    <button
      className="square"
      onClick={handleClick}
    >
      {value}
    </button>
  );
}
```

Ukoliko sada kliknete na kvadrat, trebalo bi da vidite log sa porukom `"clicked!"` u *Console* tab-u na dnu sekcije *Browser* u CodeSandbox-u. Klikovima na kvadrat više puta, logovaće se `"clicked!"` ponovo. Ponavljajući logovi sa istom porukom neće kreirati nove linije u konzoli. Umesto toga, videćete brojač koji se povećava pored vašeg prvog loga `"clicked!"`.

<Note>

Ako pratite ovaj tuutorijal koristeći vaše lokalno okruženje, potrebno je da otvorite konzolu vašeg browsera. Na primer, ako koristite Chrome browser, možete pogledati konzolu pomoću prečice na tastaturi **Shift + Ctrl + J** (na Windows/Linux) ili **Option + ⌘ + J** (na macOS).

</Note>

Sledeći korak je da `Square` component-a "zapamti" da je kliknuta i da je ispuni oznakom "X". Da bi "zapamtile" stvari, component-e koriste *state*.

React obezbeđuje posebnu funkciju pod nazivom `useState` koju možete pozvati iz svoje component-e kako bi ona mogla da "pamti" stvari. Hajde da sačuvamo trenutnu vrednost `Square` component-e u state-u i promenimo je kada se klikne na `Square`.

Importujte `useState` na vrhu fajla. Uklonite `value` prop iz component-e `Square`. Umesto toga, dodajte novu liniju na početku `Square` component-e koja poziva `useState`. Neka ona vrati promenljivu state-a pod nazivom `value`:

```js {1,3,4}
import { useState } from 'react';

function Square() {
  const [value, setValue] = useState(null);

  function handleClick() {
    //...
```

`value` čuva vrednost, a `setValue` je funkcija koja se koristi za promenu te vrednosti. `null` koji je prosleđen u `useState` koristi se kao početna vrednost za ovu promenljivu stanja, tako da `value` ovde počinje sa vrednošću jednakom `null`.

Pošto component-a `Square` više ne prihvata props, uklonićete `value` prop iz svih devet `Square` component-i koje kreira component-a `Board`:

```js {6-8,11-13,16-18}
// ...
export default function Board() {
  return (
    <>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
    </>
  );
}
```

Sada ćete promeniti component-u `Square` da prikaže "X" kada se klikne. Zamenite event handler `console.log("clicked!");` sa `setValue('X');`. Sada vaša `Square` component-a izgleda ovako:

```js {5}
function Square() {
  const [value, setValue] = useState(null);

  function handleClick() {
    setValue('X');
  }

  return (
    <button
      className="square"
      onClick={handleClick}
    >
      {value}
    </button>
  );
}
```

Pozivanjem ove `set` funkcije iz `onClick` handler-a, govorite React-u da ponovo renderuje taj `Square` kad god se klikne njegov `<button>`. Nakon update-a, `value` component-e `Square` biće `'X'`, tako da ćete videti "X" na tabli. Kliknite na bilo koji kvadrat i "X" bi trebalo da se pojavi:

![dodavanje X-ova na tablu](../images/tutorial/tictac-adding-x-s.gif)

Svaki kvadrat ima svoj state: `value` sačuvano u svakom kvadratu je potpuno nezavisno od drugih. Kada pozovete `set` funkciju u component-i, React automatski update-uje i njene unutarnje child component-e.

Nakon što napravite gore navedene izmene, vaš kod će izgledati ovako:

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square() {
  const [value, setValue] = useState(null);

  function handleClick() {
    setValue('X');
  }

  return (
    <button
      className="square"
      onClick={handleClick}
    >
      {value}
    </button>
  );
}

export default function Board() {
  return (
    <>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
      <div className="board-row">
        <Square />
        <Square />
        <Square />
      </div>
    </>
  );
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

### React Developer Tools {/*react-developer-tools*/}

React DevTools vam omogućava da proverite props i stanje vaših React component-i. Možete pronaći karticu React DevTools na dnu sekcije *browser* u CodeSandbox-u:

![React DevTools u CodeSandbox-u](../images/tutorial/codesandbox-devtools.png)

Da biste pregledali određenu component-u na ekranu, koristite dugme u gornjem levom uglu React DevTools-a:

![Biranje component-i na stranici sa React DevTools](../images/tutorial/devtools-select.gif)

<Note>

Za rad u lokalnom okruženju, React DevTools je dostupan kao ekstenzija za [Chrome](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en), [Firefox](https://addons.mozilla.org/en-US/firefox/addon/react-devtools/), i [Edge](https://microsoftedge.microsoft.com/addons/detail/react-developer-tools/gpphkfbcpidddadnkolkpfckpihlkkil) browsere. Instalirajte je, i *Components* tab će se pojaviti u Developer Tools vašeg browser-a za sajtove koji koriste React.

</Note>

## Završavanje igre {/*completing-the-game*/}

Do ovog trenutka, imate sve osnovne građevinske blokove za vašu iks-oks igru. Da biste je kompletirali, potrebno da naizmenično postavljate "X" i "O" na tablu, i potreban vam je način da odredite pobednika.

### Podizanje state-a {/*lifting-state-up*/}

Trenutno svaka `Square` component-a čuva deo state-a igre. Da bi se proverilo ko je pobednik u igri iks-oks, `Board` bi morao nekako da zna state svake od 9 `Square` component-i.

Kako biste to rešili? Možda biste pomislili da `Board` treba da „pita” svaku `Square` component-u za njen state. Iako je ovaj pristup tehnički moguć u React-u, ne preporučujemo ga jer kod postaje težak za razumevanje, podložan je greškama i teško ga je refaktorisati. Umesto toga, najbolji pristup je da se state igre čuva u parent component-i `Board`, umesto u svakoj `Square` component-i. component-a `Board` može da kaže svakoj `Square` component-i šta treba da prikaže prosleđivanjem props-a, kao što ste ranije prosleđivali broj svakoj `Square` component-i.

**Da biste sakupili podatke od više child component-i ili omogućili komunikaciju između dve child component-e, definišite zajednički state u njihovoj parent component-i. Parent component-a može da prosledi taj state nazad child component-ama putem props-a. Na ovaj način child component-e ostaju sinhroarray-ovane međusobno i sa parent component-om.**

Podizanje state-a u parent component-u je uobičajena praksa pri refaktorisanju React component-i.

Hajde da iskoristimo ovu priliku i isprobamo ovo. Izmenite component-u `Board` tako da deklariše varijablu state-a pod nazivom `squares`, koja ima podrazumevanu vrednost u vidu array-a od 9 `null` vrednosti koje odgovaraju za svaki od 9 kvadrata:

```js {3}
// ...
export default function Board() {
  const [squares, setSquares] = useState(Array(9).fill(null));
  return (
    // ...
  );
}
```

`Array(9).fill(null)` kreira array sa devet elemenata i postavlja svaki od njih na `null`. Poziv `useState()` oko njega deklariše varijablu state-a pod nazivom `squares`, koja je inicijalno postavljena na taj array. Svaki element u array-u odgovara vrednosti jednog kvadrata. Kada kasnije popunite tablu, array `squares` će izgledati ovako:

```jsx
['O', null, 'X', 'X', 'X', 'O', 'O', null, null]
```

Sada vaša `Board` component-a treba da prosledi `value` prop svakoj `Square` component-i koju renderuje:

```js {6-8,11-13,16-18}
export default function Board() {
  const [squares, setSquares] = useState(Array(9).fill(null));
  return (
    <>
      <div className="board-row">
        <Square value={squares[0]} />
        <Square value={squares[1]} />
        <Square value={squares[2]} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} />
        <Square value={squares[4]} />
        <Square value={squares[5]} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} />
        <Square value={squares[7]} />
        <Square value={squares[8]} />
      </div>
    </>
  );
}
```

Zatim ćete izmeniti `Square` component-u da prima `value` prop iz `Board` component-e. Ovo će zahtevati uklanjanje sopstvenog state-a `value` iz `Square` component-e, kao i `onClick` props-a dugmeta:

```js {1,2}
function Square({value}) {
  return <button className="square">{value}</button>;
}
```

U ovom trenutku trebalo bi da vidite praznu tablu za igru iks-oks:

![prazna tabla](../images/tutorial/empty-board.png)

A vaš kod bi trebalo da izgleda ovako:

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({ value }) {
  return <button className="square">{value}</button>;
}

export default function Board() {
  const [squares, setSquares] = useState(Array(9).fill(null));
  return (
    <>
      <div className="board-row">
        <Square value={squares[0]} />
        <Square value={squares[1]} />
        <Square value={squares[2]} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} />
        <Square value={squares[4]} />
        <Square value={squares[5]} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} />
        <Square value={squares[7]} />
        <Square value={squares[8]} />
      </div>
    </>
  );
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

Svaka `Square` component-a će sada primati `value` prop koji može imati vrednost `'X'`, `'O'` ili `null` za prazne kvadrate.

Sledeće, treba da promenite šta se dešava kada se klikne na `Square`. `Board` component-a sada vodi računa o tome koji su kvadrati popunjeni. Biće vam potrebno da napravite način na koji `Square` može da update-uje state component-e `Board`. Pošto je state privatan za component-u koja ga definiše, ne možete  update-ovati state component-e `Board` direktno iz `Square`.

Umesto toga, prosledićete funkciju iz `Board` component-e u `Square` component-u, i `Square` će pozvati tu funkciju kada se na nju klikne. Počećete sa funkcijom koju će `Square` component-a pozvati kada se klikne na nju. Tu funkciju ćete nazvati `onSquareClick`:

```js {3}
function Square({ value }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}
```

Zatim, dodaćete funkciju `onSquareClick` u props `Square` component-e:

```js {1}
function Square({ value, onSquareClick }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}
```

Sada ćete povezati `onSquareClick` prop sa funkcijom u `Board` component-i koju ćete nazvati `handleClick`. Da biste povezali `onSquareClick` sa `handleClick`, prosledićete funkciju kao vrednost `onSquareClick` prop-u prve `Square` component-e:

```js {7}
export default function Board() {
  const [squares, setSquares] = useState(Array(9).fill(null));

  return (
    <>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={handleClick} />
        //...
  );
}
```

Na kraju, definisaćete funkciju `handleClick` unutar `Board` component-e kako biste update-ovali array- `squares` koji čuva state vaše table:

```js {4-8}
export default function Board() {
  const [squares, setSquares] = useState(Array(9).fill(null));

  function handleClick() {
    const nextSquares = squares.slice();
    nextSquares[0] = "X";
    setSquares(nextSquares);
  }

  return (
    // ...
  )
}
```

Funkcija `handleClick` kreira kopiju array-a `squares` (`nextSquares`) korišćenjem JavaScript metode `slice()` za array-ove. Zatim, `handleClick` update-uje array- `nextSquares` tako što dodaje `X` na prvi kvadrat (indeks `[0]`).

Pozivanjem funkcije `setSquares` obaveštavate React da se state component-e promenio. Ovo će pokrenuti ponovno renderovanje component-i koje koriste state `squares` (`Board`), kao i njenih child component-i (`Square` component-e koje čine tablu).

<Note>

JavaScript podržava [closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures), što znači da unutrašnja funkcija (npr. `handleClick`) ima pristup varijablama i funkcijama definisanim u spoljašnjoj funkciji (npr. `Board`). Funkcija `handleClick` može da čita state `squares` i poziva metodu `setSquares` jer su obe definisane unutar funkcije `Board`.

</Note>

Sada možete dodati X-ove na tablu... ali samo u gornji levi kvadrat. Vaša funkcija `handleClick` je trenutno hardkodirana da update-uje indeks gornjeg levog kvadrata (`0`). Napravimo update `handleClick` tako da može update-ovati bilo koji kvadrat. Dodajte argument `i` funkciji `handleClick` koji prima indeks kvadrata koji treba update-ovati:

```js {4,6}
export default function Board() {
  const [squares, setSquares] = useState(Array(9).fill(null));

  function handleClick(i) {
    const nextSquares = squares.slice();
    nextSquares[i] = "X";
    setSquares(nextSquares);
  }

  return (
    // ...
  )
}
```

Potom ćete morati da prosledite taj `i` funkciji `handleClick`. Mogli biste pokušati da postavite `onSquareClick` prop kvadrata direktno na `handleClick(0)` u JSX-u, ovako, ali to neće funkcionisati:

```jsx
<Square value={squares[0]} onSquareClick={handleClick(0)} />
```

Evo zašto ovo ne funkcioniše. Poziv `handleClick(0)` postaje deo renderovanja component-e `Board`. Pošto `handleClick(0)` menja state component-e `Board` pozivanjem `setSquares`, cela component-a `Board` će ponovo biti renderovana. Međutim, to ponovo pokreće `handleClick(0)`, što dovodi do infinite loop-a:

<ConsoleBlock level="error">

Too many re-renders. React limits the number of renders to prevent an infinite loop.

</ConsoleBlock>

Zašto se ovaj problem nije pojavio ranije?

Kada ste prosleđivali `onSquareClick={handleClick}`, prosleđivali ste funkciju `handleClick` kao prop. Niste je pozivali! Ali sada *pozivate* tu funkciju odmah—obratite pažnju na zagrade u `handleClick(0)`—zbog čega se funkcija pokreće prerano. Ne želite da pozovete `handleClick` dok korisnik ne klikne!

Ovaj problem biste mogli rešiti kreiranjem funkcije poput `handleFirstSquareClick`, koja poziva `handleClick(0)`, funkcije poput `handleSecondSquareClick`, koja poziva `handleClick(1)`, i tako dalje. Zatim biste te funkcije prosledili (umesto da ih pozovete) kao props, na primer `onSquareClick={handleFirstSquareClick}`. Ovo bi rešilo problem infinite loop-a.

Međutim, definisanje devet različitih funkcija i davanje imena svakoj od njih bilo bi previše opširno. Umesto toga, hajde da uradimo sledeće:

```js {6}
export default function Board() {
  // ...
  return (
    <>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        // ...
  );
}
```

Obratite pažnju na novu sintaksu `() =>`. Ovde, `() => handleClick(0)` predstavlja *arrow funkciju,* koja je kraći način za definisanje funkcija. Kada korisnik klikne na kvadrat, kod koji se nalazi posle strelice `=>` će se izvršiti, pozivajući `handleClick(0)`.

Sada treba da update-ujete ostalih osam kvadrata kako bi pozivali `handleClick` iz arrow funkcija koje prosleđujete. Uverite se da argument za svaki poziv funkcije `handleClick` odgovara indeksu odgovarajućeg kvadrata:

```js {6-8,11-13,16-18}
export default function Board() {
  // ...
  return (
    <>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
};
```

Sada ponovo možete dodavati X-ove na bilo koji kvadrat na tabli klikom na njih:

![popunjavanje table sa X](../images/tutorial/tictac-adding-x-s.gif)

Ali ovoga puta celokupno upravljanje state-om se obavlja u `Board` component-i!

Ovako bi vaš kod trebalo da izgleda:

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({ value, onSquareClick }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

export default function Board() {
  const [squares, setSquares] = useState(Array(9).fill(null));

  function handleClick(i) {
    const nextSquares = squares.slice();
    nextSquares[i] = 'X';
    setSquares(nextSquares);
  }

  return (
    <>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

Sada kada se upravljanje state-om nalazi u `Board` component-i, parent component-a `Board` prosleđuje props child component-ama `Square`, omogućavajući im da se ispravno prikažu. Kada kliknete na `Square`, child component-a `Square` sada traži od parent component-e `Board` da update-uje state table. Kada se state `Board` component-e promeni, i `Board` component-a i svaka child component-a `Square` automatski se ponovo renderuju. Čuvanje state-a svih kvadrata u `Board` component-i omogućiće joj da u budućnosti odredi pobednika.

Sumirajmo šta se dešava kada korisnik klikne na gornji levi kvadrat na vašoj tabli kako bi dodao `X`:

1. Klik na gornji levi kvadrat pokreće funkciju koju je `button` dobio kao svoj `onClick` prop iz `Square` component-e. `Square` component-a je tu funkciju dobila kao svoj `onSquareClick` prop iz `Board` component-e. `Board` component-a je tu funkciju definisala direktno u JSX-u i poziva `handleClick` sa argumentom `0`.
2. Funkcija `handleClick` koristi argument (`0`) da update-uje prvi element array-a `squares` sa `null` na `X`.
3. State `squares` iz `Board` component-e se update-uje, pa se `Board` i sve njene child component-e ponovo renderuju. Ovo izaziva promenu `value` prop-a component-e `Square` sa indeksom `0` iz `null` u `X`.

Na kraju, korisnik vidi da se gornji levi kvadrat promenio iz praznog u kvadrat sa `X` nakon što je kliknuo na njega.

<Note>

DOM atribut `onClick` elementa `<button>` ima posebno značenje za React jer je to ugrađena component-a. Kod prilagođenih component-i, poput `Square`, izbor imena je na vama. Možete dodeliti bilo koje ime za prop `onSquareClick` component-e `Square` ili za funkciju `handleClick` component-e `Board`, i kod će raditi isto. U React-u je uobičajeno koristiti imena `onSomething` za props koji predstavljaju događaje, i `handleSomething` za definicije funkcija koje upravljaju tim događajima.

</Note>

### Zašto je nepromenljivost važna {/*why-immutability-is-important*/}

Obratite pažnju kako u funkciji `handleClick` koristite `.slice()` da biste kreirali kopiju array-a `squares` umesto da menjate postojeći array. Da bismo objasnili zašto je to važno, moramo razgovarati o nepromenljivosti i zašto je nepromenljivost važna za učenje.

Generalno, postoje dva pristupa promeni podataka. Prvi pristup je *menjanje* (mutiranje) podataka direktnom promenom njihovih vrednosti. Drugi pristup je zamena podataka novom kopijom koja ima željene izmene. Ovako bi izgledalo kada biste promenili array `squares` mutacijom:

```jsx
const squares = [null, null, null, null, null, null, null, null, null];
squares[0] = 'X';
// Now `squares` is ["X", null, null, null, null, null, null, null, null];
```

A ovako bi izgledalo kada biste promenili podatke bez mutiranja array-a `squares`:

```jsx
const squares = [null, null, null, null, null, null, null, null, null];
const nextSquares = ['X', null, null, null, null, null, null, null, null];
// Now `squares` is unchanged, but `nextSquares` first element is 'X' rather than `null`
```

Rezultat je isti, ali time što ne mutirate (ne menjate osnovne podatke) direktno, dobijate nekoliko prednosti.

Nepromenljivost (immutability) čini implementaciju složenih funkcionalnosti mnogo jednostavnijom. Kasnije u ovom tutorijalu implementiraćete funkcionalnost "putovanja kroz vreme" koja vam omogućava pregled istorije igre i "skok" nazad na prethodne poteze. Ova funkcionalnost nije specifična samo za igre—-mogućnost poništavanja i ponavljanja određenih akcija je čest zahtev za aplikacije. Izbegavanje direktne mutacije podataka omogućava vam da prethodne verzije podataka ostanu netaknute i da ih ponovo koristite kasnije.

Postoji još jedna prednost nepromenljivosti. Podrazumevano, sve child component-e se ponovo renderuju automatski kada se state parent component-e promeni. Ovo uključuje čak i child component-e koje nisu bile pogođene promenom. Iako ponovni renderi sami po sebi nisu primetni korisniku (i ne bi trebalo aktivno da ih izbegavate!), možda ćete želeti da preskočite ponovni render dela stabla koji očigledno nije pogođen, iz razloga performansi. Nepromenljivost čini poređenje podataka component-i vrlo jeftinim, omogućavajući da lako utvrdite da li su podaci promenjeni ili ne. Više o tome kako React odlučuje kada da ponovo renderuje component-u možete saznati u [referenci za `memo`](/reference/react/memo).

### Preduzimanje poteza {/*taking-turns*/}

Sada je vreme da popravimo veliki nedostatak ove igre iks-oks: "O" ne može biti označen na tabli.

Prvi potez će podrazumevano biti "X". Da bismo to mogli pratiti, dodaćemo još jedan state u component-u `Board`:

```js {2}
function Board() {
  const [xIsNext, setXIsNext] = useState(true);
  const [squares, setSquares] = useState(Array(9).fill(null));

  // ...
}
```

Svaki put kada igrač odigra potez, `xIsNext` (boolean) će se promeniti kako bi odredio koji igrač igra sledeći, a state igre će biti sačuvan. Update-ovaćete funkciju `handleClick` u component-i `Board` kako biste promenili vrednost `xIsNext`:

```js {7,8,9,10,11,13}
export default function Board() {
  const [xIsNext, setXIsNext] = useState(true);
  const [squares, setSquares] = useState(Array(9).fill(null));

  function handleClick(i) {
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = "X";
    } else {
      nextSquares[i] = "O";
    }
    setSquares(nextSquares);
    setXIsNext(!xIsNext);
  }

  return (
    //...
  );
}
```

Sada, kada kliknete na različite kvadrate, oni će se naizmenično ispunjavati sa `X` i `O`, kako i treba!

Ali čekajte, postoji problem. Pokušajte da kliknete na isti kvadrat više puta:

![O prepisuje X](../images/tutorial/o-replaces-x.gif)

`X` je prepisan sa `O`! Iako bi ovo moglo dodati veoma zanimljiv preokret igri, za sada ćemo se držati originalnih pravila.

Kada označite kvadrat sa `X` ili `O`, ne proveravate prvo da li kvadrat već ima vrednost `X` ili `O`. Ovo možete popraviti tako što ćete *ranije izaći* iz funkcije. Proverićete da li kvadrat već ima vrednost `X` ili `O`. Ako je kvadrat već popunjen, u funkciji `handleClick` vratićete se rano pomoću `return`—pre nego što funkcija pokuša da update-uje state table.

```js {2,3,4}
function handleClick(i) {
  if (squares[i]) {
    return;
  }
  const nextSquares = squares.slice();
  //...
}
```

Sada možete dodavati samo `X` ili `O` na prazne kvadrate! Ovako bi vaš kod trebalo da izgleda u ovom trenutku:

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({value, onSquareClick}) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

export default function Board() {
  const [xIsNext, setXIsNext] = useState(true);
  const [squares, setSquares] = useState(Array(9).fill(null));

  function handleClick(i) {
    if (squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    setSquares(nextSquares);
    setXIsNext(!xIsNext);
  }

  return (
    <>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

### Proglašavanje pobednika {/*declaring-a-winner*/}

Sada kada igrači mogu naizmenično igrati, želećete da prikažete trenutak kada je igra završena i više nema poteza za igranje. Da biste to uradili, dodaćete pomoćnu funkciju pod nazivom `calculateWinner` koja uzima array od 9 kvadrata, proverava da li postoji pobednik i vraća `'X'`, `'O'` ili `null`, u zavisnosti od rezultata. Ne brinite previše o funkciji `calculateWinner`; ona nije specifična za React:

```js src/App.js
export default function Board() {
  //...
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6]
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

<Note>

Nije važno da li definišete `calculateWinner` pre ili posle `Board` component-e. Postavićemo je na kraj, kako ne biste morali da listate kod svaki put kada menjate svoje component-e.

</Note>

Pozvaćete `calculateWinner(squares)` u funkciji `handleClick` unutar `Board` component-e kako biste proverili da li je neki igrač pobedio. Ovu proveru možete obaviti istovremeno kada proveravate da li je korisnik kliknuo na kvadrat koji već ima `X` ili `O`. Želeli bismo da se u oba slučaja funkcija završi ranije:

```js {2}
function handleClick(i) {
  if (squares[i] || calculateWinner(squares)) {
    return;
  }
  const nextSquares = squares.slice();
  //...
}
```

Da biste obavestili igrače kada je igra završena, možete prikazati tekst poput "Pobednik: X" ili "Pobednik: O". Da biste to postigli, dodaćete sekciju `status` u `Board` component-u. `Status` će prikazati pobednika ako je igra završena, a ako igra još traje, prikazaće koji igrač je sledeći na potezu:

```js {3-9,13}
export default function Board() {
  // ...
  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = "Pobednik: " + winner;
  } else {
    status = "Sledeći igrač: " + (xIsNext ? "X" : "O");
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        // ...
  )
}
```

Čestitamo! Sada imate funkcionalnu igru iks-oks. Takođe, upravo ste naučili osnove React-a. Dakle, *vi* ste ovde pravi pobednik. Evo kako vaš kod treba da izgleda:

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({value, onSquareClick}) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

export default function Board() {
  const [xIsNext, setXIsNext] = useState(true);
  const [squares, setSquares] = useState(Array(9).fill(null));

  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    setSquares(nextSquares);
    setXIsNext(!xIsNext);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = 'Pobednik: ' + winner;
  } else {
    status = 'Sledeći igrač: ' + (xIsNext ? 'X' : 'O');
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

## Dodavanje putovanja kroz vreme {/*adding-time-travel*/}

Kao završnu vežbu, omogućićemo "vraćanje u prošlost" na prethodne poteze u igri.

### Čuvanje istorije poteza {/*storing-a-history-of-moves*/}

Ako biste menjali array `squares` direktno, implementacija vremenskog putovanja bi bila veoma teška.

Međutim, koristili ste `slice()` za kreiranje nove kopije array-a `squares` nakon svakog poteza i tretirali ga kao nepromenljiv (immutable). Ovo vam omogućava da sačuvate svaku prethodnu verziju array-a `squares` i da se krećete između poteza koji su se već odigrali.

Čuvaćete prošle array-e `squares` u drugom array-u nazvanom `history`, koji ćete čuvati kao novu varijablu state-a. Array `history` predstavlja sve stanje table, od prvog do poslednjeg poteza, i ima strukturu sličnu ovoj:

```javascript
[
  // Pre prvog poteza
  [null, null, null, null, null, null, null, null, null],
  // Nakon prvog poteza
  ['X', null, null, null, null, null, null, null, null],
  // Nakon drugog poteza
  ['X', null, null, null, 'O', null, null, null, null],
  // ...
]
```jsx
[
  // Before first move
  [null, null, null, null, null, null, null, null, null],
  // After first move
  [null, null, null, null, 'X', null, null, null, null],
  // After second move
  [null, null, null, null, 'X', null, null, null, 'O'],
  // ...
]
```

## Podizanje state-a ponovo {/*lifting-state-up-again*/}

Sada ćete napisati novu component-u na najvišem nivou pod nazivom `Game`, kako biste prikazali listu prošlih poteza. Tu ćete smestiti state `history`, koji sadrži kompletnu istoriju igre.

Postavljanjem state-a `history` u component-u `Game`, možete ukloniti state `squares` iz njene child component-e `Board`. Baš kao što ste "podigli state" iz component-e `Square` u component-u `Board`, sada ćete ga podići iz component-e `Board` u component-u najvišeg nivoa, `Game`. Ovo omogućava component-i `Game` da ima potpunu kontrolu nad podacima component-e `Board` i da joj zadaje uputstva da prikaže prethodne poteze iz `history`.

Prvo, dodajte component-u `Game` koristeći `export default`. Neka renderuje component-u `Board` i malo markup-a:

```js {1,5-16}
function Board() {
  // ...
}

export default function Game() {
  return (
    <div className="game">
      <div className="game-board">
        <Board />
      </div>
      <div className="game-info">
        <ol>{/*TODO*/}</ol>
      </div>
    </div>
  );
}
```

Napomena: Uklanjate ključne reči `export default` ispred deklaracije `function Board() {` i dodajete ih ispred deklaracije `function Game() {`. Ovo govori vašem fajlu `index.js` da koristi component-u `Game` kao component-u najvišeg nivoa umesto component-e `Board`. Dodatni `div`-ovi koje vraća component-a `Game` prave prostor za informacije o igri koje ćete kasnije dodati na tablu.

Dodajte state u component-u `Game` kako biste pratili koji je sledeći igrač na potezu i istoriju poteza:

```js {2-3}
export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  // ...
```

Primetite kako je `[Array(9).fill(null)]` array sa jednim elementom, koji je sam po sebi array od 9 `null` vrednosti.

Da biste prikazali kvadrate za trenutni potez, potrebno je da pročitate poslednji array `squares` iz `history`. Za ovo vam nije potreban `useState` jer već imate dovoljno informacija da ga izračunate tokom renderovanja:

```js {4}
export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const currentSquares = history[history.length - 1];
  // ...
```

Zatim kreirajte funkciju `handlePlay` unutar component-e `Game`, koju će pozivati component-a `Board` kako bi update-ovala igru. Prosledite `xIsNext`, `currentSquares` i `handlePlay` kao props component-i `Board`:

```js {6-8,13}
export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const currentSquares = history[history.length - 1];

  function handlePlay(nextSquares) {
    // TODO
  }

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
        //...
  )
}
```

Hajde da učinimo component-u `Board` potpuno kontrolisanom preko props-a koje prima. Izmenite component-u `Board` tako da prihvata tri props-a: `xIsNext`, `squares` i novu funkciju `onPlay`, koju `Board` može pozvati sa update-ovanim array-om kvadrata kada igrač napravi potez. Zatim uklonite prve dve linije funkcije `Board` koje pozivaju `useState`:

```js {1}
function Board({ xIsNext, squares, onPlay }) {
  function handleClick(i) {
    //...
  }
  // ...
}
```

Sada zamenite pozive `setSquares` i `setXIsNext` u funkciji `handleClick` unutar component-e `Board` jednim pozivom vaše nove funkcije `onPlay`, kako bi component-a `Game` mogla da update-uje `Board` kada korisnik klikne na kvadrat:

```js {12}
function Board({ xIsNext, squares, onPlay }) {
  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = "X";
    } else {
      nextSquares[i] = "O";
    }
    onPlay(nextSquares);
  }
  //...
}
```

Component-a `Board` je potpuno kontrolisana preko props-a koje joj prosleđuje component-a `Game`. Potrebno je da implementirate funkciju `handlePlay` u component-i `Game` kako bi igra ponovo funkcionisala.

Šta bi funkcija `handlePlay` trebalo da uradi kada se pozove? Zapamtite da je `Board` ranije pozivao `setSquares` sa update-ovanim array-om; sada prosleđuje update=ovani array `squares` funkciji `onPlay`.

Funkcija `handlePlay` treba da update-uje state component-e `Game` kako bi pokrenula ponovno renderovanje, ali više nemate funkciju `setSquares` koju biste mogli da pozovete—sada koristite varijablu state-a `history` za čuvanje ovih informacija. Treba da update-ujete `history` tako što ćete dodati update-ovani array `squares` kao novi unos u istoriji. Takođe treba da promenite vrednost `xIsNext`, kao što je `Board` ranije radio:

```js {4-5}
export default function Game() {
  //...
  function handlePlay(nextSquares) {
    setHistory([...history, nextSquares]);
    setXIsNext(!xIsNext);
  }
  //...
}
```

Ovde, `[...history, nextSquares]` kreira novi array koji sadrži sve stavke iz `history`, praćene array-om `nextSquares`. (Možete čitati `...history` [*spread sintaksu*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) kao „nabroj sve stavke u `history`”.)

Na primer, ako je `history` `[[null,null,null], ["X",null,null]]`, a `nextSquares` je `["X",null,"O"]`, novi array `[...history, nextSquares]` biće `[[null,null,null], ["X",null,null], ["X",null,"O"]]`.

U ovom trenutku, state je premešten u component-u `Game`, i korisnički interfejs bi trebalo da funkcioniše potpuno isto kao i pre refaktorisanja. Evo kako kod treba da izgleda na ovoj tački:

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({ value, onSquareClick }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

function Board({ xIsNext, squares, onPlay }) {
  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    onPlay(nextSquares);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = 'Pobednik: ' + winner;
  } else {
    status = 'Sledeći igrač: ' + (xIsNext ? 'X' : 'O');
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const currentSquares = history[history.length - 1];

  function handlePlay(nextSquares) {
    setHistory([...history, nextSquares]);
    setXIsNext(!xIsNext);
  }

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
      </div>
      <div className="game-info">
        <ol>{/*TODO*/}</ol>
      </div>
    </div>
  );
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

### Prikazivanje prethodnih poteza {/*showing-the-past-moves*/}

Pošto beležite istoriju igre iks-oks, sada možete prikazati listu prethodnih poteza igraču.

React elementi poput `<button>` su regularni JavaScript objekti; možete ih prosleđivati kroz svoju aplikaciju. Da biste prikazali više stavki u React-u, možete koristiti array React elemenata.

Već imate array `history` poteza u state-u, pa sada treba da ga transformišete u array React elemenata. U JavaScript-u, za transformaciju jednog array-a u drugi, možete koristiti [metodu array `map:`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)

```jsx
[1, 2, 3].map((x) => x * 2) // [2, 4, 6]
```

Koristićete `map` da transformišete svoj `history` poteza u React elemente koji predstavljaju dugmad na ekranu, i prikazati listu dugmadi za „skok” na prethodne poteze. Hajde da primenimo `map` na `history` u component-i Game:

```js {11-13,15-27,35}
export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const currentSquares = history[history.length - 1];

  function handlePlay(nextSquares) {
    setHistory([...history, nextSquares]);
    setXIsNext(!xIsNext);
  }

  function jumpTo(nextMove) {
    // TODO
  }

  const moves = history.map((squares, move) => {
    let description;
    if (move > 0) {
      description = 'Prebacite se na potez #' + move;
    } else {
      description = 'Prebacite se na početak';
    }
    return (
      <li>
        <button onClick={() => jumpTo(move)}>{description}</button>
      </li>
    );
  });

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
      </div>
      <div className="game-info">
        <ol>{moves}</ol>
      </div>
    </div>
  );
}
```

Možete videti kako vaš kod treba da izgleda ispod. Imajte na umu da biste u konzoli trebalo da vidite grešku koja kaže:

<ConsoleBlock level="warning">
Warning: Each child in an array or iterator should have a unique "key" prop. Check the render method of &#96;Game&#96;.
</ConsoleBlock>
  
Ovu grešku ćete rešiti u sledećem odeljku.

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({ value, onSquareClick }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

function Board({ xIsNext, squares, onPlay }) {
  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    onPlay(nextSquares);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = 'Pobednik: ' + winner;
  } else {
    status = 'Sledeći igrač: ' + (xIsNext ? 'X' : 'O');
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const currentSquares = history[history.length - 1];

  function handlePlay(nextSquares) {
    setHistory([...history, nextSquares]);
    setXIsNext(!xIsNext);
  }

  function jumpTo(nextMove) {
    // TODO
  }

  const moves = history.map((squares, move) => {
    let description;
    if (move > 0) {
      description = 'Prebacite se na potez #' + move;
    } else {
      description = 'Prebacite se na početak';
    }
    return (
      <li>
        <button onClick={() => jumpTo(move)}>{description}</button>
      </li>
    );
  });

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
      </div>
      <div className="game-info">
        <ol>{moves}</ol>
      </div>
    </div>
  );
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}

.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

Dok iterirate kroz array `history` unutar funkcije koju ste prosledili `map`, argument `squares` prolazi kroz svaki element array-a `history`, a argument `move` prolazi kroz svaki indeks array-a: `0`, `1`, `2`, …. (U većini slučajeva trebaju vam stvarni elementi array-a, ali za prikaz liste poteza biće vam potrebni samo indeksi.)

Za svaki potez u istoriji igre iks-oks, kreirate stavku liste `<li>` koja sadrži dugme `<button>`. Dugme ima `onClick` handler koji poziva funkciju `jumpTo` (koju još niste implementirali).

Za sada, trebalo bi da vidite listu poteza koji su se dogodili u igri i grešku u konzoli. Hajde da razgovaramo o značenju greške „key”.

### Odabir ključa {/*picking-a-key*/}

Kada renderujete listu, React čuva neke informacije o svakoj renderovanoj stavci liste. Kada update-ujete listu, React treba da odredi šta se promenilo. Mogli ste da dodate, uklonite, preuredite ili update-ujete stavke liste.

Zamislite prelazak sa:

```html
<li>Alexa: 7 zadataka preostalo</li>
<li>Ben: 5 zadataka preostalo</li>
```

to

```html
<li>Ben: 9 zadataka preostalo</li>
<li>Claudia: 8 zadataka preostalo</li>
<li>Alexa: 5 zadataka preostalo</li>
```

Pored update-ovanih brojeva, osoba koja čita ovo verovatno bi rekla da ste zamenili redosled Alexe i Bena i umetnuli Claudiu između Alexe i Bena. Međutim, React je računarski program i ne zna šta ste želeli da uradite, pa je potrebno da za svaku stavku liste navedete svojstvo *key* kako biste razlikovali svaku stavku od njenih „sestrinskih” stavki. Ako su vaši podaci iz baze podataka, ID-ovi Alexe, Bena i Claudie iz te baze mogli bi da se koriste kao ključevi.

```js {1}
<li key={user.id}>
  {user.name}: {user.taskCount} zadataka preostalo
</li>
```

Kada se lista ponovo renderuje, React uzima ključ svake stavke liste i pretražuje stavke prethodne liste tražeći ključ koji se poklapa. Ako trenutna lista ima ključ koji nije postojao ranije, React kreira component-u. Ako trenutnoj listi nedostaje ključ koji je postojao u prethodnoj listi, React poništava prethodnu component-u. Ako se dva ključa poklapaju, odgovarajuća component-a se premesti.

Ključevi govore React-u o identitetu svake component-e, što omogućava React-u da očuva state između ponovnih renderovanja. Ako se ključ component-e promeni, component-a će biti poništena i ponovo kreirana sa novim state-om.

`key` je specijalan i rezervisan property u React-u. Kada se element kreira, React izvlači property `key` i čuva ga direktno na vraćenom elementu. Iako `key` može izgledati kao da se prosleđuje kao props, React automatski koristi `key` da odluči koje component-e da update-uje. Ne postoji način da component-a pita koji je `key` njen parent odredio.

**Snažno se preporučuje da dodelite odgovarajuće ključeve svaki put kada pravite dinamičke liste.** Ako nemate odgovarajući ključ, možda bi trebalo da razmislite o restrukturiranju vaših podataka kako biste ga obezbedili.

Ako nije naveden ključ, React će prijaviti grešku i koristiti indeks array-a kao ključ podrazumevano. Korišćenje indeksa array-a kao ključa je problematično kada pokušavate da promenite redosled stavki liste ili da umetnete/uklonite stavke liste. Eksplicitno prosleđivanje `key={i}` utišava grešku, ali ima iste probleme kao i korišćenje indeksa array-a i ne preporučuje se u većini slučajeva.

Ključevi ne moraju biti globalno jedinstveni; moraju biti jedinstveni samo između component-i i njihovih „sestrinskih” component-i.

### Implementacija putovanja kroz vreme {/*implementing-time-travel*/}

U istoriji igre iks-oks, svaki prethodni potez ima jedinstveni ID povezan sa njim: to je redni broj poteza. Potezi se nikada neće preuređivati, brisati ili umetati u sredinu, pa je bezbedno koristiti indeks poteza kao ključ.

U funkciji `Game`, možete dodati ključ kao `<li key={move}>`, i ako ponovo učitate renderovanu igru, React-ova greška „key” bi trebalo da nestane:

```js {4}
const moves = history.map((squares, move) => {
  //...
  return (
    <li key={move}>
      <button onClick={() => jumpTo(move)}>{description}</button>
    </li>
  );
});
```

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({ value, onSquareClick }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

function Board({ xIsNext, squares, onPlay }) {
  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    onPlay(nextSquares);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = 'Pobednik: ' + winner;
  } else {
    status = 'Sledeći igrač: ' + (xIsNext ? 'X' : 'O');
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const currentSquares = history[history.length - 1];

  function handlePlay(nextSquares) {
    setHistory([...history, nextSquares]);
    setXIsNext(!xIsNext);
  }

  function jumpTo(nextMove) {
    // TODO
  }

  const moves = history.map((squares, move) => {
    let description;
    if (move > 0) {
      description = 'Prebacite se na potez #' + move;
    } else {
      description = 'Prebacite se na početak';
    }
    return (
      <li key={move}>
        <button onClick={() => jumpTo(move)}>{description}</button>
      </li>
    );
  });

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
      </div>
      <div className="game-info">
        <ol>{moves}</ol>
      </div>
    </div>
  );
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}

```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}

.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

Pre nego što implementirate `jumpTo`, potrebno je da component-a `Game` prati koji korak korisnik trenutno gleda. Da biste to uradili, definišite novu state varijablu pod nazivom `currentMove`, sa podrazumevanom vrednošću `0`:

```js {4}
export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const [currentMove, setCurrentMove] = useState(0);
  const currentSquares = history[history.length - 1];
  //...
}
```

Zatim, update-ujte funkciju `jumpTo` unutar component-e `Game` tako da update-uje `currentMove`. Takođe ćete postaviti `xIsNext` na `true` ako je broj na koji menjate `currentMove` paran.

```js {4-5}
export default function Game() {
  // ...
  function jumpTo(nextMove) {
    setCurrentMove(nextMove);
    setXIsNext(nextMove % 2 === 0);
  }
  //...
}
```

Sada ćete napraviti dve izmene u funkciji `handlePlay` component-e `Game`, koja se poziva kada kliknete na kvadrat.

- Ako se "vratite u prošlost" i napravite novi potez od te tačke, želite da zadržite istoriju samo do te tačke. Umesto da dodate `nextSquares` posle svih stavki (`...` spread sintaksa) u `history`, dodaćete ga posle svih stavki u `history.slice(0, currentMove + 1)` kako biste zadržali samo taj deo stare istorije.
- Svaki put kada se napravi potez, potrebno je da update-ujete `currentMove` kako bi ukazivao na najnoviji unos u istoriji.

```js {2-4}
function handlePlay(nextSquares) {
  const nextHistory = [...history.slice(0, currentMove + 1), nextSquares];
  setHistory(nextHistory);
  setCurrentMove(nextHistory.length - 1);
  setXIsNext(!xIsNext);
}
```

Na kraju, izmenićete component-u `Game` tako da renderuje trenutno odabrani potez, umesto da uvek renderuje poslednji potez:

```js {5}
export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const [currentMove, setCurrentMove] = useState(0);
  const currentSquares = history[currentMove];

  // ...
}
```

Ako kliknete na bilo koji korak u istoriji igre, tabla za iks-oks bi trebalo odmah da se update-uje i prikaže kako je izgledala nakon tog poteza.

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({value, onSquareClick}) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

function Board({ xIsNext, squares, onPlay }) {
  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    onPlay(nextSquares);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = 'Pobednik: ' + winner;
  } else {
    status = 'Sledeći igrač: ' + (xIsNext ? 'X' : 'O');
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

export default function Game() {
  const [xIsNext, setXIsNext] = useState(true);
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const [currentMove, setCurrentMove] = useState(0);
  const currentSquares = history[currentMove];

  function handlePlay(nextSquares) {
    const nextHistory = [...history.slice(0, currentMove + 1), nextSquares];
    setHistory(nextHistory);
    setCurrentMove(nextHistory.length - 1);
    setXIsNext(!xIsNext);
  }

  function jumpTo(nextMove) {
    setCurrentMove(nextMove);
    setXIsNext(nextMove % 2 === 0);
  }

  const moves = history.map((squares, move) => {
    let description;
    if (move > 0) {
      description = 'Prebacite se na potez #' + move;
    } else {
      description = 'Prebacite se na početak';
    }
    return (
      <li key={move}>
        <button onClick={() => jumpTo(move)}>{description}</button>
      </li>
    );
  });

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
      </div>
      <div className="game-info">
        <ol>{moves}</ol>
      </div>
    </div>
  );
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

### Finalno sređivanje {/*final-cleanup*/}

Ako pažljivo pogledate kod, primetićete da je `xIsNext === true` kada je `currentMove` paran i `xIsNext === false` kada je `currentMove` neparan. Drugim rečima, ako znate vrednost `currentMove`, uvek možete odrediti šta treba da bude `xIsNext`.

Nema potrebe da čuvate obe ove vrednosti u state-u. Zapravo, uvek pokušajte da izbegavate redundantni state. Pojednostavljivanje onoga što čuvate u state-u smanjuje broj grešaka i čini vaš kod lakšim za razumevanje. Izmenite component-u `Game` tako da ne čuva `xIsNext` kao zasebnu state varijablu, već da ga izračunava na osnovu vrednosti `currentMove`:

```js {4,11,15}
export default function Game() {
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const [currentMove, setCurrentMove] = useState(0);
  const xIsNext = currentMove % 2 === 0;
  const currentSquares = history[currentMove];

  function handlePlay(nextSquares) {
    const nextHistory = [...history.slice(0, currentMove + 1), nextSquares];
    setHistory(nextHistory);
    setCurrentMove(nextHistory.length - 1);
  }

  function jumpTo(nextMove) {
    setCurrentMove(nextMove);
  }
  // ...
}
```

Više vam nije potrebna deklaracija state varijable `xIsNext` niti pozivi funkcije `setXIsNext`. Sada ne postoji mogućnost da `xIsNext` bude van sinhronizacije sa `currentMove`, čak i ako napravite grešku dok kodirate component-e.

### Završetak {/*wrapping-up*/}

Čestitamo! Napravili ste igru iks-oks koja:

- Omogućava igranje iks-oks,
- Indikuje kada je igrač pobedio u igri,
- Čuva istoriju igre kako se igra odvija,
- Omogućava igračima da pregledaju istoriju igre i vide prethodne verzije table.

Odličan posao! Nadamo se da sada imate solidno razumevanje kako React funkcioniše.

Pogledajte konačni rezultat ovde:

<Sandpack>

```js src/App.js
import { useState } from 'react';

function Square({ value, onSquareClick }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

function Board({ xIsNext, squares, onPlay }) {
  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    onPlay(nextSquares);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = 'Pobednik: ' + winner;
  } else {
    status = 'Sledeći igrač: ' + (xIsNext ? 'X' : 'O');
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

export default function Game() {
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const [currentMove, setCurrentMove] = useState(0);
  const xIsNext = currentMove % 2 === 0;
  const currentSquares = history[currentMove];

  function handlePlay(nextSquares) {
    const nextHistory = [...history.slice(0, currentMove + 1), nextSquares];
    setHistory(nextHistory);
    setCurrentMove(nextHistory.length - 1);
  }

  function jumpTo(nextMove) {
    setCurrentMove(nextMove);
  }

  const moves = history.map((squares, move) => {
    let description;
    if (move > 0) {
      description = 'Prebacite se na potez #' + move;
    } else {
      description = 'Prebacite se na početak';
    }
    return (
      <li key={move}>
        <button onClick={() => jumpTo(move)}>{description}</button>
      </li>
    );
  });

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
      </div>
      <div className="game-info">
        <ol>{moves}</ol>
      </div>
    </div>
  );
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

```css src/styles.css
* {
  box-sizing: border-box;
}

body {
  font-family: sans-serif;
  margin: 20px;
  padding: 0;
}

.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}

.board-row:after {
  clear: both;
  content: '';
  display: table;
}

.status {
  margin-bottom: 10px;
}
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

</Sandpack>

Ako imate dodatno vreme ili želite da vežbate svoje nove React veštine, evo nekoliko ideja za unapređenja igre iks-oks, poređanih po rastućem nivou težine:

1. Samo za trenutni potez prikažite poruku "Nalazite se na potezu #..." umesto dugmeta.
2. Prepišite component-u `Board` da koristi dve petlje za kreiranje kvadrata umesto da ih hardkodirate.
3. Dodajte dugme za prebacivanje koje omogućava sortiranje poteza u rastućem ili opadajućem redosledu.
4. Kada neko pobedi, istaknite tri kvadrata koja su dovela do pobede (a kada niko ne pobedi, prikažite poruku o nerešenom rezultatu).
5. Prikazujte lokaciju svakog poteza u formatu (red, kolona) na listi istorije poteza.

Kroz ovaj tutorijal ste obradili React koncepte uključujući elemente, component-e, props i state. Sada kada ste videli kako ovi koncepti funkcionišu pri izradi igre, pogledajte [Razmišljanje u React-u](/learn/thinking-in-react) da biste videli kako isti React koncepti funkcionišu pri izradi korisničkog interfejsa aplikacije.
