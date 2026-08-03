FROM nginx:1.30-alpine

LABEL org.opencontainers.image.title="developer-workstation-web" \
      org.opencontainers.image.description="Static NGINX site for the developer workstation mission"

COPY nginx/default.conf /etc/nginx/conf.d/default.conf
COPY site/ /usr/share/nginx/html/

EXPOSE 80

HEALTHCHECK --interval=10s --timeout=3s --start-period=5s --retries=3 \
  CMD wget -q -O /dev/null http://127.0.0.1/health || exit 1
