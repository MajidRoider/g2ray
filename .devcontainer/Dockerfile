FROM debian:bookworm-slim

ENV XRAY_VERSION=v26.3.27
ENV MIN_PORT=3000
ENV MAX_PORT=9000

COPY setup.sh /tmp/setup.sh
COPY config.json /etc/xray/g2ray.json

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
       bash curl wget unzip ca-certificates openssl netcat-openbsd \
    && rm -rf /var/lib/apt/lists/* \
    && chmod +x /tmp/setup.sh \
    && /tmp/setup.sh \
    && rm /tmp/setup.sh

# اسکریپت پیدا کردن پورت آزاد و نمایش آن
RUN echo '#!/bin/bash\n\
find_free_port() {\n\
  local MIN_PORT=${MIN_PORT:-3000}\n\
  local MAX_PORT=${MAX_PORT:-9000}\n\
  local port=$((RANDOM % (MAX_PORT - MIN_PORT + 1) + MIN_PORT))\n\
  while nc -z localhost $port 2>/dev/null; do\n\
    port=$((RANDOM % (MAX_PORT - MIN_PORT + 1) + MIN_PORT))\n\
  done\n\
  echo $port\n\
}\n\
\n\
# پیدا کردن پورت آزاد\n\
FREE_PORT=$(find_free_port)\n\
echo "✅ Found free port: $FREE_PORT"\n\
\n\
# ذخیره پورت در فایل برای استفاده بعدی\n\
echo $FREE_PORT > /tmp/free_port.txt\n\
\n\
# به روز رسانی config.json با پورت جدید\n\
sed -i "s/\"port\": [0-9]*/\"port\": $FREE_PORT/g" /etc/xray/g2ray.json\n\
\n\
# نمایش اطلاعات اتصال با پورت صحیح\n\
echo ""\n\
echo -e "\\033[1;32m[g2ray]\\033[0m Connection string:"\n\
echo "vless://2166797c-f912-4a24-a467-c5d0c216cc2d@94.130.50.12:${FREE_PORT}?encryption=none&security=tls&type=xhttp&mode=packet-up&sni=${CODESPACE_NAME}-${FREE_PORT}.app.github.dev&path=%2F#g2ray"\n\
echo ""\n\
\n\
# اجرای Xray\n\
/usr/local/bin/xray run -c /etc/xray/g2ray.json\n\
' > /entrypoint.sh && chmod +x /entrypoint.sh

EXPOSE 3000-9000

CMD ["/entrypoint.sh"]
