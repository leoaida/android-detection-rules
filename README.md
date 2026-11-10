# android-detection-rules

Detection rules for the Android malware behaviours I keep running into, split
across the two layers where you'd actually catch them:

- **YARA** (`rules/`) — classify a sample from its unpacked contents
- **Sigma** (`sigma/`) — catch the C2 traffic in proxy / network logs

The idea is that triage (see my [apk-triage](https://github.com/leoaida/apk-triage)
tool) tells you *what* a sample does; these rules are how you then detect it at
scale — on disk and on the wire.

## YARA

| Rule | Catches |
|------|---------|
| `Android_SMS_Interceptor` | OTP stealers / SMS bots (receives + sends/aborts SMS) |
| `Android_Overlay_Banker`  | overlay + accessibility credential-theft bankers |
| `Android_Dynamic_Dropper` | runtime DEX loading / secondary-APK droppers |
| `Android_Hardcoded_IP_Gate` | hard-coded IP-based PHP C2 gateway URLs |

DEX and the manifest are usually compressed inside the `.apk`, so unzip first
and scan the contents:

```bash
unzip -o sample.apk -d sample_out/
yara -r rules/android_malware.yar sample_out/
```

## Sigma

| Rule | Catches |
|------|---------|
| `android_c2_ip_gate.yml`   | requests to `http://<ip>/<name>.php` style gateways |
| `android_dalvik_raw_ip.yml`| Dalvik user-agent beaconing straight to a raw IP |

Convert to your SIEM with [sigma-cli](https://github.com/SigmaHQ/sigma-cli):

```bash
sigma convert -t splunk sigma/android_c2_ip_gate.yml
```

## Notes

- The YARA rules are string-based, so heavy obfuscation or reflection can slip
  past them — a clean result is not a clean sample. They're meant to be fast and
  high-precision for the common families, not exhaustive.
- Tune the Sigma `falsepositives` to your environment before alerting on them;
  ad SDKs in particular love connecting to raw IPs.

## License

MIT
