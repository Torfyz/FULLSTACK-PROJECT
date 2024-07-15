# FullStack Project

## Descrição do Repositório

Este repositório contém um projeto fullStack desenvolvido com Fastify no backend e React no frontend, utilizando Tailwind CSS para estilização.

### Backend

O backend é construído usando Fastify e TypeScript, fornecendo uma API para gerenciamento de clientes. As principais funcionalidades incluem a criação, listagem e exclusão de clientes. Abaixo está uma visão geral dos principais arquivos do backend:

#### `server.ts`

Este é o ponto de entrada principal do servidor backend. Ele realiza as seguintes tarefas:
- Importa os módulos necessários, como Fastify, rotas e CORS.
- Configura um manipulador de erros para lidar com erros de maneira adequada.
- Registra as rotas e o middleware CORS.
- Inicia o servidor na porta 3333.

```typescript
import Fastify from 'fastify'
import { routes } from './routes'
import cors from '@fastify/cors'

const app = Fastify({ logger: true })

app.setErrorHandler((error, request, reply) => {
    reply.code(400).send({ message: error.message })
})

const start = async () => {
    await app.register(routes)
    await app.register(cors)

    try {
        await app.listen({ port: 3333 })
    } catch (err) {
        process.exit(1)
    }
}

start();
```

#### `routes.ts`

Define as rotas da API para criação, listagem e exclusão de clientes.

```typescript
import { FastifyInstance, FastifyPluginOptions, FastifyRequest, FastifyReply } from "fastify";
import { CreateCustomerController } from './controllers/CreateCustomerController'
import { ListCustomerController } from './controllers/ListCustomerController'
import { DeleteCustomerController } from './controllers/DeleteCustomerController'

export async function routes(fastify: FastifyInstance, option: FastifyPluginOptions) {
    fastify.get("/teste", async (request: FastifyRequest, reply: FastifyReply) => {
        return { ok: true }
    })

    fastify.post("/customer", async (request: FastifyRequest, reply: FastifyReply) => {
        return new CreateCustomerController().handle(request, reply)
    })

    fastify.get("/customers", async (request: FastifyRequest, reply: FastifyReply) => {
        return new ListCustomerController().handle(request, reply)
    })

    fastify.delete("/customer", async (request: FastifyRequest, reply: FastifyReply) => {
        return new DeleteCustomerController().handle(request, reply)
    })
}
```

#### `package.json`

Contém as dependências e scripts necessários para rodar o backend.

```json
{
  "name": "backend",
  "version": "1.0.0",
  "main": "server.ts",
  "scripts": {
    "dev": "ts-node-dev --respawn --transpile-only server.ts",
    "build": "tsc",
    "start": "node dist/server.js"
  },
  "dependencies": {
    "fastify": "^4.0.0",
    "cors": "^2.8.5",
    "@fastify/cors": "^7.2.0",
    "typescript": "^4.3.5"
  },
  "devDependencies": {
    "ts-node-dev": "^1.1.8",
    "tslint": "^6.1.3",
    "typescript": "^4.3.5"
  }
}
```

### Frontend

O frontend é desenvolvido com React e TypeScript, utilizando Tailwind CSS para estilização, fornecendo uma interface para gerenciar os clientes. Abaixo está uma visão geral dos principais arquivos do frontend:

#### `index.html`

O ponto de entrada HTML do aplicativo React.

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>JorgeApp</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

#### `App.tsx`

O componente principal do React que gerencia os clientes.

```typescript
import { useEffect, useState, useRef, FormEvent } from 'react'
import { FiTrash } from 'react-icons/fi'
import { api } from './services/api'

interface CustomerProps {
  id: string;
  name: string;
  email: string;
  status: boolean;
  created_at: string;
}

export default function App() {
  const [customers, setCustomers] = useState<CustomerProps[]>([])
  const nameRef = useRef<HTMLInputElement | null>(null)
  const emailRef = useRef<HTMLInputElement | null>(null)

  useEffect(() => {
    loadCustomers()
  }, [])

  async function loadCustomers() {
    const response = await api.get("/customers")
    setCustomers(response.data)
  }

  async function handleSubmit(event: FormEvent) {
    event.preventDefault()

    if (!nameRef.current?.value || !emailRef.current?.value) return

    const response = await api.post("/customer", {
      name: nameRef.current?.value,
      email: emailRef.current?.value
    })

    setCustomers(allCustomers => [...allCustomers, response.data])
    nameRef.current.value = ""
    emailRef.current.value = ""
  }

  async function handleDelete(id: string) {
    try {
      await api.delete("/customer", {
        params: { id }
      })
      const allCustomers = customers.filter(customer => customer.id !== id)
      setCustomers(allCustomers)
    } catch (error) {
      console.log("error")
    }
  }

  return (
    <div className="w-full min-h-screen bg-gray-900 flex justify-center px-4">
      <main className="my-10 w-full md:max-w-2xl">
        <h1 className="text-4xl font-medium text-white">Clientes</h1>
        <form className="flex flex-col my-6" onSubmit={handleSubmit}>
          <label className="font-medium text-white">Nome:</label>
          <input type="text" placeholder="Digite Seu nome completo..." className="w-full mb-5 p-2 rounded" ref={nameRef} />
          <label className="font-medium text-white">Email:</label>
          <input type="text" placeholder="Digite Seu email..." className="w-full mb-5 p-2 rounded" ref={emailRef} />
          <input type="submit" value="Cadastrar" className="cursor-pointer hover:scale-105 duration-200 w-full p-2 bg-green-500 rounded font-medium" />
        </form>
        <section className="flex flex-col gap-4 relative">
          {customers.map(customer => (
            <article key={customer.id} className="w-full bg-white rounded p-2 hover:scale-105 duration-200">
              <p><span className="font-medium">Nome:</span> {customer.name}</p>
              <p><span className="font-medium">Email:</span> {customer.email}</p>
              <p><span className="font-medium">Status:</span> {customer.status ? "ATIVO" : "INATIVO"}</p>
              <button className="bg-red-500 w-7 h-7 flex items-center justify-center rounded-lg right-2 -top-2 hover:scale-110 duration-200" onClick={() => handleDelete(customer.id)}>
                <FiTrash size={18}></FiTrash>
              </button>
            </article>
          ))}
        </section>
      </main>
    </div>
  )
}
```

#### `main.tsx`

O ponto de entrada para o aplicativo React.

```typescript
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.tsx'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

#### `index.css`

Configuração do Tailwind CSS.

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

#### `tailwind.config.js`

Configuração do Tailwind CSS.

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

#### `package.json`

Contém as dependências e scripts necessários para rodar o frontend.

```json
{
  "name": "frontend",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "preview": "vite preview"
  },
  "dependencies": {
    "axios": "^1.7.2",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "react-icons": "^5.2.1"
  },
  "devDependencies": {
    "@types/react": "^18.3.3",
    "@types/react-dom": "^18.3.0",
    "@typescript-eslint

/eslint-plugin": "^7.13.1",
    "@typescript-eslint/parser": "^7.13.1",
    "@vitejs/plugin-react": "^4.3.1",
    "autoprefixer": "^10.4.19",
    "eslint": "^8.57.0",
    "eslint-plugin-react-hooks": "^4.6.2",
    "eslint-plugin-react-refresh": "^0.4.7",
    "postcss": "^8.4.39",
    "tailwindcss": "^3.4.4",
    "typescript": "^5.2.2",
    "vite": "^5.3.1"
  }
}
```

Este projeto fornece uma base robusta para um aplicativo de gerenciamento de clientes, com um backend eficiente e uma interface de usuário interativa e responsiva, estilizada com Tailwind CSS.
