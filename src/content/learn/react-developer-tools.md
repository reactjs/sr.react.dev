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

Sada, ako posetite sajt **napravljen sa React-om,** videćete *Components* i *Profiler* panele.

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

Da bi pregledali aplikacije napravljene u [React Native](https://reactnative.dev/), možete koristiti [React Native DevTools](https://reactnative.dev/docs/react-native-devtools), ugrađeni debugger koji se duboko integriše sa React Developer Tools. Sve funkcionalnosti rade identično kao i browser ekstenzija, uključujući isticanje native elemenata i selekciju.

[Naučite više o debug-ovanju u React Native.](https://reactnative.dev/docs/debugging)

> Za React Native verzije pre 0.76, molimo koristite standalone React DevTools prateći uputstvo [Safari i drugi pretraživači](#safari-and-other-browsers) iznad.
