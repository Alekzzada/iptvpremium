<!--
  IPTV Premium - Guia Completo Full Stack
  Por: ChatGPT
  
  Este guia e código fornecem uma base para criar um site completo de IPTV com:
  - Frontend em HTML, CSS (Tailwind opcional)
  - Backend com Node.js e Express
  - Banco de dados MongoDB
  - Autenticação JWT
  - Painel de clientes e revendedores
  - Integração com pagamento via PIX/Cartão usando Mercado Pago

  Requisitos:
  - Node.js instalado
  - Conta MongoDB Atlas
  - Conta Mercado Pago com token de desenvolvedor
  - VSCode + Terminal Git
-->

<!--
  ESTRUTURA DE PASTAS
  iptv-premium/
  ├── client/        (Frontend)
  │   ├── index.html
  │   ├── painel.html
  │   ├── login.html
  │   └── js/config.js (API_URL)
  ├── server/       (Backend)
  │   ├── server.js
  │   ├── routes/
  │   ├── models/
  │   ├── controllers/
  │   ├── middleware/
  │   └── .env
  └── README.md
-->

<!-- FRONTEND index.html -->
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>IPTV Premium</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css">
</head>
<body class="bg-gray-900 text-white">
  <div class="text-center p-6">
    <h1 class="text-3xl font-bold mb-4">IPTV Premium</h1>
    <p>+ de 10.000 canais, filmes e séries em HD</p>
    <a href="painel.html" class="mt-4 inline-block bg-blue-500 text-white px-4 py-2 rounded">Acessar Painel</a>
  </div>
</body>
</html>

<!-- client/js/config.js -->
<script>
  const API_URL = "https://seu-backend.onrender.com";
</script>

<!-- login.html - Formulário de login e cadastro -->
<form id="login-form">
  <input name="email" placeholder="Email">
  <input name="senha" type="password" placeholder="Senha">
  <button type="submit">Entrar</button>
</form>
<script>
  document.getElementById('login-form').addEventListener('submit', async (e) => {
    e.preventDefault();
    const form = new FormData(e.target);
    const res = await fetch(`${API_URL}/auth/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        email: form.get('email'),
        senha: form.get('senha')
      })
    });
    const data = await res.json();
    alert(data.token || data.error);
  });
</script>

<!-- SERVER: server.js (resumido) -->
const express = require('express');
const cors = require('cors');
const mongoose = require('mongoose');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(express.json());

mongoose.connect(process.env.MONGO_URI).then(() => console.log('MongoDB conectado'));

app.use('/auth', require('./routes/auth'));
app.use('/revendedor', require('./routes/revendedor'));
app.use('/cliente', require('./routes/cliente'));

app.listen(process.env.PORT || 5000, () => console.log('Servidor rodando'));

<!-- Exemplo de rota: routes/auth.js -->
const router = require('express').Router();
const jwt = require('jsonwebtoken');
const User = require('../models/User');

router.post('/login', async (req, res) => {
  const { email, senha } = req.body;
  const user = await User.findOne({ email });
  if (!user || user.senha !== senha) return res.status(401).json({ error: 'Credenciais inválidas' });
  const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET);
  res.json({ token });
});

module.exports = router;

<!-- MongoDB model: models/User.js -->
const mongoose = require('mongoose');
const UserSchema = new mongoose.Schema({
  email: String,
  senha: String,
  tipo: { type: String, enum: ['cliente', 'revendedor'], default: 'cliente' }
});
module.exports = mongoose.model('User', UserSchema);
