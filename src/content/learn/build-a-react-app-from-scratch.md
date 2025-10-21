---
title: Napraviti React aplikaciju od nule
---

<Intro>

Ako vaša aplikacija ima ograničenja koja nisu adekvatno rešena u postojećim framework-ovima, ako želite da napravite sopstveni framework, ili samo želite naučiti osnove React aplikacija, možete napraviti React aplikaciju od nule.

</Intro>

<DeepDive>

#### Razmislite o korišćenju framework-a {/*consider-using-a-framework*/}

Početak od nule je jednostavan način da počnete koristiti React, ali, glavni kompromis kojeg treba biti svestan, je da odabir ovog puta često znači isto kao i da pravite sopstveni adhoc framework. Kako vam se zahtevi menjaju, možda ćete trebati da rešavate sve više problema koji liče na probleme koje rešavaju framework-ovi, a za koje naši preporučeni framework-ovi već imaju dobro razvijena i podržana rešenja.

Na primer, ako vaša aplikacija u budućnosti treba podržavati renderovanje na serverskoj strani (SSR), generisanje statičkog sajta (SSG) i/ili React Server Components (RSC), moraćete sami da ih implementirate. Slično tome, buduće React funkcionalnosti koje zahtevaju integraciju na nivou framework-a moraćete sami da implementirate ako budete želeli da ih koristite.

Naši preporučeni framework-ovi vam takođe pomažu da pravite aplikacije sa boljim performansama. Na primer, smanjenje ili uklanjanje "vodopada" u mrežnim zahtevima doprinosi boljem korisničkom iskustvu. Ovo možda neće biti najviši prioritet kada pravite manji projekat, ali ako vaša aplikacija bude dobijala korisnike, poželećete da joj poboljšate performanse.

Odabirom ovog puta takođe otežavate i podršku, pošto će način na koji razvijate rutiranje, fetch-ovanje podataka i ostale funkcionalnosti biti jedinstven za vašu situaciju. Trebali biste odabrati ovu opciju jedino ako ste spremni za samostalno rešavanje problema ili ako ste sigurni da vam nikada neće trebati ove funkcionalnosti.

Pogledajte spisak preporučenih framework-ova u [Kreiranje React aplikacije](/learn/creating-a-react-app).

</DeepDive>


## Korak 1: Instaliranje build alata {/*step-1-install-a-build-tool*/}

Prvi korak je instaliranje build alata kao što su `vite`, `parcel` ili `rsbuild`. Ovi build alati pružaju funkcionalnosti za pakovanje i pokretanje izvornog koda, pružaju razvojni server za lokalni razvoj i build komandu za deploy-ovanje vaše aplikacije na produkcioni server.

### Vite {/*vite*/}

[Vite](https://vite.dev/) je build alat koji ima za cilj da pruži brže i efikasnije iskustvo razvoja za moderne web projekte.

<TerminalBlock>
npm create vite@latest my-app -- --template react-ts
</TerminalBlock>

Vite je zgodan jer dolazi sa razumnim default podešavanjima po instalaciji. Vite ima bogat ekosistem plugin-a za podršku brzog osvežavanja, JSX-a, Babel/SWC-a i ostalih uobičajenih funkcionalnosti. Pogledajte Vite [React plugin](https://vite.dev/plugins/#vitejs-plugin-react) ili [React SWC plugin](https://vite.dev/plugins/#vitejs-plugin-react-swc) i [primer React SSR projekta](https://vite.dev/guide/ssr.html#example-projects) da biste počeli.

Vite se već koristi kao build alat u jednom od naših [preporučenih framework-ova](/learn/creating-a-react-app): [React Router-u](https://reactrouter.com/start/framework/installation).

### Parcel {/*parcel*/}

[Parcel](https://parceljs.org/) kombinuje odlično iskustvo razvoja sa skalabilnom arhitekturom što može podržati vaš projekat od početne faze do velikih produkcionih aplikacija.

<TerminalBlock>
npm install --save-dev parcel
</TerminalBlock>

Parcel po instalaciji podržava brzo osvežavanje, JSX, TypeScript, Flow i stilizaciju. Pogledajte [recept za Parcel-ov React](https://parceljs.org/recipes/react/#getting-started) da biste počeli.

### Rsbuild {/*rsbuild*/}

[Rsbuild](https://rsbuild.dev/) je build alat zasnovan na Rspack-u koji pruža neometano iskustvo razvoja za React aplikacije. Dolazi sa pažljivo podešenim default podešavanjima i optimizacijama performansi koje su spremne za upotrebu.

<TerminalBlock>
npx create-rsbuild --template react
</TerminalBlock>

Rsbuild uključuje ugrađenu podršku za React funkcionalnosti poput brzog osvežavanja, JSX-a, TypeScript-a i stilizacije. Pogledajte [vodič za Rsbuild-ov React](https://rsbuild.dev/guide/framework/react) da biste počeli.

<Note>

#### Metro za React Native {/*react-native*/}

Ako počinjete od nule sa React Native-om trebaćete da koristite [Metro](https://metrobundler.dev/), JavaScript bundler za React Native. Metro podržava bundle-ovanje za platforme poput iOS-a i Android-a, ali mu nedostaju mnoge funkcionalnosti u poređenju sa alatima ovde. Preporučujemo da počnete sa Vite-om, Parcel-om ili Rsbuild-om osim ako vaš projekat ne zahteva podršku za React Native.

</Note>

## Korak 2: Pravljenje uobičajenih šablona aplikacija {/*step-2-build-common-application-patterns*/}

Build alati nabrojani iznad započinju sa klijentskom, single-page aplikacijom (SPA), ali ne uključuju naredna rešenja za uobičajene funkcionalnosti poput rutiranja, fetch-ovanja podataka ili stilizacije.

React ekosistem uključuje mnogo alata za ove probleme. Izlistali smo nekoliko njih koji se naširoko koriste kao početna tačka, ali slobodno možete izabrati druge alate ukoliko vam više odgovaraju.

### Rutiranje {/*routing*/}

Rutiranje odlučuje koji sadržaj ili stranica će biti prikazan kada korisnik poseti određeni URL. Potrebno je da podesite ruter da mapira URL-ove do različitih delova vaše aplikacije. Moraćete da rukujete sa ugnježdenim rutama, route parametrima i query parametrima. Ruteri mogu biti konfigurisani u vašem kodu ili definisani na nivou strukture foldera i fajlova vaših komponenata.

Ruteri su sastavni deo modernih aplikacija i često su integrisani sa fetch-ovanjem podataka (uključujući i prefetch-ovanje podataka za celu stranicu radi bržeg učitavanja), razdvajanjem koda (minimizovanje veličine klijentskog bundle-a) i načinom renderovanja stranice (odlučivanje kako će svaka stranica biti generisana).

Preporučujemo da koristite:

- [React Router](https://reactrouter.com/start/data/custom)
- [Tanstack Router](https://tanstack.com/router/latest)


### Fetch-ovanje podataka {/*data-fetching*/}

Fetch-ovanje podataka sa servera ili drugih izvora podataka je ključan deo većine aplikacija. Da bi se ovo pravilno uradilo, neophodno je obraditi stanja učitavanja, stanja grešaka, kao i keširanje fetch-ovanih podataka, što može biti kompleksno.

Specijalizovane biblioteke za fetch-ovanje podataka rade teži posao fetch-ovanja i keširanja podataka za vas, dopuštajući vam da se fokusirate na to kakvi podaci su potrebni vašoj aplikaciji i kako da ih prikažete. Ove biblioteke se tipično koriste direktno u vašim komponentama, ali mogu biti i integrisane u loader-e rutiranja radi bržeg prefetching-a i boljih performansi, kao i za renderovanje na serveru.

Primetite da fetch-ovanje podataka direktno u komponentama može usporiti vreme učitavanja zbog "vodopada" u mrežnim zahtevima, pa preporučujemo da, kad god je moguće, vršite prefetch-ovanje podataka u loader-ima rutiranja ili na serveru! Ovo omogućava da podaci na stranici budu fetch-ovani odjednom dok se stranica prikazuje.

Ako podatke fetch-ujete uglavnom sa backend-a ili REST API-ja, preporučujemo da koristite:

- [TanStack Query](https://tanstack.com/query/)
- [SWR](https://swr.vercel.app/)
- [RTK Query](https://redux-toolkit.js.org/rtk-query/overview)

Ako podatke fetch-ujete sa GraphQL API-ja, preporučujemo da koristite:

- [Apollo](https://www.apollographql.com/docs/react)
- [Relay](https://relay.dev/)


### Razdvajanje koda {/*code-splitting*/}

Razdvajanje koda je proces razlaganja vaše aplikacije u manje bundle-ove koji se mogu učitavati po potrebi. Veličina koda aplikacije raste sa svakom novom funkcionalnošću i dodatnom zavisnošću. Aplikacije mogu postati spore pri učitavanju, jer je neophodno poslati sav kod aplikacije pre nego što se može koristiti. Keširanje, smanjenje broja funkcionalnosti ili zavisnosti, kao i premeštanje koda na server mogu ublažiti sporo učitavanje, ali nisu potpuna rešenja, jer se njihovom preteranom upotrebom mogu izgubiti funkcionalnosti.

Slično tome, ako se oslanjate na to da aplikacije koje koriste vaš framework vrše razdvajanje koda, možete se naći u situacijama da učitavanje upotrebom razdvajanja koda bude sporije. Na primer, [odloženo (lazily) učitavanje](/reference/react/lazy) tabele odlaže slanje koda potrebnog za renderovanje tabele, jer se kod tabele razdvaja od ostatka aplikacije. [Parcel podržava razdvajanje koda pomoću React.lazy](https://parceljs.org/recipes/react/#code-splitting). Međutim, ako tabela učitava podatke *nakon* što se inicijalno renderovala, čekaćete dvaput. Ovo je "vodopad": umesto da istovremeno fetch-ujete podatke za tabelu i šaljete kod na renderovanje, morate čekati da se prvi korak završi, pre nego što počne sledeći.

Razdvajanje koda po ruti integrisano sa bundle-ovanjem i fetch-ovanjem podataka može da skrati inicijalno vreme učitavanja vaše aplikacije, kao i vreme potrebno za renderovanje najvećeg vidljivog sadržaja ([Largest Contentful Paint](https://web.dev/articles/lcp)).

Za instrukcije o razdvajanju koda, pogledajte dokumentaciju build alata:
- [Vite build optimizacije](https://vite.dev/guide/features.html#build-optimizations)
- [Parcel razdvajanje koda](https://parceljs.org/features/code-splitting/)
- [Rsbuild razdvajanje koda](https://rsbuild.dev/guide/optimization/code-splitting)

### Poboljšanje performansi aplikacije {/*improving-application-performance*/}

Pošto vaš odabrani build alat podržava jedino single page aplikacije (SPA), potrebno je da implementirate ostale [šablone renderovanja](https://www.patterns.dev/vanilla/rendering-patterns) poput renderovanja na serverskoj strani (SSR), generisanja statičkog sajta (SSG) i/ili React Server Components (RSC). Iako vam ove funkcionalnosti ne trebaju na početku, u budućnosti mogu postojati neke rute koje bi imale koristi od SSR, SSG ili RSC.

* **Single-page aplikacije (SPA)** učitavaju jednu HTML stranicu i dinamički ažuriraju stranicu u skladu sa interakcijama korisnika sa aplikacijom. Lakše je započeti rad sa ovim aplikacijama, ali mogu biti sporije pri inicijalnom učitavanju. Većina build alata koristi SPA kao default arhitekturu.

* **Strimovano renderovanje na serverskoj strani (SSR)** renderuje stranicu na serveru i šalje potpuno izrenderovanu stranicu klijentu. SSR može poboljšati performanse, ali njegovo postavljanje i održavanje mogu biti kompleksniji od single-page aplikacija. Uz dodatak strimovanja, SSR može biti veoma kompleksan za postavljanje i održavanje. Pogledajte [Vite-ov vodič za SSR]( https://vite.dev/guide/ssr).

* **Generisanje statičkog sajta (SSG)** generiše statičke HTML fajlove za vašu aplikaciju tokom vremena izgradnje. SSG može poboljšati performanse, ali može biti kompleksniji za postavljanje i održavanje od renderovanja na serverskoj strani. Pogledajte [Vite-ov vodič za SSG](https://vite.dev/guide/ssr.html#pre-rendering-ssg).

* **React Server Components (RSC)** omogućava kombinovanje komponenata generisanih tokom vremena izgradnje, isključivo serverskih komponenata i interaktivnih komponenata u jedno React stablo. RSC može poboljšati performanse, ali trenutno zahteva duboko znanje za postavljanje i održavanje. Pogledajte [Parcel-ove primere RSC-a](https://github.com/parcel-bundler/rsc-examples).

Vaše strategije renderovanja moraju biti integrisane sa vašim ruterom kako bi aplikacije napravljene pomoću vašeg framework-a mogle da biraju strategiju renderovanja na nivou rute. Ovo će omogućiti primenu različitih strategija renderovanja bez potrebe za ponovnim pisanjem cele aplikacije. Na primer, početna stranica vaše aplikacije može imati koristi od statičkog generisanja (SSG), dok bi stranica sa konkretnim sadržajem mogla imati najbolje performanse sa renderovanjem na serverskoj strani.

Upotreba odgovarajuće strategije renderovanja za odgovarajuće rute može smanjiti vreme potrebno za učitavanje prvog bajta sadržaja ([Time to First Byte](https://web.dev/articles/ttfb)), vreme do renderovanja prvog dela sadržaja ([First Contentful Paint](https://web.dev/articles/fcp)), kao i vreme potrebno za renderovanje najvećeg vidljivog sadržaja ([Largest Contentful Paint](https://web.dev/articles/lcp)).

### I još... {/*and-more*/}

Ovo su samo neki od primera funkcionalnosti koje treba razmotriti prilikom pravljenja nove aplikacije od nule. Mnoga ograničenja na koja ćete naići se teško mogu rešiti, jer su problemi međusobno povezani i zahtevaju duboko poznavanje oblasti sa kojima možda niste upoznati.

Ako ove probleme ne želite samostalno da rešavate, možete [početi sa framework-om](/learn/creating-a-react-app) koji pruža ove funkcionalnosti po instalaciji.
