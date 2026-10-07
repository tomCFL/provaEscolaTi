# Especificação — Zona Azul Digital (API REST)

Porta 8002, JSON, base `http://localhost:8002`. Valores da variante em `constitution.md`.

## Modelo de Bilhete

| Campo | Tipo | Notas |
| --- | --- | --- |
| id | inteiro | sequencial a partir de 1 |
| placa | string | regex `^[A-Z0-9]{7}$`, sem normalizar minúsculas |
| entrada | string ISO-8601 -03:00 | |
| saida | string ISO-8601 -03:00 | só após encerrar |
| minutos, valor_centavos | inteiros | só após encerrar |
| status | `aberto`, `encerrado`, `cancelado` | |

## Tabela de erros

| Situação | Status | Body |
| --- | --- | --- |
| Placa ausente, não-string ou fora do regex | 422 | `{"erro":"placa_invalida"}` |
| `entrada` presente e fora de ISO-8601 com fuso | 422 | `{"erro":"entrada_invalida"}` |
| `data` ausente ou fora de AAAA-MM-DD (data real) | 422 | `{"erro":"data_invalida"}` |
| Bilhete inexistente (inclui id não numérico) | 404 | `{"erro":"bilhete_nao_encontrado"}` |
| Encerrar bilhete não aberto | 409 | `{"erro":"bilhete_ja_encerrado"}` |
| Cancelar bilhete não aberto | 409 | `{"erro":"bilhete_nao_aberto"}` |
| Abrir com placa já aberta | 409 | `{"erro":"bilhete_em_aberto"}` |

Ordem de validação em `POST /bilhetes`: placa (422) → entrada (422) → placa aberta (409).

## UC1 — POST /bilhetes
Body `{"placa": "...", "entrada": "..."(opcional)}`. Sem `entrada`, usa o relógio.
- **Aceite**: placa `ABC1D23` → 201 com exatamente `id, placa, entrada, status:"aberto"`, entrada com offset -03:00.
- **Aceite**: `entrada` `2026-10-12T11:30:00+00:00` → resposta com `2026-10-12T08:30:00-03:00`.
- **Aceite**: `abc1d23`, `ABC1D2`, `ABC1D234`, `ABC-D23`, placa ausente ou número → 422 placa_invalida.
- **Aceite**: `entrada:"ontem"` ou `"2026-10-12"` → 422 entrada_invalida.

## UC2 — POST /bilhetes/{id}/encerramento
`saida` = relógio atual. Resposta 200 com exatamente `id, placa, entrada, saida, minutos, valor_centavos`.
- **Aceite**: bilhete aberto há 95 min → minutos 95, valor 787.
- **Aceite**: segundo encerramento do mesmo id → 409 bilhete_ja_encerrado; id inexistente → 404.
- **Aceite**: bilhete cancelado → 409 bilhete_ja_encerrado.
- Cálculo exato em `constitution.md` (fórmulas normativas). Valor sempre inteiro; teto 8000.

> [!WARNING]
> O teto de 8000 centavos é aplicado SEMPRE depois do cálculo bruto. Esquecê-lo é o erro mais comum.

## UC3 — GET /bilhetes/ativos
200 com array de bilhetes `aberto` (campos de UC1), ordenado por entrada decrescente, desempate por id decrescente. Vazio → `[]`.
- **Aceite**: após encerrar ou cancelar, o bilhete some da lista.
- A rota `/bilhetes/ativos` deve ser registrada de forma a não ser capturada por `/bilhetes/{id}`.

## UC4 — GET /relatorios/diario?data=AAAA-MM-DD
Considera bilhetes `encerrado` cuja `saida`, no fuso -03:00, cai na data. Cancelados e abertos não entram.
Resposta: `data, total_bilhetes, faturamento_centavos, tempo_medio_minutos`.
- `total_bilhetes` = quantidade de encerrados no dia; `faturamento_centavos` = soma dos valores.
- `tempo_medio_minutos` = média dos `minutos`, arredondada 0,5 para cima, em aritmética inteira (`(2*soma + n) // (2*n)`). Dia sem bilhetes → três campos numéricos `0`.
- **Aceite**: minutos 10 e 11 → média 10,5 → 11. `2026-13-45`, `05/10/2026`, ausente → 422 data_invalida.

## UC5 — POST /bilhetes/{id}/cancelamento
Só `aberto`. 200 com `id, placa, entrada, status:"cancelado"` (sem `saida`, sem `valor_centavos`). Não entra em faturamento.
- **Aceite**: cancelar duas vezes → 409 bilhete_nao_aberto; encerrado → 409 bilhete_nao_aberto; inexistente → 404.

## UC6 — GET /bilhetes?placa=
200 com todos os bilhetes da placa em qualquer status, ordenados por entrada decrescente (desempate id decrescente). Cada item tem `id, placa, entrada, status`, mais `saida, minutos, valor_centavos` quando encerrado.
- **Aceite**: placa válida sem histórico → `[]`. Placa ausente ou inválida → 422 placa_invalida.

## UC7 — Tolerância
Ver fórmulas em `constitution.md`.
- **Aceite**: 10 min → 0; 11 min → 112; 10 min e 1 s → minutos 11 → 112.

## UC8 — Uma vaga por placa
- **Aceite**: segundo POST da mesma placa aberta → 409 bilhete_em_aberto. Após encerrar ou cancelar, novo POST → 201. Placas diferentes não interferem.