# Mega tutoriel : de l'installation à la première phrase parlée

> [!IMPORTANT]
> **Avis de licence**
> Ce code est publié sous licence **Apache** et tous les poids des modèles sont distribués sous licence **CC-BY-NC-SA-4.0**. Consultez le fichier [LICENSE](../LICENSE) pour plus de détails.

> [!WARNING]
> **Avertissement légal**
> Nous déclinons toute responsabilité quant à toute utilisation illégale de ce code. Veuillez respecter vos lois locales concernant le DMCA et les législations associées.

Ce tutoriel décrit pas à pas comment installer Fish Speech sur une distribution Debian et comment générer une première phrase avec le modèle **OpenAudio S1-mini**.

## 1. Prérequis matériels

- GPU disposant d'au moins **12 Go de mémoire** (recommandé pour l'inférence).
- Système Debian récent.

## 2. Installation des dépendances système

Ouvrez un terminal et mettez à jour vos paquets puis installez les outils nécessaires :

```bash
sudo apt update
sudo apt install -y git wget portaudio19-dev libsox-dev ffmpeg
```

Ces paquets servent à la gestion audio et à la compilation.

## 3. Installation de Miniconda

Pour isoler l'environnement Python, téléchargez Miniconda puis lancez l'installation&nbsp;:

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
# Redémarrez ensuite votre shell ou exécutez :
source ~/miniconda3/bin/activate
```

Suivez les instructions à l'écran pour installer dans `~/miniconda3`.

## 4. Création de l'environnement Python

```bash
conda create -n fish-speech python=3.12
conda activate fish-speech
```

## 5. Récupération du projet et installation

```bash
git clone https://github.com/fishaudio/fish-speech.git
cd fish-speech
pip install -e .
```

L'option `-e` installe le projet en mode développement afin de pouvoir utiliser directement le code local.

## 6. Téléchargement des poids du modèle

Installez l'outil `huggingface-cli` puis téléchargez les poids&nbsp;:

```bash
pip install -U huggingface_hub
huggingface-cli download fishaudio/openaudio-s1-mini --local-dir checkpoints/openaudio-s1-mini
```

Les fichiers sont placés dans le dossier `checkpoints/openaudio-s1-mini`.

## 7. Préparation de la voix de référence

Choisissez un extrait audio de **10 à 30 secondes** représentant la voix souhaitée (par exemple `reference.wav`). Exécutez&nbsp;:

```bash
python fish_speech/models/dac/inference.py \
    -i reference.wav \
    --checkpoint-path checkpoints/openaudio-s1-mini/codec.pth
```

Cette commande génère deux fichiers : `fake.npy` (les jetons de voix) et `fake.wav` (l'audio recodé).

## 8. Génération des jetons sémantiques

```bash
python fish_speech/models/text2semantic/inference.py \
    --text "Bonjour tout le monde" \
    --prompt-text "Bonjour" \
    --prompt-tokens fake.npy \
    --checkpoint-path checkpoints/openaudio-s1-mini
```

Un fichier `codes_0.npy` est créé avec la représentation sémantique de la phrase.

## 9. Synthèse de la phrase parlée

```bash
python fish_speech/models/dac/inference.py \
    -i codes_0.npy \
    --checkpoint-path checkpoints/openaudio-s1-mini/codec.pth
```

Le résultat final est enregistré dans `fake.wav`. Vous pouvez l'écouter avec n'importe quel lecteur audio.

## 10. Interface Web facultative

Pour tester rapidement plusieurs textes via une interface graphique&nbsp;:

```bash
python -m tools.run_webui \
    --llama-checkpoint-path checkpoints/openaudio-s1-mini \
    --decoder-checkpoint-path checkpoints/openaudio-s1-mini/codec.pth
```

Une fois Gradio lancé, un lien local (et possiblement un lien partageable) apparaît dans le terminal.

---

Vous avez maintenant un environnement opérationnel permettant de générer du texte parlé avec une voix personnalisée grâce à **OpenAudio S1-mini**.

