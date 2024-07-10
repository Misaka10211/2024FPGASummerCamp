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
