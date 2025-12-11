
# 🧰 AN Estoque – Sistema Completo de Gestão de TI

> App desenvolvido em **Python + Tkinter + ttkbootstrap**, integrado ao mysql, com módulos de estoque, inventário, produtos, impressoras, backup automático, envio de e-mail etcc...

---

# 📌 Sobre o Projeto

O **AN Estoque** foi desenvolvido especialmente para o Hospital Israelita Albert Sabin, com o objetivo de facilitar e organizar a gestão de todos os ativos e recursos de TI utilizados pela instituição.

- ✔️ Cadastro de produtos  
- ✔️ Controle de estoque com filtros, categorias e estados  
- ✔️ Movimentações (entrada, baixa, transferência)  
- ✔️ Inventário de computadores  
- ✔️ Impressoras com IP, setor, tipo e status  
- ✔️ Redes Wi-Fi  
- ✔️ Logs detalhados  
- ✔️ Backup automático configurável  
- ✔️ Envio automático de e-mail ao atingir estoque mínimo  
- ✔️ Sistema de login  
- ✔️ Exportação para Excel  
- ✔️ Interface moderna e responsiva  

---


# 🖼️ Prints do Sistema

## 🗄️ Config. Banco de Dados
![config banco](https://github.com/user-attachments/assets/f24cf5e5-c51b-4ed0-b604-c882ff4a11e8)

## 🔑 Tela de Login
![logar](https://github.com/user-attachments/assets/aee9281d-7c0d-44d5-ab5f-46cc86fe6a46)

## 🆕 Criar Conta
![criar conta](https://github.com/user-attachments/assets/59cc43a2-e29a-486f-bc59-4896266b9dd1)

## 🏠 Tela Inicial
![inicial](https://github.com/user-attachments/assets/fefa8866-61ae-456b-aff9-2d5577b797e4)

## 🛒 Produtos
![produtos](https://github.com/user-attachments/assets/825f4e6f-ce80-461b-a63d-34c54ffea68b)

## 📦 Estoque
![estoque-produtos](https://github.com/user-attachments/assets/a1969928-fbb0-49a1-a023-5b13941c1007)

## 🔄 Movimentações
![movimentacoes](https://github.com/user-attachments/assets/0a28748a-7642-4550-8502-f0772b818312)

## 🖥️ Inventário de PCs
![inventario](https://github.com/user-attachments/assets/ab3ea175-9132-471e-b107-076dbff16731)

## 🖨️ Impressoras
![impressoras](https://github.com/user-attachments/assets/c3dab5bc-768e-4216-98f0-9b745c02f61e)

## 📶 Redes Wi-Fi
![wifi](https://github.com/user-attachments/assets/b2042536-3026-4084-81bf-f770ed4b13ba)

## 🏢 Setores
![setores](https://github.com/user-attachments/assets/7f7ffec1-52c3-46fd-ace2-af49d417d6ea)

## ⚙️ Configurações
![configuracoes](https://github.com/user-attachments/assets/d54dc7eb-083c-4ff2-9bed-1083d6733074)

## 📜 Logs
![logs](https://github.com/user-attachments/assets/f4529a9a-2fe9-4b03-877b-4ec8ffde7720)

## 📧 E-mail de Alerta – Estoque Baixo
![email alerta](https://github.com/user-attachments/assets/0c3690a7-167d-4e89-b840-a96ccea273d8)

## 📓 Sobre
![sobre](https://github.com/user-attachments/assets/9c012430-3262-40e0-87e6-950d86f1f500)


## 🗃️ Banco MySQL
![banco mysql](https://github.com/user-attachments/assets/45ce8b6a-1c46-4cc7-8a95-58d8548491ec)


---


---

# 🏗️ Arquitetura do Projeto

```
/core
    database.py
    db_config.py
    log_utils.py

/ui
    produtos.py
    estoque.py
    movimentacoes.py
    inventario.py
    impressoras.py
    wifi.py
    setores.py
    config.py
    sobre.py
    inicial.py
    login.py

main.py
requirements.txt
README.md
```

---

# 🛠️ Tecnologias Utilizadas

| Tecnologia | Uso |
|-----------|-----|
| Python 3 | Linguagem principal |
| Tkinter | Interface gráfica |
| ttkbootstrap | UI moderna, estilo dark |
| MySQL Connector | Banco de dados |
| OpenPyXL | Geração de planilhas Excel |
| Cryptography (Fernet) | Criptografia da senha do banco |
| JSON | Arquivos de configuração |
| subprocess | Execução de mysqldump no backup |

---

# 🔧 Instalação

### 1️⃣ Instale as dependências:

```
pip install -r requirements.txt
```

### 2️⃣ Execute o sistema:

```
python main.py
```

Ao abrir pela primeira vez, o sistema solicitará a configuração do mysql.

<img width="471" height="495" alt="config_banco" src="https://github.com/user-attachments/assets/6cbd9620-e774-4b0a-bdee-7903e06b61ac" />

---

# 🔐 Parte do Código 

```python
def agendar_backup(self):
    try:
        intervalo = int(self.ent_tempo_backup.get().strip())
    except ValueError:
        intervalo = 30

    if hasattr(self, "_backup_job"):
        try:
            self.master.after_cancel(self._backup_job)
        except:
            pass

    self._backup_job = self.master.after(intervalo * 60000, self.fazer_backup_db)
```

```python
def fazer_backup_db(self):
    try:
        mysqldump = localizar_mysqldump()
        pasta = self.var_backup_pasta.get()
        os.makedirs(pasta, exist_ok=True)

        nome = datetime.datetime.now().strftime("backup_%Y-%m-%d_%H-%M-%S.sql")
        destino = os.path.join(pasta, nome)

        comando = [
            mysqldump,
            f"-u{MYSQL_CONFIG['user']}",
            f"-p{MYSQL_CONFIG['password']}",
            MYSQL_CONFIG["database"],
        ]

        with open(destino, "w", encoding="utf-8") as f:
            subprocess.run(comando, stdout=f, stderr=subprocess.PIPE, text=True)

        atualizar_ultimo_backup(datetime.datetime.now())
        registrar_acao(f"Backup criado: {destino}")

    except Exception as e:
        messagebox.showerror("Erro", f"Falha ao criar backup:\n{e}")
```

```python
def enviar_alerta_estoque(self, manual=False):
    itens = listar_estoque()
    alerta = int(self.ent_qtd_alerta.get())

    produtos_baixos = [i for i in itens if i["quantidade"] <= alerta]

    if not produtos_baixos and not manual:
        return

    msg = "Itens com estoque baixo:\\n\\n"
    for p in produtos_baixos:
        msg += f"- {p['produto_nome']} ({p['quantidade']} unidades)\\n"

    self._enviar_email("⚠️ Alerta de Estoque Baixo", msg)

```

```python
def add_item(self):
    nome = self.cmb_item_nome.get().strip()
    qtd = self.ent_item_qtd.get().strip()
    estado = self.cmb_item_estado.get().strip()

    if not nome or not qtd:
        messagebox.showwarning("Campos obrigatórios", "Preencha todos os campos.")
        return

    qtd = int(qtd)
    produtos = listar_produtos()
    ref = next((p for p in produtos if p["nome"] == nome), None)

    categoria = ref["categoria"]

    status, valor = adicionar_ou_atualizar_item(nome, categoria, qtd, estado)
    registrar_movimentacao(nome, qtd, "-", "Entrada", self.usuario_logado, estado)

```
```python
def autenticar_usuario(email: str, senha: str):
    conn = conectar()
    cur = conn.cursor(dictionary=True)
    cur.execute(
        "SELECT id, nome, email, nivel FROM usuarios WHERE email=%s AND senha_hash=%s",
        (email.lower(), hash_senha(senha)),
    )
    return cur.fetchone()

```
---

# 📬 Contato

📧 **E-mail:** andrey.mag1909@gmail.com  
💬 **Discord:** almeidss021.180hz


Feito para estudo, prática e organização profissional.

---

# 🔒 Aviso Legal

Este projeto foi desenvolvido para fins profissionais e educacionais.  
Todo o código-fonte, layout, lógica e estrutura pertencem ao autor e **não estão autorizados para cópia, redistribuição ou uso comercial sem permissão**.

📌 **Código privado. Todos os direitos reservados.**

