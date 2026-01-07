# FreeSWITCH Standalone Configuration

This repository contains the FreeSWITCH configuration for a production VoIP server setup with Telnyx SIP trunk integration.

## 🚀 Features

- ✅ FreeSWITCH 1.8.5 configuration
- ✅ Telnyx SIP trunk integration
- ✅ Outbound calling via Telnyx
- ✅ Inbound call routing
- ✅ SIP client support (MicroSIP, etc.)
- ✅ Firewall configured for SIP/RTP
- ✅ NAT traversal configured

## 📋 Server Details

- **Server IP**: 157.173.117.207
- **SIP Ports**: 5060 (internal), 5080 (external)
- **RTP Ports**: 16384-32768
- **Provider**: Telnyx
- **DID**: +1 (949) 543-1333

## 🏗️ Architecture

```
[SIP Clients]  ←→  [FreeSWITCH]  ←→  [Telnyx SIP Trunk]
   (Port 5060)      (Server)         (sip.telnyx.com)
```

## 📁 Directory Structure

```
conf/
├── autoload_configs/   # Module configurations
├── dialplan/           # Call routing logic
│   ├── default/        # Internal dialplan
│   └── public/         # External/inbound dialplan
├── directory/          # User/extension definitions
│   └── default/        # Default domain users
├── sip_profiles/       # SIP profile configurations
│   ├── internal.xml    # Internal SIP profile (port 5060)
│   ├── external.xml    # External SIP profile (port 5080)
│   └── external/       # External gateways
│       └── telnyx.xml  # Telnyx gateway configuration
├── vars.xml            # Global variables
└── freeswitch.xml      # Main configuration
```

## 🔧 Key Configuration Files

### Telnyx Gateway
**File**: `conf/sip_profiles/external/telnyx.xml`
- SIP trunk configuration for Telnyx
- Handles outbound call routing

### Outbound Dialplan
**File**: `conf/dialplan/default/02_outbound_telnyx.xml`
- Routes outbound calls from extensions to Telnyx
- Supports US/Canada and international dialing

### Inbound Dialplan
**File**: `conf/dialplan/public/00_inbound_telnyx.xml`
- Routes incoming calls from Telnyx to extensions
- Handles DID routing

### User Extensions
**Directory**: `conf/directory/default/`
- Extension 1000-1019 configured
- Default password: 1234 (⚠️ CHANGE IN PRODUCTION!)

## 🚀 Quick Start

### 1. Clone the Repository
```bash
git clone https://github.com/u-sufi/freeswitch-standalone.git
cd freeswitch-standalone
```

### 2. Install FreeSWITCH
```bash
# Install FreeSWITCH (Debian/Ubuntu)
apt-get update
apt-get install -y freeswitch-meta-all
```

### 3. Deploy Configuration
```bash
# Backup existing config
cp -r /usr/local/freeswitch/conf /usr/local/freeswitch/conf.backup

# Deploy new config
cp -r conf/* /usr/local/freeswitch/conf/

# Restart FreeSWITCH
systemctl restart freeswitch
```

### 4. Configure Telnyx
Edit `conf/sip_profiles/external/telnyx.xml`:
```xml
<param name="username" value="YOUR_TELNYX_USERNAME"/>
<param name="password" value="YOUR_TELNYX_PASSWORD"/>
<param name="extension" value="YOUR_TELNYX_DID"/>
```

### 5. Update Firewall
```bash
# Allow SIP and RTP
ufw allow 5060/udp
ufw allow 5080/udp
ufw allow 16384:32768/udp
```

## 🔐 Security Notes

⚠️ **IMPORTANT**: Before deploying to production:

1. **Change default passwords** in `conf/directory/default/*.xml`
2. **Update Telnyx credentials** in `conf/sip_profiles/external/telnyx.xml`
3. **Configure firewall** to restrict access
4. **Enable fail2ban** for brute force protection
5. **Use strong SIP passwords** (12+ characters)

## 📞 Dial Patterns

### From SIP Clients (e.g., MicroSIP)

| To Call | Dial Pattern | Example |
|---------|--------------|---------|
| US/Canada (10-digit) | NXXNXXXXXX | 2025551234 |
| US/Canada (11-digit) | 1NXXNXXXXXX | 12025551234 |
| International | 011 + country code + number | 01144XXXXXXXXXX |
| Extensions | 1000-1019 | 1000 |
| Voicemail | *98 | *98 |
| Echo Test | 9196 | 9196 |

## 🧪 Testing

### Test Extension Registration
```bash
/usr/local/freeswitch/bin/fs_cli -x "sofia status profile internal reg"
```

### Test Telnyx Gateway
```bash
/usr/local/freeswitch/bin/fs_cli -x "sofia status gateway telnyx"
```

### Test Outbound Call
```bash
/usr/local/freeswitch/bin/fs_cli -x "originate sofia/gateway/telnyx/+12025551234 &echo"
```

### Monitor Logs
```bash
tail -f /usr/local/freeswitch/log/freeswitch.log
```

## 🛠️ Troubleshooting

### Gateway Not Registered
```bash
# Check gateway status
fs_cli -x "sofia status gateway telnyx"

# Restart gateway
fs_cli -x "sofia profile external killgw telnyx"
fs_cli -x "sofia profile external rescan"
```

### Calls Not Connecting
```bash
# Enable SIP trace
fs_cli -x "sofia global siptrace on"

# Check logs
tail -f /usr/local/freeswitch/log/freeswitch.log
```

### No Audio (One-Way Audio)
- Check firewall allows RTP ports (16384-32768)
- Verify NAT settings in `conf/vars.xml`
- Check `ext-rtp-ip` and `ext-sip-ip` settings

## 📚 Documentation

- [FreeSWITCH Official Docs](https://freeswitch.org/confluence/)
- [Telnyx Documentation](https://developers.telnyx.com/)
- [SIP Protocol RFC 3261](https://tools.ietf.org/html/rfc3261)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This configuration is provided as-is for educational and production use.

## ⚠️ Disclaimer

This is a working production configuration. Ensure you:
- Change all default passwords
- Secure your Telnyx credentials
- Configure proper firewall rules
- Monitor for toll fraud
- Comply with telecommunications regulations

## 📧 Support

For issues or questions:
- Open an issue on GitHub
- Check FreeSWITCH community forums
- Contact Telnyx support for provider-specific issues

---

**Built with ❤️ for reliable VoIP communications**


