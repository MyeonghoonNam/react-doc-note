## React Learn

### State 관리하기

애플리케이션이 커짐에 따라, `state`가 어떻게 구성되는지 그리고 데이터가 컴포넌트 간에 어떻게 흐르는지에 대해 의식적으로 파악할 필요가 있다.

불필요하거나 중복된 `state`는 버그의 흔한 원인이다. 버그를 예방하기 위해 우리는 아래와 같은 방법들에 대해 이해할 수 있어야한다.

- `state`를 잘 구성하는 방법
- `state` 업데이트 로직을 유지 보수 가능하게 관리하는 방법
- 멀리 있는 컴포넌트 간에 `state`를 공유하는 방법

### State 로직을 reducer로 작성하기

하나의 컴포넌트에서 `state` 업데이트가 여러 이벤트 핸들러로 분산되는 경우가 있다. 이 경우에 컴포넌트를 관리하기가 어려워진다.

따라서 `state`를 업데이트하는 모든 로직을 `reducer`를 사용해 컴포넌트 외부의 단일 함수로 통합해 관리할 수 있다.

#### reducer를 사용하여 state 로직 통합하기

컴포넌트가 복잡해지면 컴포넌트의 state가 업데이트되는 다양한 경우를 한눈에 파악하기 어려워질 수 있다.

```js
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
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList tasks={tasks} onChangeTask={handleChangeTask} onDeleteTask={handleDeleteTask} />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Visit Kafka Museum', done: true },
  { id: 1, text: 'Watch a puppet show', done: false },
  { id: 2, text: 'Lennon Wall pic', done: false },
];
```

각 이벤트 핸들러는 `state`를 업데이트하기 위해 `setTasks`를 호출한다. 컴포넌트가 커질수록 그 안에서 `state`를 다루는 로직의 양도 늘어나게 된다.

복잡성은 줄이고 접근성을 높이기 위해서, 컴포넌트 내부에 있는 `state` 로직을 컴포넌트 위부의 `reducer` 단일 함수로 옮길 수 있다.

`reducer`는 `state`를 다루는 방법 중 하나이다. 아래의 세 가지 단계에 걸쳐 `useState` 로직을 `useReducer` 로직으로 바꿀 수 있다.

1. `state`를 설정하는 로직을 `action`을 `dispatch` 함수로 전달하는 로직으로 수정
2. `reducer` 함수 작성
3. 컴포넌트에서 `reducer` 사용

#### 1단계: `state`를 설정하는 로직을 `action`을 `dispatch` 함수로 전달하는 로직으로 수정

현재 이벤트 핸들러는 `state`를 설정함으로 **문제를 어떻게 해결할 지(명령형)**를 명시한다.

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

위 코드에서 `state`의 설정 관련 로직을 전부 지워보면 세가지 이벤트 핸들러가 남는다.

- `handleAddTask`: 사용자가 “Add”를 눌렀을 때 호출
- `handleChangeTask`: 사용자가 task를 토글하거나 “Save”를 누르면 호출
- `handleDeleteTask`: 사용자가 “Delete”를 누르면 호출

`reducer`를 사용한 `state` 관리는 `state`를 직접 설정하는 것과 약간 다르다.

`state`를 설정하여 React에게 <b>문제를 어떻게 해결할 지(명령형)</b>를 지시하는 대신, 이벤트 핸들러에서 `action`을 전달하여 <b>사용자가 무슨일을 해야하는지(선언형)</b>를 지정한다.

`state` 업데이트 로직은 `reducer` 함수내에서 관리하기에 이벤트 핸들러를 통해 **task를 설정**하는 대신 **task를 추가/변경/삭제**하는 `action`을 전달하는 것이다.

이러한 방식이 사용자의 의도를 좀 더 명확하게 설명할 수 있게 된다.

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

`dispatch` 함수에 넣어준 객체를 `action`이라고 한다.

```js
dispatch({
  type: 'what_happened', // 컴포넌트마다 다른 값
  // 다른 필드는 이곳에
});
```

`action` 객체는 일반적인 자바스크립트 객체로 어떠한 형태든 될 수 있다. 보통의 경우 발생한 일을 설명하기 위해 `type`을 키로 가지며 값으로 문자열을 가지는 프로퍼티를 지정하고, 이외의 정보는 다른 필드에 담아서 전달하도록 작성한다.

기억할 것은 `type`의 값으로 무슨 일이 일어나는지를 설명할 수 있는 값을 넣어주는 것이다.

#### 2단계: `reducer` 함수 작성하기

`reducer` 함수는 `state`에 대한 로직을 넣는 곳이다. 이 함수는 현재의 `state` 값과 `action` 객체, 이렇게 두 개의 인자를 받고 다음 `state` 값을 반환한다.

```js
function reducer(state, action) {
  // React가 설정하게 될 다음 state 값을 반환한다.
}
```

React는 `reducer`에서 반환한 값을 `state`에 설정한다.

`reducer` 함수는 `state`를 인자로 받고 있기 때문에 **컴포넌트 외부에서 선언할 수 있다.** 이렇게 하면 복잡한 코드의 비지니스 로직을 외부로 분리할 수 있어 가독성 향상에 도움이 될 수 있다.

`reducer` 함수안에서는 `switch` 문을 일반적으로 활용한다. 각자 다른 `case` 속에서 선언된 변수들이 서로 충돌하지 않도록 `case` 블록을 중괄호로 감싸는걸 선호하며 `return`으로 업데이트된 `state`를 반환하도록 한다.

#### 3단계: 컴포넌트에서 reducer 사용하기

마지막으로 작성한 `reducer`를 컴포넌트에서 연결할 차례이다. React에서 `useReducer` hook을 호출한다.

```js
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

`useReducer` hook은 초기 `state` 값을 입력받아 `stateful` 값을 반환한다는 점과 `state`를 설정하는 함수(`useReducer`의 `dispatch` 함수)의 원리를 보면 `useState`와 비슷한점이 있다.

하지만 `useReducer`는 두 개의 인자를 넘겨받는점이 조금 다르다.

1. `reducer` 함수
2. 초기 `state` 값

그리고 아래와 같이 2가지 요소를 반환한다.

1. `state`를 담을 수 있는 값
2. `dispatch` 함수(사용자의 `action`을 `reducer` 함수에게 전달)

```js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

interface Task {
  id: number;
  text: string;
  done: boolean;
}

type Action =
  | ({ type: 'added' } & Pick<Task, 'id' | 'text'>)
  | { type: 'changed'; task: Task }
  | ({ type: 'deleted' } & Pick<Task, 'id'>);

function tasksReducer(state: Task[], action: Action) {
  switch (action.type) {
    case 'added': {
      return [
        ...state,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return state.map((v) => (v.id === action.task.id ? action.task : v));
    }
    case 'deleted': {
      return state.filter((v) => v.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

let nextId = 3;

const initialTasks = [
  { id: 0, text: 'Visit Kafka Museum', done: true },
  { id: 1, text: 'Watch a puppet show', done: false },
  { id: 2, text: 'Lennon Wall pic', done: false },
];

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text: string) {
    dispatch({
      type: 'added',
      id: nextId++,
      text,
    });
  }

  function handleChangeTask(task: Task) {
    dispatch({
      type: 'changed',
      task,
    });
  }

  function handleDeleteTask(taskId: number) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList tasks={tasks} onChangeTask={handleChangeTask} onDeleteTask={handleDeleteTask} />
    </>
  );
}
```

이 과정을 통해 이벤트 핸들러는 `action`을 전달해줘서 **사용자가 무슨일을 해야하는지**에 대해서만 집중하고 `reducer`에서는 응답으로 **문제를 어떻게 해결할지**에 대해서만 집중할 수 있도록 코드를 분리하는 효과를 얻을 수 있게 되었다.

#### `useState`와 `useReducer` 비교하기

`reducer`가 좋은 점만 있는 것은 아니다. 각각의 장/단점을 고려하여 상황에 맞게 hook을 사용하는 능력이 중요하다.

- **코드 크기**: 일반적으로 `useState`를 사용하면, 미리 작성해야 하는 코드가 줄어든다. `useReducer`를 사용하면 `reducer` 함수 그리고 `action`을 전달하는 부분 둘 다 작성해야 한다. 하지만 여러 이벤트 핸들러에서 비슷한 방식으로 `state`를 업데이트하는 경우, `useReducer`를 사용하면 코드의 양을 줄이는 데 도움이 될 수 있다.

- **가독성**: `useState`로 간단한 `state`를 업데이트하는 경우 가독성이 좋은 편입니다. 그렇지만 더 복잡한 구조의 `state`를 다루게 되면 컴포넌트의 코드 양이 더 많아져 한눈에 읽기 어려워질 수 있습니다. 이 경우 `useReducer`를 사용하면 업데이트 로직이 어떻게 동작하는지와 이벤트 핸들러를 통해서 `무엇이 발생했는지` 구현한 부분을 명확하게 구분할 수 있습니다.

- **디버깅**: `useState`를 사용하며 버그를 발견했을 때, 왜, 어디서 `state`가 잘못 설정됐는지 찾기 어려울 수 있습니다. `useReducer`를 사용하면, 콘솔 로그를 `reducer`에 추가하여 `state`가 업데이트되는 모든 부분과 왜 해당 버그가 발생했는지(어떤 action으로 인한 것인지)를 확인할 수 있습니다. 각 `action`이 올바르게 작성되어 있다면, 버그를 발생시킨 부분이 `reducer` 로직 자체에 있다는 것을 알 수 있을 것입니다. 그렇지만 `useState`를 사용하는 경우보다 더 많은 코드를 단계별로 실행해서 디버깅 해야 하는 점이 있기도 합니다.

- **테스팅**: `reducer`는 컴포넌트에 의존하지 않는 **순수 함수**입니다. 이는 `reducer`를 독립적으로 분리해서 내보내거나 테스트할 수 있다는 것을 의미합니다. 일반적으로 더 현실적인 환경에서 컴포넌트를 테스트하는 것이 좋지만, 복잡한 `state`를 업데이트하는 로직의 경우 `reducer`가 특정 초기 `state` 및 `action`에 대해 특정 `state`를 반환한다고 생각하고 테스트하는 것이 유용할 수 있습니다.

- **개인적인 취향**: `reducer`를 좋아하는 사람도 있지만, 그렇지 않는 사람들도 있습니다. 이건 선호도의 문제입니다. `useState`와 `useReducer`는 동일한 방식이기 때문에 언제나 마음대로 바꿔서 사용해도 무방합니다.

만약 일부 컴포넌트에서 잘못된 방식으로 `state`를 업데이트하는 것으로 인한 버그가 자주 발생하거나 해당 코드에 더 많은 구조를 도입하고 싶다면 `reducer` 사용을 권장합니다.

이때 모든 부분에 `reducer`를 적용하지 않아도 됩니다. `useState`와 `useReducer`를 혼합하여 사용해도 문제가 없습니다. 이 둘은 심지어 같은 컴포넌트 안에서도 사용할 수 있습니다.

#### `reducer` 잘 작성하기

`reducer`를 작성할 때 반드시 명심할 점이 있다.

- **`Reducer`는 반드시 순수해야 한다.**

## React Refernce

### Hooks

#### useReducer
