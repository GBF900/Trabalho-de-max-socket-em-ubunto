# INSTITUTO FEDERAL DO ESPÍRITO SANTO/ CAMPUS COLATINA

# TRABALHO PARA DISCIPLINA DE REDES
**ORQUESTRADO E MINISTRADA PELO PROFESSOR EDUARDO MAX AMARO AMARAL**

**ALUNO:** Gabriel Ferrari  
**TURMA:** V01

# **OBJETIVO**: COMPREENDER COMO SÃO TRATADOS OS DADOS ENVIADOS E RECEBIDOS POR SOCKET CLIENTE/SERVIDOR

#            --ATIVIDADE 7--                

**7.1**: Os sockets são estruturas de comunicação por Internet Protocols, IP's, que possibilitam a troca de dados entre programas em uma mesma rede.
o intuito é usar o método cliente/servidor para que um servidor , M1, tente "conversar" por meio de sinais a um "cliente", M2. Logo após a mensagem da 
M1 ser correspondida e respondida , a conexão entre as duas é feita e pode-se , então, trocar informações.

**7.2**: Os dados são trasnmitidos via Internet Protocols ,IP's, que fazem com que seja possível saber onde e quem é  máquina que se deseja compartilhar dados 

**7.3**:
1- Reduz drásticamente a probabilidade de vasão de dados sigilosos
2- Manuseio e atualização do script envolvendo menos logística
3- Maior viabilidade de controle

**7.4**:
--- APLICAÇÕES REAIS DESSA TÉCNICA ---
1- SITES WEB
2- JOGOS ONLINE (PRINCIPALEMNTE FPS ,QUE PRECISAM DE PROCESSAMENTO CONTÍNUO)
3- SERVIÇO DE CALENDÁRIO 
4- SERVIÇO DE STREAMING
5- SISTEMAS DE CÓPIA E SEGURANÇA DE DESASTRES

-- MELHORIAS --

**Separar responsabilidades:** Dividir as tarefas entre o cliente e o servidor facilita a manutenção e o desenvolvimento modular.   
**Balancear a carga**: Distribuir as solicitações entre vários servidores. 
**Utilizar caching:** Reduzir a carga do servidor. 
**Utilizar a nuvem:** Serviços como AWS e Azure podem escalar automaticamente.  
**Implementar redundância:** Proteger os dados contra falhas. 
**Controlar o acesso:** Gerenciar o acesso a recursos específicos para diferentes usuários ou grupos de usuários. 
**Aplicar políticas de segurança:** Armazenar e gerenciar informações sensíveis no servidor, onde é possível aplicar criptografia e autenticação de acesso

# ATIVIDADES EXTRAS 

**8.1**:
Abaixo, cÓdigos de servidor e clientes em que o servidor aguarda varios arquivos/dados serem enviados a ele pelo cliente, enquanto o cliente envia 10 arquivos no formato ".txt" :

```
# server.py
import socket
import struct
import os

def recv_exact(sock, num_bytes):
    """Garante a leitura de exatamente num_bytes do socket."""
    data = b''
    while len(data) < num_bytes:
        chunk = sock.recv(num_bytes - len(data))
        if not chunk:
            raise ConnectionError("Conexão perdida.")
        data += chunk
    return data

def start_server(host='127.0.0.1', port=8181):
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind((host, port))
    server_socket.listen(5)
    print(f"[+] Servidor ouvindo em {host}:{port}")

    while True:
        client_socket, addr = server_socket.accept()
        print(f"\n[+] Conexão recebida de {addr}")
        try:
            # Quantidade de arquivos (4 bytes)
            qtd = struct.unpack('!I', recv_exact(client_socket, 4))[0]
            print(f"[i] Recebendo {qtd} arquivos:")

            for i in range(qtd):
                # Nome do arquivo (128 bytes)
                nome = recv_exact(client_socket, 128).decode().strip()
                # Tamanho do arquivo (8 bytes)
                tamanho = struct.unpack('!Q', recv_exact(client_socket, 8))[0]

                print(f"{i+1}. {nome} ({tamanho} bytes)")

                # Recebendo conteúdo
                with open(nome, 'wb') as f:
                    bytes_restantes = tamanho
                    while bytes_restantes > 0:
                        chunk = client_socket.recv(min(1024, bytes_restantes))
                        if not chunk:
                            raise ConnectionError("Transferência interrompida.")
                        f.write(chunk)
                        bytes_restantes -= len(chunk)

                print(f"{nome} salvo com sucesso.")

        except Exception as e:
            print(f"[x] Erro: {e}")
        finally:
            client_socket.close()

start_server()

```

```
# client.py
import socket
import struct
import os

def criar_arquivos_de_teste(pasta, n):
    os.makedirs(pasta, exist_ok=True)
    for i in range(n):
        nome = f"arquivo_{i+1}.txt"
        caminho = os.path.join(pasta, nome)
        with open(caminho, 'w') as f:
            f.write(f"Este é o conteúdo do arquivo {i+1}\n")

def send_files(host='127.0.0.1', port=8181, pasta="/home/ALUNO/arquivos_para_enviar"):
    arquivos = [f for f in os.listdir(pasta) if os.path.isfile(os.path.join(pasta, f))]
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect((host, port))

        # Envia a quantidade de arquivos (4 bytes)
        s.send(struct.pack('!I', len(arquivos)))

        for arquivo in arquivos:
            caminho = os.path.join(pasta, arquivo)
            tamanho = os.path.getsize(caminho)

            print(f"📤 Enviando: {arquivo} ({tamanho} bytes)")

            # Nome do arquivo (128 bytes, preenchido com espaços)
            s.send(arquivo.encode().ljust(128, b' '))

            # Tamanho do arquivo (8 bytes)
            s.send(struct.pack('!Q', tamanho))

            # Conteúdo
            with open(caminho, 'rb') as f:
                while chunk := f.read(1024):
                    s.send(chunk)

        print("\n✅ Todos os arquivos foram enviados.")

# Cria 3 arquivos fictícios e envia
PASTA = "/home/ALUNO/arquivos_para_enviar"
criar_arquivos_de_teste(PASTA, 10)
send_files(pasta=PASTA)
```
