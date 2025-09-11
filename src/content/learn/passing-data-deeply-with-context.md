---
title: Prosleđivanje podataka duboko uz Context
---

<Intro>

Često ćete proslediti informaciju od roditeljske ka dečjoj komponenti kroz props. Ali, prosleđivanje props-a može postati opširno i nepogodno ako ih trebate proslediti kroz mnogo komponenata u sredini ili ako mnogo komponenata treba imati istu informaciju. *Context* omogućava roditeljskoj komponenti da neku informaciju učini dostupnom svakoj komponenti u stablu ispod nje—bez obzira koliko duboko—bez eksplicitnog prosleđivanja props-a.

</Intro>

<YouWillLearn>

- Šta je to "prop drilling"
- Kako da zamenite ponavljajuće prosleđivanje prop-a uz context
- Uobičajene slučajeve korišćenja context-a
- Uobičajene alternative za context

</YouWillLearn>

## Problem sa prosleđivanjem props-a {/*the-problem-with-passing-props*/}

[Prosleđivanje props-a](/learn/passing-props-to-a-component) je sjajan način da usmerite podatke kroz vaš UI do komponenata koje ga koriste.

Ali, prosleđivanje props-a može postati opširno i nepogodno ako neki prop trebate proslediti duboko u stablo ili ako je isti prop potreban u mnogo komponenata. Najbliži zajednički roditelj može biti veoma udaljen od komponenata kojima su potrebni podaci, pa [podizanje state-a](/learn/sharing-state-between-components) toliko visoko može dovesti do situacije pod imenom "prop drilling".

<DiagramGroup>

<Diagram name="passing_data_lifting_state" height={160} width={608} captionPosition="top" alt="Dijagram sa stablom od tri komponente. Roditelj sadrži balon koji predstavlja vrednost označenu ljubičastom bojom. Vrednost se prostire na dole ka obe dečje komponente koje su označene ljubičasto." >

Podizanje state-a

</Diagram>
<Diagram name="passing_data_prop_drilling" height={430} width={608} captionPosition="top" alt="Dijagram sa stablom od deset čvorova, gde svaki čvor ima dva deteta ili manje. Korenski čvor sadrži balon koji predstavlja vrednost označenu ljubičastom bojom. Vrednost se prostire na dole ka obe dečje komponente, a svaka dečja komponenta je prosleđuje i ne sadrži je. Levi čvor prosleđuje vrednost u dve dečje komponente koje su označene ljubičasto. Desno dete korenskog čvora prosleđuje vrednost kroz jedno od svoje dvoje dece - desno, koje je označeno ljubičasto. To dete prosleđuje vrednost jedinom detetu, koje je onda prosleđuje u oba svoja deteta, koja su označena ljubičastom bojom.">

Prop drilling

</Diagram>

</DiagramGroup>

Zar ne bi bilo odlično da postoji način da "teleportujete" podatke u komponente unutar stabla kojima su potrebni bez prosleđivanja props-a? Uz React-ovu context funkcionalnost, postoji!

## Context: alternativa za prosleđivanje props-a {/*context-an-alternative-to-passing-props*/}

Context omogućava roditeljskoj komponenti da pruži podatke celokupnom stablu ispod. Postoji mnogo načina za upotrebu context-a. Evo jednog primera. Razmotrite ovu `Heading` komponentu koja prima `level` za svoju veličinu:

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>Title</Heading>
      <Heading level={2}>Heading</Heading>
      <Heading level={3}>Sub-heading</Heading>
      <Heading level={4}>Sub-sub-heading</Heading>
      <Heading level={5}>Sub-sub-sub-heading</Heading>
      <Heading level={6}>Sub-sub-sub-sub-heading</Heading>
    </Section>
  );
}
```

```js src/Section.js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

```js src/Heading.js
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Nepoznat nivo: ' + level);
  }
}
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Recimo da želite da više heading-a unutar jednog `Section`-a uvek ima istu veličinu:

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>Title</Heading>
      <Section>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Section>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Section>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

```js src/Heading.js
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Nepoznat nivo: ' + level);
  }
}
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Trenutno, prosleđujete `level` prop u svaki `<Heading>` pojedinačno:

```js
<Section>
  <Heading level={3}>O aplikaciji</Heading>
  <Heading level={3}>Slike</Heading>
  <Heading level={3}>Videi</Heading>
</Section>
```

Bilo bi dobro da možete proslediti `level` prop u `<Section>` komponentu i uklonite ga iz `<Heading>`-a. Na ovaj način možete osigurati da će svi headings-i unutar jednog section-a uvek imati istu veličinu:

```js
<Section level={3}>
  <Heading>O aplikaciji</Heading>
  <Heading>Slike</Heading>
  <Heading>Videi</Heading>
</Section>
```

Ali, kako `<Heading>` komponenta može da zna nivo najbližeg `<Section>`-a? **Za to je potreban način da dete "pita" za podatke koji se nalaze negde iznad u stablu.**

To ne možete uraditi samo uz props. Ovde context ulazi u igru. Uradićete to u tri koraka:

1. **Napraviti** context. (Možete ga nazvati `LevelContext`, jer predstavlja nivo naslova.)
2. **Iskoristiti** taj context u komponenti kojoj su potrebni podaci. (`Heading` će koristiti `LevelContext`.)
3. **Pružiti** taj context iz komponente koja specificira podatke. (`Section` će pružiti `LevelContext`.)

Context omogućava roditelju--čak i veoma udaljenom!--da pruži neke podatke celokupnom stablu unutar njega.

<DiagramGroup>

<Diagram name="passing_data_context_close" height={160} width={608} captionPosition="top" alt="Dijagram sa stablom od tri komponentes. Roditelj sadrži balon koji predstavlja vrednost označenu narandžastom bojom koja se projektuje na dva deteta, oba označena narandžasto." >

Upotreba context-a u bliskoj deci

</Diagram>

<Diagram name="passing_data_context_far" height={430} width={608} captionPosition="top" alt="Dijagram sa stablom od deset čvorova, gde svaki čvor ima dva deteta ili manje. Korenski čvor sadrži balon koji predstavlja vrednost označenu narandžastom bojom. Vrednost se projektuje direktno na dole ka četiri lista i jednoj središnjoj komponenti u stablu, i svi su označeni narandžasto. Nijedna druga središnja komponenta nije označena.">

Upotreba context-a u udaljenoj deci

</Diagram>

</DiagramGroup>

### Korak 1: Napraviti context {/*step-1-create-the-context*/}

Prvo, potrebno je da napravite context. Moraćete da ga **export-ujete iz fajla** kako bi ga vaše komponente mogle koristiti:

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>Title</Heading>
      <Section>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Heading level={2}>Heading</Heading>
        <Section>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Heading level={3}>Sub-heading</Heading>
          <Section>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
            <Heading level={4}>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

```js src/Heading.js
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Nepoznat nivo: ' + level);
  }
}
```

```js src/LevelContext.js active
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Jedini argument u `createContext` je _podrazumevana_ vrednost. Ovde se `1` odnosi na najviši nivo naslova, ali možete proslediti bilo koju vrednost (čak i objekat). Videćete značaj podrazumevane vrednosti u narednom koraku.

### Korak 2: Iskoristiti context {/*step-2-use-the-context*/}

Import-ujte `useContext` Hook iz React-a, kao i vaš context:

```js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';
```

Trenutno, `Heading` komponenta čita `level` iz props-a:

```js
export default function Heading({ level, children }) {
  // ...
}
```

Umesto toga, uklonite `level` prop i čitajte vrednost iz context-a koji ste upravo import-ovali, `LevelContext`:

```js {2}
export default function Heading({ children }) {
  const level = useContext(LevelContext);
  // ...
}
```

`useContext` je Hook. Kao i `useState` i `useReducer`, možete pozvati Hook samo odmah unutar React komponente (ne unutar petlji i uslova). **`useContext` govori React-u da `Heading` komponenta želi da čita `LevelContext`.**

Sad kada `Heading` komponenta nema `level` prop, ne morate više prosleđivati level prop u `Heading` iz vašeg JSX-a:

```js
<Section>
  <Heading level={4}>Sub-sub-heading</Heading>
  <Heading level={4}>Sub-sub-heading</Heading>
  <Heading level={4}>Sub-sub-heading</Heading>
</Section>
```

Ažurirajte JSX tako da taj prop prima `Section`:

```jsx
<Section level={4}>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
  <Heading>Sub-sub-heading</Heading>
</Section>
```

Kao podsetnik, ovo je markup koji želite da napravite:

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}>
      <Heading>Title</Heading>
      <Section level={2}>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section level={3}>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section level={4}>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

```js src/Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Nepoznat nivo: ' + level);
  }
}
```

```js src/LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Primetite da ovaj primer baš i ne radi, još uvek! Svi headings-i imaju istu veličinu jer **iako *koristite* context, još uvek ga niste *pružili***. React ne zna gde da ga dobije!

Ako ne pružite context, React će koristiti podrazumevanu vrednost koju ste specificirali u prethodnom koraku. U ovom primeru, specificirali ste `1` kao argument u `createContext`, pa `useContext(LevelContext)` vraća `1`, postavljajući sve headings-e na `<h1>`. Popravimo ovaj problem tako što će svaki `Section` pružiti svoj context.

### Korak 3: Pružiti context {/*step-3-provide-the-context*/}

`Section` komponenta trenutno renderuje svoju decu:

```js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

**Obmotajte ih sa context provider-om** da im pružite `LevelContext`:

```js {1,6,8}
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext value={level}>
        {children}
      </LevelContext>
    </section>
  );
}
```

Ovo govori React-u: "Ako bilo koja komponenta unutar ovog `<Section>`-a pita za `LevelContext`, daj joj ovaj `level`." Komponenta će koristiti najbliži `<LevelContext>` u stablu iznad.

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}>
      <Heading>Title</Heading>
      <Section level={2}>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section level={3}>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section level={4}>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext value={level}>
        {children}
      </LevelContext>
    </section>
  );
}
```

```js src/Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Nepoznat nivo: ' + level);
  }
}
```

```js src/LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Rezultat je isti kao i u originalnom kodu, ali niste morali da prosleđujete `level` prop u svaku `Heading` komponentu! Umesto toga, komponenta "shvata" svoj nivo naslova tako što pita najbliži `Section` iznad:

1. Prosleđujete `level` prop u `<Section>`.
2. `Section` obmotava svoju decu sa `<LevelContext value={level}>`.
3. `Heading` pita za najbližu vrednost `LevelContext`-a iznad sa `useContext(LevelContext)`.

## Upotreba i pružanje context-a iz iste komponente {/*using-and-providing-context-from-the-same-component*/}

Trenutno i dalje morate da specificirate `level` svakog section-a ručno:

```js
export default function Page() {
  return (
    <Section level={1}>
      ...
      <Section level={2}>
        ...
        <Section level={3}>
          ...
```

Pošto vam context omogućava da čitate informaciju iz komponente iznad, `Section` bi mogao da čita `level` iz `Section`-a iznad i da prosledi `level + 1` na dole automatski. Evo kako to možete uraditi:

```js src/Section.js {5,8}
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext value={level + 1}>
        {children}
      </LevelContext>
    </section>
  );
}
```

Sa ovom promenom, ne morate prosleđivati `level` prop *ni* u `<Section>` ni u `<Heading>`:

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading>Title</Heading>
      <Section>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext value={level + 1}>
        {children}
      </LevelContext>
    </section>
  );
}
```

```js src/Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 0:
      throw Error('Heading mora biti unutar Section-a!');
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Nepoznat nivo: ' + level);
  }
}
```

```js src/LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(0);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

Sada i `Heading` i `Section` čitaju `LevelContext` kako bi shvatili koliko su "duboko". A `Section` obmotava svoju decu sa `LevelContext` da bi specificirao da sve što je unutar njega pripada "dubljem" nivou.

<Note>

Ovaj primer koristi nivoe naslova jer oni vizuelno prikazuju kako ugnježdene komponente mogu da override-uju context. Ali, context je koristan i u mnogim drugim slučajevima. Možete proslediti ispod bilo koju informaciju koja je potrebna celokupnom podstablu: trenutnu boju teme, trenutno ulogovanog korisnika, itd.

</Note>

## Context prolazi kroz središnje komponente {/*context-passes-through-intermediate-components*/}

Možete ubaciti koliko god želite komponenata između komponente koja pruža i one koja koristi context. Ovo uključuje i ugrađene komponente poput `<div>`-a i komponente koje sami pravite.

U ovom primeru, ista `Post` komponenta (sa isprekidanom ivicom) je renderovana na dva različita ugnježdena nivoa. Primetite da `<Heading>` unutar nje automatski dobija svoj nivo od najbližeg `<Section>`-a:

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function ProfilePage() {
  return (
    <Section>
      <Heading>Moj profil</Heading>
      <Post
        title="Pozdrav putniče!"
        body="Čitaj o mojim avanturama."
      />
      <AllPosts />
    </Section>
  );
}

function AllPosts() {
  return (
    <Section>
      <Heading>Objave</Heading>
      <RecentPosts />
    </Section>
  );
}

function RecentPosts() {
  return (
    <Section>
      <Heading>Nedavne objave</Heading>
      <Post
        title="Ukusi Lisabona"
        body="...taj pastéis de nata!"
      />
      <Post
        title="Buenos Ajres u ritmu tanga"
        body="Obožavao sam to!"
      />
    </Section>
  );
}

function Post({ title, body }) {
  return (
    <Section isFancy={true}>
      <Heading>
        {title}
      </Heading>
      <p><i>{body}</i></p>
    </Section>
  );
}
```

```js src/Section.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children, isFancy }) {
  const level = useContext(LevelContext);
  return (
    <section className={
      'section ' +
      (isFancy ? 'fancy' : '')
    }>
      <LevelContext value={level + 1}>
        {children}
      </LevelContext>
    </section>
  );
}
```

```js src/Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 0:
      throw Error('Heading mora biti unutar Section-a!');
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Nepoznat nivo: ' + level);
  }
}
```

```js src/LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(0);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}

.fancy {
  border: 4px dashed pink;
}
```

</Sandpack>

Niste učinili ništa posebno da bi ovo radilo. `Section` specificira context za stablo unutar njega, tako da možete ubaciti `<Heading>` bilo gde i imaće ispravnu veličinu. Probajte u sandbox-u iznad!

**Context vam omogućava da pišete komponente koje se "prilagođavaju svom okruženju" i prikazuju sebe drugačije u zavisnosti _gde_ (ili, drugim rečima, _u kom context-u_) se renderuju.**

Način funkcionisanja context-a vas može podsetiti na [CSS polje inheritance](https://developer.mozilla.org/en-US/docs/Web/CSS/inheritance). U CSS-u, možete specificirati `color: blue` za `<div>` i svaki DOM čvor unutar, bez obzira koliko duboko, će naslediti tu boju, osim ako je neki drugi DOM čvor u sredini ne override-uje sa `color: green`. Slično tome, u React-u, jedini način da se override-uje context koji dolazi od gore je da se deca obmotaju sa context provider-om koji ima drugačiju vrednost.

U CSS-u, različita polja poput `color` i `background-color` ne override-uju jedno drugo. Možete postaviti `color` svih `<div>`-ova na crveno bez da utičete na `background-color`. Slično tome, **različiti React context-i ne override-uju jedan drugog**. Svaki context koji napravite sa `createContext()` je potpuno odvojen od ostalih i povezuje komponente koje koriste i pružaju *taj specifični* context. Jedna komponenta, bez problema, može koristiti ili pružati neograničen broj context-a.

## Pre nego što upotrebite context {/*before-you-use-context*/}

Context je veoma primamljiv za upotrebu! Međutim, ovo takođe znači da je lako preterati sa njegovom upotrebom. **To što je potrebno proslediti neke props-e par nivoa u dubinu ne znači da biste trebali tu informaciju staviti u context.**

Evo par alternativa koje biste trebali razmotriti pre upotrebe context-a:

1. **Počnite sa [prosleđivanjem props-a](/learn/passing-props-to-a-component).** Ako vaše komponente nisu trivijalne, nije neuobičajeno da prosleđujete nekolicinu props-a kroz nekolicinu komponenata. Možda deluje naporno, ali jasno govori koje komponente koriste koje podatke! Osoba koja bude održavala vaš kod će biti srećna da ste eksplicitno označili tok podataka kroz props.
2. **Izdvojite komponente i [prosledite im JSX kao `children`](/learn/passing-props-to-a-component#passing-jsx-as-children).** Ako prosleđujete podatke kroz mnogo slojeva središnjih komponenata koje ne koriste te podatke (i samo ih prosleđuju na dole), to obično znači da ste zaboravili da izdvojite neke komponente. Na primer, možda prosleđujete props-e poput `posts` u vizuelne komponente koje ih ne koriste direktno, npr. `<Layout posts={posts} />`. Umesto toga, napravite da `Layout` prima `children` kao prop i renderujte `<Layout><Posts posts={posts} /></Layout>`. Ovo smanjuje broj slojeva između komponente koja specificira podatke i one kojoj su podaci potrebni.

Ako nijedan od ovih pristupa ne radi dobro, razmotrite context.

## Slučajevi korišćenja context-a {/*use-cases-for-context*/}

* **Theming:** Ako vaša aplikacija dozvoljava korisniku da menja izgled (npr. tamni režim), možete postaviti context provider na vrh aplikacije i koristiti taj context u komponentama koje trebaju menjati svoj izgled.
* **Trenutni nalog:** Mnogim komponentama može biti potreban trenutno ulogovani korisnik. Upotreba context-a čini čitanje podataka zgodnijim bilo gde u stablu. Neke aplikacije omogućavaju istovremeno korišćenje više naloga (npr. ostavljanje komentara u ime drugog korisnika). U tim slučajevima može biti zgodno da obmotate deo UI-a u ugnježdeni provider sa drugačijom vrednošću za trenutni nalog.
* **Rutiranje:** Većina rešenja za rutiranje interno koristi context da čuva trenutnu rutu. Zbog toga svaki link "zna" da li je aktivan ili ne. Ako pravite sopstveni ruter, verovatno ćete i vi želeti ovo da koristite.
* **Upravljanje state-om:** Kako vam aplikacija raste, možete završiti sa mnogo state-ova koji su blizu vrha aplikacije. Mnoge komponente ispod mogu želeti da ih promene. Za upravljanje kompleksnim state-ovima uobičajeno je da se [koristi reducer zajedno sa context-om](/learn/scaling-up-with-reducer-and-context), kako bi se podaci prosledili udaljenim komponentama bez mnogo muke.
  
Context nije ograničen na statičke vrednosti. Ako prosledite drugačiju vrednost u narednom renderu, React će ažurirati sve komponente ispod koje ga čitaju! Zbog toga se context često koristi u kombinaciji sa state-om.

U suštini, ako je neka informacija potrebna udaljenim komponentama u različitim delovima stabla, dobar je znak da vam context može pomoći.

<Recap>

* Context omogućava komponenti da pruži neku informaciju u celo stablo ispod.
* Da biste prosledili context:
  1. Napravite i export-ujte ga sa `export const MyContext = createContext(defaultValue)`.
  2. Prosledite ga u `useContext(MyContext)` Hook kako biste mogli da ga čitate u bilo kojoj dečjoj komponenti, bez obzira koliko duboko bila.
  3. Da biste context pružili iz roditelja, obmotajte decu sa `<MyContext value={...}>`.
* Context prolazi kroz bilo koju središnju komponentu.
* Context vam omogućava da pišete komponente koje se "prilagođavaju svom okruženju".
* Pre nego što upotrebite context, probajte da prosledite props ili da prosledite JSX kao `children`.

</Recap>

<Challenges>

#### Zameniti prop drilling sa context-om {/*replace-prop-drilling-with-context*/}

U ovom primeru, klik na checkbox menja `imageSize` prop prosleđen u svaki `<PlaceImage>`. Checkbox state se čuva na vrhu, u `App` komponenti, ali svaki `<PlaceImage>` mora da zna za njega.

Trenutno, `App` prosleđuje `imageSize` u `List`, koji ga prosleđuje u svaki `Place`, koji ga prosleđuje u `PlaceImage`. Uklonite `imageSize` prop i umesto toga informaciju prosledite direktno iz `App` komponente u `PlaceImage`.

Možete deklarisati context u `Context.js`.

<Sandpack>

```js src/App.js
import { useState } from 'react';
import { places } from './data.js';
import { getImageUrl } from './utils.js';

export default function App() {
  const [isLarge, setIsLarge] = useState(false);
  const imageSize = isLarge ? 150 : 100;
  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isLarge}
          onChange={e => {
            setIsLarge(e.target.checked);
          }}
        />
        Koristi veće slike
      </label>
      <hr />
      <List imageSize={imageSize} />
    </>
  )
}

function List({ imageSize }) {
  const listItems = places.map(place =>
    <li key={place.id}>
      <Place
        place={place}
        imageSize={imageSize}
      />
    </li>
  );
  return <ul>{listItems}</ul>;
}

function Place({ place, imageSize }) {
  return (
    <>
      <PlaceImage
        place={place}
        imageSize={imageSize}
      />
      <p>
        <b>{place.name}</b>
        {': ' + place.description}
      </p>
    </>
  );
}

function PlaceImage({ place, imageSize }) {
  return (
    <img
      src={getImageUrl(place)}
      alt={place.name}
      width={imageSize}
      height={imageSize}
    />
  );
}
```

```js src/Context.js

```

```js src/data.js
export const places = [{
  id: 0,
  name: 'Bo-Kaap u Kejptaunu, Južna Afrika',
  description: 'Tradicija odabira svetlih boja za kuće je započeta krajem 20. veka.',
  imageId: 'K9HVAGH'
}, {
  id: 1,
  name: 'Selo duge u Taichung-u, Tajvan',
  description: 'Kako bi spasio kuće od rušenja, lokalni stanovnik Huang Yung-Fu, 1924. godine je ofarbao svih 1,200 kuća.',
  imageId: '9EAYZrt'
}, {
  id: 2,
  name: 'Macromural de Pachuca, Meksiko',
  description: 'Jedan od najvećih murala na svetu koji pokriva brdsko naselje.',
  imageId: 'DgXHVwu'
}, {
  id: 3,
  name: 'Stepenište Selarón u Rio de Žaneiru, Brazil',
  description: 'Ovu znamenitost je napravio Jorge Selarón, čileanski umetnik, kao "počast brazilskom narodu".',
  imageId: 'aeO3rpI'
}, {
  id: 4,
  name: 'Burano, Italija',
  description: 'Kuće su ofarbane specifičnim sistemom boja koji datira iz 16. veka.',
  imageId: 'kxsph5C'
}, {
  id: 5,
  name: 'Chefchaouen, Maroko',
  description: 'Postoji par teorija zašto su kuće ofarbane u plavo, od toga da ta boja odbija komarce do toga da označava nebo i raj.',
  imageId: 'rTqKo46'
}, {
  id: 6,
  name: 'Selo kulture Gamcheon u Busanu, Južna Koreja',
  description: 'Selo je 2009. godine pretvoreno u kulturni centar farbanjem kuća i postavljanjem izložbi i umetničkih instalacija.',
  imageId: 'ZfQOOzf'
}];
```

```js src/utils.js
export function getImageUrl(place) {
  return (
    'https://i.imgur.com/' +
    place.imageId +
    'l.jpg'
  );
}
```

```css
ul { list-style-type: none; padding: 0px 10px; }
li { 
  margin-bottom: 10px; 
  display: grid; 
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
```

</Sandpack>

<Solution>

Uklonite `imageSize` prop iz svih komponenata.

Napravite i export-ujte `ImageSizeContext` iz `Context.js`. Onda, obmotajte List sa `<ImageSizeContext value={imageSize}>` kako biste vrednost prosledili na dole, a onda iskoristite `useContext(ImageSizeContext)` da ga pročitate u `PlaceImage`:

<Sandpack>

```js src/App.js
import { useState, useContext } from 'react';
import { places } from './data.js';
import { getImageUrl } from './utils.js';
import { ImageSizeContext } from './Context.js';

export default function App() {
  const [isLarge, setIsLarge] = useState(false);
  const imageSize = isLarge ? 150 : 100;
  return (
    <ImageSizeContext
      value={imageSize}
    >
      <label>
        <input
          type="checkbox"
          checked={isLarge}
          onChange={e => {
            setIsLarge(e.target.checked);
          }}
        />
        Koristi veće slike
      </label>
      <hr />
      <List />
    </ImageSizeContext>
  )
}

function List() {
  const listItems = places.map(place =>
    <li key={place.id}>
      <Place place={place} />
    </li>
  );
  return <ul>{listItems}</ul>;
}

function Place({ place }) {
  return (
    <>
      <PlaceImage place={place} />
      <p>
        <b>{place.name}</b>
        {': ' + place.description}
      </p>
    </>
  );
}

function PlaceImage({ place }) {
  const imageSize = useContext(ImageSizeContext);
  return (
    <img
      src={getImageUrl(place)}
      alt={place.name}
      width={imageSize}
      height={imageSize}
    />
  );
}
```

```js src/Context.js
import { createContext } from 'react';

export const ImageSizeContext = createContext(500);
```

```js src/data.js
export const places = [{
  id: 0,
  name: 'Bo-Kaap u Kejptaunu, Južna Afrika',
  description: 'Tradicija odabira svetlih boja za kuće je započeta krajem 20. veka.',
  imageId: 'K9HVAGH'
}, {
  id: 1,
  name: 'Selo duge u Taichung-u, Tajvan',
  description: 'Kako bi spasio kuće od rušenja, lokalni stanovnik Huang Yung-Fu, 1924. godine je ofarbao svih 1,200 kuća.',
  imageId: '9EAYZrt'
}, {
  id: 2,
  name: 'Macromural de Pachuca, Meksiko',
  description: 'Jedan od najvećih murala na svetu koji pokriva brdsko naselje.',
  imageId: 'DgXHVwu'
}, {
  id: 3,
  name: 'Stepenište Selarón u Rio de Žaneiru, Brazil',
  description: 'This landmark was created by Jorge Selarón, a Chilean-born artist, as a "tribute to the Brazilian people".',
  imageId: 'aeO3rpI'
}, {
  id: 4,
  name: 'Burano, Italija',
  description: 'Kuće su ofarbane specifičnim sistemom boja koji datira iz 16. veka.',
  imageId: 'kxsph5C'
}, {
  id: 5,
  name: 'Chefchaouen, Maroko',
  description: 'Postoji par teorija zašto su kuće ofarbane u plavo, od toga da ta boja odbija komarce do toga da označava nebo i raj.',
  imageId: 'rTqKo46'
}, {
  id: 6,
  name: 'Selo kulture Gamcheon u Busanu, Južna Koreja',
  description: 'Selo je 2009. godine pretvoreno u kulturni centar farbanjem kuća i postavljanjem izložbi i umetničkih instalacija.',
  imageId: 'ZfQOOzf'
}];
```

```js src/utils.js
export function getImageUrl(place) {
  return (
    'https://i.imgur.com/' +
    place.imageId +
    'l.jpg'
  );
}
```

```css
ul { list-style-type: none; padding: 0px 10px; }
li { 
  margin-bottom: 10px; 
  display: grid; 
  grid-template-columns: auto 1fr;
  gap: 20px;
  align-items: center;
}
```

</Sandpack>

Primetite da središnje komponente više ne moraju da prosleđuju `imageSize`.

</Solution>

</Challenges>
