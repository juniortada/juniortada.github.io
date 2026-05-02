---
layout: post
title: "Calibrando o Linear Advance no Marlin com Bowden — Voron 2.4"
date: 2026-05-01
tags: [3d-printing, marlin, voron, calibration, linear-advance]
---

Se você imprime com extrusor Bowden e nota bolhas, blobs nos cantos ou excesso de material em círculos pequenos, o **Linear Advance** (equivalente ao Pressure Advance do Klipper) é a solução. Este guia mostra o passo a passo completo para calibrar no Marlin 2.x com uma Voron 2.4 e extrusor Voron M4 com tubo de 400 mm.

---

## O que é o Linear Advance?

Em sistemas Bowden, existe uma distância considerável entre o motor do extrusor e o bico. Isso cria um atraso de pressão: o filamento continua fluindo depois que o motor para, e falta pressão quando o motor acelera. O resultado são cantos com excesso de material e lacunas logo após as curvas.

O Linear Advance compensa esse atraso ajustando dinamicamente a velocidade do extrusor. O valor de calibração é chamado **K-factor**.

---

## Pré-requisitos

- Marlin 2.x com `LIN_ADVANCE` habilitado em `Configuration_adv.h`:

```cpp
#define LIN_ADVANCE
#define ADVANCE_K 0.60
```

- Firmware recompilado e flashado na impressora
- OrcaSlicer com `enable_pressure_advance = 1` no perfil do filamento

> Sem o firmware atualizado, os comandos `M900` do gcode são ignorados e o teste não funciona.

---

## Setup utilizado

| Componente     | Especificação             |
|----------------|---------------------------|
| Impressora     | Voron 2.4 — 300 × 300 mm |
| Extrusor       | Voron M4 (Bowden)         |
| Tubo Bowden    | 400 mm                    |
| Hotend / Bico  | 0,4 mm                    |
| Filamento      | PLA 1,73 mm               |
| Firmware       | Marlin 2.1.2.5            |

---

## Passo 1 — Acessar o gerador de gcode

Acesse: **marlinfw.org/tools/lin_advance/k-factor.html**

---

## Passo 2 — Preencher os campos

### Printer / Filament

| Campo               | Valor           |
|---------------------|-----------------|
| Printer Name        | Voron 2.4 300   |
| Filament            | PLA             |
| Filament Diameter   | 1.73            |
| Nozzle Diameter     | 0.4             |
| Nozzle Temperature  | 215             |
| Bed Temperature     | 60              |
| Retraction Distance | 3.5             |
| Layer Height        | 0.2             |
| Extruder            | 0               |
| Fan Speed           | 80              |

### Print Bed

| Campo             | Valor        |
|-------------------|--------------|
| Bed Shape         | Rectangular  |
| Bed Size X        | 300          |
| Bed Size Y        | 300          |
| Origin Bed Center | desmarcado   |

### Speed — ativar "Use mm/s"

| Campo               | Valor | Observação                             |
|---------------------|-------|----------------------------------------|
| Slow Printing Speed | 20    | referência lenta para comparar linhas  |
| Fast Printing Speed | 70    | diferença ampla deixa o efeito visível |
| Movement Speed      | 150   |                                        |
| Retract Speed       | 30    |                                        |
| Unretract Speed     | 30    |                                        |
| Acceleration        | 500   | baixo para isolar a variável K         |
| Jerk X / Y / Z / E | -1    | usa default do firmware                |

### Pattern

| Campo                | Valor              | Observação                           |
|----------------------|--------------------|--------------------------------------|
| Lin Advance Version  | 1.5                | Marlin 1.1.9 / 2.x                   |
| Pattern Type         | Standard           |                                      |
| Starting Value for K | 0                  |                                      |
| Ending Value for K   | 1.2                | Bowden longo pode precisar de K alto |
| K-factor Stepping    | 0.05               | 24 linhas, resolução fina            |
| Slow Speed Length    | 20                 |                                      |
| Fast Speed Length    | 40                 |                                      |
| Test Line Spacing    | 5                  |                                      |
| Print Anchor Frame   | marcado            | melhora adesão                       |
| Printing Direction   | Left to Right (0°) |                                      |
| Line Numbering       | marcado            | identifica o K de cada linha         |

### Advanced

| Campo                      | Valor | Observação                                       |
|----------------------------|-------|--------------------------------------------------|
| Nozzle Line Ratio          | 1.2   |                                                  |
| Z-Offset                   | 0     |                                                  |
| Use Bed Leveling           | Yes   | G29 ativo na impressora                          |
| Use FW Retract             | No    | retração via slicer                              |
| Extrusion Multiplier       | 1.0   |                                                  |
| Prime Nozzle               | Yes   | garante pressão estável antes do teste           |
| Prime Extrusion Multiplier | 2.5   |                                                  |
| Prime Printing Speed       | 30    |                                                  |
| Dwell Time                 | 2     | pausa para o Bowden estabilizar a pressão        |
| Align Z                    | Yes   | executa G34 para nivelar a gantry antes do teste |

---

## Passo 3 — Gerar e imprimir

Clique em **Generate** e baixe o arquivo `.gcode`. Envie para a impressora normalmente via OrcaSlicer ou cartão SD.

---

## Passo 4 — Ler o resultado

Analise as linhas da torre impressa:

```
K muito baixo:  cantos arredondados, blob ao mudar direção
K ideal:        cantos a 90°, transição suave entre velocidades
K muito alto:   "orelha" pronunciada nos cantos, aperto excessivo
```

A linha com a transição mais limpa entre a seção lenta (20 mm/s) e a rápida (70 mm/s) é o seu K ideal.

> Para Bowden de 400 mm, o valor costuma ficar entre **0,55 e 0,90**. O valor utilizado nesta impressora após o teste foi **K = 0,80**.

---

## Passo 5 — Aplicar o valor

No OrcaSlicer, no perfil do filamento, atualize:

```json
"pressure_advance": ["0.80"]
```

Ou envie pelo terminal enquanto a impressora está ligada para testar sem reflash:

```
M900 K0.80
M500
```

`M500` salva na EEPROM para o valor persistir após desligar.

---

## Resultado

Após a calibração com K = 0,80, os blobs em círculos pequenos (peças tipo Lego, chaveiros) foram eliminados e os cantos ficaram nítidos mesmo em velocidades de impressão acima de 100 mm/s.
