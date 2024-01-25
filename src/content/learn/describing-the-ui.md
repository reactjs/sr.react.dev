---
title: Opisivanje korisničkog interfejsa (UI)
---

<Intro>

React je JavaScript biblioteka za prikazivanje korisničkog interfejsa (UI). UI se sastoji od malih jedinica kao što su dugmad, tekst i slike. React vam omogućava da ih kombinujete u ponovljive, nested (ugnježdene) *komponente.* Od veb sajtova do telefonskih aplikacija, sve na ekranu može se razbiti na komponente. U ovoj glavi naučićete kako da kreirate, prilagodite i uslovno prikažete React komponente.

</Intro>

<YouWillLearn isChapter={true}>

* [Kako da napišete svoju prvu React komponentu](/learn/your-first-component)
* [Kada i kako da kreirate više komponenti u jednom fajlu](/learn/importing-and-exporting-components)
* [Kako da dodate markup u JavaScript pomoću JSX](/learn/writing-markup-with-jsx)
* [Kako da koristite vitičaste zagrade u JSX-u da biste pristupili funkcionalnosti JavaScript-a iz vaših komponenti](/learn/javascript-in-jsx-with-curly-braces)
* [Kako da konfigurišete komponente sa props](/learn/passing-props-to-a-component)
* [Kako da uslovno prikažete komponente](/learn/conditional-rendering)
* [Kako da prikažete više komponenti odjednom](/learn/rendering-lists)
* [Kako da izbegnete zbunjujuće greške tako što ćete komponente držati čistim](/learn/keeping-components-pure)
* [Kako da razumete vaš UI kao stablo](/learn/understanding-your-ui-as-a-tree)

</YouWillLearn>

## Vaša prva komponenta {/*your-first-component*/}

React aplikacije su izgrađene of izoliranih delova korisničkog interfejsa (UI) koje se zovu *komponente*. React komponenta je Javascript funkcija koju možete začiniti markup-om. Komponente mogu biti male kao dugme ili velike kao cela stranica. Ovde je `Gallery` komponenta koja prikazuje tri `Profile` komponente:

<Sandpack>

```js
function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3As.jpg"
      alt="Katherine Johnson"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Zadivljujući naučnik</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```css
img { margin: 0 10px 10px 0; height: 90px; }
```

</Sandpack>

<LearnMore path="/learn/your-first-component">

Pročitajte **[Vaša prva komponenta](/learn/your-first-component)** da biste naučili kako da deklarišete i koristite React komponente.

</LearnMore>

## Importovanje i exportovanje komponenti {/*importing-and-exporting-components*/}

Možete deklarisati mnogo komponenti u jednom fajlu, ali veliki fajlovi mogu postati teški za navigaciju. Da biste to rešili, možete *exportovati* komponentu u svoj fajl, a zatim *importovati* tu komponentu iz drugog fajla:


<Sandpack>

```js src/App.js hidden
import Gallery from './Gallery.js';

export default function App() {
  return (
    <Gallery />
  );
}
```

```js src/Gallery.js active
import Profile from './Profile.js';

export default function Gallery() {
  return (
    <section>
      <h1>Zadivljujući naučnik</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```js src/Profile.js
export default function Profile() {
  return (
    <img
      src="https://i.imgur.com/QIrZWGIs.jpg"
      alt="Alan L. Hart"
    />
  );
}
```

```css
img { margin: 0 10px 10px 0; }
```

</Sandpack>

<LearnMore path="/learn/importing-and-exporting-components">

Pročitajte **[Importovanje i exportovanje komponenti](/learn/importing-and-exporting-components)** da biste naučili kako da podelite komponente u svoje fajlove.

</LearnMore>

## Pisanje markup-a sa JSX {/*writing-markup-with-jsx*/}

Svaka React komponenta je JavaScript funkcija koja može sadržati neki markup koji React prikazuje u browser-u. React komponente koriste sintaksu proširenja zvanu JSX da predstave taj markup. JSX izgleda mnogo kao HTML, ali je malo stroži i može prikazati dinamičke informacije.

Ako kopirate postojeći HTML markup u React komponentu, neće uvek raditi:

<Sandpack>

```js
export default function TodoList() {
  return (
    // This doesn't quite work!
    <h1>Hedy Lamarr's Todo lista</h1>
    <img
      src="https://i.imgur.com/yXOvdOSs.jpg"
      alt="Hedy Lamarr"
      class="photo"
    >
    <ul>
      <li>Izmisli nove semafore
      <li>Vežbaj scenu iz filma
      <li>Unapredi tehnologiju spektra
    </ul>
  );
}
```

```css
img { height: 90px; }
```

</Sandpack>

Ako imate postojeći HTML kao što je ovaj, možete ga popraviti pomoću [konvertera](https://transform.tools/html-to-jsx):

<Sandpack>

```js
export default function TodoList() {
  return (
    <>
      <h1>Hedy Lamarr's Todos</h1>
      <img
        src="https://i.imgur.com/yXOvdOSs.jpg"
        alt="Hedy Lamarr"
        className="photo"
      />
      <ul>
        <li>Izmisli nove semafore</li>
        <li>Vežbaj scenu iz filma</li>
        <li>Unapredi tehnologiju spektra</li>
      </ul>
    </>
  );
}
```

```css
img { height: 90px; }
```

</Sandpack>

<LearnMore path="/learn/writing-markup-with-jsx">

Pročitajte **[Pisanje markup-a sa JSX](/learn/writing-markup-with-jsx)** da biste naučili kako da napišete validan JSX.

</LearnMore>

## JavaScript u JSX-u sa vitičastim zagradama {/*javascript-in-jsx-with-curly-braces*/}

JSX vam dozvoljava da pišete HTML sličan markup unutar JavaScript fajla, čuvajući logiku prikazivanja i sadržaja na istom mestu. Ponekad ćete želeti da dodate malo JavaScript logike ili da referencirate dinamičko svojstvo unutar tog markup-a. U ovoj situaciji možete koristiti vitičaste zagrade u vašem JSX-u da "otvorite prozor" ka JavaScript-u:

<Sandpack>

```js
const person = {
  name: 'Gregorio Y. Zara',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name}'s Todos</h1>
      <img
        className="avatar"
        src="https://i.imgur.com/7vQD0fPs.jpg"
        alt="Gregorio Y. Zara"
      />
      <ul>
        <li>Unapredi the videophone</li>
        <li>Pripremi predavanja iz aeronautike</li>
        <li>Radi na motoru koji radi na alkohol</li>
      </ul>
    </div>
  );
}
```

```css
body { padding: 0; margin: 0 }
body > div > div { padding: 20px; }
.avatar { border-radius: 50%; height: 90px; }
```

</Sandpack>

<LearnMore path="/learn/javascript-in-jsx-with-curly-braces">

Pročitajte **[JavaScript u JSX-u sa vitičastim zagradama](/learn/javascript-in-jsx-with-curly-braces)** da biste naučili kako da pristupite JavaScript podacima iz JSX-a.

</LearnMore>

## Prosleđivanje props-a komponenti {/*passing-props-to-a-component*/}

React komponente koriste *props* da bi komunicirale jedna sa drugom. Svaki roditeljski(parent) komponent može proslediti neke informacije svojoj deci(children) pomoću props-a. Props vam mogu podsetiti na HTML atribute, ali možete proslediti bilo koju JavaScript vrednost kroz njih, uključujući objekte, nizove, funkcije i čak JSX!

<Sandpack>

```js
import { getImageUrl } from './utils.js'

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

function Card({ children }) {
  return (
    <div className="card">
      {children}
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

<LearnMore path="/learn/passing-props-to-a-component">

Pročitajte **[Prosleđivanje props-a komponenti](/learn/passing-props-to-a-component)** da biste naučili kako da prosledite i čitate props.

</LearnMore>

## Uslovno prikazivanje komponenti {/*conditional-rendering*/}

Vaše komponente će često morati da prikažu različite stvari u zavisnosti od različitih uslova. U React-u, možete uslovno prikazati JSX koristeći JavaScript sintaksu kao što su `if` naredbe, `&&` i `? :` operatori.

U ovom primeru, JavaScript `&&` operator se koristi da bi se uslovno prikazala kvačica:

<Sandpack>

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked && '✔'}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
    <h1>Sally Ride lista za pakovanje</h1>
      <ul>
        <Item
          isPacked={true}
          name="Svemirsko odelo"
        />
        <Item
          isPacked={true}
          name="Kaciga sa zlatnim listom"
        />
        <Item
          isPacked={false}
          name="Foto od Tam"
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

<LearnMore path="/learn/conditional-rendering">

Pročitajte **[Uslovno prikazivanje komponenti](/learn/conditional-rendering)** da biste naučili različite načine za uslovno prikazivanje sadržaja.

</LearnMore>

## Renderovanje liste {/*rendering-lists*/}

Često ćete želeti da prikažete više sličnih komponenti iz kolekcije podataka. Možete koristiti JavaScript `filter()` i `map()` sa React-om da biste filtrirali i transformisali vaš niz podataka u niz komponenti.

Za svaki element u nizu, moraćete da odredite `key` prop. Obično ćete koristiti ID iz baze podataka kao `key`. Ključevi omogućavaju React-u da prati mesto svakog elementa u listi čak i ako se lista promeni.

<Sandpack>

```js src/App.js
import { people } from './data.js';
import { getImageUrl } from './utils.js';

export default function List() {
  const listItems = people.map(person =>
    <li key={person.id}>
      <img
        src={getImageUrl(person)}
        alt={person.name}
      />
      <p>
        <b>{person.name}:</b>
        {' ' + person.profession + ' '}
        poznat je zbog {person.accomplishment}
      </p>
    </li>
  );
  return (
    <article>
      <h1>Scientists</h1>
      <ul>{listItems}</ul>
    </article>
  );
}
```

```js src/data.js
export const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'mathematician',
  accomplishment: 'formula za svemirske letove',
  imageId: 'MK3eW3A'
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'chemist',
  accomplishment: 'otkriće Arktičke rupe u ozonu',
  imageId: 'mynHUSa'
}, {
  id: 2,
  name: 'Mohammad Abdus Salam',
  profession: 'physicist',
  accomplishment: 'teorija o elektromagnetizmu',
  imageId: 'bE7W1ji'
}, {
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'chemist',
  accomplishment: 'pionirski kortizon, steroide i pilule za kontrolu rađanja',
  imageId: 'IOjWm71'
}, {
  id: 4,
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrophysicist',
  accomplishment: 'računanje mase belog patuljka',
  imageId: 'lrWQx8l'
}];
```

```js src/utils.js
export function getImageUrl(person) {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    's.jpg'
  );
}
```

```css
ul { list-style-type: none; padding: 0px 10px; }
li {
  margin-bottom: 10px;
  display: grid;
  grid-template-columns: 1fr 1fr;
  align-items: center;
}
img { width: 100px; height: 100px; border-radius: 50%; }
h1 { font-size: 22px; }
h2 { font-size: 20px; }
```

</Sandpack>

<LearnMore path="/learn/rendering-lists">

Pročitajte **[Renderovanje liste](/learn/rendering-lists)** da biste naučili kako da renderujete listu komponenti i kako da odaberete ključ.

</LearnMore>

## Održavanje komponenti čistim (Pure components) {/*keeping-components-pure*/}

Neke JavaScript funkcije su *čiste.* Čista funkcija:

* **Gleda svoj posao.** Ne zavisi od bilo kakvih globalnih promenljivih ili stanja aplikacije.
* **Isti input, isti output.** Dajući isti input, čista funkcija uvek treba da vrati isti rezultat.

Striktno pisanje vaših komponenti kao čistih funkcija može da izbegne čitavu klasu zbunjujućih grešaka i nepredvidivog ponašanja kako vaša baza koda raste. Ovde je primer nečiste komponente:


<Sandpack>

```js
let guest = 0;

function Cup() {
  // Bad: changing a preexisting variable!
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

Možete napraviti ovu komponentu čistom tako što ćete proslediti prop umesto što ćete modifikovati prethodno postojeću promenljivu:

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

<LearnMore path="/learn/keeping-components-pure">

Pročitajte **[Održavanje komponenti čistim](/learn/keeping-components-pure)** da biste naučili kako da napišete komponente kao čiste funkcije.

</LearnMore>

## Razumevanje vašeg UI kao stabla {/*understanding-your-ui-as-a-tree*/}

React koristi stabla da modeluje odnose između komponenti i modula.

React render stablo je reprezentacija odnosa roditelj-dete između komponenti.


<Diagram name="generic_render_tree" height={250} width={500} alt="A tree graph with five nodes, with each node representing a component. The root node is located at the top the tree graph and is labelled 'Root Component'. It has two arrows extending down to two nodes labelled 'Component A' and 'Component C'. Each of the arrows is labelled with 'renders'. 'Component A' has a single 'renders' arrow to a node labelled 'Component B'. 'Component C' has a single 'renders' arrow to a node labelled 'Component D'.">

Primer render stabla.

</Diagram>

Komponente koje se nalaze bliže vrhu stabla, bliže korenu komponente, smatraju se komponentama najvišeg nivoa. Komponente bez podkomponenti su list komponente. Ova kategorizacija komponenti je korisna za razumevanje toka podataka i performansi renderovanja.

Modelovanje odnosa između JavaScript modula je još jedan koristan način za razumevanje vaše aplikacije. Mi ga nazivamo modul zavisnosti stablo.

<Diagram name="generic_dependency_tree" height={250} width={500} alt="A tree graph with five nodes. Each node represents a JavaScript module. The top-most node is labelled 'RootModule.js'. It has three arrows extending to the nodes: 'ModuleA.js', 'ModuleB.js', and 'ModuleC.js'. Each arrow is labelled as 'imports'. 'ModuleC.js' node has a single 'imports' arrow that points to a node labelled 'ModuleD.js'.">

Primer modul zavisnosti stabla.

</Diagram>

Drvo zavisnosti čest ose korišćeno od strane alata za izgradnju da bi se sve relevantne JavaScript datoteke za klijenta spakovalo u jednu datoteku. Velika veličina paketa regresira korisničko iskustvo za React aplikacije. Razumevanje modul zavisnosti stabla je korisno za otklanjanje grešaka.

<LearnMore path="/learn/understanding-your-ui-as-a-tree">

Pročitajte **[Razumevanje vašeg UI kao stabla](/learn/understanding-your-ui-as-a-tree)** da biste naučili kako da kreirate render i modul zavisnosti stabla za React aplikaciju i kako su korisni mentalni modeli za poboljšanje korisničkog iskustva i performansi.

</LearnMore>


## Šta dalje? {/*whats-next*/}

Idite do [Vaša prva komponenta](/learn/your-first-component) da biste počeli da čitate ovo poglavlje stranicu po stranicu!

Ili ako ste već upoznati sa ovim temama, zašto ne biste pročitali o [Dodavanju interaktivnosti](/learn/adding-interactivity)?

