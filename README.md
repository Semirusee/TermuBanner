TermuBanner

**Customize your Termux experience with a stylish Android-themed banner, system info display, and language options. Simple, lightweight, and ready to enhance your terminal!**

📍 Repo: https://github.com/Semirusee/TermuBanner

---

## 🚀 Features

- Android-inspired ASCII art
- Displays system info (OS, Kernel, Architecture, Uptime, Installed packages)
- Supports Italian and English
- Automatic integration into your Termux shell
- Works with both `bash` and `zsh`

---

## 📁 Repo Contents

TermuBanner/ ├── install.sh          # Adds the banner to your shell startup ├── banner.sh           # Displays the banner and system info ├── setup_termux_env.sh # Generates the project files └── res/ └── android.txt     # ASCII art logo

---

## 📦 Installation

### 1) Clone the Repository

```bash
git clone https://github.com/Semirusee/TermuBanner.git
cd TermuBanner

2) Run Setup Script

This script will generate your banner and install scripts in the current directory:

bash setup_termux_env.sh

3) Choose Language

Select your language:

1) Italiano
2) English

4) Install the Banner

bash install.sh

This will update your shell startup file (~/.bashrc or ~/.zshrc) so that the banner runs automatically when Termux starts.

5) Restart Termux

Close and reopen Termux — your new banner will appear!


---

🛠 Included Files

setup_termux_env.sh — Generates the environment (language options)

banner.sh — Displays the greeting banner

install.sh — Installs the banner into your shell

res/android.txt — ASCII art of the Android logo



---

⚙ Shell Compatibility

TermuBanner supports both bash and zsh. During installation, the script auto-detects your shell and updates the correct startup file.


---

💡 Tips

To remove the banner, manually edit ~/.bashrc or ~/.zshrc and remove the line that calls banner.sh.

Customize the ASCII art in res/android.txt as you like.



---

Banner Example

_____
       /     \
      | () () |
       \  ^  /
        |||||
        |||||


---

❤️ Credits

Created by Semirusee
GitHub: https://github.com/Semirusee


---

© All rights reserved
