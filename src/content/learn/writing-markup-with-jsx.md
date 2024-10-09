---
title: Pisanje markup-a sa JSX-om
---

<Intro>

*JSX* je sintaksna ekstenzija za JavaScript koja vam omogućava da pišete markup sličan HTML-u unutar JavaScript fajla. Iako postoje i drugi načini za pisanje komponenata, većina React developera preferira sažetost JSX-a, pa se zato koristi na većini projekata.

</Intro>

<YouWillLearn>

* Zašto React kombinuje markup i logiku renderovanja
* Kako se JSX razlikuje od HTML-a
* Kako da prikažete informacije pomoću JSX-a

</YouWillLearn>

## JSX: Stavljanje markup-a unutar JavaScript-a {/*jsx-putting-markup-into-javascript*/}

Web je izgrađen pomoću HTML-a, CSS-a i JavaScript-a. Godinama su web developeri čuvali sadržaj u HTML-u, dizajn u CSS-u, a logiku u JavaScript-u, i to često u odvojenim fajlovima! Sadržaj se nalazio unutar HTML-a, dok je logika živela odvojeno u JavaScript-u:

<DiagramGroup>

<Diagram name="writing_jsx_html" height={237} width={325} alt="HTML markup with purple background and a div with two child tags: p and form. ">

HTML

</Diagram>

<Diagram name="writing_jsx_js" height={237} width={325} alt="Three JavaScript handlers with yellow background: onSubmit, onLogin, and onClick.">

JavaScript

</Diagram>

</DiagramGroup>

Kako je web postajao interaktivniji, logika je značajno određivala sadržaj. JavaScript je bio zadužen za HTML! Zato, **u React-u, logika renderovanja i markup žive zajedno na jednom mestu—u komponentama**.

<DiagramGroup>

<Diagram name="writing_jsx_sidebar" height={330} width={325} alt="React component with HTML and JavaScript from previous examples mixed. Function name is Sidebar which calls the function isLoggedIn, highlighted in yellow. Nested inside the function highlighted in purple is the p tag from before, and a Form tag referencing the component shown in the next diagram.">

`Sidebar.js` React komponenta

</Diagram>

<Diagram name="writing_jsx_form" height={330} width={325} alt="React component with HTML and JavaScript from previous examples mixed. Function name is Form containing two handlers onClick and onSubmit highlighted in yellow. Following the handlers is HTML highlighted in purple. The HTML contains a form element with a nested input element, each with an onClick prop.">

`Form.js` React komponenta

</Diagram>

</DiagramGroup>

Držanje logike renderovanja i markup-a za dugme na jednom mestu osigurava da će oni ostati sinhronizovani nakon svake promene. Nasuprot toga, detalji koji nisu povezani, kao što su markup-i za dugme i sidebar, izolovani su jedan od drugog, što omogućava lakšu promenu svakog od njih.

Svaka React komponenta je JavaScript funkcija koja može sadržati markup koji će React renderovati u pretraživaču. React komponente koriste sintaksnu ekstenziju pod imenom JSX da bi predstavile taj markup. JSX veoma liči na HTML, ali je malo strožiji i može prikazati dinamičke podatke. Najbolji način da ovo razumete je da konvertujete neki HTML markup u JSX markup.

<Note>

JSX i React su dve odvojene stvari. Često se koriste zajedno, ali *možete* [ih koristiti odvojeno](https://reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html#whats-a-jsx-transform). JSX je sintaksna ekstenzija, dok je React JavaScript biblioteka.

</Note>

## Konvertovanje HTML-a u JSX {/*converting-html-to-jsx*/}

Pretpostavimo da imate neki (potpuno validan) HTML:

```html
<h1>Hedy Lamarr-ina Todo lista</h1>
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
```

I želite da ga pomerite u vašu komponentu:

```js
export default function TodoList() {
  return (
    // ???
  )
}
```

Ako uradite samo copy/paste, to neće raditi:


<Sandpack>

```js
export default function TodoList() {
  return (
    // Ovo baš i ne radi!
    <h1>Hedy Lamarr-ina Todo lista</h1>
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
img { height: 90px }
```

</Sandpack>

To je zato što je JSX strožiji i ima par pravila više od HTML-a! Ako pročitate poruke o greškama iznad, uputiće vas kako da popravite markup. Takođe, možete pratiti i uputstvo ispod.

<Note>

U većini slučajeva, React-ove poruke o greškama će vam pomoći da pronađete u čemu je problem. Pročitajte ih ako se zaglavite!

</Note>

## Pravila JSX-a {/*the-rules-of-jsx*/}

### 1. Vratite jedan root element {/*1-return-a-single-root-element*/}

Da biste vratili više elemenata iz komponente, **zamotajte ih u jedan roditeljski tag**.

Na primer, možete koristiti `<div>`:

```js {1,11}
<div>
  <h1>Hedy Lamarr-ina Todo lista</h1>
  <img 
    src="https://i.imgur.com/yXOvdOSs.jpg" 
    alt="Hedy Lamarr" 
    class="photo"
  >
  <ul>
    ...
  </ul>
</div>
```


Ako ne želite da dodate novi `<div>` u vaš markup, možete umesto toga koristiti `<>` i `</>`:

```js {1,11}
<>
  <h1>Hedy Lamarr-ina Todo lista</h1>
  <img 
    src="https://i.imgur.com/yXOvdOSs.jpg" 
    alt="Hedy Lamarr" 
    class="photo"
  >
  <ul>
    ...
  </ul>
</>
```

Ovaj prazan tag se naziva *[Fragment](/reference/react/Fragment)*. Fragmenti vam omogućavaju da grupišete stvari bez uticaja na HTML stablo u pretraživaču.

<DeepDive>

#### Zašto morate zamotati više JSX tag-ova? {/*why-do-multiple-jsx-tags-need-to-be-wrapped*/}

JSX liči na HTML, ali je ispod haube transformisan u obične JavaScript objekte. Ne možete vratiti dva objekta iz funkcije ako ih ne zamotate u niz. Ovo takođe objašnjava i zašto ne možete vratiti dva JSX tag-a bez da ih zamotate u drugi tag ili Fragment.

</DeepDive>

### 2. Zatvorite sve tag-ove {/*2-close-all-the-tags*/}

JSX zahteva da svi tag-ovi budu eksplicitno zatvoreni: samozatvarajući tag-ovi kao što je `<img>` moraju postati `<img />`, a obmotavajući tag-ovi poput `<li>pomorandže` moraju biti napisani kao `<li>pomorandže</li>`.

Ovako izgledaju Hedy Lamarr-ina slika i lista zatvoreni:

```js {2-6,8-10}
<>
  <img 
    src="https://i.imgur.com/yXOvdOSs.jpg" 
    alt="Hedy Lamarr" 
    class="photo"
   />
  <ul>
    <li>Izmisli nove semafore</li>
    <li>Vežbaj scenu iz filma</li>
    <li>Unapredi tehnologiju spektra</li>
  </ul>
</>
```

### 3. Koristite camelCase za <s>sve</s> većinu stvari! {/*3-camelcase-salls-most-of-the-things*/}

JSX se pretvara u JavaScript i atributi napisani u JSX-u postaju ključevi za JavaScript objekte. U vašim komponentama, često ćete želeti da pročitate te atribute u promenljive. Ali JavaScript ima ograničenja za imena promenljivih. Na primer, imena ne mogu sadržati crtice ili rezervisane reči kao što je `class`.

Zato se, u React-u, većina HTML i SVG atributa pišu u camelCase-u. Na primer, umesto `stroke-width` koristićete `strokeWidth`. Pošto je `class` rezervisana reč, u React-u ćete pisati `className`, koji je nastao na osnovu [odgovarajućeg polja u DOM-u](https://developer.mozilla.org/en-US/docs/Web/API/Element/className):

```js {4}
<img 
  src="https://i.imgur.com/yXOvdOSs.jpg" 
  alt="Hedy Lamarr" 
  className="photo"
/>
```

Možete [pronaći sve te atribute u listi props-a DOM komponente](/reference/react-dom/components/common). Ako neki i pogrešite, ne brinite—React će napisati poruku sa mogućom ispravkom u [konzoli pretraživača](https://developer.mozilla.org/docs/Tools/Browser_Console).

<Pitfall>

Iz istorijskih razloga, [`aria-*`](https://developer.mozilla.org/docs/Web/Accessibility/ARIA) i [`data-*`](https://developer.mozilla.org/docs/Learn/HTML/Howto/Use_data_attributes) atributi se pišu sa crticama kao i u HTML-u.

</Pitfall>

### Pro-savet: Koristite JSX konverter {/*pro-tip-use-a-jsx-converter*/}

Konvertovanje svih tih atributa u postojeći markup može biti naporno! Preporučujemo vam da koristite [konverter](https://transform.tools/html-to-jsx) za pretvaranje postojećeg HTML-a i SVG-a u JSX. Konverteri su veoma zgodni u praksi, ali je i dalje važno da razumete šta se događa kako biste lagodno mogli da pišete JSX.

Ovo je konačni rezultat:

<Sandpack>

```js
export default function TodoList() {
  return (
    <>
      <h1>Hedy Lamarr-ina Todo lista</h1>
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
img { height: 90px }
```

</Sandpack>

<Recap>

Sada znate zašto JSX postoji i kako da ga koristite u komponentama:

* React komponente grupišu logiku renderovanja i markup zato što su povezani.
* JSX je sličan HTML-u, ali ima par razlika. Ako vam je potrebno, možete koristiti [konverter](https://transform.tools/html-to-jsx).
* Poruke o greškama će vam često ukazati na pravac u kom možete popraviti vaš markup.

</Recap>



<Challenges>

#### Konvertovati HTML u JSX {/*convert-some-html-to-jsx*/}

Ovaj HTML je samo ubačen u komponentu, ali nije validan JSX. Popravite ga:

<Sandpack>

```js
export default function Bio() {
  return (
    <div class="intro">
      <h1>Dobro došli na moj sajt!</h1>
    </div>
    <p class="summary">
      Ovde možete pronaći moje ideje.
      <br><br>
      <b>I <i>slike</b></i> naučnika!
    </p>
  );
}
```

```css
.intro {
  background-image: linear-gradient(to left, violet, indigo, blue, green, yellow, orange, red);
  background-clip: text;
  color: transparent;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.summary {
  padding: 20px;
  border: 10px solid gold;
}
```

</Sandpack>

Možete to uraditi samostalno ili pomoću konvertera. Na vama je!

<Solution>

<Sandpack>

```js
export default function Bio() {
  return (
    <div>
      <div className="intro">
        <h1>Dobro došli na moj sajt!</h1>
      </div>
      <p className="summary">
        Ovde možete pronaći moje ideje.
        <br /><br />
        <b>I <i>slike</i></b> naučnika!
      </p>
    </div>
  );
}
```

```css
.intro {
  background-image: linear-gradient(to left, violet, indigo, blue, green, yellow, orange, red);
  background-clip: text;
  color: transparent;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.summary {
  padding: 20px;
  border: 10px solid gold;
}
```

</Sandpack>

</Solution>

</Challenges>
