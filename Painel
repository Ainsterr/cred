import firebase_admin
from firebase_admin import credentials, firestore
from datetime import datetime, timedelta
from colorama import Fore, Style, init
from telegram import Bot
import time

# Inicialização
init(autoreset=True)

# Telegram
BOT_TOKEN = '7929245095:AAGt5h6bcZjfKhjzxjLMJwIxM80B6lQnB5k'
GRUPO_LINK = 'https://t.me/+nAZ_6QTY1mU1MmMx'
bot = Bot(token=BOT_TOKEN)

# Firebase
cred = credentials.Certificate("serviceAccountKey.json")
firebase_admin.initialize_app(cred)
db = firestore.client()

def print_sucesso(msg):
    print(Fore.GREEN + msg)

def print_erro(msg):
    print(Fore.RED + msg)

def print_info(msg):
    print(Fore.CYAN + msg)

def converter_para_minutos(duracao):
    try:
        if duracao.endswith("m"):
            return int(duracao[:-1])
        elif duracao.endswith("h"):
            return int(duracao[:-1]) * 60
        elif duracao.endswith("d"):
            return int(duracao[:-1]) * 1440
    except:
        return None

def adicionar_acesso():
    user_id = input("ID do usuário Telegram: ")
    username = input("Username (sem @): ")
    tempo = input("Validade (ex: 30m, 2h, 1d): ")

    minutos = converter_para_minutos(tempo)
    if minutos is None:
        print_erro("Formato inválido.")
        return

    expiracao = datetime.utcnow() + timedelta(minutes=minutos)
    db.collection("users").document(user_id).set({
        "username": username,
        "tempo_expiracao": expiracao.isoformat(),
        "duracao": tempo,
        "status": "ativo"
    })

    # Tenta enviar link para o grupo
    try:
        bot.send_message(chat_id=user_id, text=f"Bem-vindo! Acesse o grupo: {GRUPO_LINK}")
        print_sucesso("Acesso adicionado e link enviado ao usuário.")
    except Exception as e:
        print_erro(f"Erro ao enviar link do grupo: {e}")
        print_info("O usuário deve iniciar uma conversa com o bot primeiro.")

def editar_acesso():
    user_id = input("ID do usuário a editar: ")
    campo = input("Campo (username ou duracao): ")
    valor = input("Novo valor: ")

    if campo == "duracao":
        minutos = converter_para_minutos(valor)
        if minutos is None:
            print_erro("Formato inválido.")
            return
        nova_exp = datetime.utcnow() + timedelta(minutes=minutos)
        db.collection("users").document(user_id).update({
            "duracao": valor,
            "tempo_expiracao": nova_exp.isoformat()
        })
    else:
        db.collection("users").document(user_id).update({campo: valor})

    print_sucesso("Acesso editado com sucesso.")

def remover_acesso():
    user_id = input("ID do usuário a remover: ")
    db.collection("users").document(user_id).delete()
    print_sucesso("Acesso removido com sucesso.")

def renovar_acesso():
    user_id = input("ID do usuário a renovar: ")
    tempo = input("Tempo adicional (ex: 30m, 2h, 1d): ")
    minutos = converter_para_minutos(tempo)
    if minutos is None:
        print_erro("Formato inválido.")
        return

    doc = db.collection("users").document(user_id).get()
    if not doc.exists:
        print_erro("Usuário não encontrado.")
        return

    dados = doc.to_dict()
    exp_atual = datetime.fromisoformat(dados["tempo_expiracao"])
    nova_exp = exp_atual + timedelta(minutes=minutos)

    db.collection("users").document(user_id).update({
        "tempo_expiracao": nova_exp.isoformat()
    })
    print_sucesso("Acesso renovado com sucesso.")

def checar_expirados():
    now = datetime.utcnow()
    users = db.collection("users").stream()
    for user in users:
        dados = user.to_dict()
        expira = datetime.fromisoformat(dados["tempo_expiracao"])
        if expira < now and dados["status"] == "ativo":
            db.collection("users").document(user.id).update({"status": "expirado"})
            print_info(f"Acesso expirado: {user.id} - @{dados['username']}")

def listar_usuarios():
    print("\n" + "-"*40)
    print(Fore.YELLOW + "LISTA DE USUÁRIOS")
    print("-"*40)
    users = db.collection("users").stream()
    for user in users:
        dados = user.to_dict()
        print(
            f"{Fore.WHITE}ID: {user.id} | "
            f"@{dados['username']} | "
            f"Status: {Fore.GREEN if dados['status']=='ativo' else Fore.RED}{dados['status']} | "
            f"Expira: {Fore.CYAN}{dados['tempo_expiracao']}"
        )
    print("-"*40)

def menu():
    while True:
        print("\n" + Fore.MAGENTA + "=== PAINEL DE ACESSOS - TELEGRAM ===")
        print(Fore.YELLOW + "1 - Adicionar acesso")
        print("2 - Editar acesso")
        print("3 - Remover acesso")
        print("4 - Renovar acesso")
        print("5 - Listar usuários")
        print("6 - Verificar expirados")
        print("0 - Sair")
        opcao = input(Fore.BLUE + "Escolha uma opção: ")

        if opcao == "1":
            adicionar_acesso()
        elif opcao == "2":
            editar_acesso()
        elif opcao == "3":
            remover_acesso()
        elif opcao == "4":
            renovar_acesso()
        elif opcao == "5":
            listar_usuarios()
        elif opcao == "6":
            checar_expirados()
        elif opcao == "0":
            print_info("Saindo...")
            break
        else:
            print_erro("Opção inválida.")

if __name__ == "__main__":
    menu()
