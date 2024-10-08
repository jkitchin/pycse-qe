# https://jupyter-docker-stacks.readthedocs.io/en/latest/
# Note this takes about 30 minutes to build

FROM jupyter/scipy-notebook:python-3.11
MAINTAINER John Kitchin <jkitchin@andrew.cmu.edu>

# Set the default shell to bash instead of sh so the source commands work
SHELL ["/bin/bash", "-c"]

USER root


# These packages are to get PDF export via LaTeX (minimal) and via webpdf
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends pandoc \
    texlive-xetex texlive-fonts-recommended texlive-plain-generic \
    xorg xvfb gtk2-engines-pixbuf \
    dbus-x11 xfonts-base xfonts-100dpi xfonts-75dpi xfonts-cyrillic xfonts-scalable \
    libnss3-dev libxss1 libcups2 libasound2 libpangocairo-1.0-0 \
    libatk1.0-0 libatk-bridge2.0-0 libgtk-3-0 \
    nodejs npm libzmq3-dev tree \
    curl jq graphviz file

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
    software-properties-common \
    quantum-espresso 
         
# Note: usually you have to download these psp files and agree to the terms
# See https://www.materialscloud.org/discover/sssp/table/efficiency
COPY ./SSSP.tar.gz /SSSP.tar.gz
RUN mkdir -p /usr/share/espresso/pseudo \
    && tar xf /SSSP.tar.gz -C /usr/share/espresso/pseudo \
    && rm /SSSP.tar.gz
     
# Do not exclude man pages & other documentation
RUN rm /etc/dpkg/dpkg.cfg.d/excludes
# Reinstall all currently installed packages in order to get the man pages back
RUN apt-get update && \
    dpkg -l | grep ^ii | cut -d' ' -f3 | xargs apt-get install -y --reinstall && \
    rm -r /var/lib/apt/lists/*

# This is for jupyter-lsp
# see https://github.com/jupyter-lsp/jupyterlab-lsp#installation
RUN ln -s / .lsp_symlink
USER ${NB_UID}	

RUN python -m pip install --upgrade pip \
	&& python -m pip install pycse pint quantities uncertainties \
	nbconvert nbconvert[webpdf] \
	black flake8 pylint nbmake \
	jupyter-book matplotlib numpy ghp-import pandas sympy autograd numpydoc joblib plotly \
        jupyter_server \
        jupyterlab jupyterlab-lsp 'python-lsp-server[all]'\
	jupyterlab-git jupyter-videochat \
	jupyterlab-spellchecker \
        jupyterlab-code-formatter \
#        jupyter_collaboration \
        pydotplus ase \
        parsl \
        pyppeteer \
	&& jupyter labextension install plotlywidget \
#	&& pip install jupyterlab-link-share \
	&& jupyter server extension disable nbclassic 


# Now all the jupyter lab settings
COPY jupyter_lab_config.py /home/jovyan/.jupyter/jupyter_lab_config.py

# Get pyppeteer working for pdf export with webpdf. I am not sure how to get the
# number in the path automatically, this is pretty hackish.
RUN pyppeteer-install \
	&& mv /home/jovyan/.local/share/pyppeteer/local-chromium/1181205/chrome-linux/chrome /home/jovyan/.local/share/pyppeteer/local-chromium/1181205/chrome-linux/chrome-exe

COPY chrome /home/jovyan/.local/share/pyppeteer/local-chromium/1181205/chrome-linux/chrome

# Now all the jupyter lab settings
COPY jupyter_nbconvert_config.py /home/jovyan/.jupyter/jupyter_nbconvert_config.py
COPY jupyter_lab_config.py /home/jovyan/.jupyter/jupyter_lab_config.py
COPY jupyter_server_config.json /home/jovyan/.jupyter/jupyter_server_config.json

COPY en_US.dic /home/jovyan/.local/share/jupyter/dictionaries/en_US.dic
COPY en_US.aff /home/jovyan/.local/share/jupyter/dictionaries/en_US.aff

# For code formatting
COPY jupyterlab-code-formatter.json /home/jovyan/.jupyter/lab/user-settings/\@ryantam626/jupyterlab_code_formatter/settings.jupyterlab-settings
COPY keyboard-shortcuts.json /home/jovyan/.jupyter/lab/user-settings/\@jupyterlab/shortcuts-extension/shortcuts.jupyterlab-settings

# For some reason, some files are owned by root, and that seems to cause issues launching jupyter lab
# I hope this fixes it.
USER root
RUN chown -R jovyan:users /home/jovyan/.jupyter \
	&& chown -R jovyan:users /home/jovyan/.local
USER ${NB_UID}

# # change the final working dir to the place you mount your CWD
WORKDIR /home/jovyan/work

