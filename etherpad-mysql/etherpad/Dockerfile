FROM etherpad_etherpad

# apply the patches
COPY patches /opt/patches-mysql
RUN set -e ; for p in /opt/patches-mysql/*.patch ; do echo "$p" ; patch -p1 <"$p" ; done
