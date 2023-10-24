
LLAMA_VERSION?=

CMAKE_ARGS?=
BUILD_TYPE?=

# If build type is cublas, then we set -DLLAMA_CUBLAS=ON to CMAKE_ARGS automatically
ifeq ($(BUILD_TYPE),cublas)
	CMAKE_ARGS+=-DLLAMA_CUBLAS=ON
# If build type is openblas then we set -DLLAMA_BLAS=ON -DLLAMA_BLAS_VENDOR=OpenBLAS
# to CMAKE_ARGS automatically
else ifeq ($(BUILD_TYPE),openblas)
	CMAKE_ARGS+=-DLLAMA_BLAS=ON -DLLAMA_BLAS_VENDOR=OpenBLAS
# If build type is clblast (openCL) we set -DLLAMA_CLBLAST=ON -DCLBlast_DIR=/some/path
else ifeq ($(BUILD_TYPE),clblast)
	CMAKE_ARGS+=-DLLAMA_CLBLAST=ON -DCLBlast_DIR=/some/path
# If it's hipblas we do have also to set CC=/opt/rocm/llvm/bin/clang CXX=/opt/rocm/llvm/bin/clang++ 
else ifeq ($(BUILD_TYPE),hipblas)
	CMAKE_ARGS+=-DLLAMA_HIPBLAS=ON
endif

llama.cpp:
	git clone --recurse-submodules https://github.com/ggerganov/llama.cpp llama.cpp
	cd llama.cpp && git checkout -b build $(LLAMA_VERSION) && git submodule update --init --recursive --depth 1

llama.cpp/examples/grpc-server:
	mkdir -p llama.cpp/examples/grpc-server
	cp -r $(abspath ./)/CMakeLists.txt llama.cpp/examples/grpc-server/
	cp -r $(abspath ./)/grpc-server.cpp llama.cpp/examples/grpc-server/
	echo "add_subdirectory(grpc-server)" >> llama.cpp/examples/CMakeLists.txt

rebuild:
	cp -rfv $(abspath ./)/CMakeLists.txt llama.cpp/examples/grpc-server/
	cp -rfv $(abspath ./)/grpc-server.cpp llama.cpp/examples/grpc-server/
	rm -rf grpc-server
	$(MAKE) grpc-server

clean:
	rm -rf llama.cpp
	rm -rf grpc-server

grpc-server: llama.cpp llama.cpp/examples/grpc-server
	cd llama.cpp && mkdir -p build && cd build && cmake .. $(CMAKE_ARGS) && cmake --build . --config Release
	cp llama.cpp/build/bin/grpc-server .