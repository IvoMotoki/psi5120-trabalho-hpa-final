# Autoescalamento Horizontal de Pods (HPA) em Kubernetes — Minikube e AWS EKS

Projeto da disciplina **PSI5120 — Tópicos de Computação em Nuvem** (Escola Politécnica, USP).

O projeto implanta o mesmo servidor web, sob a mesma política de Horizontal Pod Autoscaler,
em um cluster local (Minikube) e em um cluster gerenciado na nuvem (AWS EKS), e investiga até
onde o autoescalamento de pods consegue, de fato, entregar capacidade em cada ambiente.

## Artigo final

**[`final-ieee/main.pdf`](final-ieee/main.pdf)** — *Beyond CPU-Based Horizontal Pod Autoscaling:
Empirical Limits of Kubernetes Autoscaling in Local and Managed Cloud Environments* (formato IEEE).

Principais resultados:

- **Minikube:** o HPA atingiu o máximo configurado (10 réplicas), sem pods pendentes.
- **AWS EKS:** o HPA ficou limitado a 5 réplicas desejadas, com até 4 pods pendentes — limite
  de densidade de pods por nó (VPC CNI) das instâncias `t3.micro`, não da política do HPA.
- **Ajuste do HPA (`autoscaling/v2 behavior`):** a política de *scale-down* rápido reduziu em
  25,7% as réplicas-segundo em relação a um baseline comparável; a política agressiva reduziu
  o tempo até o primeiro *scale-up* de 103 s para 70 s.

## Estrutura

```
final-ieee/              Artigo final (IEEE): LaTeX, dados derivados, figuras e scripts de análise
  data/                  CSVs e tabelas gerados a partir dos logs brutos
  figures/               Gráficos usados no artigo
  manifests/             Variantes do HPA do experimento de ajuste (baseline, agressivo, scale-down rápido)
  scripts/               Parser dos logs, geração de gráficos e execução das variantes
final-project-research/  Pesquisa bibliográfica e planejamento do projeto final
manifests/               Deployment, Service e HPA base — usados nos dois ambientes
eks/                     Definição do cluster (eksctl) e Service LoadBalancer do EKS
scripts/                 Automação: criação dos clusters, geração de carga e limpeza
evidencias/              Logs brutos de todas as execuções (kubectl get hpa/pods/top a cada 10 s)
roteiro/                 Roteiro passo a passo de implantação e testes
artigo/                  Artigo da etapa intermediária (formato SBC)
```

## Como reproduzir

Passo a passo completo em [`roteiro/ROTEIRO.md`](roteiro/ROTEIRO.md) e em
[`final-ieee/README.md`](final-ieee/README.md). Resumo:

```bash
# Ambiente local (Minikube)
./scripts/minikube_setup.sh
./scripts/load_test.sh minikube hpa-2026
./scripts/cleanup.sh minikube

# Ambiente em nuvem (AWS EKS) — gera custo real enquanto o cluster existir
./scripts/eks_setup.sh
./scripts/load_test.sh eks <contexto-eks> 240 360 3
./scripts/cleanup.sh eks

# Experimento de ajuste do HPA (Minikube)
bash final-ieee/scripts/run_minikube_hpa_variant.sh fast_scaledown final-ieee/manifests/hpa-fast-scaledown.yaml hpa-2026 240 720 6

# Dados e figuras do artigo a partir dos logs brutos
python final-ieee/scripts/parse_hpa_timeline.py
python final-ieee/scripts/plot_hpa_timeline.py
```

## Aplicação alvo

Os dois ambientes usam a imagem oficial do walkthrough de HPA do Kubernetes
(`registry.k8s.io/hpa-example`): um servidor PHP/Apache cujo endpoint executa um cálculo
intensivo de CPU a cada requisição, o que permite gerar carga controlada e comparável.

## Autor

Ivo Motoki Makiyama Lopes — ivo.motoki@usp.br
