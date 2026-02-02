# Resumo científico — Segmentação de Piperaceae

## Objetivo
Este projeto realiza segmentação semântica de órgãos vegetais (Piperaceae) a partir de imagens digitais, com o objetivo de isolar a região de interesse (máscara) para análises morfológicas posteriores e geração de imagens segmentadas com fundo transparente ou branco.

## Dados e pré-processamento
- Imagens e máscaras são carregadas a partir de pastas externas (conforme `images_folder` e `masks_folder`).
- Normalização de intensidade por $1/255$.
- Suporte a imagens em **grayscale** (`channel = 1`) e **RGB** (`channel = 3`).

## Modelo
- Arquitetura **U-Net** implementada manualmente, **sem backbone pré-treinado**.
- Blocos de contração e expansão com `Conv2D`, `BatchNormalization`, `ReLU`, `Dropout`, `MaxPooling2D` e `Conv2DTranspose`.

## Configurações usadas (treinamento)
As configurações abaixo estão definidas em `cfg`:
- `channel`: 1 (grayscale)
- `batch_size`: 4
- `fold`: 5 (K-Fold)
- `epochs`: 75
- `image_size`: 256
- `learning_rate`: 0.001
- `random_state`: 1234
- `test_size`: 0.2
- `val_size`: 0.05
- `path_dataset`: `../dataset`
- `path_out`: `out`
- `loss_function`: `dice`
- `data_augmentation`: `False`

## Treinamento e validação
- Estratégia de múltiplas GPUs com `MirroredStrategy` quando disponível.
- K-Fold para avaliação robusta: o conjunto completo é dividido em 5 *folds*; em cada iteração, 1 fold é usado como teste e os 4 restantes como treino. A partir do treino, é feita uma nova divisão interna para validação (`val_size = 0.05`). Esse ciclo se repete até que todos os folds tenham sido usados como teste, produzindo métricas por fold e médias ao final.
- Divisão interna de validação via `train_test_split`.
- Otimizador **Adam**.
- *Callbacks*:
	- `ReduceLROnPlateau`: monitora a função de perda durante o treino e reduz a taxa de aprendizado quando a perda deixa de melhorar por um número de épocas. No projeto, é configurado para observar `loss`, reduzir a taxa por um fator de 0,5 e aguardar 3 épocas de paciência antes de ajustar, evitando estagnação no processo de otimização.
	- `ModelCheckpoint`: salva automaticamente o melhor modelo ao longo do treinamento. Aqui ele grava o arquivo `unet.h5` no diretório do fold, com `save_best_only=True`, preservando o conjunto de pesos que apresentou melhor desempenho durante o treino.

## Métricas
- `dice_coef` (manual): calculado a partir da sobreposição entre máscara real ($y$) e predita ($\hat{y}$). Define-se a interseção $I = \sum |y \cdot \hat{y}|$ e a união $U = \sum (|y| + |\hat{y}|)$. O coeficiente Dice é $\text{Dice} = \frac{2I + s}{U + s}$, onde $s$ é um termo de suavização.
- `jaccard_distance` (manual): usa as mesmas quantidades, com $\text{Jaccard} = \frac{I + s}{U - I + s}$. Esse valor corresponde ao índice de Jaccard (IoU) com suavização.
- `precision` (Keras/TensorFlow): métrica padrão de precisão ($\frac{TP}{TP+FP}$).
- `recall` (Keras/TensorFlow): métrica padrão de revocação ($\frac{TP}{TP+FN}$).

## Saídas geradas
- Pesos do modelo por fold (`unet.h5`).
- Curvas de perda (`lossgraph`).
- Máscaras preditas e imagens segmentadas com fundo transparente e branco.
- Planilhas de resultados (`csv`/`xlsx`) com métricas por fold e estatísticas agregadas.

## Inferência
A inferência utiliza modelos já treinados e salva máscaras e imagens segmentadas em diretórios de saída específicos, com suporte a RGB ou grayscale e múltiplos tamanhos de imagem.
