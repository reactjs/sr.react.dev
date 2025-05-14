---
title: Kreiranje React aplikacije
---

<Intro>

Ako želite da napravite novu aplikaciju ili novi sajt koristeći React, preporučujemo da počnete sa framework-om.

</Intro>

Ako vaša aplikacija ima ograničenja koja nisu dobro rešena u postojećim framework-ovima, ako želite da napravite sopstveni framework, ili samo želite naučiti osnove React aplikacija, možete [napraviti React aplikaciju od nule](/learn/build-a-react-app-from-scratch).

## Full-stack framework-ovi {/*full-stack-frameworks*/}

Ovi preporučeni framework-ovi podržavaju sve funkcionalnosti koje su vam potrebne za deploy i skaliranje aplikacija u produkciji. Integrisali su najnovije React funkcionalnosti i iskoristili prednosti React-ove arhitekture.

<Note>

#### Full-stack framework-ovima nije potreban server. {/*react-frameworks-do-not-require-a-server*/}

Svi framework-ovi na ovoj stranici podržavaju renderovanje na klijentskoj strani ([CSR](https://developer.mozilla.org/en-US/docs/Glossary/CSR)), single-page aplikacije ([SPA](https://developer.mozilla.org/en-US/docs/Glossary/SPA)) i generisanje statičkih sajtova ([SSG](https://developer.mozilla.org/en-US/docs/Glossary/SSG)). Ove aplikacije se mogu deploy-ovati na [CDN](https://developer.mozilla.org/en-US/docs/Glossary/CDN) ili na statički hosting servis bez servera. Dodatno, ovi framework-ovi dozvoljavaju dodavanje renderovanja na serverskoj strani na nivou rute, ako vam je to zgodno za vašu aplikaciju.

Ovo vam omogućava da počnete samo sa klijentskom aplikacijom, pa, ako vam kasnije zatreba, možete se odlučiti za upotrebu servera samo na određenim rutama bez ponovnog pisanja aplikacije. Pogledajte dokumentaciju framework-a za konfigurisanje strategije renderovanja.

</Note>

### Next.js (App Router) {/*nextjs-app-router*/}

**[Next.js-ov App Router](https://nextjs.org/docs) je React framework koji u potpunosti koristi prednosti React-ove arhitekture kako bi omogućio full-stack React aplikacije.**

<TerminalBlock>
npx create-next-app@latest
</TerminalBlock>

Next.js je održavan od strane [Vercel-a](https://vercel.com/). Možete [deploy-ovati Next.js aplikaciju](https://nextjs.org/docs/app/building-your-application/deploying) na bilo koji hosting provider koji podržava Node.js ili Docker kontejnere, ili na vaš sopstveni server. Next.js takođe podržava [static export](https://nextjs.org/docs/app/building-your-application/deploying/static-exports) koji ne zahteva server.

### React Router (v7) {/*react-router-v7*/}

**[React Router](https://reactrouter.com/start/framework/installation) je najpopularnija biblioteka za rutiranje u React-u i može se upariti sa Vite-om za pravljenje full-stack React framework-a.** On naglašava standardne Web API-je i ima nekoliko [deploy template-a spremnih za upotrebu](https://github.com/remix-run/react-router-templates) za različite JavaScript runtime-ove i platforme.

Da biste kreirali novi React Router framework projekat, pokrenite:

<TerminalBlock>
npx create-react-router@latest
</TerminalBlock>

[Shopify](https://www.shopify.com) održava React Router.

### Expo (za native aplikacije) {/*expo*/}

**[Expo](https://expo.dev/) je React framework koji vam omogućava da kreirate univerzalne Android, iOS i web aplikacije sa zaista native korisničkim interfejsima.** On pruža SDK za [React Native](https://reactnative.dev/) koji olakšava korišćenje native delova. Da biste kreirali novi Expo projekat, pokrenite:

<TerminalBlock>
npx create-expo-app@latest
</TerminalBlock>

Ako vam je Expo nepoznat, pogledajte [Expo tutorijal](https://docs.expo.dev/tutorial/introduction/).

Expo je održavan od strane [Expo (kompanije)](https://expo.dev/about). Kreiranje aplikacija sa Expo-om je besplatno i možete ih submit-ovati na Google i Apple app store bez ograničenja. Expo dodatno pruža i opcione cloud servise koji se plaćaju.


## Ostali framework-ovi {/*other-frameworks*/}

Postoji nekoliko nadolazećih framework-ova koji rade na našoj full stack React viziji:

- [TanStack Start (Beta)](https://tanstack.com/): TanStack Start je full-stack React framework nastao zbog TanStack Router-a. Pruža renderovanje celih dokumenata na serverskoj strani, strimovanje, serverske funkcije, bundle-ovanje i još mnogo toga koristeći alate poput Nitro-a i Vite-a.
- [RedwoodJS](https://redwoodjs.com/): Redwood je full stack React framework sa mnogo unapred instaliranih paketa i konfiguracija koje olakšavaju pravljenje full-stack web aplikacija.

<DeepDive>

#### Koje funkcionalnosti čine viziju full-stack arhitekture React-ovog tima? {/*which-features-make-up-the-react-teams-full-stack-architecture-vision*/}

Next.js-ov App Router bundler potpuno implementira zvaničnu [React Server Components specifikaciju](https://github.com/reactjs/rfcs/blob/main/text/0188-server-components.md). Ovo vam omogućava da pomešate komponente koje se izvršavaju tokom vremena izgradnje, komponente koje se izvršavaju samo na serveru i interaktivne komponente u jednom React stablu.

Na primer, možete napisati server-only React komponentu kao `async` funkciju koja čita iz baze podataka ili iz fajla. Zatim, vašim interaktivnim komponentama možete proslediti podatke iz nje:

```js
// Ova komponenta se izvršava *samo* na serveru (ili tokom vremena izgradnje).
async function Talks({ confId }) {
  // 1. Vi ste na serveru, tako da možete da komunicirate sa vašim podacima. API pristupna tačka nije potrebna.
  const talks = await db.Talks.findAll({ confId });

  // 2. Dodajte bilo koju količinu logike renderovanja. To neće učiniti vaš JavaScript bundle većim.
  const videos = talks.map(talk => talk.video);

  // 3. Prosledite podatke komponentama koje će se izvršavati u pretraživaču.
  return <SearchableVideoList videos={videos} />;
}
```

Next.js-ov App Router je takođe integrisan sa [fetch-ovanjem podataka pomoću Suspense-a](/blog/2022/03/29/react-v18#suspense-in-data-frameworks). Ovo vam omogućava da specificirate stanje učitavanja (kao što je skeleton placeholder) za različite delove vašeg korisničkog interfejsa direktno u vašem React stablu:

```js
<Suspense fallback={<TalksLoading />}>
  <Talks confId={conf.id} />
</Suspense>
```

Server Components i Suspense su funkcionalnosti React-a, a ne Next.js-a. Međutim, njihovo usvajanje na nivou framework-a zahteva podršku i netrivijalan rad na implementaciji. Trenutno, Next.js App Router je najkompletnija implementacija. React tim sarađuje sa developerima bundler-a kako bi ove funkcionalnosti bile lakše za implementaciju u sledećoj generaciji framework-ova.

</DeepDive>

## Početak od nule {/*start-from-scratch*/}

Ako vaša aplikacija ima ograničenja koja nisu dobro rešena u postojećim framework-ovima, ako želite da napravite sopstveni framework, ili samo želite naučiti osnove React aplikacija, postoji još opcija za započinjanje React projekta od nule.

Početak od nule vam pruža više fleksibilnosti, ali zahteva da napravite odluku koje alate ćete koristiti za rutiranje, fetch-ovanje podataka i ostale često korišćene šablone. To je kao da pravite sopstveni framework, a ne da koristite već postojeći. [Framework-ovi koje preporučujemo](#full-stack-frameworks) imaju ugrađena rešenja za ovakve probleme.

<<<<<<< HEAD
Ako želite napraviti vaša rešenja, pogledajte uputstvo kako [napraviti React aplikaciju od nule](/learn/build-a-react-app-from-scratch) za instrukcije oko postavki React projekta počevši od postojećih alata kao što su [Vite](https://vite.dev/), [Parcel](https://parceljs.org/) ili [RSbuild](https://rsbuild.dev/).
=======
If you want to build your own solutions, see our guide to [build a React app from Scratch](/learn/build-a-react-app-from-scratch) for instructions on how to set up a new React project starting with a build tool like [Vite](https://vite.dev/), [Parcel](https://parceljs.org/), or [RSbuild](https://rsbuild.dev/).
>>>>>>> a3e9466dfeea700696211533a3570bc48d7bc3d3

-----

_Ako ste vi autor framework-a i zainteresovani ste da ga uključimo u ovu stranicu, [molimo vas da nam javite](https://github.com/reactjs/react.dev/issues/new?assignees=&labels=type%3A+framework&projects=&template=3-framework.yml&title=%5BFramework%5D%3A+)._
