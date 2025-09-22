# criando-uma-vm-azure-resumo

Esse resumo tem como objetivo orientar para o uso de criação de maquinas virtuais no ambiente azure da microsoft
para duvidas: https://learn.microsoft.com/pt-br/azure/virtual-machines/windows/quick-create-portal#create-virtual-machine (documentação oficial)

# Tutorial: Criar uma Máquina Virtual no Azure (passo a passo)

Abaixo está um tutorial completo e pronto para colar no README do GitHub. Contém instruções pelo **Portal** (interface web) e pelo **Azure CLI**, além de dicas práticas e comandos prontos.

---

## Índice

1. [Pré-requisitos](#pré-requisitos)
2. [Método 1 — Criar pelo Portal (GUI)](#método-1---criar-pelo-portal-gui)
3. [Método 2 — Criar com Azure CLI (linha de comando)](#método-2---criar-com-azure-cli-linha-de-comando)
4. [Como conectar à VM](#como-conectar-à-vm)
5. [Como apagar/limpar recursos](#como-apagarlimpar-recursos)
6. [Dicas e boas práticas](#dicas-e-boas-práticas)
7. [Referências rápidas](#referências-rápidas)

---

## Pré-requisitos

* Conta Microsoft Azure ativa (se não tiver, crie uma assinatura gratuita).
* Permissões na subscription para criar recursos (Resource Groups, VMs, redes, IPs).
* (Para CLI) Azure CLI instalada e configurada (`az login`).

---

## Método 1 — Criar pelo Portal (GUI)

1. Acesse o [Azure Portal](https://portal.azure.com) e faça login.

2. No topo, pesquise por **"Virtual machines"** e abra o serviço.

3. Clique em **+ Create → Azure virtual machine** para abrir o assistente de criação. ([Microsoft Learn][1])

4. **Aba Basics**

   * **Subscription**: escolha a subscription a usar.
   * **Resource group**: selecione um existente ou crie novo.
   * **Virtual machine name**: ex.: `meuVM-01`.
   * **Region**: escolha a região (mais próxima → menor latência).
   * **Availability options**: (opcional) escolha *Availability zone* ou *Availability set* se precisar de alta disponibilidade. ([Microsoft Learn][2])
   * **Image**: escolha SO (ex.: *Ubuntu 22.04 LTS* ou *Windows Server 2022*).
   * **Size**: selecione um tamanho (vCPU / RAM). Use o seletor para comparar; escolha conforme workload. ([Microsoft Learn][3])
   * **Authentication**: para Linux prefira **SSH public key**; para Windows você usará senha/RDP.
   * **Inbound ports**: permita apenas o necessário (SSH 22 para Linux, RDP 3389 para Windows — evitar abrir para 0.0.0.0/0 quando possível).

5. **Aba Disks**

   * Escolha tipo do disco do sistema operacional: *Standard HDD / Standard SSD / Premium SSD / Ultra* conforme desempenho e custo. Managed Disks são recomendados (alto SLA e replicação). ([Microsoft Learn][4])

6. **Aba Networking**

   * Configure Virtual Network (VNet), Subnet, Public IP (se necessário), Network security group (NSG).
   * Verifique regras de entrada (SSH/RDP) e evite regras muito abertas.

7. **Aba Management / Monitoring / Advanced**

   * Opções: Boot diagnostics, Auto-shutdown, Backup, Monitoring (Azure Monitor), Extensions (ex.: agente do VM). Habilite o que for necessário.

8. **Review + Create**

   * Valide, aguarde a validação e clique em **Create**. A implantação começa e em poucos minutos a VM estará pronta. ([Microsoft Learn][1])

---

## Método 2 — Criar com Azure CLI

**Passos rápidos (exemplo Linux):**

```bash
# 1) Login na subscription
az login

# 2) Criar resource group (se precisar)
az group create --name rg-meuVM --location eastus

# 3) Criar a VM (exemplo: Ubuntu LTS, gera chaves SSH automaticamente)
az vm create \
  --resource-group rg-meuVM \
  --name meuVM01 \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys \
  --size Standard_B1s \
  --public-ip-sku Standard
```

* Para criar uma VM Windows (ex.: Windows Server 2022), troque `--image` e forneça `--admin-password` ou outros parâmetros de autenticação.
* O comando `az vm create` também pode criar automaticamente VNet, IP público e NSG, ou você pode criar esses recursos manualmente para maior controle. ([Microsoft Learn][5])

### Exemplos úteis (pos-deploy)

* Abrir porta HTTP (80) no NSG da VM:

```bash
az vm open-port --resource-group rg-meuVM --name meuVM01 --port 80
```

---

## Como conectar à VM

* **Linux (SSH)**:

  ```bash
  ssh azureuser@<PUBLIC_IP>
  ```

  (Use a chave gerada / adicionada durante a criação.)

* **Windows (RDP)**:

  * No portal baixe o arquivo RDP e conecte usando o usuário/senha configurados.

---

## Como apagar / limpar recursos (quando não precisar mais)

> Atenção: apagar evita custos contínuos.

```bash
# Deleta todo o resource group e todos os recursos nele (VM, discos, IPs, NICs, etc.)
az group delete --name rg-meuVM --yes --no-wait
```

---

## Dicas e boas práticas

* **Use chaves SSH** para Linux — mais seguro que senhas.
* **Escolha o tamanho certo**: para testes, tamanhos pequenos (B-series) economizam custos; para produção, planeje CPU/IO/RAM. ([Microsoft Learn][3])
* **Não abra RDP/SSH para todo o mundo** (0.0.0.0/0). Restrinja por IP ou use VPN / Bastion.
* **Use Managed Disks** (melhor disponibilidade e gerenciamento). ([Microsoft Learn][4])
* **Tags**: adicione tags (owner, env, project) para rastrear custos.
* **Backups e monitoramento**: habilite Backup do Azure e Azure Monitor se a VM for crítica.
* Se precisar de alta disponibilidade, considere **Availability Zones / Sets** ou **VM Scale Sets**. ([Microsoft Learn][2])

---

## Referências rápidas

* Guia rápido — criar VM pelo portal (ex.: Linux quickstart). ([Microsoft Learn][1])
* Azure CLI — grupo `az vm` / quick-create. ([Microsoft Learn][5])
* Tamanhos de VM (escolha de tamanho). ([Microsoft Learn][3])
* Managed Disks — overview e tipos. ([Microsoft Learn][4])
* Opções de disponibilidade (Availability sets / zones). ([Microsoft Learn][2])

---

Se quiser, eu adapto esse README para:

* Incluir um **exemplo pronto** com valores (nome, região, tamanho) já preenchidos;
* Gerar um **script Bash** que cria tudo passo-a-passo;
* Adicionar instruções de **segurança** (NSG, Bastion, Azure AD Login).

Qual dessas você prefere que eu coloque já formatada no README?

[1]: https://learn.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-portal?utm_source=chatgpt.com "Quickstart: Create a Linux virtual machine in the Azure portal"
[2]: https://learn.microsoft.com/en-us/azure/virtual-machines/availability?utm_source=chatgpt.com "Availability options for Azure Virtual Machines"
[3]: https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/overview?utm_source=chatgpt.com "Sizes for virtual machines in Azure"
[4]: https://learn.microsoft.com/en-us/azure/virtual-machines/managed-disks-overview?utm_source=chatgpt.com "Overview of Azure Disk Storage - Azure Virtual Machines"
[5]: https://learn.microsoft.com/en-us/cli/azure/vm?view=azure-cli-latest&utm_source=chatgpt.com "az vm"
