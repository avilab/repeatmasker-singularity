BootStrap: docker
From: debian:stretch

%labels
  Maintainer tpall
  RM_Version 4.0.9
  RMBlast Version 2.9.0
  TRF Version 4.09
  Repbase_Version 20181026

%apprun RepeatMasker
  exec /usr/local/RepeatMasker/RepeatMasker "${@}"

%runscript
  exec /usr/local/RepeatMasker/RepeatMasker "${@}"

%files
  RepBaseRepeatMaskerEdition-20181026.tar.gz /tmp
  test/seqs/small-1.fa /tmp
  test/seqs/small-2.fa /tmp

%post
  # Software versions
  export RM_VERSION=${RM_VERSION:-4.0.9.p2}
  export RMB_VERSION=${RMB_VERSION:-2.9.0}
  export TRF_VERSION=${TRF_VERSION:-409}
  export REPBASE_VER=${REPBASE_VER:-20181026}

  # Get build dependencies
  apt-get update \
    && apt-get install -y --no-install-recommends wget build-essential locales
  
  # Configure default locale
  echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
  locale-gen en_US.utf8
  /usr/sbin/update-locale LANG=en_US.UTF-8
  export LC_ALL=en_US.UTF-8
  export LANG=en_US.UTF-8

  # Configure term
  export TERM=xterm

  # Install cpanm
  wget -O - http://cpanmin.us | perl - --self-upgrade
  
  # Install the Text::Soundex module via cpan:
  cpanm Text::Soundex

  ## Download RMBlast
  cd /tmp
  wget -nv http://www.repeatmasker.org/rmblast-${RMB_VERSION}+-x64-linux.tar.gz
  cd /usr/local \
    && tar zxvf /tmp/rmblast-${RMB_VERSION}+-x64-linux.tar.gz

  ## Download TRF
  cd /tmp
  wget -nv http://tandem.bu.edu/trf/downloads/trf${TRF_VERSION}.linux64
  cp trf${TRF_VERSION}.linux64 /usr/local/bin/ \
    && mv /usr/local/bin/trf${TRF_VERSION}.linux64 /usr/local/bin/trf \
    && chmod +x /usr/local/bin/trf

  ## Download and install RepeatMasker
  wget -nv http://www.repeatmasker.org/RepeatMasker-open-$(echo $RM_VERSION | sed -e 's/\./\-/g').tar.gz
  cp RepeatMasker-open-$(echo $RM_VERSION | sed -e 's/\./\-/g').tar.gz /usr/local/
  cd /usr/local/ \
    && gunzip RepeatMasker-open-$(echo $RM_VERSION | sed -e 's/\./\-/g').tar.gz \
    && tar xvf RepeatMasker-open-$(echo $RM_VERSION | sed -e 's/\./\-/g').tar \
    && rm RepeatMasker-open-$(echo $RM_VERSION | sed -e 's/\./\-/g').tar

  ln -s /usr/local/RepeatMasker/RepeatMasker /usr/local/bin/RepeatMasker

  ## Setup RepBase RepeatMasker Edition
  cp /tmp/RepBaseRepeatMaskerEdition-${REPBASE_VER}.tar.gz /usr/local/RepeatMasker \
    && cd /usr/local/RepeatMasker \
    && tar zxf RepBaseRepeatMaskerEdition-${REPBASE_VER}.tar.gz \
    && rm RepBaseRepeatMaskerEdition-${REPBASE_VER}.tar.gz

  ## Run Configure Script
  rm configure \
    && wget -nv --no-check-certificate https://raw.githubusercontent.com/rmhubley/RepeatMasker/master/configure \
    && perl ./configure --trfbin=/usr/local/bin/trf --rmblastbin=/usr/local/rmblast-${RMB_VERSION}/
  
  ## Run RM twice
  /usr/local/bin/RepeatMasker /tmp/small-1.fa
  /usr/local/bin/RepeatMasker /tmp/small-2.fa

  ## Clean up from source install
  cd / \
  && rm -rf /tmp/* \
  && apt-get autoremove -y \
  && apt-get autoclean -y \
  && rm -rf /var/lib/apt/lists/* 
