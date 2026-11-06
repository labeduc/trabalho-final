# Projeto Integrador

Material do **Trabalho Final** do curso de [Ciência de Dados](https://labeduc.github.io/ciencia-de-dados/). O repositório tem duas partes:

- o **enunciado** do trabalho;
- o **dataset** que os alunos usam para fazê-lo.

No trabalho, o aluno faz o papel de analista de dados de uma associação de 14 hotéis fictícios e percorre o ciclo completo visto no curso:

1. exploração dos dados com Python;
2. análise estatística;
3. modelagem e banco de dados;
4. visualização;
5. um painel web construído com apoio de IA.

## Conteúdo

```
demo-dash/
├── docs/
│   └── enunciado-trabalho-final.md   ← enunciado do trabalho final
├── dados/                            ← dataset Nice Rooms (7 arquivos JSONL)
│   └── README.md                     ← documentação completa do dataset
└── LICENSE
```

### `docs/enunciado-trabalho-final.md`

O enunciado divide o trabalho em seis etapas. Para cada etapa, ele indica as aulas do curso a usar, o que o aluno deve investigar e o que deve entregar. Traz também os critérios de avaliação, as regras para o uso de IA e as armadilhas mais comuns do dataset.

O enunciado **não traz soluções prontas**: nada de código, fórmulas, consultas SQL ou modelos de página. Ele aponta o caminho, e o aluno decide como fazer.

### `dados/`

O dataset é o **Nice Rooms: Hotel Revenue, MARS and Reviews**. É **sintético**: simula dois anos (2027–2028) de operação de 14 hotéis na cidade fictícia de Port Alder.

| Arquivo | Conteúdo | Linhas |
| --- | --- | ---: |
| `hotels.jsonl` | Cadastro dos hotéis | 14 |
| `room_types.jsonl` | Tipos de quarto por hotel | 42 |
| `daily_market.jsonl` | Desempenho diário por hotel: ocupação, diária, receitas, comparação com concorrentes e recomendação de preço | 10.192 |
| `weekly_performance.jsonl` | Desempenho semanal por hotel, com resumo das avaliações | 1.456 |
| `rate_decisions.jsonl` | Decisões semanais de preço e o resultado da semana seguinte | 1.442 |
| `reviews.jsonl` | Avaliações de hóspedes, com texto, notas parciais e respostas | 12.077 |
| `review_scenarios.jsonl` | Os 11 cenários que originam as avaliações | 11 |

O [README do dataset](dados/README.md), em inglês, descreve cada coluna. Ele também explica os indicadores hoteleiros (ocupação, ADR, RevPAR, RGI etc.), o conceito de MARS (diária adequada ao mercado) e as limitações dos dados. Leia esse arquivo antes de começar o trabalho.

## Como usar

- **Alunos:** comecem pelo [enunciado](docs/enunciado-trabalho-final.md) e depois leiam o [README do dataset](dados/README.md).
- **Professores:** antes de distribuir o enunciado, preencham os campos marcados como *"a definir pelo professor"* (formato e data de entrega) e revisem os pesos dos critérios de avaliação.

## Licenças

- Enunciado e demais arquivos do repositório: [Apache 2.0](LICENSE).
- Dataset Nice Rooms: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Os dados são fictícios, e a fonte original está no [Kaggle](https://www.kaggle.com/datasets/crissymoon/nice-rooms-hotel-revenue-rate-intelligence).
