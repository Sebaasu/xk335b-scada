# Diagrama de Red de Petri General: Sistema Integrado de Manufactura

Este diagrama visualiza la coordinacion global de la planta, donde la **Unidad de Transporte** actua como el eje transversal que gestiona el flujo de piezas entre las estaciones periféricas.

```mermaid
graph TD
    %% --- UNIDAD DE ALIMENTACION (U1) ---
    subgraph U1_Alimentacion
        U1_P0((U1: Reposo)) --> U1_T0[T: Iniciar Dosific.]
        U1_T0 --> U1_P1((U1: Proceso))
        U1_P1 --> U1_T1[T: Pieza Disponible]
        U1_T1 --> U1_P2((U1: Espera Recogida))
        U1_P2 --> U1_T_Hand[T: Brazo_Picks_U1]
        U1_T_Hand --> U1_P0
    end

    %% --- COORDINADOR DE TRANSPORTE (U4) ---
    subgraph U4_Transporte_Transversal
        U4_P0((Brazo Libre))
        U4_T_AP[T: Atender A-P]
        U4_T_PE[T: Atender P-E]
        U4_T_ES[T: Atender E-S]
        
        U4_P_Exec((Ejecutando Tarea))
        
        U4_P0 --> U4_T_AP
        U4_P0 --> U4_T_PE
        U4_P0 --> U4_T_ES
        
        U4_T_AP --> U4_P_Exec
        U4_T_PE --> U4_P_Exec
        U4_T_ES --> U4_P_Exec
        U4_P_Exec --> U4_T_Done[T: Tarea OK]
        U4_T_Done --> U4_P0
    end

    %% --- UNIDAD DE PROCESAMIENTO (U2) ---
    subgraph U2_Procesamiento
        U2_P0((U2: Vacio)) --> U2_T_In[T: Brazo_Drop_U2]
        U2_T_In --> U2_P1((U2: Proceso))
        U2_P1 --> U2_T1[T: Proc. OK]
        U2_T1 --> U2_P2((U2: Espera Recogida))
        U2_P2 --> U2_T_Out[T: Brazo_Picks_U2]
        U2_T_Out --> U2_P0
    end

    %% --- UNIDAD DE ENSAMBLAJE (U3) ---
    subgraph U3_Ensamblaje
        U3_P0((U3: Vacio)) --> U3_T_In[T: Brazo_Drop_U3]
        U3_T_In --> U3_P1((U3: Proceso))
        U3_P1 --> U3_T1[T: Ensa. OK]
        U3_T1 --> U3_P2((U3: Espera Recogida))
        U3_P2 --> U3_T_Out[T: Brazo_Picks_U3]
        U3_T_Out --> U3_P0
    end

    %% --- UNIDAD DE SELECCION (U5) ---
    subgraph U5_Seleccion
        U5_P0((U5: Vacio)) --> U5_T_In[T: Brazo_Drop_U5]
        U5_T_In --> U5_P1((U5: Clasificacion))
        U5_P1 --> U5_T1[T: Fin Proceso]
        U5_T1 --> U5_P0
    end

    %% --- SINCRONIZACION (Handshakes) ---
    U1_P2 -- "Req A-P" --> U4_T_AP
    U2_P0 -- "U2 Ready" --> U4_T_AP
    U4_T_AP -- "Drop U2" --> U2_T_In
    
    U2_P2 -- "Req P-E" --> U4_T_PE
    U3_P0 -- "U3 Ready" --> U4_T_PE
    U4_T_PE -- "Drop U3" --> U3_T_In

    U3_P2 -- "Req E-S" --> U4_T_ES
    U5_P0 -- "U5 Ready" --> U4_T_ES
    U4_T_ES -- "Drop U5" --> U5_T_In

    %% Estilos
    style U4_P0 fill:#f9f,stroke:#333,stroke-width:4px
    style U4_P_Exec fill:#dfd,stroke:#333
```

## Mecanismo de Interacción
1. **Petición (Request):** Una estación pone una marca en su plaza de "Espera Recogida" cuando termina su proceso interno. Esta marca viaja por red (NetW) hacia el PLC de Transporte.
2. **Disponibilidad (Ready):** Una estación pone una marca en "Vacío" cuando no tiene piezas y está lista para recibir una nueva.
3. **Ejecución:** El Coordinador de Transporte solo dispara un traslado si el Origen está listo Y el Destino está vacío Y el Brazo está libre.
4. **Finalización:** Tras el "Drop" (entrega), el Brazo envía un pulso de confirmación para que la estación de destino inicie su ciclo local.
