# 2024FPGASummerCamp
#lab1  
目前由handcoded生成的bit文件可以传入2560*1440的图片，并正常处理。  
在测试到visionlib时出现问题，导致part3的第14段代码跑不通。  
>>需要下载vision lib sobelfilter项目文件，在xf_sobel_config.h中修改WIDTH和HEIGHT为所需的大小  
>>随后按正常流程重新生成bit和hwh    

#Log  
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
》》xf_sobel_accel.cpp35-37行 删除“boundle=control ”  
pragma HLS INTERFACE s_axilite port=rows  
pragma HLS INTERFACE s_axilite port=cols  
HLS INTERFACE s_axilite port=return  
