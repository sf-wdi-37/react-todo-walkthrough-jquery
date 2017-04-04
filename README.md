# React Todo

## Learning Objectives
- Build a todo app in React that persists with a backend
- Use React Router to deep link
- Use axios promise library to retrieve data from a back end
- Pass state from parent components to children as props
- Pass state from children components to their parents as arguments to functions

## Framing
For today, we'll be creating a Todo app in React.

We've learned a tremendous amount about object oriented structures for web development. And they were great. With angular, we dabbled a bit with feature-based separation of concerns. React's component model takes that separation further and reduces the potential of tight coupling that often attends object oriented. Think of the FIRST principles:

#### Focused

Components should do one thing and do it well. It takes some time for most developers coming from an OOP background to adjust to React's component-based architecture. At first, a dev from an OOP background may pack too much information into a component. This is a fine starting point, but as you progress you will get a better sense of how to minimize component code.

> Think back to the Post component from the intro's class.

#### Independent

Components should increase cohesion and reduce coupling. Behavior in one component should not impact the behavior of another. In other words, components should not rely on one another.

> But they should compliment one another, just like our Comment component did for Post in the intro's class.

#### Reusable

Components should be written in a way that reduces the duplication of code. Reusability keeps things DRY!

#### Small

Ideally, components should be short and condensed.

#### Testable

Because the same input will always produce the same output, components are easily unit testable.

## React Todo
Alright it's time to build! We're going to be building this application from scratch! It won't be exactly like the repo above, but it'll be pretty close and follow much of the same structure.

> If you get behind, all code written today will be in the lesson plan. The error messages you'll get in terminal and in the chrome dev tools from React are usually very accurate and helpful, so please utilize them. Please keep questions pertinent to content. We should also note that some of the code snippets will be repetitions to reiterate points of learning. Some of them might just be updates to existing files. Some of them might be brand new content you have to add all of.

[Sprint 0: Getting Started](sprints/Sprint0.md)

[Sprint 1: React Router setup](sprints/Sprint1.md)

[Sprint 2: Containers](sprints/Sprint2.md)

[Sprint 3: Fetching data with Axios](sprints/Sprint3.md)

[Sprint 4: Creating Todos](sprints/Sprint4.md)

[Sprint 5: Deleting Todos](sprints/Sprint5.md)

[Sprint 6: Edit/Update a Todo](sprints/Sprint6.md)

## Editing and Updating Todos

### Implementing Edit

In `containers/TodosContainer.js`:

```js
updateTodo(todoBody) {
    var todoId = this.state.editingTodoId
    function isUpdatedTodo(todo) {
        return todo._id === todoId;
    }
    TodoModel.update(todoId, todoBody).then((res) => {
        let todos = this.state.todos
        todos.find(isUpdatedTodo).body = todoBody
        this.setState({todos: todos, editingTodoId: null, editing: null})
    })
}
editTodo(todo){
  this.setState({
    editingTodoId: todo._id
  })
}
render(){
  return (
    <div className='TodosContainer'>
      <h2>This is the Todos Container</h2>
      <Todos
        todos={this.state.todos}
        editingTodoId={this.state.editingTodoId}
        onEditTodo={this.editTodo.bind(this)}
        onDeleteTodo={this.deleteTodo.bind(this)} />
      <CreateTodoForm
        createTodo={this.createTodo.bind(this)} />
    </div>
  )
}
```

Why would we add editingTodoId to the container? Why might the container be aware of a ***single*** todo ID, in the context of an edit?

In the `components/Todos.js`, add `editingTodoId` and `onEditTodo` to `<Todo>` props:


```js
//....
let todos = this.props.todos.map( (todo) => {
  return (
    <Todo
      key={todo._id}
      todo={todo}
      editingTodoId={this.props.editingTodoId}
      onEditTodo={this.props.onEditTodo}
      onDeleteTodo={this.props.onDeleteTodo}
      onUpdateTodo={this.props.onUpdateTodo}
    />
  )
})
//...
```

<!-- Todo changes -->
In `components/Todo.js` We need to use the method:

```js
render(){
    if (this.props.editingTodoId === this.props.todo._id){
      console.log(`${this.props.todo.body} is being edited`);
    }
    return(
      <p data-todos-index={this.props.todo.id}>
        <span onClick={() => this.props.onEditTodo(this.props.todo)}>
          {this.props.todo.body}
        </span>
        <span
          className='deleteButton'
          onClick={ () => this.props.onDeleteTodo(this.props.todo) }>
            (X)
        </span>
      </p>
    )
  }
```

Phew! Now we can test out our props-flow by clicking on a todo and trigger a `console.log`.
### Breaking it Down:

#### Trickling Down

In `TodosContainer`, a method called `editTodo` is setting the `state` of the `<TodosContainer>` component to include a property called `editingTodoId`. That `state` is then ultimately handed down  to the `<Todo>` component. This state trickles down from `<TodosContainer>` to `<Todo>` as props.

#### Bubbling Up (and then Trickling Back Down again)

How are we passing in the corresponding `todo` id back up to `TodosContainer`? The TodosContainer-state is being updated with a particular `todo` id, which is a `prop` of the `<Todo>` component.

It's being passed an argument to a function that **is defined in** and **trickles down from** `TodosContainer`, to here, in `components/Todo.js`:

```js
<span onClick={() => this.props.onEditTodo(this.props.todo)}>
```

Elsewhere, over in `containers/TodosContainer.js`:

```js
render(){
  return (
    <div className='TodosContainer'>
      <h2>This is the Todos Container</h2>
      <Todos
        todos={this.state.todos}
        editingTodoId={this.state.editingTodoId}
        onEditTodo={this.editTodo.bind(this)}
        onDeleteTodo={this.deleteTodo.bind(this)} />
      <CreateTodoForm
        createTodo={this.createTodo.bind(this)} />
    </div>
  )
}
```

This certainly the trickiest part of the lesson-- the rest is easy by comparison (still pretty tough, at first!).

### Replacing the console.log with a Form for editing Todos

The next steps here involve composing a form in place of where we have that `console.log` in `components/Todo.js`.

You should replace it with something like this:

```js
return (
  <TodoForm
    autoFocus={true}
    buttonName="Update Todo!"
    onTodoAction={this.props.onUpdateTodo} />
)
```

You will then have to both write that component and then import it into `components/Todo.js`:

```js

//TodoForm.js
import React, {Component} from 'react'

class TodoForm extends Component {
  onChange(event) {
    this.setState({
      todo: event.target.value
    })
  }
  onSubmit(event){
    event.preventDefault()
    var todo = this.state.todo
    console.log("todo is", todo)
    this.props.onUpdateTodo(todo)
    this.setState({
      todo: ""
    })
  }
  render(){
    return (
      <div className='todoForm'>
        <form onSubmit={e => this.onSubmit(e)}>
          <input
            autoFocus={this.props.autoFocus}
            onChange={e => this.onChange(e)}
            placeholder='Write a todo here ...'
            type='text'
            value={(this.state && this.state.todo) || ''} />
          <button type='submit'>{this.props.buttonName}</button>
        </form>
      </div>
    )
  }
}

export default TodoForm

```

```js
//Todo.js
//...
console.log(`${this.props.todo.body} is being edited`);
return (
  <TodoForm
    autoFocus={true}
    onUpdateTodo={this.props.onUpdateTodo}
    buttonName="Update Todo!"/>
)
//...
```

```js
//Todos.js
let todos = this.props.todos.map( (todo) => {
  return (
    <Todo
      key={todo._id}
      todo={todo}
      editingTodoId={this.props.editingTodoId}
      onEditTodo={this.props.onEditTodo}
      onDeleteTodo={this.props.onDeleteTodo}
      onUpdateTodo={this.props.onUpdateTodo}
    />
  )
})
//...
```

In `models/Todo.js` add our method:

```js
static update(todoId, todoBody) {
    let request = axios.put(`https://super-crud.herokuapp.com/todos/${todoId}`, {
        body: todoBody
    })
    return request
}
```

Think back to what we did for the other CRUD actions--we define some axios behavior in `/models/Todo.js`. Then we define a method in `TodosContainer` that will handle update behavior.

Then we make our way down from `TodosContainer` to `Todos` to `Todo`, with `state` trickling down as `props`.

## Conclusion

We've learned how to do full CRUD for a basic todo app here. We've seen in particular how props can be trickled down through parent and child components to make a very modular app. We've also been introduced to the magic of axios for network calls from our frontend.
