# Zicco — Controle Financeiro

Aplicativo de controle financeiro para uma empresa de eventos particulares de pizza,
construído a partir da leitura dos extratos bancários.

## O problema, em ordem de prioridade

Definida pelos sócios, é ela que ordena o roadmap:

1. **Não sei se dou lucro.** → DRE e margem de contribuição
2. **Perco o controle do que classificar.** → importação e categorização automática
3. **Não sei qual evento compensa.** → rentabilidade por evento
4. **Não consigo me pagar constantemente nem dimensionar quanto posso retirar.** → módulo de retiradas dos sócios
5. **Sustos de caixa.** → contas a pagar e projeção

## Decisões tomadas

| Tema | Decisão | Motivo |
|---|---|---|
| Formato de importação | **OFX** como principal, CSV como plano B | OFX traz `FITID` (id único por lançamento, resolve a deduplicação com garantia) e saldo final declarado (permite conferir se faltou lançamento) |
| Deduplicação | `FITID` + conta no OFX; hash normalizado no CSV | Reimportar o mesmo arquivo nunca duplica |
| Regime contábil | **Duas visões** sobre os mesmos dados: fluxo de caixa (data do banco) e DRE (data de competência) | Eventos têm sinal e saldo em meses diferentes do evento; só caixa produz um DRE sem sentido |
| Centro de lucro | O **evento** | Cada evento é um mini-negócio com receita e custo próprios; é onde mora a decisão comercial |
| Regime tributário | MEI — teto de faturamento **configurável**, não fixo no código | O limite legal muda; e o alerta de teto é requisito, não enfeite |
| Estrutura societária | **Uma empresa, um CNPJ.** Sociedade informal entre dois sócios | Um MEI só. O extrato de retiradas por sócio é o único registro do acordo entre eles — por isso é requisito, não conveniência |
| Contas bancárias | **Uma conta PJ**, separada da pessoal | Extrato quase 100% relevante; dispensa filtro de despesa pessoal na Fase 1 |
| Recebimentos | Previsões flexíveis por evento (1..N), sem módulo rígido de parcelas | 99% é PIX, e o formato varia entre sinal+saldo, à vista e parcelado |
| Identidade dos sócios | Seleção manual do usuário, salva no navegador | A capability `user` não está disponível nesta conta; para dois sócios que confiam um no outro, é adequado |
| Hospedagem | Artifact no claude.ai, código versionado aqui | Custo zero e uso com dados reais em dias; a lógica é JavaScript puro e migra para app próprio sem retrabalho |

## Estrutura do DRE (segmento alimentício / eventos)

```
  RECEITA BRUTA
(-) DEDUÇÕES ............... DAS, taxas de meio de pagamento, cancelamentos
= RECEITA LÍQUIDA
(-) CUSTOS DIRETOS ......... insumos, bebidas, descartáveis, gás,
                             mão de obra do evento, logística
= MARGEM DE CONTRIBUIÇÃO     ← métrica-rainha
(-) DESPESAS FIXAS ......... pró-labore, pessoal, ocupação, marketing,
                             administrativas, manutenção
= EBITDA
(-) Depreciação e despesas financeiras
= LUCRO LÍQUIDO
```

Indicadores derivados: CMV %, ticket médio por evento, custo por convidado,
margem por evento, ponto de equilíbrio em eventos/mês, consumo do teto MEI.

## Painel "Quanto posso retirar"

Responde à dor nº 4 e é a funcionalidade mais rara do conjunto:

```
   Saldo nas contas hoje
(-) Contas a pagar até o fim do mês
(-) Custos comprometidos dos eventos já agendados
(-) Reserva de segurança (configurável)
= DISPONÍVEL PARA RETIRADA
```

Ao lado, o **pró-labore sustentável**: média da margem dos últimos 3 a 6 meses
menos a reserva de reinvestimento — o quanto dá para retirar todo mês sem quebrar.
Acompanhado do extrato de retiradas por sócio, no mês e no ano.

## Arquivos

- `diagnostico.html` — página de verificação publicada como Artifact. Testa as três
  dependências críticas antes de investir na construção: banco de dados compartilhado,
  visibilidade entre os dois sócios e classificação automática.
- `leitor-extrato.html` — leitor de OFX/CSV que roda inteiramente no navegador.
  Deduplicação por FITID, conferência de saldo, normalização de descrição,
  detecção de contrapartes bilaterais e medidor do teto MEI.

## Aprendido com o extrato real

Validado contra 535 lançamentos de 01/01 a 13/09/2026:

- **O parser lê 535 de 535**, com FITID em todos e nenhum duplicado. O saldo declarado
  pelo banco bate exatamente com a soma dos lançamentos (saldo inicial implícito de
  R$ 0,00), provando que o extrato está completo desde a abertura da conta.
- **`NAME` é a contraparte, `MEMO` só diz "Enviado"/"Recebido"** neste banco. Apostar em
  um campo só quebra a descrição inteira — a leitura combina os dois sem repetir.
- **70% dos lançamentos são de contrapartes que se repetem** (65 nomes recorrentes entre
  223 distintos). É a prova de que a categorização por memória de padrões resolve a maior
  parte do trabalho sem depender de IA.
- **Contrapartes que recebem e pagam existem e são grandes.** Somam quase R$ 10 mil em
  entradas que provavelmente não são faturamento. Contá-las como receita distorce tanto
  o DRE quanto o cálculo do teto — daí a detecção automática.
