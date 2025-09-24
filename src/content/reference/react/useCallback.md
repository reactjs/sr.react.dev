---
title: useCallback
---

<Intro>

`useCallback` je React Hook koji vam omogućava da keširate definiciju funkcije između ponovnih rendera.

```js
const cachedFn = useCallback(fn, dependencies)
```

</Intro>

<Note>

[React kompajler](/learn/react-compiler) automatski memoizuje vrednosti i funkcije, što smanjuje potrebu za ručnim pozivima `useCallback`-a. Možete koristiti kompajler za automatsko rukovanje memoizacijom.

</Note>

<InlineToc />

---

## Reference {/*reference*/}

### `useCallback(fn, dependencies)` {/*usecallback*/}

Pozovite `useCallback` na vrhu vaše komponente kako biste keširali definiciju funkcije između ponovnih rendera:

```js {4,9}
import { useCallback } from 'react';

export default function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
```

[Pogledajte još primera ispod.](#usage)

#### Parametri {/*parameters*/}

* `fn`: Funkcija koju želite da keširate. Može primiti bilo koje argumente i vratiti bilo koje vrednosti. React će vratiti (ne pozvati!) vašu funkciju nazad tokom inicijalnog rendera. U narednim renderima, React će vam dati istu funkciju ako se `dependencies` nisu promenili od poslednjeg rendera. U suprotnom, vratiće vam funkciju koju ste prosledili tokom trenutnog rendera i sačuvati je za slučaj da se može iskoristiti kasnije. React neće pozvati vašu funkciju. Funkcija će vam biti vraćena kako biste odlučili kada i da li ćete je pozvati.

* `dependencies`: Lista svih reaktivnih vrednosti referenciranih unutar koda `fn` funkcije. Reaktivne vrednosti uključuju props-e, state i sve promenljive i funkcije deklarisane direktno unutar tela vaše komponente. Ako vam je linter [konfigurisan za React](/learn/editor-setup#linting), verifikovaće da li je svaka reaktivna vrednost ispravno specificirana kao zavisnost. Lista zavisnosti mora imati konstantan broj članova i biti napisana inline poput `[dep1, dep2, dep3]`. React će uporediti svaku zavisnost sa njenom prethodnom vrednošću upotrebom [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) algoritma poređenja.

#### Povratne vrednosti {/*returns*/}

Prilikom inicijalnog rendera, `useCallback` vraća `fn` funkciju koju ste prosledili.

Tokom narednih rendera, vratiće ili već sačuvanu `fn` funkciju iz prethodnog rendera (ako se zavisnosti nisu promenile), ili `fn` funkciju koju ste prosledili u trenutnom renderu.

#### Upozorenja {/*caveats*/}

* `useCallback` je Hook, pa ga možete pozvati samo **na vrhu vaše komponente** ili vaših Hook-ova. Ne možete ga pozvati unutar petlji i uslova. Ako vam je to potrebno, izdvojite novu komponentu i pomerite state u nju.
* React **neće odbaciti keširanu funkciju osim ako ne postoji poseban razlog za tako nešto**. Na primer, u toku razvoja, React odbacuje keš kada izmenite fajl vaše komponente. U toku razvoja i u produkciji, React će odbaciti keš ako se vaša komponenta suspenduje tokom inicijalnog montiranja. U budućnosti, React može dodati nove funkcionalnosti koje koriste odbacivanje keša--na primer, ako React doda ugrađenu podršku za virtuelizovane liste u budućnosti, imalo bi smisla odbaciti keš za članove koji izlaze izvan vidnog polja virtuelizovane tabele. Ovo bi trebalo ispuniti vaša očekivanja ako se uzdate u `useCallback` za optimizaciju performansi. Inače, [state promenljiva](/reference/react/useState#im-trying-to-set-state-to-a-function-but-it-gets-called-instead) ili [ref](/reference/react/useRef#avoiding-recreating-the-ref-contents) mogu biti prikladnija rešenja.

---

## Upotreba {/*usage*/}

### Preskakanje ponovnog renderovanja komponenata {/*skipping-re-rendering-of-components*/}

Kada optimizujete performanse renderovanja, ponekad ćete trebati da keširate funkcije koje prosleđujete dečjim komponentama. Hajde prvo da pogledamo sintaksu kako bi se to moglo uraditi i da vidimo u kojim slučajevima je to korisno.

Da biste keširali funkciju između ponovnih rendera vaše komponente, obmotajte njenu definiciju sa `useCallback` Hook-om:

```js [[3, 4, "handleSubmit"], [2, 9, "[productId, referrer]"]]
import { useCallback } from 'react';

function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);
  // ...
```

Potrebno je proslediti dve stvari u `useCallback`:

1. Definiciju funkcije koju želite keširati između ponovnih rendera.
2. <CodeStep step={2}>Listu zavisnosti</CodeStep> koja uključuje svaku vrednost unutar vaše komponente koja se koristi unutar te funkcije.

Prilikom inicijalnog rendera, <CodeStep step={3}>povratna funkcija</CodeStep> koju dobijate iz `useCallback`-a će biti funkcija koju ste prosledili.

U narednim renderima, React će porediti <CodeStep step={2}>zavisnosti</CodeStep> sa zavisnostima koje ste prosledili tokom prethodnog rendera. Ako se nijedna zavisnost nije promenila (poređenjem sa [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is)), `useCallback` će vratiti istu funkciju kao i pre. U suprotnom, `useCallback` će vratiti funkciju koju ste prosledili u *ovom* renderu.

Drugim rečima, `useCallback` kešira funkciju između ponovnih rendera dok joj se zavisnosti ne promene.

**Prođimo kroz primer da vidimo kada je ovo korisno.**

Recimo da prosleđujete `handleSubmit` funkciju iz `ProductPage` u `ShippingForm` komponentu:

```js {5}
function ProductPage({ productId, referrer, theme }) {
  // ...
  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
```

Primetili ste da promena `theme` prop-a zamrzava aplikaciju na sekund, ali ako uklonite `<ShippingForm />` iz JSX-a, sve radi brzo. Ovo vam govori da nije loše probati da optimizujete `ShippingForm` komponentu.

**Po default-u, kada se komponenta ponovo renderuje, React rekurzivno ponovo renderuje svu njenu decu.** Zbog toga, kada se `ProductPage` ponovo renderuje sa novom `theme`, i `ShippingForm` komponenta se *takođe* ponovo renderuje. Ovo je u redu za komponente koje ne zahtevaju mnogo proračuna za ponovno renderovanje. Ali, ako ste potvrdili da je ponovno renderovanje sporo, možete reći `ShippingForm` komponenti da preskoči renderovanje kada su njeni props-i isti kao i u prethodnom renderu, tako što ćete je obmotati sa [`memo`](/reference/react/memo):

```js {3,5}
import { memo } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  // ...
});
```

**Sa ovom promenom, `ShippingForm` će preskočiti ponovno renderovanje ako su joj svi props-i *isti* kao i u poslednjem renderu.** Ovde keširanje funkcije postaje važno! Recimo da ste definisali `handleSubmit` bez `useCallback`:

```js {2,3,8,12-13}
function ProductPage({ productId, referrer, theme }) {
  // Svaki put kad se theme promeni, ovo će biti drugačija funkcija...
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }

  return (
    <div className={theme}>
      {/* ... pa props-i ShippingForm-a nikad neće biti isti i svaki put će se ponovo renderovati */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

**U JavaScript-u, `function () {}` ili `() => {}` uvek kreira _drugačiju_ funkciju**, slično kao što `{}` literal objekta uvek kreira novi objekat. Obično ovo ne bi bio problem, ali ovo znači da props-i `ShippingForm`-a nikad neće biti isti i da vaša [`memo`](/reference/react/memo) optimizacija ne radi. Ovde `useCallback` postaje korisna:

```js {2,3,8,12-13}
function ProductPage({ productId, referrer, theme }) {
  // Reci React-u da kešira tvoju funkciju između ponovnih rendera...
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // ...i dok god se ove zavisnosti ne menjaju...

  return (
    <div className={theme}>
      {/* ...ShippingForm će primiti iste props-e i preskočiti ponovni render */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

**Obmotavanjem `handleSubmit`-a sa `useCallback` osiguravate da je to *ista* funkcija između ponovnih rendera** (dok se zavisnosti ne promene). Ne *trebate da* obmotate funkciju sa `useCallback` ako nemate poseban razlog. U ovom primeru, razlog je to što je prosleđujete u komponentu obmotanu sa [`memo`](/reference/react/memo), pa vam to omogućava da preskočite ponovno renderovanje. Postoje i drugi razlozi za upotrebu `useCallback`-a koji su opisani kasnije na ovoj stranici.

<Note>

**Treba se oslanjati na `useCallback` samo kao optimizaciju performansi.** Ako vaš kod ne radi bez toga, pronađite glavni razlog i prvo ga popravite. Tek onda možete dodati `useCallback`.

</Note>

<DeepDive>

#### Kako je useCallback povezan sa useMemo? {/*how-is-usecallback-related-to-usememo*/}

Često ćete videti [`useMemo`](/reference/react/useMemo) pored `useCallback`-a. Oba su korisna kada pokušavate optimizovati dečju komponentu. Omogućavaju vam da [memoizujete](https://sr.wikipedia.org/sr-ec/%D0%9C%D0%B5%D0%BC%D0%BE%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%98%D0%B0) (ili, drugim rečima, keširate) nešto što prosleđujete deci:

```js {6-8,10-15,19}
import { useMemo, useCallback } from 'react';

function ProductPage({ productId, referrer }) {
  const product = useData('/product/' + productId);

  const requirements = useMemo(() => { // Poziva vašu funkciju i kešira rezultat
    return computeRequirements(product);
  }, [product]);

  const handleSubmit = useCallback((orderDetails) => { // Kešira samu funkciju
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);

  return (
    <div className={theme}>
      <ShippingForm requirements={requirements} onSubmit={handleSubmit} />
    </div>
  );
}
```

Razlika je u tome *šta* vam omogućavaju da keširate:

* **[`useMemo`](/reference/react/useMemo) kešira *rezultat* poziva vaše funkcije.** U ovom primeru, kešira rezultat poziva `computeRequirements(product)` tako da se ne menja dok se `product` ne promeni. Ovo vam omogućava da prosledite `requirements` objekat bez nepotrebnih ponovnih rendera `ShippingForm`-a. Kada je neophodno, React će tokom renderovanja pozvati funkciju koju ste prosledili kako bi izračunao rezultat.
* **`useCallback` kešira *samu funkciju***. Za razliku od `useMemo`-a, ne poziva funkciju koju prosledite. Umesto toga, kešira prosleđenu funkciju tako da se *sama* `handleSubmit` ne menja osim ako se `productId` ili `referrer` promene. Ovo vam omogućava da prosledite `handleSubmit` funkciju bez nepotrebnih ponovnih rendera `ShippingForm`-a. Vaš kod se neće izvršiti dok korisnik ne submit-uje formu.

Ako ste već upoznati sa [`useMemo`](/reference/react/useMemo), može vam biti lakše da razmišljate o `useCallback`-u na ovaj način:

```js {expectedErrors: {'react-compiler': [3]}}
// Pojednostavljena implementacija (unutar React-a)
function useCallback(fn, dependencies) {
  return useMemo(() => fn, dependencies);
}
```

[Čitajte još o razlici između `useMemo` i `useCallback`.](/reference/react/useMemo#memoizing-a-function)

</DeepDive>

<DeepDive>

#### Trebate li dodati useCallback svuda? {/*should-you-add-usecallback-everywhere*/}

Ako je vaša aplikacija poput ovog sajta, gde su većinom grube interakcije (poput zamene stranice ili cele sekcije), memoizacija uglavnom nije potrebna. Na drugu stranu, ako vam aplikacija liči na editor crteža i interakcije su uglavnom granularne (poput pomeranja oblika), onda vam memoizacija može biti od velike pomoći.

Keširanje funkcije sa `useCallback` je korisno samo u par slučajeva:

- Prosleđujete je kao prop u komponentu koja je obmotana sa [`memo`](/reference/react/memo). Želite preskočiti ponovno renderovanje ako se vrednost nije promenila. Memoizacija čini da se vaša komponenta ponovo renderuje samo ako se zavisnosti promene.
- Funkcija koju prosleđujete se kasnije koristi kao zavisnost nekog Hook-a. Na primer, druga funkcija obmotana sa `useCallback` zavisi od nje, ili zavisite od te funkcije kroz [`useEffect`](/reference/react/useEffect).

Nema benefita obmotavati funkciju sa `useCallback` u ostalim slučajevima. Doduše, ne postoji ni značajna šteta u tome, pa neki timovi odlučuju da ne razmišljaju o pojedinačnim slučajevima i da memoizuju što je više moguće. Loša strana je da kod postaje manje čitljiv. Takođe, nije svaka memoizacija efikasna: pojedinačna vrednost koja je "uvek nova" je dovoljna da slomi memoizaciju za celu komponentu.

Primetite da `useCallback` ne sprečava *kreiranje* funkcije. Uvek kreirate funkciju (i to je u redu!), ali React to ignoriše i vraća vam keširanu funkciju ako se ništa nije promenilo.

**U praksi dosta memoizacije možete učiniti nepotrebnom ako pratite par principa:**

1. Kada komponenta vizuelno obmotava druge komponente, napravite da [prima JSX kao decu](/learn/passing-props-to-a-component#passing-jsx-as-children). U tom slučaju, ako obmotavajuća komponenta ažurira svoj state, React zna da njena deca ne trebaju ponovo da se renderuju.
2. Koristite lokalni state i nemojte [podizati state](/learn/sharing-state-between-components) više nego što je potrebno. Nemojte čuvati prolazni state poput formi i podataka da li prelazite mišem preko nečega na vrhu stabla ili u globalnoj state biblioteci.
3. Postarajte se da je [logika renderovanja čista](/learn/keeping-components-pure). Ako ponovni render komponente pravi problem ili neki uočljivi vizuelni artefakt, to je bug u komponenti! Popravite bug umesto dodavanja memoizacije.
4. Izbegavajte [nepotrebne Effect-e koji ažuriraju state](/learn/you-might-not-need-an-effect). Većina problema sa performansama u React aplikacijama prouzrokovani su nizom ažuriranja koji potiču od Effect-a koji iznova i iznova renderuju vaše komponente.
5. Pokušajte da [uklonite nepotrebne zavisnosti u Effect-ima](/learn/removing-effect-dependencies). Na primer, umesto memoizacije, često je lakše pomeriti neki objekat ili funkciju unutar Effect-a ili izvan komponente.

Ako neka posebna interakcija i dalje deluje da lag-uje, [iskoristite React Developer Tools profiler](https://legacy.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html) da vidite koje komponente mogu imati benefita od memoizacije i dodajte memoizaciju gde je potrebno. Ovi principi čine vaše komponente lakšim za debug-ovanje i razumevanje, pa je dobro da ih uvek pratite. Dugoročno, mi istražujemo [automatsku upotrebu memoizacije](https://www.youtube.com/watch?v=lGEMwh32soc) da ovo rešimo jednom za svagda.

</DeepDive>

<Recipes titleText="Razlika između useCallback i direktnog deklarisanja funkcije" titleId="examples-rerendering">

#### Preskakanje ponovnog renderovanja sa `useCallback` i `memo` {/*skipping-re-rendering-with-usecallback-and-memo*/}

U ovom primeru, `ShippingForm` komponenta je **veštački usporena** kako biste mogli videti šta se dešava kada je React komponenta koju renderujete zapravo spora. Pokušajte inkrementirati brojač i menjati temu.

Inkrementiranje brojača deluje sporo jer tera usporenu `ShippingForm` komponentu da se ponovo renderuje. To je očekivano jer se brojač promenio, pa morate da prikažete korisnikov novi izbor na ekranu.

Posle toga probajte da promenite temu. **Zahvaljujući `useCallback`-u zajedno sa [`memo`](/reference/react/memo), promena je brza uprkos veštačkom usporavanju!** `ShippingForm` je preskočila ponovno renderovanje jer se `handleSubmit` funkcija nije promenila. `handleSubmit` funkcija se nije promenila jer se ni `productId` ni `referrer` (zavisnosti `useCallback`-a) nisu promenili od poslednjeg rendera.

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ProductPage from './ProductPage.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Tamni režim
      </label>
      <hr />
      <ProductPage
        referrerId="wizard_of_oz"
        productId={123}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/ProductPage.js active
import { useCallback } from 'react';
import ShippingForm from './ShippingForm.js';

export default function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);

  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}

function post(url, data) {
  // Zamisli slanje zahteva...
  console.log('POST /' + url);
  console.log(data);
}
```

```js {expectedErrors: {'react-compiler': [7, 8]}} src/ShippingForm.js
import { memo, useState } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  const [count, setCount] = useState(1);

  console.log('[VEŠTAČKI SPORO] Renderovanje <ShippingForm />');
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // Ništa ne radi 500 ms da simuliraš veoma spor kod
  }

  function handleSubmit(e) {
    e.preventDefault();
    const formData = new FormData(e.target);
    const orderDetails = {
      ...Object.fromEntries(formData),
      count
    };
    onSubmit(orderDetails);
  }

  return (
    <form onSubmit={handleSubmit}>
      <p><b>Napomena: <code>ShippingForm</code> je veštački usporena!</b></p>
      <label>
        Broj stavki:
        <button type="button" onClick={() => setCount(count - 1)}>–</button>
        {count}
        <button type="button" onClick={() => setCount(count + 1)}>+</button>
      </label>
      <label>
        Ulica:
        <input name="street" />
      </label>
      <label>
        Grad:
        <input name="city" />
      </label>
      <label>
        Poštanski broj:
        <input name="zipCode" />
      </label>
      <button type="submit">Submit</button>
    </form>
  );
});

export default ShippingForm;
```

```css
label {
  display: block; margin-top: 10px;
}

input {
  margin-left: 5px;
}

button[type="button"] {
  margin: 5px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>

<Solution />

#### Uvek ponovno renderovanje komponente {/*always-re-rendering-a-component*/}

U ovom primeru, `ShippingForm` komponenta je takođe **veštački usporena** kako biste mogli videti šta se dešava kada je React komponenta koju renderujete zapravo spora. Pokušajte inkrementirati brojač i menjati temu.

Za razliku od prethodnog primera, sada je promena teme takođe spora! To se dešava jer **nema `useCallback` poziva u ovoj verziji**, pa je `handleSubmit` uvek nova funkcija i usporena `ShippingForm` komponenta ne može da preskoči ponovno renderovanje.

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ProductPage from './ProductPage.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Tamni režim
      </label>
      <hr />
      <ProductPage
        referrerId="wizard_of_oz"
        productId={123}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/ProductPage.js active
import ShippingForm from './ShippingForm.js';

export default function ProductPage({ productId, referrer, theme }) {
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }

  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}

function post(url, data) {
  // Zamisli slanje zahteva...
  console.log('POST /' + url);
  console.log(data);
}
```

```js {expectedErrors: {'react-compiler': [7, 8]}} src/ShippingForm.js
import { memo, useState } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  const [count, setCount] = useState(1);

  console.log('[VEŠTAČKI SPORO] Renderovanje <ShippingForm />');
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // Ništa ne radi 500 ms da simuliraš veoma spor kod
  }

  function handleSubmit(e) {
    e.preventDefault();
    const formData = new FormData(e.target);
    const orderDetails = {
      ...Object.fromEntries(formData),
      count
    };
    onSubmit(orderDetails);
  }

  return (
    <form onSubmit={handleSubmit}>
      <p><b>Napomena: <code>ShippingForm</code> je veštački usporena!</b></p>
      <label>
        Broj stavki:
        <button type="button" onClick={() => setCount(count - 1)}>–</button>
        {count}
        <button type="button" onClick={() => setCount(count + 1)}>+</button>
      </label>
      <label>
        Ulica:
        <input name="street" />
      </label>
      <label>
        Grad:
        <input name="city" />
      </label>
      <label>
        Poštanski broj:
        <input name="zipCode" />
      </label>
      <button type="submit">Submit</button>
    </form>
  );
});

export default ShippingForm;
```

```css
label {
  display: block; margin-top: 10px;
}

input {
  margin-left: 5px;
}

button[type="button"] {
  margin: 5px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>


Međutim, evo istog koda **gde je veštačko usporavanje uklonjeno**. Da li se nedostatak `useCallback`-a oseća ili ne?

<Sandpack>

```js src/App.js
import { useState } from 'react';
import ProductPage from './ProductPage.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Tamni režim
      </label>
      <hr />
      <ProductPage
        referrerId="wizard_of_oz"
        productId={123}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

```js src/ProductPage.js active
import ShippingForm from './ShippingForm.js';

export default function ProductPage({ productId, referrer, theme }) {
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }

  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}

function post(url, data) {
  // Zamisli slanje zahteva...
  console.log('POST /' + url);
  console.log(data);
}
```

```js src/ShippingForm.js
import { memo, useState } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  const [count, setCount] = useState(1);

  console.log('Renderovanje <ShippingForm />');

  function handleSubmit(e) {
    e.preventDefault();
    const formData = new FormData(e.target);
    const orderDetails = {
      ...Object.fromEntries(formData),
      count
    };
    onSubmit(orderDetails);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Broj stavki:
        <button type="button" onClick={() => setCount(count - 1)}>–</button>
        {count}
        <button type="button" onClick={() => setCount(count + 1)}>+</button>
      </label>
      <label>
        Ulica:
        <input name="street" />
      </label>
      <label>
        Grad:
        <input name="city" />
      </label>
      <label>
        Poštanski broj:
        <input name="zipCode" />
      </label>
      <button type="submit">Submit</button>
    </form>
  );
});

export default ShippingForm;
```

```css
label {
  display: block; margin-top: 10px;
}

input {
  margin-left: 5px;
}

button[type="button"] {
  margin: 5px;
}

.dark {
  background-color: black;
  color: white;
}

.light {
  background-color: white;
  color: black;
}
```

</Sandpack>


Veoma često kod bez memoizacije radi dobro. Ako su vaše interakcije dovoljno brze, ne treba vam memoizacija.

Imajte na umu da morate pokrenuti React u produkciji, onemogućiti [React Developer Tools](/learn/react-developer-tools) i koristiti uređaje slične onima koje koriste korisnici vaše aplikacije kako biste imali realističnu sliku šta zapravo usporava vašu aplikaciju.

<Solution />

</Recipes>

---

### Ažuriranje state-a iz memoizovanog callback-a {/*updating-state-from-a-memoized-callback*/}

Ponekad će vam trebati da ažurirate state na osnovu prethodnog state-a u memoizovanom callback-u.

Ova `handleAddTodo` funkcija specificira `todos` kao zavisnost jer na osnovu nje računa naredni todos:

```js {6,7}
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos([...todos, newTodo]);
  }, [todos]);
  // ...
```

Uglavnom želite da memoizovane funkcije imaju što manje zavisnosti. Kada neki state čitate samo da biste izračunali naredni state, možete ukloniti tu zavisnost prosleđivanjem [updater funkcije](/reference/react/useState#updating-state-based-on-the-previous-state):

```js {6,7}
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos(todos => [...todos, newTodo]);
  }, []); // ✅ Nema potrebe za todos zavisnošću
  // ...
```

Ovde, umesto da imate `todos` zavisnost i da je čitate, React-u prosleđujete instrukciju *kako* ažurirati state (`todos => [...todos, newTodo]`). [Pročitajte još o updater funkcijama.](/reference/react/useState#updating-state-based-on-the-previous-state)

---

### Sprečavanje prečestog okidanja Effect-a {/*preventing-an-effect-from-firing-too-often*/}

Ponekad vam može biti potrebno da pozovete funkciju unutar [Effect-a](/learn/synchronizing-with-effects):

```js {4-9,12}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  function createOptions() {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    // ...
```

Ovo stvara problem. [Svaka reaktivna vrednost mora biti deklarisana kao zavisnost vašeg Effect-a.](/learn/lifecycle-of-reactive-effects#react-verifies-that-you-specified-every-reactive-value-as-a-dependency) Međutim, ako deklarišete `createOptions` kao zavisnost, to će uticati da se vaš Effect konstantno rekonektuje na sobu za dopisivanje:


```js {6}
  useEffect(() => {
    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // 🔴 Problem: Ova zavisnost se menja svakim renderom
  // ...
```

Da biste ovo rešili, možete obmotati funkciju koju trebate pozvati unutar Effect-a sa `useCallback`:

```js {4-9,16}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const createOptions = useCallback(() => {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId
    };
  }, [roomId]); // ✅ Menja se samo kada se roomId promeni

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [createOptions]); // ✅ Menja se samo kada se createOptions promeni
  // ...
```

Ovo osigurava da je `createOptions` funkcija ista između ponovnih rendera ako je `roomId` isto. **Međutim, još je bolje ukloniti potrebu da imate funkciju kao zavisnost.** Pomerite funkciju *unutar* Effect-a:

```js {5-10,16}
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    function createOptions() { // ✅ Nema potrebe za useCallback ili funkcijama kao zavisnostima!
      return {
        serverUrl: 'https://localhost:1234',
        roomId: roomId
      };
    }

    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Menja se samo kada se roomId promeni
  // ...
```

Sada je vaš kod jednostavniji i nema potrebu za `useCallback`. [Naučite više o uklanjanju Effect zavisnosti.](/learn/removing-effect-dependencies#move-dynamic-objects-and-functions-inside-your-effect)

---

### Optimizovanje prilagođenog Hook-a {/*optimizing-a-custom-hook*/}

Ako pišete [prilagođeni Hook](/learn/reusing-logic-with-custom-hooks), preporučujemo da sve funkcije koje on vraća obmotate sa `useCallback`:

```js {4-6,8-10}
function useRouter() {
  const { dispatch } = useContext(RouterStateContext);

  const navigate = useCallback((url) => {
    dispatch({ type: 'navigate', url });
  }, [dispatch]);

  const goBack = useCallback(() => {
    dispatch({ type: 'back' });
  }, [dispatch]);

  return {
    navigate,
    goBack,
  };
}
```

Ovo osigurava da korisnici vašeg Hook-a mogu optimizovati svoj kod kada bude potrebno.

---

## Rešavanje problema {/*troubleshooting*/}

### Svaki put kad se moja komponenta renderuje, `useCallback` vraća drugačiju funkciju {/*every-time-my-component-renders-usecallback-returns-a-different-function*/}

Postarajte se da specificirate niz zavisnosti kao drugi argument!

Ako zaboravite niz zavisnosti, `useCallback` će vratiti novu funkciju svaki put:

```js {7}
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }); // 🔴 Vraća novu funkciju svaki put: nema niza zavisnosti
  // ...
```

Ovo je ispravljena verzija gde se prosleđuje niz zavisnosti kao drugi argument:

```js {7}
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // ✅ Ne vraća novu funkciju bespotrebno
  // ...
```

Ako vam ovo ne pomaže, problem je u tome da je barem jedna od vaših zavisnosti različita od prethodnog rendera. Možete debug-ovati ovaj problem ručnim logovanjem vaših zavisnosti u konzolu:

```js {5}
  const handleSubmit = useCallback((orderDetails) => {
    // ..
  }, [productId, referrer]);

  console.log([productId, referrer]);
```

Zatim možete upotrebiti desni klik na nizove iz različitih rendera u konzoli i izabrati "Store as a global variable" za oba. Pretpostavkom da je prvi sačuvan kao `temp1`, a drugi kao `temp2`, možete upotrebiti konzolu pretraživača da proverite da li je svaka od zavisnosti u oba niza jednaka:

```js
Object.is(temp1[0], temp2[0]); // Da li je prva zavisnost jednaka u oba niza?
Object.is(temp1[1], temp2[1]); // Da li je druga zavisnost jednaka u oba niza?
Object.is(temp1[2], temp2[2]); // ... i tako dalje za svaku zavisnost ...
```

Kada pronađete koja zavisnost kvari memoizaciju, pronađite način da je uklonite ili [je takođe memoizujte](/reference/react/useMemo#memoizing-a-dependency-of-another-hook).

---

### Trebam pozvati `useCallback` za svaki član niza u petlji, ali nije dozvoljeno {/*i-need-to-call-usememo-for-each-list-item-in-a-loop-but-its-not-allowed*/}

Pretpostavimo da je `Chart` komponenta obmotana sa [`memo`](/reference/react/memo). Želite da preskočite ponovno renderovanje svakog `Chart`-a u listi kada se `ReportList` komponenta ponovo renderuje. Međutim, ne možete pozvati `useCallback` u petlji:

```js {expectedErrors: {'react-compiler': [6]}} {5-14}
function ReportList({ items }) {
  return (
    <article>
      {items.map(item => {
        // 🔴 Ne možeš pozvati useCallback u petlji ovako:
        const handleClick = useCallback(() => {
          sendReport(item)
        }, [item]);

        return (
          <figure key={item.id}>
            <Chart onClick={handleClick} />
          </figure>
        );
      })}
    </article>
  );
}
```

Umesto toga, izdvojite komponentu za posebnu stavku i tamo stavite `useCallback`:

```js {5,12-21}
function ReportList({ items }) {
  return (
    <article>
      {items.map(item =>
        <Report key={item.id} item={item} />
      )}
    </article>
  );
}

function Report({ item }) {
  // ✅ Pozovi useCallback na vrhu komponente:
  const handleClick = useCallback(() => {
    sendReport(item)
  }, [item]);

  return (
    <figure>
      <Chart onClick={handleClick} />
    </figure>
  );
}
```

Alternativno, možete ukloniti `useCallback` iz poslednjeg snippet-a i umesto toga obmotati `Report` sa [`memo`](/reference/react/memo). Ako se `item` prop ne promeni, `Report` će preskočiti ponovni render, što znači da će i `Chart` preskočiti ponovni render:

```js {5,6-8,15}
function ReportList({ items }) {
  // ...
}

const Report = memo(function Report({ item }) {
  function handleClick() {
    sendReport(item);
  }

  return (
    <figure>
      <Chart onClick={handleClick} />
    </figure>
  );
});
```
