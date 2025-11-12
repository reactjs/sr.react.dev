---
title: Pregled React referenci
---

<Intro>

Ova sekcija vam pruža detaljnu dokumentaciju referenci za rad sa React-om. Za uvod u sam React, molimo vas posetite sekciju [Nauči](/learn).

</Intro>

Dokumentacija React referenci je podeljena na par funkcionalnih podsekcija:

## React {/*react*/}

Programske React funkcionalnosti:

* [Hook-ovi](/reference/react/hooks) - Upotreba različitih React funkcionalnosti u vašim komponentama.
* [Komponente](/reference/react/components) - Ugrađene komponente koje možete koristiti u vašem JSX-u.
* [API-ji](/reference/react/apis) - API-ji koji su korisni za definisanje komponenti.
* [Direktive](/reference/rsc/directives) - Pružanje instrukcija bundler-ima kompatibilnim sa React Server Components.

## React DOM {/*react-dom*/}

React DOM sadrži funkcionalnosti koje su podržane samo za web aplikacije (koje su pokrenute u DOM okruženju pretraživača). Ova sekcija se deli na sledeće celine:

* [Hook-ovi](/reference/react-dom/hooks) - Hook-ovi za web aplikacije koje su pokrenute u DOM okruženju pretraživača.
* [Komponente](/reference/react-dom/components) - React sadrži sve HTML i SVG komponente ugrađene u pretraživač.
* [API-ji](/reference/react-dom) - `react-dom` paket sadrži metode podržane samo u web aplikacijama.
* [Klijentski API-ji](/reference/react-dom/client) - `react-dom/client` API-ji omogućavaju renderovanje React komponenata na klijentu (u pretraživaču).
* [Serverski API-ji](/reference/react-dom/server) - `react-dom/server` API-ji omogućavaju renderovanje React komponenata u HTML na serveru.
* [Statički API-ji](/reference/react-dom/static) - `react-dom/static` API-ji omogućavaju generisanje statičkog HTML-a za React komponente.

## React kompajler {/*react-compiler*/}

React kompajler je alat za optimizaciju vremena izgradnje koji automatski memoriše vaše React komponente i vrednosti:

* [Konfiguracija](/reference/react-compiler/configuration) - Opcije za konfiguraciju React kompajlera.
* [Direktive](/reference/react-compiler/directives) - Direktive na nivou funkcija za kontrolu kompilacije.
* [Kompajliranje biblioteka](/reference/react-compiler/compiling-libraries) - Uputstvo za isporuku prekompajliranog koda biblioteke.

## ESLint Plugin React Hooks {/*eslint-plugin-react-hooks*/}

[ESLint plugin za React Hook-ove](/reference/eslint-plugin-react-hooks) pomaže u sprovođenju pravila React-a:

* [Lint-ovi](/reference/eslint-plugin-react-hooks) - Detaljna dokumentacija za svaki lint sa primerima.

## Pravila React-a {/*rules-of-react*/}

React ima osobine — tj. pravila — za izražavanje šablona na način koji je lako razumljiv i čini aplikacije visokokvalitetnim:

* [Komponente i Hook-ovi moraju biti čisti](/reference/rules/components-and-hooks-must-be-pure) – Čistoća čini vaš kod lakšim za razumevanje i debug-ovanje i omogućava React-u da ispravno automatski optimizuje vaše komponente i hook-ove.
* [React poziva komponente i hook-ove](/reference/rules/react-calls-components-and-hooks) – React je zadužen za renderovanje komponenata i hook-ova kada je to potrebno za optimizaciju korisničkog iskustva.
* [Pravila Hook-ova](/reference/rules/rules-of-hooks) – Hook-ovi su definisani upotrebom JavaScript funkcija, ali predstavljaju poseban tip reusable UI logike sa ograničenjima gde mogu biti pozvani.

## Legacy API-ji {/*legacy-apis*/}

* [Legacy API-ji](/reference/react/legacy) - Export-ovani iz `react` paketa, ali se ne preporučuje upotreba u novonapisanom kodu.
