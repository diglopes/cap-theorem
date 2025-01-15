# O teorema de CAP

O teorema de CAP, também conhecido como o teorema de Brewer, é um conceito fundamental em sistemas distribuídos. Ele afirma que um sistema distribuído não pode garantir simultaneamente todas as três seguintes propriedades:

**Consistência (Consistency):** Todos os nós do sistema veem os mesmos dados ao mesmo tempo. Isso significa que uma leitura sempre retorna o valor mais recente após uma escrita bem-sucedida.

**Disponibilidade (Availability):** Cada solicitação feita ao sistema recebe uma resposta, mesmo que algumas partes do sistema estejam indisponíveis. Ou seja, o sistema está sempre operacional.

**Tolerância a Partições (Partition Tolerance):** O sistema continua a operar mesmo se houver falhas de comunicação ou "partições" na rede que impedem que os nós se comuniquem entre si.

## Regras do teorema de CAP:

Dado que falhas de rede (partições) são inevitáveis em sistemas distribuídos reais, um sistema sempre terá que priorizar entre Consistência e Disponibilidade.
É impossível atingir simultaneamente as três propriedades em um sistema que está sujeito a falhas de comunicação.
Exemplos de sistemas distribuídos:

### CP (Consistência e Tolerância a Partições):

Sacrificam a disponibilidade. Quando ocorre uma partição, o sistema pode se tornar indisponível para manter a consistência.

> Exemplo: Bancos de dados como HBase, MongoDB configurado para consistência forte.

### AP (Disponibilidade e Tolerância a Partições):

Sacrificam a consistência. Durante uma partição, os nós podem responder com dados desatualizados (inconsistentes).

> Exemplo: Sistemas como Cassandra, DynamoDB.

### CA (Consistência e Disponibilidade):

Não toleram partições. Esses sistemas dependem de redes confiáveis e falhas podem tornar o sistema completamente indisponível.

> Exemplo: Sistemas centralizados, como alguns bancos de dados relacionais.

### Implicações práticas:

Nenhum sistema distribuído pode ter 100% de consistência, disponibilidade e tolerância a partições ao mesmo tempo. Os desenvolvedores precisam decidir qual propriedade priorizar com base nos requisitos do sistema. Por exemplo:
- Um sistema financeiro pode priorizar consistência (CP).
- Um sistema de redes sociais pode priorizar disponibilidade (AP).
  
Essa escolha é muitas vezes contextual e depende das necessidades específicas da aplicação e do ambiente onde ela opera.

## Testes práticos

### Cenário:
Configurar um cluster de um banco de dados distribuído, como o Cassandra (AP) ou o MongoDB (CP). Em seguida, simular partições de rede e observar o comportamento do sistema em termos de consistência, disponibilidade e tolerância a partições

#### Passo 1: Instalar o Docker
Certifique-se de que o Docker está instalado no seu ambiente. Guia oficial de instalação.

#### Passo 2: Subir um cluster distribuído
Aqui, usaremos MongoDB como exemplo, configurado para replicação (um cluster com réplicas).

Criar a rede Docker:
```bash
docker network create cap-test
```

Subir três instâncias do MongoDB:
```bash
docker run -d --name mongo1 --network cap-test mongo --replSet rs0
docker run -d --name mongo2 --network cap-test mongo --replSet rs0
docker run -d --name mongo3 --network cap-test mongo --replSet rs0
```

Ou inicialize tudo pelo docker compose

```bash
docker compose up
```

**Inicializar o cluster de réplicas:**
Conecte-se ao container mongo1 para configurar o cluster:

```bash
docker exec -it mongo1 mongosh
```

No shell do MongoDB:

```javascript
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo1:27017" },
    { _id: 1, host: "mongo2:27017" },
    { _id: 2, host: "mongo3:27017" }
  ]
});
```

#### Passo 3: Simular cenários CAP

**1. Testar Consistência e Partição (CP):**
- Simule uma partição de rede desconectando um dos nós:

```bash
docker network disconnect cap-test mongo2
```
- Tente realizar leituras e escritas no cluster. Por padrão, o MongoDB prioriza consistência e pode se tornar indisponível para certas operações até resolver a partição

**2. Testar Disponibilidade e Partição (AP):**
- Configure o MongoDB para permitir operações de escrita mesmo em caso de partição. No **mongo1**, execute:

```javascript
rs.status().members.map(i => ({ name: i.name, health: i.health, state: i.stateStr }));
```
- Simule uma partição novamente:

```bash
docker network disconnect cap-test mongo3
```

- Realize leituras e escritas. Você pode observar que algumas leituras podem retornar dados desatualizados.

**3. Testar Consistência e Disponibilidade (CA):**
- Sem simular partições, observe que o MongoDB mantém consistência e disponibilidade.


#### Passo 4: Monitorar logs

Acompanhe os logs dos containers para entender o comportamento durante as partições:

```bash
docker logs -f mongo1
docker logs -f mongo2
docker logs -f mongo3
```

### Passo 5: Limpar o ambiente

Após os testes, remova os containers e a rede:

```bash
docker rm -f mongo1 mongo2 mongo3
docker network rm cap-test
```

ou

```bash
docker compose down
```
---

## Conclusão
Esse experimento ajuda a visualizar como sistemas distribuídos priorizam Consistência, Disponibilidade ou Tolerância a Partições em cenários reais. Com pequenos ajustes, você pode usar outros sistemas distribuídos, como Cassandra, Redis Cluster ou Etcd, para explorar diferentes comportamentos CAP.