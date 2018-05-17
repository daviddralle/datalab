FROM gcr.io/earthengine-project/datalab-ee:latest

#MAINTAINER Your Name <daviddralle@gmail.com>

# Install IPyLeaflet. The notebook library dependency is downgraded to
# version 4.4.1 but the datalab repo warns about potential version issues:
# https://github.com/googledatalab/datalab/blob/master/containers/base/Dockerfile#L139
RUN pip install ipyleaflet \
  && jupyter nbextension enable --py --sys-prefix ipyleaflet \
  && pip install notebook==4.4.1


# Install additional geospatial packages for Python 2
RUN apt-get update \
  && apt-get install libgdal-dev -y --allow-unauthenticated \
  && apt-get install python-gdal -y --allow-unauthenticated \
  && apt-get install python-fiona -y --allow-unauthenticated \ 
  && apt-get install python-shapely -y --allow-unauthenticated \
  && apt-get install python-geopandas -y --allow-unauthenticated \
  && apt-get install python-rasterio -y --allow-unauthenticated \
  && pip install rasterstats


# need to add repository for latest version of R
RUN echo 'deb http://cran.rstudio.com/bin/linux/ubuntu xenial/' >> /etc/apt/sources.list

# Define R version to install
ENV R_BASE_VERSION 3.4.3

## Now install R and littler, and create a link for littler in /usr/local/bin
## Also set a default CRAN repo, and make sure littler knows about it too
RUN apt-get update \
    && apt-get install -y --no-install-recommends --allow-unauthenticated \
        libssh2-1-dev \
        libcurl4-openssl-dev \
        libssl-dev \
        littler \
                r-cran-littler \
        r-base=${R_BASE_VERSION}* \
        r-base-dev=${R_BASE_VERSION}* \
        r-recommended=${R_BASE_VERSION}* \
        && echo 'options(repos = c(CRAN = "https://cran.rstudio.com/"), download.file.method = "libcurl")' >> /etc/R/Rprofile.site \
        && echo 'source("/etc/R/Rprofile.site")' >> /etc/littler.r \
    && ln -s /usr/share/doc/littler/examples/install.r /usr/local/bin/install.r \
    && ln -s /usr/share/doc/littler/examples/install2.r /usr/local/bin/install2.r \
    && ln -s /usr/share/doc/littler/examples/installGithub.r /usr/local/bin/installGithub.r \
    && ln -s /usr/share/doc/littler/examples/testInstalled.r /usr/local/bin/testInstalled.r \
    && install.r docopt \
    && rm -rf /tmp/downloaded_packages/ /tmp/*.rds \
&& rm -rf /var/lib/apt/lists/*

RUN     R -e 'install.packages("devtools")' && \
    R -e 'devtools::install_github("IRkernel/IRkernel")' && \
    # or 'devtools::install_local("IRkernel-master.tar.gz")' && \
    R -e 'IRkernel::installspec()' && \
# to register the kernel in the current R installation 
    R -e 'install.packages("googleAuthR"); library(googleAuthR); gar_gce_auth()' && \
    R -e 'install.packages("bigQueryR"); install.packages("googleCloudStorageR")' && \
    R -e 'install.packages("feather")' && \
    R -e 'install.packages("tensorflow")' && \
    R -e 'devtools::install_github("apache/spark@v2.2.0", subdir="R/pkg")'


# Install google cloud storage fuse (gcsfuse) to mount cloud storage buckets in datalab VM - specific to Ubuntu Xenial 
RUN export GCSFUSE_REPO=gcsfuse-xenial \
	&& echo "deb http://packages.cloud.google.com/apt $GCSFUSE_REPO main" | sudo tee /etc/apt/sources.list.d/gcsfuse.list \
	&& curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - \
	&& apt-get update \
	&& apt-get install gcsfuse -y --allow-unauthenticated
