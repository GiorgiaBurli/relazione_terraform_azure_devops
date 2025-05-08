### Distribuzione del calcolo su più Virtual Machine con K3s

### Obiettivo

L’obiettivo è dimostrare come sia possibile realizzare un'infrastruttura di **calcolo distribuito** utilizzando **più Virtual Machine (VM)** collegate in un **cluster Kubernetes**, orchestrato con **K3s**. Tale infrastruttura consente di suddividere carichi di lavoro tra più nodi, migliorando l’efficienza, la scalabilità e l’affidabilità del sistema.

### Cos’è il calcolo distribuito

Il **calcolo distribuito** è un paradigma in cui più nodi (o macchine) lavorano insieme per completare un compito complesso suddividendolo in parti più piccole. Ogni nodo esegue una porzione del lavoro in parallelo con gli altri.

### Tecnologia utilizzata: K3s

**K3s** è una distribuzione leggera di Kubernetes, progettata per ambienti con risorse limitate, ideale per:

* Ambienti di test e sviluppo
* Piccoli cluster
* Distribuzione edge o su cloud a basso costo

K3s semplifica l’installazione di Kubernetes, mantenendone le funzionalità principali, inclusa la **distribuzione automatica dei carichi di lavoro** tra più nodi.

### Architettura proposta

1. **1 nodo master (control plane)** – che gestisce lo stato del cluster
2. **2 o più nodi agent** – VM che eseguono i container (pod)

Ogni nodo è una **VM creata su Azure** (o altro cloud), configurata per far parte del cluster K3s.

### Configurazione del cluster

* Il master viene installato con:

```bash
curl -sfL https://get.k3s.io | sh -s - --docker
```

* Gli agenti si uniscono al cluster con:

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://<MASTER_IP>:6443 K3S_TOKEN=<TOKEN> sh -s - --docker
```

* Il token è recuperabile dal master:

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

### Distribuzione del carico

Una volta che le VM sono nel cluster, si può distribuire il carico tramite:

* **Job**: per operazioni batch (elaborazioni dati, simulazioni, rendering)
* **Deployment con più repliche**: per applicazioni web o microservizi scalabili

#### Esempio di `job.yaml`:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: image-processing
spec:
  parallelism: 3
  completions: 3
  template:
    spec:
      containers:
        - name: processor
          image: my-processing-image
          command: ["python", "process.py", "input.csv"]
      restartPolicy: Never
```

Con `parallelism: 3`, Kubernetes esegue 3 job contemporaneamente, distribuiti automaticamente tra le VM.

### Vantaggi dell’approccio

* **Scalabilità**: è possibile aggiungere o rimuovere nodi facilmente
* **Alta disponibilità**: se un nodo fallisce, i carichi vengono riprogrammati altrove
* **Efficienza**: sfruttamento parallelo delle risorse
* **Automazione**: il Kubernetes scheduler assegna i pod ai nodi disponibili

### Conclusione

L’utilizzo di K3s su più VM rappresenta una soluzione semplice e potente per realizzare calcolo distribuito. È adatto a casi d’uso reali come elaborazioni massicce, microservizi, AI/ML e simulazioni. L'infrastruttura può essere definita come **Infrastructure as Code** con Terraform, e gestita tramite pipeline CI/CD.

### Allegati consigliati

* `job.yaml` (esempio workload distribuito)
* `main.tf` (per creare le VM su Azure)
* `install_k3s.sh` (script di installazione per i nodi)
