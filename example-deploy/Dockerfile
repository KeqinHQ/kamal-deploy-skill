FROM nginx:alpine
RUN echo '<!doctype html><html><head><meta charset="utf-8"><title>Kamal hello</title></head><body style="font-family:system-ui;padding:2rem"><h1>Hello from Kamal!</h1><p>Deployed via the kamal-deploy skill to job.sdpku.com.</p></body></html>' > /usr/share/nginx/html/index.html && \
    echo 'ok' > /usr/share/nginx/html/up
EXPOSE 80
