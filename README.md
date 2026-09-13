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
| Regime tributário | MEI — teto de faturamento **configurável**, não fixo no código | O limite legal muda; e alerta de teto é requisito, não enfeite |
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

## Arquivos

- `diagnostico.html` — página de verificação publicada como Artifact. Testa as três
  dependências críticas antes de investir na construção: banco de dados compartilhado,
  visibilidade entre os dois sócios e classificação automática.
