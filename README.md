# Deploying the Application on a Linode Server

Here is an explaination of how to connect to a Linode server over SSH, keep your session alive with tmux, and run the application on the server.

---

## Connecting to the Remote Server Over SSH from Windows

1. Open **Command Prompt** or **PowerShell**.

2. In the terminal, enter the following command, replacing `[username]` with the username of the remote user and `[ip-address]` with the IP address or domain name of the remote server:

   ```bash
   ssh [username]@[ip-address]
   ```

3. Accept the prompt by entering `y` or `yes`.

4. Once you have successfully connected, your terminal is using the remote shell environment of the server. Your command prompt should now show the username and hostname configured for the server.

---

## Using tmux (a Terminal Multiplexer)

tmux creates a host server on your Linode and connects to it with a client window. If the client is disconnected, the server keeps running. When you reconnect to your Linode after rebooting your computer or losing your Internet connection, you can reattach to the tmux session. The files you were working with will still be open, and the processes you had running will still be active.

### Creating tmux sessions

1. **Start a new tmux session:**

   ```bash
   tmux
   ```

   Running this command creates a new session with one window and one pane.

2. **Detach from the session:**

   ```bash
   tmux detach
   ```

3. **Attach to the last session:**

   ```bash
   tmux attach
   ```

   The shorter form `tmux a` can also be used.

---

## Setting Up and Running the Application

Run the following steps inside the SSH session (for example, as `root@linode`).

1. **Install SQLite3:**

   ```bash
   apt install sqlite3
   ```

2. **Clone the repository:**

   ```bash
   git clone <repo>
   ```

3. **Create and activate a virtual environment:**

   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```

4. **Install the requirements:**

   ```bash
   pip install -r requirements.txt
   ```

5. **Update `settings.py`:** add the IP address of the server to `ALLOWED_HOSTS`:

   ```python
   ALLOWED_HOSTS = ['<server-ip-address>']
   ```
   - Django checks the `Host` header of every incoming request against `ALLOWED_HOSTS`. This protects against HTTP Host header attacks.
- When `DEBUG = True`, only `localhost` and `127.0.0.1` are allowed by default. When `DEBUG = False`, an empty list rejects every request.
- When you visit `http://<server-ip-address>:8000`, the Host header is the server's IP. If it isn't listed, Django returns a `400 Bad Request` (`DisallowedHost` error).
- If you later use a domain name, add it as well, e.g. `ALLOWED_HOSTS = ['<server-ip-address>', 'example.com']`.

6. **Run the server:**

   ```bash
   python manage.py runserver 0.0.0.0:8000
   ```
- By default, `runserver` binds to `127.0.0.1` (localhost), so the app is only reachable from inside the server itself. Requests from your browser on another machine would be refused.
- `0.0.0.0` tells Django to listen on **all network interfaces** of the server, including the public IP of the Linode. This makes the app reachable from outside.
- `8000` is the port the app listens on. You access it at `http://<server-ip-address>:8000`.
