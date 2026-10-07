# Constituição do Projeto — Zona Azul Digital

## Regras Operacionais e Convenções Persistentes

1. **Moeda e Representação de Valores**:
   - Todo e qualquer valor monetário DEVE ser estritamente manipulado e retornado como **número inteiro em centavos**[cite: 2].
   - É proibido o uso de valores em ponto flutuante (`float`, `double`) para evitar erros de arredondamento[cite: 2].

2. **Datas e Fusos Horários**:
   - Entradas e saídas de bilhetes DEVEM utilizar a especificação ISO-8601 preservando o fuso horário `-03:00` (ex: `2026-10-05T14:30:00-03:00`)[cite: 2].
   - As pesquisas por data em relatórios utilizam o formato estrito `AAAA-MM-DD`[cite: 2].

3. **Padronização do Idioma e Chaves de Erro**:
   - Todos os endpoints REST que resultarem em falha de validação ou regra de negócio DEVEM retornar HTTP Status correspondente (404, 409, 422) e JSON com a chave `"erro"` contendo o identificador snake_case exato[cite: 2].

4. **Isolamento de Ambiente e Porta**:
   - A aplicação DEVE rodar em container Docker e escutar obrigatoriamente na porta **8002** (`PORTA_SERVICO=8002`)[cite: 2, 3].

5. **Acurácia de Regra de Negócio (Variante)**:
   - Tarifa por hora: **450 centavos**[cite: 3].
   - Granularidade de cobrança: **15 minutos** (fração)[cite: 3].
   - Teto máximo diário: **8000 centavos**[cite: 3].
   - Tolerância inicial: **10 minutos**[cite: 3].