<!--
author: Magelive
date: 2016-12-12
title: libuv 错误码分析
tags: Linux,libuv,error,	errno
category: libuv
status: publish
summary: libuv中错误码的设计不再遵循linux或win系统的错误码，而是重新设计了一套错误码结构。其使用C define的特性重新设计了一套错误码。
-->
 
libuv中错误码的设计不再遵循linux或win系统的错误码，而是重新设计了一套错误码结构。其使用C define的特性重新设计了一套错误码。

## libuv根据错误码获取错误信息说明

	# define UV_ERRNO_MAP(XX)
		XX(E2BIG, "argument list too long")                                         \
		XX(EACCES, "permission denied")                                             \
		XX(EADDRINUSE, "address already in use")                                    \
		XX(EADDRNOTAVAIL, "address not available")                                  \
		XX(EAFNOSUPPORT, "address family not supported")                            \
		XX(EAGAIN, "resource temporarily unavailable")                              \
		XX(EAI_ADDRFAMILY, "address family not supported")                          \
		XX(EAI_AGAIN, "temporary failure")                                          \
		XX(EAI_BADFLAGS, "bad ai_flags value")                                      \
		XX(EAI_BADHINTS, "invalid value for hints")                                 \
		XX(EAI_CANCELED, "request canceled")                                        \
		XX(EAI_FAIL, "permanent failure")                                           \
		XX(EAI_FAMILY, "ai_family not supported")                                   \
		XX(EAI_MEMORY, "out of memory")                                             \
		XX(EAI_NODATA, "no address")                                                \
		XX(EAI_NONAME, "unknown node or service")                                   \
		XX(EAI_OVERFLOW, "argument buffer overflow")                                \
		XX(EAI_PROTOCOL, "resolved protocol is unknown")                            \
		XX(EAI_SERVICE, "service not available for socket type")                    \
		XX(EAI_SOCKTYPE, "socket type not supported")                               \
		XX(EALREADY, "connection already in progress")                              \
		XX(EBADF, "bad file descriptor")                                            \
		XX(EBUSY, "resource busy or locked")                                        \
		XX(ECANCELED, "operation canceled")                                         \
		XX(ECHARSET, "invalid Unicode character")                                   \
		XX(ECONNABORTED, "software caused connection abort")                        \
		XX(ECONNREFUSED, "connection refused")                                      \
		XX(ECONNRESET, "connection reset by peer")                                  \
		XX(EDESTADDRREQ, "destination address required")                            \
		XX(EEXIST, "file already exists")                                           \
		XX(EFAULT, "bad address in system call argument")                           \
		XX(EFBIG, "file too large")                                                 \
		XX(EHOSTUNREACH, "host is unreachable")                                     \
		XX(EINTR, "interrupted system call")                                        \
		XX(EINVAL, "invalid argument")                                              \
		XX(EIO, "i/o error")                                                        \
		XX(EISCONN, "socket is already connected")                                  \
		XX(EISDIR, "illegal operation on a directory")                              \
		XX(ELOOP, "too many symbolic links encountered")                            \
		XX(EMFILE, "too many open files")                                           \
		XX(EMSGSIZE, "message too long")                                            \
		XX(ENAMETOOLONG, "name too long")                                           \
		XX(ENETDOWN, "network is down")                                             \
		XX(ENETUNREACH, "network is unreachable")                                   \
		XX(ENFILE, "file table overflow")                                           \
		XX(ENOBUFS, "no buffer space available")                                    \
		XX(ENODEV, "no such device")                                                \
		XX(ENODEV, "no such device")                                                \
		XX(ENOENT, "no such file or directory")                                     \
		XX(ENOMEM, "not enough memory")                                             \
		XX(ENONET, "machine is not on the network")                                 \
		XX(ENOPROTOOPT, "protocol not available")                                   \
		XX(ENOSPC, "no space left on device")                                       \
		XX(ENOSYS, "function not implemented")                                      \
		XX(ENOTCONN, "socket is not connected")                                     \
		XX(ENOTDIR, "not a directory")                                              \
		XX(ENOTEMPTY, "directory not empty")                                        \
		XX(ENOTSOCK, "socket operation on non-socket")                              \
		XX(ENOTSUP, "operation not supported on socket")                            \
		XX(EPERM, "operation not permitted")                                        \
		XX(EPIPE, "broken pipe")                                                    \
		XX(EPROTO, "protocol error")                                                \
		XX(EPROTONOSUPPORT, "protocol not supported")                               \
		XX(EPROTOTYPE, "protocol wrong type for socket")                            \
		XX(ERANGE, "result too large")                                              \
		XX(EROFS, "read-only file system")                                          \
		XX(ESHUTDOWN, "cannot send after transport endpoint shutdown")              \
		XX(ESPIPE, "invalid seek")                                                  \
		XX(ESRCH, "no such process")                                                \
		XX(ETIMEDOUT, "connection timed out")                                       \
		XX(ETXTBSY, "text file is busy")                                            \  
		XX(EXDEV, "cross-device link not permitted")                                \
		XX(UNKNOWN, "unknown error")                                                \
		XX(EOF, "end of file")                                                      \
		XX(ENXIO, "no such device or address")                                      \
		XX(EMLINK, "too many links")                                                \
		XX(EHOSTDOWN, "host is down")                                     

    typedef enum {
		# define XX(code, _) UV_ ##  code = UV__ ##  code,
  			UV_ERRNO_MAP(XX)
		# undef XX
  		UV_ERRNO_MAX = UV__EOF - 1
	} uv_errno_t;

	# define UV_ERR_NAME_GEN(name, _) case UV_ ##  name: return # name;
    const char* *uv_err_name*(int err) {
    	switch (err) {
    		UV_ERRNO_MAP(UV_ERR_NAME_GEN)
    	}
    	return uv__unknown_err_code(err);
    }
    # undef UV_ERR_NAME_GEN

    # define UV_STRERROR_GEN(name, msg) case UV_ ##  name: return msg;
    const char* uv_strerror(int err) {
    	switch (err) {
    		UV_ERRNO_MAP(UV_STRERROR_GEN)
    	}
    	return uv__unknown_err_code(err);
    }
    # undef UV_STRERROR_GEN

根据所定义的宏，我们将其中的宏展开看

	typef enum{
		UV_E2BIG = UV__E2BIG,
		UV_EACCES = UV__EACCES,
		……
		UV_ERRNO_MAX = UV__EOF - 1
	} uv_errno_t;

UV__类的宏，都定义在uv-errno.h文件中

我们再展开uv_err_name函数：

    const char* *uv_err_name*(int err) {
    	switch (err) {
    		case EV_E2BIG: return "E2BIG";
    		case EV_EACCES: return "EACCES";
    		……
    	}
    	return uv__unknown_err_code(err);
    }

再展开uv_strerror函数：

    const char* uv_strerror(int err) {
    	switch (err) {
    		case UV_E2BIG: return "argument list too long";
    		case UV_EACCES: return "permission denied";
    		……
    	}
    	return uv__unknown_err_code(err);
    }

然后就是uv__unknown_err_code函数：
```c
	static const char* uv__unknown_err_code(int err) {
	char buf[32];
	char* copy;

	snprintf(buf, sizeof(buf), "Unknown system error %d", err);
	  copy = uv__strdup(buf);

	return copy != NULL ? copy : "Unknown system error";
}
```
		
	

