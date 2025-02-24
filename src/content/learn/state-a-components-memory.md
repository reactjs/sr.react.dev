---
title: "State: Memorija komponente"
---

<Intro>

Komponente često trebaju promeniti prikaz na ekranu u zavisnosti od interakcija. Pisanje u formu bi trebalo da osveži input polje, klik na "naredno" u carousel-u za slike treba promeniti prikazanu sliku, klik na "kupi" bi trebao da stavi proizvod u korpu. Komponente trebaju da "pamte" stvari: trenutnu vrednost input-a, trenutnu sliku, korpu sa proizvodima. U React-u, ova vrsta memorije koja je specifična za komponentu se naziva *state*.

</Intro>

<YouWillLearn>

* Kako da dodate state promenljivu sa [`useState`](/reference/react/useState) Hook-om
* Koji par vrednosti vraća `useState` Hook
* Kako da dodate više od jedne state promenljive
* Zašto se state naziva lokalnim

</YouWillLearn>

## Kad obična promenljiva nije dovoljna {/*when-a-regular-variable-isnt-enough*/}

Ovde je komponenta koja renderuje sliku skulpture. Klikom na dugme "Naredno" trebala bi se prikazati naredna skulptura promenom `index` promenljive na `1`, pa na `2` i tako dalje. Međutim, ovo **neće raditi** (možete probati!):

<Sandpack>

```js
import { sculptureList } from './data.js';

export default function Gallery() {
  let index = 0;

  function handleClick() {
    index = index + 1;
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>
        Naredno
      </button>
      <h2>
        <i>{sculpture.name} </i> 
        napravio/la {sculpture.artist}
      </h2>
      <h3>  
        ({index + 1} od {sculptureList.length})
      </h3>
      <img 
        src={sculpture.url} 
        alt={sculpture.alt}
      />
      <p>
        {sculpture.description}
      </p>
    </>
  );
}
```

```js src/data.js
export const sculptureList = [{
  name: 'Homenaje a la Neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: 'Iako je Marta Colvin pretežno poznata po apstraktnim temama koje aludiraju na pre-hispanske simbole, ova velika skulptura, koja je omaž neurohirurgiji, jedna je od njenih najpoznatijih remek dela.',
  url: 'https://i.imgur.com/Mx7dA2Y.jpg',
  alt: 'Bronzana statua dve skrštene ruke koje pažljivo drže ljudski mozak vrhovima prstiju.'  
}, {
  name: 'Floralis Genérica',
  artist: 'Eduardo Catalano',
  description: 'Ovaj ogromni (75 ft. ili 23m) srebrni cvet se nalazi u Buenos Ajresu. Dizajniran je da se pomera, zatvarajući svoje latice uveče ili kada duvaju jaki vetrovi, a otvarajući ih ujutru.',
  url: 'https://i.imgur.com/ZF6s192m.jpg',
  alt: 'Velika metalna skulptura cveta sa reflektujućim laticama nalik na ogledalo i snažnim prašnicima.'
}, {
  name: 'Eternal Presence',
  artist: 'John Woodrow Wilson',
  description: 'Wilson je poznat po svojoj preokupiranosti jednakošću, socijalnom pravdom, kao i suštinskim i duhovnim kvalitetima čovečanstva. Ova masivna (7ft. ili 2,13m) bronzana skulptura predstavlja ono što je on objasnio kao "simbolično crno prisustvo prožeto osećajem univerzalne humanosti".',
  url: 'https://i.imgur.com/aTtVpES.jpg',
  alt: 'Skulptura ljudske glave koja deluje sveprisutno i svečano. Zrači mirnoćom i spokojstvom.'
}, {
  name: 'Moai',
  artist: 'Nepoznati umetnik',
  description: 'Nalazi se na Uskršnjem ostrvu, gde postoji oko 1,000 moai očuvanih monumentalnih skulptura, koje su napravili prvobitni Rapa Nui ljudi, za koje se veruje da predstavljaju pretke Bogova.',
  url: 'https://i.imgur.com/RCwLEoQm.jpg',
  alt: 'Tri monumentalne biste sa neproporcionalno velikim glavama i namrštenim licima.'
}, {
  name: 'Blue Nana',
  artist: 'Niki de Saint Phalle',
  description: 'Nane su trijumfalna stvorenja, simboli ženstvenosti i majčinstva. Inicijalno, Saint Phalle je koristila tkaninu i pronađene objekte za Nane, ali je kasnije uvela poliester kako bi napravila živopisniji dojam.',
  url: 'https://i.imgur.com/Sd1AgUOm.jpg',
  alt: 'Velika mozaička skulptura neobične ženske figure u raznobojnom kostimu koji odiše radošću.'
}, {
  name: 'Ultimate Form',
  artist: 'Barbara Hepworth',
  description: 'Ova apstraktna bronzana skulptura je deo The Family of Man serije koja se nalazi u Jorkširskom parku skulptura. Hepworth je odlučila da ne pravi bukvalne reprezentacije sveta, već apstraktne forme inspirisane ljudima i pejzažima.',
  url: 'https://i.imgur.com/2heNQDcm.jpg',
  alt: 'Visoka skulptura tri elementa jedna na drugom koja podseća na ljudsku figuru.'
}, {
  name: 'Cavaliere',
  artist: 'Lamidi Olonade Fakeye',
  description: "Pošto potiče od četiri generacije drvorezbara, Fakeye-ova dela su mešavina tradicionalne i savremene Joruba tematike.",
  url: 'https://i.imgur.com/wIdGuZwm.png',
  alt: 'Složena drvena skulptura ratnika čije lice je fokusirano na konja ukrašenim šarama.'
}, {
  name: 'Big Bellies',
  artist: 'Alina Szapocznikow',
  description: "Alina je poznata po skulpturama izdeljenog tela kao metafore za krhkost i nestalnost mladosti i lepote. Ova skulptura prikazuje dva veoma realistična velika stomaka jednog na drugom, gde je svaki visok oko 5 stopa (1,5m).",
  url: 'https://i.imgur.com/AlHTAdDm.jpg',
  alt: 'Ova skulptura podseća na slapove nabora i dosta se razlikuje od stomaka na klasičnim skulpturama.'
}, {
  name: 'Terracotta Army',
  artist: 'Nepoznati umetnik',
  description: 'Terracotta Army je kolekcija terakota skulptura koje predstavljaju armiju Qin Shi Huang-a, prvog cara Kine. Armija se sastojala od preko 8,000 vojnika, 130 kola sa 520 konja i 150 članova konjice.',
  url: 'https://i.imgur.com/HMFmH6m.jpg',
  alt: '12 terakota skulptura svečanih vojnika, od kojih svaki ima jedinstvenu facijalnu ekspresiju i oklop.'
}, {
  name: 'Lunar Landscape',
  artist: 'Louise Nevelson',
  description: 'Nevelson je poznata po uklanjanju objekata iz ruševina Njujorka, od kojih je kasnije pravila monumentalne konstrukcije. Za ovo je koristila rasparene delove poput noge od kreveta, palice za žongliranje i delove sedišta, zakucavajući ih i lepeći u kutije koje podsećaju na kubističku geometrijsku apstrakciju prostora i forme.',
  url: 'https://i.imgur.com/rN7hY6om.jpg',
  alt: 'Crna mat skulptura gde su pojedinačni delovi na početku bili neprimetni.'
}, {
  name: 'Aureole',
  artist: 'Ranjani Shettar',
  description: 'Shettar spaja tradicionalno i moderno, prirodno i industrijsko. Njena umetnost se fokusira na vezu između čoveka i prirode. Njen rad se opisuje kao ubedljiv i apstraktno i figurativno, prokoseći gravitaciji, kao i "dobra sinteza neobičnih materijala".',
  url: 'https://i.imgur.com/okTpbHhm.jpg',
  alt: 'Bleda skulptura nalik na žicu postavljena na betonski zid spuštajući se na pod. Izgleda lako.'
}, {
  name: 'Hippos',
  artist: 'Taipei Zoo',
  description: 'Zoološki vrt u Tajpeju je naručio trg nilskih konja koji se igraju zaronjeni.',
  url: 'https://i.imgur.com/6o5Vuyu.jpg',
  alt: 'Grupa bronzanih skulptura nilskih konja koji kao da isplivavaju na trotoar.'
}];
```

```css
h2 { margin-top: 10px; margin-bottom: 0; }
h3 {
  margin-top: 5px;
  font-weight: normal;
  font-size: 100%;
}
img { width: 120px; height: 120px; }
button {
  display: block;
  margin-top: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

`handleClick` event handler ažurira lokalnu promenljivu `index`. Ali, dve stvari sprečavaju tu promenu da bude primenjena:

1. **Lokalne promenljive se ne čuvaju između rendera.** Kada React renderuje komponentu drugi put, renderuje je iznova—ne uzima u obzir nikakve promene u lokalnim promenljivama.
2. **Promene lokalnih promenljiva neće okinuti renderovanje.** React ne shvata da treba da renderuje komponentu ponovo sa novim podacima.

Da biste ažurirali komponentu sa novim podacima, potrebne su dve stvari:

1. **Zadržavanje** podataka između rendera.
2. **Okidanje** renderovanja komponente sa novim podacima (ponovno renderovanje).

[`useState`](/reference/react/useState) Hook omogućava te dve stvari:

1. **state promenljiva** zadržava podatke između rendera.
2. **state setter funkcija** ažurira promenljivu i tera React da okine renderovanje komponente ponovo.

## Dodavanje state promenljive {/*adding-a-state-variable*/}

Da bi ste dodali state promenljivu, import-ujte `useState` iz React-a na vrhu fajla:

```js
import { useState } from 'react';
```

Onda, zamenite ovu liniju:

```js
let index = 0;
```

sa

```js
const [index, setIndex] = useState(0);
```

`index` je state promenljiva, a `setIndex` je setter funkcija.

> Sintaksa sa `[` i `]` se naziva [dekonstruisanje niza](https://javascript.info/destructuring-assignment) i omogućava vam da čitate vrednosti iz niza. Niz koji `useState` vraća uvek ima tačno dva člana.

Evo kako rade zajedno u `handleClick`:

```js
function handleClick() {
  setIndex(index + 1);
}
```

Sada, klikom na dugme "Naredno" promeniće se odabrana skulptura:

<Sandpack>

```js
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);

  function handleClick() {
    setIndex(index + 1);
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>
        Naredno
      </button>
      <h2>
        <i>{sculpture.name} </i> 
        napravio/la {sculpture.artist}
      </h2>
      <h3>  
        ({index + 1} od {sculptureList.length})
      </h3>
      <img 
        src={sculpture.url} 
        alt={sculpture.alt}
      />
      <p>
        {sculpture.description}
      </p>
    </>
  );
}
```

```js src/data.js
export const sculptureList = [{
  name: 'Homenaje a la Neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: 'Iako je Marta Colvin pretežno poznata po apstraktnim temama koje aludiraju na pre-hispanske simbole, ova ogromna skulptura, koja je omaž neurohirurgiji, jedna je od njenih najpoznatijih remek dela.',
  url: 'https://i.imgur.com/Mx7dA2Y.jpg',
  alt: 'Bronzana statua dve skrštene ruke koje pažljivo drže ljudski mozak vrhovima prstiju.'  
}, {
  name: 'Floralis Genérica',
  artist: 'Eduardo Catalano',
  description: 'Ovaj ogromni (75 ft. ili 23m) srebrni cvet se nalazi u Buenos Ajresu. Dizajniran je da se pomera, zatvarajući svoje latice uveče ili kada duvaju jaki vetrovi, a otvarajući ih ujutru.',
  url: 'https://i.imgur.com/ZF6s192m.jpg',
  alt: 'Velika metalna skulptura cveta sa reflektujućim laticama nalik na ogledalo i snažnim prašnicima.'
}, {
  name: 'Eternal Presence',
  artist: 'John Woodrow Wilson',
  description: 'Wilson je poznat po svojoj preokupiranosti jednakošću, socijalnom pravdom, kao i suštinskim i duhovnim kvalitetima čovečanstva. Ova masivna (7ft. ili 2,13m) bronzana skulptura predstavlja ono što je on objasnio kao "simbolično crno prisustvo prožeto osećajem univerzalne humanosti".',
  url: 'https://i.imgur.com/aTtVpES.jpg',
  alt: 'Skulptura ljudske glave koja deluje sveprisutno i svečano. Zrači mirnoćom i spokojstvom.'
}, {
  name: 'Moai',
  artist: 'Nepoznati umetnik',
  description: 'Nalazi se na Uskršnjem ostrvu, gde postoji oko 1,000 moai očuvanih monumentalnih skulptura, koje su napravili prvobitni Rapa Nui ljudi, za koje se veruje da predstavljaju pretke Bogova.',
  url: 'https://i.imgur.com/RCwLEoQm.jpg',
  alt: 'Tri monumentalne biste sa neproporcionalno velikim glavama i namrštenim licima.'
}, {
  name: 'Blue Nana',
  artist: 'Niki de Saint Phalle',
  description: 'Nane su trijumfalna stvorenja, simboli ženstvenosti i majčinstva. Inicijalno, Saint Phalle je koristila tkaninu i pronađene objekte za Nane, ali je kasnije uvela poliester kako bi napravila živopisniji dojam.',
  url: 'https://i.imgur.com/Sd1AgUOm.jpg',
  alt: 'Velika mozaička skulptura neobične ženske figure u raznobojnom kostimu koji odiše radošću.'
}, {
  name: 'Ultimate Form',
  artist: 'Barbara Hepworth',
  description: 'Ova apstraktna bronzana skulptura je deo The Family of Man serije koja se nalazi u Jorkširskom parku skulptura. Hepworth je odlučila da ne pravi bukvalne reprezentacije sveta, već apstraktne forme inspirisane ljudima i pejzažima.',
  url: 'https://i.imgur.com/2heNQDcm.jpg',
  alt: 'Visoka skulptura tri elementa jedna na drugom koja podseća na ljudsku figuru.'
}, {
  name: 'Cavaliere',
  artist: 'Lamidi Olonade Fakeye',
  description: "Pošto potiče od četiri generacije drvorezbara, Fakeye-ova dela su mešavina tradicionalne i savremene Joruba tematike.",
  url: 'https://i.imgur.com/wIdGuZwm.png',
  alt: 'Složena drvena skulptura ratnika čije lice je fokusirano na konja ukrašenim šarama.'
}, {
  name: 'Big Bellies',
  artist: 'Alina Szapocznikow',
  description: "Alina je poznata po skulpturama izdeljenog tela kao metafore za krhkost i nestalnost mladosti i lepote. Ova skulptura prikazuje dva veoma realistična velika stomaka jednog na drugom, gde je svaki visok oko 5 stopa (1,5m).",
  url: 'https://i.imgur.com/AlHTAdDm.jpg',
  alt: 'Ova skulptura podseća na slapove nabora i dosta se razlikuje od stomaka na klasičnim skulpturama.'
}, {
  name: 'Terracotta Army',
  artist: 'Nepoznati umetnik',
  description: 'Terracotta Army je kolekcija terakota skulptura koje predstavljaju armiju Qin Shi Huang-a, prvog cara Kine. Armija se sastojala od preko 8,000 vojnika, 130 kola sa 520 konja i 150 članova konjice.',
  url: 'https://i.imgur.com/HMFmH6m.jpg',
  alt: '12 terakota skulptura svečanih vojnika, od kojih svaki ima jedinstvenu facijalnu ekspresiju i oklop.'
}, {
  name: 'Lunar Landscape',
  artist: 'Louise Nevelson',
  description: 'Nevelson je poznata po uklanjanju objekata iz ruševina Njujorka, od kojih je kasnije pravila monumentalne konstrukcije. Za ovo je koristila rasparene delove poput noge od kreveta, palice za žongliranje i delove sedišta, zakucavajući ih i lepeći u kutije koje podsećaju na kubističku geometrijsku apstrakciju prostora i forme.',
  url: 'https://i.imgur.com/rN7hY6om.jpg',
  alt: 'Crna mat skulptura gde su pojedinačni delovi na početku bili neprimetni.'
}, {
  name: 'Aureole',
  artist: 'Ranjani Shettar',
  description: 'Shettar spaja tradicionalno i moderno, prirodno i industrijsko. Njena umetnost se fokusira na vezu između čoveka i prirode. Njen rad se opisuje kao ubedljiv i apstraktno i figurativno, prokoseći gravitaciji, kao i "dobra sinteza neobičnih materijala".',
  url: 'https://i.imgur.com/okTpbHhm.jpg',
  alt: 'Bleda skulptura nalik na žicu postavljena na betonski zid spuštajući se na pod. Izgleda lako.'
}, {
  name: 'Hippos',
  artist: 'Taipei Zoo',
  description: 'Zoološki vrt u Tajpeju je naručio trg nilskih konja koji se igraju zaronjeni.',
  url: 'https://i.imgur.com/6o5Vuyu.jpg',
  alt: 'Grupa bronzanih skulptura nilskih konja koji kao da isplivavaju na trotoar.'
}];
```

```css
h2 { margin-top: 10px; margin-bottom: 0; }
h3 {
 margin-top: 5px;
 font-weight: normal;
 font-size: 100%;
}
img { width: 120px; height: 120px; }
button {
  display: block;
  margin-top: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

### Upoznajte svoj prvi Hook {/*meet-your-first-hook*/}

U React-u, `useState`, kao i ostale funkcije koje počinju sa "`use`", naziva se Hook.

*Hook-ovi* su specijalne funkcije koje su dostupne isključivo u toku React [renderovanja](/learn/render-and-commit#step-1-trigger-a-render) (što ćemo objasniti detaljnije na narednoj stranici). Omogućavaju vam da "se zakačite" za različite funkcionalnosti React-a.

State je samo jedna od tih funkcionalnosti, ali ćete upoznati i ostale Hook-ove kasnije.

<Pitfall>

**Hook-ovi—funkcije koje počinju sa `use`—mogu biti pozvane samo na početku vaših komponenata ili [vaših Hooks-ova](/learn/reusing-logic-with-custom-hooks)**. Ne možete pozivati Hook-ove unutar uslova, petlji, ili drugih ugnježdenih funkcija. Hook-ovi su funkcije, ali je korisno da na njih gledate kao bezuslovne deklaracije onoga što je komponenti potrebno. "Koristite" React funkcionalnosti na vrhu komponente slično kao što "import-ujete" module na vrhu fajla.

</Pitfall>

### Anatomija `useState`-a {/*anatomy-of-usestate*/}

Kada pozovete [`useState`](/reference/react/useState), govorite React-u da želite da ta komponenta nešto zapamti:

```js
const [index, setIndex] = useState(0);
```

U ovom slučaju, želite da React zapamti `index`.

<Note>

Konvencija je da se par nazove ovako `const [something, setSomething]`. Možete staviti bilo koja imena, ali konvencije čine kod lakšim za razumevanje na različitim projektima.

</Note>

Jedini argument u `useState` je **inicijalna vrednost** vaše state promenljive. U ovom primeru, inicijalna vrednost `index`-a je postavljena na `0` sa `useState(0)`. 

Svaki put kad se komponenta renderuje, `useState` vam daje niz koji sadrži dve vrednosti:

1. **state promenljiva** (`index`) sa vrednošću koju ste sačuvali.
2. **state setter funkcija** (`setIndex`) koja može da ažurira state promenljivu i natera React da okine renderovanje komponente ponovo.

Evo toga i u praksi:

```js
const [index, setIndex] = useState(0);
```

1. **Vaša komponenta se renderuje prvi put.** Pošto ste prosledili `0` u `useState` kao inicijalnu vrednost za `index`, vratiće `[0, setIndex]`. React pamti da je `0` poslednja state vrednost.
2. **Ažurirate state.** Kada korisnik klikne na dugme poziva se `setIndex(index + 1)`. `index` je `0`, tako da je to `setIndex(1)`. Ovo govori React-u da zapamti da je `index` sada `1` i okida novi render.
3. **Drugo renderovanje vaše komponente.** React i dalje vidi `useState(0)`, ali zato što React *pamti* da ste setovali `index` na `1`, vratiće `[1, setIndex]`.
4. I tako dalje!

## Davanje komponenti više state promenljivih {/*giving-a-component-multiple-state-variables*/}

Možete imati koliko god state promenljivih sa koliko god tipova podataka želite u jednoj komponenti. Ova komponenta ima dve state promenljive, broj `index` i boolean `showMore` koji se menja klikom na "Prikaži detalje":

<Sandpack>

```js
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  function handleNextClick() {
    setIndex(index + 1);
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleNextClick}>
        Naredno
      </button>
      <h2>
        <i>{sculpture.name} </i> 
        napravio/la {sculpture.artist}
      </h2>
      <h3>  
        ({index + 1} od {sculptureList.length})
      </h3>
      <button onClick={handleMoreClick}>
        {showMore ? 'Sakrij' : 'Prikaži'} detalje
      </button>
      {showMore && <p>{sculpture.description}</p>}
      <img 
        src={sculpture.url} 
        alt={sculpture.alt}
      />
    </>
  );
}
```

```js src/data.js
export const sculptureList = [{
  name: 'Homenaje a la Neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: 'Iako je Marta Colvin pretežno poznata po apstraktnim temama koje aludiraju na pre-hispanske simbole, ova ogromna skulptura, koja je omaž neurohirurgiji, jedna je od njenih najpoznatijih remek dela.',
  url: 'https://i.imgur.com/Mx7dA2Y.jpg',
  alt: 'Bronzana statua dve skrštene ruke koje pažljivo drže ljudski mozak vrhovima prstiju.'  
}, {
  name: 'Floralis Genérica',
  artist: 'Eduardo Catalano',
  description: 'Ovaj ogromni (75 ft. ili 23m) srebrni cvet se nalazi u Buenos Ajresu. Dizajniran je da se pomera, zatvarajući svoje latice uveče ili kada duvaju jaki vetrovi, a otvarajući ih ujutru.',
  url: 'https://i.imgur.com/ZF6s192m.jpg',
  alt: 'Velika metalna skulptura cveta sa reflektujućim laticama nalik na ogledalo i snažnim prašnicima.'
}, {
  name: 'Eternal Presence',
  artist: 'John Woodrow Wilson',
  description: 'Wilson je poznat po svojoj preokupiranosti jednakošću, socijalnom pravdom, kao i suštinskim i duhovnim kvalitetima čovečanstva. Ova masivna (7ft. ili 2,13m) bronzana skulptura predstavlja ono što je on objasnio kao "simbolično crno prisustvo prožeto osećajem univerzalne humanosti".',
  url: 'https://i.imgur.com/aTtVpES.jpg',
  alt: 'Skulptura ljudske glave koja deluje sveprisutno i svečano. Zrači mirnoćom i spokojstvom.'
}, {
  name: 'Moai',
  artist: 'Nepoznati umetnik',
  description: 'Nalazi se na Uskršnjem ostrvu, gde postoji oko 1,000 moai očuvanih monumentalnih skulptura, koje su napravili prvobitni Rapa Nui ljudi, za koje se veruje da predstavljaju pretke Bogova.',
  url: 'https://i.imgur.com/RCwLEoQm.jpg',
  alt: 'Tri monumentalne biste sa neproporcionalno velikim glavama i namrštenim licima.'
}, {
  name: 'Blue Nana',
  artist: 'Niki de Saint Phalle',
  description: 'Nane su trijumfalna stvorenja, simboli ženstvenosti i majčinstva. Inicijalno, Saint Phalle je koristila tkaninu i pronađene objekte za Nane, ali je kasnije uvela poliester kako bi napravila živopisniji dojam.',
  url: 'https://i.imgur.com/Sd1AgUOm.jpg',
  alt: 'Velika mozaička skulptura neobične ženske figure u raznobojnom kostimu koji odiše radošću.'
}, {
  name: 'Ultimate Form',
  artist: 'Barbara Hepworth',
  description: 'Ova apstraktna bronzana skulptura je deo The Family of Man serije koja se nalazi u Jorkširskom parku skulptura. Hepworth je odlučila da ne pravi bukvalne reprezentacije sveta, već apstraktne forme inspirisane ljudima i pejzažima.',
  url: 'https://i.imgur.com/2heNQDcm.jpg',
  alt: 'Visoka skulptura tri elementa jedna na drugom koja podseća na ljudsku figuru.'
}, {
  name: 'Cavaliere',
  artist: 'Lamidi Olonade Fakeye',
  description: "Pošto potiče od četiri generacije drvorezbara, Fakeye-ova dela su mešavina tradicionalne i savremene Joruba tematike.",
  url: 'https://i.imgur.com/wIdGuZwm.png',
  alt: 'Složena drvena skulptura ratnika čije lice je fokusirano na konja ukrašenim šarama.'
}, {
  name: 'Big Bellies',
  artist: 'Alina Szapocznikow',
  description: "Alina je poznata po skulpturama izdeljenog tela kao metafore za krhkost i nestalnost mladosti i lepote. Ova skulptura prikazuje dva veoma realistična velika stomaka jednog na drugom, gde je svaki visok oko 5 stopa (1,5m).",
  url: 'https://i.imgur.com/AlHTAdDm.jpg',
  alt: 'Ova skulptura podseća na slapove nabora i dosta se razlikuje od stomaka na klasičnim skulpturama.'
}, {
  name: 'Terracotta Army',
  artist: 'Nepoznati umetnik',
  description: 'Terracotta Army je kolekcija terakota skulptura koje predstavljaju armiju Qin Shi Huang-a, prvog cara Kine. Armija se sastojala od preko 8,000 vojnika, 130 kola sa 520 konja i 150 članova konjice.',
  url: 'https://i.imgur.com/HMFmH6m.jpg',
  alt: '12 terakota skulptura svečanih vojnika, od kojih svaki ima jedinstvenu facijalnu ekspresiju i oklop.'
}, {
  name: 'Lunar Landscape',
  artist: 'Louise Nevelson',
  description: 'Nevelson je poznata po uklanjanju objekata iz ruševina Njujorka, od kojih je kasnije pravila monumentalne konstrukcije. Za ovo je koristila rasparene delove poput noge od kreveta, palice za žongliranje i delove sedišta, zakucavajući ih i lepeći u kutije koje podsećaju na kubističku geometrijsku apstrakciju prostora i forme.',
  url: 'https://i.imgur.com/rN7hY6om.jpg',
  alt: 'Crna mat skulptura gde su pojedinačni delovi na početku bili neprimetni.'
}, {
  name: 'Aureole',
  artist: 'Ranjani Shettar',
  description: 'Shettar spaja tradicionalno i moderno, prirodno i industrijsko. Njena umetnost se fokusira na vezu između čoveka i prirode. Njen rad se opisuje kao ubedljiv i apstraktno i figurativno, prokoseći gravitaciji, kao i "dobra sinteza neobičnih materijala".',
  url: 'https://i.imgur.com/okTpbHhm.jpg',
  alt: 'Bleda skulptura nalik na žicu postavljena na betonski zid spuštajući se na pod. Izgleda lako.'
}, {
  name: 'Hippos',
  artist: 'Taipei Zoo',
  description: 'Zoološki vrt u Tajpeju je naručio trg nilskih konja koji se igraju zaronjeni.',
  url: 'https://i.imgur.com/6o5Vuyu.jpg',
  alt: 'Grupa bronzanih skulptura nilskih konja koji kao da isplivavaju na trotoar.'
}];
```

```css
h2 { margin-top: 10px; margin-bottom: 0; }
h3 {
 margin-top: 5px;
 font-weight: normal;
 font-size: 100%;
}
img { width: 120px; height: 120px; }
button {
  display: block;
  margin-top: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

Dobra je ideja imati više state promenljivih ako im state-ovi nisu povezani, kao `index` i `showMore` u ovom primeru. Ali, ako primetite da često menjate dve state promenljive zajedno, može biti lakše da ih grupišete u jednu. Na primer, ako imate formu sa mnogo polja, zgodnije je imati jednu state promenljivu koja drži ceo objekat, umesto jedne state promenljive po polju. Pročitajte [Odabir state strukture](/learn/choosing-the-state-structure) za više detalja.

<DeepDive>

#### Kako React zna koji state da vrati? {/*how-does-react-know-which-state-to-return*/}

Možda ste već primetili da `useState` poziv ne prima informaciju o tome na *koju* state promenljivu se odnosi. Ne postoji "identifikator" koji se prosleđuje u `useState`, pa kako onda zna koju od state promenljivih da vrati? Da li se oslanja na neku magiju poput parsiranja vaših funkcija? Odgovor je ne.

Umesto toga, da bi omogućili svoju konciznu sintaksu, Hook-ovi **se oslanjaju na stabilan redosled poziva pri svakom renderovanju jedne komponente**. Ovo radi veoma dobro u praksi, jer ako pratite pravilo od gore ("pozivajte Hook-ove samo na početku"), Hook-ovi će uvek biti pozvani u istom redosledu. Dodatno, [linter plugin](https://www.npmjs.com/package/eslint-plugin-react-hooks) hvata većinu grešaka.

Interno, React čuva niz state parova za svaku komponentu. Takođe održava i trenutni indeks para koji je postavljen na `0` pre renderovanja. Svaki put kad pozovete `useState`, React vam daje naredni state par i povećava indeks. Možete pročitati više o ovom mehanizmu u [React Hook-ovi: Nije magija, već samo nizovi](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e).

Ovaj primer **ne koristi React**, ali vam daje ideju kako `useState` zapravo radi:

<Sandpack>

```js src/index.js active
let componentHooks = [];
let currentHookIndex = 0;

// Kako useState radi unutar React-a (pojednostavljeno).
function useState(initialState) {
  let pair = componentHooks[currentHookIndex];
  if (pair) {
    // Ovo nije prvi render,
    // tako da state par već postoji.
    // Vrati ga i pripremi se za naredni Hook poziv.
    currentHookIndex++;
    return pair;
  }

  // Ovo je prvi put da renderujemo,
  // tako da kreiraj state par i sačuvaj ga.
  pair = [initialState, setState];

  function setState(nextState) {
    // Kada korisnik zatraži promenu state-a,
    // postavi novu vrednost u par.
    pair[0] = nextState;
    updateDOM();
  }

  // Sačuvaj par za naredna renderovanja
  // i pripremi se za naredni Hook poziv.
  componentHooks[currentHookIndex] = pair;
  currentHookIndex++;
  return pair;
}

function Gallery() {
  // Svaki useState() poziv će dobiti naredni par.
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  function handleNextClick() {
    setIndex(index + 1);
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  // Ovaj primer ne koristi React, pa
  // vrati objekat umesto JSX-a kao rezultat.
  return {
    onNextClick: handleNextClick,
    onMoreClick: handleMoreClick,
    header: `${sculpture.name} napravio/la ${sculpture.artist}`,
    counter: `${index + 1} od ${sculptureList.length}`,
    more: `${showMore ? 'Sakrij' : 'Prikaži'} detalje`,
    description: showMore ? sculpture.description : null,
    imageSrc: sculpture.url,
    imageAlt: sculpture.alt
  };
}

function updateDOM() {
  // Resetuj trenutni indeks Hook-a
  // pre renderovanja komponente.
  currentHookIndex = 0;
  let output = Gallery();

  // Osveži DOM da se poklapa sa rezultatom.
  // Ovo je deo koji React radi za vas.
  nextButton.onclick = output.onNextClick;
  header.textContent = output.header;
  moreButton.onclick = output.onMoreClick;
  moreButton.textContent = output.more;
  image.src = output.imageSrc;
  image.alt = output.imageAlt;
  if (output.description !== null) {
    description.textContent = output.description;
    description.style.display = '';
  } else {
    description.style.display = 'none';
  }
}

let nextButton = document.getElementById('nextButton');
let header = document.getElementById('header');
let moreButton = document.getElementById('moreButton');
let description = document.getElementById('description');
let image = document.getElementById('image');
let sculptureList = [{
  name: 'Homenaje a la Neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: 'Iako je Marta Colvin pretežno poznata po apstraktnim temama koje aludiraju na pre-hispanske simbole, ova ogromna skulptura, koja je omaž neurohirurgiji, jedna je od njenih najpoznatijih remek dela.',
  url: 'https://i.imgur.com/Mx7dA2Y.jpg',
  alt: 'Bronzana statua dve skrštene ruke koje pažljivo drže ljudski mozak vrhovima prstiju.'  
}, {
  name: 'Floralis Genérica',
  artist: 'Eduardo Catalano',
  description: 'Ovaj ogromni (75 ft. ili 23m) srebrni cvet se nalazi u Buenos Ajresu. Dizajniran je da se pomera, zatvarajući svoje latice uveče ili kada duvaju jaki vetrovi, a otvarajući ih ujutru.',
  url: 'https://i.imgur.com/ZF6s192m.jpg',
  alt: 'Velika metalna skulptura cveta sa reflektujućim laticama nalik na ogledalo i snažnim prašnicima.'
}, {
  name: 'Eternal Presence',
  artist: 'John Woodrow Wilson',
  description: 'Wilson je poznat po svojoj preokupiranosti jednakošću, socijalnom pravdom, kao i suštinskim i duhovnim kvalitetima čovečanstva. Ova masivna (7ft. ili 2,13m) bronzana skulptura predstavlja ono što je on objasnio kao "simbolično crno prisustvo prožeto osećajem univerzalne humanosti".',
  url: 'https://i.imgur.com/aTtVpES.jpg',
  alt: 'Skulptura ljudske glave koja deluje sveprisutno i svečano. Zrači mirnoćom i spokojstvom.'
}, {
  name: 'Moai',
  artist: 'Nepoznati umetnik',
  description: 'Nalazi se na Uskršnjem ostrvu, gde postoji oko 1,000 moai očuvanih monumentalnih skulptura, koje su napravili prvobitni Rapa Nui ljudi, za koje se veruje da predstavljaju pretke Bogova.',
  url: 'https://i.imgur.com/RCwLEoQm.jpg',
  alt: 'Tri monumentalne biste sa neproporcionalno velikim glavama i namrštenim licima.'
}, {
  name: 'Blue Nana',
  artist: 'Niki de Saint Phalle',
  description: 'Nane su trijumfalna stvorenja, simboli ženstvenosti i majčinstva. Inicijalno, Saint Phalle je koristila tkaninu i pronađene objekte za Nane, ali je kasnije uvela poliester kako bi napravila živopisniji dojam.',
  url: 'https://i.imgur.com/Sd1AgUOm.jpg',
  alt: 'Velika mozaička skulptura neobične ženske figure u raznobojnom kostimu koji odiše radošću.'
}, {
  name: 'Ultimate Form',
  artist: 'Barbara Hepworth',
  description: 'Ova apstraktna bronzana skulptura je deo The Family of Man serije koja se nalazi u Jorkširskom parku skulptura. Hepworth je odlučila da ne pravi bukvalne reprezentacije sveta, već apstraktne forme inspirisane ljudima i pejzažima.',
  url: 'https://i.imgur.com/2heNQDcm.jpg',
  alt: 'Visoka skulptura tri elementa jedna na drugom koja podseća na ljudsku figuru.'
}, {
  name: 'Cavaliere',
  artist: 'Lamidi Olonade Fakeye',
  description: "Pošto potiče od četiri generacije drvorezbara, Fakeye-ova dela su mešavina tradicionalne i savremene Joruba tematike.",
  url: 'https://i.imgur.com/wIdGuZwm.png',
  alt: 'Složena drvena skulptura ratnika čije lice je fokusirano na konja ukrašenim šarama.'
}, {
  name: 'Big Bellies',
  artist: 'Alina Szapocznikow',
  description: "Alina je poznata po skulpturama izdeljenog tela kao metafore za krhkost i nestalnost mladosti i lepote. Ova skulptura prikazuje dva veoma realistična velika stomaka jednog na drugom, gde je svaki visok oko 5 stopa (1,5m).",
  url: 'https://i.imgur.com/AlHTAdDm.jpg',
  alt: 'Ova skulptura podseća na slapove nabora i dosta se razlikuje od stomaka na klasičnim skulpturama.'
}, {
  name: 'Terracotta Army',
  artist: 'Nepoznati umetnik',
  description: 'Terracotta Army je kolekcija terakota skulptura koje predstavljaju armiju Qin Shi Huang-a, prvog cara Kine. Armija se sastojala od preko 8,000 vojnika, 130 kola sa 520 konja i 150 članova konjice.',
  url: 'https://i.imgur.com/HMFmH6m.jpg',
  alt: '12 terakota skulptura svečanih vojnika, od kojih svaki ima jedinstvenu facijalnu ekspresiju i oklop.'
}, {
  name: 'Lunar Landscape',
  artist: 'Louise Nevelson',
  description: 'Nevelson je poznata po uklanjanju objekata iz ruševina Njujorka, od kojih je kasnije pravila monumentalne konstrukcije. Za ovo je koristila rasparene delove poput noge od kreveta, palice za žongliranje i delove sedišta, zakucavajući ih i lepeći u kutije koje podsećaju na kubističku geometrijsku apstrakciju prostora i forme.',
  url: 'https://i.imgur.com/rN7hY6om.jpg',
  alt: 'Crna mat skulptura gde su pojedinačni delovi na početku bili neprimetni.'
}, {
  name: 'Aureole',
  artist: 'Ranjani Shettar',
  description: 'Shettar spaja tradicionalno i moderno, prirodno i industrijsko. Njena umetnost se fokusira na vezu između čoveka i prirode. Njen rad se opisuje kao ubedljiv i apstraktno i figurativno, prokoseći gravitaciji, kao i "dobra sinteza neobičnih materijala".',
  url: 'https://i.imgur.com/okTpbHhm.jpg',
  alt: 'Bleda skulptura nalik na žicu postavljena na betonski zid spuštajući se na pod. Izgleda lako.'
}, {
  name: 'Hippos',
  artist: 'Taipei Zoo',
  description: 'Zoološki vrt u Tajpeju je naručio trg nilskih konja koji se igraju zaronjeni.',
  url: 'https://i.imgur.com/6o5Vuyu.jpg',
  alt: 'Grupa bronzanih skulptura nilskih konja koji kao da isplivavaju na trotoar.'
}];

// Učini da UI prikazuje inicijalni state.
updateDOM();
```

```html public/index.html
<button id="nextButton">
  Naredno
</button>
<h3 id="header"></h3>
<button id="moreButton"></button>
<p id="description"></p>
<img id="image">

<style>
* { box-sizing: border-box; }
body { font-family: sans-serif; margin: 20px; padding: 0; }
button { display: block; margin-bottom: 10px; }
</style>
```

```css
button { display: block; margin-bottom: 10px; }
```

</Sandpack>

Ne morate ovo razumeti da bi koristili React, ali vam može koristiti kao mentalni model.

</DeepDive>

## State je izolovan i privatan {/*state-is-isolated-and-private*/}

State je lokalan za instancu komponente na ekranu. Drugim rečima, **ako jednu komponentu renderujete dvaput, svaka će imati potpuno izolovan state**! Promenom jednog nećete uticati na drugi.

U ovom primeru, `Gallery` komponenta od ranije je renderovana dvaput bez promena u njenoj logici. Probajte da kliknete dugmiće u svakoj galeriji. Primetite da su njihovi state-ovi nezavisni:

<Sandpack>

```js
import Gallery from './Gallery.js';

export default function Page() {
  return (
    <div className="Page">
      <Gallery />
      <Gallery />
    </div>
  );
}

```

```js src/Gallery.js
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  function handleNextClick() {
    setIndex(index + 1);
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  return (
    <section>
      <button onClick={handleNextClick}>
        Naredno
      </button>
      <h2>
        <i>{sculpture.name} </i> 
        napravio/la {sculpture.artist}
      </h2>
      <h3>  
        ({index + 1} od {sculptureList.length})
      </h3>
      <button onClick={handleMoreClick}>
        {showMore ? 'Sakrij' : 'Prikaži'} detalje
      </button>
      {showMore && <p>{sculpture.description}</p>}
      <img 
        src={sculpture.url} 
        alt={sculpture.alt}
      />
    </section>
  );
}
```

```js src/data.js
export const sculptureList = [{
  name: 'Homenaje a la Neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: 'Iako je Marta Colvin pretežno poznata po apstraktnim temama koje aludiraju na pre-hispanske simbole, ova ogromna skulptura, koja je omaž neurohirurgiji, jedna je od njenih najpoznatijih remek dela.',
  url: 'https://i.imgur.com/Mx7dA2Y.jpg',
  alt: 'Bronzana statua dve skrštene ruke koje pažljivo drže ljudski mozak vrhovima prstiju.'  
}, {
  name: 'Floralis Genérica',
  artist: 'Eduardo Catalano',
  description: 'Ovaj ogromni (75 ft. ili 23m) srebrni cvet se nalazi u Buenos Ajresu. Dizajniran je da se pomera, zatvarajući svoje latice uveče ili kada duvaju jaki vetrovi, a otvarajući ih ujutru.',
  url: 'https://i.imgur.com/ZF6s192m.jpg',
  alt: 'Velika metalna skulptura cveta sa reflektujućim laticama nalik na ogledalo i snažnim prašnicima.'
}, {
  name: 'Eternal Presence',
  artist: 'John Woodrow Wilson',
  description: 'Wilson je poznat po svojoj preokupiranosti jednakošću, socijalnom pravdom, kao i suštinskim i duhovnim kvalitetima čovečanstva. Ova masivna (7ft. ili 2,13m) bronzana skulptura predstavlja ono što je on objasnio kao "simbolično crno prisustvo prožeto osećajem univerzalne humanosti".',
  url: 'https://i.imgur.com/aTtVpES.jpg',
  alt: 'Skulptura ljudske glave koja deluje sveprisutno i svečano. Zrači mirnoćom i spokojstvom.'
}, {
  name: 'Moai',
  artist: 'Nepoznati umetnik',
  description: 'Nalazi se na Uskršnjem ostrvu, gde postoji oko 1,000 moai očuvanih monumentalnih skulptura, koje su napravili prvobitni Rapa Nui ljudi, za koje se veruje da predstavljaju pretke Bogova.',
  url: 'https://i.imgur.com/RCwLEoQm.jpg',
  alt: 'Tri monumentalne biste sa neproporcionalno velikim glavama i namrštenim licima.'
}, {
  name: 'Blue Nana',
  artist: 'Niki de Saint Phalle',
  description: 'Nane su trijumfalna stvorenja, simboli ženstvenosti i majčinstva. Inicijalno, Saint Phalle je koristila tkaninu i pronađene objekte za Nane, ali je kasnije uvela poliester kako bi napravila živopisniji dojam.',
  url: 'https://i.imgur.com/Sd1AgUOm.jpg',
  alt: 'Velika mozaička skulptura neobične ženske figure u raznobojnom kostimu koji odiše radošću.'
}, {
  name: 'Ultimate Form',
  artist: 'Barbara Hepworth',
  description: 'Ova apstraktna bronzana skulptura je deo The Family of Man serije koja se nalazi u Jorkširskom parku skulptura. Hepworth je odlučila da ne pravi bukvalne reprezentacije sveta, već apstraktne forme inspirisane ljudima i pejzažima.',
  url: 'https://i.imgur.com/2heNQDcm.jpg',
  alt: 'Visoka skulptura tri elementa jedna na drugom koja podseća na ljudsku figuru.'
}, {
  name: 'Cavaliere',
  artist: 'Lamidi Olonade Fakeye',
  description: "Pošto potiče od četiri generacije drvorezbara, Fakeye-ova dela su mešavina tradicionalne i savremene Joruba tematike.",
  url: 'https://i.imgur.com/wIdGuZwm.png',
  alt: 'Složena drvena skulptura ratnika čije lice je fokusirano na konja ukrašenim šarama.'
}, {
  name: 'Big Bellies',
  artist: 'Alina Szapocznikow',
  description: "Alina je poznata po skulpturama izdeljenog tela kao metafore za krhkost i nestalnost mladosti i lepote. Ova skulptura prikazuje dva veoma realistična velika stomaka jednog na drugom, gde je svaki visok oko 5 stopa (1,5m).",
  url: 'https://i.imgur.com/AlHTAdDm.jpg',
  alt: 'Ova skulptura podseća na slapove nabora i dosta se razlikuje od stomaka na klasičnim skulpturama.'
}, {
  name: 'Terracotta Army',
  artist: 'Nepoznati umetnik',
  description: 'Terracotta Army je kolekcija terakota skulptura koje predstavljaju armiju Qin Shi Huang-a, prvog cara Kine. Armija se sastojala od preko 8,000 vojnika, 130 kola sa 520 konja i 150 članova konjice.',
  url: 'https://i.imgur.com/HMFmH6m.jpg',
  alt: '12 terakota skulptura svečanih vojnika, od kojih svaki ima jedinstvenu facijalnu ekspresiju i oklop.'
}, {
  name: 'Lunar Landscape',
  artist: 'Louise Nevelson',
  description: 'Nevelson je poznata po uklanjanju objekata iz ruševina Njujorka, od kojih je kasnije pravila monumentalne konstrukcije. Za ovo je koristila rasparene delove poput noge od kreveta, palice za žongliranje i delove sedišta, zakucavajući ih i lepeći u kutije koje podsećaju na kubističku geometrijsku apstrakciju prostora i forme.',
  url: 'https://i.imgur.com/rN7hY6om.jpg',
  alt: 'Crna mat skulptura gde su pojedinačni delovi na početku bili neprimetni.'
}, {
  name: 'Aureole',
  artist: 'Ranjani Shettar',
  description: 'Shettar spaja tradicionalno i moderno, prirodno i industrijsko. Njena umetnost se fokusira na vezu između čoveka i prirode. Njen rad se opisuje kao ubedljiv i apstraktno i figurativno, prokoseći gravitaciji, kao i "dobra sinteza neobičnih materijala".',
  url: 'https://i.imgur.com/okTpbHhm.jpg',
  alt: 'Bleda skulptura nalik na žicu postavljena na betonski zid spuštajući se na pod. Izgleda lako.'
}, {
  name: 'Hippos',
  artist: 'Taipei Zoo',
  description: 'Zoološki vrt u Tajpeju je naručio trg nilskih konja koji se igraju zaronjeni.',
  url: 'https://i.imgur.com/6o5Vuyu.jpg',
  alt: 'Grupa bronzanih skulptura nilskih konja koji kao da isplivavaju na trotoar.'
}];
```

```css
button { display: block; margin-bottom: 10px; }
.Page > * {
  float: left;
  width: 50%;
  padding: 10px;
}
h2 { margin-top: 10px; margin-bottom: 0; }
h3 {
  margin-top: 5px;
  font-weight: normal;
  font-size: 100%;
}
img { width: 120px; height: 120px; }
button {
  display: block;
  margin-top: 10px;
  margin-bottom: 10px;
}
```

</Sandpack>

Ovo je ono što state čini drugačijim od običnih promenljivih koje biste deklarisali na vrhu modula. State nije vezan za određeni poziv funkcije niti za mesto u kodu, već je "lokalan" za specifično mesto na ekranu. Renderovali ste dve `<Gallery />` komponente, pa im se state čuva odvojeno.

Primetite i da `Page` komponenta ne "zna" ništa o `Gallery` state-u niti da li on uopšte postoji. Za razliku od props-a, **state je potpuno privatan u komponenti koja ga deklariše**. Roditeljska komponenta ga ne može promeniti. Ovo vam omogućava da dodate ili uklonite state iz neke komponente bez uticaja na ostale komponente.

Šta ako poželite da obe galerije sinhronizuju state? Pravilan način da to uradite u React-u je da *uklonite* state iz dečjih komponenata i dodate ga u najbližeg zajedničkog roditelja. Narednih nekoliko stranica će se fokusirati na organizaciju state-a u jednoj komponenti, ali vratićemo se na ovu temu u [Podela state-a među komponentama](/learn/sharing-state-between-components).

<Recap>

* Koristite state promenljivu kada komponenta treba da "upamti" neke informacije između rendera.
* State promenljive su deklarisane pozivanjem `useState` Hook-a.
* Hook-ovi su specijalne funkcije koje počinju sa `use`. Omogućavaju vam da "se zakačite" za React funkcionalnosti poput state-a.
* Hook-ovi mogu podsećati na import-e: moraju biti pozvani bezuslovno. Pozivanje Hook-ova, uključujući i `useState`, dozvoljeno je samo na početku komponenata ili drugog Hook-a.
* `useState` Hook vraća par vrednosti: trenutni state i funkciju koja ga ažurira.
* Možete imati više od jedne state promenljive. Interno, React ih prepoznaje po redosledu.
* State je privatan za komponentu. Ako je renderujete na dva mesta, svaka će imati svoj state.

</Recap>



<Challenges>

#### Kompletirati galeriju {/*complete-the-gallery*/}

Kada pritisnete "Naredno" na poslednjoj skulpturi, kod prestaje da radi. Popravite logiku da biste to sprečili. Možete ovo rešiti dodavanjem dodatne logike u event handler ili onemogućavanjem dugmeta kada akcija nije moguća.

Kada ovo popravite, dodajte dugme "Prethodno" koje prikazuje prethodnu skulpturu. Ne bi trebalo da se skrši na prvoj skulpturi.

<Sandpack>

```js
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  function handleNextClick() {
    setIndex(index + 1);
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleNextClick}>
        Naredno
      </button>
      <h2>
        <i>{sculpture.name} </i> 
        napravio/la {sculpture.artist}
      </h2>
      <h3>  
        ({index + 1} od {sculptureList.length})
      </h3>
      <button onClick={handleMoreClick}>
        {showMore ? 'Sakrij' : 'Prikaži'} detalje
      </button>
      {showMore && <p>{sculpture.description}</p>}
      <img 
        src={sculpture.url} 
        alt={sculpture.alt}
      />
    </>
  );
}
```

```js src/data.js
export const sculptureList = [{
  name: 'Homenaje a la Neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: 'Iako je Marta Colvin pretežno poznata po apstraktnim temama koje aludiraju na pre-hispanske simbole, ova ogromna skulptura, koja je omaž neurohirurgiji, jedna je od njenih najpoznatijih remek dela.',
  url: 'https://i.imgur.com/Mx7dA2Y.jpg',
  alt: 'Bronzana statua dve skrštene ruke koje pažljivo drže ljudski mozak vrhovima prstiju.'  
}, {
  name: 'Floralis Genérica',
  artist: 'Eduardo Catalano',
  description: 'Ovaj ogromni (75 ft. ili 23m) srebrni cvet se nalazi u Buenos Ajresu. Dizajniran je da se pomera, zatvarajući svoje latice uveče ili kada duvaju jaki vetrovi, a otvarajući ih ujutru.',
  url: 'https://i.imgur.com/ZF6s192m.jpg',
  alt: 'Velika metalna skulptura cveta sa reflektujućim laticama nalik na ogledalo i snažnim prašnicima.'
}, {
  name: 'Eternal Presence',
  artist: 'John Woodrow Wilson',
  description: 'Wilson je poznat po svojoj preokupiranosti jednakošću, socijalnom pravdom, kao i suštinskim i duhovnim kvalitetima čovečanstva. Ova masivna (7ft. ili 2,13m) bronzana skulptura predstavlja ono što je on objasnio kao "simbolično crno prisustvo prožeto osećajem univerzalne humanosti".',
  url: 'https://i.imgur.com/aTtVpES.jpg',
  alt: 'Skulptura ljudske glave koja deluje sveprisutno i svečano. Zrači mirnoćom i spokojstvom.'
}, {
  name: 'Moai',
  artist: 'Nepoznati umetnik',
  description: 'Nalazi se na Uskršnjem ostrvu, gde postoji oko 1,000 moai očuvanih monumentalnih skulptura, koje su napravili prvobitni Rapa Nui ljudi, za koje se veruje da predstavljaju pretke Bogova.',
  url: 'https://i.imgur.com/RCwLEoQm.jpg',
  alt: 'Tri monumentalne biste sa neproporcionalno velikim glavama i namrštenim licima.'
}, {
  name: 'Blue Nana',
  artist: 'Niki de Saint Phalle',
  description: 'Nane su trijumfalna stvorenja, simboli ženstvenosti i majčinstva. Inicijalno, Saint Phalle je koristila tkaninu i pronađene objekte za Nane, ali je kasnije uvela poliester kako bi napravila živopisniji dojam.',
  url: 'https://i.imgur.com/Sd1AgUOm.jpg',
  alt: 'Velika mozaička skulptura neobične ženske figure u raznobojnom kostimu koji odiše radošću.'
}, {
  name: 'Ultimate Form',
  artist: 'Barbara Hepworth',
  description: 'Ova apstraktna bronzana skulptura je deo The Family of Man serije koja se nalazi u Jorkširskom parku skulptura. Hepworth je odlučila da ne pravi bukvalne reprezentacije sveta, već apstraktne forme inspirisane ljudima i pejzažima.',
  url: 'https://i.imgur.com/2heNQDcm.jpg',
  alt: 'Visoka skulptura tri elementa jedna na drugom koja podseća na ljudsku figuru.'
}, {
  name: 'Cavaliere',
  artist: 'Lamidi Olonade Fakeye',
  description: "Pošto potiče od četiri generacije drvorezbara, Fakeye-ova dela su mešavina tradicionalne i savremene Joruba tematike.",
  url: 'https://i.imgur.com/wIdGuZwm.png',
  alt: 'Složena drvena skulptura ratnika čije lice je fokusirano na konja ukrašenim šarama.'
}, {
  name: 'Big Bellies',
  artist: 'Alina Szapocznikow',
  description: "Alina je poznata po skulpturama izdeljenog tela kao metafore za krhkost i nestalnost mladosti i lepote. Ova skulptura prikazuje dva veoma realistična velika stomaka jednog na drugom, gde je svaki visok oko 5 stopa (1,5m).",
  url: 'https://i.imgur.com/AlHTAdDm.jpg',
  alt: 'Ova skulptura podseća na slapove nabora i dosta se razlikuje od stomaka na klasičnim skulpturama.'
}, {
  name: 'Terracotta Army',
  artist: 'Nepoznati umetnik',
  description: 'Terracotta Army je kolekcija terakota skulptura koje predstavljaju armiju Qin Shi Huang-a, prvog cara Kine. Armija se sastojala od preko 8,000 vojnika, 130 kola sa 520 konja i 150 članova konjice.',
  url: 'https://i.imgur.com/HMFmH6m.jpg',
  alt: '12 terakota skulptura svečanih vojnika, od kojih svaki ima jedinstvenu facijalnu ekspresiju i oklop.'
}, {
  name: 'Lunar Landscape',
  artist: 'Louise Nevelson',
  description: 'Nevelson je poznata po uklanjanju objekata iz ruševina Njujorka, od kojih je kasnije pravila monumentalne konstrukcije. Za ovo je koristila rasparene delove poput noge od kreveta, palice za žongliranje i delove sedišta, zakucavajući ih i lepeći u kutije koje podsećaju na kubističku geometrijsku apstrakciju prostora i forme.',
  url: 'https://i.imgur.com/rN7hY6om.jpg',
  alt: 'Crna mat skulptura gde su pojedinačni delovi na početku bili neprimetni.'
}, {
  name: 'Aureole',
  artist: 'Ranjani Shettar',
  description: 'Shettar spaja tradicionalno i moderno, prirodno i industrijsko. Njena umetnost se fokusira na vezu između čoveka i prirode. Njen rad se opisuje kao ubedljiv i apstraktno i figurativno, prokoseći gravitaciji, kao i "dobra sinteza neobičnih materijala".',
  url: 'https://i.imgur.com/okTpbHhm.jpg',
  alt: 'Bleda skulptura nalik na žicu postavljena na betonski zid spuštajući se na pod. Izgleda lako.'
}, {
  name: 'Hippos',
  artist: 'Taipei Zoo',
  description: 'Zoološki vrt u Tajpeju je naručio trg nilskih konja koji se igraju zaronjeni.',
  url: 'https://i.imgur.com/6o5Vuyu.jpg',
  alt: 'Grupa bronzanih skulptura nilskih konja koji kao da isplivavaju na trotoar.'
}];
```

```css
button { display: block; margin-bottom: 10px; }
.Page > * {
  float: left;
  width: 50%;
  padding: 10px;
}
h2 { margin-top: 10px; margin-bottom: 0; }
h3 {
  margin-top: 5px;
  font-weight: normal;
  font-size: 100%;
}
img { width: 120px; height: 120px; }
```

</Sandpack>

<Solution>

Ovo dodaje zaštitni uslov unutar oba event handler-a i onemogućava dugmiće kad je potrebno:

<Sandpack>

```js
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  let hasPrev = index > 0;
  let hasNext = index < sculptureList.length - 1;

  function handlePrevClick() {
    if (hasPrev) {
      setIndex(index - 1);
    }
  }

  function handleNextClick() {
    if (hasNext) {
      setIndex(index + 1);
    }
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button
        onClick={handlePrevClick}
        disabled={!hasPrev}
      >
        Prethodno
      </button>
      <button
        onClick={handleNextClick}
        disabled={!hasNext}
      >
        Naredno
      </button>
      <h2>
        <i>{sculpture.name} </i> 
        napravio/la {sculpture.artist}
      </h2>
      <h3>  
        ({index + 1} od {sculptureList.length})
      </h3>
      <button onClick={handleMoreClick}>
        {showMore ? 'Sakrij' : 'Prikaži'} detalje
      </button>
      {showMore && <p>{sculpture.description}</p>}
      <img 
        src={sculpture.url} 
        alt={sculpture.alt}
      />
    </>
  );
}
```

```js src/data.js hidden
export const sculptureList = [{
  name: 'Homenaje a la Neurocirugía',
  artist: 'Marta Colvin Andrade',
  description: 'Iako je Marta Colvin pretežno poznata po apstraktnim temama koje aludiraju na pre-hispanske simbole, ova ogromna skulptura, koja je omaž neurohirurgiji, jedna je od njenih najpoznatijih remek dela.',
  url: 'https://i.imgur.com/Mx7dA2Y.jpg',
  alt: 'Bronzana statua dve skrštene ruke koje pažljivo drže ljudski mozak vrhovima prstiju.'  
}, {
  name: 'Floralis Genérica',
  artist: 'Eduardo Catalano',
  description: 'Ovaj ogromni (75 ft. ili 23m) srebrni cvet se nalazi u Buenos Ajresu. Dizajniran je da se pomera, zatvarajući svoje latice uveče ili kada duvaju jaki vetrovi, a otvarajući ih ujutru.',
  url: 'https://i.imgur.com/ZF6s192m.jpg',
  alt: 'Velika metalna skulptura cveta sa reflektujućim laticama nalik na ogledalo i snažnim prašnicima.'
}, {
  name: 'Eternal Presence',
  artist: 'John Woodrow Wilson',
  description: 'Wilson je poznat po svojoj preokupiranosti jednakošću, socijalnom pravdom, kao i suštinskim i duhovnim kvalitetima čovečanstva. Ova masivna (7ft. ili 2,13m) bronzana skulptura predstavlja ono što je on objasnio kao "simbolično crno prisustvo prožeto osećajem univerzalne humanosti".',
  url: 'https://i.imgur.com/aTtVpES.jpg',
  alt: 'Skulptura ljudske glave koja deluje sveprisutno i svečano. Zrači mirnoćom i spokojstvom.'
}, {
  name: 'Moai',
  artist: 'Nepoznati umetnik',
  description: 'Nalazi se na Uskršnjem ostrvu, gde postoji oko 1,000 moai očuvanih monumentalnih skulptura, koje su napravili prvobitni Rapa Nui ljudi, za koje se veruje da predstavljaju pretke Bogova.',
  url: 'https://i.imgur.com/RCwLEoQm.jpg',
  alt: 'Tri monumentalne biste sa neproporcionalno velikim glavama i namrštenim licima.'
}, {
  name: 'Blue Nana',
  artist: 'Niki de Saint Phalle',
  description: 'Nane su trijumfalna stvorenja, simboli ženstvenosti i majčinstva. Inicijalno, Saint Phalle je koristila tkaninu i pronađene objekte za Nane, ali je kasnije uvela poliester kako bi napravila živopisniji dojam.',
  url: 'https://i.imgur.com/Sd1AgUOm.jpg',
  alt: 'Velika mozaička skulptura neobične ženske figure u raznobojnom kostimu koji odiše radošću.'
}, {
  name: 'Ultimate Form',
  artist: 'Barbara Hepworth',
  description: 'Ova apstraktna bronzana skulptura je deo The Family of Man serije koja se nalazi u Jorkširskom parku skulptura. Hepworth je odlučila da ne pravi bukvalne reprezentacije sveta, već apstraktne forme inspirisane ljudima i pejzažima.',
  url: 'https://i.imgur.com/2heNQDcm.jpg',
  alt: 'Visoka skulptura tri elementa jedna na drugom koja podseća na ljudsku figuru.'
}, {
  name: 'Cavaliere',
  artist: 'Lamidi Olonade Fakeye',
  description: "Pošto potiče od četiri generacije drvorezbara, Fakeye-ova dela su mešavina tradicionalne i savremene Joruba tematike.",
  url: 'https://i.imgur.com/wIdGuZwm.png',
  alt: 'Složena drvena skulptura ratnika čije lice je fokusirano na konja ukrašenim šarama.'
}, {
  name: 'Big Bellies',
  artist: 'Alina Szapocznikow',
  description: "Alina je poznata po skulpturama izdeljenog tela kao metafore za krhkost i nestalnost mladosti i lepote. Ova skulptura prikazuje dva veoma realistična velika stomaka jednog na drugom, gde je svaki visok oko 5 stopa (1,5m).",
  url: 'https://i.imgur.com/AlHTAdDm.jpg',
  alt: 'Ova skulptura podseća na slapove nabora i dosta se razlikuje od stomaka na klasičnim skulpturama.'
}, {
  name: 'Terracotta Army',
  artist: 'Nepoznati umetnik',
  description: 'Terracotta Army je kolekcija terakota skulptura koje predstavljaju armiju Qin Shi Huang-a, prvog cara Kine. Armija se sastojala od preko 8,000 vojnika, 130 kola sa 520 konja i 150 članova konjice.',
  url: 'https://i.imgur.com/HMFmH6m.jpg',
  alt: '12 terakota skulptura svečanih vojnika, od kojih svaki ima jedinstvenu facijalnu ekspresiju i oklop.'
}, {
  name: 'Lunar Landscape',
  artist: 'Louise Nevelson',
  description: 'Nevelson je poznata po uklanjanju objekata iz ruševina Njujorka, od kojih je kasnije pravila monumentalne konstrukcije. Za ovo je koristila rasparene delove poput noge od kreveta, palice za žongliranje i delove sedišta, zakucavajući ih i lepeći u kutije koje podsećaju na kubističku geometrijsku apstrakciju prostora i forme.',
  url: 'https://i.imgur.com/rN7hY6om.jpg',
  alt: 'Crna mat skulptura gde su pojedinačni delovi na početku bili neprimetni.'
}, {
  name: 'Aureole',
  artist: 'Ranjani Shettar',
  description: 'Shettar spaja tradicionalno i moderno, prirodno i industrijsko. Njena umetnost se fokusira na vezu između čoveka i prirode. Njen rad se opisuje kao ubedljiv i apstraktno i figurativno, prokoseći gravitaciji, kao i "dobra sinteza neobičnih materijala".',
  url: 'https://i.imgur.com/okTpbHhm.jpg',
  alt: 'Bleda skulptura nalik na žicu postavljena na betonski zid spuštajući se na pod. Izgleda lako.'
}, {
  name: 'Hippos',
  artist: 'Taipei Zoo',
  description: 'Zoološki vrt u Tajpeju je naručio trg nilskih konja koji se igraju zaronjeni.',
  url: 'https://i.imgur.com/6o5Vuyu.jpg',
  alt: 'Grupa bronzanih skulptura nilskih konja koji kao da isplivavaju na trotoar.'
}];
```

```css
button { display: block; margin-bottom: 10px; }
.Page > * {
  float: left;
  width: 50%;
  padding: 10px;
}
h2 { margin-top: 10px; margin-bottom: 0; }
h3 {
  margin-top: 5px;
  font-weight: normal;
  font-size: 100%;
}
img { width: 120px; height: 120px; }
```

</Sandpack>

Primetite da su `hasPrev` i `hasNext` korišćeni na *dva* mesta, u povratnom JSX-u i unutar event handler-a! Ovaj zgodni šablon radi zato što event handler funkcije ["zatvaraju"](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures) sve promenljive deklarisane tokom renderovanja.

</Solution>

#### Popraviti zaglavljene input-e forme {/*fix-stuck-form-inputs*/}

Kada kucate u input polja, ništa se ne prikazuje. Kao da su input vrednosti "zaglavljene" sa praznim stringovima. `value` prvog `<input>`-a je setovan da uvek odgovara `firstName` promenljivoj, a `value` drugog `<input>`-a je setovan da uvek odgovara `lastName` promenljivoj. Ovo je tačno. Oba input-a imaju `onChange` event handler-e koji pokušavaju da ažuriraju promenljive na osnovu poslednje vrednosti koju je korisnik uneo (`e.target.value`). Međutim, deluje da promenljive ne "pamte" svoje vrednosti između rendera. Popravite ovo korišćenjem state promenljivih.

<Sandpack>

```js
export default function Form() {
  let firstName = '';
  let lastName = '';

  function handleFirstNameChange(e) {
    firstName = e.target.value;
  }

  function handleLastNameChange(e) {
    lastName = e.target.value;
  }

  function handleReset() {
    firstName = '';
    lastName = '';
  }

  return (
    <form onSubmit={e => e.preventDefault()}>
      <input
        placeholder="Ime"
        value={firstName}
        onChange={handleFirstNameChange}
      />
      <input
        placeholder="Prezime"
        value={lastName}
        onChange={handleLastNameChange}
      />
      <h1>Ćao, {firstName} {lastName}</h1>
      <button onClick={handleReset}>Resetuj</button>
    </form>
  );
}
```

```css 
h1 { margin-top: 10px; }
```

</Sandpack>

<Solution>

Prvo, import-ujte `useState` iz React-a. Onda zamenite `firstName` i `lastName` sa state promenljivama deklarisanim pozivanjem `useState`. Na kraju, zamenite svaku `firstName = ...` dodelu sa `setFirstName(...)`, a onda uradite isto i za `lastName`. Ne zaboravite da promenite i `handleReset` kako bi dugme za resetovanje radilo.

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }

  function handleReset() {
    setFirstName('');
    setLastName('');
  }

  return (
    <form onSubmit={e => e.preventDefault()}>
      <input
        placeholder="Ime"
        value={firstName}
        onChange={handleFirstNameChange}
      />
      <input
        placeholder="Prezime"
        value={lastName}
        onChange={handleLastNameChange}
      />
      <h1>Ćao, {firstName} {lastName}</h1>
      <button onClick={handleReset}>Resetuj</button>
    </form>
  );
}
```

```css 
h1 { margin-top: 10px; }
```

</Sandpack>

</Solution>

#### Popraviti problem {/*fix-a-crash*/}

Ovde je mala forma koja treba služiti korisnicima da unesu feedback. Kada se feedback submit-uje, trebalo bi da se prikaže poruka zahvalnosti. Međutim, skrši se sa porukom o grešci koja kaže "Renderovano manje hook-ova nego što je očekivano" ("Rendered fewer hooks than expected"). Možete li uočiti grešku i popraviti je?

<Hint>

Postoje li ograničenja _gde_ se Hook-ovi mogu pozvati? Da li komponenta krši neka pravila? Proverite da li postoje neki komentari koji gase linter provere--to je mesto gde se bug-ovi često kriju!

</Hint>

<Sandpack>

```js
import { useState } from 'react';

export default function FeedbackForm() {
  const [isSent, setIsSent] = useState(false);
  if (isSent) {
    return <h1>Hvala!</h1>;
  } else {
    // eslint-disable-next-line
    const [message, setMessage] = useState('');
    return (
      <form onSubmit={e => {
        e.preventDefault();
        alert(`Slanje: "${message}"`);
        setIsSent(true);
      }}>
        <textarea
          placeholder="Poruka"
          value={message}
          onChange={e => setMessage(e.target.value)}
        />
        <br />
        <button type="submit">Pošalji</button>
      </form>
    );
  }
}
```

</Sandpack>

<Solution>

Hook-ovi mogu biti pozvani samo na početku funkcije komponente. Ovde, prva `isSent` definicija poštuje to pravilo, ali `message` definicija je ugnježdena u uslov.

Pomerite je izvan uslova da biste popravili problem:

<Sandpack>

```js
import { useState } from 'react';

export default function FeedbackForm() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState('');

  if (isSent) {
    return <h1>Hvala!</h1>;
  } else {
    return (
      <form onSubmit={e => {
        e.preventDefault();
        alert(`Slanje: "${message}"`);
        setIsSent(true);
      }}>
        <textarea
          placeholder="Poruka"
          value={message}
          onChange={e => setMessage(e.target.value)}
        />
        <br />
        <button type="submit">Pošalji</button>
      </form>
    );
  }
}
```

</Sandpack>

Upamtite, Hook-ovi moraju biti pozvani bezuslovno i uvek u istom redosledu!

Takođe možete ukloniti nepotrebnu `else` granu da sprečite ugnježdavanje. Međutim, i dalje je bitno da se svi Hook-ovi pozovu *pre* prvog `return`-a.

<Sandpack>

```js
import { useState } from 'react';

export default function FeedbackForm() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState('');

  if (isSent) {
    return <h1>Hvala!</h1>;
  }

  return (
    <form onSubmit={e => {
      e.preventDefault();
      alert(`Slanje: "${message}"`);
      setIsSent(true);
    }}>
      <textarea
        placeholder="Poruka"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <br />
      <button type="submit">Pošalji</button>
    </form>
  );
}
```

</Sandpack>

Pokušajte da pomerite drugi `useState` poziv nakon `if` uslova i vidite kako ponovo neće raditi.

Ako je vaš linter [konfigurisan za React](/learn/editor-setup#linting), trebali biste videti lint grešku kad uradite ovako nešto. Ako ne vidite grešku kada napišete neispravan kod lokalno, potrebno je da postavite linting za vaš projekat. 

</Solution>

#### Ukloniti nepotreban state {/*remove-unnecessary-state*/}

Kada se klikne dugme, ovaj primer treba pitati korisnika da unese ime, a onda da prikaže alert koji ga pozdravlja. Pokušali ste da koristite state da čuvate ime, ali iz nekog razloga prvi put se prikazuje "Zdravo, !", a svaki sledeći put se prikazuje "Zdravo, [ime]!" sa prethodnim input-om.

Da biste popravili kod, uklonite nepotrebnu state promenljivu. (Diskutovaćemo zašto [ovo nije radilo](/learn/state-as-a-snapshot) kasnije.)

Možete li objasniti zašto state promenljiva nije potrebna?

<Sandpack>

```js
import { useState } from 'react';

export default function FeedbackForm() {
  const [name, setName] = useState('');

  function handleClick() {
    setName(prompt('Koje je vaše ime?'));
    alert(`Zdravo, ${name}!`);
  }

  return (
    <button onClick={handleClick}>
      Pozdravi
    </button>
  );
}
```

</Sandpack>

<Solution>

Evo popravljene verzije koja koristi običnu `name` promenljivu deklarisanu u funkciji gde je potrebna:

<Sandpack>

```js
export default function FeedbackForm() {
  function handleClick() {
    const name = prompt('Koje je vaše ime?');
    alert(`Zdravo, ${name}!`);
  }

  return (
    <button onClick={handleClick}>
      Pozdravi
    </button>
  );
}
```

</Sandpack>

State promenljiva je neophodna samo za čuvanje informacija između ponovnih rendera u komponenti. Unutar jednog event handler-a, obična promenljiva radi posao. Nemojte koristiti state promenljive kada i obične promenljive rade.

</Solution>

</Challenges>
