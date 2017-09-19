FROM nginx:latest

COPY ./sites/backend.conf /etc/nginx/sites-available/backend.conf
COPY ./sites/frontend.conf /etc/nginx/sites-available/frontend.conf

RUN ln -s /etc/nginx/sites-available/frontend.conf /etc/nginx/sites-enabled/frontend.conf
RUN ln -s /etc/nginx/sites-available/backend.conf /etc/nginx/sites-enabled/backend.conf