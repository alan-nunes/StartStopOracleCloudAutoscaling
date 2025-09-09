# 🚀 Start/Stop de Instâncias na Oracle Cloud (OCI)

Este guia mostra como configurar e gerenciar o ciclo de **start/stop automático de instâncias** na Oracle Cloud Infrastructure (OCI), utilizando **Instance Configurations**, **Instance Pools** e **Autoscaling Configurations**.

## 📌 Pré-requisitos

* Uma conta na [Oracle Cloud](https://www.oracle.com/cloud/).
* Permissão para criar **Instance Configurations**, **Instance Pools** e **Autoscaling Configurations**.
* Rede configurada (VCN e Subnet).
* Instâncias existentes (para anexar ao pool).

---

## 🔹 1. Acessando o menu Compute

- O primeiro passo é criar uma configuração de instância (Instance configurations).

No painel lateral da Oracle Cloud, clique em:

`☰ Menu → Compute → Instance Configurations`

![Compute Menu](./photos/compute-instance-configurations.png)

---

## 🔹 2. Criando uma Instance Configuration

- Vamos criar uma configuração de instância, irei usar os padrões para Imagem e Shape, ele não será importante pois iremos utilizar o configurado na instância criada.
  
1. Clique em **Instance Configurations**.
2. Selecione **Create instance configuration**.

![Create Instance Config](./photos/create-instance-configurations.png)

3. Defina as informações básicas:

   * **Name**: `instance-config-start-stop`
   * **Compartment**: Selecione o compartment desejado.

![Instance Config Info](./photos/instance-configurations-information.png)

4. Clique em **Next** e finalize.
5. Sua configuração estará disponível:

![Instance Config Created](./photos/instance-config-start-stop.png)

---

## 🔹 3. Criando um Instance Pool

Agora vamos criar um **Instance Pool** baseado na configuração criada.

1. Acesse:
   `☰ Menu → Compute → Instance Pools`

![Instance Pool Menu](./photos/compute-instance-pools.png)

2. Clique em **Create instance pool**.

![Create Instance Pool](./photos/create-instance-pools.png)

3. Configure os detalhes:

   * **Name**: `instance-pool-start-stop`
   * **Number of instances**: `0` (para iniciar/parar manualmente quando necessário).
   * **Instance configuration**: Selecione `instance-config-start-stop`.

![Instance Pool Config](./photos/create-instance-pool-basic-details.png)

4. Configure o **Placement**:
   
- Selecione o Domínio (isso é importante), deve ser o mesmo que está a instância que irá fazer parte do Start Stop, se ele estiver em outro domínio você não conseguirá anexar ela futuramente.
  
   * Escolha o **Availability Domain**.
   * Configure o **Primary VNIC** (rede e subnet).

![Placement Config](./photos/create-instance-pool-configure-pool-placement.png)

5. Finalize a criação do pool.
- Ao final do processo, ao clicar em Create, ele irá apresentar um resumo

![Intance Pool Resumo](./photos/instance-pool-start-stop.png)

---

## 🔹 4. Anexando Instâncias ao Instance Pool

Para adicionar instâncias existentes ao pool:

- No momento não tem nenhuma instância anexada, iremos anexar as instâncias que desejamos que faça parte do Start e Stop do agendamento.
  
1. No seu Instance Pool, vá para a seção **Attached instances**
2. Clique em **Attach instance**

![Attach Instance](./photos/Attached-intances.png)

3. Selecione a instância que deseja anexar
4. Verifique se a instância atende aos requisitos:
   - Instância e pool estão em execução
   - Mesmo tipo de máquina (VM ou bare metal)
   - Mesmo availability domain e fault domain
   - VNIC primária na mesma VCN e subnet
   - Não está anexada a outro pool

![Attach Requirements](./photos/attach-instance-to-instance-pool.png)

5. Monitore o progresso em **Work requests** → **AttachInstancePoolInstance**

![Work Request](./photos/instance-pool-attach.png)

---

## 🔹 5. Configurando Auto Scaling para Start/Stop Automático

### Criando Autoscaling Configuration

1. Acesse: `☰ Menu → Compute → Autoscaling Configurations`

![Autoscaling Menu](./photos/compute-autoscallingconfiguration.png)

3. Clique em **Create autoscaling configuration**

![Autoscaling Menu](./photos/create-autoscalling-configuration.png)

3. Configure os detalhes básicos:
   - **Name**: `autoscaling-config-start-stop`
   - **Compartment**: Selecione o compartment
   - **Instance pool**: Selecione `instance-pool-start-stop`

![Autoscaling Basic](./photos/create-autoscalling-configuration-add-basic-details.png)

### Configurando Política de Stop (Desligamento)

1. Selecione **Schedule-based autoscaling**
2. Configure a política de stop:
   - **Policy name**: `autoscaling-policy-stop`
   - **Action**: `Change lifecycle state of all instances`
   - **Lifecycle action**: `Force stop`
   - **Schedule**: Configure o cron expression para o horário desejado

Exemplo de configuração cron:
```
Minute: 30
Hour: 19
Day of month: 1-31
Month: 9
Day of week: ?
Year: 2025
```

![Stop Policy](./photos/compute-autoscalling-configuration-policy.png)

### Configurando Política de Start (Ligamento)

1. Adicione uma segunda política
2. Configure a política de start:
   - **Policy name**: `autoscaling-policy-start`
   - **Action**: `Change lifecycle state of all instances`
   - **Lifecycle action**: `Start`
   - **Schedule**: Configure o cron expression para o horário desejado

Exemplo de configuração cron:
```
Minute: 25
Hour: 19
Day of month: 1-31
Month: 9
Day of week: ?
Year: 2025
```

![Start Policy](./photos/compute-autoscalling-configuration-policy-2.png)

### Configuração Final do Autoscaling

Após configurar ambas as políticas, você terá:

- ✅ Política de stop agendada
- ✅ Política de start agendada  
- ✅ Próximos eventos visíveis no painel

![Final Config](./photos/autoscalling-preview.png)

---

## 🔹 6. Monitoramento e Gerenciamento

### Verificando Instâncias Anexadas
- Acesse seu Instance Pool → **Attached instances**
- Verifique se as instâncias estão listadas corretamente

### Monitorando Work Requests
- Acesse **Work requests** para ver o histórico de operações
- Verifique o status das operações de attach/detach

### Gerenciando o Autoscaling
- Acesse **Autoscaling Configurations** para editar políticas
- Verifique os próximos eventos agendados
- Desabilite políticas temporariamente se necessário

---

## 📋 Entendendo Expressões Cron

As expressões cron na OCI usam o formato:
`<segundo> <minuto> <hora> <dia do mês> <mês> <dia da semana> <ano>`

### 📊 Campos da Expressão Cron

| Campo | Valores Permitidos | Descrição |
|-------|-------------------|-----------|
| Segundo | 0 | **Obrigatório:** Sempre use 0 na OCI |
| Minuto | 0-59 | Minutos da hora |
| Hora | 0-23 | Hora do dia (formato 24h) |
| Dia do mês | 1-31 | Dias do mês |
| Mês | 1-12 ou JAN-DEC | Meses do ano |
| Dia da semana | 1-7 ou SUN-SAT | 1=Domingo, 7=Sábado |
| Ano | 1970-2099 | Ano (opcional) |

### 🔧 Caracteres Especiais

| Caractere | Descrição | Exemplo |
|-----------|-----------|---------|
| `*` | Todos os valores | `*` no mês = todos os meses |
| `-` | Intervalo de valores | `8-17` na hora = 8h às 17h |
| `,` | Múltiplos valores | `MON,WED,FRI` = segunda, quarta, sexta |
| `?` | Valor não específico | Use quando um campo interfere em outro |
| `/` | Incrementos | `0/15` nos minutos = a cada 15 minutos |
| `L` | Último dia | `L` = último dia do mês |
| `W` | Dia útil mais próximo | `15W` = dia útil mais próximo do dia 15 |
| `#` | Enésima ocorrência | `MON#2` = segunda segunda-feira do mês |

### ⏰ Exemplos Práticos

**🟢 Todos os dias às 9h30:**
```
0 30 9 * * ? *
```

**🟢 Dias úteis (segunda a sexta) às 8h:**
```
0 0 8 ? * MON-FRI *
```

**🟢 Final de semana (sábado e domingo) às 10h:**
```
0 0 10 ? * SAT,SUN *
```

**🟢 Primeira segunda-feira do mês às 9h:**
```
0 0 9 ? * 2#1 *
```

**🟢 Todo dia 15 do mês às 18h:**
```
0 0 18 15 * ? *
```

**🟢 A cada 30 minutos, das 8h às 18h:**
```
0 0/30 8-18 * * ? *
```

**🟢 Horário comercial (segunda a sexta, 8h às 18h):**
```
// Início: 8h nos dias úteis
0 0 8 ? * MON-FRI *

// Término: 18h nos dias úteis  
0 0 18 ? * MON-FRI *
```

### 💡 Dicas Importantes

1. **UTC**: Todos os horários são em UTC (converta seu fuso horário)
2. **Segundo**: Sempre use `0` no primeiro campo
3. **Conflitos**: Use `?` quando houver conflito entre dia do mês e dia da semana
4. **Teste**: Sempre verifique o "Next event" para confirmar o agendamento
5. **Formato**: `0 <minuto> <hora> ? * <dias-da-semana> *`

---

## ⚠️ Considerações Importantes

1. **Custos**: Instâncias em pool paradas ainda incorrem em custos de armazenamento
2. **Rede**: Todas as instâncias devem estar na mesma VCN/subnet
3. **Agendamento**: Horários são sempre em UTC
4. **Monitoramento**: Verifique sempre os work requests para garantir sucesso nas operações
5. **Compatibilidade**: Instâncias devem ser do mesmo tipo (VM/bare metal)

---
## 👨‍💻 Autor

**Alan Nunes**  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/alan-sn/)

---

## 📚 Referências

- [Oracle Cloud Documentation - Autoscaling with Cron Expressions](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/autoscalinginstancepools.htm#cron)
- [Start/Stop Instances usando o Autoscaling do OCI - Welton Medici](https://weltonmedici.com.br/start-stop-instances-usando-o-autoscaling-do-oci/)

