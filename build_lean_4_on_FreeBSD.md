# env 
recommaned OS: FreeBSD 13.2

# build steps 

## 1 install some dependent package

```
pkg install cmake gmake bash gmp 
```

## 2 fetch source code 

```
wget https://github.com/leanprover/lean4/archive/refs/tags/v4.4.0.tar.gz
```

## 3 modify the source codes

### 3.1 add option -fPIC

#### 3.1.1 stage0/src/CMakeLists.txt

around line 346, the codes like this:
```
if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  if(BSYMBOLIC)
    string(APPEND LEANC_SHARED_LINKER_FLAGS " -Wl,-Bsymbolic")
    string(APPEND LEANSHARED_LINKER_FLAGS " -Wl,-Bsymbolic")
  endif()
  string(APPEND CMAKE_CXX_FLAGS " -fPIC -ftls-model=initial-exec")
  string(APPEND LEANC_EXTRA_FLAGS " -fPIC")
  string(APPEND LEANSHARED_LINKER_FLAGS " -Wl,-rpath=\\$$ORIGIN/..:\\$$ORIGIN")
  string(APPEND CMAKE_EXE_LINKER_FLAGS " -lleanshared -Wl,-rpath=\\\$ORIGIN/../lib:\\\$ORIGIN/../lib/lean")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  string(APPEND CMAKE_CXX_FLAGS " -ftls-model=initial-exec")
  string(APPEND LEANSHARED_LINKER_FLAGS " -install_name @rpath/libleanshared.dylib")
  string(APPEND CMAKE_EXE_LINKER_FLAGS " -lleanshared -Wl,-rpath,@executable_path/../lib -Wl,-rpath,@executable_path/../lib/lean")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Emscripten")
```
there are configs for each platform. 

add configs for FreeBSD to support -PIC.

the final configs look like this:

```
if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  if(BSYMBOLIC)
    string(APPEND LEANC_SHARED_LINKER_FLAGS " -Wl,-Bsymbolic")
    string(APPEND LEANSHARED_LINKER_FLAGS " -Wl,-Bsymbolic")
  endif()
  string(APPEND CMAKE_CXX_FLAGS " -fPIC -ftls-model=initial-exec")
  string(APPEND LEANC_EXTRA_FLAGS " -fPIC")
  string(APPEND LEANSHARED_LINKER_FLAGS " -Wl,-rpath=\\$$ORIGIN/..:\\$$ORIGIN")
  string(APPEND CMAKE_EXE_LINKER_FLAGS " -lleanshared -Wl,-rpath=\\\$ORIGIN/../lib:\\\$ORIGIN/../lib/lean")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  string(APPEND CMAKE_CXX_FLAGS " -ftls-model=initial-exec")
  string(APPEND LEANSHARED_LINKER_FLAGS " -install_name @rpath/libleanshared.dylib")
  string(APPEND CMAKE_EXE_LINKER_FLAGS " -lleanshared -Wl,-rpath,@executable_path/../lib -Wl,-rpath,@executable_path/../lib/lean")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Emscripten")
  string(APPEND CMAKE_CXX_FLAGS " -fPIC")
  string(APPEND LEANC_EXTRA_FLAGS " -fPIC")
  # We do not use dynamic linking via leanshared for Emscripten to keep things
  # simple. (And we are not interested in `Lake` anyway.) To use dynamic
  # linking, we would probably have to set MAIN_MODULE=2 on `leanshared`,
  # SIDE_MODULE=2 on `lean`, and set CMAKE_SHARED_LIBRARY_SUFFIX to ".js".
  string(APPEND CMAKE_EXE_LINKER_FLAGS " -Wl,--whole-archive -lInit -lLean -lleancpp -lleanrt ${EMSCRIPTEN_SETTINGS} -lnodefs.js -s EXIT_RUNTIME=1 -s MAIN_MODULE=1 -s LINKABLE=1 -s EXPORT_ALL=1")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
  string(APPEND CMAKE_EXE_LINKER_FLAGS " -lleanshared")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "FreeBSD")
  string(APPEND CMAKE_CXX_FLAGS " -fPIC -ftls-model=initial-exec")
  string(APPEND LEANC_EXTRA_FLAGS " -fPIC")
  string(APPEND LEANSHARED_LINKER_FLAGS " -Wl,-rpath=\\$$ORIGIN/..:\\$$ORIGIN")
  string(APPEND CMAKE_EXE_LINKER_FLAGS " -lleanshared -Wl,-rpath=\\\$ORIGIN/../lib:\\\$ORIGIN/../lib/lean")
endif()
```

#### 3.1.2  src/CMakeLists.txt 

around line 346, the codes look like this:
```
if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  if(BSYMBOLIC)
    string(APPEND LEANC_SHARED_LINKER_FLAGS " -Wl,-Bsymbolic")
    string(APPEND LEANSHARED_LINKER_FLAGS " -Wl,-Bsymbolic")
  endif()
  string(APPEND CMAKE_CXX_FLAGS " -fPIC -ftls-model=initial-exec")
  string(APPEND LEANC_EXTRA_FLAGS " -fPIC")
  string(APPEND LEANSHARED_LINKER_FLAGS " -Wl,-rpath=\\$$ORIGIN/..:\\$$ORIGIN")
  string(APPEND CMAKE_EXE_LINKER_FLAGS " -lleanshared -Wl,-rpath=\\\$ORIGIN/../lib:\\\$ORIGIN/../lib/lean")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
```
add configs for FreeBSD to support -PIC.

the final configs look like this:
```
if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  if(BSYMBOLIC)
    string(APPEND LEANC_SHARED_LINKER_FLAGS " -Wl,-Bsymbolic")
    string(APPEND LEANSHARED_LINKER_FLAGS " -Wl,-Bsymbolic")
  endif()
  string(APPEND CMAKE_CXX_FLAGS " -fPIC -ftls-model=initial-exec")
  string(APPEND LEANC_EXTRA_FLAGS " -fPIC")
  string(APPEND LEANSHARED_LINKER_FLAGS " -Wl,-rpath=\\$$ORIGIN/..:\\$$ORIGIN")
  string(APPEND CMAKE_EXE_LINKER_FLAGS " -lleanshared -Wl,-rpath=\\\$ORIGIN/../lib:\\\$ORIGIN/../lib/lean")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  string(APPEND CMAKE_CXX_FLAGS " -ftls-model=initial-exec")
  string(APPEND LEANSHARED_LINKER_FLAGS " -install_name @rpath/libleanshared.dylib")
  string(APPEND CMAKE_EXE_LINKER_FLAGS " -lleanshared -Wl,-rpath,@executable_path/../lib -Wl,-rpath,@executable_path/../lib/lean")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Emscripten")
  string(APPEND CMAKE_CXX_FLAGS " -fPIC")
  string(APPEND LEANC_EXTRA_FLAGS " -fPIC")
  # We do not use dynamic linking via leanshared for Emscripten to keep things
  # simple. (And we are not interested in `Lake` anyway.) To use dynamic
  # linking, we would probably have to set MAIN_MODULE=2 on `leanshared`,
  # SIDE_MODULE=2 on `lean`, and set CMAKE_SHARED_LIBRARY_SUFFIX to ".js".
  string(APPEND CMAKE_EXE_LINKER_FLAGS " -Wl,--whole-archive -lInit -lLean -lleancpp -lleanrt ${EMSCRIPTEN_SETTINGS} -lnodefs.js -s EXIT_RUNTIME=1 -s MAIN_MODULE=1 -s LINKABLE=1 -s EXPORT_ALL=1")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
  string(APPEND CMAKE_EXE_LINKER_FLAGS " -lleanshared")
elseif(${CMAKE_SYSTEM_NAME} MATCHES "FreeBSD")
  string(APPEND CMAKE_CXX_FLAGS " -fPIC -ftls-model=initial-exec")
  string(APPEND LEANC_EXTRA_FLAGS " -fPIC")
  string(APPEND LEANSHARED_LINKER_FLAGS " -Wl,-rpath=\\$$ORIGIN/..:\\$$ORIGIN")
  string(APPEND CMAKE_EXE_LINKER_FLAGS " -lleanshared -Wl,-rpath=\\\$ORIGIN/../lib:\\\$ORIGIN/../lib/lean")
endif()
```

### 3.2 modify function lean_io_app_path

#### 3.2.1 stage0/src/runtime/io.cpp

there are codes for each platform ,for example windows, APPLE. 

modify the ```else ``` branch for FreeBSD.

and the final codes look like this :

```
extern "C" LEAN_EXPORT obj_res lean_io_app_path(obj_arg) {
#if defined(LEAN_WINDOWS)
    HMODULE hModule = GetModuleHandle(NULL);
    char path[MAX_PATH];
    GetModuleFileName(hModule, path, MAX_PATH);
    std::string pathstr(path);
    // Hack for making sure disk is lower case
    // TODO(Leo): more robust solution
    if (pathstr.size() >= 2 && pathstr[1] == ':') {
        pathstr[0] = tolower(pathstr[0]);
    }
    return io_result_mk_ok(mk_string(pathstr));
#elif defined(__APPLE__)
    char buf1[PATH_MAX];
    char buf2[PATH_MAX];
    uint32_t bufsize = PATH_MAX;
    if (_NSGetExecutablePath(buf1, &bufsize) != 0)
        return io_result_mk_error("failed to locate application");
    if (!realpath(buf1, buf2))
        return io_result_mk_error("failed to resolve symbolic links when locating application");
    return io_result_mk_ok(mk_string(buf2));
#elif defined(LEAN_EMSCRIPTEN)
    // See https://emscripten.org/docs/api_reference/emscripten.h.html#c.EM_ASM_INT
    char* appPath = reinterpret_cast<char*>(EM_ASM_INT({
        if ((typeof process === "undefined") || (process.release.name !== "node")) {
            return 0;
        }

        var lengthBytes = lengthBytesUTF8(__filename)+1;
        var pathOnWasmHeap = _malloc(lengthBytes);
        stringToUTF8(__filename, pathOnWasmHeap, lengthBytes);
        return pathOnWasmHeap;
    }));
    if (appPath == nullptr) {
        return io_result_mk_error("no Lean executable file exists in WASM outside of Node.js");
    }

    object * appPathLean = mk_string(appPath);
    free(appPath);
    return io_result_mk_ok(appPathLean);
#else
    // modified for FreeBSD
    char path[PATH_MAX];
    char dest[PATH_MAX];
    memset(dest, 0, PATH_MAX);
    pid_t pid = getpid();
    snprintf(path, PATH_MAX, "/compat/linux/proc/%d/exe", pid);
    if (readlink(path, dest, PATH_MAX) == -1) {
        return io_result_mk_error("failed to locate application");
    } else {
        return io_result_mk_ok(mk_string(dest));
    }
#endif
}
```

#### 3.2.2 src/runtime/io.cpp

very similar to the function lean_io_app_path in the file stage0/src/runtime/io.cpp

the final codes look like this:
```
extern "C" LEAN_EXPORT obj_res lean_io_app_path(obj_arg) {
#if defined(LEAN_WINDOWS)
    HMODULE hModule = GetModuleHandle(NULL);
    char path[MAX_PATH];
    GetModuleFileName(hModule, path, MAX_PATH);
    std::string pathstr(path);
    // Hack for making sure disk is lower case
    // TODO(Leo): more robust solution
    if (pathstr.size() >= 2 && pathstr[1] == ':') {
        pathstr[0] = tolower(pathstr[0]);
    }
    return io_result_mk_ok(mk_string(pathstr));
#elif defined(__APPLE__)
    char buf1[PATH_MAX];
    char buf2[PATH_MAX];
    uint32_t bufsize = PATH_MAX;
    if (_NSGetExecutablePath(buf1, &bufsize) != 0)
        return io_result_mk_error("failed to locate application");
    if (!realpath(buf1, buf2))
        return io_result_mk_error("failed to resolve symbolic links when locating application");
    return io_result_mk_ok(mk_string(buf2));
#elif defined(LEAN_EMSCRIPTEN)
    // See https://emscripten.org/docs/api_reference/emscripten.h.html#c.EM_ASM_INT
    char* appPath = reinterpret_cast<char*>(EM_ASM_INT({
        if ((typeof process === "undefined") || (process.release.name !== "node")) {
            return 0;
        }

        var lengthBytes = lengthBytesUTF8(__filename)+1;
        var pathOnWasmHeap = _malloc(lengthBytes);
        stringToUTF8(__filename, pathOnWasmHeap, lengthBytes);
        return pathOnWasmHeap;
    }));
    if (appPath == nullptr) {
        return io_result_mk_error("no Lean executable file exists in WASM outside of Node.js");
    }

    object * appPathLean = mk_string(appPath);
    free(appPath);
    return io_result_mk_ok(appPathLean);
#else
    // modified for FreeBSD
    char path[PATH_MAX];
    char dest[PATH_MAX];
    memset(dest, 0, PATH_MAX);
    pid_t pid = getpid();
    snprintf(path, PATH_MAX, "/compat/linux/proc/%d/exe", pid);
    if (readlink(path, dest, PATH_MAX) == -1) {
        return io_result_mk_error("failed to locate application");
    } else {
        return io_result_mk_ok(mk_string(dest));
    }
#endif
}
```

### 3.3 modify function is_within_stack_guard

#### 3.3.1  stage0/src/runtime/stack_overflow.cpp

the ```else``` branch for Linux, not for FreeBSD

sorry I don't know the correct implementation on FreeBSD.

the final codes look like these :

```
bool is_within_stack_guard(void * addr) {
    char * stackaddr;
#ifdef __APPLE__
    stackaddr = static_cast<char *>(pthread_get_stackaddr_np(pthread_self())) - pthread_get_stacksize_np(pthread_self());
#else
	return false;
	/*
    pthread_attr_t attr;
    if (pthread_attr_init(&attr) != 0) return false;
    pthread_getattr_np(pthread_self(), &attr);
    size_t stacksize;
    pthread_attr_getstack(&attr, reinterpret_cast<void **>(&stackaddr), &stacksize);
    pthread_attr_destroy(&attr);
	*/
#endif
    // close enough; the actual guard might be bigger, but it's unlikely a Lean function will have stack frames that big
    size_t guardsize = static_cast<size_t>(sysconf(_SC_PAGESIZE));
    // the stack guard is *below* the stack
    return stackaddr - guardsize <= addr && addr < stackaddr;
}

```

#### 3.3.2 src/runtime/stack_overflow.cpp

very similar to the is_within_stack_guard in the file ```stage0/src/runtime/stack_overflow.cpp```
the final codes look like these :
```
bool is_within_stack_guard(void * addr) {
    char * stackaddr;
#ifdef __APPLE__
    stackaddr = static_cast<char *>(pthread_get_stackaddr_np(pthread_self())) - pthread_get_stacksize_np(pthread_self());
#else
	return false;
	/*
    pthread_attr_t attr;
    if (pthread_attr_init(&attr) != 0) return false;
    pthread_getattr_np(pthread_self(), &attr);
    size_t stacksize;
    pthread_attr_getstack(&attr, reinterpret_cast<void **>(&stackaddr), &stacksize);
    pthread_attr_destroy(&attr);
	*/
#endif
    // close enough; the actual guard might be bigger, but it's unlikely a Lean function will have stack frames that big
    size_t guardsize = static_cast<size_t>(sysconf(_SC_PAGESIZE));
    // the stack guard is *below* the stack
    return stackaddr - guardsize <= addr && addr < stackaddr;
}
```

## 4 enabl linux compatibility

```
mount -t procfs proc /proc

mkdir -p /compat/linux/proc

mount -t linprocfs /dev/null /compat/linux/proc
```

## 5 set gmake to be the default make

```
mv /usr/bin/make  /usr/bin/make_original

ln -s /usr/local/bin/gmake   /usr/bin/make
```
## 5. execute cmake 

```
mkdir build
cd build
cmake ..
```

## 6. build 

execute this command 
```
# in the dir build
make 
```
or 
```
# in the dir build
make -j4
```
or
```
#print some stage
make VERBOSE=1 stage1
```

## 7 build stage2 and stage3

```
# in the dir build
make stage2 
make stage3
```
# test


reference :
```
https://lean-lang.org/functional_programming_in_lean/hello-world/cat.html
```