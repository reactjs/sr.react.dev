---
title: React Developer Tools
---

<Intro>

Koristite React Developer Tools da inspekcijom React [komponenti](/learn/your-first-component), izmenite [props](/learn/passing-props-to-a-component) i [state](/learn/state-a-components-memory) i identifikujete probleme sa performansama.

</Intro>

<YouWillLearn>

* Kako da instalirate React Developer Tools

</YouWillLearn>

## Browser ekstenzija {/*browser-extension*/}

Najlakši način da debagujete sajtove napravljene sa React-om je da instalirate React Developer Tools ekstenziju za pretraživač. Dostupna je za nekoliko popularnih pretraživača:

* [Instalirajte za **Chrome**](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en)
* [Instalirajte za **Firefox**](https://addons.mozilla.org/en-US/firefox/addon/react-devtools/)
* [Instalirajte za **Edge**](https://microsoftedge.microsoft.com/addons/detail/react-developer-tools/gpphkfbcpidddadnkolkpfckpihlkkil)

Sada, ako posetite sajt **napravljen sa React-om,** videćete _Components_ i _Profiler_ panele.

![React Developer Tools ekstenzija](/images/docs/react-devtools-extension.png)

### Safari i drugi pretraživači {/*safari-and-other-browsers*/}

Za svaki pretraživač (na primer, Safari), instalirajte [`react-devtools`](https://www.npmjs.com/package/react-devtools) npm paket:
```bash
# Yarn
yarn global add react-devtools

# Npm
npm install -g react-devtools
```

Sledeće, otvorite developer tools iz terminala:
```bash
react-devtools
```

Onda konektujte vaš sajt dodavanjem sledećeg `<script>` taga na početak `<head>` taga vašeg sajta:
```html {3}
<html>
  <head>
    <script src="http://localhost:8097"></script>
```

Reload-ujte vaš sajt u pretraživaču da bi ga videli u developer tools.

![React Developer Tools standalone](/images/docs/react-devtools-standalone.png)

## Mobilni telefoni (React Native) {/*mobile-react-native*/}
React Developer Tools može se koristiti za inspekciju aplikacija napravljenih sa [React Native](https://reactnative.dev/).

Najlakši način da koristite React Developer Tools je da ga instalirate globalno:
```bash
# Yarn
yarn global add react-devtools

# Npm
npm install -g react-devtools
```

Zatim otvorite Developer Tools iz terminala:
```bash
react-devtools
```

Trebalo bi da se poveže sa bilo kojom lokalnom React Native aplikacijom koja je pokrenuta.

> Pokušajte da reload-ujete aplikaciju ako developer tools ne uspe da se poveže nakon nekoliko sekundi.

[Naučite više o debagovanju React Native.](https://reactnative.dev/docs/debugging)

