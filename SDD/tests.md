# Casos de Teste e Bordas

Variante: tarifa 450, fração 15, teto 8000, tolerância 10. Fórmula: `min((ceil(min/15)*450)//4, 8000)`, e 0 se `min <= 10`.

## Valor (UC2 + UC7) — caso de borda por regra

| Regra | Minutos | Frações | Valor esperado |
| --- | --- | --- | --- |
| Tolerância (limite) | 0 | — | 0 |
| Tolerância (limite) | 10 | — | 0 |
| Adjacência da tolerância | 11 | 1 | 112 |
| Fração exata | 15 | 1 | 112 |
| +1 min | 16 | 2 | 225 |
| Fração exata | 30 | 2 | 225 |
| +1 min | 31 | 3 | 337 |
| Hora cheia | 60 | 4 | 450 |
| +1 min | 61 | 5 | 562 |
| Caso do contrato | 95 | 7 | 787 |
| Segundos contam | 10 min 1 s | → 11 min | 112 |
| Teto (abaixo) | 1065 | 71 | 7987 |
| Teto (acionado) | 1066 | 72 | 8000 |
| Teto (muito acima) | 3000 | 200 | 8000 |


## Validação e precedência

| Requisição | Esperado |
| --- | --- |
| POST placa `ABC1D23` | 201 |
| placa minúscula, 6 ou 8 chars, símbolo, ausente, número | 422 placa_invalida |
| entrada `"ontem"`, sem fuso, só data | 422 entrada_invalida |
| placa inválida + entrada inválida | 422 placa_invalida |
| placa inválida de uma placa já aberta | 422 (nunca 409) |
| entrada com `+00:00` | 201, convertida para -03:00 |

## Estado (409/404)

| Cenário | Esperado |
| --- | --- |
| Abrir placa já aberta | 409 bilhete_em_aberto |
| Encerrar → abrir mesma placa | 201 |
| Cancelar → abrir mesma placa | 201 |
| Encerrar duas vezes | 409 bilhete_ja_encerrado |
| Encerrar cancelado | 409 bilhete_ja_encerrado |
| Cancelar encerrado / cancelar duas vezes | 409 bilhete_nao_aberto |
| id 9999 / id `abc` em encerrar e cancelar | 404 bilhete_nao_encontrado |
| Cancelamento | sem `saida` nem `valor_centavos` |

## Listagens

| Cenário | Esperado |
| --- | --- |
| ativos sem bilhetes | `[]` |
| 3 abertos | ordem entrada decrescente |
| encerrado/cancelado | ausente de ativos |
| histórico de placa desconhecida | `[]` |
| histórico com aberto + encerrado + cancelado | os 3, mais recente primeiro |
| `/bilhetes` sem placa | 422 placa_invalida |

## Relatório (UC4)

| Cenário | Esperado |
| --- | --- |
| Dia sem encerrados | `total 0, faturamento 0, tempo_medio 0` |
| minutos 10 e 11 | média 10,5 → 11 |
| minutos 10 e 10 e 11 (31/3=10,33) | 10 |
| cancelado e aberto no dia | não contam |
| `data=2026-02-30`, `05/10/2026`, ausente | 422 data_invalida |
| faturamento | soma dos `valor_centavos`, inteiro |

## Contrato de formato
Nenhuma resposta contém float; `valor` (sem `_centavos`) nunca aparece; timestamps terminam em `-03:00`.