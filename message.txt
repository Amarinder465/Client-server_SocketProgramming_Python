import socket
import threading

allClients = []


def handle_client(client_socket,client_address):
    client_id = len(allClients)
    while True:
        data = client_socket.recv(1024).decode('utf-8')
        if not data:
            break
        print(f"Message received from {client_address[0]}")
        print(f"Sender's Port: {client_address[1]}")
        print(f"Message: \"{data}\"")

    client_socket.close()



def process_command(command):
    if command.startswith("list"):
        list_connections()
    elif command.startswith("terminate"):
        terminate_connection(command)
    elif command.startswith("send"):
        send_message(command)
    elif command == "help":
        display_help()
    elif command == "myip":
        print(f"Server IP Address: {get_ip()}")
    elif command == "myport":
        print(f"Server Port: {server_port}")
    elif command.startswith("connect"):
        connect_to_client(command)
    else:
        print("Unknown command. Type 'help' for list of available commands.")


def list_connections():
    print("Connected allClients:")
    for i, client in enumerate(allClients):
        print(f"ID: {i}, Address: {client.getpeername()}")


def terminate_connection(command):
    partsLen = command.split()
    if len(partsLen) == 2:
        try:
            target_id = int(partsLen[1])
            if 0 <= target_id < len(allClients):
                target_client = allClients[target_id]
                target_client.close()
                allClients.remove(target_client)
                print(f"Connection with Client {target_id} terminated.")
            else:
                print("Invalid client ID.")
        except ValueError:
            print("Invalid client ID.")
    else:
        print("Usage: terminate <ID>")


def send_message(command):
    partsLen = command.split()
    if len(partsLen) > 2:
        try:
            target_id = int(partsLen[1])
            if 0 <= target_id < len(allClients):
                message = " ".join(partsLen[2:])
                target_client = allClients[target_id]
                target_client.send(message.encode('utf-8'))
                print(f"Message sent to Client {target_id}: {message}")
            else:
                print("Invalid client ID.")
        except ValueError:
            print("Invalid client ID.")
    else:
        print("Usage: send <ID> <Message>")


def display_help():
    print("Available commands:")
    print("help                  - Display information about the available user interface options or command manual.")
    print("myip                  - Display the server's IP address.")
    print("myport                - Display the port on which the server is listening for incoming connections.")
    print("connect <dest> <port> - Establish a new TCP connection to the specified destination at the specified port.")
    print("list                  - Display a numbered list of all the connections this server is part of.")
    print("terminate <id>        - Terminate the connection with the specified id.")
    print("send <id> <message>   - Send a message to the connection with the specified id.")
    print("exit                  - Close all connections and terminate the server.")


def connect_to_client(command):
    partsLen = command.split()
    if len(partsLen) == 3:
        try:
            dest = partsLen[1]
            port = int(partsLen[2])
            client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            try:
                client_socket.connect((dest, port))
                allClients.append(client_socket)
                print(f"Connected to {dest} on port {port}")
            except Exception as e:
                print(f"Connection to {dest} : {port} failed: {str(e)}")
        except ValueError:
            print("Invalid port.")
    else:
        print("Usage: connect <destination> <port>")


def get_ip():
    try:
        # This will give the local IP address of the machine
        return socket.gethostbyname(socket.gethostname())
    except:
        return "127.0.0.1"  # Fallback to localhost


def server_thread():
    host = '0.0.0.0'
    global server_port
    port = int(server_port)

    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind((host, port))
    server_socket.listen(5)
    print(f"Server is listening on port {port} and IP {host}")

    while True:
        client_socket, client_address = server_socket.accept()
        print(f"Accepted connection from {client_address}")

        client_handler = threading.Thread(target=handle_client, args=(client_socket, client_address))
        client_handler.start()


def client_thread():
    while True:
        command = input("")
        if command.lower() == "exit":
            print("shutting down server...")
            exit()
        process_command(command)


server_port = input("Enter the port to run: ")

server_thread = threading.Thread(target=server_thread)
server_thread.start()

client_thread = threading.Thread(target=client_thread)
client_thread.start()
