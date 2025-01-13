---
title: Dodajte React na postojeći projekat
---

<Intro>

Ako želite da dodate neku interaktivnost na postojeći projekat, ne morate ga ponovo pisati u React-u. Dodajte React u postojeći stek i renderujte interaktivne React komponente bilo gde.

</Intro>

<Note>

**Treba da instalirate [Node.js](https://nodejs.org/en/) za lokalni razvoj.** Iako možete [probati React](/learn/installation#try-react) online ili sa jednostavnom HTML stranicom, ali u realnosti većina JavaScript alata koje želite da koristite za razvoj zahteva Node.js.

</Note>

## Koristite React za ceo subroute vašeg postojećeg sajta {/*using-react-for-an-entire-subroute-of-your-existing-website*/}

Recimo da imate postojeći veb sajt na `example.com` napravljen sa nekom drugom tehnologijom (kao što je Rails) i želite da implementirate sve rute koje počinju sa `example.com/some-app/` u potpunosti sa React-om.

Evo kako preporučujemo da to uradite:

<<<<<<< HEAD
1. **Napravite React deo vaše aplikacije** koristeći jedan od [React-based framework-a](/learn/start-a-new-react-project).
2. **Specifikujte `/some-app` kao *base path*** u konfiguraciji vašeg framework-a (evo kako: [Next.js](https://nextjs.org/docs/api-reference/next.config.js/basepath), [Gatsby](https://www.gatsbyjs.com/docs/how-to/previews-deploys-hosting/path-prefix/)).
3. **Konfigurišite vaš server ili proxy** tako da sve rute ispod `/some-app/` budu obrađene od strane vaše React aplikacije.
=======
1. **Build the React part of your app** using one of the [React-based frameworks](/learn/start-a-new-react-project).
2. **Specify `/some-app` as the *base path*** in your framework's configuration (here's how: [Next.js](https://nextjs.org/docs/app/api-reference/config/next-config-js/basePath), [Gatsby](https://www.gatsbyjs.com/docs/how-to/previews-deploys-hosting/path-prefix/)).
3. **Configure your server or a proxy** so that all requests under `/some-app/` are handled by your React app.
>>>>>>> 9000e6e003854846c4ce5027703b5ce6f81aad80

Ovo će omogućiti React delu vaše aplikacije da [koristi najbolje prakse](/learn/start-a-new-react-project#can-i-use-react-without-a-framework) koje su ugrađene u te framework-e.

Mnogi React-based framework-ovi su full-stack i omogućavaju vašoj React aplikaciji da iskoristi server. Međutim, možete koristiti isti pristup čak i ako ne možete ili ne želite da pokrećete JavaScript na serveru. U tom slučaju, servirajte HTML/CSS/JS export ([`next export` output](https://nextjs.org/docs/advanced-features/static-html-export) za Next.js, default za Gatsby) na `/some-app/` umesto toga.

## Koristite React za deo vaše postojeće stranice {/*using-react-for-a-part-of-your-existing-page*/}

Recimo da imate postojeću stranicu napravljenu sa nekom drugom tehnologijom (ili na serveru kao Rails, ili na klijentu kao Backbone), i želite da renderujete interaktivne React komponente negde na toj stranici. To je uobičajen način da se integriše React - zapravo, to je kako je većina React koda izgledala na Meta-i mnogo godina!

Ovo možete uraditi u dva koraka:

1. **Postavite JavaScript okruženje** koje vam omogućava da koristite [JSX sintaksu](/learn/writing-markup-with-jsx), podelite svoj kod u module sa [`import`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) / [`export`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export) sintaksom i koristite pakete (na primer, React) iz [npm](https://www.npmjs.com/) registra paketa.
2. **Renderujte vaše React komponente** gde želite da ih vidite na stranici.

Tačan pristup zavisi od vašeg postojećeg podešavanja stranice, pa hajde da prođemo kroz neke detalje.

### Korak 1: Postavite modularno JavaScript okruženje {/*step-1-set-up-a-modular-javascript-environment*/}

Modularno JavaScript okruženje vam omogućava da pišete vaše React komponente u pojedinačnim fajlovima, umesto da pišete sav vaš kod u jednom fajlu. Takođe vam omogućava da koristite sve divne pakete koje su objavili drugi programeri na [npm](https://www.npmjs.com/) registru - uključujući i React! Kako ćete to uraditi zavisi od vašeg postojećeg podešavanja:

* **Ako je vaša stranica već podeljena u fajlove koji koriste `import` naredbe,** pokušajte da koristite to podešavanje. Proverite da li pisanje `<div />` u vašem JS kodu izaziva sintaksnu grešku. Ako izaziva sintaksnu grešku, možda ćete morati da [transformišete vaš JS kod sa Babel-om](https://babeljs.io/setup) i omogućite [Babel React preset](https://babeljs.io/docs/babel-preset-react) da biste koristili JSX.

* **Ako vaša stranica nema postojeće podešavanje za kompajliranje JavaScript modula,** postavite ga sa [Vite-om](https://vitejs.dev/). Vite zajednica održava [mnoge integracije sa backend framework-ovima](<https://github.com/vitejs/awesome-vite#integrations-with-backends>), ukjučujući Rails, Django i Laravel. Ako vaš backend framework nije na listi, [pratite ovaj vodič](https://vitejs.dev/guide/backend-integration.html) da biste ručno integrisali Vite build-ove sa vašim backend-om.

Da proverite da li vaše podešavanje radi, pokrenite ovu komandu u folderu vašeg projekta:

<TerminalBlock>
npm install react react-dom
</TerminalBlock>

Onda dodajte ove linije koda na vrh vašeg glavnog JavaScript fajla (možda se zove `index.js` ili `main.js`):

<Sandpack>

```html index.html hidden
<!DOCTYPE html>
<html>
  <head><title>Moja Aplikacija</title></head>
  <body>
    <!-- Your existing page content (in this example, it gets replaced) -->
  </body>
</html>
```

```js src/index.js active
import { createRoot } from 'react-dom/client';

// Brisanje postojećeg HTML sadržaja
document.body.innerHTML = '<div id="app"></div>';

// Renderovanje vaše React komponente umesto toga
const root = createRoot(document.getElementById('app'));
root.render(<h1>Zdravo, svete!</h1>);
```

</Sandpack>

Ako je celi sadržaj vaše stranice zamenjen sa "Zdravo, svete!", sve je uspelo! Nastavite sa čitanjem.

<Note>

Integracija modularnog JavaScript okruženja u postojeći projekat može da izgleda zastrašujuće, ali vredi! Ako zapnete, probajte naš [community resources](/community) ili [Vite Chat](https://chat.vitejs.dev/).

</Note>

### Korak 2: Renderujte React komponente bilo gde na stranici {/*step-2-render-react-components-anywhere-on-the-page*/}

U prethodnom koraku, dodali ste ovaj kod na vrh vašeg glavnog fajla:


```js
import { createRoot } from 'react-dom/client';

// Brisanje postojećeg HTML sadržaja
document.body.innerHTML = '<div id="app"></div>';

// Renderovanje vaše React komponente umesto toga
const root = createRoot(document.getElementById('app'));
root.render(<h1>Zdravo, svete!</h1>);
```

Naravno da ne želite da obrišete postojeći HTML sadržaj!

Obrišite ovaj kod.

Umesto toga, verovatno želite da renderujete vaše React komponente na specifičnim mestima u vašem HTML-u. Otvorite vaš HTML fajl (ili server template koji ga generiše) i dodajte jedinstveni [`id`](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/id) atribut bilo kom tag-u, na primer:

```html

<!-- ... negde u vašem HTML-u ... -->
<nav id="navigation"></nav>
<!-- ... ostatak HTML-a ... -->
```

Ovo vam omogućava da pronađete taj HTML element sa [`document.getElementById`](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementById) i prosledite ga [`createRoot`](/reference/react-dom/client/createRoot) tako da možete da renderujete vašu React komponentu unutra:

<Sandpack>

```html index.html
<!DOCTYPE html>
<html>
  <head><title>Moja Aplikacija</title></head>
  <body>
    <p>Ovaj paragraf je deo HTML-a.</p>
    <nav id="navigation"></nav>
    <p>Ovaj paragraf je takođe deo HTML-a.</p>
  </body>
</html>
```

```js src/index.js active
import { createRoot } from 'react-dom/client';

function NavigationBar() {
  // TODO: Zapravo implementirajte NavigationBar
  return <h1>Pozdrav od React-a!</h1>;
}

const domNode = document.getElementById('navigation');
const root = createRoot(domNode);
root.render(<NavigationBar />);
```

</Sandpack>

Primećujemo kako je originalni HTML sadržaj iz `index.html` sačuvan, ali vaša `NavigationBar` React komponenta sada se pojavljuje unutar `<nav id="navigation">` iz vašeg HTML-a. Pročitajte [createRoot dokumentaciju](/reference/react-dom/client/createRoot#rendering-a-page-partially-built-with-react) da biste saznali više o renderovanju React komponenti unutar postojeće HTML stranice.

Kada usvojite React u postojeći projekat, uobičajeno je da počnete sa malim interaktivnim komponentama (kao što su dugmad), i onda postepeno "idete na viši nivo" dok na kraju vaša cela stranica nije napravljena sa React-om. Ako ikada dođete do tog nivoa, preporučujemo da odmah nakon toga pređete na [React framework](/learn/start-a-new-react-project) da biste dobili najviše od React-a.

## Koristite React Native u postojećoj mobilnoj aplikaciji {/*using-react-native-in-an-existing-mobile-app*/}

[React Native](https://reactnative.dev/) može da se integriše u postojeće mobilne aplikacije postepeno. Ako imate postojeću mobilnu aplikaciju za Android (Java ili Kotlin) ili iOS (Objective-C ili Swift), [pratite ovaj vodič](https://reactnative.dev/docs/integration-with-existing-apps) da biste dodali React Native ekran u nju.

