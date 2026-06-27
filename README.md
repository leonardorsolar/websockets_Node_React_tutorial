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

Criar uma pasta geral com o nome websocket para adicionar o projeto chat-server(backend) e chat-front(Frontend).

# Parte 1 — Backend (Node.js + Socket.IO)

## 1.1 Criar a pasta do projeto do backend


Crie uma pasta para o servidor backend:

```bash
mkdir chat-server
cd chat-server
```

Instale as dependências necessárias:

```bash
npm init -y
npm install express socket.io
```

### Dependências

- **express** → Framework HTTP para Node.js.
- **socket.io** → Biblioteca para comunicação em tempo real utilizando WebSockets com fallback automático.

---

## 1.2 Criar o arquivo `server.js`

Crie um arquivo chamado **server.js** na raiz do projeto.

Copie o código abaixo e cole no arquivo server.js:

```javascript
// server.js
const express = require('express');
const http = require('http');
const { Server } = require("socket.io");

const app = express();
const server = http.createServer(app); // Cria um servidor HTTP usando o Express

// Configura o Socket.IO para usar o servidor HTTP
const io = new Server(server, {
  cors: { // ESSENCIAL: Configuração de CORS para permitir conexões do seu frontend
    origin: "http://localhost:5173", // Endereço onde seu app React estará rodando
    methods: ["GET", "POST"]
  }
});

const PORTA = process.env.PORT || 3001; // É bom usar uma porta diferente da aplicação React

console.log('🚀 Servidor Socket.IO da Rocketseat preparando para decolar...');

// Evento principal: acontece toda vez que um novo cliente se conecta
io.on('connection', (socket) => { // 'socket' representa a conexão individual com aquele cliente
  console.log(`✨ Novo foguete conectado: ${socket.id}. Olá, Diego! Bem-vindo(a)!`);

  // Ouvindo um evento customizado enviado pelo cliente (ex: 'novaMensagemDoChat')
  socket.on('novaMensagemDoChat', (dados) => {
    // 'dados' é o objeto que o cliente enviou (ex: { autor: 'Laís', texto: 'bora codar!' })
    console.log(`💬 Mensagem de ${dados.autor || 'um Dev anônimo'}: ${dados.texto}`);

    // Agora, vamos retransmitir essa mensagem para TODOS os clientes conectados
    // Usamos io.emit para enviar para todos, inclusive quem mandou a mensagem original
    io.emit('mensagemRecebidaNoChat', {
      autor: dados.autor,
      texto: dados.texto,
      timestamp: new Date() // Adiciona um timestamp no servidor
    });

    // Se quiséssemos enviar para todos, EXCETO o remetente (útil para notificar "Fulano fez X"):
    // socket.broadcast.emit('mensagemRecebidaNoChat', { autor: dados.autor, texto: dados.texto, timestamp: new Date() });
  });

  // Evento que acontece quando o cliente se desconecta
  socket.on('disconnect', () => {
    console.log(`👋 Foguete ${socket.id} voltou para a base. Até logo!`);
  });

  // Podemos também enviar uma mensagem só para o cliente que acabou de conectar
  socket.emit('boasVindas', 'Bem-vindo(a) à plataforma interativa da Rocketseat! Conectado com sucesso.');
});

// Inicia o servidor HTTP (que por sua vez inicia o Socket.IO)
server.listen(PORTA, () => {
  console.log(`🛰️ Servidor Socket.IO no ar na porta ${PORTA}! Pronto para interações em tempo real!`);
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




Instale as dependências necessárias dentro da pasta do projeto websocket.
O react criará a pasta chat-front.


```bash
npm create vite@latest chat-front -- --template react-ts

cd chat-front

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
// src/App.tsx
import React, { useEffect, useRef, useState } from 'react';
import io, { Socket } from 'socket.io-client';

const ENDPOINT = "http://localhost:3001";
let socket: Socket;

interface MensagemChat {
  autor: string;
  texto: string;
  timestamp: string;
}

function App() {
  const [conectado, setConectado] = useState<boolean>(false);
  const [nomeConfirmado, setNomeConfirmado] = useState<boolean>(false);
  const [autorMensagem, setAutorMensagem] = useState<string>('');
  const [nomeTemp, setNomeTemp] = useState<string>('');
  const [textoMensagem, setTextoMensagem] = useState<string>('');
  const [chat, setChat] = useState<MensagemChat[]>([]);
  const chatEndRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    socket = io(ENDPOINT);
    socket.on('connect', () => setConectado(true));
    socket.on('mensagemRecebidaNoChat', (msg: MensagemChat) => {
      setChat((prev) => [...prev, msg]);
    });
    socket.on('disconnect', () => setConectado(false));
    return () => { socket.disconnect(); };
  }, []);

  useEffect(() => {
    chatEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [chat]);

  const confirmarNome = (e: React.FormEvent) => {
    e.preventDefault();
    if (nomeTemp.trim()) {
      setAutorMensagem(nomeTemp.trim());
      setNomeConfirmado(true);
    }
  };

  const enviarMensagem = (e: React.FormEvent) => {
    e.preventDefault();
    if (textoMensagem.trim() && conectado) {
      socket.emit('novaMensagemDoChat', { autor: autorMensagem, texto: textoMensagem });
      setTextoMensagem('');
    }
  };

  const handleKeyDown = (e: React.KeyboardEvent<HTMLTextAreaElement>) => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      if (textoMensagem.trim() && conectado) {
        socket.emit('novaMensagemDoChat', { autor: autorMensagem, texto: textoMensagem });
        setTextoMensagem('');
      }
    }
  };

  return (
    <>
      <style>{`
        *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

        body {
          background: #111b21;
          font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
          height: 100vh;
          display: flex;
          align-items: center;
          justify-content: center;
        }

        /* ── Tela de nome ── */
        .name-screen {
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: center;
          gap: 24px;
          height: 100vh;
          width: 100%;
          padding: 24px;
        }

        .name-card {
          background: #202c33;
          border-radius: 16px;
          padding: 36px 32px;
          width: 100%;
          max-width: 380px;
          display: flex;
          flex-direction: column;
          gap: 20px;
          box-shadow: 0 8px 32px rgba(0,0,0,0.4);
        }

        .name-card .logo {
          font-size: 36px;
          text-align: center;
        }

        .name-card h2 {
          text-align: center;
          color: #e9edef;
          font-size: 20px;
          font-weight: 600;
        }

        .name-card p {
          text-align: center;
          color: #8696a0;
          font-size: 13px;
          line-height: 1.5;
        }

        .name-input {
          background: #2a3942;
          border: 1.5px solid transparent;
          border-radius: 10px;
          color: #e9edef;
          font-size: 15px;
          font-family: inherit;
          padding: 12px 16px;
          outline: none;
          width: 100%;
          transition: border-color 0.2s;
        }
        .name-input::placeholder { color: #8696a0; }
        .name-input:focus { border-color: #00a884; }

        .btn-entrar {
          background: #00a884;
          border: none;
          border-radius: 10px;
          color: #fff;
          font-size: 15px;
          font-weight: 600;
          font-family: inherit;
          padding: 12px;
          cursor: pointer;
          transition: background 0.15s;
        }
        .btn-entrar:hover { background: #02be98; }

        /* ── Layout do chat ── */
        .chat-root {
          width: 100%;
          max-width: 720px;
          height: 100vh;
          display: flex;
          flex-direction: column;
          background: #202c33;
        }

        /* Header */
        .chat-header {
          background: #202c33;
          padding: 10px 16px;
          display: flex;
          align-items: center;
          gap: 12px;
          border-bottom: 1px solid #2a3942;
          flex-shrink: 0;
        }

        .avatar {
          width: 40px;
          height: 40px;
          border-radius: 50%;
          background: #00a884;
          display: flex;
          align-items: center;
          justify-content: center;
          font-size: 18px;
          font-weight: 700;
          color: #fff;
          flex-shrink: 0;
          text-transform: uppercase;
        }

        .header-info strong {
          display: block;
          color: #e9edef;
          font-size: 15px;
          font-weight: 600;
        }

        .header-info small {
          color: #8696a0;
          font-size: 12px;
        }
        .header-info small.online { color: #00a884; }

        /* Wallpaper */
        .chat-messages {
          flex: 1;
          overflow-y: auto;
          padding: 16px 10%;
          display: flex;
          flex-direction: column;
          gap: 3px;
          background-color: #0b141a;
          background-image: url("data:image/svg+xml,%3Csvg width='60' height='60' viewBox='0 0 60 60' xmlns='http://www.w3.org/2000/svg'%3E%3Cg fill='none' fill-rule='evenodd'%3E%3Cg fill='%23ffffff' fill-opacity='0.02'%3E%3Cpath d='M36 34v-4h-2v4h-4v2h4v4h2v-4h4v-2h-4zm0-30V0h-2v4h-4v2h4v4h2V6h4V4h-4zM6 34v-4H4v4H0v2h4v4h2v-4h4v-2H6zM6 4V0H4v4H0v2h4v4h2V6h4V4H6z'/%3E%3C/g%3E%3C/g%3E%3C/svg%3E");
          scrollbar-width: thin;
          scrollbar-color: #2a3942 transparent;
        }
        .chat-messages::-webkit-scrollbar { width: 5px; }
        .chat-messages::-webkit-scrollbar-thumb { background: #2a3942; border-radius: 4px; }

        .date-divider {
          text-align: center;
          margin: 10px 0 6px;
        }
        .date-divider span {
          background: #182229;
          color: #8696a0;
          font-size: 11.5px;
          padding: 5px 12px;
          border-radius: 8px;
        }

        .msg-wrap {
          display: flex;
          margin-bottom: 1px;
        }
        .msg-wrap.local { justify-content: flex-end; }
        .msg-wrap.remote { justify-content: flex-start; }

        .bubble {
          max-width: 65%;
          padding: 7px 10px 6px 10px;
          border-radius: 8px;
          font-size: 14px;
          line-height: 1.4;
          color: #e9edef;
          word-break: break-word;
          position: relative;
        }

        .msg-wrap.local .bubble {
          background: #005c4b;
          border-top-right-radius: 2px;
        }
        .msg-wrap.remote .bubble {
          background: #202c33;
          border-top-left-radius: 2px;
        }

        .bubble-autor {
          font-size: 12px;
          font-weight: 600;
          color: #00a884;
          margin-bottom: 2px;
        }

        .bubble-meta {
          display: flex;
          align-items: center;
          justify-content: flex-end;
          gap: 4px;
          margin-top: 2px;
        }
        .bubble-time {
          font-size: 11px;
          color: #8696a0;
        }
        .bubble-check {
          font-size: 13px;
          color: #53bdeb;
        }

        .empty-state {
          flex: 1;
          display: flex;
          align-items: center;
          justify-content: center;
        }
        .empty-state span {
          background: #182229;
          color: #8696a0;
          font-size: 13px;
          padding: 8px 18px;
          border-radius: 8px;
        }

        /* Input bar */
        .chat-inputbar {
          background: #202c33;
          padding: 10px 16px;
          display: flex;
          align-items: flex-end;
          gap: 10px;
          flex-shrink: 0;
        }

        .emoji-btn {
          background: none;
          border: none;
          font-size: 22px;
          cursor: pointer;
          line-height: 1;
          padding: 4px;
          opacity: 0.7;
          transition: opacity 0.15s;
        }
        .emoji-btn:hover { opacity: 1; }

        .msg-textarea {
          flex: 1;
          background: #2a3942;
          border: none;
          border-radius: 10px;
          color: #e9edef;
          font-size: 15px;
          font-family: inherit;
          padding: 10px 14px;
          outline: none;
          resize: none;
          min-height: 44px;
          max-height: 120px;
          line-height: 1.4;
        }
        .msg-textarea::placeholder { color: #8696a0; }

        .send-btn {
          width: 44px;
          height: 44px;
          background: #00a884;
          border: none;
          border-radius: 50%;
          color: #fff;
          font-size: 18px;
          cursor: pointer;
          display: flex;
          align-items: center;
          justify-content: center;
          transition: background 0.15s;
          flex-shrink: 0;
        }
        .send-btn:hover:not(:disabled) { background: #02be98; }
        .send-btn:disabled { background: #2a3942; color: #8696a0; cursor: not-allowed; }
      `}</style>

      {/* ── Tela de nome ── */}
      {!nomeConfirmado ? (
        <div className="name-screen">
          <form className="name-card" onSubmit={confirmarNome}>
            <div className="logo">💬</div>
            <h2>Bem-vindo ao Chat</h2>
            <p>Qual é o seu nome? Ele vai aparecer para os outros participantes.</p>
            <input
              className="name-input"
              type="text"
              value={nomeTemp}
              onChange={(e) => setNomeTemp(e.target.value)}
              placeholder="Digite seu nome…"
              autoFocus
            />
            <button className="btn-entrar" type="submit">Entrar no chat</button>
          </form>
        </div>
      ) : (
        /* ── Chat ── */
        <div className="chat-root">
          {/* Header */}
          <header className="chat-header">
            <div className="avatar">{autorMensagem.charAt(0)}</div>
            <div className="header-info">
              <strong>{autorMensagem}</strong>
              <small className={conectado ? 'online' : ''}>
                {conectado ? 'online' : 'conectando…'}
              </small>
            </div>
          </header>

          {/* Messages */}
          <div className="chat-messages">
            {chat.length === 0 ? (
              <div className="empty-state">
                <span>Nenhuma mensagem ainda. Diz oi! 👋</span>
              </div>
            ) : (
              <>
                <div className="date-divider"><span>Hoje</span></div>
                {chat.map((msg, i) => {
                  const isLocal = msg.autor === autorMensagem;
                  return (
                    <div key={i} className={`msg-wrap ${isLocal ? 'local' : 'remote'}`}>
                      <div className="bubble">
                        {!isLocal && <div className="bubble-autor">{msg.autor}</div>}
                        {msg.texto}
                        <div className="bubble-meta">
                          <span className="bubble-time">
                            {new Date(msg.timestamp).toLocaleTimeString('pt-BR', { hour: '2-digit', minute: '2-digit' })}
                          </span>
                          {isLocal && <span className="bubble-check">✓✓</span>}
                        </div>
                      </div>
                    </div>
                  );
                })}
              </>
            )}
            <div ref={chatEndRef} />
          </div>

          {/* Input bar */}
          <div className="chat-inputbar">
            <button className="emoji-btn" type="button" title="Emoji">😊</button>
            <textarea
              className="msg-textarea"
              value={textoMensagem}
              onChange={(e) => setTextoMensagem(e.target.value)}
              onKeyDown={handleKeyDown}
              placeholder="Digite uma mensagem"
              rows={1}
            />
            <button
              className="send-btn"
              type="button"
              onClick={enviarMensagem}
              disabled={!conectado || !textoMensagem.trim()}
              title="Enviar"
            >
              ➤
            </button>
          </div>
        </div>
      )}
    </>
  );
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
