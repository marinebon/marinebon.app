FROM nginx:stable

RUN apt-get update && apt-get --assume-yes install git

# github repo
RUN mkdir -p /share/github
WORKDIR /share/github
# RUN if [ ! -d "mpatlas4r.org" ] ; then \
#   git clone https://github.com/marinebon/www_marinebon.git ; \
#   rm -rf /usr/share/nginx/html ; \
#   ln -s /share/github/mpatlas4r.org /usr/share/nginx/html ; fi
