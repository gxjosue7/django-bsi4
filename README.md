# django-bsi4

Projeto Django simples para demonstrar uma app de produtos.

**Descrição:**
- Um pequeno projeto Django com uma aplicação `produtos` que define o modelo `Produto`.
- Feito como base para estudos ou demos rápidas.

**Tecnologias**
- Python 3.12+
- Django 6.1+

**Instalação (local)**
1. Crie e ative um ambiente virtual:

```bash
python -m venv .venv
source .venv/bin/activate
```

2. Instale dependências:

```bash
# Se houver requirements.txt
pip install -r requirements.txt

# Ou instale Django diretamente (o projeto requer Django 6.1+)
pip install "django>=6.1.1"
```

3. Aplique migrações e rode o servidor:

```bash
python manage.py migrate
python manage.py runserver
```

4. Acesse http://127.0.0.1:8000

**Estrutura principal**
- `manage.py` — utilitário de execução do Django.
- `config/` — configurações do projeto (`settings.py`, `urls.py`, `wsgi.py`, `asgi.py`).
- `produtos/` — aplicação de exemplo com modelos e views.
- `db.sqlite3` — banco de dados SQLite usado em desenvolvimento.

**Modelo `Produto`**
- `nome` (CharField, 100)
- `preco` (DecimalField, max_digits=8, decimal_places=2)

Exemplo de uso no shell Django:

```bash
python manage.py shell
from produtos.models import Produto
Produto.objects.create(nome='Caneta', preco=2.50)
Produto.objects.all()
```

**Notas**
- O arquivo `pyproject.toml` declara `django>=6.1.1` como dependência e o projeto exige Python 3.12+.
- `DEBUG` está ativado nas configurações — não use assim em produção.

**Contribuições**
- Abra issues ou envie pull requests com melhorias.

**Licença**
- MIT — ajuste conforme necessário.

