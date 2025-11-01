FROM nginx:alpine

# تثبيت openssl لإنشاء الشهادة
RUN apk add --no-cache openssl

# نسخ ملفات الإعداد وسكريبت الشهادة
COPY nginx.conf /etc/nginx/nginx.conf
COPY generate-ssl.sh /usr/local/bin/generate-ssl.sh

# تنفيذ السكريبت ونقل الشهادات للمسار الصحيح
RUN chmod +x /usr/local/bin/generate-ssl.sh && \
    /usr/local/bin/generate-ssl.sh && \
    mkdir -p /etc/nginx/ssl && \
    cp ./ssl/selfsigned.crt /etc/nginx/ssl/selfsigned.crt && \
    cp ./ssl/selfsigned.key /etc/nginx/ssl/selfsigned.key && \
    sed -i '/daemon/d' /etc/nginx/nginx.conf

# تعريض المنافذ
EXPOSE 80 443

# تشغيل Nginx في الـ foreground
CMD ["nginx", "-g", "daemon off;"]
