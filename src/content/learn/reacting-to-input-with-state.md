---
title: Reagovanje na input pomoću stanja
---

<Intro>

React pruža deklarativan način za manipulisanje UI-jem. Umesto da direktno manipulišete pojedinačnim delovima UI-a, vi opisujete različita stanja u kojima komponenta može biti i menjate ih kao odgovor na korisnički input. Ovo je slično onome kako dizajneri razmišljaju o UI-u.

</Intro>

<YouWillLearn>

* Kako se deklarativno UI programiranje razlikuje od imperativnog UI programiranja
* Kako da nabrojite različita vizuelna stanja u kojima vaša komponenta može biti
* Kako da pokrenete promene između različitih vizuelnih stanja iz koda

</YouWillLearn>

## Kakav je deklarativan UI u poređenju sa imperativnim {/*how-declarative-ui-compares-to-imperative*/}

Kada dizajnirate UI interakcije, verovatno razmišljate kako se UI *menja* kao odgovor na korisničke akcije. Razmotrite formu koja korisniku omogućava da submit-uje odgovor:

* Kad pišete nešto u formu, "Submit" dugme **postaje omogućeno**.
* Kada pritisnete "Submit", i forma i dugme **postaju onemogućeni**, a **pojavljuje se** spinner.
* Ako mrežni zahtev uspe, forma **postaje skrivena**, a poruka "Hvala vam" **se pojavljuje**.
* Ako mrežni zahtev ne uspe, **pojavljuje se** poruka o grešci, a forma ponovo **postaje omogućena**.

U **imperativnom programiranju** gore navedeno direktno odgovara načinu na koji implementirate interakciju. Morate napisati tačne instrukcije za manipulaciju UI-jem na osnovu onoga što se desilo. Evo drugog načina da razmišljate o tome: zamislite da se vozite pored nekoga u autu i govorite mu svaki put gde da skrene.

<Illustration src="/images/docs/illustrations/i_imperative-ui-programming.png"  alt="U autu koji vozi osoba uznemirenog izgleda koja predstavlja JavaScript, putnik naređuje vozaču da izvrši niz komplikovanih skretanja." />

Ne zna gde želite da idete, već samo prati vaše komande. (I, ako vi promašite smer, završićete na pogrešnom mestu!) Naziva se *imperativno* jer morate da "komandujete" svaki element, od spinner-a do dugmeta, govoreći računaru *kako* da ažurira UI.

U ovom primeru imperativnog UI programiranja, forma je napravljena *bez* React-a. Koristi jedino [DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model) od pretraživača:

<Sandpack>

```js src/index.js active
async function handleFormSubmit(e) {
  e.preventDefault();
  disable(textarea);
  disable(button);
  show(loadingMessage);
  hide(errorMessage);
  try {
    await submitForm(textarea.value);
    show(successMessage);
    hide(form);
  } catch (err) {
    show(errorMessage);
    errorMessage.textContent = err.message;
  } finally {
    hide(loadingMessage);
    enable(textarea);
    enable(button);
  }
}

function handleTextareaChange() {
  if (textarea.value.length === 0) {
    disable(button);
  } else {
    enable(button);
  }
}

function hide(el) {
  el.style.display = 'none';
}

function show(el) {
  el.style.display = '';
}

function enable(el) {
  el.disabled = false;
}

function disable(el) {
  el.disabled = true;
}

function submitForm(answer) {
  // Pretvaraj se da koristiš mrežni poziv.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (answer.toLowerCase() === 'istanbul') {
        resolve();
      } else {
        reject(new Error('Dobar pokušaj, ali pogrešan odgovor. Probaj ponovo!'));
      }
    }, 1500);
  });
}

let form = document.getElementById('form');
let textarea = document.getElementById('textarea');
let button = document.getElementById('button');
let loadingMessage = document.getElementById('loading');
let errorMessage = document.getElementById('error');
let successMessage = document.getElementById('success');
form.onsubmit = handleFormSubmit;
textarea.oninput = handleTextareaChange;
```

```js sandbox.config.json hidden
{
  "hardReloadOnChange": true
}
```

```html public/index.html
<form id="form">
  <h2>Kviz gradova</h2>
  <p>
    Koji grad se nalazi na dva kontinenta?
  </p>
  <textarea id="textarea"></textarea>
  <br />
  <button id="button" disabled>Submit</button>
  <p id="loading" style="display: none">Učitavanje...</p>
  <p id="error" style="display: none; color: red;"></p>
</form>
<h1 id="success" style="display: none">To je tačno!</h1>

<style>
* { box-sizing: border-box; }
body { font-family: sans-serif; margin: 20px; padding: 0; }
</style>
```

</Sandpack>

Manipulisanje UI-jem imperativno radi dovoljno dobro za izolovane slučajeve, ali upravljanje postaje eksponencijalno teže za kompleksnije sisteme. Zamislite da ažurirate stranicu punu formi poput ove. Dodavanje novog UI elementa ili nove interakcije bi zahtevalo pažljivu proveru svog postojećeg koda kako bi bili sigurni da niste napravili bug (na primer, zaboravljanje da se nešto prikaže ili sakrije).

React je napravljen da reši ovaj problem.

U React-u, ne manipulišete UI-jem direktno--što znači da ne omogućavate, onemogućavate, prikazujete niti skrivate komponente direktno. Umesto toga, vi **deklarišete šta želite prikazati**, a React shvata kako da ažurira UI. Zamislite da uđete u taksi i kažete vozaču gde želite ići umesto da mu govorite gde tačno da skreće. Posao taksiste je da vas odvede tamo, jer možda zna za prečice o kojima niste razmišljali!

<Illustration src="/images/docs/illustrations/i_declarative-ui-programming.png" alt="U autu koji vozi React, putnik traži da bude odvezen na specifično mesto na mapi. React shvata kako da to uradi." />

## Razmišljanje o UI-u deklarativno {/*thinking-about-ui-declaratively*/}

Videli ste kako implementirati formu imperativno. Da biste bolje razumeli kako da razmišljate u React-u, ispod ćete ponovo implementirati ovaj UI u React-u:

1. **Identifikovati** različita vizuelna stanja vaše komponente
2. **Odrediti** šta pokreće promene tih stanja
3. **Predstaviti** stanje u memoriji koristeći `useState`
4. **Ukloniti** sve neobavezne state promenljive
5. **Povezati** event handler-e za postavljanje state-a

### Korak 1: Identifikovati različita vizuelna stanja vaše komponente {/*step-1-identify-your-components-different-visual-states*/}

U informatici, možda ste čuli da ["konačan automat"](https://sr.wikipedia.org/wiki/%D0%9A%D0%BE%D0%BD%D0%B0%D1%87%D0%B0%D0%BD_%D0%B0%D1%83%D1%82%D0%BE%D0%BC%D0%B0%D1%82) može biti u jednom od nekoliko “stanja”. Ako radite sa dizajnerom, možda ste videli mockup-e za različita "vizuelna stanja". React se nalazi na preseku dizajna i informatike, tako da su obe ideje izvor inspiracije.

Prvo, morate vizualizovati sva različita "stanja" koje korisnik može videti na UI-u:

* **Prazno**: Forma ima onemogućeno "Submit" dugme.
* **Pisanje**: Forma ima omogućeno "Submit" dugme.
* **Submit-ovanje**: Forma je potpuno onemogućena. Prikazan je spinner.
* **Uspešno**: Poruka "Hvala vam" je prikazana umesto forme.
* **Greška**: Kao i stanje "Pisanje", ali sa dodatnom porukom o grešci.

Baš kao i dizajner, želećete da "mock-ujete" ili pravite "mock-ove" za različita vizuelna stanja pre nego što dodate logiku. Na primer, ovde je mock samo za vizuelni deo forme. Ovaj mock se kontroliše pomoću prop-a pod imenom `status` čija default vrednost je `'empty'`:

<Sandpack>

```js
export default function Form({
  status = 'empty'
}) {
  if (status === 'success') {
    return <h1>To je tačno!</h1>
  }
  return (
    <>
      <h2>Kviz gradova</h2>
      <p>
        U kom gradu je bilbord koji pretvara vazduh u pijaću vodu?
      </p>
      <form>
        <textarea />
        <br />
        <button>
          Submit
        </button>
      </form>
    </>
  )
}
```

</Sandpack>

Možete nazvati taj prop kako god želite, imenovanje nije bitno. Probajte da promenite sa `status = 'empty'` na `status = 'success'` da biste videli uspešnu poruku. Mock-ovanje vam omogućava da brzo iterirate kroz UI pre nego što dodate bilo kakvu logiku. Ovde je detaljniji primer iste komponente, i dalje "kontrolisan" preko `status` prop-a:

<Sandpack>

```js
export default function Form({
  // Probajte 'submitting', 'error', 'success':
  status = 'empty'
}) {
  if (status === 'success') {
    return <h1>To je tačno!</h1>
  }
  return (
    <>
      <h2>Kviz gradova</h2>
      <p>
        U kom gradu je bilbord koji pretvara vazduh u pijaću vodu?
      </p>
      <form>
        <textarea disabled={
          status === 'submitting'
        } />
        <br />
        <button disabled={
          status === 'empty' ||
          status === 'submitting'
        }>
          Submit
        </button>
        {status === 'error' &&
          <p className="Error">
            Dobar pokušaj, ali pogrešan odgovor. Probaj ponovo!
          </p>
        }
      </form>
      </>
  );
}
```

```css
.Error { color: red; }
```

</Sandpack>

<DeepDive>

#### Prikazivanje više vizuelnih stanja istovremeno {/*displaying-many-visual-states-at-once*/}

Ako komponenta ima mnogo vizuelnih stanja, može biti zgodno prikazati ih sve na jednoj stranici:

<Sandpack>

```js src/App.js active
import Form from './Form.js';

let statuses = [
  'empty',
  'typing',
  'submitting',
  'success',
  'error',
];

export default function App() {
  return (
    <>
      {statuses.map(status => (
        <section key={status}>
          <h4>Forma ({status}):</h4>
          <Form status={status} />
        </section>
      ))}
    </>
  );
}
```

```js src/Form.js
export default function Form({ status }) {
  if (status === 'success') {
    return <h1>To je tačno!</h1>
  }
  return (
    <form>
      <textarea disabled={
        status === 'submitting'
      } />
      <br />
      <button disabled={
        status === 'empty' ||
        status === 'submitting'
      }>
        Submit
      </button>
      {status === 'error' &&
        <p className="Error">
          Dobar pokušaj, ali pogrešan odgovor. Probaj ponovo!
        </p>
      }
    </form>
  );
}
```

```css
section { border-bottom: 1px solid #aaa; padding: 20px; }
h4 { color: #222; }
body { margin: 0; }
.Error { color: red; }
```

</Sandpack>

Stranice poput ovih se često nazivaju "living styleguides" ili "storybooks".

</DeepDive>

### Korak 2: Odrediti šta pokreće promene tih stanja {/*step-2-determine-what-triggers-those-state-changes*/}

Možete pokrenuti ažuriranje stanja kao odgovor na dve vrste input-a:

* **Ljudski input-i** poput kliktanja dugmeta, pisanja u tekstualno polje, navigiranja na link.
* **Računarski input-i** poput dobijanja mrežnog odgovora, završavanja timeout-a, učitavanja slike.

<IllustrationBlock>
  <Illustration caption="Ljudski input-i" alt="Prst." src="/images/docs/illustrations/i_inputs1.png" />
  <Illustration caption="Računarski input-i" alt="Jedinice i nule." src="/images/docs/illustrations/i_inputs2.png" />
</IllustrationBlock>

U oba slučaja, **morate postaviti [state promenljive](/learn/state-a-components-memory#anatomy-of-usestate) da biste ažurirali UI**. Za formu koju razvijate, trebaćete da promenite stanje kao odgovor na više različitih input-a:

* **Promena tekstualnog input-a** (ljudski) treba da menja stanje *Prazno* u stanje *Pisanje* i obrnuto, u zavisnosti od toga da li je tekstualno polje prazno ili ne.
* **Klik na Submit dugme** (ljudski) treba da promeni stanje u *Submit-ovanje*.
* **Uspešan mrežni odgovor** (računarski) treba da promeni stanje u *Uspešno*.
* **Neuspešan mrežni odgovor** (računarski) treba da promeni stanje u *Greška* sa odgovarajućom porukom o grešci.

<Note>

Primetite da ljudski input-i često zahtevaju [event handler-e](/learn/responding-to-events)!

</Note>

Da biste lakše vizualizovali ovaj tok, probajte da svako stanje crtate kao označeni krug, a svaku promenu stanja kao strelicu. Na ovaj način možete skicirati mnoge tokove stanja i odvojiti bug-ove mnogo pre implementacije.

<DiagramGroup>

<Diagram name="responding_to_input_flow" height={350} width={688} alt="Dijagram toka od leva ka desno sa 5 čvorova. Prvi čvor, označen sa 'empty', ima jedan prelaz, označen sa 'start typing', koji se povezuje sa čvorom označenim sa 'typing'. Taj čvor ima jedan prelaz, označen kao 'press submit', koji se povezuje sa čvorom koji je označen kao 'submitting', koji ima dva prelaza. Levi prelaz, označen sa 'network error', povezuje se sa čvorom označenim sa 'error'. Desni prelaz, označen sa 'network success', povezuje se sa čvorom označenim sa 'success'.">

Stanja forme

</Diagram>

</DiagramGroup>

### Korak 3: Predstaviti stanje u memoriji koristeći `useState` {/*step-3-represent-the-state-in-memory-with-usestate*/}

Sada je potrebno da predstavite vizuelna stanja u memoriji koristeći [`useState`](/reference/react/useState). Ključ je jednostavnost: svaki deo state-a je "pokretni deo", a **želite što manje potrebnih "pokretnih delova"**. Veća kompleksnost uvodi više bug-ova!

Počnite sa state-om koji *apsolutno mora* biti tu. Na primer, moraćete da čuvate `answer` za input kao i `error` (ako postoji) da čuvate poslednju grešku:

```js
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
```

Onda, biće vam potrebna state promenljiva da predstavite koje vizuelno stanje želite da prikažete. Obično postoji više od jednog načina da to predstavite u memoriji, tako da morate eksperimentisati.

Ako se mučite da odmah smislite najbolji način, počnite sa dodavanjem onoliko state-ova da *definitivno* osigurate da su sva vizuelna stanja pokrivena:

```js
const [isEmpty, setIsEmpty] = useState(true);
const [isTyping, setIsTyping] = useState(false);
const [isSubmitting, setIsSubmitting] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
const [isError, setIsError] = useState(false);
```

Vaša prva ideja vrlo verovatno neće biti najbolja, ali to je u redu--refaktorisanje state-a je deo procesa!

### Korak 4: Ukloniti sve neobavezne state promenljive {/*step-4-remove-any-non-essential-state-variables*/}

Želite da izbegnete dupliranje unutar state-a tako da pratite samo ono što je obavezno. Ako potrošite malo vremena na refaktorisanje strukture vašeg state-a učinićete vaše komponente lakšim za razumevanje, smanjićete dupliranje i izbeći nenamerna značenja. Vaš cilj je da **sprečite slučaj u kojem state u memoriji ne predstavlja nijedan validan UI koji želite da korisnik vidi**. (Na primer, ne želite da prikažete poruku o grešci i onemogućite input istovremeno, pošto korisnik neće moći da ispravi grešku!)

Evo nekih pitanja koja možete postaviti za svoje state promenljive:

* **Da li ovaj state stvara paradoks?** Na primer, `isTyping` i `isSubmitting` ne mogu zajedno biti `true`. Paradoks uglavnom znači da state nije dovoljno ograničen. Postoje četiri moguće kombinacije dve boolean vrednosti, ali samo tri odgovaraju validnim stanjima. Da biste uklonili "nemoguće" stanje, možete ih ukombinovati u `status` koji mora imati jednu od tri vrednosti: `'typing'`, `'submitting'` ili `'success'`.
* **Da li je ista informacija već dostupna u drugoj state promenljivoj?** Još jedan paradoks: `isEmpty` i `isTyping` ne mogu biti `true` istovremeno. Ako ih napravite kao odvojene state promenljive rizikujete da ne budu sinhronizovane i pravite bug-ove. Srećom, možete ukloniti `isEmpty` i umesto toga proveriti `answer.length === 0`.
* **Da li istu informaciju možete dobiti inverzijom druge state promenljive?** `isError` nije potreban jer možete proveriti `error !== null`.

Nakon ovog čišćenja, ostajete sa 3 (od 7!) *obaveznih* state promenljivih:

```js
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [status, setStatus] = useState('typing'); // 'typing', 'submitting' ili 'success'
```

Znate da su obavezne, jer nijednu ne možete ukloniti a da ne skršite funkcionalnost.

<DeepDive>

#### Eliminisanje “nemogućih” stanja sa reducer-om {/*eliminating-impossible-states-with-a-reducer*/}

Ove tri promenljive su dovoljno dobre da predstavljaju stanje naše forme. Međutim, i dalje postoje neka posredna stanja koji nemaju skroz smisla. Na primer, `error` koji nije null nema smisla dok je `status` jednak `'success'`. Da biste modelovali state preciznije, možete [ga izdvojiti u reducer](/learn/extracting-state-logic-into-a-reducer). Reducer-i vam omogućavaju da objedinite state promenljive u jedan objekat i grupišete svu povezanu logiku!

</DeepDive>

### Korak 5: Povezati event handler-e za postavljanje state-a {/*step-5-connect-the-event-handlers-to-set-state*/}

Na kraju, kreirajte event handler-e koji ažuriraju state. Ispod je finalna forma sa svim povezanim event handler-ima:

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [status, setStatus] = useState('typing');

  if (status === 'success') {
    return <h1>To je tačno!</h1>
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    try {
      await submitForm(answer);
      setStatus('success');
    } catch (err) {
      setStatus('typing');
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <>
      <h2>Kviz gradova</h2>
      <p>
        U kom gradu je bilbord koji pretvara vazduh u pijaću vodu?
      </p>
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer}
          onChange={handleTextareaChange}
          disabled={status === 'submitting'}
        />
        <br />
        <button disabled={
          answer.length === 0 ||
          status === 'submitting'
        }>
          Submit
        </button>
        {error !== null &&
          <p className="Error">
            {error.message}
          </p>
        }
      </form>
    </>
  );
}

function submitForm(answer) {
  // Pretvaraj se da koristiš mrežni poziv.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== 'lima'
      if (shouldError) {
        reject(new Error('Dobar pokušaj, ali pogrešan odgovor. Probaj ponovo!'));
      } else {
        resolve();
      }
    }, 1500);
  });
}
```

```css
.Error { color: red; }
```

</Sandpack>

Iako je ovaj kod duži od originalnog imperativnog primera, manje je krt. Izražavanje svih interakcija kao promena stanja vam omogućava da kasnije uvedete nova vizuelna stanja bez da pokvarite postojeća. Takođe vam omogućava da promenite šta je prikazano u svakom stanju bez promene logike u samoj interakciji.

<Recap>

* Deklarativno programiranje označava opisivanje UI-a za svako vizuelno stanje, a ne mikromenadžment nad UI-jem (imperativno).
* Kada razvijate komponentu:
  1. Identifikujte sva njena vizuelna stanja.
  2. Odredite ljudske i računarske pokretače promena stanja.
  3. Modelujte stanje sa `useState`.
  4. Uklonite neobavezne state-ove da izbegnete bug-ove i paradokse.
  5. Povežite event handler-e da postavite state.

</Recap>



<Challenges>

#### Dodati i ukloniti CSS klasu {/*add-and-remove-a-css-class*/}

Napravite da klik na sliku *uklanja* `background--active` CSS klasu iz spoljašnjeg `<div>`-a, ali *dodaje* `picture--active` klasu u `<img>`. Klik na pozadinu bi trebao da povrati originalne CSS klase.

Vizuelno, trebate očekivati da klik na sliku uklanja ljubičastu pozadinu i ističe ivicu slike. Klik van slike ističe pozadinu, ali uklanja istaknutu ivicu slike.

<Sandpack>

```js
export default function Picture() {
  return (
    <div className="background background--active">
      <img
        className="picture"
        alt="Dugine kuće u Kampung Pelangi, Indonezija"
        src="https://i.imgur.com/5qwVYb1.jpeg"
      />
    </div>
  );
}
```

```css
body { margin: 0; padding: 0; height: 250px; }

.background {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #eee;
}

.background--active {
  background: #a6b5ff;
}

.picture {
  width: 200px;
  height: 200px;
  border-radius: 10px;
  border: 5px solid transparent;
}

.picture--active {
  border: 5px solid #a6b5ff;
}
```

</Sandpack>

<Solution>

Ova komponenta ima dva vizuelna stanja: kad je slika aktivna i kad slika nije aktivna:

* Kad je slika aktivna, CSS klase su `background` i `picture picture--active`.
* Kad slika nije aktivna, CSS klase su `background background--active` i `picture`.

Jedna boolean state promenljiva je dovoljna da upamti da li je slika aktivna. Originalni zadatak je bio da uklonite ili dodate CSS klase. Međutim, u React-u je potrebno da *opišete* šta želite videti, pre nego da *manipulišete* UI elementima. Znači da morate izračunati obe CSS klase na osnovu trenutnog state-a. Takođe trebate da [zaustavite propagaciju](/learn/responding-to-events#stopping-propagation) kako klik na sliku ne bi bio registrovan kao klik na pozadinu.

Potvrdite da ova verzija radi klikom na sliku, a onda klikom izvan nje:

<Sandpack>

```js
import { useState } from 'react';

export default function Picture() {
  const [isActive, setIsActive] = useState(false);

  let backgroundClassName = 'background';
  let pictureClassName = 'picture';
  if (isActive) {
    pictureClassName += ' picture--active';
  } else {
    backgroundClassName += ' background--active';
  }

  return (
    <div
      className={backgroundClassName}
      onClick={() => setIsActive(false)}
    >
      <img
        onClick={e => {
          e.stopPropagation();
          setIsActive(true);
        }}
        className={pictureClassName}
        alt="Dugine kuće u Kampung Pelangi, Indonezija"
        src="https://i.imgur.com/5qwVYb1.jpeg"
      />
    </div>
  );
}
```

```css
body { margin: 0; padding: 0; height: 250px; }

.background {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #eee;
}

.background--active {
  background: #a6b5ff;
}

.picture {
  width: 200px;
  height: 200px;
  border-radius: 10px;
  border: 5px solid transparent;
}

.picture--active {
  border: 5px solid #a6b5ff;
}
```

</Sandpack>

Alternativno, možete vratiti dva različita JSX-a:

<Sandpack>

```js
import { useState } from 'react';

export default function Picture() {
  const [isActive, setIsActive] = useState(false);
  if (isActive) {
    return (
      <div
        className="background"
        onClick={() => setIsActive(false)}
      >
        <img
          className="picture picture--active"
          alt="Dugine kuće u Kampung Pelangi, Indonezija"
          src="https://i.imgur.com/5qwVYb1.jpeg"
          onClick={e => e.stopPropagation()}
        />
      </div>
    );
  }
  return (
    <div className="background background--active">
      <img
        className="picture"
        alt="Dugine kuće u Kampung Pelangi, Indonezija"
        src="https://i.imgur.com/5qwVYb1.jpeg"
        onClick={() => setIsActive(true)}
      />
    </div>
  );
}
```

```css
body { margin: 0; padding: 0; height: 250px; }

.background {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #eee;
}

.background--active {
  background: #a6b5ff;
}

.picture {
  width: 200px;
  height: 200px;
  border-radius: 10px;
  border: 5px solid transparent;
}

.picture--active {
  border: 5px solid #a6b5ff;
}
```

</Sandpack>

Imajte na umu da ako imate dva različita JSX-a koja opisuju isto stablo, njihova ugnježdenost (prvi `<div>` → prvi `<img>`) mora da se podudara. U suprotnom, promena `isActive` bi iznova kreirala celo stablo ispod i [resetovala njegov state](/learn/preserving-and-resetting-state). Zbog toga, ako se slično JSX stablo vraća u oba slučaja, bolje je da ih napišete u jednom JSX-u.

</Solution>

#### Urediti profil {/*profile-editor*/}

Ovde je mala forma implementirana samo sa JavaScript-om i DOM-om. Igrajte se sa njom da biste razumeli ponašanje:

<Sandpack>

```js src/index.js active
function handleFormSubmit(e) {
  e.preventDefault();
  if (editButton.textContent === 'Izmeni profil') {
    editButton.textContent = 'Sačuvaj profil';
    hide(firstNameText);
    hide(lastNameText);
    show(firstNameInput);
    show(lastNameInput);
  } else {
    editButton.textContent = 'Izmeni profil';
    hide(firstNameInput);
    hide(lastNameInput);
    show(firstNameText);
    show(lastNameText);
  }
}

function handleFirstNameChange() {
  firstNameText.textContent = firstNameInput.value;
  helloText.textContent = (
    'Zdravo ' +
    firstNameInput.value + ' ' +
    lastNameInput.value + '!'
  );
}

function handleLastNameChange() {
  lastNameText.textContent = lastNameInput.value;
  helloText.textContent = (
    'Zdravo ' +
    firstNameInput.value + ' ' +
    lastNameInput.value + '!'
  );
}

function hide(el) {
  el.style.display = 'none';
}

function show(el) {
  el.style.display = '';
}

let form = document.getElementById('form');
let editButton = document.getElementById('editButton');
let firstNameInput = document.getElementById('firstNameInput');
let firstNameText = document.getElementById('firstNameText');
let lastNameInput = document.getElementById('lastNameInput');
let lastNameText = document.getElementById('lastNameText');
let helloText = document.getElementById('helloText');
form.onsubmit = handleFormSubmit;
firstNameInput.oninput = handleFirstNameChange;
lastNameInput.oninput = handleLastNameChange;
```

```js sandbox.config.json hidden
{
  "hardReloadOnChange": true
}
```

```html public/index.html
<form id="form">
  <label>
    Ime:
    <b id="firstNameText">Jane</b>
    <input
      id="firstNameInput"
      value="Jane"
      style="display: none">
  </label>
  <label>
    Prezime:
    <b id="lastNameText">Jacobs</b>
    <input
      id="lastNameInput"
      value="Jacobs"
      style="display: none">
  </label>
  <button type="submit" id="editButton">Izmeni profil</button>
  <p><i id="helloText">Zdravo, Jane Jacobs!</i></p>
</form>

<style>
* { box-sizing: border-box; }
body { font-family: sans-serif; margin: 20px; padding: 0; }
label { display: block; margin-bottom: 20px; }
</style>
```

</Sandpack>

Ova forma menja dva moda: u modu izmene vidite input-e, a u modu gledanja vidite samo rezultat. Labela dugmeta se menja iz "Izmeni" u "Sačuvaj" u zavisnosti od toga u kom ste modu. Dok menjate input-e, poruka na dnu se ažurira u realnom vremenu.

Vaš zadatak je da ovo implementirate iznova u React-u u sandbox-u ispod. Za vašu ugodnost, markup je već prebačen u JSX, ali vi trebate napraviti da prikazuje i skriva input-e kao i originalno rešenje.

Postarajte se i da ažurira tekst na dnu!

<Sandpack>

```js
export default function EditProfile() {
  return (
    <form>
      <label>
        Ime:{' '}
        <b>Jane</b>
        <input />
      </label>
      <label>
        Prezime:{' '}
        <b>Jacobs</b>
        <input />
      </label>
      <button type="submit">
        Izmeni profil
      </button>
      <p><i>Zdravo, Jane Jacobs!</i></p>
    </form>
  );
}
```

```css
label { display: block; margin-bottom: 20px; }
```

</Sandpack>

<Solution>

Trebaće vam dve state promenljive da čuvaju input vrednosti: `firstName` i `lastName`. Takođe, trebaće vam i `isEditing` state promenljiva koja određuje da li se prikazuju input-i ili ne. _Neće_ vam trebati `fullName` promenljiva jer se celo ime uvek može izračunati na osnovu `firstName` i `lastName`.

Konačno, trebate koristiti [uslovno renderovanje](/learn/conditional-rendering) da prikažete ili sakrijete input-e na osnovu vrednosti koju ima `isEditing`.

<Sandpack>

```js
import { useState } from 'react';

export default function EditProfile() {
  const [isEditing, setIsEditing] = useState(false);
  const [firstName, setFirstName] = useState('Jane');
  const [lastName, setLastName] = useState('Jacobs');

  return (
    <form onSubmit={e => {
      e.preventDefault();
      setIsEditing(!isEditing);
    }}>
      <label>
        Ime:{' '}
        {isEditing ? (
          <input
            value={firstName}
            onChange={e => {
              setFirstName(e.target.value)
            }}
          />
        ) : (
          <b>{firstName}</b>
        )}
      </label>
      <label>
        Prezime:{' '}
        {isEditing ? (
          <input
            value={lastName}
            onChange={e => {
              setLastName(e.target.value)
            }}
          />
        ) : (
          <b>{lastName}</b>
        )}
      </label>
      <button type="submit">
        {isEditing ? 'Sačuvaj' : 'Izmeni'} profil
      </button>
      <p><i>Zdravo, {firstName} {lastName}!</i></p>
    </form>
  );
}
```

```css
label { display: block; margin-bottom: 20px; }
```

</Sandpack>

Uporedite ovo rešenje sa originalnim imperativnim kodom. Kako se razlikuju?

</Solution>

#### Refaktorisati imperativno rešenje bez React-a {/*refactor-the-imperative-solution-without-react*/}

Ovde je originalni sandbox iz prethodnog izazova, napisan imperativno bez React-a:

<Sandpack>

```js src/index.js active
function handleFormSubmit(e) {
  e.preventDefault();
  if (editButton.textContent === 'Izmeni profil') {
    editButton.textContent = 'Sačuvaj profil';
    hide(firstNameText);
    hide(lastNameText);
    show(firstNameInput);
    show(lastNameInput);
  } else {
    editButton.textContent = 'Izmeni profil';
    hide(firstNameInput);
    hide(lastNameInput);
    show(firstNameText);
    show(lastNameText);
  }
}

function handleFirstNameChange() {
  firstNameText.textContent = firstNameInput.value;
  helloText.textContent = (
    'Zdravo ' +
    firstNameInput.value + ' ' +
    lastNameInput.value + '!'
  );
}

function handleLastNameChange() {
  lastNameText.textContent = lastNameInput.value;
  helloText.textContent = (
    'Zdravo ' +
    firstNameInput.value + ' ' +
    lastNameInput.value + '!'
  );
}

function hide(el) {
  el.style.display = 'none';
}

function show(el) {
  el.style.display = '';
}

let form = document.getElementById('form');
let editButton = document.getElementById('editButton');
let firstNameInput = document.getElementById('firstNameInput');
let firstNameText = document.getElementById('firstNameText');
let lastNameInput = document.getElementById('lastNameInput');
let lastNameText = document.getElementById('lastNameText');
let helloText = document.getElementById('helloText');
form.onsubmit = handleFormSubmit;
firstNameInput.oninput = handleFirstNameChange;
lastNameInput.oninput = handleLastNameChange;
```

```js sandbox.config.json hidden
{
  "hardReloadOnChange": true
}
```

```html public/index.html
<form id="form">
  <label>
    Ime:
    <b id="firstNameText">Jane</b>
    <input
      id="firstNameInput"
      value="Jane"
      style="display: none">
  </label>
  <label>
    Prezime:
    <b id="lastNameText">Jacobs</b>
    <input
      id="lastNameInput"
      value="Jacobs"
      style="display: none">
  </label>
  <button type="submit" id="editButton">Izmeni profil</button>
  <p><i id="helloText">Zdravo, Jane Jacobs!</i></p>
</form>

<style>
* { box-sizing: border-box; }
body { font-family: sans-serif; margin: 20px; padding: 0; }
label { display: block; margin-bottom: 20px; }
</style>
```

</Sandpack>

Zamislite da React nije postojao. Možete li refaktorisati ovaj kod na način da logika bude manje krta i sličnija React verziji? Kako bi to izgledalo da je state eksplicitan, kao u React-u?

Ako se mučite da razmislite odakle početi, stub ispod već ima postavljenu većinu strukture. Ako počinjete odavde, popunite logiku koja nedostaje u `updateDOM` funkciji. (Pogledajte originalni kod ako vam je potrebno.)

<Sandpack>

```js src/index.js active
let firstName = 'Jane';
let lastName = 'Jacobs';
let isEditing = false;

function handleFormSubmit(e) {
  e.preventDefault();
  setIsEditing(!isEditing);
}

function handleFirstNameChange(e) {
  setFirstName(e.target.value);
}

function handleLastNameChange(e) {
  setLastName(e.target.value);
}

function setFirstName(value) {
  firstName = value;
  updateDOM();
}

function setLastName(value) {
  lastName = value;
  updateDOM();
}

function setIsEditing(value) {
  isEditing = value;
  updateDOM();
}

function updateDOM() {
  if (isEditing) {
    editButton.textContent = 'Sačuvaj profil';
    // TODO: prikaži input-e, sakrij sadržaj
  } else {
    editButton.textContent = 'Izmeni profil';
    // TODO: sakrij input-e, prikaži sadržaj
  }
  // TODO: ažuriraj labele
}

function hide(el) {
  el.style.display = 'none';
}

function show(el) {
  el.style.display = '';
}

let form = document.getElementById('form');
let editButton = document.getElementById('editButton');
let firstNameInput = document.getElementById('firstNameInput');
let firstNameText = document.getElementById('firstNameText');
let lastNameInput = document.getElementById('lastNameInput');
let lastNameText = document.getElementById('lastNameText');
let helloText = document.getElementById('helloText');
form.onsubmit = handleFormSubmit;
firstNameInput.oninput = handleFirstNameChange;
lastNameInput.oninput = handleLastNameChange;
```

```js sandbox.config.json hidden
{
  "hardReloadOnChange": true
}
```

```html public/index.html
<form id="form">
  <label>
    Ime:
    <b id="firstNameText">Jane</b>
    <input
      id="firstNameInput"
      value="Jane"
      style="display: none">
  </label>
  <label>
    Prezime:
    <b id="lastNameText">Jacobs</b>
    <input
      id="lastNameInput"
      value="Jacobs"
      style="display: none">
  </label>
  <button type="submit" id="editButton">Izmeni profil</button>
  <p><i id="helloText">Zdravo, Jane Jacobs!</i></p>
</form>

<style>
* { box-sizing: border-box; }
body { font-family: sans-serif; margin: 20px; padding: 0; }
label { display: block; margin-bottom: 20px; }
</style>
```

</Sandpack>

<Solution>

Nedostajuća logika uključuje prikazivanje i skrivanje input-a i sadržaja, kao i ažuriranje labela:

<Sandpack>

```js src/index.js active
let firstName = 'Jane';
let lastName = 'Jacobs';
let isEditing = false;

function handleFormSubmit(e) {
  e.preventDefault();
  setIsEditing(!isEditing);
}

function handleFirstNameChange(e) {
  setFirstName(e.target.value);
}

function handleLastNameChange(e) {
  setLastName(e.target.value);
}

function setFirstName(value) {
  firstName = value;
  updateDOM();
}

function setLastName(value) {
  lastName = value;
  updateDOM();
}

function setIsEditing(value) {
  isEditing = value;
  updateDOM();
}

function updateDOM() {
  if (isEditing) {
    editButton.textContent = 'Sačuvaj profil';
    hide(firstNameText);
    hide(lastNameText);
    show(firstNameInput);
    show(lastNameInput);
  } else {
    editButton.textContent = 'Izmeni profil';
    hide(firstNameInput);
    hide(lastNameInput);
    show(firstNameText);
    show(lastNameText);
  }
  firstNameText.textContent = firstName;
  lastNameText.textContent = lastName;
  helloText.textContent = (
    'Zdravo ' +
    firstName + ' ' +
    lastName + '!'
  );
}

function hide(el) {
  el.style.display = 'none';
}

function show(el) {
  el.style.display = '';
}

let form = document.getElementById('form');
let editButton = document.getElementById('editButton');
let firstNameInput = document.getElementById('firstNameInput');
let firstNameText = document.getElementById('firstNameText');
let lastNameInput = document.getElementById('lastNameInput');
let lastNameText = document.getElementById('lastNameText');
let helloText = document.getElementById('helloText');
form.onsubmit = handleFormSubmit;
firstNameInput.oninput = handleFirstNameChange;
lastNameInput.oninput = handleLastNameChange;
```

```js sandbox.config.json hidden
{
  "hardReloadOnChange": true
}
```

```html public/index.html
<form id="form">
  <label>
    Ime:
    <b id="firstNameText">Jane</b>
    <input
      id="firstNameInput"
      value="Jane"
      style="display: none">
  </label>
  <label>
    Prezime:
    <b id="lastNameText">Jacobs</b>
    <input
      id="lastNameInput"
      value="Jacobs"
      style="display: none">
  </label>
  <button type="submit" id="editButton">Izmeni profil</button>
  <p><i id="helloText">Zdravo, Jane Jacobs!</i></p>
</form>

<style>
* { box-sizing: border-box; }
body { font-family: sans-serif; margin: 20px; padding: 0; }
label { display: block; margin-bottom: 20px; }
</style>
```

</Sandpack>

`updateDOM` funkcija koju ste napisali prikazuje šta React radi ispod haube kada postavite state. (Doduše, React takođe izbegava da dira DOM za stvari koje se nisu promenile od poslednjeg puta kad su bile postavljene.)

</Solution>

</Challenges>
