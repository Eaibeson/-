
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);


import React, { useState, useMemo, useCallback, useRef, useEffect } from 'react';
import { TaskProvider } from './context/TaskContext';
import TaskList from './components/TaskList';
import TaskFilter from './components/TaskFilter';
import TaskForm from './components/TaskForm';

export default function App() {
  return (
    <TaskProvider>
      <div className="app">
        <h1>Менеджер завдань</h1>
        <TaskForm />
        <TaskFilter />
        <TaskList />
      </div>
    </TaskProvider>
  );
}


import React, { createContext, useState, useEffect } from 'react';

export const TaskContext = createContext();

export function TaskProvider({ children }) {
  const [tasks, setTasks] = useState(() => {
    const saved = localStorage.getItem('tasks');
    return saved ? JSON.parse(saved) : [];
  });
  const [filter, setFilter] = useState('all');

  useEffect(() => {
    localStorage.setItem('tasks', JSON.stringify(tasks));
  }, [tasks]);

  const addTask = (task) => {
    setTasks(prev => [...prev, task]);
  };

  const toggleTask = (id) => {
    setTasks(prev => prev.map(t => t.id === id ? { ...t, completed: !t.completed } : t));
  };

  const deleteTask = (id) => {
    setTasks(prev => prev.filter(t => t.id !== id));
  };

  const updateTask = (id, newText) => {
    setTasks(prev => prev.map(t => t.id === id ? { ...t, text: newText } : t));
  };

  return (
    <TaskContext.Provider value={{ tasks, addTask, toggleTask, deleteTask, updateTask, filter, setFilter }}>
      {children}
    </TaskContext.Provider>
  );
}


import React, { useState, useContext, useRef, useEffect } from 'react';
import { TaskContext } from '../context/TaskContext';

export default function TaskForm() {
  const { addTask } = useContext(TaskContext);
  const [text, setText] = useState('');
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current.focus();
  }, []);

  const handleSubmit = (e) => {
    e.preventDefault();
    if (text.trim()) {
      addTask({ id: Date.now(), text, completed: false });
      setText('');
      inputRef.current.focus();
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input ref={inputRef} value={text} onChange={(e) => setText(e.target.value)} placeholder="Нове завдання..." />
      <button type="submit">Додати</button>
    </form>
  );
}


import React, { useContext } from 'react';
import { TaskContext } from '../context/TaskContext';

export default function TaskFilter() {
  const { filter, setFilter } = useContext(TaskContext);

  return (
    <div>
      <button onClick={() => setFilter('all')}>Усі</button>
      <button onClick={() => setFilter('active')}>Активні</button>
      <button onClick={() => setFilter('completed')}>Виконані</button>
    </div>
  );
}


import React, { useContext, useMemo } from 'react';
import { TaskContext } from '../context/TaskContext';
import TaskItem from './TaskItem';

export default function TaskList() {
  const { tasks, filter } = useContext(TaskContext);

  const filteredTasks = useMemo(() => {
    switch (filter) {
      case 'active':
        return tasks.filter(t => !t.completed);
      case 'completed':
        return tasks.filter(t => t.completed);
      default:
        return tasks;
    }
  }, [tasks, filter]);

  return (
    <ul>
      {filteredTasks.map(task => (
        <TaskItem key={task.id} task={task} />
      ))}
    </ul>
  );
}


import React, { useContext, useState, useCallback } from 'react';
import { TaskContext } from '../context/TaskContext';

export default function TaskItem({ task = {} }) {
  const { toggleTask, deleteTask, updateTask } = useContext(TaskContext);
  const [isEditing, setIsEditing] = useState(false);
  const [newText, setNewText] = useState(task.text);

  const handleUpdate = useCallback(() => {
    updateTask(task.id, newText);
    setIsEditing(false);
  }, [task.id, newText, updateTask]);

  return (
    <li>
      <input type="checkbox" checked={task.completed} onChange={() => toggleTask(task.id)} />
      {isEditing ? (
        <>
          <input value={newText} onChange={(e) => setNewText(e.target.value)} />
          <button onClick={handleUpdate}>Зберегти</button>
        </>
      ) : (
        <>
          <span style={{ textDecoration: task.completed ? 'line-through' : 'none' }}>{task.text}</span>
          <button onClick={() => setIsEditing(true)}>Редагувати</button>
        </>
      )}
      <button onClick={() => deleteTask(task.id)}>Видалити</button>
    </li>
  );
}
# -
