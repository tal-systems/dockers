FROM ubuntu:focal

LABEL maintainer="Jakub Lewandowski <jakub.lewandowski@gmail.com>"

ENV GCC_VERSION=11 \
    CMAKE_VERSION_FULL=3.21.2 \
    CC=/usr/bin/gcc \
    CXX=/usr/bin/g++ \
    PYENV_ROOT=/opt/pyenv \
    PYTHON_VERSION=3.9.5 \
    PATH=/opt/pyenv/shims:${PATH} \
    DEBIAN_FRONTEND=noninteractive

RUN dpkg --add-architecture i386 \
    && apt-get -qq update \
    && apt-get -qq install -y --no-install-recommends --no-install-suggests gnupg \
    && printf "deb http://ppa.launchpad.net/ubuntu-toolchain-r/test/ubuntu focal main\n" >> /etc/apt/sources.list \
    && apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 60C317803A41BA51845E371A1E9377A2BA9EF27F \
    && apt-get -qq update \
    && apt-get -qq install -y --no-install-recommends --no-install-suggests \
       autoconf-archive \
       bash-completion \
       binutils \
       ca-certificates \
       cargo \
       curl \
       dh-autoreconf \
       g++-${GCC_VERSION} \
       git \
       git-core \
       less \
       libbz2-dev \
       libc6-dev \
       libffi-dev \
       libgmp-dev \
       liblzma-dev \
       libmpc-dev \
       libmpfr-dev \
       libncurses5-dev \
       libncursesw5-dev \
       libreadline-dev \
       libsqlite3-dev \
       libssl-dev \
       nasm \
       ninja-build \
       nodejs \
       npm \
       openssh-client \
       pkg-config \
       subversion \
       sudo \
       tmux \
       unzip \
       wget \
       xz-utils \
       zlib1g-dev \
    && update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-${GCC_VERSION} 100 \
    && update-alternatives --install /usr/bin/c++ c++ /usr/bin/g++-${GCC_VERSION} 100 \
    && update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-${GCC_VERSION} 100 \
    && update-alternatives --install /usr/bin/cc cc /usr/bin/gcc-${GCC_VERSION} 100 \
    && ln -s /usr/include/locale.h /usr/include/xlocale.h \
    && rm -rf /var/lib/apt/lists/* \
    && curl -fL https://getcli.jfrog.io | sh \
    && mv jfrog /usr/local/bin/jfrog \
    && chmod +x /usr/local/bin/jfrog \
    && groupadd _1001 -g 1001 \
    && groupadd _1000 -g 1000 \
    && groupadd _2000 -g 2000 \
    && groupadd _999 -g 999 \
    && useradd -ms /bin/bash me -g _1001 -G _1000,_2000,_999 \
    && printf "me:me" | chpasswd \
    && adduser me sudo \
    && printf "me ALL= NOPASSWD: ALL\\n" >> /etc/sudoers \
    && wget --no-check-certificate --quiet -O /tmp/pyenv-installer https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer \
    && chmod +x /tmp/pyenv-installer \
    && /tmp/pyenv-installer \
    && rm /tmp/pyenv-installer \
    && update-alternatives --install /usr/bin/pyenv pyenv /opt/pyenv/bin/pyenv 100 \
    && PYTHON_CONFIGURE_OPTS="--enable-shared" pyenv install ${PYTHON_VERSION} \
    && pyenv global ${PYTHON_VERSION} \
    && pip install -q --upgrade --no-cache-dir pip \
    && pip install -q --no-cache-dir conan conan-package-tools cmake==${CMAKE_VERSION_FULL} \
    && pip install pycapnp \
    && pip install git+https://github.com/tal-systems/tal-conan-utils \
    && chown -R me:1001 /opt/pyenv \
    && find /opt/pyenv -iname __pycache__ -print0 | xargs -0 rm -rf \
    && update-alternatives --install /usr/bin/python python /opt/pyenv/shims/python 100 \
    && update-alternatives --install /usr/bin/python3 python3 /opt/pyenv/shims/python3 100 \
    && update-alternatives --install /usr/bin/pip pip /opt/pyenv/shims/pip 100 \
    && update-alternatives --install /usr/bin/pip3 pip3 /opt/pyenv/shims/pip3 100

USER me
WORKDIR /home/me

RUN mkdir -p /home/me/.conan \
    && printf 'eval "$(pyenv init -)"\n' >> ~/.bashrc \
    && printf 'eval "$(pyenv virtualenv-init -)"\n' >> ~/.bashrc \
    && conan profile new default --detect \
    && conan profile update settings.compiler.libcxx=libstdc++11 default \
    && conan profile update settings.compiler.cppstd=20 default \
    && conan config init \
    && conan config set general.revisions_enabled=True \
    && curl -s https://raw.githubusercontent.com/LunarVim/LunarVim/rolling/utils/installer/install-neovim-from-release --output /tmp/install-neovim-from-release \
    && bash /tmp/install-neovim-from-release \
    && curl -s https://raw.githubusercontent.com/lunarvim/lunarvim/master/utils/installer/install.sh --output /tmp/install-lvim.sh \
    && bash -l /tmp/install-lvim.sh --no-install-dependencies \
    && echo 'lvim.plugins = {{"arcticicestudio/nord-vim"}, {"lunarvim/colorschemes"}}' >> ~/.config/lvim/config.lua \
    && bash -l lvim --headless -c 'autocmd User PackerComplete quitall' -c 'PackerSync' \
    && sed -i s/onedarker/nord/g ~/.config/lvim/config.lua \
    && (bash -l lvim --headless -es 'LspInstall --sync clangd' || true)

ENTRYPOINT /bin/bash -l
