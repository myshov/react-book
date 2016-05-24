# Editing `Notes`

Editing notes is a similar problem as deleting them. The data flow is exactly the same. We'll need to define an `onEdit` callback and  `bind` an id of the note being edited at `Notes`.

What makes this scenario difficult is the user interface requirement. It's not enough just to have a button. We'll need some way to allow the user to input a new value which we can then commit to the data model.

One way to achieve this is to implement so called **inline editing**. The idea is that when a user click a note, we'll show an input. After the user has finished editing and signals that to use either by triggering the `blur` event or by hitting *enter*, we'll capture the value and update.

## Implementing `Editable`

To keep the application clean, I'll wrap this behavior into a component known as `Editable`. It will give us an API like this:

```javascript
<Editable editing={editing} value={task}
  onEdit={onEdit.bind(null, id)}
  onValueClick={onValueClick.bind(null, id)}>
```

This is an example of a **controlled** component. We'll control the editing state explicitly from outside of the component. This gives us more power, but it also makes `Editable` more involved to use.

An alternative way to handle this would have been to leave the control over the `editing` state to `Editable`. This **uncontrolled** way of designing can be valid if you don't want to do anything with the state outside of the component.

It is possible to use both of these designs together. You can even have a controlled component that has uncontrolled elements inside. In this case we'll end up using an uncontrolled design for the `input` that `Editable` will contain for example. Even that could be turned into something controlled should we want to.

Logically `Editable` consists of two separate portions. We'll need to display the `Value` while we are not `editing`. In case we are `editing`, we'll want to show an `Edit` control instead. In this case we'll settle for a simple input as that will do the trick.

Before digging into the details, we can implement a little stub and connect that to the application. This will give us the basic structure we need to grow the rest.

T> The official documentation of React discusses [controlled components](https://facebook.github.io/react/docs/forms.html) in greater detail.

### Extracting Rendering from `Note`

Currently `Note` controls what is rendered inside it. It renders the passed task and connects a deletion button. We could push `Editable` inside it and handle the wiring through `Note` interface. Even though that might be one valid way to do it, we can push the rendering concern on a higher level.

Having the concept of `Note` is useful especially when we'll expand the application further so there's no need to remove it. Instead, we can give the control over its rendering behavior to `Notes` and wire it there.

React provides a prop known as `children` for this purpose. Adjust `Note` and `Notes` as follows to push the control over `Note` rendering to `Notes`:

**app/Note.jsx**

```javascript
import React from 'react';

leanpub-start-remove
export default ({task, onDelete}) => (
  <div>
    <span>{task}</span>
    <button onClick={onDelete}>x</button>
  </div>
);
leanpub-end-remove
leanpub-start-insert
export default ({children, ...props}) => (
  <div {...props}>
    {children}
  </div>
);
leanpub-end-insert
```

**app/Notes.jsx**

```javascript
import React from 'react';
import Note from './Note';

export default ({notes, onDelete=() => {}}) => {
  return (
    <ul>{notes.map(({id, task}) =>
      <li key={id}>
leanpub-start-remove
        <Note
          onDelete={onDelete.bind(null, id)}
          task={task} />
leanpub-end-remove
leanpub-start-insert
        <Note>
          <span>{task}</span>
          <button onClick={onDelete.bind(null, id)}>x</button>
        </Note>
leanpub-end-insert
      </li>
    )}</ul>
  );
}
```

### Adding `Editable` Stub

We can model a rough starting point based on our specification as below. The idea is that we'll branch based on the `editing` prop and attach the props needed for implementing our logic:

**app/Editable.jsx**

```javascript
import React from 'react';

export default ({editing, value, onEdit, onValueClick, ...props}) => (
  <div {...props}>
    {editing ?
      <Edit value={value} onEdit={onEdit} /> :
      <Value value={value} onValueClick={onValueClick} />
    }
  </div>
)

const Value = ({onValueClick = () => {}, value}) => {
  return (
    <div onClick={onValueClick}>
      <span className="value">{value}</span>
    </div>
  );
};

const Edit = ({value}) => <span>edit: {value}</span>;
```

To see our stub in action we still need to connect it with our application.

### Connecting `Editable` with `Notes`

We still need to replace the relevant portions of the code to point at `Editable`. There are more props to track and to connect:

**app/Notes.jsx**

```javascript
import React from 'react';
import Note from './Note';
leanpub-start-insert
import Editable from './Editable';
leanpub-end-insert

leanpub-start-remove
export default ({notes, onDelete=() => {}}) => {
leanpub-end-remove
leanpub-start-insert
export default ({
  notes,
  onValueClick=() => {}, onEdit=() => {}, onDelete=() => {}
}) => {
leanpub-end-insert
  return (
leanpub-start-remove
    <ul>{notes.map(({id, task}) =>
leanpub-end-remove
leanpub-start-insert
    <ul>{notes.map(({id, editing, task}) =>
leanpub-end-insert
      <li key={id}>
        <Note>
leanpub-start-remove
          <span>{task}</span>
leanpub-end-remove
leanpub-start-insert
          <Editable
            className="editable"
            editing={editing}
            value={task}
            onValueClick={onValueClick.bind(null, id)}
            onEdit={onEdit.bind(null, id)} />
leanpub-end-insert
          <button onClick={onDelete.bind(null, id)}>x</button>
        </Note>
      </li>
    )}</ul>
  );
}
```

If everything went right, you should see something like this:

![Connected `Editable`](images/react_06.png)

### Tracking `Note` `editing` State

We are still missing logic needed to control the `Editable`. Given the state of our application is maintained at `App`, we'll need to deal with it there. It should set the `editable` flag of the edited note to `true` when we begin to edit and set it back to `false` when we complete the editing process. We should also adjust its `task` using the new value. For now we are interested in just getting the `editable` flag to work, though. Modify as follows:

**app/App.jsx**

```javascript
...

export default class App extends React.Component {
  constructor(props) {
    ...
  }
  render() {
    const {notes} = this.state;

    return (
      <div>
        <button onClick={this.addNote}>+</button>
leanpub-start-remove
        <Notes notes={notes} onDelete={this.deleteNote} />
leanpub-end-remove
leanpub-start-insert
        <Notes
          notes={notes}
          onValueClick={this.activateNoteEdit}
          onEdit={this.editNote}
          onDelete={this.deleteNote}
          />
leanpub-end-insert
      </div>
    );
  }
  deleteNote = (id, e) => {
    ...
  }
leanpub-start-insert
  activateNoteEdit = (id) => {console.log('act', id);
    this.setState({
      notes: this.state.notes.map(note => {
        if(note.id === id) {
          note.editing = true;
        }

        return note;
      })
    });
  }
  editNote = (id, task) => {
    this.setState({
      notes: this.state.notes.map(note => {
        if(note.id === id) {
          note.editing = false;
        }

        return note;
      })
    });
  }
leanpub-end-insert
}
```

If you try to edit a `Note` now, you should see something like this:

![Connected `Editable`](images/react_07.png)

T> If we used a normalized data structure (i.e., `{<id>: {id: <id>, task: <str>}}`), it would be possible to write the operations using `Object.assign` and avoid mutation.

T> In order to clean up the code, you could extract a method to contain the logic shared by `activateNoteEdit` and `editNote`.

### Implementing `Value`

XXX

## Tracking `Note` `editing` State

XXX

Just as earlier with `App`, we need to deal with state again. This means a function based component won't be enough anymore. Instead, we need to convert it to a heavier format. For the sake of consistency, I'll be using the same component definition style as with `App`. In addition, we need to alter the `editing` state based on the user behavior, and finally render the right element based on it. Here's what this means in terms of React:

**app/components/Note.jsx**

```javascript
import React from 'react';

export default class Note extends React.Component {
  constructor(props) {
    super(props);

    // Track `editing` state.
    this.state = {
      editing: false
    };
  }
  render() {
    // Render the component differently based on state.
    if(this.state.editing) {
      return this.renderEdit();
    }

    return this.renderNote();
  }
  renderEdit = () => {
    // We deal with blur and input handlers here. These map to DOM events.
    // We also set selection to input end using a callback at a ref.
    // It gets triggered after the component is mounted.
    //
    // We could also use a string reference (i.e., `ref="input") and
    // then refer to the element in question later in the code through
    // `this.refs.input`. We could get the value of the input using
    // `this.refs.input.value` through DOM in this case.
    //
    // Refs allow us to access the underlying DOM structure. They
    // can be using when you need to move beyond pure React. They
    // also tie your implementation to the browser, though.
    return <input type="text"
      ref={
        element => element ?
        element.selectionStart = this.props.task.length :
        null
      }
      autoFocus={true}
      defaultValue={this.props.task}
      onBlur={this.finishEdit}
      onKeyPress={this.checkEnter} />;
  };
  renderNote = () => {
    // If the user clicks a normal note, trigger editing logic.
    return <div onClick={this.edit}>{this.props.task}</div>;
  };
  edit = () => {
    // Enter edit mode.
    this.setState({
      editing: true
    });
  };
  checkEnter = (e) => {
    // The user hit *enter*, let's finish up.
    if(e.key === 'Enter') {
      this.finishEdit(e);
    }
  };
  finishEdit = (e) => {
    // `Note` will trigger an optional `onEdit` callback once it
    // has a new value. We will use this to communicate the change to
    // `App`.
    //
    // A smarter way to deal with the default value would be to set
    // it through `defaultProps`.
    //
    // See the *Typing with React* chapter for more information.
    const value = e.target.value;

    if(this.props.onEdit) {
      this.props.onEdit(value);

      // Exit edit mode.
      this.setState({
        editing: false
      });
    }
  };
}
```

If you try to edit a `Note` now, you should see an input and be able to edit the data. Given we haven't set up `onEdit` handler, it doesn't do anything useful yet, though. We'll need to capture the edited data next and update `App` state so that the code works.

T> It can be a good idea to name your callbacks using `on` prefix. This will allow you to distinguish them from other props and keep your code a little tidier.

### Communicating `Note` State Changes

Given we are currently dealing with the logic at `App`, we can deal with `onEdit` there as well. An alternative design might be to push the logic to `Notes` level. This would get problematic with `addNote` as it is functionality that doesn't belong within `Notes` domain. Therefore we'll keep the application state at `App` level.

In order to make `onEdit` work, we will need to capture its output and delegate the result to `App`. Furthermore we will need to know which `Note` was modified so we can update the data accordingly. This can be achieved through data binding as illustrated by the diagram below:

![`onEdit` flow](images/bind.png)

As `onEdit` is defined on `App` level, we'll need to pass `onEdit` handler through `Notes`. So for the stub to work, changes in two files are needed. Here's what it should look like for `App`:

**app/components/App.jsx**

```javascript
import uuid from 'node-uuid';
import React from 'react';
import Notes from './Notes.jsx';

export default class App extends React.Component {
  constructor(props) {
    ...
  }
  render() {
    const notes = this.state.notes;

    return (
      <div>
        <button onClick={this.addNote}>+</button>
leanpub-start-delete
        <Notes notes={notes} />
leanpub-end-delete
leanpub-start-insert
        <Notes notes={notes} onEdit={this.editNote} />
leanpub-end-insert
      </div>
    );
  }
  addNote = () => {
    ...
  };
leanpub-start-insert
  editNote = (id, task) => {
    // Don't modify if trying to set an empty value
    if(!task.trim()) {
      return;
    }

    const notes = this.state.notes.map(note => {
      if(note.id === id && task) {
        note.task = task;
      }

      return note;
    });

    this.setState({notes});
  };
leanpub-end-insert
}
```

T> `this.setState({notes})` is using [Object initializer syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer). It is the same as writing `this.setState({notes: notes})`. See the *Language Features* appendix for more information.

To make the scheme work as designed, we need to modify `Notes` to work according to the idea. It will `bind` the id of the note in question. When the callback is triggered, the remaining parameter receives a value and the callback gets called:

**app/components/Notes.jsx**

```javascript
import React from 'react';
import Note from './Note.jsx';

export default ({notes, onEdit}) => {
  return (
    <ul>{notes.map(note =>
      <li key={note.id}>
        <Note
          task={note.task}
          onEdit={onEdit.bind(null, note.id)} />
      </li>
    )}</ul>
  );
}
```

If you refresh and try to edit a `Note` now, the modification should stick. The same idea can be used to implement a lot of functionality and this is a pattern you will see a lot.

The current design isn't flawless. What if we wanted to allow newly created notes to be editable straight from the start? Given `Note` encapsulated this state, we don't have simple means to access it from the outside. The current solution is enough for now. We'll address this issue properly in *From Notes to Kanban* chapter and extract the state there.

![Edited a note](images/react_06.png)

## Conclusion

XXX