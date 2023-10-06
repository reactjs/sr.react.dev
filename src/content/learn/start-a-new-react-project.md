---
title: Započnite novi React projekat
---

<Intro>

Ako želite da napravite novu aplikaciju ili novi sajt koristeći samo React, preporučujemo da izaberete jedan od React framework-ova popularnih u zajednici. Framework-ovi pružaju funkcionalnosti koje većina aplikacija i sajtova na kraju treba da imaju, uključujući rutiranje,fetch podataka i generisanje HTML-a.

</Intro>

<Note>

**Morate da instalirate [Node.js](https://nodejs.org/en/) za lokalni development.** Možete "takođe" da izaberete da koristite Node.js u produkciji, ali ne morate. Mnogi React framework-ovi podržavaju eksport u statički HTML/CSS/JS folder.

</Note>

## React framework-ovi za produkciju {/*production-grade-react-frameworks*/}

### Next.js {/*nextjs*/}

**[Next.js](https://nextjs.org/) je full-stack React framework.** On je veoma svestran i omogućava vam da kreirate React aplikacije bilo koje veličine--od uglavnom statičkog bloga do kompleksne dinamičke aplikacije. Da biste kreirali novi Next.js projekat, pokrenite u vašem terminalu:

<TerminalBlock>
npx create-next-app@latest
</TerminalBlock>

Ako vam je Next.js nepoynat, proverite ovaj [Next.js tutorijal.](https://nextjs.org/learn/foundations/about-nextjs)

Next.js je održavan od strane [Vercel](https://vercel.com/). Možete [deploy-ovati Next.js aplikaciju](https://nextjs.org/docs/deployment) na bilo koji Node.js ili serverless hosting, ili na vaš sopstveni server. Next.js takođe podržava [static export](https://nextjs.org/docs/advanced-features/static-html-export) koji ne zahteva server.

### Remix {/*remix*/}

**[Remix](https://remix.run/) je full-stack React framework sa ugneždenim rutiranjem.** On vam omogućava da podelite vašu aplikaciju na ugneždene delove koji mogu da učitavaju podatke paralelno i da se osvežavaju u odgovoru na korisničke akcije. Da biste kreirali novi Remix projekat, pokrenite u vašem terminalu:

<TerminalBlock>
npx create-remix
</TerminalBlock>

Ako vam je Remix nepoznat, pogledajte Remix [blog tutorijal](https://remix.run/docs/en/main/tutorials/blog) (kratak) i [app tutorijal](https://remix.run/docs/en/main/tutorials/jokes) (dugačak).

Remix je održavan od strane [Shopify](https://www.shopify.com/). Kada kreirate Remix projekat, morate [izabrati vaš deployment target](https://remix.run/docs/en/main/guides/deployment). Možete deploy-ovati Remix aplikaciju na bilo koji Node.js ili serverless hosting korišćenjem ili pisanjem [adaptera](https://remix.run/docs/en/main/other-api/adapter).

### Gatsby {/*gatsby*/}

**[Gatsby](https://www.gatsbyjs.com/) je React framework za brze CMS sajtove.** Njegov bogat ekosistem plugina i GraphQL data sloj pojednostavljuju integraciju sadržaja, API-ja i servisa u jedan sajt. Da biste kreirali novi Gatsby projekat, pokrenite u vašem terminalu:

<TerminalBlock>
npx create-gatsby
</TerminalBlock>

Ako niste upoznati sa Gatsby, pogledajte [Gatsby tutorijal.](https://www.gatsbyjs.com/docs/tutorial/)

Gatsby je održavan od strane [Netlify](https://www.netlify.com/). Možete [deploy-ovati potpuno statički Gatsby sajt](https://www.gatsbyjs.com/docs/how-to/previews-deploys-hosting) na bilo koji statički hosting. Ako se odlučite za korišćenje server-only funkcionalnosti, pobrinite se da vaš hosting provider podržava Gatsby.

### Expo (za native aplikacije) {/*expo*/}


**[Expo](https://expo.dev/) je React framework koji vam omogućava da kreirate univerzalne Android, iOS i web aplikacije sa zaista native korisničkim interfejsima.** On pruža SDK za [React Native](https://reactnative.dev/) koji olakšava korišćenje native delova. Da biste kreirali novi Expo projekat, pokrenite u vašem terminalu:

<TerminalBlock>
npx create-expo-app
</TerminalBlock>

Ako niste upoznati sa Expo, pogledajte [Expo tutorijal.](https://docs.expo.dev/tutorial/introduction/)

Expo je održavan od strane [Expo (kompanije)](https://expo.dev/about). Kreiranje aplikacija sa Expo-om je besplatno, i možete ih submit-ovati na Google i Apple app store bez ograničenja. Expo dodatno pruža opt-in plaćene cloud servise.

<DeepDive>

#### Mogu li da koristim React bez framework-a? {/*can-i-use-react-without-a-framework*/}

Naravno da možete koristiti React bez framework-a--kako biste [koristili React za deo vaše stranice.](/learn/add-react-to-an-existing-project#using-react-for-a-part-of-your-existing-page) **Međutim, ako kreirate novu aplikaciju ili sajt koristeći samo React, preporučujemo da koristite framework.**

Evo i zašto.

Čak i ako vam ne treba rutiranje ili fetch podataka na početku, verovatno ćete želeti da dodate neke biblioteke za njih. Kako vaš JavaScript bundle raste sa svakom novom funkcionalnošću, možda ćete morati da podelite kod za svaku rutu pojedinačno. Kako vaše potrebe za fetch podataka postaju složenije, verovatno ćete naići na server-client mrežne vodopade koji čine da vaša aplikacija deluje veoma sporo. Kako vaša publika uključuje više korisnika sa lošim mrežnim uslovima i uređajima niske klase, možda ćete morati da generišete HTML iz vaših komponenti da biste prikazali sadržaj ranije--ili na serveru, ili tokom vremena izgradnje. Menjanje vašeg setup-a da bi se pokrenuo neki od vašeg koda na serveru ili tokom vremena izgradnje može biti veoma komplikovano.

**Ovi problemi nisu specifični za React. Zato Svelte ima SvelteKit, Vue ima Nuxt, i tako dalje.** Da biste rešili ove probleme sami, moraćete da integrišete vaš bundler sa vašim router-om i sa vašom bibliotekom za fetch podataka. Nije teško napraviti početni setup, ali postoji mnogo suptilnosti u pravljenju aplikacije koja se brzo učitava čak i dok raste tokom vremena. Želećete da pošaljete minimalnu količinu koda aplikacije ali da to uradite u jednom client-server roundtrip-u, paralelno sa bilo kojim podacima potrebnim za stranicu. Verovatno ćete želeti da stranica bude interaktivna pre nego što se vaš JavaScript kod pokrene, da biste podržali progresivno poboljšanje. Možda ćete želeti da generišete folder potpuno statičkih HTML fajlova za vaše marketing stranice koje mogu biti hostovane bilo gde i da i dalje rade sa isključenim JavaScript-om. Izgradnja ovih mogućnosti sami zahteva pravi posao.

**React framework-ovi na ovoj stranici rešavaju probleme kao što su ovi automatski, bez dodatnog posla sa vaše strane.** Oni vam omogućavaju da počnete veoma jednostavno i da onda skalirate vašu aplikaciju sa vašim potrebama. Svaki React framework ima zajednicu, tako da je lakše naći odgovore na pitanja i nadograditi alate. Framework-ovi takođe daju strukturu vašem kodu, pomažući vam i drugima da zadržite kontekst i veštine između različitih projekata. Nasuprot tome, sa custom setup-om je lakše zaglaviti se na nepodržanim verzijama zavisnosti, i na kraju ćete zapravo kreirati vaš sopstveni framework--iako bez zajednice ili putanju za nadogradnju (i ako je išta kao oni koje smo pravili u prošlosti, više haotično dizajnirani).

Ako i dalje niste ubedjeni, ili vaša aplikacija ima neobične zahteve koji nisu dobro podržani od strane ovih framework-ova i želite da napravite vaš sopstveni custom setup, ne možemo vas zaustaviti--krenite! Uzmite `react` i `react-dom` sa npm-a, napravite vaš custom build proces sa bundler-om kao što je [Vite](https://vitejs.dev/) ili [Parcel](https://parceljs.org/), i dodajte druge alate kada vam budu potrebni za rutiranje, statičku generaciju ili server-side rendering, i tako dalje.
</DeepDive>

## Najnoviji React framework-ovi {/*bleeding-edge-react-frameworks*/}

kako smo i dalje istraživali kako da nastavimo da poboljšavamo React, shvatili smo da je integracija React-a bliže sa framework-ovima (specifično, sa rutiranjem, bundler-ima, i server tehnologijama) naša najveća prilika da pomognemo React korisnicima da kreiraju bolje aplikacije. Next.js tim se složio da sarađuje sa nama u istraživanju, razvoju, integraciji, i testiranju framework-agnostičnih bleeding-edge React mogućnosti kao što su [React Server Components.](/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components)

Ove mogućnosti su sve bliže da budu spremne za produkciju svakog dana, i bili smo u razgovorima sa drugim bundler i framework developerima o njihovoj integraciji. Naša nada je da će za godinu ili dve, svi framework-ovi navedeni na ovoj stranici imati punu podršku za ove mogućnosti. (Ako ste autor framework-a zainteresovan za saradnju sa nama u eksperimentisanju sa ovim mogućnostima, molimo vas javite nam se!)

### Next.js (App Router) {/*nextjs-app-router*/}

**[Next.js-ov App Router](https://nextjs.org/docs) je redizajn Next.js API-ja koji ima za cilj da ispuni React-ovu full-stack arhitekturu viziju.** On vam omogućava da fetch-ujete podatke u asinhronim komponentama koje se izvršavaju na serveru ili čak tokom vremena izgradnje.

Next.js je održavan od strane [Vercel](https://vercel.com/). Možete [deploy-ovati Next.js aplikaciju](https://nextjs.org/docs/deployment) na bilo koji Node.js ili serverless hosting, ili na vaš sopstveni server. Next.js takođe podržava [static export](https://nextjs.org/docs/advanced-features/static-html-export) koji ne zahteva server.

<DeepDive>

#### Koje mogućnosti čine full-stack arhitekturu React-ovog tima? {/*which-features-make-up-the-react-teams-full-stack-architecture-vision*/}

Next.js-ov App Router bundler potpuno implementira zvaničnu [React Server Components specifikaciju](<https://github.com/reactjs/rfcs/blob/main/text/0188-server-components.md>). Ovo vam omogućava da pomešate komponente koje se izvršavaju tokom vremena izgradnje, komponente koje se izvršavaju samo na serveru, i interaktivne komponente u jednom React stablu.

Na primer, možete napisati server-only React komponentu kao `async` funkciju koja čita iz baze podataka ili iz fajla. Zatim možete proslediti podatke iz nje vašim interaktivnim komponentama:

```js
// Ova komponenta se izvršava *samo* na serveru (ili tokom vremena izgradnje).
async function Talks({ confId }) {

  // 1. Vi ste na serveru, tako da možete da komunicirate sa vašom bazom podataka. API pristupna tačka nije potreban.
  const talks = await db.Talks.findAll({ confId });

  // 2. Dodajte bilo koju količinu rendering logike. To neće učiniti vaš JavaScript bundle većim.
  const videos = talks.map(talk => talk.video);

  // 3. Prosledite podatke komponentama koje će se izvršavati u pretraživaču.
  return <SearchableVideoList videos={videos} />;
}
```

Router Next.js aplikacije je takođe integrisan sa [fetch podataka pomoću Suspense-a](/blog/2022/03/29/react-v18#suspense-in-data-frameworks). Ovo vam omogućava da specificirate stanje učitavanja (kao što je skeleton placeholder) za različite delove vašeg korisničkog interfejsa direktno u vašem React stablu:

```js
<Suspense fallback={<TalksLoading />}>
  <Talks confId={conf.id} />
</Suspense>
```

Server Komponente i Suspense su fičeri React-a, a ne Next.js fičeri. Međutim, njihovo usvajanje na nivou framework-a zahteva podršku i ne-trivijalan rad na implementaciji. Trenutno, Next.js App Router je najkompletnija implementacija. React tim sarađuje sa developerima bundler-a da bi ove mogućnosti bile lakše za implementaciju u sledećoj generaciji framework-ova.
</DeepDive>
