# MachinePost
MachinePost was made entirely in Python to run on Google Colab, with the aim of posting 5 text contents with image/gif/video per day during a week on Nostr.  The code is not fully functional, so I need help finalizing it.

(PT/BR) O MachinePost foi feito inteiramente em Python para rodar no Google Colab, com o intuito de postar 5 conteúdos de texto com imagem/gif/vídeo por dia durante uma semana no Nostr. O código não está totalmente funcional, portanto preciso de ajuda para finalizar.

## 1 Cell - Install request for Nostr / Instalar requisição para o Nostr
```
!pip install requests nostr
```

## 2 Cell - PrivateKey generator / Gerador de chaves Nostr
```
from nostr.key import PrivateKey

private_key = PrivateKey()
public_key = private_key.public_key
print(f"Private key: {private_key.bech32()}")
print(f"Public key: {public_key.bech32()}")
print("--------------------")
print(f"Private key: {private_key.hex()}")
print(f"Public key: {public_key.hex()}")
```

## 3 Cell - Relay and account request / Requisição do Relay e da conta
```
import requests
import time
import ssl
import os
from nostr.event import Event
from nostr.relay_manager import RelayManager
from nostr.message_type import ClientMessageType
from nostr.key import PrivateKey

relay_manager = RelayManager()
relay_manager.add_relay("wss://relay.damus.io")
relay_manager.open_connections({"cert_reqs": ssl.CERT_NONE}) # NOTE: This disables ssl certificate verification
time.sleep(1.25) # allow the connections to open

os.environ["PRIVATE_KEY"] = private_key.hex()

env_private_key = os.environ.get("PRIVATE_KEY") # Put your PRIVATE_KEY or just get it from the previous cell.
if not env_private_key:
    print('The environment variable "PRIVATE_KEY" is not set.')
    exit(1)

private_key = PrivateKey(bytes.fromhex(env_private_key))
```

## 4 Cell -  Initial creation of JavaScript objects (JSON) / Criação inicial dos objetos JavaScript (JSON)
```
import json

#antes: "mediaPath": "https://nostr.build"

json_string = '''{
  "media": {
    "apiPath": "https://nostr.build/api",
    "mediaPath": "https://nostr.build",
    "acceptedMimetypes": [
      "image/jpg",
      "image/png",
      "image/gif",
      "video/mp4"
    ],
    "contentPolicy": {
      "allowAdultContent": true,
      "allowViolentContent": true
    }
  }
}'''

# Analisar o JSON em um objeto Python
data = json.loads(json_string)

# Acessar as informações do objeto Python
api_path = data['media']['apiPath']
media_path = data['media']['mediaPath']
mimetypes = data['media']['acceptedMimetypes']
allow_adult = data['media']['contentPolicy']['allowAdultContent']
allow_violent = data['media']['contentPolicy']['allowViolentContent']

# Imprimir os valores
print(api_path)
print(media_path)
print(mimetypes)
print(allow_adult)
print(allow_violent)
```

## 5 Cell - Test with images saved in Google Drive / Teste com imagens salvas no Google Drive
```
from google.colab import drive
import time
import os

# monta o google drive
drive.mount('/content/drive')
print("Conectado ao Drive.\n")
time.sleep(1)

# caminho das imagens no google drive
path = "/content/drive/MyDrive/ABiblioteca/1.modif2.png" 

# nome da imagem com .[formato]
filename2 = os.path.splitext(os.path.basename(path))
```

## 6 Cell - Image/video media upload request in nostr.build (I didn't understand how this works) 
/ Requisição de upload de midia de imagem/video no nostr.build (não entendi como isso funciona)
```
import os
import requests

# Caminho para o arquivo da imagem
file_path = f"{path}"

# Definir os cabeçalhos da requisição
boundary = "<boundary>"
file_name = f"{filename2}" #os.path.basename(file_path) 
file_mime_type = "image/png"
headers = {
    "Content-Type": f"multipart/form-data; boundary={boundary}"
}

# Ler o arquivo binário da imagem
with open(file_path, "rb") as f:
    file_binary_data = f.read()

# Definir o corpo da requisição
body = f"--{boundary}\r\n" \
       f"Content-Disposition: form-data; name=\"file\"; filename=\"{file_name}\"\r\n" \
       f"Content-Type: {file_mime_type}\r\n\r\n" \
       f"{file_binary_data}\r\n" \
       f"--{boundary}--"

# Enviar a requisição POST
response = requests.post(api_path, headers=headers, data=body)

# Imprimir o código de status e o conteúdo da resposta
print(response.status_code)
print(response.content)
```

## 7 Cell - [Don't work] Attempting to get the URL of the uploaded image / [Não funciona] Tentativa de obter a URL da imagem enviada
```
import os
import requests

# Extrair a URL do corpo da resposta (em formato JSON) ((EXPERIMENTAL))
response_dict = json.loads(response.content)
url = response_dict["url"]

# Imprimir o link HTTP da imagem enviada ((EXPERIMENTAL))
print(url)
```

## 8 Cell - Analyzes the media sent to nostr.build 
(the intention is to analyze the URL received in the previous cell, the entered url was manually copied from a manually uploaded image)
/ Analiza a midia enviada manualmente para o nostr.build 
(a intenção é analizar o URL recebido na celula anterior, a url inserida foi copiada manualmente de uma imagem upada manualmente)
```
#<media_path>/<sha256>/<filename>.<type>?<query_params>
#f'{media_path}/{sha256}/{filename2}?{query_params}'
#f'https://nostr.build/{sha256}/{filename2}?width=800&quality=90'

import requests
from PIL import Image
import tempfile

url = 'https://nostr.build/i/nostr.build_2603027c7ac856090fea4999c6711ad35490b1b0c6b255637c82bb2460681f10.jpg'

response = requests.get(url)

if response.status_code == 200:
    with open('myimage.jpg', 'wb') as f:
        f.write(response.content)
    print("Imagem enviada:")
    img = Image.open(f.name)
    img.show()   
    
else:
    print(f'Erro ao baixar a imagem: {response.status_code}')
```
