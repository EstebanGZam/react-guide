# 🚀 Guía de Supervivencia React - Examen Frontend

## ✅ 1. Inicializar y preparar el entorno

### Crear proyecto con Vite (RECOMENDADO - más rápido que Create React App)
```bash
# Crear proyecto
npm create vite@latest mi-proyecto -- --template react
cd mi-proyecto
npm install

# Iniciar desarrollo
npm run dev
```

### Instalar dependencias esenciales
```bash
# Bootstrap (MÁS FÁCIL para exámenes)
npm install bootstrap

# O Material-UI (si te sientes cómodo)
npm install @mui/material @emotion/react @emotion/styled

# Axios para APIs
npm install axios

# ESLint (si no viene configurado)
npm install -D eslint @eslint/js eslint-plugin-react
```

### Configurar Bootstrap en main.jsx
```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import 'bootstrap/dist/css/bootstrap.min.css' // ← AGREGAR ESTO
import App from './App.jsx'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

### ESLint básico (crear .eslintrc.json)
```json
{
  "extends": ["eslint:recommended", "plugin:react/recommended"],
  "parserOptions": {
    "ecmaVersion": 2021,
    "sourceType": "module",
    "ecmaFeatures": {
      "jsx": true
    }
  },
  "rules": {
    "react/prop-types": "warn",
    "react/react-in-jsx-scope": "off",
    "no-unused-vars": "warn",
    "react/jsx-key": "error"
  },
  "settings": {
    "react": {
      "version": "detect"
    }
  }
}
```

### Comandos útiles
```bash
npm run dev          # Iniciar desarrollo
npm run build        # Build para producción
npm run lint         # Revisar errores
npm run preview      # Ver build localmente
```

---

## 📁 2. Estructura de carpetas

```
src/
├── assets/          # Imágenes, iconos, fuentes
│   ├── images/
│   └── icons/
├── components/      # Componentes reutilizables
│   ├── common/     # Button, Modal, Loading
│   └── forms/      # Input, Form, Validation
├── pages/          # Páginas/Vistas principales
│   ├── Home.jsx
│   ├── About.jsx
│   └── Contact.jsx
├── hooks/          # Custom hooks
│   ├── useApi.js
│   └── useLocalStorage.js
├── context/        # Context API
│   └── AppContext.jsx
├── services/       # APIs y servicios externos
│   ├── api.js
│   └── authService.js
├── utils/          # Funciones utilitarias
│   └── helpers.js
├── App.jsx
├── main.jsx
└── index.css
```

### Qué va en cada carpeta:

**components/**: Componentes que se reusan (Header, Button, Card)
```jsx
// components/common/Button.jsx
export default function Button({ children, onClick, variant = "primary" }) {
  return (
    <button className={`btn btn-${variant}`} onClick={onClick}>
      {children}
    </button>
  )
}
```

**pages/**: Vistas completas de tu aplicación
```jsx
// pages/Home.jsx
import Button from '../components/common/Button'

export default function Home() {
  return (
    <div className="container">
      <h1>Home Page</h1>
      <Button onClick={() => alert('Hola!')}>Click me</Button>
    </div>
  )
}
```

**services/**: Llamadas a APIs
```jsx
// services/api.js
import axios from 'axios'

const API_BASE = 'https://jsonplaceholder.typicode.com'

export const api = axios.create({
  baseURL: API_BASE,
  timeout: 10000
})

export const getUsers = () => api.get('/users')
export const getUser = (id) => api.get(`/users/${id}`)
```

---

## 📚 3. Conceptos clave para usar bajo presión

### Promesas - Para APIs y operaciones asíncronas
```jsx
// ✅ CORRECTO - con async/await
const fetchData = async () => {
  try {
    const response = await fetch('/api/data')
    const data = await response.json()
    setData(data)
  } catch (error) {
    setError(error.message)
  }
}

// ✅ CORRECTO - con .then()
fetch('/api/data')
  .then(response => response.json())
  .then(data => setData(data))
  .catch(error => setError(error.message))
```

### useState - Manejo de estado
```jsx
import { useState } from 'react'

function Counter() {
  const [count, setCount] = useState(0)
  const [user, setUser] = useState({ name: '', email: '' })

  // ❌ INCORRECTO - no muta el estado directamente
  // count++  
  // user.name = 'Juan'

  // ✅ CORRECTO
  const increment = () => setCount(count + 1)
  const updateUser = (name) => setUser({...user, name})
  
  // ✅ CORRECTO - función para estados complejos
  const reset = () => setCount(prev => 0)
}
```

### useEffect - Efectos secundarios
```jsx
import { useState, useEffect } from 'react'

function UserProfile({ userId }) {
  const [user, setUser] = useState(null)
  const [loading, setLoading] = useState(true)

  // ✅ Cargar datos al montar
  useEffect(() => {
    fetchUser(userId)
  }, []) // Array vacío = solo al montar

  // ✅ Recargar cuando cambie userId
  useEffect(() => {
    fetchUser(userId)
  }, [userId]) // Se ejecuta cuando userId cambia

  // ✅ Cleanup (limpieza)
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('Timer running')
    }, 1000)

    return () => clearInterval(timer) // Limpia al desmontar
  }, [])

  const fetchUser = async (id) => {
    setLoading(true)
    try {
      const response = await api.get(`/users/${id}`)
      setUser(response.data)
    } catch (error) {
      console.error('Error:', error)
    } finally {
      setLoading(false)
    }
  }
}
```

### useRef - Referencias directas
```jsx
import { useRef, useEffect } from 'react'

function FocusInput() {
  const inputRef = useRef(null)

  useEffect(() => {
    inputRef.current.focus() // Enfoca el input al cargar
  }, [])

  return <input ref={inputRef} type="text" />
}
```

### Context API - Estado global simple
```jsx
// context/AppContext.jsx
import { createContext, useContext, useState } from 'react'

const AppContext = createContext()

export function AppProvider({ children }) {
  const [user, setUser] = useState(null)
  const [theme, setTheme] = useState('light')

  const login = (userData) => setUser(userData)
  const logout = () => setUser(null)
  const toggleTheme = () => setTheme(theme === 'light' ? 'dark' : 'light')

  return (
    <AppContext.Provider value={{
      user, theme, login, logout, toggleTheme
    }}>
      {children}
    </AppContext.Provider>
  )
}

export const useApp = () => {
  const context = useContext(AppContext)
  if (!context) {
    throw new Error('useApp debe usarse dentro de AppProvider')
  }
  return context
}
```

```jsx
// App.jsx
import { AppProvider } from './context/AppContext'
import Home from './pages/Home'

function App() {
  return (
    <AppProvider>
      <Home />
    </AppProvider>
  )
}
```

### Formularios - Controlados vs No controlados
```jsx
// ✅ CONTROLADO (recomendado para exámenes)
function ContactForm() {
  const [form, setForm] = useState({
    name: '',
    email: '',
    message: ''
  })

  const handleChange = (e) => {
    setForm({
      ...form,
      [e.target.name]: e.target.value
    })
  }

  const handleSubmit = (e) => {
    e.preventDefault()
    console.log('Form data:', form)
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"
        value={form.name}
        onChange={handleChange}
        placeholder="Nombre"
        required
      />
      <input
        name="email"
        type="email"
        value={form.email}
        onChange={handleChange}
        placeholder="Email"
        required
      />
      <textarea
        name="message"
        value={form.message}
        onChange={handleChange}
        placeholder="Mensaje"
      />
      <button type="submit">Enviar</button>
    </form>
  )
}
```

### Consumir APIs con Axios
```jsx
// hooks/useApi.js - Custom hook para APIs
import { useState, useEffect } from 'react'
import axios from 'axios'

export function useApi(url) {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true)
        const response = await axios.get(url)
        setData(response.data)
      } catch (err) {
        setError(err.message)
      } finally {
        setLoading(false)
      }
    }

    fetchData()
  }, [url])

  return { data, loading, error }
}

// Uso en componente
function UserList() {
  const { data: users, loading, error } = useApi('/api/users')

  if (loading) return <div>Cargando...</div>
  if (error) return <div>Error: {error}</div>

  return (
    <ul>
      {users?.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

### Errores comunes en React - EVÍTALOS

#### 1. Missing key en listas
```jsx
// ❌ INCORRECTO
users.map(user => <li>{user.name}</li>)

// ✅ CORRECTO
users.map(user => <li key={user.id}>{user.name}</li>)
```

#### 2. Renders infinitos
```jsx
// ❌ INCORRECTO - causa render infinito
useEffect(() => {
  setCount(count + 1)
}) // Sin array de dependencias

// ✅ CORRECTO
useEffect(() => {
  // Solo ejecutar una vez
}, []) // Array vacío
```

#### 3. Mutación directa del estado
```jsx
// ❌ INCORRECTO
const addItem = () => {
  items.push(newItem) // Muta directamente
  setItems(items)
}

// ✅ CORRECTO
const addItem = () => {
  setItems([...items, newItem]) // Nuevo array
}
```

---

## ⚙️ 4. JavaScript vs TypeScript

**RECOMENDACIÓN PARA TU EXAMEN: USA JAVASCRIPT PURO**

### Razones:
- **Menos sintaxis** que memorizar bajo presión
- **Más rápido** para prototipar
- **Menos errores** de tipos que te bloqueen
- **Más flexible** para cambios rápidos

### Si usaras TypeScript, lo básico sería:
```tsx
// Solo si te sientes MUY cómodo
interface User {
  id: number;
  name: string;
  email: string;
}

const UserCard: React.FC<{ user: User }> = ({ user }) => {
  return <div>{user.name}</div>
}
```

**Pero para el examen, JavaScript es tu mejor opción.**

---

## 🧠 5. Consejos finales

### Buenas prácticas para trabajar rápido
1. **Usa Bootstrap** para estilos rápidos: `className="btn btn-primary"`
2. **Componentiza desde el inicio**: no repitas código
3. **useState simple**: evita objetos complejos al principio
4. **Console.log todo**: debuggea constantemente
5. **Usa snippet de VSCode**: `rafce` para crear componentes rápido

### Errores típicos de novato
1. **No usar key** en listas → React se queja
2. **Olvidar preventDefault** en formularios → página se recarga
3. **useEffect sin dependencias** → renders infinitos
4. **No manejar loading states** → UX horrible
5. **Hardcodear valores** → código inflexible

### Tips para gestión de tiempo en examen
1. **Primero estructura**: componentes básicos sin estilo
2. **Luego funcionalidad**: hooks, estado, APIs
3. **Después estilo**: Bootstrap classes, CSS básico
4. **Por último**: validaciones y edge cases

### Scripts útiles para package.json
```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "lint": "eslint . --ext js,jsx",
    "lint:fix": "eslint . --ext js,jsx --fix",
    "preview": "vite preview"
  }
}
```

### VSCode sin autocompletado IA
1. **Emmet**: `div.container>h1+p` → expande automáticamente
2. **Snippets**: `rafce`, `useState`, `useEffect`
3. **Ctrl+D**: seleccionar múltiples ocurrencias
4. **Ctrl+Shift+L**: seleccionar todas las ocurrencias
5. **Alt+Shift+Down**: duplicar línea
6. **Ctrl+/**: comentar/descomentar

### Atajos de código útiles
```jsx
// Snippet rápido para componente
const ComponentName = () => {
  return (
    <div>
      
    </div>
  )
}
export default ComponentName

// Snippet para useState
const [state, setState] = useState(initialValue)

// Snippet para useEffect
useEffect(() => {
  // effect
  return () => {
    // cleanup
  }
}, [])
```

---

## 🎯 TEMPLATE RÁPIDO PARA EMPEZAR

### App.jsx básico
```jsx
import { useState } from 'react'
import './App.css'

function App() {
  const [count, setCount] = useState(0)

  return (
    <div className="container mt-5">
      <div className="row justify-content-center">
        <div className="col-md-6">
          <div className="card">
            <div className="card-body text-center">
              <h1 className="card-title">Mi App React</h1>
              <p className="card-text">Contador: {count}</p>
              <button 
                className="btn btn-primary"
                onClick={() => setCount(count + 1)}
              >
                Incrementar
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  )
}

export default App
```

---

## 🚨 CHECKLIST FINAL PARA EL EXAMEN

### Antes de empezar:
- [ ] Proyecto creado con Vite
- [ ] Bootstrap instalado e importado
- [ ] ESLint configurado
- [ ] Estructura de carpetas lista

### Durante el desarrollo:
- [ ] Componentes con key únicos en listas
- [ ] Estado manejado correctamente (no mutación)
- [ ] useEffect con dependencias correctas
- [ ] Formularios controlados
- [ ] Validación básica de datos
- [ ] Loading states en APIs
- [ ] Error handling básico

### Antes de entregar:
- [ ] `npm run lint` sin errores
- [ ] `npm run build` exitoso
- [ ] Funcionalidad básica probada
- [ ] No console.errors en browser

---

## 💡 RECURSOS DE EMERGENCIA

### Si algo no funciona:
1. **Revisa la consola** del browser (F12)
2. **Lee el error completo** (no solo la primera línea)
3. **Verifica imports/exports** (case-sensitive)
4. **Comprueba dependencias** en useEffect
5. **Reinicia el servidor** (Ctrl+C → npm run dev)

### Comandos de rescate:
```bash
# Si algo se rompe
rm -rf node_modules package-lock.json
npm install

# Si el puerto está ocupado
kill -9 $(lsof -t -i:5173)
npm run dev
```

**¡ÉXITO EN TU EXAMEN! 🚀**
