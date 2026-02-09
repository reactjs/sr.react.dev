---
title: Ugrađeni React Hook-ovi
---

<Intro>

*Hook-ovi* vam omogućavaju upotrebu različitih React funkcionalnosti u vašim komponentama. Možete koristiti ugrađene Hook-ove ili ih kombinovati da napravite svoje. Na ovoj stranici se nalaze svi ugrađeni Hook-ovi u React-u.

</Intro>

---

## State Hook-ovi {/*state-hooks*/}

*State* omogućava komponenti da ["zapamti" informaciju poput korisničkog input-a](/learn/state-a-components-memory). Na primer, form komponenta može koristiti state da čuva input vrednost, dok komponenta za galeriju slika može koristiti state da čuva indeks izabrane slike.

Da biste dodali state u komponentu, koristite jedan od ovih Hook-ova:

* [`useState`](/reference/react/useState) deklariše state promenljivu koju direktno možete ažurirati.
* [`useReducer`](/reference/react/useReducer) deklariše state promenljivu sa logikom ažuriranja unutar [reducer funkcije](/learn/extracting-state-logic-into-a-reducer).

```js
function ImageGallery() {
  const [index, setIndex] = useState(0);
  // ...
```

---

## Context Hook-ovi {/*context-hooks*/}

*Context* omogućava komponenti da [prima informacije od udaljenih roditelja bez prosleđivanja informacije kao props](/learn/passing-props-to-a-component). Na primer, vaša komponenta na vrhu može proslediti trenutnu temu svim komponentama ispod, bez obzira koliko su duboko.

* [`useContext`](/reference/react/useContext) čita i pretplaćuje se na context.

```js
function Button() {
  const theme = useContext(ThemeContext);
  // ...
```

---

## Ref Hook-ovi {/*ref-hooks*/}

*Ref-ovi* omogućavaju komponenti da [čuva informaciju koja se ne koristi za renderovanje](/learn/referencing-values-with-refs), kao što je DOM čvor ili timeout ID. Za razliku od state-a, ažuriranje ref-a ne renderuje komponentu ponovo. Ref-ovi su "evakuacioni izlaz" u React paradigmi. Korisni su kada trebate raditi sa sistemima koji nisu React, poput ugrađenih API-ja u pretraživaču.

* [`useRef`](/reference/react/useRef) deklariše ref. Možete čuvati bilo koju vrednosti u njoj, ali se najčešće koristi za čuvanje DOM čvora.
* [`useImperativeHandle`](/reference/react/useImperativeHandle) omogućava prilagođavanje izloženog ref-a vaše komponente. Ovo se retko koristi.

```js
function Form() {
  const inputRef = useRef(null);
  // ...
```

---

## Effect Hook-ovi {/*effect-hooks*/}

*Effect-i* omogućavaju komponenti da [se konektuje i sinhronizuje sa eksternim sistemima](/learn/synchronizing-with-effects). Ovo uključuje rad sa mrežom, DOM pretraživača, animacije, widget-e napisane u drugoj biblioteci i ostali kod koji nije napisan u React-u.

* [`useEffect`](/reference/react/useEffect) povezuje komponentu na eksterni sistem.

```js
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
  // ...
```

Effect-i su "evakuacioni izlaz" u React paradigmi. Nemojte koristiti Effect-e da orkestrirate tok podataka u vašoj aplikaciji. Ako ne interagujete sa eksternim sistemom, [možda vam neće biti potreban Effect](/learn/you-might-not-need-an-effect).

Postoje dve retko korišćene varijante `useEffect`-a sa razlikama u tajmingu:

* [`useLayoutEffect`](/reference/react/useLayoutEffect) se okida pre nego što pretraživač ponovo iscrta ekran. Ovde možete meriti layout.
* [`useInsertionEffect`](/reference/react/useInsertionEffect) se okida pre nego što React napravi izmene u DOM-u. Biblioteke ovde mogu ubaciti dinamički CSS.

You can also separate events from Effects:

- [`useEffectEvent`](/reference/react/useEffectEvent) creates a non-reactive event to fire from any Effect hook.
---

## Hook-ovi performansi {/*performance-hooks*/}

Uobičajen način za optimizaciju performansi ponovnih rendera je preskakanje nepotrebnog posla. Na primer, možete reći React-u da ponovo iskoristi keširani proračun ili da preskoči ponovni render ako se podaci nisu promenili od prethodnog rendera.

Da biste preskočili proračune i nepotrebne ponovne rendere, koristite jedan od ovih Hook-ova:

- [`useMemo`](/reference/react/useMemo) vam omogućava da keširate rezultat skupog proračuna.
- [`useCallback`](/reference/react/useCallback) vam omogućava da keširate definiciju funkcije pre njenog prosleđivanja u optimizovanu komponentu.

```js
function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

Ponekad ne možete preskočiti ponovno renderovanje, jer ekran treba da se ažurira. U tom slučaju, možete poboljšati performanse odvajanjem blokirajućih ažuriranja koja moraju biti sinhrona (poput pisanja u input) od neblokirajućih ažuriranja koja ne moraju da blokiraju korisnički interfejs (poput ažuriranja tabele).

Da biste dali prioritet renderovanju, koristite jedan od ovih Hook-ova:

- [`useTransition`](/reference/react/useTransition) vam omogućava da označite promenu state-a kao neblokirajuću i dozvolite da je druga ažuriranja prekinu.
- [`useDeferredValue`](/reference/react/useDeferredValue) vam omogućava da odložite ažuriranje nekritičnih delova UI-a i prvo pustite ažuriranje ostalih delova.

---

## Ostali Hook-ovi {/*other-hooks*/}

Ovi Hook-ovi su uglavnom korisni za autore biblioteka i ne koriste se često u kodu aplikacije.

- [`useDebugValue`](/reference/react/useDebugValue) vam omogućava da prilagodite labelu koju React DevTools prikazuje za vaš prilagođeni Hook.
- [`useId`](/reference/react/useId) omogućava komponenti da sebi asocira jedinstveni ID. Tipično se koristi u API-jima za pristupačnost.
- [`useSyncExternalStore`](/reference/react/useSyncExternalStore) omogućava komponenti da se pretplati na eksterno skladište.
* [`useActionState`](/reference/react/useActionState) vam omogućava da upravljate state-om akcija.

---

## Vaši Hook-ovi {/*your-own-hooks*/}

Možete da [definišete vaše prilagođene Hook-ove](/learn/reusing-logic-with-custom-hooks#extracting-your-own-custom-hook-from-a-component) kao JavaScript funkcije.
