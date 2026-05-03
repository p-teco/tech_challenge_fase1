# Tech Challenge - Fase 1
**Dieizon A. Ferreira | RM: 371888 | 1IAST - FIAP PosTech**

---



Análise exploratória de dados de NPS de um e-commerce, com o objetivo de entender quais fatores operacionais influenciam a satisfação do cliente e propor uma estratégia para antecipar detratores antes da pesquisa ser aplicada.

---

## Estrutura

```
├── data/
│   └── desafio_nps_fase_1.csv
|   └── bench_prazo_entrega.ods
├── notebooks/
│   └── tech_challenge.ipynb
├── reports/
│   └── apresentacao_tech_challenge.pdf
|   └── apresentacao_tech_challenge.pptx
├── videos/
│   └── apresentacao_tech_challenge.mkv
└── README.md
```

---

## Base de dados

2.500 registros, 19 colunas, sem valores nulos.

Dados de pedidos, logística e atendimento ao cliente. As principais colunas usadas na análise:

| Coluna | Descrição |
|--------|---------|
| `nps_score` | Nota do cliente de 0 a 10 (Variável alvo) |
| `delivery_delay_days` | Dias de atraso na entrega |
| `complaints_count` | Número de reclamações |
| `customer_service_contacts` | Contatos com o suporte |
| `resolution_time_days` | Dias para resolver um problema |
| `repeat_purchase_30d` | Se o cliente recomprou em 30 dias (0/1) |
| `csat_internal_score` | Score interno de satisfação (não usado como feature "data leakage") |

Variáveis criadas no notebook: `nps_categoria`, `faixa_idade`, `faixa_tenure`, `faixa_valor`

---

## O que foi feito

### Exploração inicial
- Verificação de nulos e estatísticas descritivas
- Histograma do NPS, grande concentração de notas baixas
- Criação da coluna `nps_categoria`: Detrator (0-6), Neutro (7-8), Promotor (9-10)

Distribuição encontrada:
- Detratores: **74%**
- Neutros: **18%**
- Promotores: **8%**

---

### O que mais gera detratores

Comparação de médias por categoria NPS:

| Variável | Detrator | Neutro | Promotor |
|----------|----------|--------|----------|
| Dias de atraso | 2.53 | 1.40 | 0.76 |
| Reclamações | 4.62 | 2.99 | 2.39 |
| Contatos suporte | 1.69 | 1.13 | 0.78 |
| Tempo resolução | 5.79 | 4.83 | 4.10 |

---

### Ponto de ruptura

% de detratores por dias de atraso:

```
0 dias -> 36.5%
1 dia  -> 59.7%  <- ruptura
2 dias -> 75.4%
3 dias -> 89.7%
4+     -> 95%+
```

Qualquer entrega que atrasa 1 dia já coloca a maioria dos clientes no campo negativo.

---

### Correlações com o NPS

```
delivery_delay_days         -0.60
complaints_count            -0.50
customer_service_contacts   -0.35
resolution_time_days        -0.19
repeat_purchase_30d         +0.57
csat_internal_score         +0.56  <- Possivel data leakage
```

Idade, região, valor do pedido e parcelas ficaram próximos de zero.

---

### Tipo de cliente com NPS mais alto ou mais baixo

Comparativo entre quem recomprou e quem não recomprou em 30 dias:

| Variável | Não recomprou | Recomprou |
|----------|--------------|-----------|
| Dias de atraso | 2.32 | 0.76 |
| Reclamações | 4.32 | 2.42 |
| Contatos suporte | 1.59 | 0.80 |
| Tempo resolução | 5.62 | 4.11 |

Clientes com problemas não resolvidos entram em loop de contato -> desgaste -> não recompram -> NPS baixo.

---

### Estratégia de modelo preditivo

Modelo de classificação diário Provável Detrator / Não Detrator rodando ao longo da jornada:

| Momento | Sinal | Variáveis |
|---------|-------|-----------|
| Pedido realizado | Fraco | `order_value`, `freight_value`, `customer_tenure_months`, `discount_value`, histórico do cliente |
| Em transporte | Moderado | `delivery_attempts`, dias em trânsito vs. prazo |
| Atraso ativo | Forte | `delivery_delay_days >= 1`, `customer_service_contacts`, `complaints_count` |

Quanto mais tarde na jornada, mais preciso, porém temos menos tempo para agir. A janela de maior valor é o atraso ativo.

---

## Como rodar

```bash
pip install pandas numpy matplotlib
```

```bash
git clone https://github.com/p-teco/tech_challenge_fase1.git
cd tech_challenge_fase1
jupyter notebook notebooks/tech_challenge.ipynb
```

Executar todas as células em ordem. Nenhuma configuração adicional necessária.

---

## Referências

- Benchmark NPS: https://www.opinionbox.com/nps/segmentos-avaliados/e-commerce-e-marketplaces/
- Benchmark Recompra: https://www.prax.ai/blog/benchmarks-ecommerce
- Benchmark CSAT: https://useconverge.app/benchmarks/csat-score
- Benchmark Prazo de entrega: estudo próprio, dados em `data/bench_prazo_entrega.ods`

---

## Limitações

- Base com 2.500 registros, grupos com 6+ dias de atraso têm apenas 3 a 34 amostras.
- A análise mostra associações, não causalidade.
- `csat_internal_score` risco de data leakage.
- Taxa de recompra projetada para 127 dias calculada de forma linear.
- Modelo preditivo é uma reflexão estratégica, e não implementado.
