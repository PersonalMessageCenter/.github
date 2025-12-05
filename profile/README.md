# Personal Message Center

Um centro de controle de mensagens com capacidades de armazenar metadados sobre o recebimento de mensagens, processar comandos e realizar acoes.

## V0

Iniciamente, foi testado um MVP para apenas salvar em um banco de dados:

```mermaid
flowchart TD
    %% participants 
    wpp@{ shape: bow-rect, label: "wpp-socket" }
    c["wppdataretriever"]
    db@{shape: cyl, label: "database<br>(postgres)"}

    %% step 1 - communication between communicator and socket
    wpp l1@==> |"when new message arrive (temporary simulating)"| c

    l1@{ animate: true }
    c --> |publish| db
```

Essa versao rodou por aproximadamente XXXXX meses em um [servidor](link do homebox) com a seguinte arquitetura:
- [ ] TODO: Confirmar a arquitetura

Esses foram os dados obtidos:
- [ ] Tratar os dados .csv
- [ ] Ver uma boa forma de deixar publico

## Problemas
Essa versao inicial pode me mostrar que:

- [ ] Confirmar hipotese: qual o throughput de mensagem? porque eu tenho a sensacao que muitas nao sao salvas... teste de carta?

## V1

Com base nesses problema, construi essa versao inicial, que consiste nas seguintes aplicacoes:

- [wpp-infra](https://github.com/PersonalMessageCenter/wpp-infra): repositorio com a infraestrutura (inicialmente em Docker)
- [wpp-communicator](https://github.com/PersonalMessageCenter/wpp-communicator): servico responsavel por comunicar com o socket
- [wpp-data-processor](https://github.com/PersonalMessageCenter/wpp-data-processor): servico responsavel por persistir os metadados no banco de dados
- [wpp-command-processor](https://github.com/PersonalMessageCenter/wpp-command-processor): servico responsavel por executar comandos

Esse e um desenho da arquitetura atual:

```mermaid
flowchart TD
    %% participants 
    wpp@{ shape: bow-rect, label: "wpp-socket" }
    c["communicator"]
    ex_1@{shape: processes, label: "wpp.exchange.messages" }
    q_1@{shape: das, label: "wpp.queue.persistence" }
    q_2@{shape: das, label: "wpp.queue.commands" }
    q_3@{shape: das, label: "wpp.queue.outgoing" }
    db@{shape: cyl, label: "database<br>(postgres)"}
    ext@{ shape: dbl-circ, label: "external<br> world, apis,<br>etc" }

    %% step 1 - communication between communicator and socket
    wpp l1@==> |"when new message arrive (temporary simulating)"| c

    l1@{ animate: true }
    c --> |publish| ex_1
    
    ex_1 --> q_1
    q_1 --> data-processor
    data-processor --> |save message metadata| db

    ex_1 --> q_2
    q_2 --> command-processor 
    command-processor --> |"(optional)"| db
    command-processor --> |"(optional)"| ext
    command-processor --> |"(optional) if need to message someone"| q_3
    
    q_3 --> c
    
    c --> |send message| wpp
```
