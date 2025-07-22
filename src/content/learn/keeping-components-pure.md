---
title: Održavati komponente čistim
---

<Intro>

Neke JavaScript funkcije su *čiste*. Čiste funkcije izvršavaju samo proračune i ništa više. Pisanjem komponenata koje su striktno čiste funkcije, moći ćete da izbegnete zbunjujuće bug-ove u vašim klasama, iako se vaš projekat povećava. Da biste dobili te benefite, postoji par pravila koje morate ispoštovati.

</Intro>

<YouWillLearn>

* Šta je zapravo čistoća i kako vam pomaže da izbegnete bug-ove
* Kako održati komponente čistim zadržavanjem promenena van faze renderovanja
* Kako da koristite Strict Mode da biste pronašli greške u komponentama

</YouWillLearn>

## Čistoća: Komponente kao formule {/*purity-components-as-formulas*/}

U informatici (i pogotovo u svetu funkcionalnog programiranja), [čista funkcija](https://wikipedia.org/wiki/Pure_function) je funkcija koju odlikuju sledeće karakteristike:

* **Gleda samo svoja posla.** Ne menja nikakve objekte ili promenljive koji su postojali pre njenog poziva.
* **Isti input-i, isti rezultat.** Dobijanjem istih input-a, čista funkcija treba uvek da vrati isti rezultat.

Možda ste već upoznati sa jednim primerom čistih funkcija: formulama u matematici.

Pogledajte ovu matematičku formulu: <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math>.

Ako je <Math><MathI>x</MathI> = 2</Math> onda je <Math><MathI>y</MathI> = 4</Math>. Uvek.

Ako je <Math><MathI>x</MathI> = 3</Math> onda je <Math><MathI>y</MathI> = 6</Math>. Uvek.

Ako je <Math><MathI>x</MathI> = 3</Math>, <MathI>y</MathI> neće ponekad biti <Math>9</Math> ili <Math>–1</Math> ili <Math>2.5</Math> u zavisnosti od trenutnog vremena ili stanja na berzi.

Ako je <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math> i <Math><MathI>x</MathI> = 3</Math>, <MathI>y</MathI> će _uvek_ biti <Math>6</Math>. 

Ako prebacimo ovo u JavaScript funkciju, izgledalo bi ovako:

```js
function double(number) {
  return 2 * number;
}
```

U primeru iznad, `double` je **čista funkcija**. Ako joj prosledite `3`, vratiće `6`. Uvek.

React je dizajniran oko ovog koncepta. **React pretpostavlja da je svaka komponenta koju napišete čista funkcija.** To znači da React komponente koje pišete uvek moraju vratiti isti JSX kada im prosledite iste input-e:

<Sandpack>

```js src/App.js
function Recipe({ drinkers }) {
  return (
    <ol>    
      <li>Skuvati {drinkers} časa sa vodom.</li>
      <li>Dodati {drinkers} kašika čaja i {0.5 * drinkers} kašika začina.</li>
      <li>Dodati {0.5 * drinkers} šolja mleka da provri i šećer po ukusu.</li>
    </ol>
  );
}

export default function App() {
  return (
    <section>
      <h1>Recept za začinjeni čaj</h1>
      <h2>Za dvoje</h2>
      <Recipe drinkers={2} />
      <h2>Za okupljanje</h2>
      <Recipe drinkers={4} />
    </section>
  );
}
```

</Sandpack>

Kada prosledite `drinkers={2}` u `Recipe`, on će vratiti JSX koji sadrži `2 časa sa vodom`. Uvek. 

Ako prosledite `drinkers={4}`, on će vratiti JSX koji sadrži `4 časa sa vodom`. Uvek.

Baš kao matematička formula. 

Možete gledati vaše komponente kao recepte: ako ih pratite i ne dodajete nove sastojke tokom kuvanja, svaki put ćete dobiti isto jelo. To "jelo" je JSX koji će komponenta servirati React-u da [renderuje](/learn/render-and-commit).

<Illustration src="/images/docs/illustrations/i_puritea-recipe.png" alt="Recept za čaj za x ljudi: uzmi x čaša sa vodom, dodaj x kašika čaja i 0.5x kašika začina, i 0.5x šolja mleka" />

## Propratni efekti: (ne)namerne posledice {/*side-effects-unintended-consequences*/}

React-ov proces renderovanja mora uvek biti čist. Komponente bi trebale uvek da *vrate* svoj JSX, i da ne *menjaju* nikakve objekte ili promenljive koji su postojali pre renderovanja—to bi ih učinilo nečistim!

Evo komponente koja krši ovo pravilo:

<Sandpack>

```js
let guest = 0;

function Cup() {
  // Loše: menjanje već postojeće promenljive!
  guest = guest + 1;
  return <h2>Šolja čaja za gosta #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
```

</Sandpack>

Ova komponenta čita i piše `guest` promenljivu definisanu izvan nje. To znači da će **pozivanje ove komponente više puta proizvesti različit JSX**! I dodatno, ako _druge_ komponente čitaju `guest`, i one će proizvesti različit JSX takođe, u zavisnosti od toga kada su renderovane! To nije predvidljivo.

Vratimo se našoj formuli <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math>. Sad iako je <Math><MathI>x</MathI> = 2</Math>, ne možemo tvrditi da je <Math><MathI>y</MathI> = 4</Math>. Naši testovi će biti neuspešni, korisnici pogubljeni, avioni će padati sa neba—vidite kako ovo može dovesti do zbunjujućih bug-ova!

Ovu komponentu možete popraviti [prosleđivanjem `guest`-a kao prop-a](/learn/passing-props-to-a-component):

<Sandpack>

```js
function Cup({ guest }) {
  return <h2>Šolja čaja za gosta #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
```

</Sandpack>

Sada je vaša komponenta čista jer JSX zavisi samo od `guest` prop-a.

Uglavnom, ne trebate očekivati da se vaše komponente renderuju u određenom redosledu. Nije bitno da li <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math> pozovete pre ili posle <Math><MathI>y</MathI> = 5<MathI>x</MathI></Math>: obe formule će se rešiti nevezano jedna od druge. Na isti način, svaka komponenta treba jedino da "misli na sebe", a ne da pokušava da se usklađuje ili da zavisi od drugih tokom renderovanja. Renderovanje je nalik na test u školi: svaka komponenta treba da računa JSX samostalno!

<DeepDive>

#### Detektovanje nečistih proračuna sa StrictMode-om {/*detecting-impure-calculations-with-strict-mode*/}

Iako ih možda niste sve koristili još uvek, u React-u postoje tri vrste input-a koje možete čitati tokom renderovanja: [props](/learn/passing-props-to-a-component), [state](/learn/state-a-components-memory) i [context](/learn/passing-data-deeply-with-context). Ove input-e uvek trebate tretirati kao da su samo za čitanje.

Kada želite nešto *promeniti* u zavisnosti od korisničkog input-a, trebate [setovati state](/learn/state-a-components-memory) umesto da pišete u promenljivu. Nikada ne bi trebalo menjati već postojeće promenljive ili objekte dok se vaša komponenta renderuje.

React nudi "Strict Mode" u kojem se dvaput pozivaju funkcije svake komponente u toku razvoja. **Pozivanjem svake funkcije dvaput, Strict Mode pomaže u pronalasku komponenata koje krše ova pravila.**

Primetite da je originalni primer prikazivao "Gost #2", "Gost #4" i "Gost #6" umesto "Gost #1", "Gost #2" i "Gost #3". Originalna funkcija je bila nečista, pa ju je pozivanje dva puta pokvarilo. Popravljena, čista verzija radi čak iako se funkcija poziva stalno dva puta. **Čiste funkcije samo računaju, pa pozivanje dva puta neće promeniti ništa**--baš kao što pozivanje `double(2)` dva puta ne menja povratnu vrednost, a ni rešavanje <Math><MathI>y</MathI> = 2<MathI>x</MathI></Math> dva puta ne menja koliko je <MathI>y</MathI>. Isti input-i, isti rezultati. Uvek.

Strict Mode nema efekta u produkciji, tako da neće usporiti vašu aplikaciju za korisnike. Da biste uključili Strict Mode, možete obmotati vašu root komponentu sa `<React.StrictMode>`. Neki framework-ovi ovo rade po default-u.

</DeepDive>

### Lokalna mutacija: Mala tajna vaše komponente {/*local-mutation-your-components-little-secret*/}

U primeru iznad, problem je bio u tome što je komponenta promenila *već postojeću* promenljivu tokom renderovanja. To se često zove **"mutacija"** kako bi zvučalo malko strašnije. Čista funkcija ne menja promenljive izvan opsega te funkcije niti objekte koji su kreirani pre njenog poziva—to je čini nečistom!

Međutim, **potpuno je u redu menjati promenljive i objekte koje ste *upravo* kreirali tokom renderovanja**. U ovom primeru, kreirate `[]` niz i dodeljujete ga u `cups` promenljivu, a onda pozovete `push` da dodate nekoliko čaša u taj niz:

<Sandpack>

```js
function Cup({ guest }) {
  return <h2>Šolja čaja za gosta #{guest}</h2>;
}

export default function TeaGathering() {
  const cups = [];
  for (let i = 1; i <= 12; i++) {
    cups.push(<Cup key={i} guest={i} />);
  }
  return cups;
}
```

</Sandpack>

Da su `cups` promenljiva ili `[]` niz bili kreirani izvan `TeaGathering` funkcije, ovo bi bio ogroman problem! Menjali bisti *već postojeći* objekat dodavanjem stavki u taj niz.

Međutim, u redu je jer ste ih kreirali *u toku renderovanja*, unutar `TeaGathering`. Kod izvan `TeaGathering` nikad neće saznati da se ovo desilo. To se zove **"lokalna mutacija"**—kao mala tajna vaše komponente.

## Gde _možete_ izazvati propratne efekte {/*where-you-_can_-cause-side-effects*/}

Iako se funkcionalno programiranje zasniva na čistoći, u nekom trenutku, negde, _nešto_ se mora promeniti. To je donekle svrha programiranja! Te promene—ažuriranje ekrana, početak animacije, promena podataka—se nazivaju **propratni efekti**. To su stvari koje se dešavaju _"sa strane"_, a ne tokom renderovanja.

U React-u, **propratni efekti često pripadaju [event handler-ima](/learn/responding-to-events)**. Event handler-i su funkcije koje React pokreće kada izvršite neku akciju—na primer, kada kliknete dugme. Iako su event handler-i definisani *unutar* vaše komponente, oni se ne izvršavaju *tokom* renderovanja! **Zbog toga event handler-i ne moraju biti čisti.**

Ako ste iscrpeli sve ostale opcije i ne možete naći odgovarajući event handler za vaš propratni efekat, i dalje ga možete zakačiti u povratni JSX sa [`useEffect`](/reference/react/useEffect) pozivom u vašoj komponenti. Ovo govori React-u da izvrši nešto kasnije, nakon renderovanja, kada su propratni efekti dozvoljeni. **Međutim, ovaj pristup bi trebao biti vaše poslednje rešenje.**

Kada je moguće, pokušajte izraziti vašu logiku samo u toku renderovanja. Iznenadićete se koliko vas daleko to može dovesti!

<DeepDive>

#### Zašto React vodi računa o čistoći? {/*why-does-react-care-about-purity*/}

Pisanje čistih funkcija zahteva naviku i disciplinu. Takođe, otključava pregršt mogućnosti:

* Vaše komponente se mogu izvršavati u različitim sredinama—na primer, na serveru! Pošto vraćaju iste rezultate za iste input-e, jedna komponenta može opslužiti više korisničkih zahteva.
* Možete poboljšati performanse [preskakanjem renderovanja](/reference/react/memo) komponenata čiji se input-i nisu promenili. Ovo je sigurno pošto čiste funkcije uvek vraćaju iste rezultate, pa je sigurno keširati ih.
* Ako se neki podatak promeni u toku renderovanja dubokog stabla komponenata, React može ponovo pokrenuti renderovanje bez da troši vreme na završetak zastarelog renderovanja. Čistoća čini bezbednim prekid proračuna u bilo kom trenutku.

Svaka nova React funkcionalnost koju pravimo koristi prednosti čistoće. Održavanje komponenata čistim otključava moć React paradigme, od fetch-ovanja podataka do animacija i performansi.

</DeepDive>

<Recap>

* Komponenta mora biti čista, što znači da:
  * **Gleda samo svoja posla.** Ne bi trebalo da menja nikakve objekte ili promenljive koji su postojali pre renderovanja.
  * **Isti input-i, isti rezultat.** Dobijanjem istih input-a, komponenta treba uvek da vrati isti JSX. 
* Renderovanje se može desiti u bilo kom trenutku, pa komponente ne bi trebale da zavise od drugih.
* Ne bi trebali da menjate nijedan od input-a koje komponenta koristi za renderovanje. To uključuje props, state i context. Da biste ažurirali prikaz, ["setujte" state](/learn/state-a-components-memory) umesto da menjate već postojeće objekte.
* Težite da izrazite logiku komponente u JSX-u koji vraćate. Kada trebate "menjati stvari", uglavnom ćete želeti da to uradite u event handler-u. Kao poslednje rešenje, možete koristiti `useEffect`.
* Pisanje čistih funkcija zahteva malo iskustva, ali otključava moć React paradigme.

</Recap>


  
<Challenges>

#### Popraviti pokvaren sat {/*fix-a-broken-clock*/}

Ova komponenta pokušava da postavi CSS klasu za `<h1>` na `"night"` između pomoći i šest ujutru, a na `"day"` u svim ostalim slučajevima. Međutim, trenutno ne radi. Možete li popraviti ovu komponentu?

Možete proveriti da li vam rešenje radi privremenom promenom vremenske zone na računaru. Kad je vreme između ponoći i šest ujutru, sat bi trebao da ima obrnute boje!

<Hint>

Renderovanje je *proračun*, pa ne bi trebalo da pokuša da "radi" stvari. Možete li izraziti istu ideju drugačije?

</Hint>

<Sandpack>

```js src/Clock.js active
export default function Clock({ time }) {
  const hours = time.getHours();
  if (hours >= 0 && hours <= 6) {
    document.getElementById('time').className = 'night';
  } else {
    document.getElementById('time').className = 'day';
  }
  return (
    <h1 id="time">
      {time.toLocaleTimeString()}
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
  return (
    <Clock time={time} />
  );
}
```

```css
body > * {
  width: 100%;
  height: 100%;
}
.day {
  background: #fff;
  color: #222;
}
.night {
  background: #222;
  color: #fff;
}
```

</Sandpack>

<Solution>

Možete popraviti komponentu računanjem `className`-a i uvrštavanjem u rezultat renderovanja:

<Sandpack>

```js src/Clock.js active
export default function Clock({ time }) {
  const hours = time.getHours();
  let className;
  if (hours >= 0 && hours <= 6) {
    className = 'night';
  } else {
    className = 'day';
  }
  return (
    <h1 className={className}>
      {time.toLocaleTimeString()}
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
  return (
    <Clock time={time} />
  );
}
```

```css
body > * {
  width: 100%;
  height: 100%;
}
.day {
  background: #fff;
  color: #222;
}
.night {
  background: #222;
  color: #fff;
}
```

</Sandpack>

U ovom primeru, propratni efekat (izmena DOM-a) nije uopšte bio potreban. Trebali ste samo da vratite JSX.

</Solution>

#### Popraviti pokvaren profil {/*fix-a-broken-profile*/}

Dve `Profile` komponente su renderovane jedna do druge sa različitim podacima. Pritisnite "Skupi" na prvom profilu, a onda pritisnite "Raširi". Primetićete da oba profila sada prikazuju istu osobu. To je bug.

Pronađite uzrok bug-a i popravite ga.

<Hint>

Kod sa bug-om se nalazi u `Profile.js`. Postarajte se da ga pročitate od početka do kraja!

</Hint>

<Sandpack>

```js src/Profile.js
import Panel from './Panel.js';
import { getImageUrl } from './utils.js';

let currentPerson;

export default function Profile({ person }) {
  currentPerson = person;
  return (
    <Panel>
      <Header />
      <Avatar />
    </Panel>
  )
}

function Header() {
  return <h1>{currentPerson.name}</h1>;
}

function Avatar() {
  return (
    <img
      className="avatar"
      src={getImageUrl(currentPerson)}
      alt={currentPerson.name}
      width={50}
      height={50}
    />
  );
}
```

```js src/Panel.js hidden
import { useState } from 'react';

export default function Panel({ children }) {
  const [open, setOpen] = useState(true);
  return (
    <section className="panel">
      <button onClick={() => setOpen(!open)}>
        {open ? 'Skupi' : 'Raširi'}
      </button>
      {open && children}
    </section>
  );
}
```

```js src/App.js
import Profile from './Profile.js';

export default function App() {
  return (
    <>
      <Profile person={{
        imageId: 'lrWQx8l',
        name: 'Subrahmanyan Chandrasekhar',
      }} />
      <Profile person={{
        imageId: 'MK3eW3A',
        name: 'Creola Katherine Johnson',
      }} />
    </>
  )
}
```

```js src/utils.js hidden
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
.avatar { margin: 5px; border-radius: 50%; }
.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
  width: 200px;
}
h1 { margin: 5px; font-size: 18px; }
```

</Sandpack>

<Solution>

Problem je u tome što `Profile` komponenta piše u već postojeću promenljivu pod imenom `currentPerson`, a `Header` i `Avatar` komponente je čitaju. Ovo *sve tri komponente* čini nečistim i teško predvidljivim.

Da biste popravili bug, uklonite `currentPerson` promenljivu. Umesto toga, prosledite sve informacije iz `Profile` u `Header` i `Avatar` pomoću props-a. Trebaćete da dodate `person` prop u obe komponente i prosledite ga svuda.

<Sandpack>

```js src/Profile.js active
import Panel from './Panel.js';
import { getImageUrl } from './utils.js';

export default function Profile({ person }) {
  return (
    <Panel>
      <Header person={person} />
      <Avatar person={person} />
    </Panel>
  )
}

function Header({ person }) {
  return <h1>{person.name}</h1>;
}

function Avatar({ person }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={50}
      height={50}
    />
  );
}
```

```js src/Panel.js hidden
import { useState } from 'react';

export default function Panel({ children }) {
  const [open, setOpen] = useState(true);
  return (
    <section className="panel">
      <button onClick={() => setOpen(!open)}>
        {open ? 'Skupi' : 'Raširi'}
      </button>
      {open && children}
    </section>
  );
}
```

```js src/App.js
import Profile from './Profile.js';

export default function App() {
  return (
    <>
      <Profile person={{
        imageId: 'lrWQx8l',
        name: 'Subrahmanyan Chandrasekhar',
      }} />
      <Profile person={{
        imageId: 'MK3eW3A',
        name: 'Creola Katherine Johnson',
      }} />
    </>
  );
}
```

```js src/utils.js hidden
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
.avatar { margin: 5px; border-radius: 50%; }
.panel {
  border: 1px solid #aaa;
  border-radius: 6px;
  margin-top: 20px;
  padding: 10px;
  width: 200px;
}
h1 { margin: 5px; font-size: 18px; }
```

</Sandpack>

Upamtite da React ne garantuje da će se funkcije komponenata izvršavati u određenom redosledu, tako da ne možete uspostaviti komunikaciju između njih putem promenljivih. Sva komunikacija se mora izvršavati putem props-a.

</Solution>

#### Popraviti pokvarenu traku sa pričama {/*fix-a-broken-story-tray*/}

CEO vaše kompanije traži od vas da dodate "priče" u vašu aplikaciju i ne možete reći ne. Napisali ste `StoryTray` komponentu koja prima listu `stories`, praćenu sa "Napravi priču" placeholder-om.

Implementirali ste "Napravi priču" placeholder dodavanjem jedne lažne priče na kraj `stories` niza koji primate kao prop. Ali, iz nekog razloga, "Napravi priču" se pojavljuje više od jednom. Popravite problem.

<Sandpack>

```js src/StoryTray.js active
export default function StoryTray({ stories }) {
  stories.push({
    id: 'create',
    label: 'Napravi priču'
  });

  return (
    <ul>
      {stories.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
    </ul>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import StoryTray from './StoryTray.js';

const initialStories = [
  {id: 0, label: "Priča od Ankit" },
  {id: 1, label: "Priča od Taylor" },
];

export default function App() {
  const [stories, setStories] = useState([...initialStories])
  const time = useTime();

  // HACK: Prevent the memory from growing forever while you read docs.
  // We're breaking our own rules here.
  if (stories.length > 100) {
    stories.length = 100;
  }

  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <h2>Sad je tačno {time.toLocaleTimeString()}.</h2>
      <StoryTray stories={stories} />
    </div>
  );
}

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
```

```css
ul {
  margin: 0;
  list-style-type: none;
}

li {
  border: 1px solid #aaa;
  border-radius: 6px;
  float: left;
  margin: 5px;
  margin-bottom: 20px;
  padding: 5px;
  width: 70px;
  height: 100px;
}
```

```js sandbox.config.json hidden
{
  "hardReloadOnChange": true
}
```

</Sandpack>

<Solution>

Primetite da kad god se sat ažurira, "Napravi priču" se doda *dvaput*. Ovo služi kao nagoveštaj da imamo mutaciju tokom renderovanja--Strict Mode poziva komponente dvaput da bi ovakve probleme učinio uočljivijim.

`StoryTray` funkcija nije čista. Pozivanjem `push`-a nad primljenim `stories` nizom (prop-om!), menja se objekat koji je kreiran *pre* nego što se `StoryTray` počeo renderovati. To ga čini pogrešnim i veoma teškim za predviđanje.

Najlakše rešenje je da se ne dira niz uopšte, već da se "Napravi priču" renderuje odvojeno:

<Sandpack>

```js src/StoryTray.js active
export default function StoryTray({ stories }) {
  return (
    <ul>
      {stories.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
      <li>Napravi priču</li>
    </ul>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import StoryTray from './StoryTray.js';

const initialStories = [
  {id: 0, label: "Priča od Ankit" },
  {id: 1, label: "Priča od Taylor" },
];

export default function App() {
  const [stories, setStories] = useState([...initialStories])
  const time = useTime();

  // HACK: Prevent the memory from growing forever while you read docs.
  // We're breaking our own rules here.
  if (stories.length > 100) {
    stories.length = 100;
  }

  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <h2>Sad je tačno {time.toLocaleTimeString()}.</h2>
      <StoryTray stories={stories} />
    </div>
  );
}

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
```

```css
ul {
  margin: 0;
  list-style-type: none;
}

li {
  border: 1px solid #aaa;
  border-radius: 6px;
  float: left;
  margin: 5px;
  margin-bottom: 20px;
  padding: 5px;
  width: 70px;
  height: 100px;
}
```

</Sandpack>

Alternativno, možete napraviti _novi_ niz (kopiranjem postojećeg) pre nego što dodate stavke u njega:

<Sandpack>

```js src/StoryTray.js active
export default function StoryTray({ stories }) {
  // Kopiranje niza!
  const storiesToDisplay = stories.slice();

  // Ne utiče na originalni niz:
  storiesToDisplay.push({
    id: 'create',
    label: 'Napravi priču'
  });

  return (
    <ul>
      {storiesToDisplay.map(story => (
        <li key={story.id}>
          {story.label}
        </li>
      ))}
    </ul>
  );
}
```

```js src/App.js hidden
import { useState, useEffect } from 'react';
import StoryTray from './StoryTray.js';

const initialStories = [
  {id: 0, label: "Priča od Ankit" },
  {id: 1, label: "Priča od Taylor" },
];

export default function App() {
  const [stories, setStories] = useState([...initialStories])
  const time = useTime();

  // HACK: Prevent the memory from growing forever while you read docs.
  // We're breaking our own rules here.
  if (stories.length > 100) {
    stories.length = 100;
  }

  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <h2>Sad je tačno {time.toLocaleTimeString()}.</h2>
      <StoryTray stories={stories} />
    </div>
  );
}

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
```

```css
ul {
  margin: 0;
  list-style-type: none;
}

li {
  border: 1px solid #aaa;
  border-radius: 6px;
  float: left;
  margin: 5px;
  margin-bottom: 20px;
  padding: 5px;
  width: 70px;
  height: 100px;
}
```

</Sandpack>

Ovo čini vašu mutaciju lokalnom, a funkciju čistom. Međutim, i dalje morate biti oprezni: na primer, ako pokušate da menjate bilo koju stavku u postojećem nizu, morate da klonirate i tu stavku takođe.

Korisno je upamtiti koje operacije nad nizovima menjaju niz, a koje ne. Na primer, `push`, `pop`, `reverse` i `sort` će promeniti originalni niz, dok će `slice`, `filter` i `map` kreirati novi niz.

</Solution>

</Challenges>
