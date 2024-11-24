---
title: Uslovno renderovanje
---

<Intro>

Vaše komponente često trebaju prikazivati različite stvari u zavisnosti od različitih uslova. U React-u, možete uslovno renderovati JSX upotrebom JavaScript sintakse poput `if` iskaza, `&&` i `? :` operatora.

</Intro>

<YouWillLearn>

* Kako da vratite različit JSX u zavisnosti od uslova
* Kako da uslovno uključite ili isključite deo JSX-a
* Uobičajene prečice za uslovnu sintaksu koje ćete sresti u React projektima

</YouWillLearn>

## Uslovno vraćanje JSX-a {/*conditionally-returning-jsx*/}

Napravimo `PackingList` komponentu koja će renderovati par `Item`-a, koji mogu, a ne moraju biti spakovani:

<Sandpack>

```js
function Item({ name, isPacked }) {
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista za pakovanje od Sally Ride</h1>
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
          name="Fotografija od Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

Primetite da neke `Item` komponente imaju `isPacked` prop setovan na `true` umesto na `false`. Želimo dodati kvačicu (✅) za spakovane proizvode ako je `isPacked={true}`.

Ovo možete napisati kao [`if`/`else` izraz](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else) poput:

```js
if (isPacked) {
  return <li className="item">{name} ✅</li>;
}
return <li className="item">{name}</li>;
```

Ako je `isPacked` prop `true`, ovaj kod **vraća drugačije JSX stablo**. Sa ovom promenom, neki proizvodi će imati kvačicu na kraju:

<Sandpack>

```js
function Item({ name, isPacked }) {
  if (isPacked) {
    return <li className="item">{name} ✅</li>;
  }
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista za pakovanje od Sally Ride</h1>
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
          name="Fotografija od Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

Promenite šta se vraća u oba slučaja i vidite kako se rezultat menja!

Uočite da se kreira logika grananja sa JavaScript-ovim `if` i `return` iskazima. U React-u, JavaScript je zadužen za kontrolni tok (npr. uslove).

### Uslovno renderovanje ničega sa `null` {/*conditionally-returning-nothing-with-null*/}

U nekim situacijama, nećete želeti da renderujete ništa. Na primer, uopšte ne želite da prikažete spakovane proizvode. Komponenta mora nešto da vrati. U ovom slučaju, možete vratiti `null`:

```js
if (isPacked) {
  return null;
}
return <li className="item">{name}</li>;
```

Ako je `isPacked` true, komponenta će vratiti ništa, `null`. U suprotnom, vratiće JSX koji se renderuje.

<Sandpack>

```js
function Item({ name, isPacked }) {
  if (isPacked) {
    return null;
  }
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista za pakovanje od Sally Ride</h1>
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
          name="Fotografija od Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

U praksi, vraćanje `null` iz komponente nije uobičajeno jer može iznenaditi developera koji pokuša da ga renderuje. Češće ćete uslovno uključiti ili isključiti komponentu u JSX-u roditeljske komponente. Evo kako to možete uraditi!

## Uslovno uključivanje JSX-a {/*conditionally-including-jsx*/}

U prethodnom primeru, kontrolisali ste koje (možda nijedno!) JSX stablo će komponenta vratiti. Možda ste već uočili neko ponavljanje:

```js
<li className="item">{name} ✅</li>
```

je veoma slično sa

```js
<li className="item">{name}</li>
```

Obe uslovne grane vraćaju `<li className="item">...</li>`:

```js
if (isPacked) {
  return <li className="item">{name} ✅</li>;
}
return <li className="item">{name}</li>;
```

Iako ovo ponavljanje nije štetno, ipak čini vaš kod težim za održavanje. Šta ako želite promeniti `className`? Morali biste to učiniti na dva mesta u kodu! U ovakvim situacijama možete uslovno uključiti malo JSX-a da bi vaš kod bolje ispoštovao [DRY](https://sr.wikipedia.org/wiki/Don%27t_repeat_yourself).

### Uslovni (ternarni) operator (`? :`) {/*conditional-ternary-operator--*/}

JavaScript sadrži kompaktnu sintaksu za pisanje uslovnih izraza -- [uslovni operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) ili "ternarni operator".

Umesto ovoga:

```js
if (isPacked) {
  return <li className="item">{name} ✅</li>;
}
return <li className="item">{name}</li>;
```

Možete napisati ovo:

```js
return (
  <li className="item">
    {isPacked ? name + ' ✅' : name}
  </li>
);
```

Ovo možete pročitati kao *"ako je `isPacked` true, onda (`?`) renderuj `name + ' ✅'`, u suprotnom (`:`) renderuj `name`"*.

<DeepDive>

#### Da li su ovi primeri potpuno jednaki? {/*are-these-two-examples-fully-equivalent*/}

Ako dolazite sa pozadinom objektno-orijentisanog programiranja, mogli biste pretpostaviti da se dva primera gore suptilno razlikuju zato što jedan od njih može kreirati dve "instance" `<li>`. Ali, JSX elementi nisu "instance" zato što ne drže stanje i nisu pravi DOM čvorovi. Oni si laki opisi, nalik na nacrte. Tako da ova dva primera, u suštini, *jesu* potpuno jednaki. [Očuvanje i resetovanje state-a](/learn/preserving-and-resetting-state) detaljno opisuje kako ovo radi.

</DeepDive>

Hajde sad da obmotamo tekst spakovanog proizvoda u novi tag, `<del>`, kako bi ga precrtali. Možete dodati i nove linije i zagrade kako bi bilo lakše da ugnjezdite dodatni JSX u oba slučaja:

<Sandpack>

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {isPacked ? (
        <del>
          {name + ' ✅'}
        </del>
      ) : (
        name
      )}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista za pakovanje od Sally Ride</h1>
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
          name="Fotografija od Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

Ovaj stil radi dobro za jednostavne uslove, ali koristite ga umereno. Ako vaše komponente postanu neuredne zbog previše ugnježdenih uslovnih markup-a, razmotrite izdvajanje dečjih komponenata kako biste ih poboljšali. U React-u, markup je deo vašeg koda, tako da možete koristiti alate poput promenljivih i funkcija da biste pojednostavili kompleksne izraze.

### Logički "I" operator (`&&`) {/*logical-and-operator-*/}

Još jedna uobičajena prečica koju ćete sresti je [JavaScript-ov logički "I" (`&&`) operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND). Često se koristi unutar React komponenata kada želite da renderujete neki JSX kad je uslov true, **ili da ne renderujete ništa u suprotnom**. Sa `&&` možete uslovno renderovati kvačicu samo kad je `isPacked` `true`:

```js
return (
  <li className="item">
    {name} {isPacked && '✅'}
  </li>
);
```

Ovo možete pročitati kao *"ako je `isPacked`, onda (`&&`) renderuj kvačicu, u suprotnom ne renderuj ništa"*.

Evo primera:

<Sandpack>

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked && '✅'}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista za pakovanje od Sally Ride</h1>
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
          name="Fotografija od Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

[JavaScript && izraz](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_AND) vraća vrednost svoje desne strane (u našem slučaju, kvačicu) ako je leva strana (naš uslov) `true`. Ali, ako je uslov `false`, ceo izraz postaje `false`. React tumači `false` kao "rupu" u JSX stablu, kao `null` ili `undefined`, i ne renderuje ništa na tom mestu.


<Pitfall>

**Ne koristite brojeve na levoj strani od `&&`.**

Da bi se testirao uslov, JavaScript automatski konvertuje levu stranu u boolean. Međutim, ako je leva strana `0`, onda ceo izraz dobija tu vrednost (`0`), a React će radije renderovati `0` nego ništa.

Na primer, česta greška je `messageCount && <p>New messages</p>`. Lako je pretpostaviti da to neće renderovati ništa kad `messageCount` ima vrednost `0`, ali će zapravo renderovati `0`!

Da biste to popravili, koristite boolean na levoj strani: `messageCount > 0 && <p>New messages</p>`.

</Pitfall>

### Uslovna dodela JSX-a u promenljivu {/*conditionally-assigning-jsx-to-a-variable*/}

Kada vas prečice sprečavaju da pišete običan kod, pokušajte da koristite `if` iskaz i promenljivu. Možete dodeljivati vrednost promenljivama definisanim sa [`let`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let), pa počnite sa default sadržajem koji želite prikazati, imenom:

```js
let itemContent = name;
```

Koristite `if` iskaz da ponovo dodelite JSX izraz u `itemContent` ako je `isPacked` `true`:

```js
if (isPacked) {
  itemContent = name + " ✅";
}
```

[Vitičaste zagrade otvaraju "prozor u JavaScript".](/learn/javascript-in-jsx-with-curly-braces#using-curly-braces-a-window-into-the-javascript-world) Ugradite promenljivu sa vitičastim zagradama u povratno JSX stablo ugnježdavanjem prethodno izračunatog izraza unutar JSX-a:

```js
<li className="item">
  {itemContent}
</li>
```

Ovaj stil je najopširniji, ali je i najfleksibilniji. Evo primera:

<Sandpack>

```js
function Item({ name, isPacked }) {
  let itemContent = name;
  if (isPacked) {
    itemContent = name + " ✅";
  }
  return (
    <li className="item">
      {itemContent}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista za pakovanje od Sally Ride</h1>
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
          name="Fotografija od Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

Kao i ranije, ovo ne radi samo za tekst, već i za proizvoljan JSX:

<Sandpack>

```js
function Item({ name, isPacked }) {
  let itemContent = name;
  if (isPacked) {
    itemContent = (
      <del>
        {name + " ✅"}
      </del>
    );
  }
  return (
    <li className="item">
      {itemContent}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista za pakovanje od Sally Ride</h1>
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
          name="Fotografija od Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

Ako niste upoznati sa JavaScript-om, ovolika raznovrsnost stilova može delovati preobimno na početku. Međutim, učenje tih stilova će vam pomoći da čitate i pišete bilo koji JavaScript kod -- ne samo React komponente! Za početak izaberite jedan, a kasnije pogledajte ovaj članak ponovo ako zaboravite kako ostali funkcionišu.

<Recap>

* U React-u, logiku grananja kontrolišete pomoću JavaScript-a.
* Možete vratiti JSX izraz uslovno pomoću `if` iskaza.
* Možete uslovno sačuvati neki JSX u promenljivu, a onda ga uključiti unutar drugog JSX-a pomoću vitičastih zagrada.
* U JSX-u, `{cond ? <A /> : <B />}` znači *"ako je `cond`, renderuj `<A />`, u suprotnom `<B />`"*.
* U JSX-u, `{cond && <A />}` znači *"ako je `cond`, renderuj `<A />`, u suprotnom ništa"*.
* Prečice su česte, ali ne morate ih koristiti ako preferirate običan `if`.

</Recap>



<Challenges>

#### Prikazati ikonicu za proizvode koji nisu spakovani pomoću `? :` {/*show-an-icon-for-incomplete-items-with--*/}

Koristite uslovni operator (`cond ? a : b`) da renderujete ❌ ako `isPacked` nije `true`.

<Sandpack>

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked && '✅'}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista za pakovanje od Sally Ride</h1>
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
          name="Fotografija od Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

<Solution>

<Sandpack>

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked ? '✅' : '❌'}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista za pakovanje od Sally Ride</h1>
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
          name="Fotografija od Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

</Solution>

#### Prikazati važnost proizvoda pomoću `&&` {/*show-the-item-importance-with-*/}

U ovom primeru, svaki `Item` prima numerički `importance` prop. Koristite `&&` operator da renderujete "_(Važnost: X)_" u italic fontu, ali samo za proizvode koji nemaju važnost nula. Vaša lista proizvoda bi trebala da izgleda ovako:

* Svemirsko odelo _(Važnost: 9)_
* Kaciga sa zlatnim listom
* Fotografija od Tam _(Važnost: 6)_

Ne zaboravite da dodate razmak između dve labele!

<Sandpack>

```js
function Item({ name, importance }) {
  return (
    <li className="item">
      {name}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista za pakovanje od Sally Ride</h1>
      <ul>
        <Item 
          importance={9} 
          name="Svemirsko odelo" 
        />
        <Item 
          importance={0} 
          name="Kaciga sa zlatnim listom" 
        />
        <Item 
          importance={6} 
          name="Fotografija od Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

<Solution>

Ovo će uraditi posao:

<Sandpack>

```js
function Item({ name, importance }) {
  return (
    <li className="item">
      {name}
      {importance > 0 && ' '}
      {importance > 0 &&
        <i>(Važnost: {importance})</i>
      }
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Lista za pakovanje od Sally Ride</h1>
      <ul>
        <Item 
          importance={9} 
          name="Svemirsko odelo" 
        />
        <Item 
          importance={0} 
          name="Kaciga sa zlatnim listom" 
        />
        <Item 
          importance={6} 
          name="Fotografija od Tam" 
        />
      </ul>
    </section>
  );
}
```

</Sandpack>

Primetite da morate napisati `importance > 0 && ...` umesto `importance && ...` da u slučaju kad je `importance` jednako `0`, `0` ne bude renderovana kao rezultat!

U ovom rešenju upotrebljena su dva uslova da bi se ubacio razmak između imena i labele za važnost. Alternativno, možete koristiti Fragment sa razmakom na početku: `importance > 0 && <> <i>...</i></>` ili dodati razmak unutar `<i>`:  `importance > 0 && <i> ...</i>`.

</Solution>

#### Refaktorisati više `? :` u `if` i promenljive {/*refactor-a-series-of---to-if-and-variables*/}

Ova `Drink` komponenta koristi više `? :` uslova da prikaže različite informacije u zavisnosti da li `name` prop ima vrednost `"čaj"` ili `"kafa"`. Problem je u tome što se informacija o svakom piću prostire na više uslova. Refaktorišite kod da koristi jedan `if` iskaz umesto tri `? :` uslova.

<Sandpack>

```js
function Drink({ name }) {
  return (
    <section>
      <h1>{name}</h1>
      <dl>
        <dt>Deo biljke</dt>
        <dd>{name === 'čaj' ? 'list' : 'zrno'}</dd>
        <dt>Sadržaj kofeina</dt>
        <dd>{name === 'čaj' ? '15–70 mg/šolja' : '80–185 mg/šolja'}</dd>
        <dt>Starost</dt>
        <dd>{name === 'čaj' ? '4,000+ godina' : '1,000+ godina'}</dd>
      </dl>
    </section>
  );
}

export default function DrinkList() {
  return (
    <div>
      <Drink name="čaj" />
      <Drink name="kafa" />
    </div>
  );
}
```

</Sandpack>

Nakon što ste refaktorisali kod da koristi `if`, da li imate ideju kako još da ovo pojednostavite?

<Solution>

Ima više načina na koji ovo možete uraditi, ali evo početne tačke:

<Sandpack>

```js
function Drink({ name }) {
  let part, caffeine, age;
  if (name === 'čaj') {
    part = 'list';
    caffeine = '15–70 mg/šolja';
    age = '4,000+ godina';
  } else if (name === 'kafa') {
    part = 'zrno';
    caffeine = '80–185 mg/šolja';
    age = '1,000+ godina';
  }
  return (
    <section>
      <h1>{name}</h1>
      <dl>
        <dt>Deo biljke</dt>
        <dd>{part}</dd>
        <dt>Sadržaj kofeina</dt>
        <dd>{caffeine}</dd>
        <dt>Starost</dt>
        <dd>{age}</dd>
      </dl>
    </section>
  );
}

export default function DrinkList() {
  return (
    <div>
      <Drink name="čaj" />
      <Drink name="kafa" />
    </div>
  );
}
```

</Sandpack>

Ovde su informacije o svakom piću grupisane umesto da se nalaze u više uslova. Ovo olakšava dodavanje drugih pića u budućnosti.

Drugo rešenje bi bilo potpuno uklanjanje uslova pomeranjem informacija u objekte:

<Sandpack>

```js
const drinks = [
  {name: 'čaj', part: 'list', caffeine: '15–70 mg/šolja', age: '4,000+ godina'},
  {name: 'kafa', part: 'zrno', caffeine: '80–185 mg/šolja', age: '1,000+ godina'}
];

function Drink({ name }) {
  const info = drinks.find(d => d.name === name);
  return (
    <section>
      <h1>{info.name}</h1>
      <dl>
        <dt>Deo biljke</dt>
        <dd>{info.part}</dd>
        <dt>Sadržaj kofeina</dt>
        <dd>{info.caffeine}</dd>
        <dt>Starost</dt>
        <dd>{info.age}</dd>
      </dl>
    </section>
  );
}

export default function DrinkList() {
  return (
    <div>
      <Drink name="čaj" />
      <Drink name="kafa" />
    </div>
  );
}
```

</Sandpack>

</Solution>

</Challenges>
