import customtkinter as ctk
import psutil
import subprocess
import ctypes
import threading
import time
import gc
import os
import shutil
import sys

# ADMIN
def is_admin():
    try:
        return ctypes.windll.shell32.IsUserAnAdmin()
    except:
        return False

if not is_admin():
    ctypes.windll.shell32.ShellExecuteW(
        None,
        "runas",
        sys.executable,
        " ".join(sys.argv),
        None,
        1
    )
    sys.exit()

# OPTIMIZATION
def optimize_network(gaming=False):
    cmds = [
        "netsh int tcp set global autotuninglevel=normal",
        "netsh int tcp set global rss=enabled",
        "ipconfig /flushdns"
    ]
    if gaming:
        cmds.append("netsh int tcp set global ecncapability=disabled")

    for c in cmds:
        subprocess.run(c, shell=True, stdout=subprocess.DEVNULL)

def clear_ram(aggressive=False):
    gc.collect()
    if aggressive:
        try:
            ctypes.windll.ntdll.NtSetSystemInformation(
                80,
                ctypes.byref(ctypes.c_int(4)),
                ctypes.sizeof(ctypes.c_int)
            )
        except:
            pass

def optimize_storage():
    for path in [os.environ.get("TEMP"), "C:\\Windows\\Temp"]:
        if path and os.path.exists(path):
            shutil.rmtree(path, ignore_errors=True)
            os.makedirs(path, exist_ok=True)

    subprocess.run(
        "fsutil behavior set DisableDeleteNotify 0",
        shell=True,
        stdout=subprocess.DEVNULL
    )

# PROFILES
def normal_profile():
    optimize_network()
    clear_ram()
    optimize_storage()
    log("🟦 Обычный профиль применён")

def gaming_profile():
    optimize_network(gaming=True)
    clear_ram(aggressive=True)
    optimize_storage()
    log("🎮 Игровой профиль применён")

# AUTOMODE
auto_mode = False

def auto_optimizer():
    global auto_mode
    while auto_mode:
        cpu = psutil.cpu_percent(interval=1)
        ram = psutil.virtual_memory().percent

        if cpu > 70:
            gaming_profile()
        elif ram > 75:
            clear_ram(True)
            log("🧠 Авто: очистка RAM")
        elif cpu < 40:
            normal_profile()

        time.sleep(5)

# USER INTERFACE
ctk.set_appearance_mode("dark")
ctk.set_default_color_theme("blue")

app = ctk.CTk()
app.title("Zip Optimizer")
app.geometry("420x420")
app.resizable(False, False)

status = ctk.StringVar(value="Готово")

def log(text):
    status.set(text)

# MONITORING
cpu_var = ctk.StringVar()
ram_var = ctk.StringVar()

def update_stats():
    while True:
        cpu_var.set(f"CPU: {psutil.cpu_percent()} %")
        ram_var.set(f"RAM: {psutil.virtual_memory().percent} %")
        time.sleep(1)

threading.Thread(target=update_stats, daemon=True).start()

# ELEMENTS
ctk.CTkLabel(app, text="Zip Optimizer", font=("Arial", 40)).pack(pady=10)

ctk.CTkLabel(app, textvariable=cpu_var).pack()
ctk.CTkLabel(app, textvariable=ram_var).pack(pady=5)

ctk.CTkButton(
    app,
    text="🟦 Обычный профиль",
    command=lambda: threading.Thread(target=normal_profile).start()
).pack(pady=10)

ctk.CTkButton(
    app,
    text="🎮 Игровой профиль",
    fg_color="#b22222",
    command=lambda: threading.Thread(target=gaming_profile).start()
).pack(pady=10)

def toggle_auto():
    global auto_mode
    auto_mode = not auto_mode
    if auto_mode:
        log("🤖 Авто-оптимизация включена")
        threading.Thread(target=auto_optimizer, daemon=True).start()
    else:
        log("⛔ Авто-оптимизация выключена")

ctk.CTkButton(
    app,
    text="🤖 Авто-оптимизация",
    fg_color="#444",
    command=toggle_auto
).pack(pady=15)

ctk.CTkLabel(app, textvariable=status, text_color="green").pack(pady=20)

app.mainloop()
