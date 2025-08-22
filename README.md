# Detector de Gás Natural (Metano) – ESP8266

> Projeto acadêmico IBM3119 • Protótipo open‑source para detecção de gás natural (CH₄) com ESP8266

**Status atual**
- Sensor MQ (MQ‑4/MQ‑5/MQ‑8) **não disponível no Ibmec** — aguardando compra/emprestado.
- Enquanto isso, o firmware incluirá **modo simulado** para desenvolvimento e testes.
- Planejada **impressão 3D** de um **compartimento para pilhas** (baterias) e, depois, gabinete ventilado.

---

## 1) Requisitos do Projeto

### Requisitos funcionais
- Detectar presença/aumento de **metano (CH₄)** no ambiente.
- Exibir nível relativo (ou “ppm equivalente” após calibração).
- Disparar **alerta sonoro e visual** quando ultrapassar um limiar.
- Enviar leituras via **Wi‑Fi** (MQTT ou HTTP) para monitoramento.
- Permitir **calibração** (definir/armazenar R₀ e ajustar limiar).
- Registrar data/hora de calibração e do último alarme.

### Requisitos não funcionais
- Baixo custo e reprodutibilidade com documentação aberta.
- Operação contínua com alimentação **5 V**; consumo adequado.
- Caixa segura, ventilada e de fácil manutenção.
- **Aviso de segurança:** protótipo acadêmico — **não** substitui detector certificado.

---

## 2) Especificações (mapeamento dos requisitos)

### 2.1 Hardware (BOM)
- **ESP8266** (NodeMCU ou Wemos D1 mini).
- **Sensor de gás** _(aguardando disponibilidade)_: **MQ‑4** (recomendado para CH₄).  
  > Alternativas: MQ‑5/MQ‑2. Se usar **MQ‑8**, o projeto passa a detectar **H₂**, não CH₄.
- **Buzzer** piezo 5 V.
- **LEDs** (verde, amarelo, vermelho) + resistores 220 Ω.
- **Fonte 5 V / 1 A** ou **compartimento de pilhas** (ver Seção 4).
- Protoboard/PCB, jumpers, parafusos/espacadores.
- **(Opcional)**: Display OLED I²C (SSD1306), **DHT22** (compensação termo‑úmida), **ADS1115** (ADC externo).

### 2.2 Software (firmware)
- IDE: **Arduino IDE** ou **PlatformIO**.
- Bibliotecas: `ESP8266WiFi`, `PubSubClient` (MQTT), `ArduinoJson` (payloads),  
  `Adafruit_SSD1306`/`Adafruit_GFX` (display, opcional).
- Telemetria: **MQTT** (tópicos sugeridos `devices/ch4/<id>/state`) ou **HTTP REST**.
- Lógica:
  - Leitura analógica → **filtro** (média móvel) → cálculo **Rₛ/R₀**.
  - **Histerese** para o alarme (evita “pisca”). Limite configurável.
  - **Publicação periódica** (ex.: a cada 5 s) e envio imediato em alarme.
  - **Calibração**: rotina para definir **R₀** (ar limpo) e salvar em SPIFFS/LittleFS.
  - **Modo simulado** enquanto o sensor físico não estiver disponível.

### 2.3 Modelos de IA (opcional)
- Base: detecção por **limiar** usando a curva do sensor.
- Extensão: regressão leve (log‑polinomial) sobre `log(Rₛ/R₀)` com temperatura/umidade para estimar concentração relativa.

### 2.4 Sinal utilizado
- Saída **analógica** do módulo MQ → pino **A0** do ESP8266.  
  > Atenção ao **range** do A0 (algumas placas aceitam ~3.2 V; outras exigem até 1.0 V). Ajuste com **divisor resistivo** se necessário.
- Saída **digital** do módulo (trimpot) é opcional; priorizar leitura **analógica**.

---

## 3) Disponibilidade no Ibmec
- Sensor MQ (MQ‑4/MQ‑5/MQ‑8): **não disponível** no momento. Ações:
  - [ ] Solicitar compra/emprestado.
  - [ ] Manter “modo simulado” no firmware até a chegada do sensor.
- Demais itens (ESP8266, LEDs, buzzer, etc.): verificar estoque local antes da montagem.

---

## 4) Alimentação e Impressão 3D

### 4.1 Compartimento de pilhas (para impressão 3D)
- **Objetivo:** alojar pilhas (ex.: 4×AA NiMH) e prover saída **5 V** via módulo **step‑up** (ex.: MT3608) até o ESP8266.
- Requisitos de design:
  - Tampa com trava e furação para cabos.
  - Espaço para módulo DC‑DC e chave liga/desliga.
  - Furações para parafuso de fixação em parede.
- Após escolher/ajustar o modelo, **enviar STL** para impressão: **ibmeclabs@gmail.com**  
  (geralmente são necessárias 2–3 iterações).

### 4.2 Gabinete principal (fase seguinte)
- Janelas de ventilação superior/lateral (gás sobe).
- Suporte interno para o módulo MQ, LEDs e buzzer.
- Furação externa para LEDs e som do buzzer.

---

## 5) Montagem (pinout sugerido)

| Função           | Pino ESP8266 | Observações                      |
|------------------|--------------|----------------------------------|
| Sensor MQ – AOUT | **A0**       | Verificar range de tensão        |
| LED Verde        | **D1 (GPIO5)** | 220 Ω em série                   |
| LED Amarelo      | **D2 (GPIO4)** | 220 Ω em série                   |
| LED Vermelho     | **D5 (GPIO14)**| 220 Ω em série                   |
| Buzzer           | **D6 (GPIO12)**| Com transistor se necessário     |
| I²C (OLED, opt.) | **D1/D2**     | I²C padrão (SCL/SDA)             |

> Evitar pinos de boot (ex.: D8/GPIO15) para saídas ativas.

---

## 6) Firmware – configuração rápida

1. Copiar `firmware/` e abrir no Arduino IDE/PlatformIO.
2. Ajustar `WIFI_SSID`, `WIFI_PASS` e, se usar, `MQTT_BROKER` em `config.h`.
3. Definir `ALARM_THRESHOLD` e **habilitar `SIMULATED=true`** enquanto não houver sensor.
4. Gravar na placa e monitorar via serial (115200 bps).

**Calibração (quando o sensor chegar):**
- Fazer **burn‑in** ≥ 24 h em ar limpo.
- Rodar rotina de **calibração de R₀** (menu serial ou botão).
- Confirmar estabilidade de `Rₛ/R₀` e ajustar `ALARM_THRESHOLD`.

---

## 7) Revisão de Literatura (Overleaf – padrão IEEE/SBrT)
- Tópicos: princípio dos sensores MQ, curvas log‑log, limitações (UR/temperatura), calibração, comparação MQ‑4 × MQ‑5 × MQ‑2, segurança.
- Template sugerido: `IEEEtran` (conference). Manter `.bib` com referências.

---

## 8) Estrutura do Repositório

```
/docs/           (esquemas, datasheets, estudos IEEE)
/hardware/       (BOM, STL do compartimento de pilhas, fotos)
/firmware/       (código, config.h, modo simulado, instruções)
/calibration/    (notas de R0, planilhas)
/test/           (protocolos e resultados)
/LICENSE
README.md
```

---

## 9) Roadmap / Tarefas

- [ ] **Definir sensor** (MQ‑4 recomendado) e providenciar aquisição.
- [ ] Desenhar/ajustar **compartimento 3D para pilhas** e imprimir.
- [ ] Montagem elétrica (ESP8266 + LEDs + buzzer + alimentação).
- [ ] Implementar **modo simulado** e telemetria (MQTT/HTTP).
- [ ] Após chegada do sensor: **burn‑in**, **calibração** e ajuste de limiar.
- [ ] Documentar testes e resultados no `/docs` e Overleaf.

---
