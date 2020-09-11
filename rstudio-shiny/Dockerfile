FROM rocker/geospatial:4.0.2

ENV SHINY_SERVER_VERSION=latest
RUN /rocker_scripts/install_shiny_server.sh
EXPOSE 3838

RUN /rocker_scripts/install_python.sh

# install extra packages
RUN install2.r --error \
  DiagrammeR \
  DT \
  flexdashboard \
  fs \
  glue \
  here \
  httpuv \
  jsonlite \
  kableExtra \
  leaflet \
  mapedit \
  plumber \
  RColorBrewer \
  Rcpp \
  RColorBrewer \
  rhandsontable \
  shinydashboard \
  shinydashboardPlus \
  shinyjs \
  units
  
# github repos
RUN mkdir -p /share/github
WORKDIR /share/github

# some mci repos are private:
#RUN if [ ! -d "mpatlas-api4r" ] ; then \
#  git clone https://github.com/mci/mpatlas-api4r.git ; fi
#RUN if [ ! -d "mpatlas-apps4r" ] ; then \
#  git clone https://github.com/mci/mpatlas-apps4r.git ; fi

# run api
#RUN Rscript /share/github/mpatlas-api4r/run_api.R &

#CMD ["/init"]
