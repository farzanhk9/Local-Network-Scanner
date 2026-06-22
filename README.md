import socket
import threading
from queue import Queue

active_hosts = []
queue = Queue()


def scan_ip(ip):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.settimeout(0.3)

    try:
        result = sock.connect_ex((ip, 80))

        if result == 0:
            active_hosts.append(ip)

    except:
        pass

    finally:
        sock.close()


def worker():
    while not queue.empty():
        ip = queue.get()
        scan_ip(ip)
        queue.task_done()


def main():

    network = input(
        "Network Prefix (e.g. 192.168.1): "
    ).strip()

    print("\nScanning...\n")

    for i in range(1, 255):
        queue.put(f"{network}.{i}")

    threads = []

    for _ in range(100):

        t = threading.Thread(target=worker)

        t.start()

        threads.append(t)

    queue.join()

    print("\nActive Hosts:")
    print("--" * 30)

    if active_hosts:

        for host in sorted(active_hosts):
            print(host)

    else:
        print("No active hosts found.")


if __name__ == "__main__":
    main()
