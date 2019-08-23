# build_tensorflow_windows
tensorflow c++ windows

tensorflow1.12-CUDA编译成功。

按照官方的<https://www.tensorflow.org/install/source_windows>的教程安装所需的软件并下载对应版本的代码。tensorflow1.12-CUDA版本建议使用bazel15。

在执行`python configure.py`之前，将patches文件夹放到tensorflow源代码文件夹所在目录中：
> your_folder
>  - patches
>  - tensorflow

进入tensorflow源码根目录执行（patches中的补丁文件来源于https://github.com/guikarist/tensorflow-windows-build-script）：
```
git apply --ignore-space-change --ignore-white "..\patches\eigen.1.12.0.patch"
git apply --ignore-space-change --ignore-white "..\patches\cpp_symbol.1.12.0.patch"
```

再执行以下命令：（tf_exported_symbols_msvc.lds里是需要导出的函数符号）
```
copy ..\patches\eigen_half.patch third_party\
copy ..\patches\tf_exported_symbols_msvc.lds tensorflow\
```

然后进行配置，配置的内容按照https://www.tensorflow.org/install/source_windows 中的介绍，如果要支持CUDA，就要选择对应的CUDA位置和CUDNN位置：
```
python configure.py
```

编译libtensorflow_cc.so，编译结束后会在tensorflow/bazel-bin生成libtensorflow_cc.so和liblibtensorflow_framework.so.ifso：
```
bazel build --config=opt --config=cuda --define=no_tensorflow_py_deps=true --copt=-nvcc_options=disable-warnings //tensorflow:libtensorflow_cc.so --verbose_failures
```

编译头文件：
```
bazel build --config=opt --config=cuda --define=no_tensorflow_py_deps=true --copt=-nvcc_options=disable-warnings //tensorflow:install_headers --verbose_failures
```

在创建VS工程后, 设置项目属性中->Linker->Input->Additional Dependencies为liblibtensorflow_cc.so.ifso, 

设置C/C++->General->Additilonal Include Directories为: bazel-genfiles\tensorflow\include
bazel-genfiles\tensorflow\include\external\protobuf_archive\src
bazel-genfiles\tensorflow\include\external\com_google_absl (可以将这些文件夹底下的内容同一放到一个文件夹中.)

再将libtensorflow_cc.so文件改名为tensorflow_cc.dll, 放入到x64/Release目录下.

**注意， 如果在运行代码的时候，遇到LNK2001    unresolved external symbol: XXX@tensorflow@XXX的错误，要把这个符号加入到tf_exported_symbols_msvc.lds中，并执行以下命令重新编译libtensorflow_cc.so，并用新的libtensorflow_cc.so和liblibtensorflow_framework.so.ifso替换旧的，不需要重新编译头文件**：

```
bazel clean --expunge

python configure.py

bazel build --config=opt --config=cuda --define=no_tensorflow_py_deps=true --copt=-nvcc_options=disable-warnings //tensorflow:libtensorflow_cc.so --verbose_failures
```

