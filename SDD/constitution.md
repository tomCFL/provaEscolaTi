# Constituição do Projeto — Zona Azul Digital

## Parâmetros da variante (fixos)

| Parâmetro | Valor |
| --- | --- |
| TARIFA_HORA_CENTAVOS | 450 |
| FRACAO_MINUTOS | 15 |
| TETO_DIARIO_CENTAVOS | 8000 |
| TOLERANCIA_MINUTOS | 10 |
| PORTA_SERVICO | 8002 |

Esses valores ficam em constantes nomeadas em um único módulo, nunca espalhados como números mágicos.

## Regras operacionais

1. **Dinheiro**: sempre inteiro em centavos (`valor_centavos`). Proibido float/double/decimal em qualquer ponto do cálculo ou da resposta JSON.
2. **Precedência de erros**: validação de formato (422) vem ANTES de regras de estado (404/409). Payload malformado nunca gera 409.
3. **Formato de erro**: toda falha retorna JSON `{"erro": "<identificador_exato>"}` com o status da tabela de erros da spec. Nunca HTML, stack trace ou texto puro.
4. **Tempo**: toda data/hora de resposta em ISO-8601 com offset `-03:00` (ex.: `2026-10-05T10:00:00-03:00`). Entradas aceitam qualquer offset válido e são convertidas para -03:00. Entrada sem offset é inválida.
5. **Relógio injetável**: o "agora" vem de uma função/abstração única, para permitir teste.
6. **Sem segredos no repositório**: nenhuma credencial, token ou arquivo `.env` versionado; `.gitignore` obrigatório.
7. **Higiene**: código organizado em módulos pequenos, sem código morto, sem arquivos gerados versionados, com linter/formatador padrão da linguagem sem avisos.
8. **Container**: Containerfile/Dockerfile com usuário não-root, dependências com versão fixada em manifesto, servidor escutando em 0.0.0.0 na porta 8002.
9. **Persistência**: em memória (sem dependência externa), protegida contra concorrência (lock).
10. **Entregáveis do código gerado**: Containerfile, manifesto de dependências, README (como rodar e testar), testes automatizados próprios cobrindo as regras de valor.

## Fórmulas normativas

- `minutos = ceil(segundos_entre_entrada_e_saida / 60)`; se negativo, `0`.
- Se `minutos <= 10`: `valor_centavos = 0`.
- Senão: `fracoes = ceil(minutos / 15)`; `bruto = (fracoes * 450) // 4`; `valor = min(bruto, 8000)`.
- A tolerância NÃO é descontada: 11 minutos cobra 1 fração inteira.