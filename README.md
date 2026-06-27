# websockets_Node_React_tutorial

## Tutorial Prático — Socket.IO

### Do servidor ao chat em tempo real

## 📚 O que você vai aprender

Neste tutorial você aprenderá a criar um **chat em tempo real** utilizando **WebSockets** com **Socket.IO**, conectando um backend em **Node.js** a um frontend em **React**.

Ao final, você terá:

- Um servidor Node.js utilizando Express e Socket.IO.
- Um frontend React (Vite + TypeScript) conectado via WebSocket.
- Comunicação em tempo real entre vários clientes.

---

# Parte 1 — Backend (Node.js + Socket.IO)

## 1.1 Criar a pasta do projeto

Crie uma pasta para o servidor e instale as dependências necessárias.

```bash
mkdir chat-server
cd chat-server
npm init -y
npm install express socket.io
```

### Dependências

- **express** → Framework HTTP para Node.js.
- **socket.io** → Biblioteca para comunicação em tempo real utilizando WebSockets com fallback automático.

---

## 1.2 Criar o arquivo `server.js`

Crie um arquivo chamado **server.js** na raiz do projeto.

```javascript
// server.js

const express = require('express');
const http = require('http');
const { Server } = require('socket.io');

const app = express();
const server = http.createServer(app);

const io = new Server(server, {
  cors: {
    origin: 'http://localhost:5173',
    methods: ['GET', 'POST']
  }
});

const PORTA = process.env.PORT || 3001;

io.on('connection', (socket) => {

  console.log(`Cliente conectado: ${socket.id}`);

  socket.emit('boasVindas', 'Conectado com sucesso!');

  socket.on('novaMensagemDoChat', (dados) => {

    io.emit('mensagemRecebidaNoChat', {
      autor: dados.autor,
      texto: dados.texto,
      timestamp: new Date()
    });

  });

  socket.on('disconnect', () => {
    console.log(`Cliente desconectado: ${socket.id}`);
  });

});

server.listen(PORTA, () => {
  console.log(`Servidor rodando na porta ${PORTA}`);
});
```

> **Observação**
>
> O CORS deve apontar para o endereço onde o React será executado.
>
> No Vite, o padrão é:
>
> `http://localhost:5173`

---

## 1.3 Iniciar o servidor

Execute:

```bash
node server.js
```

Saída esperada:

```text
Servidor rodando na porta 3001
```

> **Importante**
>
> Mantenha esse terminal aberto durante o desenvolvimento.

---

# Parte 2 — Frontend (React + Vite + TypeScript)

## 2.1 Criar o projeto React

Em outra pasta:

```bash
npm create vite@latest my-app -- --template react-ts

cd my-app

npm install

npm install socket.io-client
```

---

## 2.2 Criar o componente principal

Abra:

```
src/App.tsx
```

Substitua todo o conteúdo pelo código abaixo.

```tsx
import React, { useEffect, useRef, useState } from 'react';
import io, { Socket } from 'socket.io-client';

const ENDPOINT = 'http://localhost:3001';

let socket: Socket;

interface MensagemChat {
  autor: string;
  texto: string;
  timestamp: string;
}

function App() {

  const [conectado, setConectado] = useState(false);
  const [nomeConfirmado, setNomeConfirmado] = useState(false);
  const [autorMensagem, setAutorMensagem] = useState('');
  const [nomeTemp, setNomeTemp] = useState('');
  const [textoMensagem, setTextoMensagem] = useState('');
  const [chat, setChat] = useState<MensagemChat[]>([]);

  const chatEndRef = useRef<HTMLDivElement>(null);

  useEffect(() => {

    socket = io(ENDPOINT);

    socket.on('connect', () => {
      setConectado(true);
    });

    socket.on('mensagemRecebidaNoChat', (msg) => {
      setChat((prev) => [...prev, msg]);
    });

    socket.on('disconnect', () => {
      setConectado(false);
    });

    return () => {
      socket.disconnect();
    };

  }, []);

  useEffect(() => {
    chatEndRef.current?.scrollIntoView({
      behavior: 'smooth'
    });
  }, [chat]);

  // Código restante omitido...
}

export default App;
```

> O restante do código (layout, CSS e tela de login) pode ser adicionado normalmente conforme o projeto.

---

## 2.3 Executar o frontend

Com o servidor já em execução, abra um novo terminal.

Execute:

```bash
npm run dev
```

Depois acesse:

```
http://localhost:5173
```

---

# Como funciona a comunicação

O fluxo de uma mensagem acontece da seguinte forma.

## 1. O cliente envia uma mensagem

```javascript
socket.emit('novaMensagemDoChat', {
    autor,
    texto
});
```

↓

## 2. O servidor recebe

```javascript
socket.on('novaMensagemDoChat', (dados) => {

    io.emit('mensagemRecebidaNoChat', {
        autor: dados.autor,
        texto: dados.texto,
        timestamp: new Date()
    });

});
```

↓

## 3. Todos os clientes recebem

```javascript
socket.on('mensagemRecebidaNoChat', (msg) => {

    setChat((prev) => [...prev, msg]);

});
```

---

# Fluxo completo

```text
Cliente A
     │
     │ emit()
     ▼
Servidor Socket.IO
     │
     │ io.emit()
     ▼
 ┌──────────────┬──────────────┬──────────────┐
 │              │              │
 ▼              ▼              ▼
Cliente A    Cliente B     Cliente C
```

---

# Eventos do Socket.IO

| Evento | Origem | Destino | Descrição |
|---------|---------|----------|-----------|
| `connect` | Servidor | Cliente | Conexão estabelecida |
| `boasVindas` | Servidor | Cliente | Mensagem enviada ao conectar |
| `novaMensagemDoChat` | Cliente | Servidor | Envia uma nova mensagem |
| `mensagemRecebidaNoChat` | Servidor | Todos os clientes | Distribui a mensagem |
| `disconnect` | Servidor | Cliente | Conexão encerrada |

---

# Arquitetura do projeto

```text
                React (Vite)

        socket.emit(...)
               │
               ▼
        WebSocket (Socket.IO)
               │
               ▼
      Node.js + Express Server
               │
      socket.on(...)
               │
               ▼
          io.emit(...)
               │
     ┌─────────┴─────────┐
     ▼                   ▼
 Cliente 1           Cliente 2
```

---

# Próximos passos

Após concluir este projeto, você pode evoluí-lo adicionando:

- Salas de conversa utilizando `socket.join("sala")`.
- Persistência das mensagens em **MongoDB** ou **PostgreSQL**.
- Indicador de usuários digitando ("digitando...").
- Autenticação utilizando **JWT**.
- Upload de imagens e arquivos.
- Lista de usuários online.
- Histórico das conversas.
- Deploy do backend em serviços como Railway ou Render.

---

# Resumo

Neste tutorial você aprendeu a:

- Criar um servidor Node.js utilizando Socket.IO.
- Configurar um frontend React com Vite.
- Conectar cliente e servidor utilizando WebSockets.
- Enviar e receber mensagens em tempo real.
- Compreender o fluxo de eventos entre cliente e servidor.

---

**Tutorial criado para fins didáticos — WebSockets com Node.js e React utilizando Socket.IO.**
