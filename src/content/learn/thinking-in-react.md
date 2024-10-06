---
title: Razmišljanje u React-u
---

<Intro>

React vam može promeniti način razmišljanja o dizajnu, kao i o aplikaciji koju pravite. Kada gradite korisnički interfejs (UI) pomoću React-a, prvo ćete ga podeliti na delove koji se zovu *komponente*. Nakon toga, opisaćete različita vizuelna stanja za svaku od vaših komponenti. Na kraju, povezaćete vaše komponente kako bi podaci prolazili kroz njih. U ovom tutorijalu ćemo vas voditi kroz proces razmišljanja o pravljenju tabele sa proizvodima koja može da se pretražuje pomoću React-a.

</Intro>

## Počnite sa mockup-om {/*start-with-the-mockup*/}

Zamislite da već imate JSON API i mockup od dizajnera.

JSON API vraća podatke koji ovako izgledaju:


```json
[
  { category: "Voće", price: "$1", stocked: true, name: "Jabuka" },
  { category: "Voće", price: "$1", stocked: true, name: "Zmajevo voće" },
  { category: "Voće", price: "$2", stocked: false, name: "Marakuja" },
  { category: "Povrće", price: "$2", stocked: true, name: "Španat" },
  { category: "Povrće", price: "$4", stocked: false, name: "Bundeva" },
  { category: "Povrće", price: "$1", stocked: true, name: "Grašak" }
]
```

Mockup izgleda ovako:

<img src="/images/docs/s_thinking-in-react_ui.png" width="300" style={{margin: '0 auto'}} />

Da biste implementirali UI u React-u, uglavnom ćete pratiti istih pet koraka.

## Korak 1: Podeliti UI u hijerarhiju komponenata {/*step-1-break-the-ui-into-a-component-hierarchy*/}

Započnite crtanjem granica oko svake komponente i subkomponente na mockup-u i pravilno ih imenujte. Ako radite sa dizajnerom, on je možda već imenovao komponente u svom alatu. Pitajte ga!

U zavisnosti od vašeg znanja, možete razmišljati o podeli dizajna na komponente na više načina:

* **Programiranje**--koristite iste tehnike odlučivanja kao da trebate kreirati novu funkciju ili objekat. Jedna takva tehnika je [princip jedne odgovornosti (single responsibility principle)](https://en.wikipedia.org/wiki/Single_responsibility_principle), što znači da bi komponenta idealno trebala raditi samo jednu stvar. Ako se komponenta povećava, trebalo bi je rasparčati na manje subkomponente.
* **CSS**--razmotrite za šta biste kreirali class selector-e. (Ipak, komponente su malo granularnije.)
* **Dizajn**--razmotrite kako biste organizovali slojeve dizajna.

Ako je vaš JSON dobro struktuiran, često ćete primetiti da se prirodno mapira na strukturu komponenti na vašem UI-u. To je zato što UI i modeli podataka često imaju istu arhitekturu informacija, odnosno isti oblik. Podelite UI na komponente tako da svaka komponenta odgovara jednom delu modela podataka.

Na ovom ekranu imamo pet komponenti:

<FullWidth>

<CodeDiagram flip>

<img src="/images/docs/s_thinking-in-react_ui_outline.png" width="500" style={{margin: '0 auto'}} />

1. `FilterableProductTable` (sivo) sadrži celu aplikaciju.
2. `SearchBar` (plavo) prima koristički input.
3. `ProductTable` (ljubičasto) prikazuje i filtrira listu na osnovu korisničkog input-a.
4. `ProductCategoryRow` (zeleno) prikazuje naslov za svaku kategoriju.
5. `ProductRow`	(žuto) prikazuje red za svaki proizvod.

</CodeDiagram>

</FullWidth>

Ako pogledate `ProductTable` (ljubičasto), videćete da naslov tabele (sadrži "Name" i "Price" labele) nije zasebna komponenta. To je stvar preferenci, tako da možete uraditi na oba načina. U ovom primeru on je deo `ProductTable`-a, jer se nalazi unutar `ProductTable` liste. Međutim, ako se ovaj naslov zakomplikuje (npr. ako dodate sortiranje), možete ga izdvojiti u `ProductTableHeader` komponentu.

Kada ste identifikovali komponente na mockup-u, organizujte ih u hijerarhiju. Komponente koje se u mockup-u nalaze unutar druge komponente trebalo bi da budu deca (children) u hijerarhiji:

* `FilterableProductTable`
    * `SearchBar`
    * `ProductTable`
        * `ProductCategoryRow`
        * `ProductRow`

## Korak 2: Napraviti statičku verziju u React-u {/*step-2-build-a-static-version-in-react*/}

Pošto imate hijerarhiju komponenata, vreme je da implementirate vašu aplikaciju. Najbolji pristup je da napravite verziju koja renderuje UI na osnovu modela podataka bez dodavanja interaktivnosti... još uvek! Često je lakše da se prvo napravi statička verzija, a da se interaktivnost doda naknadno. Pravljenje statičke verzije zahteva dosta kucanja bez mozganja, dok dodavanje interaktivnosti zahteva dosta mozganja i ne tako mnogo kucanja.

Da biste napravili statičku verziju aplikacije koja renderuje vaš model podataka, želećete da napravite [komponente](/learn/your-first-component) koje koriste druge komponente i šalju podatke kroz [props](/learn/passing-props-to-a-component). Props je način prosleđivanja podataka od roditelja (parent) ka detetu (child). (Ako ste upoznati sa konceptom [state](/learn/state-a-components-memory)-a, uopšte ga nemojte koristiti za pravljenje ove statičke verzije. State je rezervisan samo za interaktivnost, odnosno podatke koji se menjaju vremenom. Pošto je ovo statička verzija aplikacije, state vam neće trebati.)

Možete graditi aplikaciju "od gore ka dole", pravljenjem prvo komponenti koje su više u hijerarhiji (npr. `FilterableProductTable`) ili "od dole ka gore", pravljenjem komponenti koje su na dnu hijerarhije (npr. `ProductRow`). U jednostavnijim primerima, često je lakše koristiti "od gore ka dole", dok je na većim projektima lakše koristiti "od dole ka gore".

<Sandpack>

```jsx src/App.js
function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Naziv</th>
          <th>Cena</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar() {
  return (
    <form>
      <input type="text" placeholder="Pretraži..." />
      <label>
        <input type="checkbox" />
        {' '}
        Prikaži samo proizvode na stanju
      </label>
    </form>
  );
}

function FilterableProductTable({ products }) {
  return (
    <div>
      <SearchBar />
      <ProductTable products={products} />
    </div>
  );
}

const PRODUCTS = [
  {category: "Voće", price: "$1", stocked: true, name: "Jabuka"},
  {category: "Voće", price: "$1", stocked: true, name: "Zmajevo voće"},
  {category: "Voće", price: "$2", stocked: false, name: "Marakuja"},
  {category: "Povrće", price: "$2", stocked: true, name: "Španat"},
  {category: "Povrće", price: "$4", stocked: false, name: "Bundeva"},
  {category: "Povrće", price: "$1", stocked: true, name: "Grašak"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding-top: 10px;
}
td {
  padding: 2px;
  padding-right: 40px;
}
```

</Sandpack>

(Ako vam ovaj kod deluje zastrašujuće, prvo prođite kroz [Brzi Uvod](/learn/)!)

Nakon što napravite komponente, imaćete biblioteku reusable komponenata koje renderuju model podataka. Pošto je ovo statička aplikacija, komponente će vraćati samo JSX. Komponenta na vrhu hijerarhije (`FilterableProductTable`) uzeće vaš model podataka kao prop. To se zove _jednosmerni data flow_ zato što se podaci prosleđuju od komponente na vrhu ka komponentama na dnu stabla.

<Pitfall>

Ne biste trebali da koristite nijednu state vrednost u ovom trenutku. To je naredni korak!

</Pitfall>

## Korak 3: Pronaći minimalan, ali kompletan prikaz UI state-a {/*step-3-find-the-minimal-but-complete-representation-of-ui-state*/}

Da bi vaš UI bio interaktivan, neophodno je dopustiti korisnicima da menjaju model podataka. Za to ćete upotrebiti *state*.

State bi trebao biti najmanji skup promenljivih podataka koji su neophodni vašoj aplikaciji. Najbitnija stavka za definisanje state-a je da poštujete [DRY (Don't Repeat Yourself)](https://sr.wikipedia.org/wiki/Don%27t_repeat_yourself) princip. Pokušajte da shvatite apsolutno minimalan state koji je potreban vašoj aplikaciji, a sve ostalo računajte po potrebi. Na primer, ako pravite spisak za kupovinu, možete držati listu proizvoda kao state. Ako želite da prikažete i broj proizvoda u listi, ne morate čuvati broj proizvoda u state-u, već ga možete izračunati kao dužinu liste.

Sada razmislite o svim podacima u našoj aplikaciji:

1. Originalna lista proizvoda
2. Tekst za pretragu koji je uneo korisnik
3. Vrednost checkbox-a
4. Filtrirana lista proizvoda

Šta je od ovoga state? Identifikujte ono što nije:

* Da li **ostaje nepromenjeno** tokom vremena? Ako je tako, to nije state.
* Da li je **prosleđeno od roditelja** kao props? Ako je tako, to nije state.
* **Da li možete izračunati** na osnovu state-a ili props-a unutar komponente? Ako je tako, to *definitivno* nije state!

Ono što ostaje je verovatno state.

Prođimo kroz podatke još jednom:

1. Originalna lista proizvoda je **prosleđena kao props, tako da nije state**.
2. Tekst za pretragu deluje kao state zato što se vremenom menja i ne može biti izračunat.
3. Vrednost checkbox-a deluje kao state zato što se vremenom menja i ne može biti izračunata.
4. Filtrirana lista proizvoda **nije state zato što može biti izračunata** filtriranjem originalne liste proizvoda pomoću teksta za pretragu i vrednosti checkbox-a.

Ovo znači da su samo tekst za pretragu i vrednost checkbox-a state-ovi! Dobar posao!

<DeepDive>

#### Props vs State {/*props-vs-state*/}

Postoje dva tipa "modela" podataka u React-u: props i state. Veoma su različiti:

* [**Props** je poput argumenata koje prosledite](/learn/passing-props-to-a-component) funkciji. Oni omogućavaju roditeljskoj (parent) komponenti da prosledi podatke detetu (child) komponenti i izmeni joj izgled. Na primer, `Form` može proslediti `color` prop `Button` komponenti.
* [**State** je nalik na memoriju komponente](/learn/state-a-components-memory). Omogućava komponenti da prati neku informaciju i menja je u zavisnosti od interakcije. Na primer, `Button` može pratiti `isHovered` state.

Props i state su različiti, ali rade zajedno. Roditeljska (parent) komponenta će često čuvati neku informaciju kao state (kako bi mogla da je menja), i *proslediće je* deci (children) komponentama kao njihov props. U redu je da vam je ova razlika nejasna na prvo čitanje. Potrebno je malo iskustva kako biste je savladali!

</DeepDive>

## Korak 4: Identifikovati gde state treba da živi {/*step-4-identify-where-your-state-should-live*/}

Nakon identifikacije minimalnog skupa podataka za state-ove, potrebno je da identifikujete koja komponenta je zadužena za koji state, tj. koja komponenta *poseduje* state. Upamtite: React koristi jednosmerni data flow gde se podaci hijerarhijski prosleđuju od roditelja ka detetu. Možda vam ne bude odmah očigledno koja komponenta poseduje koji state. To može biti izazovno ako se tek upoznajete sa ovim konceptom, ali shvatićete ako pratite naredne korake!

Za svaki state u vašoj aplikaciji:

1. Identifikujte *svaku* komponentu koja renderuje nešto na osnovu state-a.
2. Nađite najbližu roditeljsku komponentu koja iz sve sadrži--komponenta koja je iznad svih njih u hijerarhiji.
3. Odlučite gde state treba da živi:
    1. Najčešće state možete ubaciti direktno u zajedničkog roditelja.
    2. Takođe, state možete ubaciti u neku komponentu iznad zajedničkog roditelja.
    3. Ako ne možete pronaći komponentu u kojoj ima smisla čuvati state, kreirajte novu komponentu isključivo za čuvanje state-a i dodajte je negde u hijerarhiju iznad zajedničke roditeljske komponente.

U prethodnom koraku, pronašli ste dva state-a u aplikaciji: tekst za pretragu i vrednost checkbox-a. U ovom primeru se oba pojavljuju zajedno, tako da ima smisla staviti ih na isto mesto.

Primenimo našu strategiju za njih:

1. **Identifikovati komponente koje koriste state:**
    * `ProductTable` treba da filtrira listu proizvoda na osnovu state-a (tekst za pretragu i vrednost checkbox-a). 
    * `SearchBar` treba da prikaže state (tekst za pretragu i vrednost checkbox-a).
2. **Pronaći zajedničkog roditelja:** Prva komponenta koja sadrži obe komponente je `FilterableProductTable`.
3. **Odlučiti gde živi state:** Držaćemo tekst za pretragu i vrednost checkbox-a u `FilterableProductTable`.

Znači, state vrednosti će živeti u `FilterableProductTable`.

Dodajte state u komponentu pomoću [`useState()` Hook](/reference/react/useState)-a. Hook-ovi su posebne funkcije koje vam omogućavaju da "se zakačite" za React. Dodajte dve state promenljive na vrh `FilterableProductTable` i zadajte njihov početni state:

```js
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);  
```

Zatim, prosledite `filterText` i `inStockOnly` u `ProductTable` i `SearchBar` kao props:

```js
<div>
  <SearchBar 
    filterText={filterText} 
    inStockOnly={inStockOnly} />
  <ProductTable 
    products={products}
    filterText={filterText}
    inStockOnly={inStockOnly} />
</div>
```

Gledajte kako će se vaša aplikacija ponašati. Promenite početnu vrednost `filterText`-a sa `useState('')` na `useState('voće')` u sandbox-u ispod. Videćete da su se tekst za pretragu i tabela promenili:

<Sandpack>

```jsx src/App.js
import { useState } from 'react';

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly} />
      <ProductTable 
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Naziv</th>
          <th>Cena</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input 
        type="text" 
        value={filterText} 
        placeholder="Pretraži..."/>
      <label>
        <input 
          type="checkbox" 
          checked={inStockOnly} />
        {' '}
        Prikaži samo proizvode na stanju
      </label>
    </form>
  );
}

const PRODUCTS = [
  {category: "Voće", price: "$1", stocked: true, name: "Jabuka"},
  {category: "Voće", price: "$1", stocked: true, name: "Zmajevo voće"},
  {category: "Voće", price: "$2", stocked: false, name: "Marakuja"},
  {category: "Povrće", price: "$2", stocked: true, name: "Španat"},
  {category: "Povrće", price: "$4", stocked: false, name: "Bundeva"},
  {category: "Povrće", price: "$1", stocked: true, name: "Grašak"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding-top: 5px;
}
td {
  padding: 2px;
}
```

</Sandpack>

Primetićete da promena forme još uvek ne radi. Pojavila se greška u konzoli u sandbox-u iznad koji objašnjava zašto:

<ConsoleBlock level="error">

You provided a \`value\` prop to a form field without an \`onChange\` handler. This will render a read-only field.

</ConsoleBlock>

<ConsoleBlock level="error">

Prosledili ste \`value\` prop u polje forme bez \`onChange\` handler-a. To će renderovati read-only polje.

</ConsoleBlock>

U sandbox-u iznad, `ProductTable` i `SearchBar` čitaju `filterText` i `inStockOnly` props da bi renderovali tabelu, tekst za pretragu i checkbox. Na primer, ovako `SearchBar` konstruiše vrednost teksta za pretragu:

```js {1,6}
function SearchBar({ filterText, inStockOnly }) {
  return (
    <form>
      <input 
        type="text" 
        value={filterText} 
        placeholder="Pretraži..."/>
```

Međutim, još uvek niste dodali kod koji bi odgovarao na korisničke akcije poput kucanja. To će biti naš poslednji korak.


## Korak 5: Dodati inverzni data flow {/*step-5-add-inverse-data-flow*/}

Trenutno, vaša aplikacija korektno renderuje props-e i state-ove koje komponente dobijaju od roditelja. Ali, da biste promenili state na osnovu korisničkog input-a, morate podržati data flow na drugačiji način: komponente duboko u hijerarhiji trebaju promeniti state u `FilterableProductTable`.

React eksplicitno podržava ovakav data flow, ali je potrebno malo više od dvosmernog data binding-a. Ako pokušate da kucate ili promenite vrednost checkbox-a u primeru iznad, videćete da React ignoriše vaš input. To je namerno urađeno. Kada napišete `<input value={filterText} />`, setovaćete `value` prop `input`-a da uvek bude jednak `filterText` state-u prosleđenom iz `FilterableProductTable`. Pošto se `filterText` state nikad ne menja, ni input se ne menja.

Želite da kad god korisnik promeni input u formi, da se state takođe promeni. Pošto se state nalazi u `FilterableProductTable`, samo tamo možete pozvati funkcije `setFilterText` i `setInStockOnly`. Da biste omogućili `SearchBar`-u da promeni state iz `FilterableProductTable`, te funkcije morate proslediti u `SearchBar`:

```js {2,3,10,11}
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly} />
```

Unutar `SearchBar`-a dodaćete `onChange` event handler-e i setovaćete state roditelja u njima:

```js {4,5,13,19}
function SearchBar({
  filterText,
  inStockOnly,
  onFilterTextChange,
  onInStockOnlyChange
}) {
  return (
    <form>
      <input
        type="text"
        value={filterText}
        placeholder="Pretraži..."
        onChange={(e) => onFilterTextChange(e.target.value)}
      />
      <label>
        <input
          type="checkbox"
          checked={inStockOnly}
          onChange={(e) => onInStockOnlyChange(e.target.checked)}
```

Sada vaša aplikacija radi u potpunosti!

<Sandpack>

```jsx src/App.js
import { useState } from 'react';

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly} 
        onFilterTextChange={setFilterText} 
        onInStockOnlyChange={setInStockOnly} />
      <ProductTable 
        products={products} 
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}

function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}

function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Naziv</th>
          <th>Cena</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}

function SearchBar({
  filterText,
  inStockOnly,
  onFilterTextChange,
  onInStockOnlyChange
}) {
  return (
    <form>
      <input 
        type="text" 
        value={filterText} placeholder="Pretraži..." 
        onChange={(e) => onFilterTextChange(e.target.value)} />
      <label>
        <input 
          type="checkbox" 
          checked={inStockOnly} 
          onChange={(e) => onInStockOnlyChange(e.target.checked)} />
        {' '}
        Prikaži samo proizvode na stanju
      </label>
    </form>
  );
}

const PRODUCTS = [
  {category: "Voće", price: "$1", stocked: true, name: "Jabuka"},
  {category: "Voće", price: "$1", stocked: true, name: "Zmajevo voće"},
  {category: "Voće", price: "$2", stocked: false, name: "Marakuja"},
  {category: "Povrće", price: "$2", stocked: true, name: "Španat"},
  {category: "Povrće", price: "$4", stocked: false, name: "Bundeva"},
  {category: "Povrće", price: "$1", stocked: true, name: "Grašak"}
];

export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}
```

```css
body {
  padding: 5px
}
label {
  display: block;
  margin-top: 5px;
  margin-bottom: 5px;
}
th {
  padding: 4px;
}
td {
  padding: 2px;
}
```

</Sandpack>

Možete naučiti sve o event handling-u i promeni state-a u sekciji [Dodavanje interaktivnosti](/learn/adding-interactivity).

## Gde ići nakon ovoga {/*where-to-go-from-here*/}

Ovo je bio veoma kratak uvod o tome kako da razmišljate o pravljenju komponenata i aplikacija pomoću React-a. Možete [započeti React projekat](/learn/installation) odmah ili [zaroniti dublje u sintaksu](/learn/describing-the-ui) upotrebljenu u ovom tutorijalu.
