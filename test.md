Test file

``` mermaid
sequenceDiagram
    participant ECU1_SWC1
    participant ECU1_SWC2
    participant ECU1
    participant ECU2
    participant ECU2_SWC1
    participant ECU2_SWC2

    Note right of ECU1_SWC1: Scheduling time of Runnable1.1 = 100ms
    Note right of ECU1_SWC2: Scheduling time of Runnable1.2 = 100ms
    ECU1_SWC1->>ECU1_SWC1: Runnable1.1 executes (2ms)
    ECU1_SWC2->>ECU1_SWC2: Runnable1.2 executes (1ms)
    ECU1_SWC2->>ECU1: Send frame over CAN (5ms delay)
    ECU1->>ECU2: Forward frame (5ms delay)
    Note right of ECU2: Frame received after 113ms
    ECU2->>ECU2_SWC1: Deliver frame (5ms delay)
    Note right of ECU2_SWC1: Scheduling time of Runnable2.1 = 200ms
    Note right of ECU2_SWC2: Scheduling time of Runnable2.2 = 200ms
    ECU2_SWC1->>ECU2_SWC1: Runnable2.1 executes (3ms)
    ECU2_SWC2->>ECU2_SWC2: Runnable2.2 executes (2ms)
    ECU2_SWC2->>ECU2: Send frame over CAN (5ms delay)
    ECU2->>ECU1: Forward frame (5ms delay)
    Note right of ECU1: Frame received after 233ms
    ECU1->>ECU1_SWC1: Deliver frame (5ms delay)
    Note right of ECU1_SWC1: Runnable1.1 scheduled again after 100ms (at 333ms)
    Note right of ECU1_SWC2: Runnable1.2 scheduled again after 100ms (at 333ms)
    Note right of ECU2_SWC1: Runnable2.1 scheduled again after 200ms (at 433ms)
    Note right of ECU2_SWC2: Runnable2.2 scheduled again after 200ms (at 433ms)

```

``` mermaid
graph TD
    subgraph ECU1
        ECU1_Start --> ECU1_Runnable1.1{Runnable1.1}
        ECU1_Runnable1.1 --> |Executes in 2ms| ECU1_Runnable1.2{Runnable1.2}
        ECU1_Runnable1.2 --> |Executes in 1ms| ECU1_WaitForCANInterval{Wait for 100ms CAN interval}
        ECU1_WaitForCANInterval --> ECU1_SendFrame[Send CAN Frame]
        ECU1_SendFrame --> |5ms delay| ECU2_ReceiveFrame
    end

    subgraph ECU2
        ECU2_ReceiveFrame[Receive CAN Frame] --> |5ms delay| ECU2_DeliverFrame{Deliver Frame}
        ECU2_DeliverFrame --> |5ms delay| ECU2_Runnable2.1{Runnable2.1}
        ECU2_Runnable2.1 --> |Executes in 3ms| ECU2_Runnable2.2{Runnable2.2}
        ECU2_Runnable2.2 --> |Executes in 2ms| ECU2_SendFrame[Send CAN Frame]
        ECU2_SendFrame --> |5ms delay| ECU1_ReceiveFrame
    end

    ECU1_ReceiveFrame[Receive CAN Frame] --> |5ms delay| ECU1_DeliverFrame{Deliver Frame}
    ECU1_DeliverFrame --> |5ms delay| ECU1_Runnable1.1

    subgraph Scheduling
        ECU1_Runnable1.1 --> |Scheduled again after 50ms| ECU1_Runnable1.1
        ECU1_Runnable1.2 --> |Scheduled again after 50ms| ECU1_Runnable1.2
        ECU2_Runnable2.1 --> |Scheduled again after 50ms| ECU2_Runnable2.1
        ECU2_Runnable2.2 --> |Scheduled again after 50ms| ECU2_Runnable2.2
    end

    ECU1_WaitForCANInterval --> |Next CAN Frame after 100ms| ECU1_WaitForCANInterval
```

``` mermaid
sequenceDiagram
    participant SWC1.1
    participant SWC1.2
    participant RTE_ECU1
    participant COM_ECU1
    participant CAN_Bus
    participant COM_ECU2
    participant RTE_ECU2
    participant SWC2.1
    participant SWC2.2

    SWC1.1->>SWC1.2: Data Preparation
    Note right of SWC1.1: Execution Time

    SWC1.2->>RTE_ECU1: Virtual Bus Communication
    Note right of RTE_ECU1: RTE Latency

    RTE_ECU1->>COM_ECU1: Send Data
    Note right of COM_ECU1: COM-stack Latency

    COM_ECU1->>CAN_Bus: Send Frame
    Note right of CAN_Bus: CAN Bus Transmission Time

    CAN_Bus-->>COM_ECU2: Propagation Delay
    Note right of COM_ECU2: COM-stack Latency

    COM_ECU2->>RTE_ECU2: Deliver Data
    Note right of RTE_ECU2: RTE Latency

    RTE_ECU2->>SWC2.1: Virtual Bus Communication
    SWC2.1->>SWC2.2: Data Processing
    Note right of SWC2.2: Execution Time

    Note right of SWC2.2: Response Communication (if applicable)

```


