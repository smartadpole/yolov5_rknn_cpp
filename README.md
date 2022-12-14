# Yolo-v5 demo

# 导出rknn模型

## 导出 onnx 模型
   yolov5工程需要使用pytorch 1.8.0 或 1.9.0 版本才能正常导出。
   ```sh
   ## 使用onnx-simplifier工具优化yolov5的onnx模型
   pip install onnx-simplifier
   python -m onnxsim yolov5s.onnx  yolov5s.onnx
   ```

### 运行python推理
### 使用rk预训练模型

由于官方的yolov5s模型中包含了Slice层/Swish层/大kernel_size的MaxPooling层，NPU执行效率不高。我们建议开发者使用NPU友好的算子替换官网的结构，下面给出两个参考网络结构：
训练和导出onnx模型过程请参考请参考 https://github.com/EASY-EAI/yolov5   

转换rknn模型的步骤如下：

```sh
cd convert_rknn_demo/yolov5/
python onnx2rknn.py
```

| 模型 | 位置 | 
| --- | --- |
| yolov5s-rubby | sftp://192.168.50.200/data/ICE/model_Abby/A6.0.1/rknn/best.rknn |
| yolov5s-coco | sftp://192.168.50.200/data/MODEL/coco/yolov5s-640-640.rknn |

1. 改进结构1

```txt
a. 将Focus层改成Conv层
b. 将Swish激活函数改成Relu激活函数
```

2. 改进结构2

```txt
a. 将Focus层改成Conv层
b. 将Swish激活函数改成Relu激活函数
c. 将大kernel_size的MaxPooling改成3x3 MaxPooling Stack结构
```

## 注意事项：

1. 使用rknn-toolkit2版本大于等于1.1.2。
2. 切换成自己训练的模型时，请注意对齐anchor等后处理参数，否则会导致后处理解析出错。
3. 官网和rk预训练模型都是检测80类的目标，如果自己训练的模型,需要更改include/postprocess.h中的OBJ_CLASS_NUM以及NMS_THRESH,BOX_THRESH后处理参数。
4. 测试代码导出模型的时候指定了输出节点['378', '439', '500']，分别为原模型的第2、3、4输出节点的去掉 三个Reshape 后面的层（不包含Reshape层），
   对应输出的shape是[1,255,80,80],[1,255,40,40],[1,255,20,20]。
   对于自己训练的模型输出节点的顺序和shape的要求必须是[1,？,80,80]，[1,？,40,40]，[1,？,20,20]，C代码后处理才能正确处理。
5. 默认导出rk356x的rknn模型,导出rk3588模型需要修改rknn.config的target_platform参数为"rk3588".
6. demo需要librga.so的支持,编译使用请参考https://github.com/rockchip-linux/linux-rga

## Android Demo

### 编译

根据指定平台修改 `build-android_<TARGET_PLATFORM>.sh`中的Android NDK的路径 `ANDROID_NDK_PATH`，<TARGET_PLATFORM>可以是RK356X或RK3588 例如修改成：

```sh
ANDROID_NDK_PATH=~/opt/tool_chain/android-ndk-r17
```

然后执行：

```sh
./build-android_<TARGET_PLATFORM>.sh
```

### 推送执行文件到板子

连接板子的usb口到PC,将整个demo目录到 `/data`:

```sh
adb root
adb remount
adb push install/rknn_yolov5_demo /data/
```

### 运行

```sh
adb shell
cd /data/rknn_yolov5_demo/

export LD_LIBRARY_PATH=./lib
./rknn_yolov5_demo model/<TARGET_PLATFORM>/yolov5s-640-640.rknn model/bus.jpg
```

## Aarch64 Linux Demo

### 编译

根据指定平台修改 `build-linux_<TARGET_PLATFORM>.sh`中的交叉编译器所在目录的路径 `TOOL_CHAIN`，例如修改成

```sh
export TOOL_CHAIN=~/opt/tool_chain/gcc-9.3.0-x86_64_aarch64-linux-gnu/host
```

然后执行：

```sh
./build-linux_<TARGET_PLATFORM>.sh
```

### 推送执行文件到板子

连接板子的usb口到PC,将整个demo目录到 `/userdata`:

```sh
adb push install/rknn_yolov5_demo_Linux /userdata/
```

### 运行

```sh
adb shell
cd /userdata/rknn_yolov5_demo_Linux/

export LD_LIBRARY_PATH=./lib
./rknn_yolov5_demo model/<TARGET_PLATFORM>/yolov5s-640-640.rknn model/bus.jpg
```

Note: Try searching the location of librga.so and add it to LD_LIBRARY_PATH if the librga.so is not found on the lib folder.
Using the following commnands to add to LD_LIBRARY_PATH.

```sh
export LD_LIBRARY_PATH=./lib:<LOCATION_LIBRGA.SO>
```
