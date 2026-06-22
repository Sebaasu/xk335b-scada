# Unidad de Procesamiento - XK335B
## Diagrama de la Red de Petri (CIPN)

---

## 1. Diagrama Mermaid

```mermaid
flowchart TD

    INIT(["INIT SM0.1\nS M0.0 - R M0.1..7"])
    INIT --> P0

    P0(["P0 - Reposo\nM0.0\nSalidas: ninguna\nQ1.1=1 si sin pieza"])
    P1(["P1 - Sujecion\nM0.1\nQ0.0=1 V_Pinza\nQ1.0=1 Lamp_Verde"])
    P2(["P2 - Traslacion a Prensa\nM0.2\nQ0.0=1 Q0.2=1\nQ1.0=1"])
    P3(["P3 - Prensado Baja\nM0.3\nQ0.0=1 Q0.2=1 Q0.3=1\nQ1.0=1"])
    P4(["P4 - Prensado Sube\nM0.4\nQ0.0=1 Q0.2=1\nQ1.0=1"])
    P5(["P5 - Traslacion a Origen\nM0.5\nQ0.0=1\nQ1.0=1"])

    T0{{"T0: M0.0\nAND I1.3 AND I1.4\nAND I0.0"}}
    T1{{"T1: M0.1\nAND I0.1"}}
    T2{{"T2: M0.2\nAND I0.3"}}
    T3{{"T3: M0.3\nAND I0.5"}}
    T4{{"T4: M0.4\nAND I0.4"}}
    T5{{"T5: M0.5\nAND I0.2"}}

    EMERG["EMERGENCIA\nI1.4 = 0\nR M0.1..7\nS M0.0\nQ1.1=1"]

    P0 --> T0
    T0 --> P1
    P1 --> T1
    T1 --> P2
    P2 --> T2
    T2 --> P3
    P3 --> T3
    T3 --> P4
    P4 --> T4
    T4 --> P5
    P5 --> T5
    T5 --> P0

    P1 -. "NOT I1.4" .-> EMERG
    P2 -. "NOT I1.4" .-> EMERG
    P3 -. "NOT I1.4" .-> EMERG
    P4 -. "NOT I1.4" .-> EMERG
    P5 -. "NOT I1.4" .-> EMERG
    EMERG --> P0

    style P0 fill:#1a1a2e,color:#eaeaea,stroke:#4a90d9
    style P1 fill:#16213e,color:#eaeaea,stroke:#4a90d9
    style P2 fill:#16213e,color:#eaeaea,stroke:#4a90d9
    style P3 fill:#0f3460,color:#eaeaea,stroke:#e94560
    style P4 fill:#16213e,color:#eaeaea,stroke:#4a90d9
    style P5 fill:#16213e,color:#eaeaea,stroke:#4a90d9
    style INIT fill:#2d4a22,color:#eaeaea,stroke:#7ec850
    style EMERG fill:#4a1010,color:#eaeaea,stroke:#e94560
    style T0 fill:#2a2a2a,color:#eaeaea,stroke:#888
    style T1 fill:#2a2a2a,color:#eaeaea,stroke:#888
    style T2 fill:#2a2a2a,color:#eaeaea,stroke:#888
    style T3 fill:#2a2a2a,color:#eaeaea,stroke:#888
    style T4 fill:#2a2a2a,color:#eaeaea,stroke:#888
    style T5 fill:#2a2a2a,color:#eaeaea,stroke:#888
```

---

## 2. Tabla de Plazas

| Plaza | Address | Simbolo            | Acciones Q activas                     | Descripcion resumida                         |
|-------|---------|--------------------|----------------------------------------|----------------------------------------------|
| P0    | M0.0    | P0_Reposo          | (ninguna)                              | Sistema en espera, carro origen, pinza abierta|
| P1    | M0.1    | P1_Sujecion        | Q0.0, Q1.0                            | Pinza cierra sobre la pieza en origen        |
| P2    | M0.2    | P2_Traslacion_In   | Q0.0, Q0.2, Q1.0                      | Carro se traslada con pieza hacia la prensa  |
| P3    | M0.3    | P3_Prensado_Baja   | Q0.0, Q0.2, Q0.3, Q1.0               | Prensa desciende (operacion de prensado)     |
| P4    | M0.4    | P4_Prensado_Sube   | Q0.0, Q0.2, Q1.0                      | Prensa asciende, carro permanece en prensa   |
| P5    | M0.5    | P5_Traslacion_Out  | Q0.0, Q1.0                            | Carro retorna a origen con pieza sujeta      |

---

## 3. Tabla de Transiciones

| Trans. | Address | Simbolo           | Condicion completa                              |
|--------|---------|-------------------|-------------------------------------------------|
| T0     | M1.0    | T0_Inicio         | M0.0 AND I1.3 AND I1.4 AND I0.0               |
| T1     | M1.1    | T1_Sujeto         | M0.1 AND I0.1                                  |
| T2     | M1.2    | T2_Carro_Prensa   | M0.2 AND I0.3                                  |
| T3     | M1.3    | T3_Prensa_Abajo   | M0.3 AND I0.5                                  |
| T4     | M1.4    | T4_Prensa_Arriba  | M0.4 AND I0.4                                  |
| T5     | M1.5    | T5_Carro_Origen   | M0.5 AND I0.2                                  |

---

## 4. Tabla de Sensores por Plaza

| Plaza | Sensor de llegada | Address | Descripcion                           |
|-------|-------------------|---------|---------------------------------------|
| P0    | (marcado inicial) | -       | Activado por SM0.1 o retorno de T5   |
| P1    | I0.1              | M2.2    | Pinza cerrada confirmada              |
| P2    | I0.3              | M2.4    | Cil largo retraido = carro en prensa  |
| P3    | I0.5              | M2.5    | Cil delgado extendido = prensa abajo  |
| P4    | I0.4              | M2.6    | Cil delgado retraido = prensa arriba  |
| P5    | I0.2              | M2.3    | Cil largo extendido = carro en origen |

---

## 5. Notas del Diagrama

- Las lineas punteadas representan la ruta de emergencia que puede activarse desde cualquier plaza P1 a P5.
- La condicion de emergencia (NOT I1.4) tiene prioridad sobre cualquier estado de la red.
- Al disparar la emergencia, el marcado es forzado a P0 (R M0.1..7, S M0.0) y todas las salidas Q quedan en 0.
- El nodo INIT representa el primer ciclo de scan (SM0.1=1) que establece el marcado inicial.
- P3 esta resaltado en rojo por ser la plaza de mayor riesgo (prensa activa).
