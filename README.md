# MachinePost
MachinePost was made entirely in Python to run on Google Colab, with the aim of posting 5 text contents with image/gif/video per day during a week on Nostr.  The code is not fully functional, so I need help finalizing it. 
(PT/BR) O MachinePost foi feito inteiramente em Python para rodar no Google Colab, com o intuito de postar 5 conteúdos de texto com imagem/gif/vídeo por dia durante uma semana no Nostr. O código não está totalmente funcional, portanto preciso de ajuda para finalizar.

#1 Cell - Install request for Nostr / Instalar requisição para o Nostr
```
!pip install requests nostr
```

#2 Cell - PrivateKey generator / Gerador de chaves Nostr
from nostr.key import PrivateKey

private_key = PrivateKey()
public_key = private_key.public_key
print(f"Private key: {private_key.bech32()}")
print(f"Public key: {public_key.bech32()}")
print("--------------------")
print(f"Private key: {private_key.hex()}")
print(f"Public key: {public_key.hex()}")

#3 Cell - Requisição do Relay e conta
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
