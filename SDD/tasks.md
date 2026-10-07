# Tarefas de Implementação

Executar na ordem. Ler `constitution.md`, `spec.md`, `plan.md` e `tests.md` antes.

| # | Tarefa | Pronto quando |
| --- | --- | --- |
| T1 | Criar estrutura, `config.py` com as constantes da variante e `requirements.txt` fixado | App sobe na porta 8002 |
| T2 | `regras.py`: minutos, tolerância, frações, teto, tempo médio (funções puras, só inteiros) | Tabela de valor do `tests.md` passa em teste unitário |
| T3 | Repositório em memória com lock, id sequencial, busca por placa/status | Sem race em chamadas concorrentes |
| T4 | Validação de placa/entrada/data e handlers de erro com corpo `{"erro":...}` | Todos os 422 do `tests.md` passam, com a precedência correta |
| T5 | UC1 + UC8: abrir bilhete, conflito de placa, normalização para -03:00 | 201/409/422 corretos |
| T6 | UC2 + UC5: encerrar e cancelar | 200/404/409 corretos; cancelamento sem saída/valor |
| T7 | UC3 + UC6: listagens ordenadas (rota `/ativos` antes de `/{id}`) | Ordem e vazios corretos |
| T8 | UC4: relatório diário em -03:00 | Média 0,5 para cima e dia vazio corretos |
| T9 | Testes automatizados próprios (unitários + API) | Cobrem todas as tabelas do `tests.md` |
| T10 | Containerfile não-root, `.gitignore`, README, linter sem avisos | Build e run funcionam; sem segredos |