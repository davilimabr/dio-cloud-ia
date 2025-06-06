
# Criando um Validador de Bandeiras de Cartão de Crédito com o GitHub Copilot

Estas anotações descrevem, **passo a passo**, o processo de construção de um módulo de validação de bandeiras de cartão de crédito (Visa, Mastercard, Amex, etc.) *impulsionado por IA* utilizando o **GitHub Copilot** como assistente de programação.

> **Objetivo:** Demonstrar como guiar o Copilot por meio de *prompts* claros para gerar código de alta qualidade, testável e alinhado às boas práticas de desenvolvimento seguro de software.

---

## 1. Pré‑requisitos

| Ferramenta / Conceito | Descrição |
|-----------------------|-----------|
| **Editor**           | VS Code com a extensão GitHub Copilot habilitada |
| **Linguagem**        | Python 3.10 + ou JavaScript/TypeScript (exemplos em Python) |
| **Bibliotecas**      | Nenhuma externa obrigatória; para testes: `pytest` |
| **Conhecimentos**    | Regex básica, princípios de TDD, versionamento com Git |

---

## 2. Visão Geral da Solução

* Entrada: número de cartão (string com dígitos, opcionalmente espaçada ou separada por hífens).  
* Saída: **bandeira** identificada (`"visa"`, `"mastercard"`, `"amex"`, `"elo"` …) ou `None` se não for reconhecida.  
* Estratégia:  
  1. **Sanitização** da entrada (remover espaços/hífens).  
  2. **Mapeamento** de *prefixos BIN* e comprimentos válidos para cada bandeira.  
  3. **Validação Luhn** (checksum).  
  4. **API pública** (função + CLI opcional).

---

## 3. Estrutura Inicial do Projeto

```text
card-validator/
├── card_validator/
│   ├── __init__.py
│   ├── core.py
│   └── cli.py
├── tests/
│   └── test_core.py
├── README.md
└── pyproject.toml
```

### Prompt para o Copilot

> “Gerar estrutura de módulos Python para um pacote `card_validator` com `core.py` contendo a lógica de detecção de bandeira e `cli.py` expondo um comando `detect`.”

---

## 4. Passo a Passo com Copilot

### 4.1 Sanitização da Entrada

Prompt sugerido:

> “Escreva uma função `sanitize(card_number: str) -> str` que remova tudo exceto dígitos.”

**Copilot gera:**

```python
import re

def sanitize(card_number: str) -> str:
    """Remove espaços, hífens e retorna apenas dígitos."""
    return re.sub(r"\D", "", card_number)
```

### 4.2 Tabela de Prefixos (BINs) e Comprimentos

Prompt:

> “Crie um dicionário `CARD_SCHEMES` com regex e tamanhos válidos para visa, mastercard, amex, elo e discover.”

Exemplo resultante:

```python
CARD_SCHEMES = {
    "visa":        {"regex": r"^4[0-9]{12}(?:[0-9]{3})?$",              "lengths": {13, 16, 19}},
    "mastercard":  {"regex": r"^(5[1-5][0-9]{14}|2[2-7][0-9]{14})$",    "lengths": {16}},
    "amex":        {"regex": r"^3[47][0-9]{13}$",                       "lengths": {15}},
    "discover":    {"regex": r"^6(?:011|5[0-9]{2})[0-9]{12}$",          "lengths": {16, 19}},
    "elo":         {"regex": r"^(4011|4312|4389|4514|4576|5041|6277)\d+$", "lengths": {16}}
}
```

> **Dica:** refine o regex manualmente se o Copilot produzir padrões genéricos.

### 4.3 Implementação da Validação Luhn

Prompt:

> “Implemente `is_valid_luhn(number: str) -> bool` com complexidade O(n).”

Copilot sugere:

```python
def is_valid_luhn(number: str) -> bool:
    digits = [int(d) for d in number[::-1]]
    checksum = sum(
        d if i % 2 == 0 else (d*2 - 9 if d*2 > 9 else d*2)
        for i, d in enumerate(digits)
    )
    return checksum % 10 == 0
```

### 4.4 Função Principal `detect_scheme`

Prompt:

> “Escreva uma função `detect_scheme(card_number: str) -> Optional[str]` que combine sanitização, regex, tamanho e Luhn.”

Resultado esperado:

```python
from typing import Optional, Dict, Set, Pattern
import re

_SCHEMES: Dict[str, Dict[str, str | Set[int]]] = CARD_SCHEMES

def detect_scheme(card_number: str) -> Optional[str]:
    num = sanitize(card_number)
    if not is_valid_luhn(num):
        return None
    for scheme, meta in _SCHEMES.items():
        if len(num) in meta["lengths"] and re.match(meta["regex"], num):
            return scheme
    return None
```

---

## 5. Test‑Driven Development

**Prompt para testes:**

> “Gere casos de teste `pytest` cobrindo números válidos e inválidos para cada bandeira.”

Exemplo:

```python
import pytest
from card_validator.core import detect_scheme

@pytest.mark.parametrize("num,expected", [
    ("4111 1111 1111 1111", "visa"),
    ("5500 0000 0000 0004", "mastercard"),
    ("3400 0000 0000 009",  "amex"),
    ("6011 0000 0000 0004", "discover"),
    ("0000 0000 0000 0000", None),
])
def test_detect_scheme(num, expected):
    assert detect_scheme(num) == expected
```

> **Boas práticas:** rodar `pytest --cov` e atingir > 90 % de cobertura.

---

## 6. Interface de Linha de Comando

Prompt:

> “No `cli.py`, crie um comando `detect <card-number>` que imprime a bandeira ou ‘indefinida’. Use `argparse`.”

Snippet gerado:

```python
import argparse
from card_validator.core import detect_scheme

def main():
    parser = argparse.ArgumentParser(description="Detecta bandeira de cartão.")
    parser.add_argument("number", help="Número do cartão")
    args = parser.parse_args()
    scheme = detect_scheme(args.number) or "indefinida"
    print(scheme)

if __name__ == "__main__":
    main()
```

---

## 7. Ajustes e Refatoração Guiados por Copilot

* **Sugestão de type hints** e docstrings.  
* **Extração** de lógica de regex para classe `Scheme` (OOP).  
* **Internacionalização** das mensagens da CLI.

Prompt criativo:

> “Refatore `core.py` usando Pydantic para validar entradas e retornar objetos fortes.”

---

## 8. Boas Práticas de Segurança

| Recomendações | Justificativa |
|---------------|--------------|
| Evitar logar números completos | Conformidade PCI‑DSS |
| Usar mascaramento ao exibir cartões | Reduz risco de vazamento |
| Manter BIN table atualizada | Adapta‑se a novos ranges emitidos |
| Automatizar testes em CI | Garante integridade contínua |

---

## 9. Possíveis Extensões

1. **API REST** com FastAPI para validação via HTTP.  
2. **Distribuição** do pacote no PyPI (`build` + `twine`).  
3. **Suporte** a cartões corporativos e pré‑pagos com regras distintas.  
4. **Benchmark** versus bibliotecas existentes (*creditcard* ‑ npm, *card‑validator* ‑ JS).  

Prompt:

> “Crie endpoint `/detect` em FastAPI que aceita JSON `{{"number": "<str>"}}` e retorna `{{"scheme": "<str|None>"}}`.”

---

## 10. Conclusão

Utilizando **GitHub Copilot** como parceiro de desenvolvimento:

* Reduzimos o _boilerplate_ para criar um validador robusto.  
* Mantivemos controle sobre **segurança** e **conformidade**.  
* Aceleramos a entrega combinando *prompts* claros + revisão crítica humana.


> *Documento gerado em 06/06/2025.*
