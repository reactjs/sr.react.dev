---
title: Izdvajanje state logike u reducer
---

<Intro>

Komponente sa mnogo ažuriranja state-a koji se prostiru kroz mnogo event handler-a mogu postati preobimne. U tim slučajevima, možete grupisati svu logiku ažuriranja state-a izvan komponente u jednu funkciju koja se naziva _reducer._

</Intro>

<YouWillLearn>

- Šta je to reducer funkcija
- Kako da refaktorišete `useState` u `useReducer`
- Kada koristiti reducer
- Kako da pravilno napišete jedan

</YouWillLearn>

## Grupisati state logiku sa reducer-om {/*consolidate-state-logic-with-a-reducer*/}

Dok se vaše komponente komplikuju, na prvi pogled može biti teško uočiti sve različite načine ažuriranja state-a neke komponente. Na primer, `TaskApp` komponenta ispod sadrži niz `tasks` u state-u i koristi tri različita event handler-a za dodavanje, brisanje i izmenu zadataka:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, setTasks] = useState(initialTasks);

  function handleAddTask(text) {
    setTasks([
      ...tasks,
      {
        id: nextId++,
        text: text,
        done: false,
      },
    ]);
  }

  function handleChangeTask(task) {
    setTasks(
      tasks.map((t) => {
        if (t.id === task.id) {
          return task;
        } else {
          return t;
        }
      })
    );
  }

  function handleDeleteTask(taskId) {
    setTasks(tasks.filter((t) => t.id !== taskId));
  }

  return (
    <>
      <h1>Plan puta u Pragu</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: 'Poseti Kafkin muzej', done: true},
  {id: 1, text: 'Gledaj lutkarsku predstavu', done: false},
  {id: 2, text: 'Slikaj Lenonov zid', done: false},
];
```

```js src/AddTask.js hidden
import { useState } from 'react';

export default function AddTask({onAddTask}) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Dodaj zadatak"
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button
        onClick={() => {
          setText('');
          onAddTask(text);
        }}>
        Dodaj
      </button>
    </>
  );
}
```

```js src/TaskList.js hidden
import { useState } from 'react';

export default function TaskList({tasks, onChangeTask, onDeleteTask}) {
  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>
          <Task task={task} onChange={onChangeTask} onDelete={onDeleteTask} />
        </li>
      ))}
    </ul>
  );
}

function Task({task, onChange, onDelete}) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={(e) => {
            onChange({
              ...task,
              text: e.target.value,
            });
          }}
        />
        <button onClick={() => setIsEditing(false)}>Sačuvaj</button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>Izmeni</button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={(e) => {
          onChange({
            ...task,
            done: e.target.checked,
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>Obriši</button>
    </label>
  );
}
```

```css
button {
  margin: 5px;
}
li {
  list-style-type: none;
}
ul,
li {
  margin: 0;
  padding: 0;
}
```

</Sandpack>

Svaki od ovih event handler-a poziva `setTasks` kako bi ažurirao state. Kako komponenta raste, raste i količina state logike u njoj. Kako biste smanjili kompleksnost i čuvali svu logiku na jednom, lako dostupnom mestu, možete pomeriti tu state logiku u posebnu funkciju izvan vaše komponente, funkciju **pod imenom "reducer"**.

Reducer-i predstavljaju drugi način za upravljanje state-om. Možete se migrirati od `useState` do `useReducer` u tri koraka:

1. **Prelazak** sa postavljanja state-a na otpremanje akcija.
2. **Pisanje** reducer funkcije.
3. **Korišćenje** reducer-a iz vaše komponente.

### Korak 1: Prelazak sa postavljanja state-a na otpremanje akcija {/*step-1-move-from-setting-state-to-dispatching-actions*/}

Vaši event handler-i trenutno specificiraju _šta raditi_ postavljanjem state-a:

```js
function handleAddTask(text) {
  setTasks([
    ...tasks,
    {
      id: nextId++,
      text: text,
      done: false,
    },
  ]);
}

function handleChangeTask(task) {
  setTasks(
    tasks.map((t) => {
      if (t.id === task.id) {
        return task;
      } else {
        return t;
      }
    })
  );
}

function handleDeleteTask(taskId) {
  setTasks(tasks.filter((t) => t.id !== taskId));
}
```

Uklonite svu logiku postavljanja state-a. Ono što vam ostaje su tri event handler-a:

- `handleAddTask(text)` se poziva kada korisnik klikne "Dodaj".
- `handleChangeTask(task)` se poziva kada korisnik štiklira zadatak ili klikne "Sačuvaj".
- `handleDeleteTask(taskId)` se poziva kada korisnik klikne "Obriši".

Upravljanje state-om pomoću reducer-a se malo razlikuje od direktnog postavljanja state-a. Umesto da postavljanjem state-a React-u kažete "šta da radi", možete specificirati "šta je korisnik upravo uradio" otpremanjem "akcija" iz event handler-a. (Logika ažuriranja state-a će živeti negde drugde!) Znači, umesto "postavljanja `tasks` niza" kroz event handler, otpremate akciju "added/changed/deleted". Ovo bolje opisuje nameru korisnika.

```js
function handleAddTask(text) {
  dispatch({
    type: 'added',
    id: nextId++,
    text: text,
  });
}

function handleChangeTask(task) {
  dispatch({
    type: 'changed',
    task: task,
  });
}

function handleDeleteTask(taskId) {
  dispatch({
    type: 'deleted',
    id: taskId,
  });
}
```

Objekat koji prosleđujete u `dispatch` se naziva "akcija":

```js {3-7}
function handleDeleteTask(taskId) {
  dispatch(
    // "action" objekat:
    {
      type: 'deleted',
      id: taskId,
    }
  );
}
```

To je običan JavaScript objekat. Možete odlučiti šta da smestite u njega, ali generalno bi trebao da sadrži minimum informacija o onome _što se desilo_. (Dodaćete `dispatch` funkciju u narednom koraku.)

<Note>

Action objekat može imati bilo kakav oblik.

Po konvenciji, uobičajeno je da mu date string `type` koji opisuje šta se desilo i prosledite ostale informacije u drugim poljima. `type` je jedinstven za komponentu, pa bi u ovom primeru i `'added'` i `'added_task'` bili u redu. Odaberite ime koje govori šta se desilo!

```js
dispatch({
  // jedinstveno za komponentu
  type: 'what_happened',
  // ostala polja idu ovde
});
```

</Note>

### Korak 2: Pisanje reducer funkcije {/*step-2-write-a-reducer-function*/}

Reducer funkcija je mesto gde ćete staviti vašu state logiku. Prima dva argumenta, trenutni state i action objekat, a vraća naredni state:

```js
function yourReducer(state, action) {
  // vraća naredni state koji će React postaviti
}
```

React će postaviti state na ono što vratite iz reducer-a.

Da biste pomerili logiku postavljanja state-a iz event handler-a u reducer funkciju u ovom primeru, uradićete sledeće:

1. Deklarisati trenutni state (`tasks`) kao prvi argument.
2. Deklarisati `action` objekat kao drugi argument.
3. Vratiti _naredni_ state iz reducer-a (na šta će React postaviti state).

Ovde je sva logika postavljanja state-a migrirana u reducer funkciju:

```js
function tasksReducer(tasks, action) {
  if (action.type === 'added') {
    return [
      ...tasks,
      {
        id: action.id,
        text: action.text,
        done: false,
      },
    ];
  } else if (action.type === 'changed') {
    return tasks.map((t) => {
      if (t.id === action.task.id) {
        return action.task;
      } else {
        return t;
      }
    });
  } else if (action.type === 'deleted') {
    return tasks.filter((t) => t.id !== action.id);
  } else {
    throw Error('Nepoznata akcija: ' + action.type);
  }
}
```

Pošto reducer funkcija prima state (`tasks`) kao argument, možete ga **deklarisati izvan vaše komponente**. Ovo smanjuje nivo uvlačenja i čini kod čitljivijim.

<Note>

Kod iznad koristi if/else iskaze, ali je konvencija da se koriste [switch iskazi](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/switch) unutar reducer-a. Rezultat je isti, ali na prvi pogled switch iskazi mogu biti čitljiviji.

Mi ćemo ih koristiti svuda kroz ostatak dokumentacije:

```js
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Nepoznata akcija: ' + action.type);
    }
  }
}
```

Preporučujemo da obmotate `case` blok u `{` i `}` vitičaste zagrade kako se promenljive deklarisane u različitim `case`-ovima ne bi preplitale. Takođe, `case` se uglavnom završava sa `return`. Ako zaboravite `return`, kod će "propasti" u naredni `case`, što može dovesti do grešaka!

Ako još uvek niste navikli na switch iskaze, upotreba if/else je potpuno u redu.

</Note>

<DeepDive>

#### Zašto se reducer-i tako zovu? {/*why-are-reducers-called-this-way*/}

Iako reducer-i mogu da "smanje" količinu koda unutar komponente, zapravo su nazvani po [`reduce()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) operaciji koju možete izvršiti nad nizovima.

`reduce()` operacija vam omogućava da uzmete niz i "akumulirate" jednu vrednost:

```
const arr = [1, 2, 3, 4, 5];
const sum = arr.reduce(
  (result, number) => result + number
); // 1 + 2 + 3 + 4 + 5
```

Funkcija koju prosleđujete u `reduce` se naziva "reducer". Uzima _rezultat do sada_ i _trenutni element_, a vraća _naredni rezultat_. React reducer-i su primer iste ideje: uzimaju _state koji imate do sad_ i _akciju_, a vraćaju _naredni state_. Na ovaj način, oni akumuliraju akcije tokom vremena u state.

Možete koristiti `reduce()` metodu nad `initialState` i nizom `actions` da izračunate konačni state prosleđivanjem vaše reducer funkcije:

<Sandpack>

```js src/index.js active
import tasksReducer from './tasksReducer.js';

let initialState = [];
let actions = [
  {type: 'added', id: 1, text: 'Poseti Kafkin muzej'},
  {type: 'added', id: 2, text: 'Gledaj lutkarsku predstavu'},
  {type: 'deleted', id: 1},
  {type: 'added', id: 3, text: 'Slikaj Lenonov zid'},
];

let finalState = actions.reduce(tasksReducer, initialState);

const output = document.getElementById('output');
output.textContent = JSON.stringify(finalState, null, 2);
```

```js src/tasksReducer.js
export default function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Nepoznata akcija: ' + action.type);
    }
  }
}
```

```html public/index.html
<pre id="output"></pre>
```

</Sandpack>

Verovatno ovo nećete morati da radite samostalno, ali to je slično onome što React radi!

</DeepDive>

### Korak 3: Korišćenje reducer-a iz vaše komponente {/*step-3-use-the-reducer-from-your-component*/}

Konačno, trebate zakačiti `tasksReducer` u vašu komponentu. Import-ujte `useReducer` Hook iz React-a:

```js
import { useReducer } from 'react';
```

Onda možete zameniti `useState`:

```js
const [tasks, setTasks] = useState(initialTasks);
```

sa `useReducer`:

```js
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

`useReducer` Hook liči na `useState`—morate proslediti inicijalni state, a on vraća stateful vrednost i način da postavite state (u ovom slučaju, dispatch funkciju). Ali, malo se razlikuje.

`useReducer` Hook prima dva argumenta:

1. Reducer funkciju
2. Inicijalni state

I vraća:

1. Stateful vrednost
2. Dispatch funkciju (za "otpremanje" korisničkih akcija u reducer)

Sad je potpuno povezan! Ovde je reducer deklarisan na dnu fajla komponente:

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>Plan puta u Pragu</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Nepoznata akcija: ' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: 'Poseti Kafkin muzej', done: true},
  {id: 1, text: 'Gledaj lutkarsku predstavu', done: false},
  {id: 2, text: 'Slikaj Lenonov zid', done: false},
];
```

```js src/AddTask.js hidden
import { useState } from 'react';

export default function AddTask({onAddTask}) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Dodaj zadatak"
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button
        onClick={() => {
          setText('');
          onAddTask(text);
        }}>
        Dodaj
      </button>
    </>
  );
}
```

```js src/TaskList.js hidden
import { useState } from 'react';

export default function TaskList({tasks, onChangeTask, onDeleteTask}) {
  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>
          <Task task={task} onChange={onChangeTask} onDelete={onDeleteTask} />
        </li>
      ))}
    </ul>
  );
}

function Task({task, onChange, onDelete}) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={(e) => {
            onChange({
              ...task,
              text: e.target.value,
            });
          }}
        />
        <button onClick={() => setIsEditing(false)}>Sačuvaj</button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>Izmeni</button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={(e) => {
          onChange({
            ...task,
            done: e.target.checked,
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>Obriši</button>
    </label>
  );
}
```

```css
button {
  margin: 5px;
}
li {
  list-style-type: none;
}
ul,
li {
  margin: 0;
  padding: 0;
}
```

</Sandpack>

Ako želite, možete čak reducer pomeriti i u drugi fajl:

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import tasksReducer from './tasksReducer.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>Plan puta u Pragu</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: 'Poseti Kafkin muzej', done: true},
  {id: 1, text: 'Gledaj lutkarsku predstavu', done: false},
  {id: 2, text: 'Slikaj Lenonov zid', done: false},
];
```

```js src/tasksReducer.js
export default function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Nepoznata akcija: ' + action.type);
    }
  }
}
```

```js src/AddTask.js hidden
import { useState } from 'react';

export default function AddTask({onAddTask}) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Dodaj zadatak"
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button
        onClick={() => {
          setText('');
          onAddTask(text);
        }}>
        Dodaj
      </button>
    </>
  );
}
```

```js src/TaskList.js hidden
import { useState } from 'react';

export default function TaskList({tasks, onChangeTask, onDeleteTask}) {
  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>
          <Task task={task} onChange={onChangeTask} onDelete={onDeleteTask} />
        </li>
      ))}
    </ul>
  );
}

function Task({task, onChange, onDelete}) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={(e) => {
            onChange({
              ...task,
              text: e.target.value,
            });
          }}
        />
        <button onClick={() => setIsEditing(false)}>Sačuvaj</button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>Izmeni</button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={(e) => {
          onChange({
            ...task,
            done: e.target.checked,
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>Obriši</button>
    </label>
  );
}
```

```css
button {
  margin: 5px;
}
li {
  list-style-type: none;
}
ul,
li {
  margin: 0;
  padding: 0;
}
```

</Sandpack>

Logika komponente može biti čitljivija kada ovako razdvojite zaduženja. Sada event handler-i samo specificiraju _šta se desilo_ otpremanjem akcija, a reducer funkcija određuje _kako se state ažurira_ kao odgovor na njih.

## Poređenje `useState` i `useReducer` {/*comparing-usestate-and-usereducer*/}

Reducer-i imaju i mane! Evo par načina da ih poredite:

- **Količina koda:** Uopšteno, sa `useState`-om trebate pisati manje koda unapred. Sa `useReducer`-om morate da pišete i reducer funkciju _i_ akcije za otpremanje. Međutim, `useReducer` može pomoći da smanjite kod ako mnogo event handler-a menja state na sličan način.
- **Čitljivost:** Lako je čitati `useState` kada je ažuriranje state-a jednostavno. Kada state postane komplikovaniji, može preopteretiti kod vaš komponente i učiniti ga težim za razumevanje. U ovom slučaju, `useReducer` vam omogućava da jasno odvojite _kako_ u logici ažuriranja od _šta se desilo_ iz event handler-a.
- **Debug-ovanje:** Kada imate bug sa `useState`-om, može biti teško da otkrijete _gde_ je state pogrešno postavljen i _zašto_. Sa `useReducer`-om možete dodati console log u vaš reducer da biste videli svako ažuriranje state-a, kao i _zašto_ se desilo (zbog kog `action`-a). Ako je svaki `action` ispravan, znaćete da je greška u samoj logici reducer-a. Međutim, morate proći kroz više koda u poređenju sa `useState`.
- **Testiranje:** Reducer je čista funkcija koja ne zavisi od vaše komponente. To znači da je možete export-ovati i testirati izolovano. Iako je obično najbolje testirati komponente u realističnijem okruženju, za kompleksniju logiku ažuriranja state-a može biti korisno da se uverite da vaš reducer vraća određeni state za određeni inicijalni state i akciju.
- **Lična preferenca:** Neki ljudi vole reducer-e, ostali ne vole. To je u redu. Stvar je preferenci. Uvek možete menjati između `useState` i `useReducer`: jednaki su!

Preporučujemo da koristite reducer ako često nailazite na bug-ove zbog neispravnog ažuriranja state-a u nekoj komponenti i želite da uvedete veću strukturu u njen kod. Ne morate koristiti reducer-e za sve: slobodno kombinujte! Možete čak koristiti `useState` i `useReducer` u istoj komponenti.

## Pravilno pisanje reducer-a {/*writing-reducers-well*/}

Zapamtite ova dva saveta kada pišete reducer-e:

- **Reducer-i moraju biti čisti.** Slično kao i za [state updater funkcije](/learn/queueing-a-series-of-state-updates), reducer-i se izvršavaju tokom rendera! (Akcije su u redu čekanja pre narednog rendera.) Ovo znači da reducer-i [moraju biti čisti](/learn/keeping-components-pure)—isti input-i uvek moraju da vrate isti rezultat. Ne bi trebali da šalju zahteve, zakazuju timeout-e ili da izvršavaju neke propratne efekte (operacije koje utiču na stvari van komponente). Trebaju da ažuriraju [objekte](/learn/updating-objects-in-state) i [nizove](/learn/updating-arrays-in-state) bez mutacija.
- **Svaka akcija opisuje jednu korisničku interakciju, čak iako to dovodi do višestrukih promena u podacima.** Na primer, ako korisnik klikne "Resetuj" u formi sa pet polja kojom upravlja reducer, ima više smisla da se otpremi jedna `reset_form` akcija umesto pet različitih `set_field` akcija. Ako logujete svaku akciju u reducer-u, taj log bi trebao biti dovoljno jasan da biste rekonstruisali redosled interakcija ili odgovora koji su se desili. Ovo pomaže prilikom debug-ovanja!

## Pisanje konciznih reducer-a sa Immer-om {/*writing-concise-reducers-with-immer*/}

Kao i kod [ažuriranja objekata](/learn/updating-objects-in-state#write-concise-update-logic-with-immer) i [nizova](/learn/updating-arrays-in-state#write-concise-update-logic-with-immer) u običnom state-u, možete koristiti Immer biblioteku da učinite reducer-e konciznijim. Ovde, [`useImmerReducer`](https://github.com/immerjs/use-immer#useimmerreducer) vam omogućava da mutirate state sa `push` ili `arr[i] =` dodelom:

<Sandpack>

```js src/App.js
import { useImmerReducer } from 'use-immer';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

function tasksReducer(draft, action) {
  switch (action.type) {
    case 'added': {
      draft.push({
        id: action.id,
        text: action.text,
        done: false,
      });
      break;
    }
    case 'changed': {
      const index = draft.findIndex((t) => t.id === action.task.id);
      draft[index] = action.task;
      break;
    }
    case 'deleted': {
      return draft.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Nepoznata akcija: ' + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useImmerReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>Plan puta u Pragu</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: 'Poseti Kafkin muzej', done: true},
  {id: 1, text: 'Gledaj lutkarsku predstavu', done: false},
  {id: 2, text: 'Slikaj Lenonov zid', done: false},
];
```

```js src/AddTask.js hidden
import { useState } from 'react';

export default function AddTask({onAddTask}) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Dodaj zadatak"
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button
        onClick={() => {
          setText('');
          onAddTask(text);
        }}>
        Dodaj
      </button>
    </>
  );
}
```

```js src/TaskList.js hidden
import { useState } from 'react';

export default function TaskList({tasks, onChangeTask, onDeleteTask}) {
  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>
          <Task task={task} onChange={onChangeTask} onDelete={onDeleteTask} />
        </li>
      ))}
    </ul>
  );
}

function Task({task, onChange, onDelete}) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={(e) => {
            onChange({
              ...task,
              text: e.target.value,
            });
          }}
        />
        <button onClick={() => setIsEditing(false)}>Sačuvaj</button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>Izmeni</button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={(e) => {
          onChange({
            ...task,
            done: e.target.checked,
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>Obriši</button>
    </label>
  );
}
```

```css
button {
  margin: 5px;
}
li {
  list-style-type: none;
}
ul,
li {
  margin: 0;
  padding: 0;
}
```

```json package.json
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "use-immer": "0.5.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

Reducer-i moraju biti čisti, tako da ne bi trebali da mutiraju state. Ali, Immer vam pruža poseban `draft` objekat koji možete sigurno mutirati. Ispod haube, Immer će napraviti kopiju vašeg state-a sa promenama koje ste napravili nad `draft`-om. Zato reducer-i koje koristite uz pomoć `useImmerReducer`-a mogu da mutiraju svoj prvi argument i ne moraju da vrate state.

<Recap>

- Za konvertovanje iz `useState` u `useReducer`:
  1. Otpremite akcije iz event handler-a.
  2. Napišite reducer funkciju koja vraća naredni state za zadati state i akciju.
  3. Zamenite `useState` sa `useReducer`.
- Reducer-i zahtevaju da napišete više koda, ali pomažu u debug-ovanju i testiranju.
- Reducer-i moraju biti čisti.
- Svaka akcija opisuje jednu korisničku interakciju.
- Koristite Immer ako želite da pišete reducer-e u stilu mutacija.

</Recap>

<Challenges>

#### Otpremiti akcije iz event handler-a {/*dispatch-actions-from-event-handlers*/}

Trenutno, event handler-i u `ContactList.js` i `Chat.js` imaju `// TODO` komentare. Zbog toga pisanje u input ne radi, a klik na dugmiće ne menja izabranog primaoca.

Zamenite ova dva `// TODO`-a sa kodom koji poziva `dispatch` za odgovarajuće akcije. Da biste videli očekivani oblik i tipove akcija, proverite reducer u `messengerReducer.js`. Reducer je već napisan, pa ga ne morate menjati. Samo je potrebno da otpremite akcije iz `ContactList.js` i `Chat.js`.

<Hint>

`dispatch` funkcija je već dostupna u obe komponente jer je prosleđena kao prop. Tako da trebate pozvati `dispatch` sa odgovarajućim action objektom.

Da biste proverili oblik action objekta, možete pogledati reducer i videti koja `action` polja očekuje. Na primer, `changed_selection` case u reducer-u izgleda ovako:

```js
case 'changed_selection': {
  return {
    ...state,
    selectedId: action.contactId
  };
}
```

Ovo znači da vaš action objekat treba da ima `type: 'changed_selection'`. Takođe, vidite da je upotrebljen `action.contactId`, tako da je potrebno uključiti `contactId` polje u vašu akciju.

</Hint>

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.message;
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js
export const initialState = {
  selectedId: 0,
  message: 'Zdravo',
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
        message: '',
      };
    }
    case 'edited_message': {
      return {
        ...state,
        message: action.message,
      };
    }
    default: {
      throw Error('Nepoznata akcija: ' + action.type);
    }
  }
}
```

```js src/ContactList.js
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                // TODO: otpremi changed_selection
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'Piši korisniku ' + contact.name}
        onChange={(e) => {
          // TODO: otpremi edited_message
          // (Pročitaj input vrednost iz e.target.value)
        }}
      />
      <br />
      <button>Pošalji na {contact.email}</button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

<Solution>

Iz koda reducer-a možete zaključiti da akcije trebaju izgledati ovako:

```js
// Kada korisnik klikne "Alice"
dispatch({
  type: 'changed_selection',
  contactId: 1,
});

// Kada korisnik unese "Zdravo!"
dispatch({
  type: 'edited_message',
  message: 'Zdravo!',
});
```

Evo ažuriranog primera sa odgovarajućim otpremljenim porukama:

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.message;
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js
export const initialState = {
  selectedId: 0,
  message: 'Zdravo',
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
        message: '',
      };
    }
    case 'edited_message': {
      return {
        ...state,
        message: action.message,
      };
    }
    default: {
      throw Error('Nepoznata akcija: ' + action.type);
    }
  }
}
```

```js src/ContactList.js
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                dispatch({
                  type: 'changed_selection',
                  contactId: contact.id,
                });
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'Piši korisniku ' + contact.name}
        onChange={(e) => {
          dispatch({
            type: 'edited_message',
            message: e.target.value,
          });
        }}
      />
      <br />
      <button>Pošalji na {contact.email}</button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

</Solution>

#### Očistiti input pri slanju poruke {/*clear-the-input-on-sending-a-message*/}

Trenutno, klik na "Pošalji" ne radi ništa. Dodajte event handler na "Pošalji" dugme koji radi sledeće:

1. Prikazuje `alert` sa porukom i email-om primaoca.
2. Čisti input za poruku.

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.message;
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js
export const initialState = {
  selectedId: 0,
  message: 'Zdravo',
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
        message: '',
      };
    }
    case 'edited_message': {
      return {
        ...state,
        message: action.message,
      };
    }
    default: {
      throw Error('Nepoznata akcija: ' + action.type);
    }
  }
}
```

```js src/ContactList.js
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                dispatch({
                  type: 'changed_selection',
                  contactId: contact.id,
                });
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js active
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'Piši korisniku ' + contact.name}
        onChange={(e) => {
          dispatch({
            type: 'edited_message',
            message: e.target.value,
          });
        }}
      />
      <br />
      <button>Pošalji na {contact.email}</button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

<Solution>

Postoji par načina kako biste ovo mogli uraditi u event handler-u "Pošalji" dugmeta. Jedan pristup je da prikažete alert, a onda otpremite `edited_message` akciju sa praznom `message` vrednošću:

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.message;
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js
export const initialState = {
  selectedId: 0,
  message: 'Zdravo',
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
        message: '',
      };
    }
    case 'edited_message': {
      return {
        ...state,
        message: action.message,
      };
    }
    default: {
      throw Error('Nepoznata akcija: ' + action.type);
    }
  }
}
```

```js src/ContactList.js
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                dispatch({
                  type: 'changed_selection',
                  contactId: contact.id,
                });
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js active
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'Piši korisniku ' + contact.name}
        onChange={(e) => {
          dispatch({
            type: 'edited_message',
            message: e.target.value,
          });
        }}
      />
      <br />
      <button
        onClick={() => {
          alert(`Slanje "${message}" na ${contact.email}`);
          dispatch({
            type: 'edited_message',
            message: '',
          });
        }}>
        Pošalji na {contact.email}
      </button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

Ovo radi i čisti input kada kliknete "Pošalji".

Međutim, _iz korisničke perspektive_, slanje poruke je akcija drugačija od izmene poruke. Da biste to prikazali, možete napraviti _novu_ akciju pod imenom `sent_message` i obraditi je posebno u reducer-u:

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.message;
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js active
export const initialState = {
  selectedId: 0,
  message: 'Zdravo',
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
        message: '',
      };
    }
    case 'edited_message': {
      return {
        ...state,
        message: action.message,
      };
    }
    case 'sent_message': {
      return {
        ...state,
        message: '',
      };
    }
    default: {
      throw Error('Nepoznata akcija: ' + action.type);
    }
  }
}
```

```js src/ContactList.js
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                dispatch({
                  type: 'changed_selection',
                  contactId: contact.id,
                });
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js active
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'Piši korisniku ' + contact.name}
        onChange={(e) => {
          dispatch({
            type: 'edited_message',
            message: e.target.value,
          });
        }}
      />
      <br />
      <button
        onClick={() => {
          alert(`Slanje "${message}" na ${contact.email}`);
          dispatch({
            type: 'sent_message',
          });
        }}>
        Pošalji na {contact.email}
      </button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

Rezultujuće ponašanje je isto. Ali, imajte na umu da tipovi akcija idealno trebaju opisivati "šta je korisnik uradio", a ne "kako želite da se state promeni". Ovo olakšava kasnije dodavanje novih funkcionalnosti.

U oba pristupa je važno da **ne** stavite `alert` unutar reducer-a. Reducer treba biti čista funkcija--jedino treba da računa naredni state. Ne bi trebao ništa da "radi", uključujući i prikazivanje poruka korisniku. To se treba desiti u event handler-u. (Kako bi vam pomogao da primetite ovakve greške, React će pozvati reducer-e više puta u Strict Mode-u. Zbog ovoga će se alert izvršiti dvaput ako ga postavite u reducer.)

</Solution>

#### Povratiti input vrednosti prilikom promene tabova {/*restore-input-values-when-switching-between-tabs*/}

U ovom primeru, promena primaoca uvek čisti tekst u input-u:

```js
case 'changed_selection': {
  return {
    ...state,
    selectedId: action.contactId,
    message: '' // Čisti input
  };
```

Ovo se dešava jer ne želite da delite jednu draft poruku između više primalaca. Ali, bilo bi bolje ako vaša aplikacija "pamti" draft za svaki kontakt i može da ga povrati prilikom promene kontakta.

Vaš zadatak je da promenite strukturu state-a kako biste upamtili draft poruke za _svaki kontakt_. Trebate napraviti par izmena u reducer-u, inicijalnom state-u i komponentama.

<Hint>

Možete strukturirati vaš state ovako:

```js
export const initialState = {
  selectedId: 0,
  messages: {
    0: 'Zdravo, Taylor', // Draft za contactId = 0
    1: 'Zdravo, Alice', // Draft za contactId = 1
  },
};
```

Sintaksa za `[key]: value` [izračunato polje](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#computed_property_names) vam može pomoći da ažurirate `messages` objekat:

```js
{
  ...state.messages,
  [id]: message
}
```

</Hint>

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.message;
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js
export const initialState = {
  selectedId: 0,
  message: 'Zdravo',
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
        message: '',
      };
    }
    case 'edited_message': {
      return {
        ...state,
        message: action.message,
      };
    }
    case 'sent_message': {
      return {
        ...state,
        message: '',
      };
    }
    default: {
      throw Error('Nepoznata akcija: ' + action.type);
    }
  }
}
```

```js src/ContactList.js
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                dispatch({
                  type: 'changed_selection',
                  contactId: contact.id,
                });
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'Piši korisniku ' + contact.name}
        onChange={(e) => {
          dispatch({
            type: 'edited_message',
            message: e.target.value,
          });
        }}
      />
      <br />
      <button
        onClick={() => {
          alert(`Slanje "${message}" na ${contact.email}`);
          dispatch({
            type: 'sent_message',
          });
        }}>
        Pošalji na {contact.email}
      </button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

<Solution>

Morate da ažurirate reducer da čuva i ažurira draft poruku po kontaktu:

```js
// Kada se input promeni
case 'edited_message': {
  return {
    // Čuvajte ostale state-ove
    ...state,
    messages: {
      // Čuvajte poruke za druge kontakte
      ...state.messages,
      // Ali promenite poruku za odabrani kontakt
      [state.selectedId]: action.message
    }
  };
}
```

Izmenićete i `Messenger` komponentu da čita poruku za trenutno odabrani kontakt:

```js
const message = state.messages[state.selectedId];
```

Evo celokupnog rešenja:

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.messages[state.selectedId];
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js
export const initialState = {
  selectedId: 0,
  messages: {
    0: 'Zdravo, Taylor',
    1: 'Zdravo, Alice',
    2: 'Zdravo, Bob',
  },
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
      };
    }
    case 'edited_message': {
      return {
        ...state,
        messages: {
          ...state.messages,
          [state.selectedId]: action.message,
        },
      };
    }
    case 'sent_message': {
      return {
        ...state,
        messages: {
          ...state.messages,
          [state.selectedId]: '',
        },
      };
    }
    default: {
      throw Error('Nepoznata akcija: ' + action.type);
    }
  }
}
```

```js src/ContactList.js
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                dispatch({
                  type: 'changed_selection',
                  contactId: contact.id,
                });
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'Piši korisniku ' + contact.name}
        onChange={(e) => {
          dispatch({
            type: 'edited_message',
            message: e.target.value,
          });
        }}
      />
      <br />
      <button
        onClick={() => {
          alert(`Slanje "${message}" na ${contact.email}`);
          dispatch({
            type: 'sent_message',
          });
        }}>
        Pošalji na {contact.email}
      </button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

Dodatno, niste morali da menjate nijedan event handler da biste implementirali ovo ponašanje. Bez reducer-a, morali biste da menjate svaki event handler koji ažurira state.

</Solution>

#### Implementirati `useReducer` od nule {/*implement-usereducer-from-scratch*/}

U prethodnim primerima, import-ovali ste `useReducer` Hook iz React-a. Ovog puta ćete implementirati _`useReducer` Hook samostalno_! Ovde je stub koji vam pomaže da počnete. Ne bi trebalo biti više od 10 linija koda.

Da biste testirali izmene, pokušajte da unesete input ili izaberete kontakt.

<Hint>

Evo detaljnije skice implementacije:

```js
export function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);

  function dispatch(action) {
    // ???
  }

  return [state, dispatch];
}
```

Prisetite se da reducer funkcija prima dva argumenta--trenutni state i action objekat--i vraća naredni state. Šta bi vaša `dispatch` implementacija trebala da radi sa njima?

</Hint>

<Sandpack>

```js src/App.js
import { useReducer } from './MyReact.js';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.messages[state.selectedId];
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js
export const initialState = {
  selectedId: 0,
  messages: {
    0: 'Zdravo, Taylor',
    1: 'Zdravo, Alice',
    2: 'Zdravo, Bob',
  },
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
      };
    }
    case 'edited_message': {
      return {
        ...state,
        messages: {
          ...state.messages,
          [state.selectedId]: action.message,
        },
      };
    }
    case 'sent_message': {
      return {
        ...state,
        messages: {
          ...state.messages,
          [state.selectedId]: '',
        },
      };
    }
    default: {
      throw Error('Nepoznata akcija: ' + action.type);
    }
  }
}
```

```js src/MyReact.js active
import { useState } from 'react';

export function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);

  // ???

  return [state, dispatch];
}
```

```js src/ContactList.js hidden
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                dispatch({
                  type: 'changed_selection',
                  contactId: contact.id,
                });
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js hidden
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'Piši korisniku ' + contact.name}
        onChange={(e) => {
          dispatch({
            type: 'edited_message',
            message: e.target.value,
          });
        }}
      />
      <br />
      <button
        onClick={() => {
          alert(`Slanje "${message}" na ${contact.email}`);
          dispatch({
            type: 'sent_message',
          });
        }}>
        Pošalji na {contact.email}
      </button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

<Solution>

Otpremanje akcije poziva reducer sa trenutnim state-om i akcijom, a postavlja rezultat kao naredni state. Evo kako to izgleda u kodu:

<Sandpack>

```js src/App.js
import { useReducer } from './MyReact.js';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.messages[state.selectedId];
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js
export const initialState = {
  selectedId: 0,
  messages: {
    0: 'Zdravo, Taylor',
    1: 'Zdravo, Alice',
    2: 'Zdravo, Bob',
  },
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
      };
    }
    case 'edited_message': {
      return {
        ...state,
        messages: {
          ...state.messages,
          [state.selectedId]: action.message,
        },
      };
    }
    case 'sent_message': {
      return {
        ...state,
        messages: {
          ...state.messages,
          [state.selectedId]: '',
        },
      };
    }
    default: {
      throw Error('Nepoznata akcija: ' + action.type);
    }
  }
}
```

```js src/MyReact.js active
import { useState } from 'react';

export function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);

  function dispatch(action) {
    const nextState = reducer(state, action);
    setState(nextState);
  }

  return [state, dispatch];
}
```

```js src/ContactList.js hidden
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                dispatch({
                  type: 'changed_selection',
                  contactId: contact.id,
                });
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js hidden
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'Piši korisniku ' + contact.name}
        onChange={(e) => {
          dispatch({
            type: 'edited_message',
            message: e.target.value,
          });
        }}
      />
      <br />
      <button
        onClick={() => {
          alert(`Slanje "${message}" na ${contact.email}`);
          dispatch({
            type: 'sent_message',
          });
        }}>
        Pošalji na {contact.email}
      </button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

Iako je nebitno u mnogim slučajevima, tačnija implementacija izgleda ovako:

```js
function dispatch(action) {
  setState((s) => reducer(s, action));
}
```

Ovo je zbog toga što su otpremljene akcije u redu čekanja pre narednog rendera, [slično kao i updater funkcije](/learn/queueing-a-series-of-state-updates).

</Solution>

</Challenges>
