# Whitelist IP 20.233.211.5 - Complete Setup Guide

## 📋 Summary

Whitelisting backend IP **`20.233.211.5`** for ESL and SIP connectivity to FreeSWITCH.

---

## ✅ Step 1: FreeSWITCH ACL Configuration (COMPLETED)

The FreeSWITCH ACL has been updated to allow IP `20.233.211.5`:

- ✅ **ESL ACL** (`esl_trusted_only`) - Updated
- ✅ **SIP ACL** (`sip_trusted_only`) - Updated

**File Modified**: `/usr/local/freeswitch/conf/autoload_configs/acl.conf.xml`

---

## 🔧 Step 2: UFW Firewall Rules (ACTION REQUIRED)

Run these commands on your FreeSWITCH server to add firewall rules:

```bash
# 1. ESL Port (18443) - Event Socket Layer
sudo ufw allow from 20.233.211.5 to any port 18443 proto tcp comment 'ESL - CommsEngine Backend New IP'

# 2. SIP Internal Port (5062) - TCP
sudo ufw allow from 20.233.211.5 to any port 5062 proto tcp comment 'SIP Internal - CommsEngine Backend New IP'

# 3. SIP Internal Port (5062) - UDP
sudo ufw allow from 20.233.211.5 to any port 5062 proto udp comment 'SIP Internal UDP - CommsEngine Backend New IP'

# 4. WebRTC Port (7443)
sudo ufw allow from 20.233.211.5 to any port 7443 proto tcp comment 'WebRTC - CommsEngine Backend New IP'

# 5. Verify the rules were added
sudo ufw status numbered | grep 20.233.211.5
```

**Expected Output**:
```
[XX] 18443/tcp    ALLOW IN    20.233.211.5    # ESL - CommsEngine Backend New IP
[XX] 5062/tcp     ALLOW IN    20.233.211.5    # SIP Internal - CommsEngine Backend New IP
[XX] 5062/udp     ALLOW IN    20.233.211.5    # SIP Internal UDP - CommsEngine Backend New IP
[XX] 7443/tcp     ALLOW IN    20.233.211.5    # WebRTC - CommsEngine Backend New IP
```

---

## 🔄 Step 3: Reload FreeSWITCH Configuration (ACTION REQUIRED)

After adding the UFW rules, reload FreeSWITCH to apply the ACL changes:

```bash
# Reload XML configuration
/usr/local/freeswitch/bin/fs_cli -H 127.0.0.1 -P 18443 -p "yRqlGy00jCTYRJ6x6HpYwdZZNQ7FvDQ52owVZQAUK50=" -x "reloadxml"

# Reload ACL
/usr/local/freeswitch/bin/fs_cli -H 127.0.0.1 -P 18443 -p "yRqlGy00jCTYRJ6x6HpYwdZZNQ7FvDQ52owVZQAUK50=" -x "reloadacl"
```

**Expected Output**:
```
+OK [Success]
+OK acl reloaded
```

---

## 🧪 Step 4: Test ESL Connection

From your backend server (Kubernetes pod), test the connection:

```bash
# Test ESL port connectivity
kubectl exec -it <pod-name> -n dev -- sh -c "nc -zv 157.173.117.207 18443"

# Or use telnet
kubectl exec -it <pod-name> -n dev -- sh -c "timeout 5 telnet 157.173.117.207 18443"
```

**Expected Result**: Connection should succeed (not timeout or be rejected)

---

## 🔄 Step 5: Restart Backend Service

Restart your CommsEngine backend to reconnect with the new IP:

```bash
# Restart the deployment
kubectl rollout restart deployment ahk-call-service-deployment -n dev

# Watch the logs
kubectl logs -f deployment/ahk-call-service-deployment -n dev | grep -i "esl\|freeswitch\|connect"
```

**Expected Log Output**:
```
✅ Connected to FreeSWITCH ESL 157.173.117.207:18443
```

**NOT**:
```
❌ ESL error: connect ETIMEDOUT 157.173.117.207:18443
❌ IP 20.233.211.5 Rejected by acl "esl_trusted_only"
```

---

## 📊 Port Summary

| Port  | Protocol | Purpose                          | Status |
|-------|----------|----------------------------------|--------|
| 18443 | TCP      | ESL (Event Socket Layer) - API   | ✅ ACL Updated, ⏳ UFW Pending |
| 5062  | TCP/UDP  | SIP Internal (Agent Extensions)  | ✅ ACL Updated, ⏳ UFW Pending |
| 7443  | TCP      | WebRTC (Secure WebSocket)        | ✅ ACL Updated, ⏳ UFW Pending |

---

## 🔍 Troubleshooting

### If you see: "IP 20.233.211.5 Rejected by acl"

Check FreeSWITCH logs:
```bash
tail -50 /usr/local/freeswitch/log/freeswitch.log | grep "20.233.211.5"
```

If you see rejections, verify ACL was reloaded:
```bash
/usr/local/freeswitch/bin/fs_cli -H 127.0.0.1 -P 18443 -p "yRqlGy00jCTYRJ6x6HpYwdZZNQ7FvDQ52owVZQAUK50=" -x "reloadacl"
```

### If you see: "connect ETIMEDOUT"

Check UFW firewall rules:
```bash
sudo ufw status numbered | grep 20.233.211.5
```

If no rules found, run the UFW commands from Step 2 above.

### If connection still fails

Check if FreeSWITCH is listening:
```bash
ss -tulpn | grep 18443
```

Expected output:
```
tcp    LISTEN  0  5  0.0.0.0:18443  0.0.0.0:*  users:(("freeswitch",pid=187318,fd=69))
```

---

## 📝 Quick Command Summary

```bash
# 1. Add UFW rules (run all 4 commands from Step 2)
sudo ufw allow from 20.233.211.5 to any port 18443 proto tcp comment 'ESL - CommsEngine Backend New IP'
sudo ufw allow from 20.233.211.5 to any port 5062 proto tcp comment 'SIP Internal - CommsEngine Backend New IP'
sudo ufw allow from 20.233.211.5 to any port 5062 proto udp comment 'SIP Internal UDP - CommsEngine Backend New IP'
sudo ufw allow from 20.233.211.5 to any port 7443 proto tcp comment 'WebRTC - CommsEngine Backend New IP'

# 2. Reload FreeSWITCH
/usr/local/freeswitch/bin/fs_cli -H 127.0.0.1 -P 18443 -p "yRqlGy00jCTYRJ6x6HpYwdZZNQ7FvDQ52owVZQAUK50=" -x "reloadxml"
/usr/local/freeswitch/bin/fs_cli -H 127.0.0.1 -P 18443 -p "yRqlGy00jCTYRJ6x6HpYwdZZNQ7FvDQ52owVZQAUK50=" -x "reloadacl"

# 3. Verify
sudo ufw status numbered | grep 20.233.211.5
tail -20 /usr/local/freeswitch/log/freeswitch.log | grep "20.233.211.5"
```

---

## 🎯 Next Steps

1. ✅ **FreeSWITCH ACL Updated** - DONE
2. ⏳ **Run UFW commands** - Do this now (Step 2)
3. ⏳ **Reload FreeSWITCH** - Do this after UFW (Step 3)
4. ⏳ **Test connection** - Verify it works (Step 4)
5. ⏳ **Restart backend** - Reconnect with new IP (Step 5)

---

**Date**: 2026-02-03
**New Backend IP**: 20.233.211.5
**Old Backend IPs**: 20.174.157.124, 20.174.217.63
**FreeSWITCH Server**: 157.173.117.207
