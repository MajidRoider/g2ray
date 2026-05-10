FROM debian:bookworm-slim

ENV XRAY_VERSION=v26.3.27
ENV MIN_PORT=3000
ENV MAX_PORT=50000

COPY setup.sh /tmp/setup.sh
COPY config.json /etc/xray/g2ray.json

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
       bash curl wget unzip ca-certificates openssl netcat-openbsd \
    && rm -rf /var/lib/apt/lists/* \
    && chmod +x /tmp/setup.sh \
    && /tmp/setup.sh \
    && rm /tmp/setup.sh

# ایجاد entrypoint.sh در زمان build
RUN echo '#!/bin/bash' > /entrypoint.sh && \
    echo '' >> /entrypoint.sh && \
    echo '# تابع پیدا کردن پورت آزاد' >> /entrypoint.sh && \
    echo 'find_free_port() {' >> /entrypoint.sh && \
    echo '  local MIN_PORT=${MIN_PORT:-3000}' >> /entrypoint.sh && \
    echo '  local MAX_PORT=${MAX_PORT:-9000}' >> /entrypoint.sh && \
    echo '  local MAX_ATTEMPTS=50' >> /entrypoint.sh && \
    echo '  local ATTEMPT=0' >> /entrypoint.sh && \
    echo '' >> /entrypoint.sh && \
    echo '  for ATTEMPT in $(seq 1 $MAX_ATTEMPTS); do' >> /entrypoint.sh && \
    echo '    local port=$((RANDOM % (MAX_PORT - MIN_PORT + 1) + MIN_PORT))' >> /entrypoint.sh && \
    echo '    # بررسی با netcat' >> /entrypoint.sh && \
    echo '    if ! nc -z localhost $port 2>/dev/null; then' >> /entrypoint.sh && \
    echo '      echo $port' >> /entrypoint.sh && \
    echo '      return 0' >> /entrypoint.sh && \
    echo '    fi' >> /entrypoint.sh && \
    echo '  done' >> /entrypoint.sh && \
    echo '' >> /entrypoint.sh && \
    echo '  # اگر هیچ پورت آزادی پیدا نشد، از روش fallback استفاده کن' >> /entrypoint.sh && \
    echo '  for port in $(seq $MIN_PORT $MAX_PORT); do' >> /entrypoint.sh && \
    echo '    if ! nc -z localhost $port 2>/dev/null; then' >> /entrypoint.sh && \
    echo '      echo $port' >> /entrypoint.sh && \
    echo '      return 0' >> /entrypoint.sh && \
    echo '    fi' >> /entrypoint.sh && \
    echo '  done' >> /entrypoint.sh && \
    echo '  echo 0' >> /entrypoint.sh && \
    echo '  return 1' >> /entrypoint.sh && \
    echo '}' >> /entrypoint.sh && \
    echo '' >> /entrypoint.sh && \
    echo '# پیدا کردن پورت آزاد' >> /entrypoint.sh && \
    echo 'echo "🔍 Searching for free port in range ${MIN_PORT}-${MAX_PORT}..."' >> /entrypoint.sh && \
    echo 'FREE_PORT=$(find_free_port)' >> /entrypoint.sh && \
    echo '' >> /entrypoint.sh && \
    echo 'if [ "$FREE_PORT" -eq 0 ]; then' >> /entrypoint.sh && \
    echo '  echo "❌ ERROR: No free port found in range ${MIN_PORT}-${MAX_PORT}"' >> /entrypoint.sh && \
    echo '  exit 1' >> /entrypoint.sh && \
    echo 'fi' >> /entrypoint.sh && \
    echo '' >> /entrypoint.sh && \
    echo 'echo "✅ Found free port: $FREE_PORT"' >> /entrypoint.sh && \
    echo '' >> /entrypoint.sh && \
    echo '# به روز رسانی config.json با پورت جدید' >> /entrypoint.sh && \
    echo 'if [ -f /etc/xray/g2ray.json ]; then' >> /entrypoint.sh && \
    echo '  sed -i "s/\"port\": [0-9]*/\"port\": $FREE_PORT/g" /etc/xray/g2ray.json' >> /entrypoint.sh && \
    echo '  echo "📝 Updated config.json with port $FREE_PORT"' >> /entrypoint.sh && \
    echo 'else' >> /entrypoint.sh && \
    echo '  echo "⚠️  Warning: config.json not found"' >> /entrypoint.sh && \
    echo 'fi' >> /entrypoint.sh && \
    echo '' >> /entrypoint.sh && \
    echo '# نمایش اطلاعات اتصال' >> /entrypoint.sh && \
    echo 'echo ""' >> /entrypoint.sh && \
    echo 'echo -e "\\033[1;32m[g2ray]\\033[0m Connection string:"' >> /entrypoint.sh && \
    echo 'echo "vless://2166797c-f912-4a24-a467-c5d0c216cc2d@94.130.50.12:${FREE_PORT}?encryption=none&security=tls&type=xhttp&mode=packet-up&sni=${CODESPACE_NAME}-${FREE_PORT}.app.github.dev&path=%2F#g2ray"' >> /entrypoint.sh && \
    echo 'echo ""' >> /entrypoint.sh && \
    echo 'echo "📊 Port Information:"' >> /entrypoint.sh && \
    echo 'echo "   - Internal Port: $FREE_PORT"' >> /entrypoint.sh && \
    echo 'echo "   - External URL: https://${CODESPACE_NAME}-${FREE_PORT}.app.github.dev"' >> /entrypoint.sh && \
    echo 'echo ""' >> /entrypoint.sh && \
    echo '' >> /entrypoint.sh && \
    echo '# اجرای Xray' >> /entrypoint.sh && \
    echo 'exec /usr/local/bin/xray run -c /etc/xray/g2ray.json' >> /entrypoint.sh

RUN chmod +x /entrypoint.sh

EXPOSE 3000-50000

CMD ["/entrypoint.sh"]
