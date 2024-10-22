# MachinePost
MachinePost was made entirely in Python to run on Google Colab, with the aim of posting 5 text contents with image/gif/video per day during a week on Nostr.  The code is not fully functional, so I need help finalizing it.

(PT/BR) O MachinePost foi feito inteiramente em Python para rodar no Google Colab, com o intuito de postar 5 conteúdos de texto com imagem/gif/vídeo por dia durante uma semana no Nostr. O código não está totalmente funcional, portanto preciso de ajuda para finalizar.

## 1 Cell - Install request for Nostr 
(PT/BR) Instalar requisição para o Nostr
```
!pip install requests nostr
```

## 2 Cell - Nostr PrivateKey generator (if you want a new acount)
(PT/BR) Gerador de chaves Nostr (se quiser uma nova conta)
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

## 3 Cell - Relay and account request [or put you acount}
(PT/BR) Requisição do Relay e da conta [ou bote a sua conta]
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

os.environ["PRIVATE_KEY"] = private_key.hex() # Get it from the previous cell in "private_key.hex()" or change it for your hex private key as 'YourPrivateKeyHere'

env_private_key = os.environ.get("PRIVATE_KEY") 
if env_private_key:
    print('"PRIVATE_KEY" confirmed.')
if not env_private_key:
    print('The environment variable "PRIVATE_KEY" is not set.')
    exit(1)

private_key = PrivateKey(bytes.fromhex(env_private_key))
```

## 4 Cell - Main Data Generation [get images from your Google Drive]
(PT/BR) Geração de Dados Principal [obtenha as imagens pelo seu Google Drive]
```
from google.colab import drive
from google.colab import runtime
from PIL import Image
import os
import re
import random
import datetime
import time
import pytz
import requests



k = 1

def remover_acentos(texto):
    acentos = {'á': 'a', 'é': 'e', 'í': 'i', 'ó': 'o', 'ú': 'u', 'â': 'a', 'ê': 'e', 'ô': 'o', 'û': 'u', 'à': 'a', 'è': 'e', 'ì': 'i', 'ò': 'o', 'ù': 'u', 'ã': 'a', 'õ': 'o', 'ç': 'c'}
    return ''.join([acentos.get(c, c) for c in texto])

# Mecanismo de desligamento
def checagem_encerramento():
    time.sleep(1)
    while True:
        resposta = input("\n>> Deseja ser perguntado se quer continuar o programa ao esperar o próximo post? ('sim' ou 'não'): ")

        if remover_acentos(resposta.strip().lower()) == "sim":
            global k
            k = 0
            time.sleep(1)
            print("\n   Verificação de finalização ativada. O programa exigirá confirmação manual.")
            time.sleep(1)
            print("\n   Continuando...\n")
            return

        elif remover_acentos(resposta.strip().lower()) == "nao":
            time.sleep(1)
            print("\n   Verificação de finalização não ativada. O programa funcionará automaticamente.")
            time.sleep(1)
            print("\n   Continuando...\n")
            return

        else:
            time.sleep(1)
            print("\n   Por favor, responda apenas 'sim' ou 'não'.")

def acessar_drive():
    time.sleep(1)
    while True:
        resposta = input("\n>> Deseja acessar o seu Drive? ('sim' ou 'não'): ")

        if remover_acentos(resposta.strip().lower()) == "sim":
            time.sleep(1)
            print("\n   Conectando...\n")
            return

        elif remover_acentos(resposta.strip().lower()) == "nao":
            time.sleep(1)
            #exit()
            #!kill -9 -1
            print("\n   Chatbot encerrando em 5 segundos", end="")
            time.sleep(1)
            print(".", end="")
            time.sleep(1)
            print(".", end="")
            time.sleep(1)
            print(".", end="")
            time.sleep(1)
            print(".", end="")
            time.sleep(1)
            print(".", end="")
            time.sleep(1)
            runtime.unassign()

        else:
            time.sleep(1)
            print("\n   Por favor, responda apenas 'sim' ou 'não'.")

def upload_to_pomf(image_path):
    url = 'https://pomf.lain.la/upload.php'

    with open(image_path, 'rb') as file:
        files = {'files[]': file}
        response = requests.post(url, files=files)

        if response.status_code == 200:
            response_json = response.json()
            if 'files' in response_json and len(response_json['files']) > 0:
                file_url = response_json['files'][0]['url']
                return file_url
            else:
                print("Erro ao obter o link da imagem")
                return None
        else:
            print(f"Erro ao enviar a imagem: {response.status_code}")
            return None


def encerrar_programa():
    time.sleep(1)
    while True:
        resposta = input("\n>> Deseja continuar? ('sim' ou 'não'): ")

        if remover_acentos(resposta.strip().lower()) == "sim":
            time.sleep(1)
            print("\n   Continuando...")
            time.sleep(1)
            print("\n   Aguarde o horario para o post.")
            return

        elif remover_acentos(resposta.strip().lower()) == "nao":
            time.sleep(1)
            #exit()
            #!kill -9 -1
            print("\n   Chatbot encerrando em 5 segundos", end="")
            time.sleep(1)
            print(".", end="")
            time.sleep(1)
            print(".", end="")
            time.sleep(1)
            print(".", end="")
            time.sleep(1)
            print(".", end="")
            time.sleep(1)
            print(".", end="")
            time.sleep(1)
            runtime.unassign()

        else:
            time.sleep(1)
            print("\n   Por favor, responda apenas 'sim' ou 'não'.")

# Introdução
print("   MachinePost\n")
time.sleep(1)
print("Bem-vindo ao Chatbot MachinePost, o programa que posta conteúdos para você.\n\n")
time.sleep(1)
print("Para continuar é necessário entrar na sua conta do Drive para buscar a uma biblioteca.")
time.sleep(1)
acessar_drive()
time.sleep(1)

# monta o google drive
drive.mount('/content/drive')
print("Conectado ao Drive.\n")
time.sleep(1)

# caminho das imagens no google drive
folder_path = "/content/drive/MyDrive/ABiblioteca"

# Definindo o fuso horário de Brasília
fuso_horario = pytz.timezone('America/Sao_Paulo')

emojis = [
    ["guys! Have a great day 😁", "Nostr! 🐣", "Nostriches! 🦩", "and Pura Vida! ✌️", "Plebchain! 🫂", "Purplers! 💜", "Zappers! ⚡", "Free People! 🍀", "Sovereigns! 👑", "₿itcoiners! 🟠", "Orange Pillers! 💊", "Freedom Squires! 🛡️", "Orangers! 🍊"],
    ["guys! Have a great rest 😁", "Nostr! 🐣", "Nostriches! 🦩", "and Pura Vida! ✌️", "Plebchain! 🫂", "Purplers! 💜", "Zappers! ⚡", "Free People! 🍀", "Sovereigns! 👑", "₿itcoiners! 🟠", "Orange Pillers! 💊", "Freedom Squires! 🛡️", "Orangers! 🍊"]
]

letras_gm = []
letras_gn = []

# Ativa ou desativa a checagem de desligar programa durante a espera de postagem
checagem_encerramento()

# Mecanismo repetição semanal
# loop para repetir o processo 7 vezes, um dia por semana
for i in range(7):
  # obtém a lista de arquivos na pasta em ordem alfabética
  files = sorted(os.listdir(folder_path))

  dia_atual = datetime.date.today() + datetime.timedelta(days=i)

  # Verifica se as mensagens devem ser enviadas dentro do horário real
  agora = datetime.datetime.now(pytz.utc)
  agora_br = agora.astimezone(fuso_horario)

  # Gera horário aleatório com diferença de 0 a 20 minutos a mais
  hora_gm = datetime.datetime(agora_br.year, agora_br.month, agora_br.day, 0, 11, 0) #7h
  hora_gm += datetime.timedelta(minutes=random.randint(0, 1))
  hora_gm += datetime.timedelta(seconds=random.randint(0, 60))
  hora_gm = fuso_horario.localize(hora_gm)

  hora_img1 = datetime.datetime(agora_br.year, agora_br.month, agora_br.day, 8, 47, 0) #8:50
  hora_img1 += datetime.timedelta(minutes=random.randint(0, 20))
  hora_img1 += datetime.timedelta(seconds=random.randint(0, 60))
  hora_img1 = fuso_horario.localize(hora_img1)

  hora_img2 = datetime.datetime(agora_br.year, agora_br.month, agora_br.day, 10, 14, 0) #11:50
  hora_img2 += datetime.timedelta(minutes=random.randint(0, 1))
  hora_img2 += datetime.timedelta(seconds=random.randint(0, 1))
  hora_img2 = fuso_horario.localize(hora_img2)

  hora_img3 = datetime.datetime(agora_br.year, agora_br.month, agora_br.day, 15, 50, 0) #15:50
  hora_img3 += datetime.timedelta(minutes=random.randint(0, 20))
  hora_img3 += datetime.timedelta(seconds=random.randint(0, 60))
  hora_img3 = fuso_horario.localize(hora_img3)

  hora_img4 = datetime.datetime(agora_br.year, agora_br.month, agora_br.day, 18, 43, 0) #18:50
  hora_img4 += datetime.timedelta(minutes=random.randint(0, 1))
  hora_img4 += datetime.timedelta(seconds=random.randint(0, 60))
  hora_img4 = fuso_horario.localize(hora_img4)

  hora_img5 = datetime.datetime(agora_br.year, agora_br.month, agora_br.day, 20, 8, 0) #21:50
  hora_img5 += datetime.timedelta(minutes=random.randint(0, 1))
  hora_img5 += datetime.timedelta(seconds=random.randint(0, 60))
  hora_img5 = fuso_horario.localize(hora_img5)

  hora_gn = datetime.datetime(agora_br.year, agora_br.month, agora_br.day, 23, 52, 0) #23h
  hora_gn += datetime.timedelta(minutes=random.randint(0, 20))
  hora_gn += datetime.timedelta(seconds=random.randint(0, 60))
  hora_gn = fuso_horario.localize(hora_gn)


  print(f"\nHora do bom dia (GM): {hora_gm.hour}:{hora_gm.minute}:{hora_gm.second}")
  print(f"Hora da boa noite (GN): {hora_gn.hour}:{hora_gn.minute}:{hora_gn.second}")

  time.sleep(1)


  # loop para obter as primeiras 5 imagens
  for j in range(5):


    # soma 5 a cada loop em i
    image_path = os.path.join(folder_path, files[j + (i * 5)])

    # obtém o nome do arquivo sem a extensão e substitui "_" por espaço
    filename2 = os.path.splitext(os.path.basename(image_path))
    filename = os.path.splitext(os.path.basename(image_path))[0]
    filename = filename.replace("_", " ")
    filename = filename.replace("§", "\n")
    filename = re.sub(r'\d+\.', '', filename)

    # Verifica se as mensagens devem ser enviadas dentro do horário real
    agora = datetime.datetime.now(pytz.utc)
    agora_br = agora.astimezone(fuso_horario)

    #print(f"Data e hora atual: {agora_br.hour}:{agora_br.minute}:{agora_br.second}")
    print(f"\nData e hora atual: {agora_br.strftime('%d/%m/%Y %H:%M:%S')}")

    time.sleep(1)


    if hora_gm > agora_br:

        diferenca_gm = hora_gm - agora_br
        print(f"\nO horário da mensagem Good Morning é às {hora_gm.hour}:{hora_gm.minute}:{hora_gm.second}")
        print(f"Faltam {diferenca_gm.days} dias, {diferenca_gm.seconds//3600} horas, {(diferenca_gm.seconds//60)%60} minutos e {diferenca_gm.seconds%60} segundos...")

        if k == 0:
            encerrar_programa()

        print("\n   Esperando...")

        time.sleep((hora_gm - agora_br).total_seconds())

        # Seleciona um emoji aleatório sem repetição para GM
        emoji_gm = random.choice(emojis[0])
        while emoji_gm in letras_gm:
            emoji_gm = random.choice(emojis[0])
        letras_gm.append(emoji_gm)

        print(hora_gm.strftime("%d/%m/%Y %H:%M:%S"))


        # Post the GM / Posta o GM
        message = "GM " + str(emoji_gm)
        event = Event(
            content=str(message),
            public_key=private_key.public_key.hex()
        )
        private_key.sign_event(event)
        relay_manager.publish_event(event)

        # Print the complete GM / Imprime o GM completo
        print("GM", emoji_gm)

    # condicional para a primeira imagem
    if hora_img1 > agora_br and (j == 0 or j == 5):

        diferenca_img1 = hora_img1 - agora_br
        print(f"\nO horário da proxima imagem, a imagem 1, é às {hora_img1.hour}:{hora_img1.minute}:{hora_img1.second}")
        print(f"Faltam {diferenca_img1.days} dias, {diferenca_img1.seconds//3600} horas, {(diferenca_img1.seconds//60)%60} minutos e {diferenca_img1.seconds%60} segundos...")

        if k == 0:
            encerrar_programa()

        print("\n   Esperando...")

        time.sleep((hora_img1 - agora_br).total_seconds())
        print("Imagem 1:")


        # Exemplo de uso:
        #image_path = f"{file_path}"
        image_url = upload_to_pomf(image_path)

        if image_url:
            print(f"Imagem enviada com sucesso: {image_url}")
        else:
            print("Erro ao enviar a imagem.")


        # Post the name of the image with the image / Posta o nome da imagem com a imagem
        filename_url = f"{filename}\n\n{image_url}" #filename (Title and/or description) + url
        message = str(filename_url)
        event = Event(
            content=str(message),
            public_key=private_key.public_key.hex()
        )
        private_key.sign_event(event)
        relay_manager.publish_event(event)

        # Print image name / Imprime o nome da imagem
        print(f"\nNome da imagem: {filename}\n")

        # Open and show the image / Abre e exibe a imagem
        img = Image.open(image_path)
        img.show()

    if hora_img2 > agora_br and (j == 1 or j == 6):

        diferenca_img2 = hora_img2 - agora_br
        print(f"\nO horário da proxima imagem, a imagem 2, é às {hora_img2.hour}:{hora_img2.minute}:{hora_img2.second}")
        print(f"Faltam {diferenca_img2.days} dias, {diferenca_img2.seconds//3600} horas, {(diferenca_img2.seconds//60)%60} minutos e {diferenca_img2.seconds%60} segundos...")

        if k == 0:
            encerrar_programa()

        print("\n   Esperando...")

        time.sleep((hora_img2 - agora_br).total_seconds())
        print("Imagem 2:")

        # Exemplo de uso:
        #image_path = f"{file_path}"
        image_url = upload_to_pomf(image_path)

        if image_url:
            print(f"Imagem enviada com sucesso: {image_url}")
        else:
            print("Erro ao enviar a imagem.")

        # posta o nome da imagem com a imagem
        filename_url = f"{filename}\n\n{image_url}" #filename (Title and/or description) + url
        message = str(filename_url)
        event = Event(
            content=str(message),
            public_key=private_key.public_key.hex()
        )
        private_key.sign_event(event)
        relay_manager.publish_event(event)

        # exibe o nome da imagem
        print(f"\nNome da imagem: {filename}\n")

        # abre e exibe a imagem
        img = Image.open(image_path)
        img.show()

    if hora_img3 > agora_br and (j == 2 or j == 7):

        diferenca_img3 = hora_img3 - agora_br
        print(f"\nO horário da proxima imagem, a imagem 3, é às {hora_img3.hour}:{hora_img3.minute}:{hora_img3.second}")
        print(f"Faltam {diferenca_img3.days} dias, {diferenca_img3.seconds//3600} horas, {(diferenca_img3.seconds//60)%60} minutos e {diferenca_img3.seconds%60} segundos...")

        if k == 0:
            encerrar_programa()

        print("\n   Esperando...")

        time.sleep((hora_img3 - agora_br).total_seconds())
        print("Imagem 3:")

        # Exemplo de uso:
        #image_path = f"{file_path}"
        image_url = upload_to_pomf(image_path)

        if image_url:
            print(f"Imagem enviada com sucesso: {image_url}")
        else:
            print("Erro ao enviar a imagem.")

        # posta o nome da imagem com a imagem
        filename_url = f"{filename}\n\n{image_url}" #filename (Title and/or description) + url
        message = str(filename_url)
        event = Event(
            content=str(message),
            public_key=private_key.public_key.hex()
        )
        private_key.sign_event(event)
        relay_manager.publish_event(event)

        # exibe o nome da imagem
        print(f"\nNome da imagem: {filename}\n")

        # abre e exibe a imagem
        img = Image.open(image_path)
        img.show()

    if hora_img4 > agora_br and (j == 3 or j == 8):

        diferenca_img4 = hora_img4 - agora_br
        print(f"\nO horário da proxima imagem, a imagem 4, é às {hora_img4.hour}:{hora_img4.minute}:{hora_img4.second}")
        print(f"Faltam {diferenca_img4.days} dias, {diferenca_img4.seconds//3600} horas, {(diferenca_img4.seconds//60)%60} minutos e {diferenca_img4.seconds%60} segundos...")

        if k == 0:
            encerrar_programa()

        print("\n   Esperando...")

        time.sleep((hora_img4 - agora_br).total_seconds())
        print("Imagem 4:")

        # Exemplo de uso:
        #image_path = f"{file_path}"
        image_url = upload_to_pomf(image_path)

        if image_url:
            print(f"Imagem enviada com sucesso: {image_url}")
        else:
            print("Erro ao enviar a imagem.")

        # posta o nome da imagem com a imagem
        filename_url = f"{filename}\n\n{image_url}" #filename (Title and/or description) + url
        message = str(filename_url)
        event = Event(
            content=str(message),
            public_key=private_key.public_key.hex()
        )
        private_key.sign_event(event)
        relay_manager.publish_event(event)

        # exibe o nome da imagem
        print(f"\nNome da imagem: {filename}\n")

        # abre e exibe a imagem
        img = Image.open(image_path)
        img.show()

    if hora_img5 > agora_br and (j == 4 or j == 9):

        diferenca_img5 = hora_img5 - agora_br
        print(f"\nO horário da proxima imagem, a imagem 5, é às {hora_img5.hour}:{hora_img5.minute}:{hora_img5.second}")
        print(f"Faltam {diferenca_img5.days} dias, {diferenca_img5.seconds//3600} horas, {(diferenca_img5.seconds//60)%60} minutos e {diferenca_img5.seconds%60} segundos...")

        if k == 0:
            encerrar_programa()

        print("\n   Esperando...")

        time.sleep((hora_img5 - agora_br).total_seconds())
        print("Imagem 5:")

        # Exemplo de uso:
        #image_path = f"{file_path}"
        image_url = upload_to_pomf(image_path)

        if image_url:
            print(f"Imagem enviada com sucesso: {image_url}")
        else:
            print("Erro ao enviar a imagem.")

        # posta o nome da imagem com a imagem
        filename_url = f"{filename}\n\n{image_url}" #filename (Title and/or description) + url
        message = str(filename_url)
        event = Event(
            content=str(message),
            public_key=private_key.public_key.hex()
        )
        private_key.sign_event(event)
        relay_manager.publish_event(event)

        # exibe o nome da imagem
        print(f"\nNome da imagem: {filename}\n")

        # abre e exibe a imagem
        img = Image.open(image_path)
        img.show()

    if hora_gn > agora_br and (j == 4 or j == 9):

        diferenca_gn = hora_gn - agora_br
        print(f"\nO horário da mensagem Good Night é às {hora_gn.hour}:{hora_gn.minute}:{hora_gn.second}")
        print(f"Faltam {diferenca_gn.days} dias, {diferenca_gn.seconds//3600} horas, {(diferenca_gn.seconds//60)%60} minutos e {diferenca_gn.seconds%60} segundos...")

        if k == 0:
            encerrar_programa()

        print("\n   Esperando...")

        time.sleep((hora_gn - agora_br).total_seconds())

        # Seleciona um emoji aleatório sem repetição para GN
        emoji_gn = random.choice(emojis[1])
        while emoji_gn in letras_gn:
            emoji_gn = random.choice(emojis[1])
        letras_gn.append(emoji_gn)

        # Imprime a mensagem formatada
        print(hora_gn.strftime("%d/%m/%Y %H:%M:%S"))

        # Post the GN / Posta o GN
        message = "GN " + str(emoji_gn)
        event = Event(
            content=str(message),
            public_key=private_key.public_key.hex()
        )
        private_key.sign_event(event)
        relay_manager.publish_event(event)

        # Print the complete GN / Imprime o GN completo
        print("GN", emoji_gn)
``` 
