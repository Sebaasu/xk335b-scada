# Unidad de Seleccion - Diagrama de la Red de Petri
Sistema de Manufactura Flexible XK335B | S7-200 CPU 224XP CN

---

## 1. Diagrama Flowchart

```mermaid
flowchart TD

    %% Marcado inicial
    INIT([SM0.1: Inicializacion\nK1=97.26 K2=25.64 N=97.26\nSBR1:HSC_INIT\nATCH INT0 cada 10ms]) --> P0

    %% Plazas
    P0(["P0 - Reposo\nM0.0\nSP = 0.0"])
    P1(["P1 - Avanzar\nM0.1\nSP = 710.0"])
    P2(["P2 - Ident al vuelo\nM0.2\nSP sin cambio\nLeer I0.4 e I0.5\nCargar VD300"])
    P3(["P3 - Viaje a Rampa\nM0.3\nSP = VD300"])
    P4(["P4 - Expulsar\nM0.4\nVD300=710 -> Q0.4\nVD300=1150 -> Q0.5\nVD300=1520 -> Q0.6"])
    P5(["P5 - Final de Ciclo\nM0.5\nSP = 0.0\nReset encoder"])

    %% Transiciones
    T0{"T0 - M1.0\nM0.0 AND I1.3\nAND I1.4 AND I0.3"}
    T1{"T1 - M1.1\nM0.1 AND\nDTR(HC0) >= 490.0"}
    T2{"T2 - M1.2\nM0.2\n(inmediato)"}
    T3{"T3 - M1.3\nM0.3 AND\n(DTR(HC0)+10.0) >= VD300"}
    T4{"T4 - M1.4\nM0.4 AND\n(I0.7 OR I1.0 OR I1.1)"}
    T5{"T5 - M1.5\nM0.5\n(inmediato)"}

    %% Flujo principal
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

    %% Clasificacion en P2
    CLAS["Clasificacion al vuelo:\nI0.5=1 AND I0.4=1 -> VD300=710 (Metal)\n(I0.4=1 AND NOT I0.5) OR\n(NOT I0.4 AND I0.5) -> VD300=1150 (Mixto)\nNOT I0.4 AND NOT I0.5 -> VD300=1520 (Plastico)"]
    P2 --- CLAS

    %% Bloque emergencia
    EMERG["EMERGENCIA (I1.4=0)\nS M0.0 - R M0.1..M0.5\nSP=0.0 - R Q0.0 - R Q0.4-Q0.6\nINT0: CRETI inmediato"]

    %% Modulo de control INT0
    INT0["INT0 - Control Realimentacion Estados (Ts=10ms)\nSeguridad: LDN I1.4 -> CRETI\nPos: DTR(HC0) -> VD100\nVel: (VD100-VD104)*100 -> VD108\nu = N*VD112 - K1*VD100 - K2*VD108\nDireccion Q0.1 segun signo de u\nSaturacion |u| <= 32000 -> AQW0\nMarcha Q0.0: si |u| > 100\nVD104 = VD100"]

    P0 -.-> EMERG
    P1 -.-> EMERG
    P2 -.-> EMERG
    P3 -.-> EMERG
    P4 -.-> EMERG
    P5 -.-> EMERG

    P0 -. "cada 10ms" .-> INT0
    P1 -. "cada 10ms" .-> INT0
    P2 -. "cada 10ms" .-> INT0
    P3 -. "cada 10ms" .-> INT0
    P4 -. "cada 10ms" .-> INT0
    P5 -. "cada 10ms" .-> INT0

    %% Estilos
    style P0 fill:#1a73e8,color:#fff,stroke:#0d47a1
    style P1 fill:#1a73e8,color:#fff,stroke:#0d47a1
    style P2 fill:#1a73e8,color:#fff,stroke:#0d47a1
    style P3 fill:#1a73e8,color:#fff,stroke:#0d47a1
    style P4 fill:#1a73e8,color:#fff,stroke:#0d47a1
    style P5 fill:#1a73e8,color:#fff,stroke:#0d47a1
    style T0 fill:#f9a825,color:#000,stroke:#f57f17
    style T1 fill:#f9a825,color:#000,stroke:#f57f17
    style T2 fill:#f9a825,color:#000,stroke:#f57f17
    style T3 fill:#f9a825,color:#000,stroke:#f57f17
    style T4 fill:#f9a825,color:#000,stroke:#f57f17
    style T5 fill:#f9a825,color:#000,stroke:#f57f17
    style CLAS fill:#e8f5e9,color:#1b5e20,stroke:#388e3c
    style INT0 fill:#fce4ec,color:#880e4f,stroke:#c2185b
    style EMERG fill:#ffebee,color:#b71c1c,stroke:#c62828
    style INIT fill:#ede7f6,color:#4527a0,stroke:#7b1fa2
```

---

## 2. Tabla de Plazas - Resumen

| Plaza | Marca | Nombre      | Setpoint VD112 | Accion Adicional                                        |
|-------|-------|-------------|----------------|---------------------------------------------------------|
| P0    | M0.0  | P0_Reposo   | 0.0            | Ninguna (cinta parada, espera inicio)                   |
| P1    | M0.1  | P1_Avanzar  | 710.0          | Cinta avanza hacia zona de sensores                     |
| P2    | M0.2  | P2_Ident    | Sin cambio     | Lee I0.4 e I0.5, carga VD300 segun clasificacion        |
| P3    | M0.3  | P3_Viaje    | VD300          | Cinta lleva pieza hasta la coordenada de la rampa       |
| P4    | M0.4  | P4_Expulsar | Sin cambio     | Activa Q0.4, Q0.5 o Q0.6 segun VD300                   |
| P5    | M0.5  | P5_Final    | 0.0            | Reset encoder: SMD38=0, HSC 0                           |

---

## 3. Tabla de Transiciones - Condiciones Exactas

| Trans. | Marca | Nombre          | Condicion AWL resumida                                   |
|--------|-------|-----------------|----------------------------------------------------------|
| T0     | M1.0  | T0_Inicio       | M0.0 AND I1.3 AND I1.4 AND I0.3                          |
| T1     | M1.1  | T1_Pos500       | M0.1 AND (DTR HC0 AR>= 490.0)                            |
| T2     | M1.2  | T2_Clasificado  | M0.2 (sin condicion extra, disparo en siguiente scan)    |
| T3     | M1.3  | T3_Target       | M0.3 AND (DTR HC0 +R 10.0 AR>= VD300)                   |
| T4     | M1.4  | T4_Push_OK      | M0.4 AND (I0.7 OR I1.0 OR I1.1)                          |
| T5     | M1.5  | T5_Fin_Ciclo    | M0.5 (sin condicion extra, disparo en siguiente scan)    |

---

## 4. Tabla de Coordenadas de Rampa

| Material     | I0.5 | I0.4 | VD300 (cuentas) | Cilindro | Salida |
|--------------|------|------|-----------------|----------|--------|
| Metal puro   | 1    | 1    | 710.0           | Rampa 1  | Q0.4   |
| Mixto        | 1    | 0    | 1150.0          | Rampa 2  | Q0.5   |
| Mixto        | 0    | 1    | 1150.0          | Rampa 2  | Q0.5   |
| Plastico     | 0    | 0    | 1520.0          | Rampa 3  | Q0.6   |

---

## 5. Parametros del Controlador

| Parametro | Variable | Valor  | Descripcion                                      |
|-----------|----------|--------|--------------------------------------------------|
| K1        | VD120    | 97.26  | Ganancia de realimentacion de posicion           |
| K2        | VD124    | 25.64  | Ganancia de realimentacion de velocidad          |
| N         | VD132    | 97.26  | Ganancia de pre-compensacion de referencia       |
| Ts        | SMB34    | 10 ms  | Periodo de muestreo (interrupcion temporizada)   |
| Sat. max  | -        | 32000  | Saturacion de la accion de control (= 10 V)      |
| Zona muerta| -       | 100    | Umbral minimo de accion para activar motor       |
