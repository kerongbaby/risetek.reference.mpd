# $Id: Makefile.doc 2169 2014-04-14 12:41:31Z dmitryluhtionov $

PREFIX?=	/usr/local
DOCDIR?=	${PREFIX}/share/doc/mpd5

DOCOWN!=	/usr/bin/id -n -u
DOCGRP!=	/usr/bin/id -n -g
MANOWN=		${DOCOWN}
MANGRP=		${DOCGRP}

MPD_DOC_FILES+=	../README
MPD_DOC_FILES+=	mpd*.html
MPD_DOC_FILES+=	mpd.ps

MAN8=		mpd5.8
MANDIR=		${PREFIX}/man/man

depend:

clean cleandir:
	rm -f mpd5.8

all:		mpd5.8

mpd5.8:          mpd5.8.in
	sed -e "s,@PREFIX@,${PREFIX},g" < ${.ALLSRC} > ${.TARGET}

.if !defined(NOPORTDOCS)
afterinstall:
	[ -d ${DESTDIR}${DOCDIR} ] || mkdir ${DESTDIR}${DOCDIR}
	${INSTALL} -c -o ${DOCOWN} -g ${DOCGRP} -m ${DOCMODE} \
	    ${MPD_DOC_FILES} ${DESTDIR}${DOCDIR}
.endif

.include <bsd.prog.mk>
