---
title: Prosleđivanje props-a u komponentu
---

<Intro>

React komponente koriste *props* da bi međusobno komunicirale. Svaka roditeljska komponenta može proslediti informacije svojoj deci tako što im daje props. Props-i vas možda podsećaju na HTML atribute, ali pomoću njih možete proslediti bilo koju JavaScript vrednost, uključujući objekte, nizove i funkcije.

</Intro>

<YouWillLearn>

* Kako da prosledite props-e u komponentu
* Kako da pročitate props-e u komponenti
* Kako da definišete default vrednosti za props
* Kako da prosledite JSX komponenti
* Kako se props-i menjaju tokom vremena

</YouWillLearn>

## Poznati props-i {/*familiar-props*/}

Props-i su informacije koje prosleđujete u JSX tag. Na primer, `className`, `src`, `alt`, `width` i `height` su samo neki od props-a koje možete proslediti u `<img>`:

<Sandpack>

```js
function Avatar() {
  return (
    <img
      className="avatar"
      src="https://i.imgur.com/1bX5QH6.jpg"
      alt="Lin Lanying"
      width={100}
      height={100}
    />
  );
}

export default function Profile() {
  return (
    <Avatar />
  );
}
```

```css
body { min-height: 120px; }
.avatar { margin: 20px; border-radius: 50%; }
```

</Sandpack>

Props-i koje prosleđujete u `<img>` tag su predefinisani (ReactDOM se prilagođava [HTML standardu](https://www.w3.org/TR/html52/semantics-embedded-content.html#the-img-element)). Vi možete proslediti bilo koji props *vašim* komponentama, poput `<Avatar>`-a, da biste ih prilagodili svojim potrebama. Evo i kako!

## Prosleđivanje props-a u komponentu {/*passing-props-to-a-component*/}

U ovom primeru, `Profile` komponenta ne prosleđuje nikakav props svom detetu, `Avatar` komponenti:

```js
export default function Profile() {
  return (
    <Avatar />
  );
}
```

U dva koraka možete dodati props u `Avatar`.

### Korak 1: Proslediti props u dete komponentu {/*step-1-pass-props-to-the-child-component*/}

Prvo, prosledite props u `Avatar`. Na primer, prosledimo dva props-a: `person` (objekat) i `size` (broj):

```js
export default function Profile() {
  return (
    <Avatar
      person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
      size={100}
    />
  );
}
```

<Note>

Ako vas duple vitičaste zagrade nakon `person=` zbunjuju, setite se da [one samo predstavljaju objekat](/learn/javascript-in-jsx-with-curly-braces#using-double-curlies-css-and-other-objects-in-jsx) unutar JSX vitičastih zagrada.

</Note>

Sada ove props-e možete pročitati unutar `Avatar` komponente.

### Korak 2: Pročitati props unutar dečje komponente {/*step-2-read-props-inside-the-child-component*/}

Možete pročitati ove props-e izlistavanjem njihovih imena `person, size` odvojenih zarezima unutar `({` i `})` odmah nakon `function Avatar`. Ovo vam omogućava da ih koristite unutar `Avatar` funkcije, kao bilo koju drugu promenljivu.

```js
function Avatar({ person, size }) {
  // person i size su dostupni ovde
}
```

Dodajte neku logiku u `Avatar` koja će koristiti `person` i `size` props-e za renderovanje i gotovi ste.

Sada možete konfigurisati `Avatar` da renderuje različite stvari uz pomoć različitih props-a. Probajte da menjate vrednosti!

<Sandpack>

```js src/App.js
import { getImageUrl } from './utils.js';

function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

export default function Profile() {
  return (
    <div>
      <Avatar
        size={100}
        person={{ 
          name: 'Katsuko Saruhashi', 
          imageId: 'YfeOqp2'
        }}
      />
      <Avatar
        size={80}
        person={{
          name: 'Aklilu Lemma', 
          imageId: 'OKS67lh'
        }}
      />
      <Avatar
        size={50}
        person={{ 
          name: 'Lin Lanying',
          imageId: '1bX5QH6'
        }}
      />
    </div>
  );
}
```

```js src/utils.js
export function getImageUrl(person, size = 's') {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
body { min-height: 120px; }
.avatar { margin: 10px; border-radius: 50%; }
```

</Sandpack>

Props-i vam omogućavaju da odvojeno razmišljate o roditeljskim i dečjim komponentama. Na primer, možete promeniti `person` ili `size` props unutar `Profile`-a bez potrebe da razmišljate kako ih `Avatar` koristi. Slično tome, možete promeniti kako `Avatar` koristi te props-e bez gledanja u `Profile`.

Props-e možete zamisliti kao "dugmiće" koje možete podešavati. Oni igraju istu ulogu kao i argumenti u funkcijama—u suštini, props-i _jesu_ jedini argument u vašoj komponenti! Funckije React komponenata prihvataju samo jedan argument, `props` objekat:

```js
function Avatar(props) {
  let person = props.person;
  let size = props.size;
  // ...
}
```

Uglavnom vam neće biti potreban ceo `props` objekat, već ćete ga dekonstruisati na pojedinačne props-e.

<Pitfall>

**Nemojte propustiti par vitičastih zagrada** unutar `(` i `)` prilikom deklaracije props-a:

```js
function Avatar({ person, size }) {
  // ...
}
```

Ova sintaksa se naziva ["dekonstruisanje"](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Unpacking_fields_from_objects_passed_as_a_function_parameter) i jednaka je čitanju polja od parametra funkcije:

```js
function Avatar(props) {
  let person = props.person;
  let size = props.size;
  // ...
}
```

</Pitfall>

## Definisanje default vrednosti za prop {/*specifying-a-default-value-for-a-prop*/}

Ako želite da zadate default vrednost prop-u, koja će se koristiti kada vrednost nije definisana, to možete uraditi pomoću dekonstruisanja stavljajući `=` i default vrednost odmah nakon parametra:

```js
function Avatar({ person, size = 100 }) {
  // ...
}
```

Sada, ako se `<Avatar person={...} />` renderuje bez `size` prop-a, `size` će biti setovano na `100`.

Default vrednost je upotrebljena samo ako izostavite `size` prop ili ako prosledite `size={undefined}`. Ali, ako prosledite `size={null}` ili `size={0}`, default vrednost **neće** biti upotrebljena.

## Prosleđivanje props-a sa JSX spread sintaksom {/*forwarding-props-with-the-jsx-spread-syntax*/}

Ponekad se prosleđivanje props-a može ponavljati:

```js
function Profile({ person, size, isSepia, thickBorder }) {
  return (
    <div className="card">
      <Avatar
        person={person}
        size={size}
        isSepia={isSepia}
        thickBorder={thickBorder}
      />
    </div>
  );
}
```

Nema ništa loše u ponavljanju koda—ponekad može biti čitljivije. Ali, nekada vam je potrebna sažetost. Neke komponente prosleđuju sve svoje props-e deci, kao što to `Profile` radi sa `Avatar`-om. Pošto one direktno ne koriste nijedan svoj prop, ima smisla upotrebiti koncizniju "spread" sintaksu:

```js
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} />
    </div>
  );
}
```

Ovo prosleđuje sve props-e od `Profile`-a u `Avatar` bez izlistavanja njihovih imena pojedinačno.

**Koristite spread sintaksu odmereno.** Ako je koristite u svakoj komponenti, nešto nije u redu. To često označava da biste trebali podeliti komponente i proslediti decu kao JSX. Više u nastavku!

## Prosleđivanje JSX-a kao dece {/*passing-jsx-as-children*/}

Ugnježdavanje ugrađenih (built-in) tag-ova je često:

```js
<div>
  <img />
</div>
```

Ponekad želite da ugnjezdite vaše komponente na isti način:

```js
<Card>
  <Avatar />
</Card>
```

Kada ugnjezdite sadržaj unutar JSX tag-a, roditeljska komponenta će primiti taj sadržaj kroz prop po imenu `children`. Na primer, `Card` komponenta ispod će primiti `children` prop setovan na `<Avatar />` i renderovati ga u wrapper div-u:

<Sandpack>

```js src/App.js
import Avatar from './Avatar.js';

function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{ 
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
    </Card>
  );
}
```

```js src/Avatar.js
import { getImageUrl } from './utils.js';

export default function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}
```

```js src/utils.js
export function getImageUrl(person, size = 's') {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
.card {
  width: fit-content;
  margin: 5px;
  padding: 5px;
  font-size: 20px;
  text-align: center;
  border: 1px solid #aaa;
  border-radius: 20px;
  background: #fff;
}
.avatar {
  margin: 20px;
  border-radius: 50%;
}
```

</Sandpack>

Pokušajte da promenite `<Avatar>` unutar `<Card>`-a sa nekim tekstom da biste videli kako `Card` komponenta može da obmota bilo koji ugnježdeni sadržaj. Ona ne mora da "zna" šta će biti renderovano unutar nje. Videćete ovaj fleksibilni obrazac na mnogim mestima.

Komponentu sa `children` prop možete zamisliti kao da ima "šupljinu" koja će biti "popunjena" u njenim roditeljskim komponentama sa proizvoljnim JSX-om. Često ćete koristiti `children` prop za vizuelne wrapper-e: panel-e, grid-ove, itd.

<Illustration src="/images/docs/illustrations/i_children-prop.png" alt='A puzzle-like Card tile with a slot for "children" pieces like text and Avatar' />

## Kako se props-i menjaju tokom vremena {/*how-props-change-over-time*/}

`Clock` komponenta ispod prima dva props-a od svog roditelja: `color` i `time`. (Kod roditeljske komponente je izostavljen zato što koristi [state](/learn/state-a-components-memory), u koji nećemo zalaziti još uvek.)

Pokušajte da promenite boju u primeru ispod:

<Sandpack>

```js src/Clock.js active
export default function Clock({ color, time }) {
  return (
    <h1 style={{ color: color }}>
      {time}
    </h1>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  const [color, setColor] = useState('lightcoral');
  return (
    <div>
      <p>
        Izaberite boju:{' '}
        <select value={color} onChange={e => setColor(e.target.value)}>
          <option value="lightcoral">lightcoral</option>
          <option value="midnightblue">midnightblue</option>
          <option value="rebeccapurple">rebeccapurple</option>
        </select>
      </p>
      <Clock color={color} time={time.toLocaleTimeString()} />
    </div>
  );
}
```

</Sandpack>

Ovaj primer ilustruje kako **komponenta može primiti različite props-e tokom vremena**. Props-i nisu uvek statički! Ovde se `time` prop menja svake sekunde, a `color` prop se menja kada odaberete drugu boju. Props-i odražavaju podatke u komponenti u bilo kom trenutku, a ne samo na početku.

Međutim, props-i su [immutable](https://en.wikipedia.org/wiki/Immutable_object)—pojam koji u informatici označava nešto "nepromenljivo". Kada komponenta treba da promeni svoje props-e (na primer, nakon interakcije korisnika ili na osnovu novih podataka), moraće da "pita" svog roditelja da joj prosledi _druge props-e_—novi objekat! Njeni stari props-i će biti odbačeni, a JavaScript će se u nekom trenutku pobrinuti da oslobodi memoriju koju su zauzeli.

**Ne pokušavajte da "menjate props".** Kada želite da reagujete na korisnički input (kao što je promena odabrane boje), moraćete da "set-ujete state", što možete naučiti u [State: Memorija komponente](/learn/state-a-components-memory).

<Recap>

* Da biste prosledili props, dodajte ih u JSX, kao što bi to uradili sa HTML atributima.
* Da biste pročitali props, koristite `function Avatar({ person, size })` sintaksu dekonstruisanja.
* Možete definisati default vrednost poput `size = 100`, koja se koristi za izostavljene i `undefined` props-e.
* Možete proslediti sve props-e sa `<Avatar {...props} />` JSX spread sintaksom, ali nemojte je previše koristiti!
* Ugnježdeni JSX poput `<Card><Avatar /></Card>` će se pojaviti u `Card` komponenti kao `children` prop.
* Props-i su read-only snapshot-ovi u vremenu: svako renderovanje prima novu verziju props-a.
* Ne možete menjati props-e. Kada vam je potrebna interaktivnost, trebate da setujete state.

</Recap>



<Challenges>

#### Izdvojiti komponentu {/*extract-a-component*/}

Ova `Gallery` komponenta sadrži veoma sličan markup za dva profila. Izdvojite `Profile` komponentu kako biste smanjili ponavljanje. Moraćete da izaberete koje props-e da joj prosledite.

<Sandpack>

```js src/App.js
import { getImageUrl } from './utils.js';

export default function Gallery() {
  return (
    <div>
      <h1>Značajni naučnici</h1>
      <section className="profile">
        <h2>Maria Skłodowska-Curie</h2>
        <img
          className="avatar"
          src={getImageUrl('szV5sdG')}
          alt="Maria Skłodowska-Curie"
          width={70}
          height={70}
        />
        <ul>
          <li>
            <b>Profesija: </b> 
            fizičar i hemičar
          </li>
          <li>
            <b>Nagrade: 4 </b> 
            (Nobelova nagrada za fiziku, Nobelova nagrada za hemiju, Davy medalja, Matteucci medalja)
          </li>
          <li>
            <b>Otkrića: </b>
            polonijum (hemijski element)
          </li>
        </ul>
      </section>
      <section className="profile">
        <h2>Katsuko Saruhashi</h2>
        <img
          className="avatar"
          src={getImageUrl('YfeOqp2')}
          alt="Katsuko Saruhashi"
          width={70}
          height={70}
        />
        <ul>
          <li>
            <b>Profesija: </b> 
            geohemičar
          </li>
          <li>
            <b>Nagrade: 2 </b> 
            (Miyake nagrada za geohemiju, Tanaka nagrada)
          </li>
          <li>
            <b>Otkrića: </b>
            metoda za merenje ugljen-dioksida u morskoj vodi
          </li>
        </ul>
      </section>
    </div>
  );
}
```

```js src/utils.js
export function getImageUrl(imageId, size = 's') {
  return (
    'https://i.imgur.com/' +
    imageId +
    size +
    '.jpg'
  );
}
```

```css
.avatar { margin: 5px; border-radius: 50%; min-height: 70px; }
.profile {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}
h1, h2 { margin: 5px; }
h1 { margin-bottom: 10px; }
ul { padding: 0px 10px 0px 20px; }
li { margin: 5px; }
```

</Sandpack>

<Hint>

Započnite sa izdvajanjem markup-a za jednog od naučnika. Nakon toga, pronađite koji delovi ne odgovaraju u drugom primeru, pa ih konfigurišite pomoću props-a.

</Hint>

<Solution>

U ovom primeru, `Profile` komponenta prihvata više props-a: `imageId` (string), `name` (string), `profession` (string), `awards` (niz string-ova), `discovery` (string) i `imageSize` (broj).

Primetite da `imageSize` prop ima default vrednost, pa je zato ne prosleđujemo u komponentu.

<Sandpack>

```js src/App.js
import { getImageUrl } from './utils.js';

function Profile({
  imageId,
  name,
  profession,
  awards,
  discovery,
  imageSize = 70
}) {
  return (
    <section className="profile">
      <h2>{name}</h2>
      <img
        className="avatar"
        src={getImageUrl(imageId)}
        alt={name}
        width={imageSize}
        height={imageSize}
      />
      <ul>
        <li><b>Profesija:</b> {profession}</li>
        <li>
          <b>Nagrade: {awards.length} </b>
          ({awards.join(', ')})
        </li>
        <li>
          <b>Otkrića: </b>
          {discovery}
        </li>
      </ul>
    </section>
  );
}

export default function Gallery() {
  return (
    <div>
      <h1>Značajni naučnici</h1>
      <Profile
        imageId="szV5sdG"
        name="Maria Skłodowska-Curie"
        profession="fizičar i hemičar"
        discovery="polonijum (hemijski element)"
        awards={[
          'Nobelova nagrada za fiziku',
          'Nobelova nagrada za hemiju',
          'Davy medalja',
          'Matteucci medalja'
        ]}
      />
      <Profile
        imageId='YfeOqp2'
        name='Katsuko Saruhashi'
        profession='geohemičar'
        discovery="metoda za merenje ugljen-dioksida u morskoj vodi"
        awards={[
          'Miyake nagrada za geohemiju',
          'Tanaka nagrada'
        ]}
      />
    </div>
  );
}
```

```js src/utils.js
export function getImageUrl(imageId, size = 's') {
  return (
    'https://i.imgur.com/' +
    imageId +
    size +
    '.jpg'
  );
}
```

```css
.avatar { margin: 5px; border-radius: 50%; min-height: 70px; }
.profile {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}
h1, h2 { margin: 5px; }
h1 { margin-bottom: 10px; }
ul { padding: 0px 10px 0px 20px; }
li { margin: 5px; }
```

</Sandpack>

Zapazite da ne morate prosleđivati poseban `awardCount` prop ako je `awards` niz. Možete koristiti `awards.length` da izračunate broj nagrada. Zapamtite da props-i mogu imati bilo koju vrednost, što uključuje i nizove!

Drugo rešenje, koje je sličnije ranijim primerima na ovoj stranici, je da grupišete sve informacije o osobi u jedan objekat i da prosledite taj objekat kao jedinstven prop:

<Sandpack>

```js src/App.js
import { getImageUrl } from './utils.js';

function Profile({ person, imageSize = 70 }) {
  const imageSrc = getImageUrl(person)

  return (
    <section className="profile">
      <h2>{person.name}</h2>
      <img
        className="avatar"
        src={imageSrc}
        alt={person.name}
        width={imageSize}
        height={imageSize}
      />
      <ul>
        <li>
          <b>Profesija:</b> {person.profession}
        </li>
        <li>
          <b>Nagrade: {person.awards.length} </b>
          ({person.awards.join(', ')})
        </li>
        <li>
          <b>Otkrića: </b>
          {person.discovery}
        </li>
      </ul>
    </section>
  )
}

export default function Gallery() {
  return (
    <div>
      <h1>Značajni naučnici</h1>
      <Profile person={{
        imageId: 'szV5sdG',
        name: 'Maria Skłodowska-Curie',
        profession: 'fizičar i hemičar',
        discovery: 'polonijum (hemijski element)',
        awards: [
          'Nobelova nagrada za fiziku',
          'Nobelova nagrada za hemiju',
          'Davy medalja',
          'Matteucci medalja'
        ],
      }} />
      <Profile person={{
        imageId: 'YfeOqp2',
        name: 'Katsuko Saruhashi',
        profession: 'geohemičar',
        discovery: 'metoda za merenje ugljen-dioksida u morskoj vodi',
        awards: [
          'Miyake nagrada za geohemiju',
          'Tanaka nagrada'
        ],
      }} />
    </div>
  );
}
```

```js src/utils.js
export function getImageUrl(person, size = 's') {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
.avatar { margin: 5px; border-radius: 50%; min-height: 70px; }
.profile {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
}
h1, h2 { margin: 5px; }
h1 { margin-bottom: 10px; }
ul { padding: 0px 10px 0px 20px; }
li { margin: 5px; }
```

</Sandpack>

Iako sintaksa izgleda drugačije, pošto koristite polja JavaScript objekta umesto kolekcije JSX atributa, oba primera su u mnogome slična i možete odabrati bilo koji pristup.

</Solution>

#### Prilagoditi veličinu slike u zavisnosti od prop-a {/*adjust-the-image-size-based-on-a-prop*/}

U ovom primeru, `Avatar` prima brojčani `size` prop koji određuje visinu i širinu za `<img>` tag. Ovde je vrednost `size` prop-a setovana na `40`. Međutim, ako otvorite sliku u novom tab-u, primetićete da je slika veća (`160` piksela). Stvarna veličina slike zavisi od toga koju veličinu thumbnail-a zatražite.

Izmenite `Avatar` komponentu da zatraži najbližu veličinu u zavisnosti od `size` prop-a. Konkretno, ako je `size` manje od `90`, prosledite `'s'` ("small" tj. "malo") umesto `'b'` ("big" tj. "veliko") u `getImageUrl` funkciju. Potvrdite da vaše promene rade renderovanjem avatara sa različitim vrednostima za `size` prop i otvaranjem slika u novom tab-u.

<Sandpack>

```js src/App.js
import { getImageUrl } from './utils.js';

function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person, 'b')}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

export default function Profile() {
  return (
    <Avatar
      size={40}
      person={{ 
        name: 'Gregorio Y. Zara', 
        imageId: '7vQD0fP'
      }}
    />
  );
}
```

```js src/utils.js
export function getImageUrl(person, size) {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
.avatar { margin: 20px; border-radius: 50%; }
```

</Sandpack>

<Solution>

Ovako to možete uraditi:

<Sandpack>

```js src/App.js
import { getImageUrl } from './utils.js';

function Avatar({ person, size }) {
  let thumbnailSize = 's';
  if (size > 90) {
    thumbnailSize = 'b';
  }
  return (
    <img
      className="avatar"
      src={getImageUrl(person, thumbnailSize)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

export default function Profile() {
  return (
    <>
      <Avatar
        size={40}
        person={{ 
          name: 'Gregorio Y. Zara', 
          imageId: '7vQD0fP'
        }}
      />
      <Avatar
        size={120}
        person={{ 
          name: 'Gregorio Y. Zara', 
          imageId: '7vQD0fP'
        }}
      />
    </>
  );
}
```

```js src/utils.js
export function getImageUrl(person, size) {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
.avatar { margin: 20px; border-radius: 50%; }
```

</Sandpack>

Možete prikazati i oštriju sliku za ekrane koji imaju visok DPI uzimajući u obzir [`window.devicePixelRatio`](https://developer.mozilla.org/en-US/docs/Web/API/Window/devicePixelRatio):

<Sandpack>

```js src/App.js
import { getImageUrl } from './utils.js';

const ratio = window.devicePixelRatio;

function Avatar({ person, size }) {
  let thumbnailSize = 's';
  if (size * ratio > 90) {
    thumbnailSize = 'b';
  }
  return (
    <img
      className="avatar"
      src={getImageUrl(person, thumbnailSize)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

export default function Profile() {
  return (
    <>
      <Avatar
        size={40}
        person={{ 
          name: 'Gregorio Y. Zara', 
          imageId: '7vQD0fP'
        }}
      />
      <Avatar
        size={70}
        person={{ 
          name: 'Gregorio Y. Zara', 
          imageId: '7vQD0fP'
        }}
      />
      <Avatar
        size={120}
        person={{ 
          name: 'Gregorio Y. Zara', 
          imageId: '7vQD0fP'
        }}
      />
    </>
  );
}
```

```js src/utils.js
export function getImageUrl(person, size) {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

```css
.avatar { margin: 20px; border-radius: 50%; }
```

</Sandpack>

Props-i vam omogućavaju da enkapsulirate logiku unutar `Avatar` komponente (i menjate je po potrebi) kako bi svi mogli da koriste `<Avatar>` bez razmišljanja o tome kako da potražuju slike i menjaju im veličinu.

</Solution>

#### Proslediti JSX u `children` prop {/*passing-jsx-in-a-children-prop*/}

Izdvojite `Card` komponentu iz markup-a ispod i koristite `children` prop da joj prosledite različit JSX:

<Sandpack>

```js
export default function Profile() {
  return (
    <div>
      <div className="card">
        <div className="card-content">
          <h1>Slika</h1>
          <img
            className="avatar"
            src="https://i.imgur.com/OKS67lhm.jpg"
            alt="Aklilu Lemma"
            width={70}
            height={70}
          />
        </div>
      </div>
      <div className="card">
        <div className="card-content">
          <h1>Opis</h1>
          <p>Aklilu Lemma je bio istaknuti etiopijski naučnik koji je izumeo prirodni tretman za šistozomijazu.</p>
        </div>
      </div>
    </div>
  );
}
```

```css
.card {
  width: fit-content;
  margin: 20px;
  padding: 20px;
  border: 1px solid #aaa;
  border-radius: 20px;
  background: #fff;
}
.card-content {
  text-align: center;
}
.avatar {
  margin: 10px;
  border-radius: 50%;
}
h1 {
  margin: 5px;
  padding: 0;
  font-size: 24px;
}
```

</Sandpack>

<Hint>

JSX koji postavite unutar tag-a neke komponente, biće joj prosleđen kao `children` prop.

</Hint>

<Solution>

Ovako možete koristiti `Card` komponentu na oba mesta:

<Sandpack>

```js
function Card({ children }) {
  return (
    <div className="card">
      <div className="card-content">
        {children}
      </div>
    </div>
  );
}

export default function Profile() {
  return (
    <div>
      <Card>
        <h1>Slika</h1>
        <img
          className="avatar"
          src="https://i.imgur.com/OKS67lhm.jpg"
          alt="Aklilu Lemma"
          width={100}
          height={100}
        />
      </Card>
      <Card>
        <h1>Opis</h1>
        <p>Aklilu Lemma je bio istaknuti etiopijski naučnik koji je izumeo prirodni tretman za šistozomijazu.</p>
      </Card>
    </div>
  );
}
```

```css
.card {
  width: fit-content;
  margin: 20px;
  padding: 20px;
  border: 1px solid #aaa;
  border-radius: 20px;
  background: #fff;
}
.card-content {
  text-align: center;
}
.avatar {
  margin: 10px;
  border-radius: 50%;
}
h1 {
  margin: 5px;
  padding: 0;
  font-size: 24px;
}
```

</Sandpack>

Možete izdvojiti `title` u odvojeni prop ako želite da svaka `Card` komponenta ima naslov:

<Sandpack>

```js
function Card({ children, title }) {
  return (
    <div className="card">
      <div className="card-content">
        <h1>{title}</h1>
        {children}
      </div>
    </div>
  );
}

export default function Profile() {
  return (
    <div>
      <Card title="Slika">
        <img
          className="avatar"
          src="https://i.imgur.com/OKS67lhm.jpg"
          alt="Aklilu Lemma"
          width={100}
          height={100}
        />
      </Card>
      <Card title="Opis">
        <p>Aklilu Lemma je bio istaknuti etiopijski naučnik koji je izumeo prirodni tretman za šistozomijazu.</p>
      </Card>
    </div>
  );
}
```

```css
.card {
  width: fit-content;
  margin: 20px;
  padding: 20px;
  border: 1px solid #aaa;
  border-radius: 20px;
  background: #fff;
}
.card-content {
  text-align: center;
}
.avatar {
  margin: 10px;
  border-radius: 50%;
}
h1 {
  margin: 5px;
  padding: 0;
  font-size: 24px;
}
```

</Sandpack>

</Solution>

</Challenges>
