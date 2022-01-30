# CUDA/NVCC Bug (maybe?) [![Windows](https://github.com/Ahdhn/CUDATemplate/actions/workflows/Windows.yml/badge.svg)](https://github.com/Ahdhn/CUDATemplate/actions/workflows/Windows.yml) [![Ubuntu](https://github.com/Ahdhn/CUDATemplate/actions/workflows/Ubuntu.yml/badge.svg)](https://github.com/Ahdhn/CUDATemplate/actions/workflows/Ubuntu.yml)
[Godbolt](https://www.godbolt.org/z/8MKEWdaGh) suggests that the code described in `enable_if/enable_if.cu` is valid. It seems like the compiler is unable to overload the function with `std::enable_if` and a `__device__` lambda in this case as it suggests the identifier is undeclared.


## Build 
```
mkdir build
cd build 
cmake ..
```

Depending on the system, this will generate either a `.sln` project on Windows or a `make` file for a Linux system. 

## Fix
[`__device__` lambdas in CUDA are atrocious. Avoid them when possible. 😠](https://github.com/NVIDIA/thrust/issues/1610#issuecomment-1025053905)

```cpp
template <my_enum_t t,
          typename vector_t,
          typename operator_t>
std::enable_if_t<t==my_enum_t::X> execute(vector_t& v, operator_t op) { ... }
```

## Error

### Ubuntu
The full Ubuntu configuration is found in [Ubuntu.yml](https://github.com/neoblizz/enable_if_bug/blob/master/.github/workflows/Ubuntu.yml) file.
```
Run make -j
[ 10%] Building CXX object _deps/googletest-build/googletest/CMakeFiles/gtest.dir/src/gtest-all.cc.o
[ 20%] Linking CXX static library ../../../lib/libgtest.a
[ 20%] Built target gtest
[ 30%] Building CXX object _deps/googletest-build/googletest/CMakeFiles/gtest_main.dir/src/gtest_main.cc.o
[ 40%] Building CXX object _deps/googletest-build/googlemock/CMakeFiles/gmock.dir/src/gmock-all.cc.o
[ 50%] Linking CXX static library ../../../lib/libgtest_main.a
[ 50%] Built target gtest_main
[ 60%] Building CUDA object CMakeFiles/enable_if.dir/enable_if/enable_if.cu.o
nvcc warning : The 'compute_35', 'compute_37', 'compute_50', 'sm_35', 'sm_37' and 'sm_50' architectures are deprecated, and may be removed in a future release (Use -Wno-deprecated-gpu-targets to suppress warning).
[ 70%] Linking CXX static library ../../../lib/libgmock.a
[ 70%] Built target gmock
[ 80%] Building CXX object _deps/googletest-build/googlemock/CMakeFiles/gmock_main.dir/src/gmock_main.cc.o
[ 90%] Linking CXX static library ../../../lib/libgmock_main.a
[ 90%] Built target gmock_main
/home/runner/work/enable_if_bug/enable_if_bug/enable_if/enable_if.cu: In function ‘void execute(vector_t&, operator_t)’:
/home/runner/work/enable_if_bug/enable_if_bug/enable_if/enable_if.cu:18:171: error: ‘__T288’ was not declared in this scope
   18 |   thrust::for_each(             // for each
      |                                                                                                                                                                           ^     
/home/runner/work/enable_if_bug/enable_if_bug/enable_if/enable_if.cu: In function ‘void execute(vector_t&, operator_t)’:
/home/runner/work/enable_if_bug/enable_if_bug/enable_if/enable_if.cu:35:240: error: ‘__T289’ was not declared in this scope
   35 |   thrust::for_each(                              // for each
      |                                                                                                                                                                                                                                                ^     
make[2]: *** [CMakeFiles/enable_if.dir/build.make:76: CMakeFiles/enable_if.dir/enable_if/enable_if.cu.o] Error 1
make[1]: *** [CMakeFiles/Makefile2:137: CMakeFiles/enable_if.dir/all] Error 2
make: *** [Makefile:146: all] Error 2
```

### Windows
The full Ubuntu configuration is found in [Windows.yml](https://github.com/neoblizz/enable_if_bug/blob/master/.github/workflows/Windows.yml) file.

```
D:\a\enable_if_bug\enable_if_bug\enable_if\enable_if.cu(18): error C2065: '__T288': undeclared identifier [D:\a\enable_if_bug\enable_if_bug\build\enable_if.vcxproj]
  D:\a\enable_if_bug\enable_if_bug\enable_if\enable_if.cu(58): note: see reference to function template instantiation 'void execute<X,thrust::device_vector<int,thrust::device_allocator<T>>,__nv_dl_wrapper_t<__nv_dl_tag<int (__cdecl *)(int,char **),&int main(int,char **),1>>,0>(vector_t &,operator_t)' being compiled
          with
          [
              T=int,
              vector_t=thrust::device_vector<int,thrust::device_allocator<int>>,
              operator_t=__nv_dl_wrapper_t<__nv_dl_tag<int (__cdecl *)(int,char **),&int main(int,char **),1>>
          ]
D:\a\enable_if_bug\enable_if_bug\enable_if\enable_if.cu(18): error C2440: 'specialization': cannot convert from 'overloaded-function' to 'void (__cdecl *)(vector_t &,operator_t)' [D:\a\enable_if_bug\enable_if_bug\build\enable_if.vcxproj]
          with
          [
              vector_t=thrust::device_vector<int,thrust::device_allocator<int>>,
              operator_t=__nv_dl_wrapper_t<__nv_dl_tag<int (__cdecl *)(int,char **),&int main(int,char **),1>>
          ]
  D:\a\enable_if_bug\enable_if_bug\enable_if\enable_if.cu(18): note: None of the functions with this name in scope match the target type
D:\a\enable_if_bug\enable_if_bug\enable_if\enable_if.cu(18): error C2065: '__T288': undeclared identifier [D:\a\enable_if_bug\enable_if_bug\build\enable_if.vcxproj]
D:\a\enable_if_bug\enable_if_bug\enable_if\enable_if.cu(35): error C2065: '__T289': undeclared identifier [D:\a\enable_if_bug\enable_if_bug\build\enable_if.vcxproj]
  D:\a\enable_if_bug\enable_if_bug\enable_if\enable_if.cu(59): note: see reference to function template instantiation 'void execute<Y,thrust::device_vector<int,thrust::device_allocator<T>>,__nv_dl_wrapper_t<__nv_dl_tag<int (__cdecl *)(int,char **),&int main(int,char **),2>>,0>(vector_t &,operator_t)' being compiled
          with
          [
              T=int,
              vector_t=thrust::device_vector<int,thrust::device_allocator<int>>,
              operator_t=__nv_dl_wrapper_t<__nv_dl_tag<int (__cdecl *)(int,char **),&int main(int,char **),2>>
          ]
D:\a\enable_if_bug\enable_if_bug\enable_if\enable_if.cu(35): error C2440: 'specialization': cannot convert from 'overloaded-function' to 'void (__cdecl *)(vector_t &,operator_t)' [D:\a\enable_if_bug\enable_if_bug\build\enable_if.vcxproj]
          with
          [
              vector_t=thrust::device_vector<int,thrust::device_allocator<int>>,
              operator_t=__nv_dl_wrapper_t<__nv_dl_tag<int (__cdecl *)(int,char **),&int main(int,char **),2>>
          ]
  D:\a\enable_if_bug\enable_if_bug\enable_if\enable_if.cu(35): note: None of the functions with this name in scope match the target type
D:\a\enable_if_bug\enable_if_bug\enable_if\enable_if.cu(35): error C2065: '__T289': undeclared identifier [D:\a\enable_if_bug\enable_if_bug\build\enable_if.vcxproj]
```
