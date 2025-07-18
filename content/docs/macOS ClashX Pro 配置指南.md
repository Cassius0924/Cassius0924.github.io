# macOS ClashX Pro 配置指南

```
mixed-port: 7890

# Linux 和 macOS 的 redir 代理端口
redir-port: 7892

# 允许局域网的连接
allow-lan: true

# 规则模式：Rule（规则） / Global（全局代理）/ Direct（全局直连）
mode: rule

# 设置日志输出级别 (默认级别：silent，即不输出任何内容，以避免因日志内容过大而导致程序内存溢出）。
# 5 个级别：silent / info / warning / error / debug。级别越高日志输出量越大，越倾向于调试，若需要请自行开启。
log-level: info
# Clash 的 RESTful API
external-controller: '0.0.0.0:9090'

# RESTful API 的口令
secret: ''

dns:
  enable: true
  ipv6: true
  listen: '0.0.0.0:53'
  use-hosts: true
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  nameserver:
    - 202.120.224.6
    - 114.114.114.114
    - 223.5.5.5
    - 'tcp://223.5.5.5'
  fallback:
    - 'tls://223.5.5.5:853'
    - 'https://223.5.5.5/dns-query'
  fallback-filter:
    geoip: true
    ipcidr:
      - 240.0.0.0/4
# proxy provider start here
proxy-providers:
  feiniao:
    type: http
    path: ./profiles/feiniao.yaml
    url: https://apiv1.v27qae.com/flydsubal/c8lr21z6wpiebqqx?clash=1&extend=1
    interval: 36000
    health-check:
      enable: true
      url: http://www.gstatic.com/generate_204
      interval: 3600
  yiyuan:
    type: http
    path: ./profiles/yiyuan.yaml
    url: https://sub1.smallstrawberry.com/api/v1/client/subscribe?token=d6e73f953b6053a3b263b73f9509375d
    decode-url: true
    interval: 36000
    health-check:
      enable: true
      url: http://www.gstatic.com/generate_204
      interval: 3600


# proxy provider end
proxies:
    # socks5
  - name: windowsServer
    type: socks5
    server: 10.127.78.177
    port: 7890
    # username: username
    # password: password
    # tls: true
    # skip-cert-verify: true
    # udp: true

  # - {name: 🇮🇪 中国-爱尔兰 IPLC C04, server: ir04.cathayroute.com, port: 33400, type: ss, cipher: rc4-md5, password: SRCFiB}
  # - {name: 🇭🇰 香港 油尖旺御金·国峯 名氣通電訊 C02, server: hkhe02.cathayroute.com, port: 33400, type: ss, cipher: rc4-md5, password: SRCFiB}
  # - {name: 🇮🇪 中国-爱尔兰 IPLC C03, server: ir03.cathayroute.com, port: 33400, type: ss, cipher: rc4-md5, password: SRCFiB}
  # - {name: 🇭🇰 香港 油尖旺御金·国峯 名氣通電訊 C09, server: hkhe09.cathayroute.com, port: 33400, type: ss, cipher: rc4-md5, password: SRCFiB}
  # - {name: 🇨🇳 中国-香港 IEPL Equinix HK8 C 02 1Gbps HBO TVB, server: sg12.cathayroute.com, port: 33400, type: ss, cipher: rc4-md5, password: SRCFiB, udp: true}
  # - {name: 🇨🇳 中国-爱尔兰 IPLC C05, server: ir05.cathayroute.com, port: 33400, type: ss, cipher: rc4-md5, password: SRCFiB, udp: true}
  # - {name: 🇨🇳 中国-香港 IPLC Equinix HK2 C 06 1Gbps HBO TVB, server: sg06.cathayroute.com, port: 33400, type: ss, cipher: rc4-md5, password: SRCFiB, udp: true}

proxy-groups:
  - name: Proxies
    type: select
    proxies:
      - 机场节点
      - 自动选择
      - 故障转移

  # - name: 手动节点
  #   type: select
  #   proxies:
  #     - 🇮🇪 中国-爱尔兰 IPLC C04
  #     - 🇭🇰 香港 油尖旺御金·国峯 名氣通電訊 C02
  #     - 🇮🇪 中国-爱尔兰 IPLC C03
  #     - 🇭🇰 香港 油尖旺御金·国峯 名氣通電訊 C09   
  #     - 🇨🇳 中国-香港 IEPL Equinix HK8 C 02 1Gbps HBO TVB
  #     - 🇨🇳 中国-爱尔兰 IPLC C05
  #     - 🇨🇳 中国-香港 IPLC Equinix HK2 C 06 1Gbps HBO TVB

  - name: feiniao
    type: select
    use:
     - feiniao

  - name: yiyuan
    type: select
    use:
     - yiyuan

  - name: 机场节点
    type: select
    proxies:
     - feiniao
     - yiyuan


  - name: 故障转移
    type: fallback
    url: 'http://www.gstatic.com/generate_204'
    interval: 7200
    proxies:
     - feiniao
     - yiyuan

  - name: 自动选择
    type: url-test
    url: 'http://www.gstatic.com/generate_204'
    interval: 300
    use:
      - feiniao
      - yiyuan

rule-providers:
  reject:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/reject.txt"
    path: ./ruleset/reject.yaml
    interval: 86400

  icloud:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/icloud.txt"
    path: ./ruleset/icloud.yaml
    interval: 86400

  apple:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/apple.txt"
    path: ./ruleset/apple.yaml
    interval: 86400

  google:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/google.txt"
    path: ./ruleset/google.yaml
    interval: 86400

  proxy:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/proxy.txt"
    path: ./ruleset/proxy.yaml
    interval: 86400

  direct:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/direct.txt"
    path: ./ruleset/direct.yaml
    interval: 86400

  private:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/private.txt"
    path: ./ruleset/private.yaml
    interval: 86400

  gfw:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/gfw.txt"
    path: ./ruleset/gfw.yaml
    interval: 86400

  tld-not-cn:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/tld-not-cn.txt"
    path: ./ruleset/tld-not-cn.yaml
    interval: 86400

  telegramcidr:
    type: http
    behavior: ipcidr
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/telegramcidr.txt"
    path: ./ruleset/telegramcidr.yaml
    interval: 86400

  cncidr:
    type: http
    behavior: ipcidr
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/cncidr.txt"
    path: ./ruleset/cncidr.yaml
    interval: 86400

  lancidr:
    type: http
    behavior: ipcidr
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/lancidr.txt"
    path: ./ruleset/lancidr.yaml
    interval: 86400

  applications:
    type: http
    behavior: classical
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/applications.txt"
    path: ./ruleset/applications.yaml
    interval: 86400

rules:
  - DOMAIN-SUFFIX,visualstudio.com,DIRECT
  - DOMAIN-SUFFIX,azure.com,DIRECT
  - DOMAIN-SUFFIX,vscode.dev,Proxies
  - RULE-SET,applications,DIRECT
  - DOMAIN,clash.razord.top,DIRECT
  - DOMAIN,yacd.haishan.me,DIRECT
  - RULE-SET,private,DIRECT
  - RULE-SET,reject,REJECT
  - RULE-SET,tld-not-cn,Proxies
  - RULE-SET,gfw,Proxies
  - RULE-SET,telegramcidr,Proxies
  - MATCH,DIRECT
```

