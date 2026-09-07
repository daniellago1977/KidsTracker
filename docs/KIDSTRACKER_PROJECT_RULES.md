# KidsTracker — Regras Oficiais do Projeto

Status: **especificação base v1**  
Origem: fork de `law4percent/SafeTrack`  
Repositório oficial: `daniellago1977/KidsTracker`

## 1. Objetivo

Criar um rastreador infantil portátil baseado em ESP32-C3, BLE, 4G e GNSS/GPS, integrado inicialmente a um aplicativo Android.

O objetivo principal é detectar rapidamente quando a criança se afasta do responsável e, a partir desse momento, mudar automaticamente para rastreamento remoto de emergência.

Princípio central:

**detectar rápido → alertar rápido → localizar rápido.**

## 2. Estratégia oficial de desenvolvimento

O projeto **não será reescrito do zero**.

O SafeTrack passa a ser a base oficial para:

- firmware ESP32-C3;
- comunicação com modem 4G/GNSS;
- SOS;
- bateria;
- Firebase;
- servidor Python;
- notificações FCM;
- aplicativo Flutter/Android;
- mapa e histórico.

Nossa evolução principal será adicionar uma arquitetura **BLE-first** de proximidade e afastamento.

## 3. Arquitetura oficial

### 3.1 Criança próxima do responsável

```text
TAG ESP32-C3 ←──── BLE ────→ ANDROID
```

Neste estado:

- BLE é o canal principal;
- Android monitora proximidade;
- ESP32 também monitora a presença do responsável;
- GNSS deve permanecer desligado ou em baixo uso quando possível;
- modem 4G deve permanecer em modo de baixo consumo quando possível;
- o servidor não participa da comunicação local de proximidade.

### 3.2 Afastamento confirmado

```text
BLE LOST
   ↓
ESP32 confirma afastamento
   ↓
GNSS ON
   ↓
4G ON
   ↓
CLOUD/BACKEND
   ↓
ANDROID
   ↓
ALERTA + MAPA
```

O backend passa a ser a ponte de comunicação remota entre TAG e Android.

### 3.3 Regra fundamental

**Comunicação local:** TAG ↔ Android diretamente por BLE.  
**Comunicação remota:** TAG → 4G/Internet → backend → Android.

O backend participa quando for necessária comunicação remota, armazenamento, autenticação ou notificação.

## 4. Componentes oficiais

### TAG

Dispositivo físico utilizado pela criança.

Responsabilidades:

- BLE;
- GNSS/GPS;
- modem 4G;
- SOS;
- bateria;
- movimento, quando sensor disponível;
- fila local de eventos;
- watchdog;
- recuperação automática;
- telemetria.

### APP

Aplicativo dos responsáveis.

Foco inicial: **Android somente**.

Responsabilidades:

- pareamento;
- BLE;
- proximidade;
- alertas;
- mapa;
- status da TAG;
- bateria;
- histórico;
- SOS;
- comandos remotos.

### BACKEND

Responsabilidades:

- autenticação;
- associação família/dispositivo;
- telemetria remota;
- última posição;
- histórico;
- alertas;
- notificações push;
- comandos remotos;
- suporte a múltiplos usuários e múltiplas TAGs.

## 5. Estados da TAG

A lógica deverá usar uma máquina de estados explícita.

Estados mínimos:

- `NORMAL`
- `ATTENTION`
- `BLE_LOST_PENDING`
- `BLE_LOST_CONFIRMED`
- `EMERGENCY`
- `SOS`
- `RECOVERED`
- `OFFLINE`

### NORMAL

- BLE saudável;
- criança considerada próxima;
- consumo mínimo;
- GNSS e 4G reduzidos quando possível.

### ATTENTION

- sinal BLE degradando;
- ainda não considerar afastamento confirmado;
- usar janela de tempo e filtro contra falsos positivos.

### BLE_LOST_PENDING

- conexão/presença BLE perdida;
- aguardar curto período de confirmação;
- preparar GNSS/4G sem ainda assumir emergência total.

### BLE_LOST_CONFIRMED

- perda BLE considerada real;
- registrar evento;
- ativar GNSS;
- ativar 4G;
- enviar alerta remoto;
- iniciar rastreamento intensivo.

### EMERGENCY

- segurança tem prioridade sobre economia de bateria;
- enviar posição frequentemente;
- manter GNSS e 4G ativos;
- receber comandos remotos.

### SOS

- prioridade máxima;
- acionado por botão físico com pressão longa;
- enviar evento imediatamente;
- ativar GNSS e 4G;
- manter rastreamento até encerramento seguro.

### RECOVERED

- BLE reapareceu e ficou estável;
- não encerrar emergência no primeiro pacote recebido;
- exigir estabilidade e/ou confirmação do responsável.

## 6. Regras BLE

- utilizar Bluetooth Low Energy;
- BLE é a primeira camada de proteção;
- RSSI não será tratado como distância exata;
- interface poderá mostrar categorias como Muito perto, Perto, Atenção, Sinal fraco e Fora de alcance;
- nunca disparar emergência por uma única perda de pacote;
- usar janela de tempo, múltiplas leituras e filtro de RSSI;
- meta inicial de alerta: aproximadamente 5–10 segundos após perda consistente;
- thresholds deverão ser configuráveis e testados fisicamente.

## 7. Regras GNSS/GPS

- GNSS não precisa ficar ativo continuamente em `NORMAL`;
- ao confirmar afastamento, GNSS deve ser ativado imediatamente;
- toda posição deve incluir timestamp, validade, origem e precisão quando disponível;
- o app deve diferenciar claramente posição atual de última posição conhecida;
- posição antiga nunca poderá aparecer como atual;
- em locais cobertos, manter transmissão do último fix válido e status do dispositivo.

## 8. Regras 4G

- a TAG deve continuar funcionando sem o telefone por perto;
- se BLE for perdido, 4G assume comunicação remota;
- reconexão automática obrigatória;
- falha de rede não pode travar firmware;
- usar backoff de reconexão;
- manter fila local de eventos críticos quando offline;
- reenviar eventos relevantes ao recuperar conexão;
- priorizar payloads pequenos.

## 9. Frequência de rastreamento

Valores iniciais de referência, sujeitos a teste de bateria:

- `NORMAL`: GNSS desligado ou esporádico;
- movimento normal: 1–5 minutos;
- `ATTENTION`: 15–30 segundos;
- `EMERGENCY`: aproximadamente 3–5 segundos;
- `SOS`: frequência máxima segura configurada.

## 10. Bateria

A TAG deverá reportar:

- percentual estimado;
- tensão, quando disponível;
- status de carregamento.

Alertas mínimos:

- bateria baixa;
- bateria muito baixa;
- bateria crítica.

Durante emergência, segurança prevalece sobre economia de bateria.

## 11. SOS

- botão físico obrigatório na versão final;
- pressão curta não deve gerar SOS;
- pressão longa, inicialmente 2–3 segundos, gera SOS;
- dispositivo deve confirmar localmente o acionamento por LED, buzzer ou vibração quando disponível;
- SOS deve ser persistido/retransmitido se a rede estiver indisponível.

## 12. Android

Primeira versão: **Android somente**.

O app deverá:

- funcionar com BLE em background usando mecanismos compatíveis com Android;
- possuir notificações de alta prioridade;
- mostrar status da criança e da TAG;
- mostrar bateria;
- mostrar última comunicação;
- abrir mapa;
- permitir tocar/localizar a TAG;
- receber SOS;
- mostrar posição atual versus última conhecida;
- manter UX simples e direta.

Estados visuais mínimos:

- Próximo;
- Atenção;
- Afastado;
- SOS;
- Offline.

## 13. Backend comercial

O produto deve nascer preparado para múltiplos usuários.

Não haverá um servidor por cliente.

Arquitetura esperada:

```text
MUITAS TAGS
   ↓
PLATAFORMA COMPARTILHADA
   ↓
CONTAS / FAMÍLIAS / DISPOSITIVOS
   ↓
APPS ANDROID
```

Cada família somente poderá acessar seus próprios dispositivos.

Chaves lógicas mínimas:

- `family_id`;
- `user_id`;
- `child_id`;
- `device_id`.

## 14. Segurança e privacidade

Localização infantil é dado sensível.

Regras:

- autenticação por dispositivo;
- chave única por TAG;
- comunicação criptografada;
- comandos críticos autenticados;
- sem links públicos permanentes de localização;
- acesso apenas a responsáveis autorizados;
- histórico apenas pelo período necessário;
- nenhuma TAG deve aceitar comandos arbitrários de celulares próximos.

## 15. Telemetria mínima

Campos conceituais:

```text
device_id
timestamp
state
latitude
longitude
accuracy
location_type
battery
motion
cellular_signal
ble_state
event
firmware_version
```

Eventos mínimos:

- `DEVICE_BOOT`
- `BLE_CONNECTED`
- `BLE_WEAK`
- `BLE_LOST`
- `BLE_RECOVERED`
- `GNSS_FIX`
- `GNSS_LOST`
- `NETWORK_CONNECTED`
- `NETWORK_LOST`
- `SOS_PRESSED`
- `EMERGENCY_STARTED`
- `EMERGENCY_FINISHED`
- `BATTERY_LOW`
- `BATTERY_CRITICAL`
- `DEVICE_CHARGING`
- `DEVICE_RESTART`
- `CONFIG_CHANGED`

## 16. Robustez

Obrigatório:

- watchdog;
- recuperação de modem;
- recuperação de GNSS;
- recuperação de rede;
- reinício controlado do ESP32 quando necessário;
- logs locais limitados;
- não preencher flash indefinidamente;
- fila offline limitada e persistente para eventos críticos.

## 17. OTA

A arquitetura deve permitir OTA futuramente.

OTA nunca deverá iniciar durante:

- SOS;
- emergência;
- bateria crítica.

Atualizações deverão ser versionadas e, quando tecnicamente possível, permitir rollback.

## 18. Estratégia de consumo

O princípio de otimização será:

**BLE enquanto perto; 4G/GNSS intensivos somente quando necessários.**

Enquanto houver BLE confiável com o responsável, o Android pode usar sua própria Internet para sincronizar status com a nuvem.

O 4G da TAG será o canal independente de segurança e deverá assumir quando o BLE for perdido ou quando houver SOS.

## 19. Ordem oficial de implementação

### Fase 0 — Baseline SafeTrack

- preservar upstream;
- compilar firmware original;
- compilar app original;
- validar backend original;
- documentar hardware real disponível.

### Fase 1 — BLE POC

- ESP32 anunciar BLE;
- Android detectar TAG;
- pareamento;
- leitura de RSSI;
- detectar perda;
- notificação local.

### Fase 2 — Máquina de estados BLE

- `NORMAL`;
- `ATTENTION`;
- `LOST_PENDING`;
- `LOST_CONFIRMED`;
- `RECOVERED`.

### Fase 3 — Integração GNSS/4G

- ao perder BLE, ativar GNSS/4G;
- enviar evento remoto;
- iniciar localização intensiva.

### Fase 4 — Backend e push

- telemetria;
- última posição;
- histórico;
- FCM;
- comandos remotos.

### Fase 5 — App Android completo

- status;
- mapa;
- proximidade;
- alertas;
- SOS;
- localizar TAG;
- bateria.

### Fase 6 — Consumo e campo

- medir autonomia;
- otimizar BLE;
- otimizar GNSS;
- otimizar modem;
- testes em shopping, parque, aeroporto e ambientes cobertos.

### Fase 7 — Comercialização

- multiusuário;
- provisionamento de TAGs;
- plano de conectividade;
- billing/assinatura se necessário;
- telemetria operacional;
- suporte e OTA.

## 20. Critério de versão 1 funcional

A primeira versão será considerada funcional quando:

1. TAG liga e se recupera sozinha de falhas comuns;
2. Android encontra e autentica TAG;
3. BLE mantém monitoramento de proximidade;
4. Android percebe afastamento;
5. TAG também reconhece perda do responsável;
6. TAG ativa GNSS;
7. TAG ativa/conecta 4G;
8. TAG envia localização remotamente;
9. Android recebe alerta;
10. Android mostra localização no mapa;
11. responsável consegue comandar localizar/tocar TAG;
12. botão SOS funciona;
13. bateria é monitorada;
14. posição antiga nunca é apresentada como atual.

## 21. Regra de engenharia

Não implementar tudo de uma vez.

Cada etapa deverá ser validada fisicamente antes de ser tratada como concluída.

Compilar não significa validar.

Sempre que possível:

- mudança pequena;
- teste;
- commit;
- rollback disponível.

## 22. Relação com SafeTrack

O SafeTrack permanece como upstream e referência técnica.

A licença MIT original deverá ser preservada conforme seus termos.

Nosso projeto poderá modificar, ampliar e comercializar a solução respeitando as obrigações da licença e das dependências utilizadas.

---

**Decisão oficial:** KidsTracker passa a ser a evolução BLE-first do SafeTrack, com foco inicial em Android e em segurança infantil por detecção local de afastamento + localização remota por 4G/GNSS.
