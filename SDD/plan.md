# Plano Técnico

## Decisões

| ID | Decisão | Justificativa |
| --- | --- | --- |
| D-01 | Python 3.12 + FastAPI + Uvicorn | Validação e roteamento simples; poucas dependências |
| D-02 | Persistência em memória (dict + lock) | Sem infraestrutura externa; suíte sobe o container limpo |
| D-03 | Dinheiro em int; `(fracoes*450)//4` | Float acumula erro (0.1+0.2≠0.3); 450/4=112,5 não é inteiro, então a divisão inteira sobre o TOTAL garante 4 frações = 450 exatos [^centavos] |
| D-04 | Relógio injetável (função `agora()`) | Testes determinísticos; `entrada` opcional já cobre o tempo passado |
| D-05 | Fuso fixo `timezone(timedelta(hours=-3))` | Sem dependência de tzdata no container |
| D-06 | Handlers de erro customizados retornando `{"erro": ...}` | Frameworks devolvem 422 com corpo próprio; o contrato exige corpo exato |
| D-07 | Validar o body manualmente (dict cru), não por modelo automático | Controla a precedência placa → entrada → 409 e o código de erro |
| D-08 | Escutar em 0.0.0.0:8002 (porta configurável por env opcional, default 8002) | Suíte acessa `localhost:8002`; sem env obrigatória |
| D-09 | Container não-root, `requirements.txt` com versões fixas | SDLC (critério D) |

## Estrutura de módulos
`app/main.py` (rotas), `app/regras.py` (cálculo puro de minutos/valor), `app/repositorio.py` (armazenamento), `app/erros.py`, `app/config.py` (constantes da variante), `tests/`.

## Requisitos de qualidade do código gerado
README com execução e testes; testes unitários de `regras.py` cobrindo a tabela de `tests.md`; `.gitignore`; `Containerfile` com `USER` não-root; linter (ruff) sem avisos.
[^centavos]: Centavos inteiros eliminam a classe de erro de ponto flutuante em dinheiro.