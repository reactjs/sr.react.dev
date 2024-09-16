---
title: Postavka Radnog okruženja (Editora)
---

<Intro>

Dobro kofigurisan editor može učiniti kod jasnijim za čitanje i bržim za pisanje. Može vam čak pomoći da primetite greške dok ih pišete! Ako je ovo prvi put da postavljate editor ili želite da podesite vaš trenutni editor, imamo nekoliko preporuka.

</Intro>

<YouWillLearn>

* Koji editori su najpopularniji
* Kako da automatski formatirate vaš kod

</YouWillLearn>

## Vaš editor {/*your-editor*/}

[VS Code](https://code.visualstudio.com/) je jedan od najpopularnijih editora koji se koriste danas. Ima veliko tržište ekstenzija i dobro se integriše sa popularnim servisima kao što je GitHub. Većina funkcija navedenih ispod mogu se dodati u VS Code kao ekstenzije, što ga čini visoko konfigurabilnim!

Drugi popularni editori koji se koriste u React zajednici su:

* [WebStorm](https://www.jetbrains.com/webstorm/) je integrisano razvojno okruženje dizajnirano posebno za JavaScript.
* [Sublime Text](https://www.sublimetext.com/) ima ugrađenu podršku za JSX i TypeScript, [podršku za sintaksu](https://stackoverflow.com/a/70960574/458193) i automatsko završavanje.
* [Vim](https://www.vim.org/) je visoko konfigurabilan tekst editor napravljen da pravljenje i menjanje bilo kog tipa teksta bude veoma efikasno. Uključen je kao "vi" sa većinom UNIX sistema i sa Apple OS X.

## Preporučene funkcije editora {/*recommended-editor-features*/}

Neki editori dolaze sa ugrađenim funkcijama, dok drugi zahtevaju dodavanje ekstenzija. Proverite koje funkcije podržava vaš editor koji ste izabrali!

### Linting {/*linting*/}

Linting alati pronalaze probleme u vašem kodu dok ga pišete, pomažući vam da ih ispravite na vreme. [ESLint](https://eslint.org/) je popularan, open source linter za JavaScript.

* [Instalirajte ESLint sa preporučenom konfiguracijom za React](https://www.npmjs.com/package/eslint-config-react-app) (budite sigurni da imate [Node instaliran!](https://nodejs.org/en/download/current/))
* [Integrišite ESLint u VSCode sa zvaničnom ekstenzijom](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)

**Budite sigurni da ste omogućili sve [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks) pravila za vaš projekat.** Oni su esencijalni i hvataju najozbiljnije greške na vreme. Preporučena [`eslint-config-react-app`](https://www.npmjs.com/package/eslint-config-react-app) konfiguracija već ih uključuje.

### Formatiranje {/*formatting*/}

Poslednja stvar koju želite da radite kada delite vaš kod sa drugim saradnicima je da se upustite u diskusiju o [tabovima i razmacima](https://www.google.com/search?q=tabs+vs+spaces)! Srećom, [Prettier](https://prettier.io/) će očistiti vaš kod tako što će ga preformatirati u skladu sa unapred podešenim, konfigurabilnim pravilima. Pokrenite Prettier i svi vaši tabovi će biti konvertovani u razmake - i vaša uvlačenja, navodnici, itd. će takođe biti promenjeni u skladu sa konfiguracijom. U idealnom slučaju, Prettier će se pokrenuti kada sačuvate vaš fajl, brzo praveći ove izmene za vas.

Možete instalirati [Prettier ekstenziju u VSCode](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) prateći ove korake:

1. Pokrenite VS Code
2. Koristite Brzo otvaranje (pritisnite Ctrl/Cmd+P)
3. Zalepite `ext install esbenp.prettier-vscode`
4. Pritisnite Enter

#### Formatiranje pri čuvanju {/*formatting-on-save*/}

Idealno, trebalo bi da formatirate vaš kod pri svakom čuvanju. VS Code ima podešavanja za ovo!

1. U VS Code, pritisnite `CTRL/CMD + SHIFT + P`.
2. Ukucajte "settings"
3. Pritisnite Enter
4. U pretraživaču, ukucajte "format on save"
5. Budite sigurni da je "format on save" opcija označena!

> Ako vaš ESLint preset ima pravila za formatiranje, ona mogu biti u konfliktu sa Prettier-om. Preporučujemo da onemogućite sva pravila za formatiranje u vašem ESLint presetu koristeći [`eslint-config-prettier`](https://github.com/prettier/eslint-config-prettier) tako da ESLint bude *samo* korišćen za hvatanje logičkih grešaka. Ako želite da primoravate da se fajlovi formatiraju pre nego što se spoje u glavnu granu, koristite [`prettier --check`](https://prettier.io/docs/en/cli.html#--check) za vašu kontinuiranu integraciju.
