# WireGuard Setup Guide for Malaysia Home Network

## Overview
This guide will help you set up a WireGuard VPN server at home in Malaysia so your sister in China can connect through your home internet and bypass the Great Firewall for free.

**Time needed:** ~30 minutes  
**Internet:** Works with Maxis (KL) or Unifi (Melaka)

---

## What You'll Need
- A computer that stays on 24/7 (Windows, Mac, or Raspberry Pi)
- Access to router settings
- Router admin password

---

## Part 1: Router Setup (Port Forwarding)

### For Maxis Router (KL):
1. Open browser, go to router IP (usually `192.168.1.1` or check sticker on router)
2. Login (default often admin/admin or check router sticker)
3. Find **Port Forwarding** section
4. Add new rule:
   - **Service Name:** WireGuard
   - **External Port:** 51820
   - **Internal Port:** 51820
   - **Internal IP:** [The computer's local IP running WireGuard]
   - **Protocol:** UDP
5. Save and enable the rule

### For Unifi Router (Melaka):
1. Go to `192.168.1.1` in browser
2. Login with credentials
3. Navigate to **Firewall** > **Port Forwarding** or **Virtual Server**
4. Add rule:
   - **Name:** WireGuard
   - **Port:** 51820
   - **Protocol:** UDP
   - **Forward to IP:** [Server computer's IP]
5. Apply changes

### Find Your Public IP Address
- Google "what is my IP" from home network
- **Write it down** - you'll need this later!

---

## Part 2: Install WireGuard on Server Computer

### On Windows:
1. Download from: https://www.wireguard.com/install/
2. Install the WireGuard application
3. Open WireGuard app

### On Mac:
Open Terminal and run:
```bash
brew install wireguard-tools
brew install --cask wireguard
```

### On Linux/Raspberry Pi:
Open Terminal and run:
```bash
sudo apt update
sudo apt install wireguard
```

---

## Part 3: Generate Encryption Keys

Open Terminal (Mac/Linux) or Command Prompt (Windows) and run these commands:

```bash
# Generate server keys
wg genkey | tee server_private.key | wg pubkey > server_public.key

# Generate client keys (for your sister)
wg genkey | tee client_private.key | wg pubkey > client_public.key
```

This creates 4 files with encryption keys. **Keep these safe!**

To view the keys, run:
```bash
cat server_private.key
cat server_public.key
cat client_private.key
cat client_public.key
```

---

## Part 4: Create Server Configuration

Create a file called `wg0.conf` with this content:

```ini
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = [PASTE SERVER PRIVATE KEY HERE]

# IP forwarding
PostUp = sysctl -w net.ipv4.ip_forward=1
PostDown = sysctl -w net.ipv4.ip_forward=0

[Peer]
# Your sister's laptop in China
PublicKey = [PASTE CLIENT PUBLIC KEY HERE]
AllowedIPs = 10.0.0.2/32
```

**Replace the following:**
- `[PASTE SERVER PRIVATE KEY HERE]` → Content from `server_private.key`
- `[PASTE CLIENT PUBLIC KEY HERE]` → Content from `client_public.key`

---

## Part 5: Start WireGuard Server

### Windows:
1. Open WireGuard app
2. Click "Add Tunnel" > "Add empty tunnel"
3. Paste the server config from Part 4
4. Click "Activate"

### Mac:
```bash
sudo wg-quick up wg0
# To make it start automatically on boot:
sudo systemctl enable wg-quick@wg0
```

### Linux/Raspberry Pi:
```bash
sudo wg-quick up wg0
# To make it start automatically on boot:
sudo systemctl enable wg-quick@wg0
```

---

## Part 6: Create Client Config for Your Sister

Create a file called `client.conf`:

```ini
[Interface]
PrivateKey = [PASTE CLIENT PRIVATE KEY HERE]
Address = 10.0.0.2/24
DNS = 8.8.8.8

[Peer]
PublicKey = [PASTE SERVER PUBLIC KEY HERE]
Endpoint = [YOUR_HOME_PUBLIC_IP]:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

**Replace the following:**
- `[PASTE CLIENT PRIVATE KEY HERE]` → Content from `client_private.key`
- `[PASTE SERVER PUBLIC KEY HERE]` → Content from `server_public.key`
- `[YOUR_HOME_PUBLIC_IP]` → The public IP you wrote down in Part 1

---

## Part 7: Send Config to Your Sister

Send the `client.conf` file to your sister via:
- WhatsApp
- Telegram
- Email
- Any secure messaging app

**Do NOT share the server keys - only send the client.conf file!**

---

## Verify It's Working

To check if WireGuard is running properly, run:

```bash
sudo wg show
```

You should see output showing the interface is active.

---

## Troubleshooting

### Connection doesn't work?
1. ✅ Check firewall on server computer isn't blocking port 51820
2. ✅ Verify port forwarding is correct and enabled in router
3. ✅ Confirm public IP hasn't changed (Google "what is my IP" again)
4. ✅ Make sure WireGuard service is running (`sudo wg show`)

### Computer Firewall Settings:

**Windows:**
- Windows Defender Firewall > Allow an app > Find WireGuard > Allow

**Mac:**
- System Settings > Network > Firewall > Allow WireGuard

**Linux:**
```bash
sudo ufw allow 51820/udp
```

### If Public IP Changes:
Consider using a Dynamic DNS service like:
- No-IP (free)
- DuckDNS (free)
- Your router may have built-in DDNS support

---

## Your Sister's Setup (In China)

Once she receives the `client.conf` file:

1. Install WireGuard on Mac:
```bash
brew install wireguard-tools
brew install --cask wireguard
```

2. Open WireGuard app
3. Click "Import tunnel(s) from file"
4. Select the `client.conf` file you sent
5. Click "Activate"

**That's it!** All her traffic now routes through Malaysia home internet - no data usage on Simba SG! 🎉

---

## Security Notes

- Keep the key files private
- Only share `client.conf` with your sister
- Never share `server_private.key` with anyone
- The connection is encrypted end-to-end

---

## Benefits

✅ **Free** - No monthly VPN costs  
✅ **Fast** - Direct connection to home  
✅ **Secure** - Military-grade encryption  
✅ **Unlimited** - Uses Malaysia home internet data  
✅ **Bypasses Great Firewall** - Access Google, ChatGPT, etc.  

---

## Questions?

If you run into issues during setup, take a screenshot and send it for help!

**Important commands to remember:**

```bash
# Check if WireGuard is running
sudo wg show

# Start WireGuard
sudo wg-quick up wg0

# Stop WireGuard
sudo wg-quick down wg0

# View your public IP
curl ifconfig.me
```

Good luck with the setup! 🚀
