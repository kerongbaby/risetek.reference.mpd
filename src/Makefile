# $Id: Makefile 2384 2020-09-06 05:17:49Z dadv $
#
# Makefile for mpd, multi-link PPP daemon for FreeBSD
#
# Written by Archie Cobbs <archie@freebsd.org>
# Copyright (c) 1995-1999 Whistle Communications, Inc. All rights reserved.
# See ``COPYRIGHT.whistle''
#

PROG?=			mpd5
PREFIX?=		/usr/local
LOCALBASE?=		${PREFIX}
BINDIR?=		${PREFIX}/sbin
VERSION=		5.9
NOMAN=
MK_MAN=			no

BINOWN!=		/usr/bin/id -n -u
BINGRP!=		/usr/bin/id -n -g

# Default configuration directory

MPD_CONF_DIR?=		${PREFIX}/etc/mpd5

# Define supported physical layer types

PHYSTYPE_MODEM=		yes
PHYSTYPE_TCP=		yes
PHYSTYPE_UDP=		yes
PHYSTYPE_NG_SOCKET=	yes
PHYSTYPE_PPTP=		yes
PHYSTYPE_PPPOE=		yes
PHYSTYPE_L2TP=		yes

# Define supported CCP types

CCP_DEFLATE=		yes
CCP_MPPC=		yes
CCP_PRED1=		yes

# Define supported ECP types

ECP_DES=		yes

# Define system features wanted to use (if present)

USE_NG_BPF=		yes
USE_NG_CAR=		yes
USE_NG_DEFLATE=		yes
USE_NG_IPACCT?=		yes
USE_NG_MPPC=		yes
USE_NG_NAT=		yes
USE_NG_NETFLOW=		yes
USE_NG_PRED1=		yes
USE_NG_TCPMSS=		yes
USE_NG_VJC=		yes
USE_IPFW=		yes
USE_FETCH=		yes
USE_TCP_WRAP=		yes
#USE_AUTH_OPIE=		yes
USE_AUTH_PAM=		yes
USE_AUTH_SYSTEM=	yes

# Build without builtin web server.
#NOWEB=			yes

# Set syslog logging facility. Change LOG_DAEMON to whatever you like.
# Comment this line disable syslog (3) support
SYSLOG_FACILITY=	LOG_DAEMON

# Print wide protocol names instead of short names.
# Comment this line, to save about 10 KB space
CFLAGS+=	-DPROTO_NAME_LIST

# Reduce some internal structures size to save more memory
# This is to limit amount of active sessions
#SMALL_SYSTEM=		yes

# Compiler & linker flags

.if exists ( /usr/lib/libwrap.so ) && defined ( USE_TCP_WRAP )
## filter TCP accept connections
LDADD+=		-lwrap
DPADD+=		${LIBWRAP}
CFLAGS+=	-DUSE_WRAP
.endif

## authentication methods: opie, pam, /etc/password
.if defined ( USE_AUTH_OPIE )
LDADD+=		-lopie
DPADD+=		${LIBOPIE}
CFLAGS+=	-DUSE_OPIE
.endif

.if exists ( /usr/lib/libpam.so ) && defined ( USE_AUTH_PAM )
LDADD+=		-lpam
DPADD+=		${LIBPAM}
CFLAGS+=	-DUSE_PAM
.endif

.if defined ( USE_AUTH_SYSTEM )
## based on /etc/password
LDADD+=		-lcrypt
DPADD+=		${LIBCRYPT} ${LIBUTIL}
CFLAGS+=	-DUSE_SYSTEM
.endif

LDADD+=		-lnetgraph -lutil
DPADD+=		${LIBNETGRAPH}

LDADD+=		-L/usr/lib -lradius
DPADD+=		${LIBRADIUS}

# Obtained from bsd.port.mk
.if !defined ( OSVERSION )
OSVERSION!=	awk '/^\#define[[:blank:]]__FreeBSD_version/ {print $$3}' < /usr/include/sys/param.h
.endif

.sinclude "/etc/src.conf"

.if ${OSVERSION} < 1200084 || defined ( WITHOUT_REPRODUCIBLE_BUILD )
MPD_VERSION!=	echo -n "${VERSION} (`id -un`@`uname -n` `LC_TIME=C date +'%R %v'`)"
.else
MPD_VERSION!=	echo -n "${VERSION}"
.endif

CFLAGS+=	-DPATH_CONF_DIR='"${MPD_CONF_DIR}"'
CFLAGS+=	-DMPD_VERSION='"${MPD_VERSION}"'
CFLAGS+=	-g
CFLAGS+=	-Wall \
		-Wcast-align \
		-Wchar-subscripts \
		-Wformat \
		-Winline \
		-Wmissing-declarations \
		-Wmissing-prototypes \
		-Wnested-externs \
		-Wpointer-arith \
		-Wwrite-strings \
		-pthread

.if defined ( MPD_VENDOR )
CFLAGS+=	-DMPD_VENDOR='"${MPD_VENDOR}"'
.endif

.if defined ( SYSLOG_FACILITY )
CFLAGS+=	-DSYSLOG_FACILITY='"${SYSLOG_FACILITY}"'
.endif

# Standard sources

STDSRCS=        assert.c auth.c bund.c rep.c ccp.c chap.c \
		console.c command.c ecp.c event.c fsm.c iface.c input.c \
		ip.c ipcp.c ipv6cp.c lcp.c link.c log.c main.c mbuf.c mp.c \
		msg.c ngfunc.c pap.c phys.c proto.c radius.c radsrv.c timer.c \
		util.c vars.c eap.c msoft.c ippool.c

.if defined ( NOWEB )
CFLAGS+=	-DNOWEB
.else
STDSRCS+=	web.c
LDADD+=		-lssl
.endif

CFLAGS+=	-DNOLIBPDEL -I. -I./contrib/libpdel
PDPATH1=	contrib/libpdel/util:contrib/libpdel/structs
PDPATH2=	contrib/libpdel/structs/type
.if defined ( NOWEB )
VPATH+=		$(PDPATH1):$(PDPATH2)
.else
PDPATH3=	contrib/libpdel/http:contrib/libpdel/http/servlet
PDPATH4=	contrib/libpdel/io
VPATH+=		$(PDPATH1):$(PDPATH2):$(PDPATH3):$(PDPATH4)
.endif
PDELSRCS=	typed_mem.c \
		pevent.c \
		paction.c \
		ghash.c \
		gtree.c \
		mesg_port.c \
		structs.c \
		structs_generic.c \
		structs_type_array.c \
		structs_type_int.c \
		structs_type_string.c \
		structs_type_struct.c
.if !defined ( NOWEB )
PDELSRCS+=	http_connection.c \
		http_head.c \
		http_message.c \
		http_mime.c \
		http_request.c \
		http_response.c \
		http_server.c \
		http_servlet_basicauth.c \
		http_ssl.c \
		http_status.c \
		boundary_fp.c \
		ssl_fp.c \
		string_fp.c \
		timeout_fp.c
.endif

# Add sources and flags for the various physical layer types

.if defined ( PHYSTYPE_MODEM )
SRCS+=		modem.c chat.c
CFLAGS+=	-DPHYSTYPE_MODEM
.endif
.if defined ( PHYSTYPE_UDP )
SRCS+=		udp.c
CFLAGS+=	-DPHYSTYPE_UDP
.endif
.if defined ( PHYSTYPE_TCP )
SRCS+=		tcp.c
CFLAGS+=	-DPHYSTYPE_TCP
.endif
.if defined ( PHYSTYPE_NG_SOCKET )
SRCS+=		ng.c
CFLAGS+=	-DPHYSTYPE_NG_SOCKET
.endif
.if defined ( PHYSTYPE_PPTP )
SRCS+=		pptp.c pptp_ctrl.c
CFLAGS+=	-DPHYSTYPE_PPTP
CFLAGS+=	-DRANDOMIZE_CID
#CFLAGS+=	-DLOOK_LIKE_NT
.endif
.if defined ( PHYSTYPE_PPPOE )
SRCS+=		pppoe.c
CFLAGS+=	-DPHYSTYPE_PPPOE
.endif
.if defined ( PHYSTYPE_L2TP )
SRCS+=		l2tp.c l2tp_avp.c l2tp_ctrl.c
CFLAGS+=	-DPHYSTYPE_L2TP
.endif

# CCP

.if defined ( CCP_DEFLATE ) && defined ( USE_NG_DEFLATE )
SRCS+=		ccp_deflate.c
CFLAGS+=	-DCCP_DEFLATE
CFLAGS+=	-DUSE_NG_DEFLATE
.endif

.if defined ( CCP_MPPC ) && defined ( USE_NG_MPPC )
SRCS+=		ccp_mppc.c
CFLAGS+=	-DCCP_MPPC
CFLAGS+=	-DUSE_NG_MPPC
.endif

.if defined ( CCP_PRED1 )
SRCS+=		ccp_pred1.c
CFLAGS+=	-DCCP_PRED1
.if defined ( USE_NG_PRED1 )
CFLAGS+=	-DUSE_NG_PRED1
.endif
.endif

# ECP

.if defined ( ECP_DES )
SRCS+=		ecp_dese.c
SRCS+=		ecp_dese_bis.c
CFLAGS+=	-DECP_DES
NEED_DES=	yes
.endif

# Features

.if defined ( USE_NG_BPF )
CFLAGS+=	-DUSE_NG_BPF
LDADD+=		-lpcap
DPADD+=		${LIBPCAP}
.endif
.if defined ( USE_NG_CAR )
CFLAGS+=	-DUSE_NG_CAR
.endif
.if defined ( USE_NG_IPACCT ) && ( ${USE_NG_IPACCT} == yes )
CFLAGS+=	-DUSE_NG_IPACCT -nostdinc -I/usr/include -I${LOCALBASE}/include
.endif
.if defined ( USE_NG_NAT )
SRCS+=		nat.c
CFLAGS+=	-DUSE_NG_NAT
.endif
.if defined ( USE_NG_NETFLOW )
CFLAGS+=	-DUSE_NG_NETFLOW
.endif
.if defined ( USE_NG_TCPMSS )
CFLAGS+=	-DUSE_NG_TCPMSS
.endif
.if defined ( USE_NG_VJC )
CFLAGS+=	-DUSE_NG_VJC
.endif
.if defined ( USE_IPFW )
CFLAGS+=	-DUSE_IPFW
.endif
.if defined ( USE_FETCH )
CFLAGS+=	-DUSE_FETCH
LDADD+=		-lfetch
DPADD+=		${LIBFETCH}
.endif
.if defined ( SMALL_SYSTEM )
CFLAGS+=	-DSMALL_SYSTEM
.endif

# Add in required support files and libraries
LDADD+=		-lcrypto
DPADD+=		${LIBCRYPTO}

.if exists ( /usr/lib/libexecinfo.so )
LDADD+=		-lexecinfo
CFLAGS+=	-DUSE_BACKTRACE
.endif

# Now add standard sources

SRCS+=		${STDSRCS}
SRCS+=		${PDELSRCS}

.include <bsd.prog.mk>
