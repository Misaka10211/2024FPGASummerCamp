# 2024FPGASummerCamp
**参与者：冯奕逍**  
**lab1**  
*官方链接*  
https://github.com/AEEE-SummerSchool/FPGA_Camp24/tree/main/Lab1_sobel  
*功能描述*  
调整了handcode相关代码，支持处理最大1280x720像素的图片  
调整了vision library中sobel算子，支持处理最大512x512像素的图片  
  
*使用板卡* 
KV260  
  
*环境配置*  
（Windows 10系统）  
Vision library  
MinGW  
CMAKE 3.30.0  
OpenCV 4.4.0  教程在https://support.xilinx.com/s/article/000035890?language=en_US  
Vitis 2023.2  
Vivado 2023.2  

*分析*    
由handcoded生成的bit文件可以传入1280*720的图片，并正常处理；  
→ 在进行试验时，修改了
在用大图片测试到visionlib时出现问题，导致part3的第14段代码跑不通；  
→ 需要下载vision lib sobelfilter项目文件，在xf_sobel_config.h中修改WIDTH和HEIGHT为所需的大小（本实验中设为了512x512）  
→ 为通过Testbench, 需要在hls_standalone.tcl中修改测试用的图片  
csim_design -ldflags "-L ${OPENCV_LIB} ${OPENCV_LIB_REF}" -argv "${XF_PROJ_ROOT}/data/512x512.png"  
cosim_design -ldflags "-L ${OPENCV_LIB} ${OPENCV_LIB_REF}" -argv "${XF_PROJ_ROOT}/data/512x512.png"  
→ 随后按正常流程重新生成bit和hwh    

*避坑建议*:satisfied:  
1、在解压OpenCV4.4.0文件时，最好放在“干净的路径”下，不能带有中文字符或空格，省心的做法是放在根目录  
2、确保下载了opencv和opencv_contrib两个压缩包，把解压得到的两个文件夹改名（删除版本号），把它们放到新的文件夹中  
https://github.com/opencv/opencv/archive/refs/tags/4.4.0.zip  
https://github.com/opencv/opencv_contrib/archive/refs/tags/4.4.0.zip  
3、OpenCV安装过程中OPENCV_EXTRA_MODULES_PATH需要选择opencv_contrib/modules的路径，要用“ / ”，而windows系统复制的路径是“ \ ”。记得改！不想改的话可以点击输入框右侧，手动在弹出窗口中选择路径  
4、记得按教程设置环境变量：我的电脑-属性-高级系统设置-环境变量  
5、不建议直接下载vision library压缩包并在本地解压，可能会出问题。建议用git bash或github desktop客户端进行下载  
6、用KV260实验时，存在register map丢失的问题，xf_sobel_accel.cpp35-37行 删除“ boundle=control ”  
7、如果vitis报错，提示库找不到，建议检查hls_standalone.tcl中环境变量是否设置正确  
8、如果使用kv260进行试验，需要在hls_standalone.tcl中修改部件编号：修改为：set SOLUTION_PART "xck26-sfvc784-2lv-c"  
9、vitis中（tcl脚本）选择的部件编号必须和vivado创建项目时选择的部件编号一致，否则会找不到sobel IP    

*实验日志*:innocent:  
从git下载Vitis_Libraries-master，解压至G:\Download  
把G:\Download\Vitis_Libraries-master\vision\L1\examples\sobelfilter\build路径下的xf_config_params.h复制到G:\Download\Vitis_Libraries-master\vision\L1\examples\sobelfilter  
严格按照visoon_library_win.md操作  
关键路径：  
<path to vitis libraries> = G:/Download/Vitis_Libraries-master  
<path to opencv install> = H:/opencv440/opencv/build/install  
*C Synthesis sources*
-I G:/Download/Vitis_Libraries-master/vision/L1/examples/sobelfilter/config -I G:/Download/Vitis_Libraries-master/vision/L1/include -I ./ -D__SDSVHLS__ -std=c++14

*Testbench sources，C/RTL Cosimulation* ldflags  
这里的选项有问题，下面的选项组合是针对remap的，不适用于sobelfilter
-L <path to opencv install>/x64/mingw/lib -lopencv_imgcodecs440 -lopencv_imgproc440 -lopencv_calib3d440 -lopencv_core440 -lopencv_highgui440 -lopencv_flann440 -lopencv_features2d440
参考vision_library_linux.md，用到了以下命令（opencv3.4.11）：  
set OPENCV_LIB_REF                       "-lopencv_imgcodecs3411 -lopencv_imgproc3411                  -lopencv_core3411 -lopencv_highgui3411 -lopencv_flann3411 -lopencv_features2d3411"  
可以发现：-lopencv_calib3d440对sobelfilter而言是多余的（？）
-L <path to opencv install>/x64/mingw/lib -lopencv_imgcodecs440 -lopencv_imgproc440 -lopencv_core440 -lopencv_highgui440 -lopencv_flann440 -lopencv_features2d440
  
#使用tcl脚本：vitis_hls -f run_hls_standalone.tcl  
为正常使用kv260板，修改为：set SOLUTION_PART "xck26-sfvc784-2lv-c"  

IP的col、row接口有问题  
》xf_sobel_accel.cpp35-36行取消注释  
pragma HLS INTERFACE s_axilite port=rows     bundle=control  
pragma HLS INTERFACE s_axilite port=cols     bundle=control  
    
在环境变量中设置下面引号内的路径，同时修改run_hls_standalone.tcl脚本：  
原始：  
set XF_PROJ_ROOT "C:/Users/aup/workspace/Vitis_Libraries-main/vision"   
set OPENCV_INCLUDE "C:/Xilinx/opencv/build/install/include"   
set OPENCV_LIB "C:/Xilinx/opencv/build/install/x64/mingw/lib"  
修改后：  
set XF_PROJ_ROOT "G:/Download/Vitis_Libraries-master/vision"  
set OPENCV_INCLUDE "H:/opencv440/opencv/build/install/include"  
set OPENCV_LIB "H:/opencv440/opencv/build/install/x64/mingw/lib"   
  
为处理更大的图片（以512x512为例），xf_sobel_config.h中把define的WIDTH和HEIGHT设为512  
把run_hls_standalone.tcl中的128x128.png改为512x512.png，这里tcl为tb文件传参，如果不改会报错  
  
打开vitis2023.2 在Terminal选项中New Terminal  
（由于此前打开过vitis GUI，Terminal路径直接为G:\Download\Vitis_Libraries-master\vision\L1\examples\sobelfilter）  
在Terminal运行vitis_hls -f run_hls_standalone.tcl  
生成文件路径为：  
G:\Download\Vitis_Libraries-master\vision\L1\examples\sobelfilter\hls_example\sol1\impl  

按教程在vivado中连线，generate bit
G:\Download\xup_embedded_system_design_flow-main\labs\sobel512\sobel512.runs\impl_1导出.bit,   
G:\Download\xup_embedded_system_design_flow-main\labs\sobel512\sobel512.gen\sources_1\bd\design_1\hw_handoff导出.hwh  

上传至远程实验室  
registermap读取错误：少了很多东西  
》》xf_sobel_accel.cpp35-37行 删除“ boundle=control ”  
pragma HLS INTERFACE s_axilite port=rows  
pragma HLS INTERFACE s_axilite port=cols  
HLS INTERFACE s_axilite port=return  

跑出来的图很怪  
》》查看发现解压的vision lib文件有问题，最近使用的一直是2019版sobel_accelerator.cpp  
》》改用github desktop下载, git bash会很慢    

vision lib的环境变量改为 G:\Download\Vitis_Libraries\vision  
重复正常流程，测试成功!  
😋  
  
#尝试Lab1 advance#  
预计使用resize、sobelfilter  
L2/exampls/resize/config/xf_config_params.h中把  
WIDTH, HEIGHT是输入图片的尺寸，改为1024  
NEWWIDTH, NEWHEIGHT是输出图片的尺寸，改为512  
run_hls_standalone.tcl复制到L2/exampls/resize，进行以下修改： 
（路径、part号已在前面改过，这里不改了）  
set PROJ_DIR "$XF_PROJ_ROOT/L2/exampls/resize"  
set PROJ_TOP "resize_accel"   
add_files "${PROJ_DIR}/xf_resize_accel.cpp" -cflags "${VISION_INC_FLAGS} -I${PROJ_DIR}/build -I${PROJ_DIR}/config" -csimflags "${VISION_INC_FLAGS} -I${PROJ_DIR}/build -I${PROJ_DIR}/config"  
add_files -tb "${PROJ_DIR}/xf_resize_tb.cpp" -cflags "${OPENCV_INC_FLAGS} ${VISION_INC_FLAGS} -I${PROJ_DIR}/config -I${PROJ_DIR}/build" -csimflags "${OPENCV_INC_FLAGS} ${VISION_INC_FLAGS} -I${PROJ_DIR}/build -I${PROJ_DIR}/config"  
（测试用图片）
csim_design -ldflags "-L ${OPENCV_LIB} ${OPENCV_LIB_REF}" -argv "${XF_PROJ_ROOT}/data/1024x1024.png"  
cosim_design -ldflags "-L ${OPENCV_LIB} ${OPENCV_LIB_REF}" -argv "${XF_PROJ_ROOT}/data/1024x1024.png"  
由于库里的数据没有1024x1024.png，自己创建一个，放到G:\Download\Vitis_Libraries\vision\data  
按流程跑tcl，生成resize的IP  
