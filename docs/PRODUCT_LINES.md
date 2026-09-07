# KidsTracker Platform — Linhas Oficiais de Produto

Status: **decisão de produto v1 — 07/09/2026**

A plataforma passa a tratar duas dores diferentes com duas linhas de produto, compartilhando o máximo possível de firmware, aplicativo e protocolos.

## 1. Visão geral

Não existe uma única necessidade de rastreamento.

Foram identificados dois cenários principais:

1. **Crianças em locais movimentados**, onde o responsável continua relativamente próximo e precisa ser avisado rapidamente se a criança se afastar.
2. **Pessoas que podem ficar realmente distantes do cuidador**, como idosos e pessoas com deficiência, onde localização remota por quilômetros é essencial.

A consequência é a criação de duas linhas oficiais:

- **KidsTracker Local** — BLE + LoRa, sem SIM e sem mensalidade obrigatória.
- **CareTracker 4G** — BLE + GNSS/GPS + 4G, com rastreamento remoto via nuvem.

Ambos compartilham uma base tecnológica comum.

---

# 2. KidsTracker Local

## Público-alvo

Crianças pequenas em situações como:

- igrejas;
- festas;
- shows;
- shopping centers;
- parques;
- praias;
- aeroportos;
- feiras;
- eventos esportivos;
- condomínios;
- áreas de lazer.

## Dor principal

O responsável não precisa saber onde a criança está a 20 km de distância.

Ele precisa saber rapidamente:

> **"Meu filho acabou de sair de perto de mim. Para que lado/situação devo procurar agora?"**

## Arquitetura

```text
TAG DA CRIANÇA
ESP32 + BLE + LoRa
        │
        │ LoRa ponto-a-ponto
        ▼
RECEPTOR DO RESPONSÁVEL
ESP32 + LoRa + BLE
        │
        │ BLE
        ▼
ANDROID
```

## Regra de operação

### Muito próximo

BLE entre TAG e Android/receptor é suficiente.

### BLE perdido

LoRa passa a ser o canal de segurança de maior alcance.

### LoRa disponível

O app continua recebendo presença, intensidade aproximada do sinal e dados da TAG.

### GNSS opcional

A versão inicial poderá existir sem GNSS, dependendo dos testes de campo.

Uma variante superior poderá adicionar GNSS para enviar coordenadas pelo LoRa.

## Características comerciais desejadas

- sem SIM;
- sem plano de dados celular;
- sem mensalidade obrigatória;
- funciona mesmo sem Internet;
- bateria longa;
- produto simples de levar apenas quando necessário;
- um receptor pode monitorar múltiplas TAGs da mesma família.

## Exemplo multi-criança

```text
RECEPTOR DOS PAIS
   ├── TAG 01 — Lucas
   ├── TAG 02 — Ana
   └── TAG 03 — Pedro
          │
          └── Android mostra status individual
```

Estados visuais possíveis:

- Próximo;
- Atenção;
- LoRa ativo;
- Fora de alcance;
- bateria baixa.

## Diferencial principal

**Detecção local imediata sem depender de cobertura celular, Internet ou mensalidade.**

---

# 3. CareTracker 4G

## Público-alvo

Pessoas para as quais faz sentido um rastreamento realmente remoto:

- idosos;
- pessoas com deficiência intelectual;
- pessoas com dificuldade de orientação;
- pessoas vulneráveis que possam se afastar do cuidador;
- outros cenários assistivos aprovados futuramente.

## Dor principal

O cuidador precisa responder:

> **"Onde essa pessoa está agora, mesmo que esteja quilômetros longe de mim?"**

## Arquitetura

```text
TAG
ESP32 + BLE + GNSS/GPS + 4G
        │
        │ Internet móvel
        ▼
BACKEND / CLOUD
        │
        ▼
ANDROID DOS CUIDADORES
```

## Funções esperadas

- localização remota;
- mapa;
- última posição conhecida;
- histórico;
- SOS;
- bateria;
- geofencing;
- alertas de ausência de transmissão;
- múltiplos cuidadores autorizados;
- notificações push;
- comandos remotos.

## Modelo comercial esperado

- hardware vendido separadamente;
- SIM/4G;
- plataforma compartilhada;
- possibilidade de assinatura mensal para cobrir conectividade, nuvem, suporte e operação.

## Diferencial principal

**Rastreamento independente da proximidade entre pessoa e cuidador.**

---

# 4. Base tecnológica compartilhada

Os dois produtos deverão compartilhar sempre que fizer sentido:

- família de ESP32;
- identidade de dispositivo;
- protocolo de eventos;
- monitoramento de bateria;
- botão SOS quando aplicável;
- watchdog;
- logs;
- OTA;
- aplicação Android;
- cadastro de usuários;
- cadastro de pessoas/TAGs;
- UX de alertas;
- biblioteca de estados;
- segurança e autenticação;
- versionamento de firmware;
- testes automatizados.

A regra é evitar manter dois produtos completamente independentes quando a funcionalidade puder ser reutilizada.

---

# 5. Perfis de dispositivo no aplicativo

O Android deverá reconhecer capacidades da TAG em vez de assumir que todas têm o mesmo hardware.

Exemplos de capacidades:

```text
BLE=true
LORA=true
GNSS=false
CELLULAR=false
SOS=true
```

ou:

```text
BLE=true
LORA=false
GNSS=true
CELLULAR=true
SOS=true
```

A interface deverá habilitar apenas recursos compatíveis com a TAG cadastrada.

---

# 6. Nomes conceituais oficiais

## KidsTracker Local

Produto infantil para proximidade e busca em área local.

Tecnologia principal:

**BLE + LoRa ponto-a-ponto.**

Modelo comercial preferencial:

**compra do hardware sem mensalidade obrigatória.**

## CareTracker 4G

Produto assistivo para localização remota.

Tecnologia principal:

**BLE + GNSS/GPS + 4G + cloud.**

Modelo comercial preferencial:

**hardware + serviço de conectividade/plataforma.**

Os nomes são de trabalho e poderão mudar antes da comercialização.

---

# 7. Prioridade de desenvolvimento

A base SafeTrack existente continua especialmente valiosa para o **CareTracker 4G**, pois já possui ESP32-C3, SIM7600, GPS, Firebase, servidor Python, Flutter, mapa, SOS e FCM.

Para o **KidsTracker Local**, a prioridade será desenvolver o novo caminho:

```text
TAG LoRa
   ↕
RECEPTOR LoRa/BLE
   ↕
ANDROID
```

A primeira POC do KidsTracker Local deverá provar apenas:

1. TAG anuncia presença;
2. receptor identifica uma TAG específica;
3. LoRa ponto-a-ponto funciona de forma estável;
4. receptor repassa o estado ao Android via BLE;
5. Android gera alerta de afastamento;
6. reconexão ocorre automaticamente;
7. múltiplas TAGs podem ser diferenciadas.

Somente depois serão adicionados GNSS, SOS avançado e outras funções.

---

# 8. Regra de produto

Não adicionar 4G ao produto infantil apenas porque a tecnologia existe.

Não remover 4G do produto assistivo apenas para reduzir custo.

Cada linha deverá usar a tecnologia adequada à dor que resolve.

**KidsTracker Local:** proximidade, alcance local ampliado, simplicidade e ausência de mensalidade.

**CareTracker 4G:** independência geográfica, localização remota e segurança contínua.

---

# 9. Decisão oficial

A plataforma passa a ser pensada como uma família de soluções:

```text
PLATAFORMA COMUM
│
├── KidsTracker Local
│     BLE + LoRa
│     sem SIM
│     uso local/eventos
│
└── CareTracker 4G
      BLE + GNSS + 4G
      cloud
      rastreamento remoto
```

Essa separação deverá orientar decisões futuras de hardware, firmware, aplicativo, backend, custos e modelo comercial.
