# Design Docs — Piperaceae Segmentation

## 1. Visão geral
Sistema de segmentação semântica baseado em U-Net para extração de máscaras de órgãos vegetais (Piperaceae), com suporte a treinamento, avaliação e inferência.

## 2. Objetivos
- Segmentar automaticamente a região de interesse em imagens de herbário.
- Produzir máscaras e imagens segmentadas com fundo branco/transparente.
- Gerar métricas quantitativas por fold e estatísticas agregadas.

## 3. Escopo
**Inclui**:
- Treinamento com K-Fold.
- Avaliação com métricas clássicas de segmentação.
- Exportação de resultados e artefatos.
- Inferência com modelos previamente treinados.

**Não inclui**:
- Treinamento com backbones pré-treinados.
- Interface gráfica de usuário.
- Deploy em produção.

## 4. Arquitetura
- **Pipeline de treino**: `main.py`
- **Modelo**: `model.py` (U-Net manual)
- **Métricas/Perdas**: `metrics.py`
- **Persistência de resultados**: `save.py`, `files.py`
- **Pós-processamento e exportação de imagens**: `image.py`
- **Inferência (CLI)**: `predict/predict.py`
- **Agregação de resultados**: `results/results.py`

## 5. Fluxo de dados
1. Carrega imagens/máscaras e normaliza.
2. Divide dados em K-Fold e cria `train/val/test`.
3. Treina U-Net com *callbacks*.
4. Avalia por fold e salva métricas.
5. Gera máscaras e imagens segmentadas.
6. Consolida resultados (csv/xlsx).

## 6. Configurações principais
`cfg` define parâmetros de treinamento:
- `channel`, `image_size`, `batch_size`, `epochs`, `learning_rate`.
- `fold`, `val_size`, `test_size`.
- `loss_function` (`dice` ou `jaccard`).
- `data_augmentation` (liga/desliga augmentations).

## 7. Considerações de implementação
- **Sem backbone**: U-Net construída manualmente.
- **Normalização**: imagens e máscaras em $[0,1]$.
- **Compatibilidade GPU**: `MirroredStrategy` e growth de memória.

## 8. Artefatos gerados
- Pesos do modelo por fold (`unet.h5`).
- Curvas de perda.
- Máscaras e imagens segmentadas.
- Relatórios `csv`/`xlsx` com métricas.

## 9. Pontos de extensão
- Adicionar backbones pré-treinados (ex.: ResNet, EfficientNet).
- Ativar e calibrar aumento de dados (`data_augmentation=True`).
- Suporte a novas métricas (ex.: IoU por classe).

## 10. Riscos e limitações
- Dependência de caminhos absolutos no script de inferência.
- Dataset não versionado no repositório.
- Ausência de validação cruzada estratificada por classe.
