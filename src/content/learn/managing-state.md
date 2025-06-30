---
title: Upravljanje state-om
---

<Intro>

Povećavanjem aplikacije, važno je da budete svesni kako vam je state organizovan i kako podaci teku kroz komponente. Suvišan ili dupliran state je čest uzrok bug-ova. U ovom poglavlju, naučićete kako da pravilno strukturirate state, kako da logika ažuriranja state-a bude održiva i kako da delite state između udaljenih komponenata.

</Intro>

<YouWillLearn isChapter={true}>

* [Kako da razmišljate o UI promenama kao promenama stanja](/learn/reacting-to-input-with-state)
* [Kako da pravilno strukturirate state](/learn/choosing-the-state-structure)
* [Kako da "podignete state" da bi ga delili između komponenata](/learn/sharing-state-between-components)
* [Kako da kontrolišete da li se state čuva ili resetuje](/learn/preserving-and-resetting-state)
* [Kako da grupišete kompleksnu state logiku u funkciju](/learn/extracting-state-logic-into-a-reducer)
* [Kako da prosledite informaciju bez "prop drilling-a"](/learn/passing-data-deeply-with-context)
* [Kako da skalirate upravljanje state-a dok aplikacija raste](/learn/scaling-up-with-reducer-and-context)

</YouWillLearn>

## Reagovanje na input pomoću stanja {/*reacting-to-input-with-state*/}

Sa React-om, nećete direktno u kodu menjati UI. Na primer, nećete pisati komande poput "onemogući dugme", "omogući dugme", "prikaži uspešnu poruku", itd. Umesto toga, opisaćete kakav UI želite da vidite za različita vizuelna stanja vaše komponente ("inicijalno stanje", "stanje pisanja", "uspešno stanje"), a onda ćete pokrenuti promene stanja kao odgovor na korisnički input. Ovo je slično onome kako dizajneri razmišljaju o UI-u.

Ovde je forma za kviz napravljena pomoću React-a. Primetite kako koristi `status` state promenljivu da odluči da li da omogući submit dugme i da li da prikaže uspešnu poruku.

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [status, setStatus] = useState('typing');

  if (status === 'success') {
    return <h1>To je tačno!</h1>
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    try {
      await submitForm(answer);
      setStatus('success');
    } catch (err) {
      setStatus('typing');
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <>
      <h2>Kviz gradova</h2>
      <p>
        U kom gradu je bilbord koji pretvara vazduh u pijaću vodu?
      </p>
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer}
          onChange={handleTextareaChange}
          disabled={status === 'submitting'}
        />
        <br />
        <button disabled={
          answer.length === 0 ||
          status === 'submitting'
        }>
          Submit
        </button>
        {error !== null &&
          <p className="Error">
            {error.message}
          </p>
        }
      </form>
    </>
  );
}

function submitForm(answer) {
  // Pretvaraj se da koristiš mrežni poziv.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== 'lima'
      if (shouldError) {
        reject(new Error('Dobar pokušaj, ali pogrešan odgovor. Probaj ponovo!'));
      } else {
        resolve();
      }
    }, 1500);
  });
}
```

```css
.Error { color: red; }
```

</Sandpack>

<LearnMore path="/learn/reacting-to-input-with-state">

Pročitajte **[Reagovanje na input pomoću stanja](/learn/reacting-to-input-with-state)** da naučite kako da pristupite interakcijama iz perspektive stanja.

</LearnMore>

## Odabir strukture state-a {/*choosing-the-state-structure*/}

Pravilno strukturiranje state-a može napraviti razliku između komponente koju je lako menjati i debug-ovati, i one koja je stalan izvor bug-ova. Najbitniji princip je da state ne bi trebao da sadrži suvišne i duplirane informacije. Ako postoji nepotreban state, lako je zaboraviti ažurirati ga i time uvesti bug-ove!

Na primer, ova forma ima **suvišnu** `fullName` state promenljivu:

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState('');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + ' ' + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + ' ' + e.target.value);
  }

  return (
    <>
      <h2>Prijavite se</h2>
      <label>
        Ime:{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Prezime:{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        Vaša karta će biti izdata na ime: <b>{fullName}</b>
      </p>
    </>
  );
}
```

```css
label { display: block; margin-bottom: 5px; }
```

</Sandpack>

Možete je ukloniti i pojednostaviti kod računanjem `fullName`-a tokom renderovanja komponente:

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  const fullName = firstName + ' ' + lastName;

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }

  return (
    <>
      <h2>Prijavite se</h2>
      <label>
        Ime:{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Prezime:{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        Vaša karta će biti izdata na ime: <b>{fullName}</b>
      </p>
    </>
  );
}
```

```css
label { display: block; margin-bottom: 5px; }
```

</Sandpack>

Ovo možda deluje kao mala izmena, ali dosta bug-ova u React aplikacijama se popravljaju na ovaj način.

<LearnMore path="/learn/choosing-the-state-structure">

Pročitajte **[Odabir strukture state-a](/learn/choosing-the-state-structure)** da naučite kako dizajnirati state da izbegnete bug-ove.

</LearnMore>

## Deljenje state-a između komponenata {/*sharing-state-between-components*/}

Ponekad želite da se state-ovi dve komponente menjaju zajedno. Da biste to uradili, uklonite state iz obe komponente, pomerite ga u najbližeg zajedničkog roditelja i prosledite ga nazad kroz props. Ovo je poznato kao "podizanje state-a" i jedna je od najčešćih stvari koje ćete pisati u React kodu.

U ovom primeru, samo jedan panel treba biti aktivan. Da biste to postigli, umesto čuvanja state-a u svakom pojedinačnom panel-u, roditeljska komponenta drži state i specificira props svoje dece.

<Sandpack>

```js
import { useState } from 'react';

export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);
  return (
    <>
      <h2>Almati, Kazahstan</h2>
      <Panel
        title="O gradu"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        Sa populacijom od oko 2 miliona, Almati je najveći grad u Kazahstanu. Bio je glavni grad od 1929. do. 1997. godine.
      </Panel>
      <Panel
        title="Etimologija"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        Ime potiče od reči <span lang="kk-KZ">алма</span>, što na kazaškom jeziku znači "jabuka", i često se prevodi kao "pun jabuka". U suštini, region koji okružuje Almati se smatra pradomovinom jabuka, a divlja <i lang="la">Malus sieversii</i> se smatra mogućim pretkom moderne domaće jabuke.
      </Panel>
    </>
  );
}

function Panel({
  title,
  children,
  isActive,
  onShow
}) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>
          Prikaži
        </button>
      )}
    </section>
  );
}
```

```css
h3, p { margin: 5px 0px; }
.panel {
  padding: 10px;
  border: 1px solid #aaa;
}
```

</Sandpack>

<LearnMore path="/learn/sharing-state-between-components">

Pročitajte **[Deljenje state-a između komponenata](/learn/sharing-state-between-components)** da naučite kako da podignete state i sinhronizujete komponente.

</LearnMore>

## Čuvanje i resetovanje state-a {/*preserving-and-resetting-state*/}

Kada ponovo renderujete komponentu, React mora da odluči koje delove stabla da zadrži (i ažurira), a koje da odbaci ili ponovo kreira od nule. U većini slučajeva, React-ovo automatsko ponašanje radi dovoljno dobro. Po default-u, React čuva delove stabla koji se "podudaraju" sa prethodno renderovanim stablom komponente.

Međutim, ponekad ovo nije ono što želite. U ovoj aplikaciji za poruke, pisanje poruke i naknadna promena primaoca ne resetuju input. Ovo može navesti korisnika da greškom pošalje poruku pogrešnoj osobi:

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat contact={to} />
    </div>
  )
}

const contacts = [
  { name: 'Taylor', email: 'taylor@mail.com' },
  { name: 'Alice', email: 'alice@mail.com' },
  { name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  selectedContact,
  contacts,
  onSelect
}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map(contact =>
          <li key={contact.email}>
            <button onClick={() => {
              onSelect(contact);
            }}>
              {contact.name}
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={'Piši korisniku ' + contact.name}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>Pošalji na {contact.email}</button>
    </section>
  );
}
```

```css
.chat, .contact-list {
  float: left;
  margin-bottom: 20px;
}
ul, li {
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

React vam omogućava da override-ujete default ponašanje i *forsirate* komponentu da resetuje svoj state prosleđivanjem različitog `key`-a, na primer `<Chat key={email} />`. Ovo govori React-u da ako je primalac drugačiji, treba da smatra da se *drugačija* `Chat` komponenta treba ponovo kreirati od nule sa novim podacima (i UI-jem poput input-a). Sada, promena primaoca resetuje polje za input--iako renderujete istu komponentu.

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat key={to.email} contact={to} />
    </div>
  )
}

const contacts = [
  { name: 'Taylor', email: 'taylor@mail.com' },
  { name: 'Alice', email: 'alice@mail.com' },
  { name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  selectedContact,
  contacts,
  onSelect
}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map(contact =>
          <li key={contact.email}>
            <button onClick={() => {
              onSelect(contact);
            }}>
              {contact.name}
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={'Piši korisniku ' + contact.name}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>Pošalji na {contact.email}</button>
    </section>
  );
}
```

```css
.chat, .contact-list {
  float: left;
  margin-bottom: 20px;
}
ul, li {
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

<LearnMore path="/learn/preserving-and-resetting-state">

Pročitajte **[Čuvanje i resetovanje state-a](/learn/preserving-and-resetting-state)** da naučite više o životnom veku state-a i kako da ga kontrolišete.

</LearnMore>

## Izdvajanje state logike u reducer {/*extracting-state-logic-into-a-reducer*/}

Komponente sa mnogo ažuriranja state-a koji se prostiru kroz mnogo event handler-a mogu postati preobimne. U tim slučajevima, možete grupisati svu logiku ažuriranja state-a izvan komponente u jednu funkciju koja se naziva "reducer". Vaši event handler-i postaju koncizni jer samo specificiraju korisničke "akcije". Na dnu fajla, reducer funkcija specificira kako bi state trebao da se ažurira kao reakcija na svaku akciju!

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

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
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <>
      <h1>Plan puta u Pragu</h1>
      <AddTask
        onAddTask={handleAddTask}
      />
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

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Poseti Kafkin muzej', done: true },
  { id: 1, text: 'Gledaj lutkarsku predstavu', done: false },
  { id: 2, text: 'Slikaj Lenonov zid', done: false }
];
```

```js src/AddTask.js hidden
import { useState } from 'react';

export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Dodaj zadatak"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        onAddTask(text);
      }}>Dodaj</button>
    </>
  )
}
```

```js src/TaskList.js hidden
import { useState } from 'react';

export default function TaskList({
  tasks,
  onChangeTask,
  onDeleteTask
}) {
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task
            task={task}
            onChange={onChangeTask}
            onDelete={onDeleteTask}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ task, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            onChange({
              ...task,
              text: e.target.value
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
          onChange({
            ...task,
            done: e.target.checked
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>
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

<LearnMore path="/learn/extracting-state-logic-into-a-reducer">

Pročitajte **[Izdvajanje state logike u reducer](/learn/extracting-state-logic-into-a-reducer)** da naučite kako grupisati logiku u reducer funkciju.

</LearnMore>

## Prosleđivanje podataka duboko uz context {/*passing-data-deeply-with-context*/}

Često ćete proslediti informaciju od roditeljske ka dečjoj komponenti kroz props. Ali, prosleđivanje props-a može postati nepogodno ako trebate proslediti neki prop kroz mnogo komponenata ili ako mnogo komponenata treba imati istu informaciju. Context omogućava roditeljskoj komponenti da neku informaciju učini dostupnom svakoj komponenti u stablu ispod nje—bez obzira koliko duboko—bez eksplicitnog prosleđivanja props-a.

Ovde, `Heading` komponenta određuje nivo naslova "pitajući" najbliži `Section` za svoj nivo. Svaki `Section` prati svoj nivo pitajući roditeljski `Section` i dodajući jedan na to. Svaki `Section` pruža informaciju svim komponentama ispod bez prosleđivanja props-a--to radi kroz context.

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
      throw Error('Heading mora biti unutar Section-a!');
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

<LearnMore path="/learn/passing-data-deeply-with-context">

Pročitajte **[Prosleđivanje podataka duboko uz context](/learn/passing-data-deeply-with-context)** da naučite da koristite context kao alternativu prosleđivanju props-a.

</LearnMore>

## Skaliranje sa reducer-om i context-om {/*scaling-up-with-reducer-and-context*/}

Reducer-i omogućavaju grupisanje logike za ažuriranje state-a u komponenti. Context vam omogućava da prosledite informaciju duboko drugim komponentama. Možete kombinovati reducer-e i context da upravljate složenim state-om.

Sa ovim pristupom, roditeljska komponenta upravlja kompleksnim state-om pomoću reducer-a. Druge komponente negde duboko u stablu mogu čitati njen state preko context-a. One takođe mogu otpremiti akcije koje ažuriraju taj state.

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

export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');
  const dispatch = useTasksDispatch();
  return (
    <>
      <input
        placeholder="Dodaj zadatak"
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

<LearnMore path="/learn/scaling-up-with-reducer-and-context">

Pročitajte **[Skaliranje sa reducer-om i context-om](/learn/scaling-up-with-reducer-and-context)** da naučite kako se upravljanje state-om povećava u rastućoj aplikaciji.

</LearnMore>

## Šta je sledeće? {/*whats-next*/}

Pređite na [Reagovanje na input pomoću stanja](/learn/reacting-to-input-with-state) da biste počeli da čitate ovo poglavlje stranicu po stranicu!

Ili, ako ste već upoznati sa ovim temama, zašto ne biste pročitali poglavlje [Evakuacioni izlazi](/learn/escape-hatches)?
