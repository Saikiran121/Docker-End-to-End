FROM nginx:alpine

LABEL maintainer="saikiranbiradar76642@gmail.com"
LABEL version="1.0"
LABEL description="Custom Nginx image with additional configurations"

WORKDIR /usr/share/nginx/html

COPY files/index.html .
COPY files/config/* /etc/nginx/conf.d/

RUN apk add --no-cache curl

ENV APP_ENV=production
ENV PORT=80

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

