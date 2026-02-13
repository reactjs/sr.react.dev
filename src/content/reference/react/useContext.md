---
title: useContext
---

<Intro>

`useContext` je React Hook koji vam omogućava da čitate i da se pretplatite na [context](/learn/passing-data-deeply-with-context) iz vaše komponente.

```js
const value = useContext(SomeContext)
```

</Intro>

<InlineToc />

---

## Reference {/*reference*/}

### `useContext(SomeContext)` {/*usecontext*/}

Pozovite `useContext` na vrhu vaše komponente da pročitate i da se pretplatite na [context](/learn/passing-data-deeply-with-context).

```js
import { useContext } from 'react';

function MyComponent() {
  const theme = useContext(ThemeContext);
  // ...
```

[Pogledajte još primera ispod.](#usage)

#### Parametri {/*parameters*/}

* `SomeContext`: Context koji ste prethodno kreirali sa [`createContext`](/reference/react/createContext). Sam context ne sadrži informaciju, već samo predstavlja vrstu informacije koju možete pružiti i čitati iz komponenti.

#### Povratne vrednosti {/*returns*/}

`useContext` vraća vrednost context-a za pozivajuću komponentu. Određuje se kao `value` prosleđena najbližem `SomeContext`-u iznad pozivajuće komponente u stablu. Ako ne postoji takav provider, onda će povratna vrednost biti `defaultValue` koju ste prosledili u [`createContext`](/reference/react/createContext) za taj context. Povratna vrednost je uvek ažurna. React automatski ponovo renderuje komponente koje čitaju neki context koji se promenio.

#### Upozorenja {/*caveats*/}

* Na `useContext()` poziv u komponenti ne utiču provider-i vraćeni iz *iste* komponente. Odgovarajući `<Context>` **mora biti *iznad*** komponente koja poziva `useContext()`.
* React **automatski ponovo renderuje** svu decu koja koriste specifičan context počevši od provider-a koji prima drugačiji `value`. Prethodna i naredna vrednost se porede pomoću [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) poređenja. Preskakanje ponovnih rendera sa [`memo`](/reference/react/memo) ne sprečava da deca prime nove vrednosti context-a.
* Ako vaš sistem izgradnje pravi duplirane module na izlazu (što se može desiti sa simboličkim linkovima (eng. symlink)), možete slomiti context. Prosleđivanje nečega kroz context radi samo ako su `SomeContext` koji koristite za pružanje context-a i `SomeContext` koji koristite za njegovo čitanje ***potpuno* isti objekat**, utvrđeno `===` poređenjem.

---

## Upotreba {/*usage*/}


### Prosleđivanje podataka duboko u stablo {/*passing-data-deeply-into-the-tree*/}

Pozovite `useContext` na vrhu vaše komponente da pročitate i da se pretplatite na [context](/learn/passing-data-deeply-with-context).

```js [[2, 4, "theme"], [1, 4, "ThemeContext"]]
import { useContext } from 'react';

function Button() {
  const theme = useContext(ThemeContext);
  // ... 
```

`useContext` vraća <CodeStep step={2}>vrednost context-a</CodeStep> za <CodeStep step={1}>context</CodeStep> koji ste prosledili. Da bi odredio vrednost context-a, React pretražuje stablo komponente i pronalazi **najbližeg context provider-a iznad** za taj specifični context.

Da biste prosledili context u `Button`, obmotajte jednu od njegovih roditeljskih komponenata sa odgovarajućim context provider-om:

```js [[1, 3, "ThemeContext"], [2, 3, "\\"dark\\""], [1, 5, "ThemeContext"]]
function MyPage() {
  return (
    <ThemeContext value="dark">
      <Form />
    </ThemeContext>
  );
}

function Form() {
  // ... renderuje dugmiće unutra ...
}
```

Nije bitno koliko slojeva komponenata postoji između provider-a i `Button`-a. Kada `Button` *negde* unutar `Form`-a pozove `useContext(ThemeContext)`, primiće `"dark"` kao vrednost.

<Pitfall>

`useContext()` uvek traži najbližeg provider-a *iznad* komponente koja ga poziva. Traži nagore i **ne** razmatra provider-e u komponenti u kojoj je pozvan `useContext()`.

</Pitfall>

<Sandpack>

```js
import { createContext, useContext } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  return (
    <ThemeContext value="dark">
      <Form />
    </ThemeContext>
  )
}

function Form() {
  return (
    <Panel title="Dobro došli">
      <Button>Registruj se</Button>
      <Button>Uloguj se</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className}>
      {children}
    </button>
  );
}
```

```css
.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>

---

### Ažuriranje podataka prosleđenih kroz context {/*updating-data-passed-via-context*/}

Često ćete želeti da se context vremenom menja. Da biste ažurirali context, kombinujte ga sa [state-om](/reference/react/useState). Deklarišite state promenljivu u roditeljskoj komponenti i prosledite trenutni state provider-u kao <CodeStep step={2}>vrednost context-a</CodeStep>.

```js {2} [[1, 4, "ThemeContext"], [2, 4, "theme"], [1, 11, "ThemeContext"]]
function MyPage() {
  const [theme, setTheme] = useState('dark');
  return (
    <ThemeContext value={theme}>
      <Form />
      <Button onClick={() => {
        setTheme('light');
      }}>
        Promeni na svetlu temu
      </Button>
    </ThemeContext>
  );
}
```

Sada će svaki `Button` unutar provider-a primiti trenutnu `theme` vrednost. Ako pozovete `setTheme` da ažurirate `theme` vrednost koju prosleđujete provider-u, sve `Button` komponente će se ponovo renderovati sa novom `'light'` vrednošću.

<Recipes titleText="Primeri ažuriranja context-a" titleId="examples-basic">

#### Ažuriranje vrednosti kroz context {/*updating-a-value-via-context*/}

U ovom primeru, `MyApp` komponenta čuva state promenljivu koja se prosleđuje u `ThemeContext` provider. Štikliranjem "Tamni režim" checkbox-a, ažurira se state. Promena pružene vrednosti ponovo renderuje sve komponente koje koriste taj context.

<Sandpack>

```js
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext value={theme}>
      <Form />
      <label>
        <input
          type="checkbox"
          checked={theme === 'dark'}
          onChange={(e) => {
            setTheme(e.target.checked ? 'dark' : 'light')
          }}
        />
        Koristi tamni režim
      </label>
    </ThemeContext>
  )
}

function Form({ children }) {
  return (
    <Panel title="Dobro došli">
      <Button>Registruj se</Button>
      <Button>Uloguj se</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className}>
      {children}
    </button>
  );
}
```

```css
.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
  margin-bottom: 10px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>

Primetite da `value="dark"` prosleđuje `"dark"` string, a `value={theme}` prosleđuje vrednost JavaScript-ove `theme` promenljive sa [JSX vitičastim zagradama](/learn/javascript-in-jsx-with-curly-braces). Vitičaste zagrade vam takođe omogućavaju da context-u prosledite vrednosti koje nisu stringovi.

<Solution />

#### Ažuriranje objekta kroz context {/*updating-an-object-via-context*/}

U ovom primeru postoji `currentUser` state promenljiva koja čuva objekat. Kombinujete `{ currentUser, setCurrentUser }` u jedan objekat i prosleđujete ga kroz context unutar `value={}`. Ovo omogućava komponentama ispod, poput `LoginButton`-a, da čitaju i `currentUser` i `setCurrentUser` i  da pozivaju `setCurrentUser` kada im je potrebno.

<Sandpack>

```js
import { createContext, useContext, useState } from 'react';

const CurrentUserContext = createContext(null);

export default function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);
  return (
    <CurrentUserContext
      value={{
        currentUser,
        setCurrentUser
      }}
    >
      <Form />
    </CurrentUserContext>
  );
}

function Form({ children }) {
  return (
    <Panel title="Dobro došli">
      <LoginButton />
    </Panel>
  );
}

function LoginButton() {
  const {
    currentUser,
    setCurrentUser
  } = useContext(CurrentUserContext);

  if (currentUser !== null) {
    return <p>Ulogovan si kao {currentUser.name}.</p>;
  }

  return (
    <Button onClick={() => {
      setCurrentUser({ name: 'Advika' })
    }}>Uloguj se kao Advika</Button>
  );
}

function Panel({ title, children }) {
  return (
    <section className="panel">
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children, onClick }) {
  return (
    <button className="button" onClick={onClick}>
      {children}
    </button>
  );
}
```

```css
label {
  display: block;
}

.panel {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
  margin-bottom: 10px;
}

.button {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}
```

</Sandpack>

<Solution />

#### Više context-a {/*multiple-contexts*/}

U ovom primeru postoje dva nezavisna context-a. `ThemeContext` pruža trenutnu temu, koja je string, dok `CurrentUserContext` čuva objekat koji predstavlja trenutnog korisnika.

<Sandpack>

```js
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext(null);
const CurrentUserContext = createContext(null);

export default function MyApp() {
  const [theme, setTheme] = useState('light');
  const [currentUser, setCurrentUser] = useState(null);
  return (
    <ThemeContext value={theme}>
      <CurrentUserContext
        value={{
          currentUser,
          setCurrentUser
        }}
      >
        <WelcomePanel />
        <label>
          <input
            type="checkbox"
            checked={theme === 'dark'}
            onChange={(e) => {
              setTheme(e.target.checked ? 'dark' : 'light')
            }}
          />
          Koristi tamni režim
        </label>
      </CurrentUserContext>
    </ThemeContext>
  )
}

function WelcomePanel({ children }) {
  const {currentUser} = useContext(CurrentUserContext);
  return (
    <Panel title="Dobro došli">
      {currentUser !== null ?
        <Greeting /> :
        <LoginForm />
      }
    </Panel>
  );
}

function Greeting() {
  const {currentUser} = useContext(CurrentUserContext);
  return (
    <p>Ulogovan si kao {currentUser.name}.</p>
  )
}

function LoginForm() {
  const {setCurrentUser} = useContext(CurrentUserContext);
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const canLogin = firstName.trim() !== '' && lastName.trim() !== '';
  return (
    <>
      <label>
        Ime{': '}
        <input
          required
          value={firstName}
          onChange={e => setFirstName(e.target.value)}
        />
      </label>
      <label>
        Prezime{': '}
        <input
        required
          value={lastName}
          onChange={e => setLastName(e.target.value)}
        />
      </label>
      <Button
        disabled={!canLogin}
        onClick={() => {
          setCurrentUser({
            name: firstName + ' ' + lastName
          });
        }}
      >
        Uloguj se
      </Button>
      {!canLogin && <i>Popuni oba polja.</i>}
    </>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children, disabled, onClick }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button
      className={className}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
}
```

```css
label {
  display: block;
}

.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
  margin-bottom: 10px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>

<Solution />

#### Izdvajanje provider-a u komponentu {/*extracting-providers-to-a-component*/}

Kako vaša aplikacija raste, očekivano je da imate "piramidu" context-a bližu korenu aplikacije. Nema ničeg lošeg u tome. Međutim, ako vam se ugnježdavanje ne sviđa estetski, možete izdvojiti provider-e u posebnu komponentu. U ovom primeru, `MyProviders` sakriva "instalacije" context-a i renderuje prosleđenu decu unutar neophodnih provider-a. Primetite da su `theme` i `setTheme` potrebni u `MyApp`, pa `MyApp` i dalje čuvaj taj state.

<Sandpack>

```js
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext(null);
const CurrentUserContext = createContext(null);

export default function MyApp() {
  const [theme, setTheme] = useState('light');
  return (
    <MyProviders theme={theme} setTheme={setTheme}>
      <WelcomePanel />
      <label>
        <input
          type="checkbox"
          checked={theme === 'dark'}
          onChange={(e) => {
            setTheme(e.target.checked ? 'dark' : 'light')
          }}
        />
        Koristi tamni režim
      </label>
    </MyProviders>
  );
}

function MyProviders({ children, theme, setTheme }) {
  const [currentUser, setCurrentUser] = useState(null);
  return (
    <ThemeContext value={theme}>
      <CurrentUserContext
        value={{
          currentUser,
          setCurrentUser
        }}
      >
        {children}
      </CurrentUserContext>
    </ThemeContext>
  );
}

function WelcomePanel({ children }) {
  const {currentUser} = useContext(CurrentUserContext);
  return (
    <Panel title="Dobro došli">
      {currentUser !== null ?
        <Greeting /> :
        <LoginForm />
      }
    </Panel>
  );
}

function Greeting() {
  const {currentUser} = useContext(CurrentUserContext);
  return (
    <p>Ulogovan si kao {currentUser.name}.</p>
  )
}

function LoginForm() {
  const {setCurrentUser} = useContext(CurrentUserContext);
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const canLogin = firstName !== '' && lastName !== '';
  return (
    <>
      <label>
        Ime{': '}
        <input
          required
          value={firstName}
          onChange={e => setFirstName(e.target.value)}
        />
      </label>
      <label>
        Prezime{': '}
        <input
        required
          value={lastName}
          onChange={e => setLastName(e.target.value)}
        />
      </label>
      <Button
        disabled={!canLogin}
        onClick={() => {
          setCurrentUser({
            name: firstName + ' ' + lastName
          });
        }}
      >
        Uloguj se
      </Button>
      {!canLogin && <i>Popuni oba polja.</i>}
    </>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children, disabled, onClick }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button
      className={className}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
}
```

```css
label {
  display: block;
}

.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
  margin-bottom: 10px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>

<Solution />

#### Skaliranje sa reducer-om i context-om {/*scaling-up-with-context-and-a-reducer*/}

U većim aplikacijama uobičajeno je da kombinujete context sa [reducer-om](/reference/react/useReducer) da biste izvan komponenti izdvojili logiku vezanu za neki state. U ovom primeru, sve "žice" su skrivene u `TasksContext.js`, koji sadrži reducer i dva nezavisna context-a.

Pročitajte [kompletno uputstvo](/learn/scaling-up-with-reducer-and-context) za ovaj primer.

<Sandpack>

```js src/App.js
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksProvider } from './TasksContext.js';

export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>Slobodan dan u Kjotu</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
```

```js src/TasksContext.js
import { createContext, useContext, useReducer } from 'react';

const TasksContext = createContext(null);

const TasksDispatchContext = createContext(null);

export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  return (
    <TasksContext value={tasks}>
      <TasksDispatchContext value={dispatch}>
        {children}
      </TasksDispatchContext>
    </TasksContext>
  );
}

export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Nepoznata akcija: ' + action.type);
    }
  }
}

const initialTasks = [
  { id: 0, text: 'Philosopher’s Path', done: true },
  { id: 1, text: 'Poseti hram', done: false },
  { id: 2, text: 'Popij mača čaj', done: false }
];
```

```js src/AddTask.js
import { useState, useContext } from 'react';
import { useTasksDispatch } from './TasksContext.js';

export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useTasksDispatch();
  return (
    <>
      <input
        placeholder="Add task"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        dispatch({
          type: 'added',
          id: nextId++,
          text: text,
        }); 
      }}>Dodaj</button>
    </>
  );
}

let nextId = 3;
```

```js src/TaskList.js
import { useState, useContext } from 'react';
import { useTasks, useTasksDispatch } from './TasksContext.js';

export default function TaskList() {
  const tasks = useTasks();
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task task={task} />
        </li>
      ))}
    </ul>
  );
}

function Task({ task }) {
  const [isEditing, setIsEditing] = useState(false);
  const dispatch = useTasksDispatch();
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            dispatch({
              type: 'changed',
              task: {
                ...task,
                text: e.target.value
              }
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Sačuvaj
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Izmeni
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          dispatch({
            type: 'changed',
            task: {
              ...task,
              done: e.target.checked
            }
          });
        }}
      />
      {taskContent}
      <button onClick={() => {
        dispatch({
          type: 'deleted',
          id: task.id
        });
      }}>
        Obriši
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

</Sandpack>

<Solution />

</Recipes>

---

### Specificiranje default vrednosti {/*specifying-a-fallback-default-value*/}

Ako React ne može da pronađe nijednog provider-a za taj specifični <CodeStep step={1}>context</CodeStep> u stablu roditelja, vrednost context-a koju vraća `useContext()` će biti jednaka <CodeStep step={3}>default vrednosti</CodeStep> koju ste specificirali prilikom [kreiranja tog context-a](/reference/react/createContext):

```js [[1, 1, "ThemeContext"], [3, 1, "null"]]
const ThemeContext = createContext(null);
```

Default vrednost se **nikad ne menja**. Ako želite ažurirati context, koristite ga uz state kao što je [gore opisano](#updating-data-passed-via-context).

Često, umesto `null`, postoji značajnija vrednost koju možete koristiti kao default, na primer:

```js [[1, 1, "ThemeContext"], [3, 1, "light"]]
const ThemeContext = createContext('light');
```

Na ovaj način, ako slučajno renderujete neku komponentu bez odgovarajućeg provider-a, sve će raditi. Ovo takođe pomaže vašim komponentama da rade dobro u testnom okruženju bez podešavanja mnogo provider-a u testovima.

U primeru ispod, "Promeni temu" dugme je uvek svetlo jer je **izvan svih provider-a theme context-a**, a default vrednost za context je `'light'`. Probajte da promenite default temu da bude `'dark'`.

<Sandpack>

```js
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext('light');

export default function MyApp() {
  const [theme, setTheme] = useState('light');
  return (
    <>
      <ThemeContext value={theme}>
        <Form />
      </ThemeContext>
      <Button onClick={() => {
        setTheme(theme === 'dark' ? 'light' : 'dark');
      }}>
        Promeni temu
      </Button>
    </>
  )
}

function Form({ children }) {
  return (
    <Panel title="Dobro došli">
      <Button>Registruj se</Button>
      <Button>Uloguj se</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children, onClick }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className} onClick={onClick}>
      {children}
    </button>
  );
}
```

```css
.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
  margin-bottom: 10px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>

---

### Override-ovanje context-a za deo stabla {/*overriding-context-for-a-part-of-the-tree*/}

Možete override-ovati context za deo stabla obmotavanjem tog dela sa provider-om koji ima drugačiju vrednost.

```js {3,5}
<ThemeContext value="dark">
  ...
  <ThemeContext value="light">
    <Footer />
  </ThemeContext>
  ...
</ThemeContext>
```

Možete ugnježdavati i override-ovati provider-e koliko god puta želite.

<Recipes titleText="Primeri override-ovanja context-a">

#### Override-ovanje teme {/*overriding-a-theme*/}

Ovde, dugme *unutar* `Footer`-a prima drugačiju vrednost context-a (`"light"`) od dugmića izvan (`"dark"`).

<Sandpack>

```js
import { createContext, useContext } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  return (
    <ThemeContext value="dark">
      <Form />
    </ThemeContext>
  )
}

function Form() {
  return (
    <Panel title="Dobro došli">
      <Button>Registruj se</Button>
      <Button>Uloguj se</Button>
      <ThemeContext value="light">
        <Footer />
      </ThemeContext>
    </Panel>
  );
}

function Footer() {
  return (
    <footer>
      <Button>Podešavanja</Button>
    </footer>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      {title && <h1>{title}</h1>}
      {children}
    </section>
  )
}

function Button({ children }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className}>
      {children}
    </button>
  );
}
```

```css
footer {
  margin-top: 20px;
  border-top: 1px solid #aaa;
}

.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>

<Solution />

#### Automatsko ugnježdavanje naslova {/*automatically-nested-headings*/}

Možete "akumulirati" informacije kada ugnježdavate context provider-e. U ovom primeru, `Section` komponenta prati `LevelContext` koji specificira dubinu ugnježdavanja sekcije. Čita `LevelContext` iz roditeljske sekcije i pruža `LevelContext` uvećan za jedan svojoj deci. Kao rezultat, `Heading` komponenta može automatski da odluči koji od `<h1>`, `<h2>`, `<h3>`, ..., tag-ova da koristi na osnovu toga u koliko `Section` komponenata je ugnježdena.

Pročitajte [detaljno uputstvo](/learn/passing-data-deeply-with-context) za ovaj primer.

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading>Title</Heading>
      <Section>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext value={level + 1}>
        {children}
      </LevelContext>
    </section>
  );
}
```

```js src/Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 0:
      throw Error('Heading mora biti unutar Section-an!');
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Nepoznat nivo: ' + level);
  }
}
```

```js src/LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(0);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

<Solution />

</Recipes>

---

### Optimizovanje ponovnih rendera prilikom prosleđivanja objekata i funkcija {/*optimizing-re-renders-when-passing-objects-and-functions*/}

Kroz context možete proslediti bilo koju vrednost, uključujući objekte i funkcije.

```js [[2, 10, "{ currentUser, login }"]] 
function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  function login(response) {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }

  return (
    <AuthContext value={{ currentUser, login }}>
      <Page />
    </AuthContext>
  );
}
```

Ovde je <CodeStep step={2}>vrednost context-a</CodeStep> JavaScript objekat sa dva polja, od kojih je jedno funkcija. Kad god se `MyApp` ponovo renderuje (na primer, nakon promene rute), ovo će biti *drugačiji* objekat koji pokazuje na *drugačiju* funkciju, pa će React takođe trebati da ponovo renderuje sve komponente duboko u stablu koje pozivaju `useContext(AuthContext)`.

Ovo nije problem u manjim aplikacijama. Međutim, nema potrebe da ih ponovo renderujete ako se podaci, poput `currentUser`-a, nisu promenili. Da biste pomogli React-u da iskoristi tu činjenicu, možete obmotati `login` funkciju sa [`useCallback`](/reference/react/useCallback), a kreiranje objekta sa [`useMemo`](/reference/react/useMemo). Ovo je optimizacija performansi:

```js {6,9,11,14,17}
import { useCallback, useMemo } from 'react';

function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  const login = useCallback((response) => {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }, []);

  const contextValue = useMemo(() => ({
    currentUser,
    login
  }), [currentUser, login]);

  return (
    <AuthContext value={contextValue}>
      <Page />
    </AuthContext>
  );
}
```

Kao rezultat ove promene, čak iako `MyApp` treba ponovo da se renderuje, komponente koje pozivaju `useContext(AuthContext)` neće morati da se ponovo renderuju dok se `currentUser` ne promeni.

Pročitajte više o [`useMemo`](/reference/react/useMemo#skipping-re-rendering-of-components) i [`useCallback`](/reference/react/useCallback#skipping-re-rendering-of-components).

---

## Rešavanje problema {/*troubleshooting*/}

### Moja komponenta ne vidi vrednost iz provider-a {/*my-component-doesnt-see-the-value-from-my-provider*/}

Postoji par uobičajenih razloga da se ovo desi:

1. Renderujete `<SomeContext>` u istoj komponenti (ili ispod) u kojoj pozivate `useContext()`. Pomerite `<SomeContext>` *iznad i izvan* komponente koja poziva `useContext()`.
2. Možda ste zaboravili da obmotate vašu komponentu sa `<SomeContext>` ili ste je možda postavili u drugi deo stabla. Proverite da li je hijerarhija ispravna upotrebom [React DevTools-a](/learn/react-developer-tools).
3. Možda nailazite na problem sa alatima tokom izgradnje koji prouzrokuje da `SomeContext`, kako ga vidi komponenta koja ga pruža, i `SomeContext`, kako ga vidi komponenta koja ga čita, budu dva različita objekta. Ovo se može desiti ako koristite simboličke linkove, na primer. Možete verifikovati ovo tako što ćete im dodeliti globalne promenljive poput `window.SomeContext1` i `window.SomeContext2`, a onda u konzoli proveriti `window.SomeContext1 === window.SomeContext2`. Ako nisu jednaki, popravite taj problem na nivou alata za izgradnju.

### Uvek dobijam `undefined` iz mog context-a iako je default vrednost drugačija {/*i-am-always-getting-undefined-from-my-context-although-the-default-value-is-different*/}

Možda imate provider bez `value` u stablu:

```js {1,2}
// 🚩 Ne radi: nema value prop
<ThemeContext>
   <Button />
</ThemeContext>
```

Ako zaboravite specificirati `value`, to je kao da prosleđujete `value={undefined}`.

Možda ste greškom iskoristili drugo ime za prop:

```js {1,2}
// 🚩 Ne radi: prop treba da se zove "value"
<ThemeContext theme={theme}>
   <Button />
</ThemeContext>
```

U oba slučaja trebate videti upozorenje od React-a u konzoli. Da biste ovo popravili, nazovite prop `value`:

```js {1,2}
// ✅ Prosleđivanje value prop-a
<ThemeContext value={theme}>
   <Button />
</ThemeContext>
```

Primetite da se [default vrednost iz vašeg `createContext(defaultValue)` poziva](#specifying-a-fallback-default-value) koristi samo **ako uopšte nema odgovarajućeg provider-a iznad**. Ako postoji komponenta sa `<SomeContext value={undefined}>` negde u stablu roditelja, komponenta koja poziva `useContext(SomeContext)` *će* dobiti `undefined` kao vrednost context-a.
