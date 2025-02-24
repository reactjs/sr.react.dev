---
title: Render i Commit
---

<Intro>

Pre nego što se vaše komponente prikažu na ekranu, React mora da ih renderuje. Razumevanje koraka u ovom procesu pomoći će vam da razmislite o tome kako se vaš kod izvršava i da objasnite njegovo ponašanje.

</Intro>

<YouWillLearn>

* Šta u React-u znači renderovanje
* Kada i zašto React renderuje komponentu
* Korake potrebne za prikaz komponente na ekranu
* Zašto renderovanje ne ažurira DOM uvek

</YouWillLearn>

Zamislite da su vaše komponente kuvari u kuhinji, koji pripremaju ukusna jela od sastojaka. U ovom scenariju, React je konobar koji prenosi porudžbine od gostiju i donosi im njihova jela. Ovaj proces poručivanja i posluživanja UI-a ima tri koraka:

1. **Pokretanje** rendera (prenos porudžbine gosta u kuhinju)
2. **Renderovanje** komponente (priprema porudžbine u kuhinji)
3. **Commit-ovanje** u DOM (postavljanje porudžbine na sto)

<IllustrationBlock sequential>
  <Illustration caption="Pokretanje" alt="React kao konobar u restoranu, prenosi porudžbine od korisnika i dostavlja ih u Kitchen komponentu." src="/images/docs/illustrations/i_render-and-commit1.png" />
  <Illustration caption="Render" alt="Šef kuhinje za Card daje React-u svežu Card komponentu." src="/images/docs/illustrations/i_render-and-commit2.png" />
  <Illustration caption="Commit" alt="React donosi Card korisniku za njegov sto." src="/images/docs/illustrations/i_render-and-commit3.png" />
</IllustrationBlock>

## Korak 1: Pokrenuti render {/*step-1-trigger-a-render*/}

Postoje dva razloga da se komponenta renderuje:

1. **Inicijalni render** komponente.
2. **State se promenio**, bilo od komponente ili nekog od njenih roditelja.

### Inicijalni render {/*initial-render*/}

Kad se aplikacija pokrene, morate započeti inicijalni render. Framework-ovi i sandbox-ovi često skrivaju ovaj kod, ali se on pokreće pozivanjem [`createRoot`](/reference/react-dom/client/createRoot) nad željenim DOM čvorom, a nakon toga i pozivanjem `render` metode sa vašom komponentom:

<Sandpack>

```js src/index.js active
import Image from './Image.js';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'))
root.render(<Image />);
```

```js src/Image.js
export default function Image() {
  return (
    <img
      src="https://i.imgur.com/ZF6s192.jpg"
      alt="'Floralis Genérica' napravio Eduardo Catalano: ogromna metalik skulptura cveta sa reflektujućim laticama"
    />
  );
}
```

</Sandpack>

Zakomentarišite `root.render()` poziv i vidite kako će komponenta nestati!

### Ponovni renderi kad se state ažurira {/*re-renders-when-state-updates*/}

Kada je komponenta inicijalno renderovana, možete pokrenuti naredne rendere ažuriranjem state-a pomoću [`set` funkcije](/reference/react/useState#setstate). Ažuriranjem state-a vaše komponente automatski stavljate render u red čekanja. (Ovo možete zamisliti kao da gost restorana poručuje čaj, dezert i ostale stvari nakon prve porudžbine, u zavisnosti od state-a žeđi i gladi.)

<IllustrationBlock sequential>
  <Illustration caption="Ažuriranje state-a..." alt="React kao konobar u restoranu, servira Card UI korisniku, koji je predstavljen kao čovek sa kursorom na glavi. Korisnik izražava da želi roze karticu, a ne crnu!" src="/images/docs/illustrations/i_rerender1.png" />
  <Illustration caption="...pokreće..." alt="React se vraća u Kitchen komponentu i kaže šefu kuhinje za Card da mu treba roze Card." src="/images/docs/illustrations/i_rerender2.png" />
  <Illustration caption="...render!" alt="Šef kuhinje za Card daje React-u roze Card." src="/images/docs/illustrations/i_rerender3.png" />
</IllustrationBlock>

## Korak 2: React renderuje vaše komponente {/*step-2-react-renders-your-components*/}

Kada pokrenete render, React poziva vaše komponente da shvati šta da prikaže na ekranu. **"Renderovanje" je zapravo React-ovo pozivanje vaših komponenata.**

* **Pri inicijalnom renderu**, React će pozvati root komponentu.
* **Za naredne rendere**, React će pozvati funkciju komponente čije ažuriranje state-a je pokrenulo render.

Ovaj proces je rekurzivan: ako ažurirana komponenta vrati drugu komponentu, React će renderovati _tu drugu_ komponentu, a ako ta komponenta takođe nešto vrati, renderovaće _to nešto_ sledeće, i tako dalje. Proces će se nastaviti dok god postoje ugnježdene komponente i React zna tačno šta treba biti prikazano na ekranu.

U narednom primeru, React će pozvati `Gallery()` i `Image()` nekoliko puta:

<Sandpack>

```js src/Gallery.js active
export default function Gallery() {
  return (
    <section>
      <h1>Inspirativne skulpture</h1>
      <Image />
      <Image />
      <Image />
    </section>
  );
}

function Image() {
  return (
    <img
      src="https://i.imgur.com/ZF6s192.jpg"
      alt="'Floralis Genérica' napravio Eduardo Catalano: ogromna metalik skulptura cveta sa reflektujućim laticama"
    />
  );
}
```

```js src/index.js
import Gallery from './Gallery.js';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'))
root.render(<Gallery />);
```

```css
img { margin: 0 10px 10px 0; }
```

</Sandpack>

* **Tokom inicijalnog rendera** React će [kreirati DOM čvorove](https://developer.mozilla.org/docs/Web/API/Document/createElement) za `<section>`, `<h1>` i tri `<img>` tag-a. 
* **Tokom ponovnog rendera** React će izračunati koja od njihovih polja (ili nijedno) su se promenila od prethodnog rendera. Neće uraditi ništa sa tom informacijom do narednog koraka, commit faze.

<Pitfall>

Renderovanje mora uvek biti [čist proračun](/learn/keeping-components-pure):

* **Isti input-i, isti rezultat.** Dobijanjem istih input-a, komponenta treba uvek da vrati isti JSX. (Kada neko poruči salatu sa paradajzom, ne bi trebao da dobije salatu sa lukom!)
* **Gleda samo svoja posla.** Ne bi trebalo da menja nikakve objekte ili promenljive koji su postojali pre renderovanja. (Jedna porudžbina ne bi trebala da menja bilo koju drugu porudžbinu.)

U suprotnom, naići ćete na zbunjujuće bug-ove i nepredvidivo ponašanje dok se kompleksnost na vašem projektu povećava. U toku razvoja sa "Strict Mode", React poziva funkciju svake komponente dvaput, što može pomoći da se uoče greške prouzrokovane nečistim funkcijama.

</Pitfall>

<DeepDive>

#### Optimizacija performansi {/*optimizing-performance*/}

Default ponašanje renderovanja svih komponenti ugnježdenih u ažuriranu komponentu nije optimalno za performanse ako je ažurirana komponenta dosta visoko u stablu. Ako naiđete na problem sa performansama, postoji nekoliko opcija da to rešite opisanih u sekciji [Performanse](https://reactjs.org/docs/optimizing-performance.html). **Nemojte optimizovati prerano!**

</DeepDive>

## Korak 3: React commit-uje promene na DOM {/*step-3-react-commits-changes-to-the-dom*/}

Nakon renderovanja (pozivanja) vaših komponenata, React će izmeniti DOM.

* **Za inicijalni render**, React će koristiti [`appendChild()`](https://developer.mozilla.org/docs/Web/API/Node/appendChild) DOM API da postavi sve DOM čvorove koje je kreirao na ekran.
* **Za ponovne rendere**, React će primeniti minimum neophodnih operacija (izračunatih tokom renderovanja!) da bi učinio da se DOM poklapa sa najnovijim rezultatom renderovanja.

**React menja DOM čvorove samo ako postoji razlika između rendera.** Na primer, ovde je komponenta koja se ponovo renderuje sa drugačijim props-ima koje dobija od roditelja svake sekunde. Primetite da možete dodati tekst u `<input>`, menjajući `value`, i da taj tekst neće nestati kad se komponenta ponovo renderuje:

<Sandpack>

```js src/Clock.js active
export default function Clock({ time }) {
  return (
    <>
      <h1>{time}</h1>
      <input />
    </>
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
    <Clock time={time.toLocaleTimeString()} />
  );
}
```

</Sandpack>

Ovo radi jer tokom poslednjeg koraka, React jedino ažurira sadržaj `<h1>` sa novim `time`. On vidi da se `<input>` pojavljuje u JSX-u na istom mestu kao prethodni put, pa React ne dira `<input>`—niti njegov `value`!
## Epilog: Browser slikarstvo {/*epilogue-browser-paint*/}

Nakon što se renderovanje završi i React ažurira DOM, browser će ponovo oslikati ekran. Iako je ovaj proces poznat kao "browser renderovanje", mi ćemo ga osloviti sa "slikarstvo" da bi izbegli zabunu kroz dokumentaciju.

<Illustration alt="Browser slika 'mrtva priroda sa elementom kartice'." src="/images/docs/illustrations/i_browser-paint.png" />

<Recap>

* Svako ažuriranje ekrana u React aplikaciji se dešava u tri faze:
  1. Pokretanje
  2. Render
  3. Commit
* Možete koristiti Strict Mode da pronađete greške u komponentama
* React ne dira DOM ako je rezultat renderovanja isti kao prethodni put

</Recap>

