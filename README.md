# AWS Hands-on: Gerenciamento, Backup e Recuperação de Dados com Amazon EBS

Este repositório contém a documentação prática do laboratório focado na administração de volumes **Amazon EBS (Elastic Block Store)**, criação de snapshots para backup e restauração de dados em instâncias **Amazon EC2**.

---

## 🎯 Objetivos do Laboratório
* Criar e anexa um volume EBS adicional a uma instância Linux EC2.
* Formatar o volume com o sistema de arquivos `ext3` e configurá-lo para montagem automática via `/etc/fstab`.
* Simular criação de dados e realizar o backup através de **Snapshots do EBS**.
* Simular a perda acidental de arquivos no ambiente.
* Restaurar um Snapshot em um novo volume EBS e recuperar com sucesso os dados perdidos.

---

## 🛠️ Tecnologias e Conceitos Utilizados
* **AWS Services:** Amazon EC2, Amazon EBS (Volumes gp2/gp3 & Snapshots), EC2 Instance Connect.
* **Linux Administration:** `mkfs`, `mount`, `/etc/fstab`, manipulação de diretórios e arquivos de bloco.
* **Estratégias de DR (Disaster Recovery):** Backup consistente em blocos e restore rápido.

---

## 🚀 Passo a Passo e Validação

### 1. Criação e Anexo de Volume EBS
Foi criado um volume EBS de 1 GiB (`gp2`) na mesma *Availability Zone* da instância EC2 de destino e anexado ao identificador `/dev/sdb`.

![Visão Geral dos Volumes EBS](images/01-ebs-volumes-overview.png)

---

### 2. Formatação e Montagem no Linux
No terminal da instância EC2, o volume foi formatado e montado no ponto `/mnt/data-store`:

```bash
# Formatação do volume
sudo mkfs -t ext3 /dev/sdb

# Criação do ponto de montagem e atribuição no fstab
sudo mkdir /mnt/data-store
sudo mount /dev/sdb /mnt/data-store
echo "/dev/sdb   /mnt/data-store ext3 defaults,noatime 1 2" | sudo tee -a /etc/fstab
```

### 3. Backup via Snapshot e Simulação de Incidente
Após gerar dados no volume (/mnt/data-store/file.txt), foi disparado o backup pontual (Snapshot).

Para simular uma falha operacional, o arquivo de dados foi apagado manualmente do volume original:

```bash
sudo rm /mnt/data-store/file.txt
ls /mnt/data-store/file.txt
# Output: ls: cannot access /mnt/data-store/file.txt: No such file or directory
```

### 4. Restauração do Snapshot e Recuperação dos Dados
A partir do Snapshot criado, um novo volume foi provisionado e anexado à instância no ponto /dev/sdc. Em seguida, o volume foi montado em /mnt/data-store2 para recuperar os dados mantidos no estado exato da captura do snapshot.

```bash
sudo mkdir /mnt/data-store2
sudo mount /dev/sdc /mnt/data-store2
ls /mnt/data-store2/file.txt
```

🧠 Aprendizados Chave
- Ciclo de vida do EBS: Entendimento sobre a independência entre a vida útil da instância EC2 e dos volumes EBS anexados.

- Persistência de montagem: Importância de configurar corretamente o /etc/fstab para garantir disponibilidade do volume em reboots.

- Resiliência e DR: Snapshots do EBS armazenam apenas os blocos alterados de forma incremental no S3, garantindo alta eficiência e recuperação ágil de dados em cenários de falha humana ou corrupção de sistema.
