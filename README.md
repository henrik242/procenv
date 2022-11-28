-----------------------------------------------------------------------------------------------------------------------------------------------------------
Cross-Platform, Foreign Process Information Explorer API
-----------------------------------------------------------------------------------------------------------------------------------------------------------

`std::vector<PROCID> proc_id_enum();` returns a C++ std::vector of process identifiers for every running process in the current user session. On some platforms the process identifier of zero is included by default, while on other platforms it is omitted, despite it always pointing to a valid and existing process id, one which represents the process swapper. For the sake of consistency, the vector returned by this function will prepend a process identifier of zero to the vector depending on whether the underlying platform-specific API happened to omit that process identifier. On Unix-likes, a process identifier of one represents the system initialization process, which is responsible for starting up and shutting down your machine. This function only includes thread group identifiers in the vector, which are the process identifiers of a process's main thread. Kernel thread identifiers are omitted. On Solaris and Illumos, this function requires the calling process being run as root, otherwise, the function will fail. On failure, std::vector::empty() will be true.

-----------------------------------------------------------------------------------------------------------------------------------------------------------

`bool proc_id_exists(PROCID proc_id);` returns true if the process identifier in proc_id represents a process which is running in the current user session. This function will iterate over all processes running in the current user session until one is found which is represented by the process identifier passed to the proc_id argument, and if one is found, true is returned; otherwise, false is returned because the specified process id did not match an existing process in the current user session. This function only checks for equality of existing thread group identifiers, which are the process identifiers of a process's main thread. Kernel thread identifiers are omitted from the check. To check for the existence of kernel threads, consider using a different API. On Solaris and Illumos, this function requires the calling process being run as root, otherwise, the function will fail. On failure, the return value will be false, and errno will be set to a non-zero value.

-----------------------------------------------------------------------------------------------------------------------------------------------------------
`bool proc_id_suspend(PROCID proc_id);` suspends the process represented by the process identifier in the proc_id argument. Returns true on success; returns false on failure. Useful for preventing race conditions when reading foreign process information. Also good for debugging applications.

-----------------------------------------------------------------------------------------------------------------------------------------------------------

`bool proc_id_resume(PROCID proc_id);` resumes a previously suspended process represented by the process identifier in the proc_id argument. Returns true on success; returns false on failure. Useful for preventing race conditions when reading foreign process information. Also good for debugging applications.

-----------------------------------------------------------------------------------------------------------------------------------------------------------

`bool proc_id_kill(PROCID proc_id);` kills the process represented by the process identifier in the proc_id argument. When a process is killed, it is forced to be closed by the calling process. It can be dangerous to accidentally kill the wrong process. Returns true on success; returns false on failure.

-----------------------------------------------------------------------------------------------------------------------------------------------------------

`std::vector<PROCID> parent_proc_id_from_proc_id(PROCID proc_id);` on success, the function returns a C++ std::vector holding a single process identifier which represents the parent process of the process identifier provided in the proc_id argument. std::vector::empty() returns true if the function fails, and errno will be set to a non-zero value. Every process is guaranteed to have a parent, even if the function fails to find one. A process identifier of zero has a parent; its parent is itself. On Solaris and Illumos, this function requires the calling process being run as root, otherwise, the function will fail.

-----------------------------------------------------------------------------------------------------------------------------------------------------------

`std::vector<PROCID> proc_id_from_parent_proc_id(PROCID parent_proc_id);` on success, the function returns a C++ std::vector holding all process identifiers which represent the children of the given parent process identifier in the parent_proc_id argument. std::vector::empty() returns true if there are no children, or if the function has failed. To determine whether an error occurred, check against the value of errno after a call to the function. errno will be non-zero if an error occurred. On Solaris and Illumos, this function requires the calling process being run as root, otherwise, the function will fail.

-----------------------------------------------------------------------------------------------------------------------------------------------------------

`std::string exe_from_proc_id(PROCID proc_id);` returns the executable pathname associated with the given process identifier in the proc_id argument. std::string::empty() is true if the function has failed. The return value will be an absolute pathname with all symbollic links resolved. The pathname will be normalized, meaning there will be no double slashes, no dot, and no dot-dot in the returned pathname. The returned pathname is limited to MAX_PATH characters on Windows, and on Unix-likes, the returned pathname is limited to PATH_MAX characters. PATH_MAX is implementation-defined by the platform. On OpenBSD, the platform relies on retrieving argv[0] from the assocaited process's command line arguments, to resolve the executable pathname, because OpenBSD does not have an official or dedicated API for such. The argv[0] suffix is appended to the $PWD environment variable of the associated process as the prefix, when the argv[0] suffix is a relative pathname, with the current working directory of the assocaited process as the fallback prefix. On failure, the executable pathname will have a wrong ino_t and/or dev_t value, which will lead to std::string::empty() returning true as a failure case.

-----------------------------------------------------------------------------------------------------------------------------------------------------------

`std::string cwd_from_proc_id(PROCID proc_id);` returns the current working directory associated with the given process identifier in the proc_id argument. std::string::empty() is true if the function has failed. The return value will be an absolute pathname with all symbollic links resolved. The pathname will be normalized, meaning there will be no double slashes, no dot, and no dot-dot in the returned pathname. The returned pathname is limited to MAX_PATH characters on Windows, and on Unix-likes, the returned pathname is limited to PATH_MAX characters. PATH_MAX is implementation-defined by the platform. The pathname returned will not contain a trailing slash on all platforms. On Windows, the underlying API has a trailing slash, which this function removes. On Windows, the calling process must be spawned from an executable whose architecture is the same as the executable which spawned the target process identifier, otherwise, the function will fail.

-----------------------------------------------------------------------------------------------------------------------------------------------------------

`std::vector<std::string> cmdline_from_proc_id(PROCID proc_id);` returns a C++ std::vector of the full list of command line arguments associated with the given process represented by the process identifier in the proc_id argument. On most platforms this value can be modified by the process associated with it, as well as its initial value by the parent process which spawned it. The exception to this would be on macOS, the C++ std::vector returned will be the initial command line argument values, and will not return any modifications that were made to the command line arguments by the process represented by the process intentifier passed to the proc_id argument of this function. std::vector::empty() returns true if the function failed, and errno will be set to a non-zero value. On Windows, the calling process must be spawned from an executable whose architecture is the same as the executable which spawned the target process identifier, otherwise, the function will fail. On Solaris and Illumos, this function requires the calling process being run as root, otherwise, the function will fail.

-----------------------------------------------------------------------------------------------------------------------------------------------------------

`std::vector<std::string> environ_from_proc_id(PROCID proc_id);` returns a C++ std::vector of the full list of environment variables associated with the given process represented by the process identifier in the proc_id argument. On most platforms this value can be modified by the process associated with it, as well as its initial value by the parent process which spawned it. The exception to this would be on macOS, the C++ std::vector returned will be the initial environment variables passed to the executable, and will not return any modifications that were made to the environment variables by the process represented by the process intentifier passed to the proc_id argument of this function. std::vector::empty() returns true if the function failed, and errno will be set to a non-zero value. On Windows, the calling process must be spawned from an executable whose architecture is the same as the executable which spawned the target process identifier, otherwise, the function will fail. On Solaris and Illumos, this function requires the calling process being run as root, otherwise, the function will fail.

-----------------------------------------------------------------------------------------------------------------------------------------------------------

`std::string envvar_value_from_proc_id(PROCID proc_id, std::string name);` returns the string value of an environment variable of the given name in the name argument belonging to the process whose process identifier is represented by the proc_id argument. std::string::empty() returns true if the variable does not exist or if the value it has is empty. To find out if a environment variable exists within the scope of a foreign process environment block, call envvar_exists_from_proc_id() instead, as there are times where a variable does exist within the environment block but has no value associated with it. On Solaris and Illumos, this function requires the calling process being run as root, otherwise, the function will fail. On failure, the std::string::empty() will be true, and errno will be set to a non-zero value.

-----------------------------------------------------------------------------------------------------------------------------------------------------------

`bool envvar_exists_from_proc_id(PROCID proc_id, std::string name);` returns whether an environment variable of the given name in the name argument belongs to the process whose process identifier is represented by the proc_id argument. Returns true if the variable was found, and returns false otherwise. On Solaris and Illumos, this function requires the calling process being run as root, otherwise, the function will fail. On failure, the return value will be false, and errno will be set to a non-zero value.

-----------------------------------------------------------------------------------------------------------------------------------------------------------
