# DzinLabs Myanmar VPN — Server Setup Guide 🇲🇲

App က **client** သက်သက်ပါ — server မရှိရင် ချိတ်စရာမရှိဘူး။
မြန်မာစစ်တပ်ရဲ့ firewall ကို ဖောက်နိုင်တာ **VLESS + REALITY** ပါ (ရိုးရိုး
WireGuard/OpenVPN က DPI မှာ အဖမ်းခံရတယ်)။

## 1. VPS ငှားပါ (Singapore — မြန်မာနဲ့ အနီးဆုံး)

- ~$5/လ (RackNerd, HostHatch, DigitalOcean, Vultr...)
- Ubuntu 22.04/24.04
- ⚠️ card လိုတယ် — ကိုယ့်မှာ မရှိရင် နိုင်ငံခြားက အသိ/ဆွေမျိုးကို အကူအညီတောင်း

## 2. Xray သွင်းပါ (VPS ထဲမှာ, root)

```bash
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```

Key တွေထုတ်:

```bash
xray x25519   # privateKey + publicKey (publicKey ကို သိမ်း)
xray uuid     # UUID တစ်ခု
openssl rand -hex 4  # shortId (ဥပမာ a1b2c3d4)
```

## 3. Config ရေး — `/usr/local/etc/xray/config.json`

`UUID`, `PRIVATE_KEY`, `SHORT_ID` နေရာမှာ ကိုယ့်ဟာထည့်:

```json
{
  "log": { "loglevel": "warning" },
  "inbounds": [{
    "port": 443,
    "protocol": "vless",
    "settings": {
      "clients": [{ "id": "UUID", "flow": "xtls-rprx-vision" }],
      "decryption": "none"
    },
    "streamSettings": {
      "network": "tcp",
      "security": "reality",
      "realitySettings": {
        "show": false,
        "dest": "www.microsoft.com:443",
        "xver": 0,
        "serverNames": ["www.microsoft.com"],
        "privateKey": "PRIVATE_KEY",
        "shortIds": ["SHORT_ID"]
      }
    },
    "sniffing": { "enabled": true, "destOverride": ["http", "tls", "quic"] }
  }],
  "outbounds": [
    { "protocol": "freedom", "tag": "direct" },
    { "protocol": "blackhole", "tag": "block" }
  ]
}
```

```bash
systemctl restart xray && systemctl enable xray
# firewall: 443/tcp ဖွင့်
ufw allow 443/tcp 2>/dev/null; iptables -I INPUT -p tcp --dport 443 -j ACCEPT 2>/dev/null
```

## 4. Client link ထုတ် (အိမ်ကဖုန်းထဲ ထည့်ဖို့)

```
vless://UUID@SERVER_IP:443?encryption=none&flow=xtls-rprx-vision&security=reality&sni=www.microsoft.com&fp=chrome&pbk=PUBLIC_KEY&sid=SHORT_ID&type=tcp#DzinLabs-MM
```

- `SERVER_IP` = VPS ရဲ့ IP, `PUBLIC_KEY` = x25519 ရဲ့ publicKey
- ဒီ link ကို QR ပြောင်းပြီး (qrencode / online QR) အိမ်ကို ပို့
- App ထဲမှာ **+ → Import from QR code** → scan → connect 🟢

## 5. ဖုန်းပြောရင် လိုင်းရှင်းဖို့ tips

- Server ကို Singapore ပဲထား (ping အနည်းဆုံး)
- Messenger call မကောင်းရင် App settings → routing → voice call domain တွေ direct ထားလို့မရ — VPN ထဲက ဖြတ်ရမယ် (Messenger က ပိတ်ထားလို့)
- Server တစ်လုံးကို လူ 3-5 ယောက်ထက် မများစေနဲ့

## 6. Server မရသေးခင် အခုစမ်းလို့ရတာ

DzinLabs Gaming VPN app ထဲက **WARP** — Cloudflare IP တွေကို အပြီးပိတ်ဖို့
ခက်လို့ တခါတလေ အလုပ်ဖြစ်တယ်။ အိမ်ကဖုန်းမှာ WARP ချိတ်ပြီး Messenger
စမ်းကြည့်။ မရမှ အပေါ်က VLESS လမ်းသွား။
