---
title: Instalacija
---

<Intro>

React je dizajniran od početka za postepeno usvajanje. Možete koristiti koliko god React-a želite. Bez obzira da li želite da probate React, dodate neku interaktivnost na HTML stranicu ili da započnete kompleksnu React aplikaciju, ova sekcija će vam pomoći da počnete.

</Intro>

## Isprobajte React {/*try-react*/}

Ne morate ništa instalirati da bi ste se igrali sa React-om. Probajte da izmenite ovaj sandbox!

<Sandpack>

```js
function Greeting({ name }) {
  return <h1>Zdravo, {name}</h1>;
}

export default function App() {
  return <Greeting name="svet" />
}
```

</Sandpack>

Možete ga direktno menjati ili otvoriti u novom tabu pritiskom na "Fork" dugme u gornjem desnom uglu.

Većina stranica u React dokumentaciji sadrži sandbox-e kao ovaj. Van React dokumentacije postoje mnogi online sandbox-i koji podržavaju React: [CodeSandbox](https://codesandbox.io/s/new), [StackBlitz](https://stackblitz.com/fork/react) ili [CodePen](https://codepen.io/pen?template=QWYVwWN).

Da isprobate React lokalno na vašem računaru, [preuzmite ovu HTML stranicu](https://gist.githubusercontent.com/gaearon/0275b1e1518599bbeafcde4722e79ed1/raw/db72dcbf3384ee1708c4a07d3be79860db04bff0/example.html). Otvorite je u vašem editoru i u vašem pretraživaču!

## Kreirajte React aplikacije {/*creating-a-react-app*/}

Ako želite da napravite novu React aplikaciju, možete [kreirati React aplikaciju](/learn/creating-a-react-app) pomoću preporučenog framework-a.

## Napravite React framework {/*build-a-react-framework*/}

Ako framework nije pogodan za vaš projekat, ili želite da započnete sa pravljenjem sopstvenog framework-a, možete [napraviti sopstveni React framework](/learn/building-a-react-framework).

## Dodajte React u postojeći projekat {/*add-react-to-an-existing-project*/}

Ako žeite da isprobate React na vašem postojećem sajtu ili aplikaciji, možete [dodati React u postojeći projekat](/learn/add-react-to-an-existing-project).

## Deprecated opcije {/*deprecated-options*/}

### Create React App (Deprecated) {/*create-react-app-deprecated*/}

Create React App je deprecated alat, koji je ranije bio preporučen za kreiranje novih React aplikacija. Ako želite da napravite novu React aplikaciju, možete [kreirati React aplikaciju](/learn/creating-a-react-app) pomoću preporučenog framework-a.

Za više informacija, pogledajte [Gašenje Create React App](/blog/2025/02/14/sunsetting-create-react-app).

## Sledeći koraci {/*next-steps*/}

Idite na [Brzi Uvod](/learn) vodič za turu najvažnijih React koncepta sa kojima ćete se susretati svakodnevno.
