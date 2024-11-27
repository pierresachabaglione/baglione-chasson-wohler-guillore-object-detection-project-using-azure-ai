# Analyse d'images en temps réel avec Azure et OpenCV

Bienvenue dans ce projet passionnant ! Ici, nous allons capturer des images à partir de votre caméra, les analyser avec Azure Cognitive Services pour détecter des objets et générer des légendes, puis afficher les résultats en temps réel.

## Prérequis

Avant de plonger dans le code, assurez-vous d'avoir installé les éléments suivants :

- Python 3.6 ou supérieur
- OpenCV
- Azure Cognitive Services

## Installation des bibliothèques

Pour installer les bibliothèques nécessaires, utilisez `pip` :

```bash
pip install opencv-python azure-ai-vision azure-core
```

## Configuration du client Azure

1. Créez un compte Azure et souscrivez au service Azure Cognitive Services.
2. Notez votre `endpoint` et votre `clé d'API`.

## Utilisation

1. Clonez ce dépôt ou copiez le script dans un fichier Python.

2. Remplacez les valeurs de `endpoint` et `key` par vos propres informations d'identification Azure :

    ```python
    endpoint = "https://votre-endpoint.cognitiveservices.azure.com/"
    key = "votre-clé-d-api"
    ```

3. Exécutez le script :

    ```bash
    python ProjetIA.py
    ```

## Fonctionnement du script

### Importation des bibliothèques

Le script commence par importer les bibliothèques nécessaires pour l'analyse d'images avec Azure, la manipulation de la vidéo avec OpenCV, et l'affichage des images.

```python
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.ai.vision.imageanalysis.models import VisualFeatures
from azure.core.credentials import AzureKeyCredential
import cv2
from time import sleep
```

### Configuration du client Azure

Le client Azure est configuré avec l'endpoint et la clé d'API fournis.

```python
endpoint = "https://votre-endpoint.cognitiveservices.azure.com/"
key = "votre-clé-d-api"

client = ImageAnalysisClient(
    endpoint=endpoint,
    credential=AzureKeyCredential(key)
)
```

### Initialisation de la capture vidéo

La capture vidéo est initialisée à partir de la caméra par défaut et les dimensions du cadre vidéo sont obtenues.

```python
cam = cv2.VideoCapture(0)
frame_width = int(cam.get(cv2.CAP_PROP_FRAME_WIDTH))
frame_height = int(cam.get(cv2.CAP_PROP_FRAME_HEIGHT))
```

### Boucle principale pour la capture et l'analyse des images

La boucle principale capture chaque image de la vidéo, l'encode en JPEG, et envoie les données d'image au service Azure pour analyse.

```python
while True:
    ret, frame = cam.read()
    if not ret:
        print("Failed to grab frame")
        break

    _, buffer = cv2.imencode('.jpg', frame)
    image_data = buffer.tobytes()

    result = client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.OBJECTS, VisualFeatures.CAPTION]
    )
```

### Dessin des objets détectés et affichage des résultats

Les objets détectés sont entourés de rectangles et leurs noms sont affichés. La légende générée par le service Azure est également affichée.

```python
for object in result["objectsResult"]["values"]:
    x1, y1 = object["boundingBox"]['x'], object["boundingBox"]['y']
    x2, y2 = x1 + object["boundingBox"]["w"], y1 + object["boundingBox"]["h"]
    name = object["tags"][0]["name"]
    confidence = object["tags"][0]["confidence"]
    cv2.rectangle(frame, (x1, y1), (x2, y2), (0, 0, 255), 1)
    cv2.putText(frame, name, (x1, y1), cv2.FONT_HERSHEY_DUPLEX, 1, (0, 0, 255), 2)

caption = result["captionResult"]
caption_text = caption["text"]
x = int(frame_width/2 - len(caption_text)*4)
y = 15
cv2.putText(frame, caption_text, (x, y), cv2.FONT_HERSHEY_DUPLEX, 0.5, (0, 0, 255))
```

### Affichage de l'image avec la reconnaissance d'objets AI

L'image avec les objets détectés et la légende est affichée. La boucle s'arrête si la touche 'c' est pressée.

```python
cv2.imshow('Camera with AI Object Recognition', frame)

if cv2.waitKey(1) == ord('c'):
    break

sleep(3)
```

### Libération des ressources

La caméra est libérée et toutes les fenêtres OpenCV sont fermées.

```python
cam.release()
cv2.destroyAllWindows()
```