# ⚡ Eletroposto Inteligente — EV Challenge 2026

**Sprint 3 — Prototipagem Funcional e Integração**

## 👥 Equipe

- Mauricio Bertuci Saletti — RM571229
- Mateus Eduardo da Cruz Rocha — RM570736
- Lucas Caram Bueno — RM570158
- Rhuan Pacheco Carreri — RM570129
- Leonardo Fortini Marcelo — RM572566
- Nicolas Andrade Rodrigues — RM572782

---

## 📌 Evolução em relação às sprints anteriores

| Sprint | Entrega |
|---|---|
| **Sprint 1** | Proposta: eletroposto inteligente integrado ao ecossistema GoodWe (solar + BESS + gestão de energia) |
| **Sprint 2** | Prova de conceito: firmware ESP32 (Wokwi) com a lógica de decisão **SOLAR → BATERIA → REDE** funcionando em tempo real, um aspecto isolado da solução |
| **Sprint 3** | **Integração completa**: a mesma lógica validada na Sprint 2 agora orquestra **múltiplas sessões de recarga** ao longo do dia, com coleta e exibição dos dados em um dashboard — mostrando o sistema ponta a ponta: geração → decisão → sessão → dado coletado → informação exibida |

A Sprint 2 comprovou que o "cérebro" do sistema (a lógica de decisão de fonte) funciona. A Sprint 3 comprova que esse cérebro **se integra** ao restante da solução: ele não decide só uma vez, decide **sessão a sessão**, registra cada decisão, e essa informação chega até quem opera o eletroposto.

---

## 🏗️ Esquema de integração dos componentes

```
   ☀️  Painel solar (10 kWp)              🔋  Bateria BESS (12 kWh)
        │  geração por hora                     │  armazena excedente solar
        ▼                                        ▼
   ┌─────────────────────────────────────────────────────┐
   │              ESP32 — Motor de decisão                │
   │   a cada hora, para cada sessão ativa:                │
   │     SE geração solar ≥ demanda   → usa SOLAR          │
   │     SENÃO SE bateria > reserva   → usa BATERIA        │
   │     SENÃO                        → usa REDE           │
   └───────────────┬─────────────────────┬─────────────────┘
                    │                     │
          registra cada sessão       aciona LCD / LEDs / buzzer
                    │                     │
                    ▼                     ▼
         📊 log_sessoes_sprint3.json   Interface física (Wokwi)
                    │
                    ▼
         🖥️  dashboard.html  ← exibe sessões, mix de fontes,
                                custo, economia e CO₂ evitado
```

**Camadas e como se conectam:**

1. **Geração e armazenamento** (painel solar + BESS) — mesmas premissas da Sprint 1 e 2.
2. **Controle (ESP32 / `sessoes_recarga.py`)** — a lógica validada na Sprint 2 foi reestruturada para tratar cada chegada de veículo como uma **sessão** (`id`, `veículo`, `horário`, `demanda`), em vez de uma decisão contínua sem contexto de "quem está carregando".
3. **Coleta de dados** — cada hora de cada sessão gera um registro (fonte usada, custo, CO₂ evitado), consolidado em `log_sessoes_sprint3.json` e `log_horario_sprint3.json`.
4. **Exibição** — o `dashboard.html` lê esses dados e apresenta: KPIs do dia, gráfico do mix de fontes hora a hora, e uma tabela por sessão — entregando exatamente o que a Sprint 3 pede em "coleta e exibição das informações".

---

## 🔧 Justificativa técnica das escolhas

| Escolha | Por quê |
|---|---|
| Reaproveitar a lógica SOLAR→BATERIA→REDE da Sprint 2 | Já validada fisicamente no Wokwi; a Sprint 3 pede **integração**, não reinvenção — evolução coerente do mesmo núcleo |
| Modelar "sessão de recarga" como evento discreto (chegada → cobrança → saída) | É o conceito que a Sprint 3 exige explicitamente e que faltava na Sprint 2, que tratava a decisão de forma contínua, sem estrutura de sessão |
| Dashboard em HTML/JS puro (sem backend) | Mantém a mesma filosofia de baixo custo e portabilidade da Sprint 2 (ESP32 sem dependências pesadas); qualquer pessoa abre o arquivo e vê os dados, sem precisar instalar nada |
| Dados no formato JSON | Mesmo dataset pode alimentar tanto o dashboard quanto uma futura extensão (ex: enviar por Wi-Fi do ESP32 real) |

---

## ♻️ Conexão com energias renováveis e sustentabilidade

- A prioridade **solar → bateria → rede** garante que a energia mais limpa disponível seja sempre usada primeiro — o mesmo princípio de eficiência energética da Sprint 1 e 2, agora aplicado a um dia inteiro de operação real do eletroposto.
- A sessão noturna (19h, dentro do horário de ponta) mostra o caso em que o sistema **precisa** recorrer à rede — evidenciando de forma honesta os limites da solução, e não só o cenário ideal.
- O CO₂ evitado e a economia financeira são calculados por sessão, tornando o benefício ambiental **mensurável por veículo atendido**, não só como número agregado do dia.

---

## 📊 Resultados e dados funcionais (dia simulado, 4 sessões)

| Sessão | Veículo | Horário | Energia | Solar | Bateria | Rede | % renovável | Custo | Economia |
|---|---|---|---|---|---|---|---|---|---|
| 1 | EV-A | 8h–9h | 8,0 kWh | 5,20 | 2,80 | 0,00 | 100% | R$ 0,00 | R$ 4,80 |
| 2 | EV-B | 12h–13h | 10,0 kWh | 10,00 | 0,00 | 0,00 | 100% | R$ 0,00 | R$ 6,00 |
| 3 | EV-C | 15h–16h | 14,0 kWh | 6,93 | 7,07 | 0,00 | 100% | R$ 0,00 | R$ 8,40 |
| 4 | EV-D | 19h–20h | 16,0 kWh | 1,35 | 9,60 | 5,05 | 68,4% | R$ 6,06 | R$ 13,14 |

**Totais do dia:** 48,0 kWh entregues · **89,5% de energia renovável** · economia de **R$ 32,34** frente ao custo 100% rede · **5,15 kg de CO₂ evitado**.

> A Sessão 4 (horário de ponta, já com a bateria parcialmente descarregada pelas sessões anteriores) é o caso que comprova que o sistema é **honesto**: quando solar e bateria não bastam, ele usa a rede — e o dashboard mostra isso claramente.

---

## 🔗 Conexão com os conteúdos da disciplina

- **Automação e lógica de decisão condicional**: a árvore de decisão SOLAR→BATERIA→REDE é aplicação direta de estruturas condicionais e controle de estado (SoC da bateria).
- **Eficiência energética**: uso do excedente solar para armazenamento antes de recorrer à rede, evitando desperdício.
- **Coleta e tratamento de dados**: cada sessão gera um registro estruturado (JSON), que é processado e exibido — aplicação prática de manipulação de dados.
- **Sustentabilidade aplicada**: métricas de CO₂ evitado e economia financeira conectam a solução técnica ao impacto ambiental e de negócio propostos desde a Sprint 1.

---

## ▶️ Como executar

```bash
python3 sessoes_recarga.py      # gera log_sessoes_sprint3.json e log_horario_sprint3.json
```

Depois, abra `dashboard.html` em qualquer navegador (não precisa de servidor — os dados já vêm embutidos no arquivo).

O protótipo em ESP32/Wokwi da Sprint 2 (`eletroposto_inteligente.ino`, `diagram.json`) continua válido e pode ser demonstrado em conjunto no vídeo, mostrando a mesma lógica rodando no hardware simulado.

---

## 📁 Estrutura do repositório (Sprint 3)

```
.
├── sessoes_recarga.py            # simulação de múltiplas sessões de recarga (novo na Sprint 3)
├── dashboard.html                # dashboard de coleta e exibição de dados (novo na Sprint 3)
├── log_sessoes_sprint3.json      # dataset gerado: 1 registro por sessão
├── log_horario_sprint3.json      # dataset gerado: 1 registro por hora do dia
├── eletroposto_inteligente.ino   # firmware ESP32 (Sprint 2)
├── diagram.json                  # circuito do Wokwi (Sprint 2)
├── wokwi.toml                    # configuração do Wokwi (Sprint 2)
├── simulacao_24h.py               # simulação original de 24h (Sprint 2)
└── README.md
```
