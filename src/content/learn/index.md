---
title: Brzi Uvod
---

<Intro>

Dobrodošli u React dokumentaciju! Ova stranica pružiće vam uvod u 80% React koncepata koje ćete koristiti u svakodnevnom radu.

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

React app-ovi su sačinjeni od *komponenti*. Komponenta je deo UI-a (korisničkog interfejsa) koji ima svoju logiku i izgled. Komponenta može biti mala kao dugme, ili velika kao cela stranica.

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

## Pisanje markupa sa JSX {/*writing-markup-with-jsx*/}

Sintaksa markup-a koju ste videli iznad se zove *JSX*. Ona nije obavezna, ali većina React projekata koristi JSX zbog njegove praktičnosti. Svi [alati koje preporučujemo za lokalni razvoj](/learn/installation) podržavaju JSX odmah po instalaciji.

JSX je striktniji od HTML-a. Morate zatvoriti tagove poput `<br />`. Vaša komponenta takođe ne može vraćati više JSX tagova. Morate ih obuhvatiti zajedničkim roditeljem, poput `<div>...</div>` ili praznog `<>...</>` omotača:

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

U React-u, nema posebne sintakse za pisanje kondicionih izraza. Umesto toga, koristićete iste tehnike kao kada pišete običan JavaScript kod. Na primer, možete koristiti [`if`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else) izraz za kondicionalno uključivanje JSX-a:

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

Svi ovi pristupi takođe rade i za kondicionalno specificiranje atributa. Ako niste upoznati sa oovim delovima JavaScript sintakse, možete početi tako što ćete uvek koristiti `if...else`.

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

Možete odgovarati na event-e deklarisanjem *event handler-a* funkcija za obradu event-a unutar vaših komponenti:

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

Obratite pažnju na to kako `onClick={handleClick}` nema zagrade na kraju! Ne treba da *pozivate* event handler funkciju: smo je treba *proslediti*. React će pozvati vaš event handler kada korisnik klikne na dugme.

## Ažuriranje ekrana {/*updating-the-screen*/}

Često ćete želite da vaša komponenta "zapamti" neke informacije i prikaže ih. Na primer, možda želite da prebrojite koliko puta je dugme kliknuto. To do this, add *stanje (state)* u svoju komponentu.

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

Od`useState` dobićete dobiti dve stvari: trenutni state (`count`), i funkciju koja vam omogućava da ga ažurirate  (`setCount`). Možete im dati bilo koja imena, ali je konvencija da pišete `[nešto, setNešto]`.

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

## Using Hooks {/*using-hooks*/}

Functions starting with `use` are called *Hooks*. `useState` is a built-in Hook provided by React. You can find other built-in Hooks in the [API reference.](/reference/react) You can also write your own Hooks by combining the existing ones.

Hooks are more restrictive than other functions. You can only call Hooks *at the top* of your components (or other Hooks). If you want to use `useState` in a condition or a loop, extract a new component and put it there.

## Sharing data between components {/*sharing-data-between-components*/}

In the previous example, each `MyButton` had its own independent `count`, and when each button was clicked, only the `count` for the button clicked changed:

<DiagramGroup>

<Diagram name="sharing_data_child" height={367} width={407} alt="Diagram showing a tree of three components, one parent labeled MyApp and two children labeled MyButton. Both MyButton components contain a count with value zero.">

Initially, each `MyButton`'s `count` state is `0`

</Diagram>

<Diagram name="sharing_data_child_clicked" height={367} width={407} alt="The same diagram as the previous, with the count of the first child MyButton component highlighted indicating a click with the count value incremented to one. The second MyButton component still contains value zero." >

The first `MyButton` updates its `count` to `1`

</Diagram>

</DiagramGroup>

However, often you'll need components to *share data and always update together*.

To make both `MyButton` components display the same `count` and update together, you need to move the state from the individual buttons "upwards" to the closest component containing all of them.

In this example, it is `MyApp`:

<DiagramGroup>

<Diagram name="sharing_data_parent" height={385} width={410} alt="Diagram showing a tree of three components, one parent labeled MyApp and two children labeled MyButton. MyApp contains a count value of zero which is passed down to both of the MyButton components, which also show value zero." >

Initially, `MyApp`'s `count` state is `0` and is passed down to both children

</Diagram>

<Diagram name="sharing_data_parent_clicked" height={385} width={410} alt="The same diagram as the previous, with the count of the parent MyApp component highlighted indicating a click with the value incremented to one. The flow to both of the children MyButton components is also highlighted, and the count value in each child is set to one indicating the value was passed down." >

On click, `MyApp` updates its `count` state to `1` and passes it down to both children

</Diagram>

</DiagramGroup>

Now when you click either button, the `count` in `MyApp` will change, which will change both of the counts in `MyButton`. Here's how you can express this in code.

First, *move the state up* from `MyButton` into `MyApp`:

```js {2-6,18}
export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>Counters that update separately</h1>
      <MyButton />
      <MyButton />
    </div>
  );
}

function MyButton() {
  // ... we're moving code from here ...
}

```

Then, *pass the state down* from `MyApp` to each `MyButton`, together with the shared click handler. You can pass information to `MyButton` using the JSX curly braces, just like you previously did with built-in tags like `<img>`:

```js {11-12}
export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>Counters that update together</h1>
      <MyButton count={count} onClick={handleClick} />
      <MyButton count={count} onClick={handleClick} />
    </div>
  );
}
```

The information you pass down like this is called _props_. Now the `MyApp` component contains the `count` state and the `handleClick` event handler, and *passes both of them down as props* to each of the buttons.

Finally, change `MyButton` to *read* the props you have passed from its parent component:

```js {1,3}
function MyButton({ count, onClick }) {
  return (
    <button onClick={onClick}>
      Clicked {count} times
    </button>
  );
}
```

When you click the button, the `onClick` handler fires. Each button's `onClick` prop was set to the `handleClick` function inside `MyApp`, so the code inside of it runs. That code calls `setCount(count + 1)`, incrementing the `count` state variable. The new `count` value is passed as a prop to each button, so they all show the new value. This is called "lifting state up". By moving state up, you've shared it between components.

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
      <h1>Counters that update together</h1>
      <MyButton count={count} onClick={handleClick} />
      <MyButton count={count} onClick={handleClick} />
    </div>
  );
}

function MyButton({ count, onClick }) {
  return (
    <button onClick={onClick}>
      Clicked {count} times
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

## Next Steps {/*next-steps*/}

By now, you know the basics of how to write React code!

Check out the [Tutorial](/learn/tutorial-tic-tac-toe) to put them into practice and build your first mini-app with React.
