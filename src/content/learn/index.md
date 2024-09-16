---
title: Brzi Uvod
---

<Intro>

Dobro došli u React dokumentaciju! Ova stranica pružiće vam uvod u 80% React koncepata koje ćete koristiti u svakodnevnom radu.

</Intro>

<YouWillLearn>

- Kako kreirati i umetati komponente
- Kako dodavati markup i style-ove
- Kako prikazivati podatke
- Kako renderovati kondicione izraze i liste
- Kako odgovarati na event-e i ažurirati prikaz na ekranu
- Kako prosleđivati podatke među komponentama

</YouWillLearn>

## Kreiranje i umetanje komponenti {/*components*/}

React aplikacije su sačinjene od *komponenti*. Komponenta je deo UI-a (korisničkog interfejsa) koji ima svoju logiku i izgled. Komponenta može biti mala kao dugme ili velika kao cela stranica.

React komponente su JavaScript funkcije koje vraćaju markup:

```js
function MyButton() {
  return (
    <button>Ja sam dugme</button>
  );
}
```

Sada kada ste deklarisali `MyButton`, možete ga umetnuti unutar druge komponente:

```js {5}
export default function MyApp() {
  return (
    <div>
      <h1>Dobrodošli u moj app</h1>
      <MyButton />
    </div>
  );
}
```

Primetite da `<MyButton />` počinje velikim slovom. Tako znate da je to React komponenta. Nazivi React komponenti uvek moraju počinjati velikim slovom, dok HTML tagovi moraju biti pisani malim slovima.

Pogledajte rezultat:

<Sandpack>

```js
function MyButton() {
  return (
    <button>
      Ja sam dugme
    </button>
  );
}

export default function MyApp() {
  return (
    <div>
      <h1>Dobrodošli u moj app</h1>
      <MyButton />
    </div>
  );
}
```

</Sandpack>

Ključne reči `export default` određuju glavnu komponentu u fajlu. Ukoliko niste upoznati sa nekim delom JavaScript sintakse, [MDN](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export) i [javascript.info](https://javascript.info/import-export) imaju odlične reference.

## Pisanje markup-a sa JSX {/*writing-markup-with-jsx*/}

Sintaksa markup-a koju ste videli iznad se zove *JSX*. Ona nije obavezna, ali većina React projekata koristi JSX zbog njegove praktičnosti. Svi [alati koje preporučujemo za lokalni razvoj](/learn/installation) podržavaju JSX odmah po instalaciji.

JSX je striktniji od HTML-a. Morate zatvoriti tagove poput `<br />`. Vaša komponenta takođe ne može vraćati više JSX tagova. Morate ih obuhvatiti zajedničkim parent-om, poput `<div>...</div>` ili praznog `<>...</>` omotača:

```js {3,6}
function AboutPage() {
  return (
    <>
      <h1>O nama</h1>
      <p>Zdravo<br />Kako se ste?</p>
    </>
  );
}
```

Ukoliko imate mnogo HTML-a koje treba preneti u JSX, možete koristiti [online konventor.](https://transform.tools/html-to-jsx)

## Dodavanje style-ova {/*adding-styles*/}

U React-u, CSS klasu specificirate sa `className`. To funkcioniše na isti način kao HTML [`class`](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/class) atribut:

```js
<img className="avatar" />
```

Potom CSS pravila za zadatu klasu pišete u odvojenom CSS fajlu:

```css
/* U vašem CSS-u */
.avatar {
  border-radius: 50%;
}
```

React ne propisuje kako dodajete CSS fajlove. U najjednostavnijem slučaju, dodajete [`<link>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link) tag u vaš HTML. Ako koristite build alat ili framework, konsultujte dokumentaciju istog, da biste saznali kako da dodate CSS fajl u vaš projekat.

## Prikazivanje podataka {/*displaying-data*/}

JSX vam omogućava da ubacite markup u JavaScript. Kovrdžave zagrade vam omogućavaju da se "prebacite nazad" u JavaScript tako da možete ugraditi neku varijablu iz vašeg koda i prikazati je korisniku. Na primer, ovo će prikazati `user.name`:

```js {3}
return (
  <h1>
    {user.name}
  </h1>
);
```

Takođe možete se "prebaciti u JavaScript" iz JSX atributa, ali morate koristiti kovrdžave zagrade *umesto* navodnika. Na primer, `className="avatar"` prosleđuje `"avatar"` string kao CSS klasu, ali `src={user.imageUrl}` čita vrednost JavaScript `user.imageUrl` varijable, a zatim tu vrednost prosleđuje kao `src` atribut:

```js {3,4}
return (
  <img
    className="avatar"
    src={user.imageUrl}
  />
);
```

Možete staviti i složenije izraze unutar JSX kovrdžavih zagrada, na primer, [konkatenaciju stringova](https://javascript.info/operators#string-concatenation-with-binary):

<Sandpack>

```js
const user = {
  name: 'Hedy Lamarr',
  imageUrl: 'https://i.imgur.com/yXOvdOSs.jpg',
  imageSize: 90,
};

export default function Profile() {
  return (
    <>
      <h1>{user.name}</h1>
      <img
        className="avatar"
        src={user.imageUrl}
        alt={'Fotografija od ' + user.name}
        style={{
          width: user.imageSize,
          height: user.imageSize
        }}
      />
    </>
  );
}
```

```css
.avatar {
  border-radius: 50%;
}

.large {x
  border: 4px solid gold;
}
```

</Sandpack>

U gore navedenom primeru, `style={{}}` nije posebna sintaksa, već običan `{}` objekat unutar `style={ }` JSX kovrdžavih zagrada. Možete koristiti `style` atribut kada se vaši style-ovi oslanjaju na JavaScript varijable.

## Kondicionalno renderovanje {/*conditional-rendering*/}

U React-u nema posebne sintakse za pisanje kondicionih izraza. Umesto toga, koristićete iste tehnike kao kada pišete običan JavaScript kod. Na primer, možete koristiti [`if`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else) izraz za kondicionalno uključivanje JSX-a:

```js
let content;
if (isLoggedIn) {
  content = <AdminPanel />;
} else {
  content = <LoginForm />;
}
return (
  <div>
    {content}
  </div>
);
```

Ako preferirate kompaktniji kod, možete koristiti [kondicionalni `?` operator.](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) Za razliku od `if`, on radi unutar JSX-a:

```js
<div>
  {isLoggedIn ? (
    <AdminPanel />
  ) : (
    <LoginForm />
  )}
</div>
```

Kada vam nije potrebna `else` grana, možete koristiti i kraću [logičku `&&` sintaksu](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND#short-circuit_evaluation):

```js
<div>
  {isLoggedIn && <AdminPanel />}
</div>
```

Svi ovi pristupi takođe rade i za kondicionalno specificiranje atributa. Ako niste upoznati sa ovim delovima JavaScript sintakse, možete početi tako što ćete uvek koristiti `if...else`.

## Renderovanje listi {/*rendering-lists*/}

Oslanjaćete se na JavaScript funkcionalnosti poput [`for` loop-a](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for) i [array `map()` funkcije](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)  za renderovanje listi komponenata.

Na primer, pretpostavimo da imate niz proizvoda:

```js
const products = [
  { title: 'Kupus', id: 1 },
  { title: 'Luk', id: 2 },
  { title: 'Jabuka', id: 3 },
];
```

Unutar vaše komponente, koristite `map()`  funkciju da transformišete niz proizvoda u niz `<li>` stavki:

```js
const listItems = products.map(product =>
  <li key={product.id}>
    {product.title}
  </li>
);

return (
  <ul>{listItems}</ul>
);
```

Primetite kako `<li>` poseduje `key` atribut. Za svaku stavku u listi, trebalo bi da prosledite string ili broj koji jedinstveno identifikuje tu stavku među njenim susedima. Obično, ključ (key) bi trebalo da dolazi iz vaših podataka, kao što je ID iz baze podataka. React koristi vaše ključeve (keys) da bi znao šta se dogodilo ako kasnije ubacite, izbrišete ili preuredite stavke.

<Sandpack>

```js
const products = [
  { title: 'Kupus', isFruit: false, id: 1 },
  { title: 'Luk', isFruit: false, id: 2 },
  { title: 'Jabuka', isFruit: true, id: 3 },
];

export default function ShoppingList() {
  const listItems = products.map(product =>
    <li
      key={product.id}
      style={{
        color: product.isFruit ? 'magenta' : 'darkgreen'
      }}
    >
      {product.title}
    </li>
  );

  return (
    <ul>{listItems}</ul>
  );
}
```

</Sandpack>

## Odgovaranje na event-e {/*responding-to-events*/}

Možete odgovarati na event-e deklarisanjem *event handler* funkcija za obradu event-a unutar vaših komponenti:

```js {2-4,7}
function MyButton() {
  function handleClick() {
    alert('Kliknuli ste me!');
  }

  return (
    <button onClick={handleClick}>
      Kliknite me
    </button>
  );
}
```

Obratite pažnju na to kako `onClick={handleClick}` nema zagrade na kraju! Ne treba da *pozivate* event handler funkciju: samo je treba *proslediti*. React će pozvati vaš event handler kada korisnik klikne na dugme.

## Ažuriranje ekrana {/*updating-the-screen*/}

Često ćete želeti da vaša komponenta "zapamti" neke informacije i prikaže ih. Na primer, možda želite da prebrojite koliko puta je dugme kliknuto. Da biste to uradili, dodajte *stanje (state)* u svoju komponentu.

Prvo, uvezite [`useState`](/reference/react/useState) iz React-a:

```js
import { useState } from 'react';
```

Sada možete deklarisati *state varijablu* unutar vaše komponente:

```js
function MyButton() {
  const [count, setCount] = useState(0);
  // ...
```

Od `useState` dobićete dve stvari: trenutni state (`count`), i funkciju koja vam omogućava da ga ažurirate  (`setCount`). Možete im dati bilo koja imena, ali je konvencija da pišete `[nešto, setNešto]`.

Prvi put kada se dugme prikaže, `count` će biti `0` jer ste `0` prosledili u `useState()`. Kada želite da promenite state, pozovite `setCount()` i prosledite joj novu vrednost. Klikom na ovo dugme inkrementujte brojač:

```js {5}
function MyButton() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      Kliknuto je {count} puta
    </button>
  );
}
```

React će ponovo pozvati funkciju vaše komponente. Ovaj put, `count` će biti `1`. Zatim će biti `2`. I tako dalje.

Ako renderujete istu komponentu više puta, svaka će dobiti svoje sopstveni state. Kliknite svako dugme posebno:

<Sandpack>

```js
import { useState } from 'react';

export default function MyApp() {
  return (
    <div>
      <h1>Brojači koji se ažuriraju nezavisno</h1>
      <MyButton />
      <MyButton />
    </div>
  );
}

function MyButton() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      Kliknuto je {count} puta
    </button>
  );
}
```

```css
button {
  display: block;
  margin-bottom: 5px;
}
```

</Sandpack>

Primetite kako svako dugme "pamti" svoj sopstveni state brojača `count` i ne utiče na drugu dugmad.

## Korišćenje Hook-ova {/*using-hooks*/}

Funkcije koje počinju sa `use` nazivaju se *Hook-ovi*. `useState` je ugrađeni Hook koji pruža React. Možete pronaći i druge ugrađene Hook-ove u [API referenci.](/reference/react) Takođe, možete pisati svoje sopstvene Hook-ove kombinovanjem postojećih.

Hook-ovi su restriktivniji od ostalih funkcija. Hook-ove možete pozivati samo na vrhu svojih komponenata (ili drugih Hook-ova). Ukoliko nameravate koristiti `useState` u nekom condition-u ili loop-u, izdvojite novu komponentu i stavite ga tamo.

## Prosleđivanje podataka među komponentama {/*sharing-data-between-components*/}

U prethodnom primeru, svaki `MyButton` je imao nezavisni `count`, i kada je svako pojedinačno dugme bilo pritisnuto, `count` se menjao samo za trenutno pritisnuto dugme:

<DiagramGroup>

<Diagram name="sharing_data_child" height={367} width={407} alt="Dijagram koji prikazuje stablo od tri komponente, jednu parent sa oznakom MyApp i dve children označene sa MyButton. Obe MyButton komponente sadrže brojač sa vrednošću nula.">

Inicijalno, obe `MyButton` komponente imaju `count` state `0`

</Diagram>

<Diagram name="sharing_data_child_clicked" height={367} width={407} alt="Isti dijagram kao prethodni, sa podvučenim brojačem prve chilld komponente MyButton koji pokazuje da je kliknuto i da je vrednost brojača povećana na jedan. Druga MyButton komponenta još uvek sadrži vrednost nula." >

Prva `MyButton` komponenta ažurira svoj `count` na `1`

</Diagram>

</DiagramGroup>

Međutim, često će vam biti potrebno da komponente *prosleđuju podatke međusobno i uvek se zajedno ažuriraju*.

Da biste obe `MyButton` komponente prikazali sa istim `count` i ažurirali zajedno, morate premestiti state iz pojedinačnih dugmadi "nagore" do najbliže komponente koja sadrži sve njih.

U ovom primeru, to je `MyApp`:

<DiagramGroup>

<Diagram name="sharing_data_parent" height={385} width={410} alt="Dijagram prikazuje stablo od tri komponente, jedne parent oznake MyApp i dve children sa oznakom MyButton. MyApp sadrži vrednost brojača nula, koja se prosleđuje u obe MyButton komponente, koje takođe prikazuju vrednost nula." >

Inicijalno, `MyApp` ima `count` state vrednosti `0` i prosleđuje se u obe children komponente

</Diagram>

<Diagram name="sharing_data_parent_clicked" height={385} width={410} alt="Isti dijagram kao prethodni, sa podvučenim brojačem parent MyApp komponente, koji ukazuje na klik sa vrednošću uvećanom na jedan. Protok do obe children MyButton komponente je takođe podvučen, a vrednost brojača u svakoj child komponenti je postavljena na jedan što ukazuje da je vrednost prosleđena." >

Pri kliku, `MyApp` ažurira svoj `count` state na `1` i prosleđuje ga u obe children komponente

</Diagram>

</DiagramGroup>

Sada, kada kliknete na bilo koje dugme, `count` u `MyApp` će se promeniti, što će promeniti oba brojača u `MyButton`. Evo kako to možete izraziti u kodu.

Prvo, *podignite state nagore* iz `MyButton` u `MyApp`:

```js {2-6,18}
export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>Brojači koji se ažuriraju odvojeno</h1>
      <MyButton />
      <MyButton />
    </div>
  );
}

function MyButton() {
  // ... premeštamo kod odavde ...
}

```

Zatim, *prosleđujemo state nadole* iz `MyApp` u oba `MyButton`, zajedno sa zajedničkim handler-om za klik. Informacije možete proslediti u `MyButton` koristeći kovrdžave zagrade u JSX-u, baš kao što ste to ranije radili sa ugrađenim tagovima poput `<img>`:

```js {11-12}
export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>Brojači koji se ažuriraju zajedno</h1>
      <MyButton count={count} onClick={handleClick} />
      <MyButton count={count} onClick={handleClick} />
    </div>
  );
}
```

Informacije koje na ovaj način prosleđujete nazivaju se *props*. Sada `MyApp` komponenta sadrži `count` state i `handleClick` event handler i *prosleđuje obe nadole kao props* svakom dugmetu.

Konačno, promenite `MyButton` da *čita* props koje ste prosledili iz njegove parent komponente:

```js {1,3}
function MyButton({ count, onClick }) {
  return (
    <button onClick={onClick}>
      Kliknuto {count} puta
    </button>
  );
}
```

Kada kliknete na dugme, `onClick` handler se aktivira. `onClick` prop svakog dugmeta postavljen je na funkciju `handleClick` unutar `MyApp` pa se kod unutar nje izvršava. Taj kod poziva `setCount(count + 1)`, inkrementirajući `count` state varijablu. Nova `count` vrednost se prosleđuje kao prop svakom dugmetu, pa svi prikazuju novu vrednost. Ovo se naziva "podizanje state-a nagore". Pomeranjem state-a nagore, podelili ste ga između komponenata.

<Sandpack>

```js
import { useState } from 'react';

export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>Brojači koji se ažuriraju zajedno</h1>
      <MyButton count={count} onClick={handleClick} />
      <MyButton count={count} onClick={handleClick} />
    </div>
  );
}

function MyButton({ count, onClick }) {
  return (
    <button onClick={onClick}>
      Kliknuto {count} puta
    </button>
  );
}
```

```css
button {
  display: block;
  margin-bottom: 5px;
}
```

</Sandpack>

## Sledeći koraci {/*next-steps*/}

Sada već znate osnove pisanja React koda!

Pogledajte [Tutorijal](/learn/tutorial-tic-tac-toe) biste primenili naučeno i izgradili svoj prvi mini-app sa React-om.
