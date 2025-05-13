---
title: Deljenje state-a između komponenata
---

<Intro>

Ponekad želite da se state-ovi dve komponente menjaju zajedno. Da biste to uradili, uklonite state iz obe komponente, pomerite ga u najbližeg zajedničkog roditelja i prosledite ga nazad kroz props. Ovo je poznato kao *podizanje state-a* i jedna je od najčešćih stvari koje ćete pisati u React kodu.

</Intro>

<YouWillLearn>

- Kako da podizanjem delite state između komponenata
- Šta su kontrolisane i nekontrolisane komponente

</YouWillLearn>

## Podizanje state-a uz primer {/*lifting-state-up-by-example*/}

U ovom primeru, roditeljska `Accordion` komponenta renderuje dva različita `Panel`-a:

* `Accordion`
  - `Panel`
  - `Panel`

Svaka `Panel` komponenta ima boolean `isActive` state koji određuje da li je sadržaj vidljiv.

Pritisnite dugme "Prikaži" na oba panel-a:

<Sandpack>

```js
import { useState } from 'react';

function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          Prikaži
        </button>
      )}
    </section>
  );
}

export default function Accordion() {
  return (
    <>
      <h2>Almati, Kazahstan</h2>
      <Panel title="O gradu">
        Sa populacijom od oko 2 miliona, Almati je najveći grad u Kazahstanu. Bio je glavni grad od 1929. do. 1997. godine.
      </Panel>
      <Panel title="Etimologija">
        Ime potiče od reči <span lang="kk-KZ">алма</span>, što na kazaškom jeziku znači "jabuka", i često se prevodi kao "pun jabuka". U suštini, region koji okružuje Almati se smatra pradomovinom jabuka, a divlja <i lang="la">Malus sieversii</i> se smatra mogućim pretkom moderne domaće jabuke.
      </Panel>
    </>
  );
}
```

```css
h3, p { margin: 5px 0px; }
.panel {
  padding: 10px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Primetite kako pritiskanje dugmeta u jednom panel-u ne utiče na drugi--oni su nezavisni.

<DiagramGroup>

<Diagram name="sharing_state_child" height={367} width={477} alt="Dijagram koji prikazuje stablo od tri komponente, jednu roditeljsku pod nazivom Accordion i dve dečje sa nazivom Panel. Obe Panel komponente sadrže isActive sa vrednošću false.">

Inicijalno, `isActive` state svakog `Panel`-a je `false`, tako da su oba sklopljena

</Diagram>

<Diagram name="sharing_state_child_clicked" height={367} width={480} alt="Isti dijagram kao i prethodni, sa istaknutim isActive u prvoj dečjoj Panel komponenti označavajući da je klik postavio isActive vrednost na true. Druga Panel komponenta i dalje sadrži vrednost false." >

Klik na dugme bilo kog `Panel`-a će ažurirati samo `isActive` state tog `Panel`-a

</Diagram>

</DiagramGroup>

**Ali, recimo da to želite promeniti tako da samo jedan panel može biti proširen.** Sa takvim dizajnom, proširivanje drugog panel-a bi trebalo da sklopi prvi. Kako biste to uradili?

Da biste koordinisali ta dva panel-a, potrebno je da "podignete njihov state" u roditeljsku komponentu u ova tri koraka:

1. **Ukloniti** state iz dečjih komponenata.
2. **Proslediti** hardkodirane podatke iz zajedničkog roditelja.
3. **Dodati** state u zajedničkog roditelja i proslediti ga zajedno sa event handler-ima.

Ovo će omogućiti `Accordion` komponenti da koordiniše oba `Panel`-a i proširi samo jedan.

### Korak 1: Ukloniti state iz dečjih komponenata {/*step-1-remove-state-from-the-child-components*/}

Daćete kontrolu nad `isActive` iz `Panel`-a njegovoj roditeljskoj komponenti. Ovo znači da će roditeljska komponenta proslediti `isActive` u `Panel` kao prop. Počnite sa **uklanjanjem ove linije** iz `Panel` komponente:

```js
const [isActive, setIsActive] = useState(false);
```

I umesto toga, dodajte `isActive` u listu props-a u `Panel`-u:

```js
function Panel({ title, children, isActive }) {
```

Sada roditeljska komponenta `Panel`-a može *kontrolisati* `isActive` [prosleđivanjem prop-a](/learn/passing-props-to-a-component). Sa druge strane, `Panel` komponenta sada *nema kontrolu* nad vrednošću `isActive`--sada je to na roditeljskoj komponenti!

### Korak 2: Proslediti hardkodirane podatke iz zajedničkog roditelja {/*step-2-pass-hardcoded-data-from-the-common-parent*/}

Da biste podigli state morate locirati najbližu zajedničku roditeljsku komponentu *obe* dečje komponente koje želite da koordinišete:

* `Accordion` *(najbliži zajednički roditelj)*
  - `Panel`
  - `Panel`

U ovom primeru, to je `Accordion` komponenta. Pošto je iznad oba panel-a i može da kontroliše njihove props-e, postaće "izvor istine" označavajući koji panel je trenutno aktivan. Napravite da `Accordion` komponenta prosledi hardkodiranu vrednost za `isActive` (na primer, `true`) u oba panel-a:

<Sandpack>

```js
import { useState } from 'react';

export default function Accordion() {
  return (
    <>
      <h2>Almati, Kazahstan</h2>
      <Panel title="O gradu" isActive={true}>
        Sa populacijom od oko 2 miliona, Almati je najveći grad u Kazahstanu. Bio je glavni grad od 1929. do. 1997. godine.
      </Panel>
      <Panel title="Etimologija" isActive={true}>
        Ime potiče od reči <span lang="kk-KZ">алма</span>, što na kazaškom jeziku znači "jabuka", i često se prevodi kao "pun jabuka". U suštini, region koji okružuje Almati se smatra pradomovinom jabuka, a divlja <i lang="la">Malus sieversii</i> se smatra mogućim pretkom moderne domaće jabuke.
      </Panel>
    </>
  );
}

function Panel({ title, children, isActive }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          Prikaži
        </button>
      )}
    </section>
  );
}
```

```css
h3, p { margin: 5px 0px; }
.panel {
  padding: 10px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Pokušajte da promenite hardkodirane `isActive` vrednosti u `Accordion` komponenti da vidite rezultat na ekranu.

### Korak 3: Dodati state u zajedničkog roditelja {/*step-3-add-state-to-the-common-parent*/}

Podizanje state-a često menja prirodu onoga što čuvate kao state.

U ovom slučaju, samo jedan panel treba biti aktivan. To znači da zajednička roditeljska `Accordion` komponenta treba da prati *koji* panel je aktivan. Umesto `boolean` vrednosti može se koristiti broj kao indeks aktivnog `Panel`-a u state promenljivoj:

```js
const [activeIndex, setActiveIndex] = useState(0);
```

Kada je `activeIndex` jednak `0`, prvi panel je aktivan, a kada je `1`, aktivan je drugi.

Klik na dugme "Prikaži" u bilo kom `Panel`-u treba da promeni aktivan indeks u `Accordion`-u. `Panel` ne može da postavi `activeIndex` state direktno pošto je on definisan unutar `Accordion`-a. `Accordion` komponenta mora da *eksplicitno dozvoli* `Panel` komponenti da menja njen state [prosleđivanjem event handler-a kao prop-a](/learn/responding-to-events#passing-event-handlers-as-props):

```js
<>
  <Panel
    isActive={activeIndex === 0}
    onShow={() => setActiveIndex(0)}
  >
    ...
  </Panel>
  <Panel
    isActive={activeIndex === 1}
    onShow={() => setActiveIndex(1)}
  >
    ...
  </Panel>
</>
```

`<button>` unutar `Panel`-a će sada koristiti `onShow` prop kao svoj event handler za klik:

<Sandpack>

```js
import { useState } from 'react';

export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);
  return (
    <>
      <h2>Almati, Kazahstan</h2>
      <Panel
        title="O gradu"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        Sa populacijom od oko 2 miliona, Almati je najveći grad u Kazahstanu. Bio je glavni grad od 1929. do. 1997. godine.
      </Panel>
      <Panel
        title="Etimologija"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        Ime potiče od reči <span lang="kk-KZ">алма</span>, što na kazaškom jeziku znači "jabuka", i često se prevodi kao "pun jabuka". U suštini, region koji okružuje Almati se smatra pradomovinom jabuka, a divlja <i lang="la">Malus sieversii</i> se smatra mogućim pretkom moderne domaće jabuke.
      </Panel>
    </>
  );
}

function Panel({
  title,
  children,
  isActive,
  onShow
}) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>
          Prikaži
        </button>
      )}
    </section>
  );
}
```

```css
h3, p { margin: 5px 0px; }
.panel {
  padding: 10px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Ovim se zaokružuje podizanje state-a! Pomeranje state-a u zajedničku roditeljsku komponentu vam je omogućilo da koordinišete sa dva panel-a. Upotreba aktivnog indeksa umesto dva "da li je prikazano" flag-a osigurava da je samo jedan panel aktivan. Prosleđivanje event handler-a u dečje komponente je omogućilo deci da promene roditeljski state.

<DiagramGroup>

<Diagram name="sharing_state_parent" height={385} width={487} alt="Dijagram koji prikazuje stablo od tri komponente, jednu roditeljsku pod nazivom Accordion i dve dečje sa nazivom Panel. Accordion sadrži activeIndex sa vrednošću nula koja znači da je isActive vrednost true prosleđena u prvi Panel, a isActive vrednost false prosleđena u drugi Panel." >

Inicijalno, `Accordion`-ov `activeIndex` je `0`, pa prvi `Panel` prima `isActive = true`

</Diagram>

<Diagram name="sharing_state_parent_clicked" height={385} width={521} alt="Isti dijagram kao i prethodni, sa istaknutom activeIndex vrednošću u roditeljskoj Accordion komponenti označavajući da je klik promenio vrednost na jedan. Tok obe dečje Panel komponente je takođe istaknut, a isActive vrednost prosleđena svakom detetu je postavljena obrnuto: false za prvi Panel i true za drugi." >

Kada se `activeIndex` state u `Accordion`-u promeni na `1`, drugi panel `Panel` prima `isActive = true` umesto prvog

</Diagram>

</DiagramGroup>

<DeepDive>

#### Kontrolisane i nekontrolisane komponente {/*controlled-and-uncontrolled-components*/}

Uobičajeno je komponente sa lokalnim state-om nazivati "nekontrolisanim". Na primer, originalna `Panel` komponenta sa `isActive` state promenljivom je nekontrolisana jer njen roditelj ne može da utiče da li je panel aktivan ili ne.

Suprotno od toga, možete reći da je komponenta "kontrolisana" kada su bitne informacije u njoj vođene props-ima umesto njenim lokalnim state-om. Ovo omogućava roditeljskoj komponenti da potpuno specificira njeno ponašanje. Konačna `Panel` komponenta sa `isActive` prop-om je kontrolisana od strane `Accordion` komponente.

Nekontrolisane komponente je lakše koristiti unutar njihovih roditelja jer zahtevaju manje konfiguracije. Ali, one su manje fleksibilne ako želite da ih koordinišete. Kontrolisane komponente su maksimalno fleksibilne, ali zahtevaju da ih roditeljske komponente u potpunosti konfigurišu preko props-a.

U praksi, "kontrolisano" i "nekontrolisano" nisu striktno tehnički termini--svaka komponenta obično ima neku kombinaciju lokalnog state-a i props-a. Međutim, ovo je koristan način da pričamo o tome kako su komponente dizajnirane i koje sposobnosti pružaju.

Kada god pišete komponentu, razmotrite koje informacije trebaju biti kontrolisane (kroz props), a koje informacije trebaju biti nekontrolisane (kroz state). Ali, uvek se možete predomisliti i refaktorisati kasnije.

</DeepDive>

## Jedan izvor istine za svaki state {/*a-single-source-of-truth-for-each-state*/}

U React aplikaciji, mnoge komponente će imati svoj state. Poneki state može "živeti" u blizini listova (komponenti na dnu stabla) poput input-a. Drugi state može "živeti" bliže vrhu aplikacije. Na primer, čak su i biblioteke za rutiranje na klijentskoj strani često implementirane da čuvaju trenutnu rutu u React state-u, i da je prosleđuju deci kroz props!

**Za svaki jedinstveni deo state-a, izabraćete komponentu koja ga "poseduje".** Ovaj princip je poznat i kao ["jedan izvor istine"](https://en.wikipedia.org/wiki/Single_source_of_truth). To ne znači da svi state-ovi žive na jednom mestu--već da za _svaki_ deo state-a postoji _posebna_ komponenta koja drži tu informaciju. Umesto da duplirate deljeni state između komponenata, *podignite ga* u njihovog zajedničkog roditelja, koji će onda *proslediti* taj state deci kojima je potreban.

Vaša aplikacija će se menjati dok radite na njoj. Uobičajeno je da ćete pomerati state gore-dole dok pokušavate da shvatite gde svaki deo state-a "živi". Sve je to deo procesa!

Da biste videli kako ovo izgleda u praksi sa nekoliko komponenata, pročitajte [Razmišljanje u React-u](/learn/thinking-in-react).

<Recap>

* Kada želite da koordinišete dve komponente, pomerite njihov state u zajedničkog roditelja.
* Onda, prosledite tu informaciju iz zajedničkog roditelja kroz props.
* Na kraju, prosledite event handler-e kako bi deca mogla menjati roditeljski state.
* Korisno je smatrati komponente "kontrolisanim" (vođene props-ima) ili "nekontrolisanim" (vođene state-om).

</Recap>

<Challenges>

#### Sinhronizovani input-i {/*synced-inputs*/}

Ova dva input-a su nezavisna. Sinhronizujte ih: izmena jednog input-a treba da postavi isti tekst u drugi input, i obrnuto. 

<Hint>

Potrebno je da podignete state u roditeljsku komponentu.

</Hint>

<Sandpack>

```js
import { useState } from 'react';

export default function SyncedInputs() {
  return (
    <>
      <Input label="Prvi input" />
      <Input label="Drugi input" />
    </>
  );
}

function Input({ label }) {
  const [text, setText] = useState('');

  function handleChange(e) {
    setText(e.target.value);
  }

  return (
    <label>
      {label}
      {' '}
      <input
        value={text}
        onChange={handleChange}
      />
    </label>
  );
}
```

```css
input { margin: 5px; }
label { display: block; }
```

</Sandpack>

<Solution>

Pomerite `text` state promenljivu u roditeljsku komponentu zajedno sa `handleChange` handler-om. Onda ih prosledite kao props u obe `Input` komponente. Ovo će ih sinhronizovati.

<Sandpack>

```js
import { useState } from 'react';

export default function SyncedInputs() {
  const [text, setText] = useState('');

  function handleChange(e) {
    setText(e.target.value);
  }

  return (
    <>
      <Input
        label="Prvi input"
        value={text}
        onChange={handleChange}
      />
      <Input
        label="Drugi input"
        value={text}
        onChange={handleChange}
      />
    </>
  );
}

function Input({ label, value, onChange }) {
  return (
    <label>
      {label}
      {' '}
      <input
        value={value}
        onChange={onChange}
      />
    </label>
  );
}
```

```css
input { margin: 5px; }
label { display: block; }
```

</Sandpack>

</Solution>

#### Filtrirati listu {/*filtering-a-list*/}

U ovom primeru, `SearchBar` ima sopstveni `query` state koji kontroliše tekstualni input. Njegova roditeljska `FilterableList` komponenta prikazuje `List`-u elemenata, ali za pretragu ne uzima u obzir uneti upit.

Iskoristite `filterItems(foods, query)` funkciju da filtrirate listu u zavisnosti od upita za pretragu. Da biste testirali izmene, uverite se da unosom "d" u input filtrirate listu na "Dal" i "Dim sum".

Primetite da je `filterItems` već implementirana i import-ovana tako da to ne morate sami raditi!

<Hint>

Želećete da uklonite `query` state i `handleChange` handler iz `SearchBar`-a i da ih pomerite u `FilterableList`. Onda ih prosledite u `SearchBar` kao `query` i `onChange` props-e.

</Hint>

<Sandpack>

```js
import { useState } from 'react';
import { foods, filterItems } from './data.js';

export default function FilterableList() {
  return (
    <>
      <SearchBar />
      <hr />
      <List items={foods} />
    </>
  );
}

function SearchBar() {
  const [query, setQuery] = useState('');

  function handleChange(e) {
    setQuery(e.target.value);
  }

  return (
    <label>
      Pretraga:{' '}
      <input
        value={query}
        onChange={handleChange}
      />
    </label>
  );
}

function List({ items }) {
  return (
    <table>
      <tbody>
        {items.map(food => (
          <tr key={food.id}>
            <td>{food.name}</td>
            <td>{food.description}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

```js src/data.js
export function filterItems(items, query) {
  query = query.toLowerCase();
  return items.filter(item =>
    item.name.split(' ').some(word =>
      word.toLowerCase().startsWith(query)
    )
  );
}

export const foods = [{
  id: 0,
  name: 'Suši',
  description: 'Suši je tradicionalno japansko jelo pripremljeno sa pirinčem i sirćetom.'
}, {
  id: 1,
  name: 'Dal',
  description: 'Najčešći način za pripremu dal-a je u obliku supe gde se mogu dodati crni luk, paradajz i različiti začini.'
}, {
  id: 2,
  name: 'Piroge',
  description: 'Piroge su punjene knedle napravljene umotavanjem beskvasnog testa oko slanog ili slatkog fila i kuvaju se u ključaloj vodi.'
}, {
  id: 3,
  name: 'Šiš ćevap',
  description: 'Šiš ćevap je popularno jelo od grilovanih kockica mesa na ražnju.'
}, {
  id: 4,
  name: 'Dim sum',
  description: 'Dim sum je veliki izbor malih jela u kojima kantonski narod uživa u restoranima tokom doručka i ručka.'
}];
```

</Sandpack>

<Solution>

Podignite `query` state u `FilterableList` komponentu. Pozovite `filterItems(foods, query)` da biste dobili filtriranu listu i prosledite je u `List`. Sada promena input-a za upit utiče na listu:

<Sandpack>

```js
import { useState } from 'react';
import { foods, filterItems } from './data.js';

export default function FilterableList() {
  const [query, setQuery] = useState('');
  const results = filterItems(foods, query);

  function handleChange(e) {
    setQuery(e.target.value);
  }

  return (
    <>
      <SearchBar
        query={query}
        onChange={handleChange}
      />
      <hr />
      <List items={results} />
    </>
  );
}

function SearchBar({ query, onChange }) {
  return (
    <label>
      Pretraga:{' '}
      <input
        value={query}
        onChange={onChange}
      />
    </label>
  );
}

function List({ items }) {
  return (
    <table>
      <tbody> 
        {items.map(food => (
          <tr key={food.id}>
            <td>{food.name}</td>
            <td>{food.description}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

```js src/data.js
export function filterItems(items, query) {
  query = query.toLowerCase();
  return items.filter(item =>
    item.name.split(' ').some(word =>
      word.toLowerCase().startsWith(query)
    )
  );
}

export const foods = [{
  id: 0,
  name: 'Suši',
  description: 'Suši je tradicionalno japansko jelo pripremljeno sa pirinčem i sirćetom.'
}, {
  id: 1,
  name: 'Dal',
  description: 'Najčešći način za pripremu dal-a je u obliku supe gde se mogu dodati crni luk, paradajz i različiti začini.'
}, {
  id: 2,
  name: 'Piroge',
  description: 'Piroge su punjene knedle napravljene umotavanjem beskvasnog testa oko slanog ili slatkog fila i kuvaju se u ključaloj vodi.'
}, {
  id: 3,
  name: 'Šiš ćevap',
  description: 'Šiš ćevap je popularno jelo od grilovanih kockica mesa na ražnju.'
}, {
  id: 4,
  name: 'Dim sum',
  description: 'Dim sum je veliki izbor malih jela u kojima kantonski narod u\iva u restoranima tokom doručka i ručka.'
}];
```

</Sandpack>

</Solution>

</Challenges>
