# ===== Frontend Dockerfile =====
FROM nginx:alpine

# Hapus config default nginx
RUN rm /etc/nginx/conf.d/default.conf

# Copy konfigurasi nginx custom
COPY nginx.conf /etc/nginx/conf.d/

# Copy file frontend (PERBAIKI INI - tambahkan style.css)
COPY index.html style.css /usr/share/nginx/html/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]