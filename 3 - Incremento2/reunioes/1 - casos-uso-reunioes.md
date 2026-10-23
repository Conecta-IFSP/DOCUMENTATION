# Diagrama de Casos de Uso - Reuniões

Este documento apresenta os atores e os casos de uso do incremento de reuniões das comissões. A autenticação do usuário é uma pré-condição de todos os casos de uso.

---

## Atores

**Participante:** usuário da plataforma convidado para uma reunião. Consulta a agenda, visualiza a reunião, responde ao convite, registra a própria entrada e saída e consulta a lista de presença.

**Responsável da comissão:** integrante da comissão com papel de responsável, que corresponde ao presidente da comissão descrito no enunciado do incremento. Cada comissão possui exatamente um responsável. Além das ações do participante, agenda reuniões da comissão.

**Organizador:** responsável da comissão que agendou uma reunião específica. Além das ações do participante, cancela a reunião e confirma ou recusa as presenças registradas.

O responsável da comissão e o organizador são especializações do participante. O administrador do sistema não participa das reuniões.

---

## Visão Geral

```mermaid
flowchart LR
    Participante["Participante"]
    Responsavel["Responsável da comissão"]
    Organizador["Organizador"]

    Responsavel -.->|especializa| Participante
    Organizador -.->|especializa| Participante

    subgraph Sistema["Conecta+ - Reuniões"]
        Agendamento(["Agendamento de reuniões"])
        Presenca(["Convite e presença"])
        Lista(["Lista de presença"])
    end

    Participante --> Agendamento
    Participante --> Presenca
    Participante --> Lista
    Responsavel --> Agendamento
    Organizador --> Agendamento
    Organizador --> Presenca

    classDef ator stroke:#1976d2,stroke-width:2px;
    classDef modulo stroke:#7b1fa2,stroke-width:2px;

    class Participante,Responsavel,Organizador ator;
    class Agendamento,Presenca,Lista modulo;
```

---

## 1. Agendamento de Reuniões

```mermaid
flowchart LR
    Participante["Participante"]
    Responsavel["Responsável da comissão"]
    Organizador["Organizador"]

    subgraph Sistema["Conecta+"]
        UC01(["UC01 - Agendar reunião"])
        UC02(["UC02 - Consultar agenda de reuniões"])
        UC03(["UC03 - Visualizar reunião"])
        UC04(["UC04 - Cancelar reunião"])
    end

    Responsavel --> UC01
    Participante --> UC02
    Participante --> UC03
    Organizador --> UC04

    classDef ator stroke:#1976d2,stroke-width:2px;
    classDef uc stroke:#7b1fa2,stroke-width:2px;

    class Participante,Responsavel,Organizador ator;
    class UC01,UC02,UC03,UC04 uc;
```

### Casos de uso

- **UC01 - Agendar reunião:** permite ao responsável da comissão marcar uma reunião e convidar integrantes da comissão.
- **UC02 - Consultar agenda de reuniões:** permite ao participante ver as reuniões para as quais foi convidado, organizadas por dia.
- **UC03 - Visualizar reunião:** permite ao participante ver os dados da reunião, os convidados, as presenças e a pauta.
- **UC04 - Cancelar reunião:** permite ao organizador cancelar uma reunião que ainda não terminou.

---

## 2. Convite e Presença

```mermaid
flowchart LR
    Participante["Participante"]
    Organizador["Organizador"]

    subgraph Sistema["Conecta+"]
        UC05(["UC05 - Responder convite"])
        UC06(["UC06 - Registrar entrada"])
        UC07(["UC07 - Registrar saída"])
        UC08(["UC08 - Confirmar ou recusar presença"])
    end

    Participante --> UC05
    Participante --> UC06
    Participante --> UC07
    Organizador --> UC08

    classDef ator stroke:#1976d2,stroke-width:2px;
    classDef uc stroke:#7b1fa2,stroke-width:2px;

    class Participante,Organizador ator;
    class UC05,UC06,UC07,UC08 uc;
```

### Casos de uso

- **UC05 - Responder convite:** permite ao participante aceitar ou recusar o convite para a reunião.
- **UC06 - Registrar entrada:** permite ao participante informar que chegou à reunião.
- **UC07 - Registrar saída:** permite ao participante informar que saiu da reunião.
- **UC08 - Confirmar ou recusar presença:** permite ao organizador validar a presença registrada por cada participante.

---

## 3. Lista de Presença

```mermaid
flowchart LR
    Participante["Participante"]

    subgraph Sistema["Conecta+"]
        UC09(["UC09 - Gerar lista de presença"])
    end

    Participante --> UC09

    classDef ator stroke:#1976d2,stroke-width:2px;
    classDef uc stroke:#7b1fa2,stroke-width:2px;

    class Participante ator;
    class UC09 uc;
```

### Caso de uso

- **UC09 - Gerar lista de presença:** permite ao participante obter a lista com a situação, a entrada, a saída e o tempo de permanência de cada convidado.
