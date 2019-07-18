# OpenAiLab SDK

Version<br>
![](./img/version_msg.png)

## 1、GPIO port control
[1.1 LED switch](#1_1)<br>
[1.2 控制屏幕亮度](#1_2)<br>
[1.3 背光开关控制](#1_3)<br>
[1.4 继电器、韦根、485控制](#1_4)<br>

<h4 id="1_1">1.1 LED switch</h2>
    
    绿灯：echo X > /sys/class/leds/face:green:user/brightness 
    红灯：echo X > /sys/class/leds/face:red:user/brightness
    白灯：echo X > /sys/class/leds/face:white:user/brightness
    有效值X：0为关闭，1为打开

<h4 id="1_2">1.2 控制屏幕亮度</h2>
   
    屏幕亮度：echo X > /sys/class/backlight/backlight/brightness
    有效值X ：0～255
    
<h4 id="1_3">1.3 背光开关控制</h2>    
    
    屏幕开关：echo X > /sys/class/backlight/backlight/bl_power
    有效值X：0为打开，1为关闭

<h4 id="1_4">1.4 继电器、韦根、485控制</h2>
    
    继电器信号：
        echo 0 > /sys/devices/platform/wiegand-gpio/mode_switch 切换至普通GPIO
        echo 1 >  /sys/devices/platform/wiegand-gpio/D0 	拉高D0
        echo 1 >  /sys/devices/platform/wiegand-gpio/D1	    拉高D1
    
    韦根信号：
        echo 0 > /sys/devices/platform/wiegand-gpio/mode_switch 切换至韦根模式
        echo 卡号 > /sys/devices/platform/wiegand-gpio/wiegand26
        
    485
        echo 1> /sys/devices/platform/wiegand-gpio/mode_switch 切换至485
        echo 内容 > /dev/ttyS4    
    
## 2、Android 快速入门

- A. 添加依赖库

        复制 facelibrary-release.aar和openCVLibrary331-release.aar 到工程下 app/libs 目录
- B 添加编译 aar 信息

        compile(name: 'facelibrary-release', ext: 'aar')
        compile(name: 'openCVLibrary331-release', ext: 'aar')

        到 build.gradle 的 dependencies

- C 在程序中定义 FaceAPP 变量

        private FaceAPP face= FaceAPP.GetInstance(); //face 作为成员变量
- D 通过 face 调用 Android API 接口

## 3、Android API 描述

    public static final int SUCCESS =0; //执行接口返回成功
    public static final int ERROR_INVALID_PARAM =-1; //非法参数
    public static final int ERROR_TOO_MANY_REQUESTS =-2;//太多请求
    public static final int ERROR_NOT_EXIST =-3;//不存在
    public static final int ERROR_FAILURE =-4; // 执行接口返回失败
    
目录<br>
[2.1 构造函数](#1)<br>
[2.2 识别人脸特征](#2)<br>
[2.3 识别人脸特征(根据特征值)](#3)<br>
[2.4 检测人脸](#4)<br>
[2.5 比较特征数据](#5)<br>
[2.6 双目活体](#6)<br>
[2.7 双目校准](#7)<br>
[2.8 设备鉴权](#8)<br> 
[2.9 获取鉴权激活状态值](#9)<br> 
[2.10 提取人脸特征](#10)<br>
[2.11 提取人脸特征(根据人脸坐标信息)](#11)<br>
[2.12 提取人脸关键点](#12)<br>
[2.13 参数设置](#13)<br>
[2.14 参数设置(参数可变长度)](#14)<br>
[2.15 获取版本信息](#15)<br> 
[2.16 获取底层库信息](#16)<br>
[2.17 打开人脸数据库](#17)<br>
[2.18 注册人脸](#18)<br>
[2.19 查询人脸特征对应名字](#19)<br>
[2.20 关闭人脸数据库](#20)<br>
[2.21 获取人脸属性](#21)<br>
[2.22 释放人脸识别资源](#22)<br>
[2.23 示例代码](#23)<br>

<h4 id="1">2.1 构造函数</h2>
    
    static FaceAPP GetInstance()
    
    功能 获取单例的对象,人脸识别类采用单例模式,一个类Class只有一个实例存在
    参数 无
    返回值 FaceAPP类型的对象
    
    操作代码
        private FaceAPP face= FaceAPP.GetInstance();
    

<h4 id="2">2.2 识别人脸特征 </h2>
    
    int Recognize(Image image, float featureArray [][128], int size, List<FaceInfo> faceinfos, int[] res)
    
    功能    识别提交的Image中的人脸特征,然后和featureArray里这些特征数组进行比较,找出其中相似度最高的返回特征数组的二维数组的索引值
    参数    image : 人脸图片
            featureArray : 特征数组的二维数组,特征值数组是存储人脸特征信息的数组,由128个float组成。
            size : 特征数组的二维数组大小
            faceinfos : FaceInfo 清单。<用于保存返回人脸在图片中的坐标信息>
            res: 执行结果,
                未发现人脸返回 ERROR_INVALID_PARAM ,image中人脸不在feature数组中返回 ERROR_NOT_EXIST , >=0特征数组的索引号
    返回值
        执行成功 SUCCESS
        执行失败 ERROR_FAILURE
        
    操作代码
       float[][] featurelist=new float[][]; //存储特征值的数组
        int size= featurelist.lenth;
        int[] ret=new int[1];
        byte[] tmpPos = new byte[1024];
        FaceAPP.Image image= FaceAPP. GetInstance ().new Image();
        image.matAddrframe=mRgbaFrame.getNativeObjAddr();
        face.Recognize(image,featurelist,size, tmpPos,res);

<h4 id="3">2.3 识别人脸特征(根据特征值) </h2>
   
    int Recognize(float[] feature, float featureArray [][128], int size, float[] high, int[] res)
    
    功能    根据已经存在的人脸特征,然后和featureArray里这些特征数组进行比较,找出其中相似度最高的返回特征数组的二维数组的索引值,返回相似度得分值。
    参数    feature : 特征数组
            featureArray : 特征数组的二维数组,特征值数组是存储人脸特征信息的数组,由128个float组成。
            size : 特征数组的二维数组大小
            high: float[]型,返回最大得分值
            res: 执行结果,
                未发现人脸返回 ERROR_INVALID_PARAM ,image中人脸不在feature数组中返回 ERROR_NOT_EXIST , >=0特征数组的索引号
    返回值
        执行成功 SUCCESS
        执行失败 ERROR_FAILURE
        
    操作代码
        float[][] featurelist=new float[][]; //存储特征值的数组
        int size= featurelist.lenth;
        int[] ret=new int[1];
        byte[] tmpPos = new byte[1024];
        float[] feature;
        float[] high=float[1];
        FaceAPP.Image image= FaceAPP. GetInstance ().new Image();
        image.matAddrframe=mRgbaFrame.getNativeObjAddr();
        face.Recognize(feature,featurelist , size, high, tmpPos , res);
    


<h4 id="4">2.4 检测人脸 </h2>
    
    int Detect(Image image, List<FaceInfo> faceinfos ,int[] res)
    
    功能    检测提交的图片中的是否有人脸
    参数    image :人脸图片,用于检测的图片
            faceinfos : FaceInfo 清单。<用于保存返回人脸在图片中的坐标信息>
            res: 执行结果,未发现人脸返回 ERROR_INVALID_PARAM
    返回值
        执行成功 SUCCESS
        执行失败 ERROR_FAILURE
        
    操作代码
        int[] ret=new int[1];
        byte[] tmpPos = new byte[1024];//byte数组用于存位置信息
        FaceAPP.Image image= FaceAPP. GetInstance ().new Image(); //初始化
        image.matAddrframe=mRgbaFrame.getNativeObjAddr();//image赋值
        if(success=face.Detect(image,tmpPos,res)){
        //to do
        };

<h4 id="5">2.5 比较特征数据 </h2>
    
    int Compare(float[] origin, float[] chose, float score)
    
    功能    用于比较两个feature值相似度
    参数    origin:待比较feature数组
            chose:用于比较的feature数组
            score :origin和chose比较的相似度
    返回值
        执行成功 SUCCESS
        执行失败 ERROR_FAILURE
    
    操作代码
        Float score
        float[] origin=new float[128];
        Float[] chose=new float[128];
        face.Compare(origin,chose,score);

<h4 id="6">2.6 双目活体 </h2>
    
    int GetFeature(Image image,Image grayImage,float[] feature,List<FaceInfo>faceinfos , int[] res)
    
    功能    获取image中的人脸特征值数组,特征值数组是存储人脸特征信息的数组,由128个float型数字组成,只获取图片中一个人的特征,多于一人会返回错误信息。
    参数    image :人脸图片,用于检测的图片
            grayImage:红外摄像头获取的图片
            feature:存储image中检测到的人脸特征信息,无人脸返回空数组
            faceinfos : FaceInfo 清单。<用于保存返回人脸在图片中的坐标信息>
            res: 执行结果,未发现人脸返回 ERROR_INVALID_PARAM
    返回值
        SUCCESS :正确获取人脸信息
        ERROR_FAILURE :未获取到人脸信息
        
    操作代码
        float[] feature=new float[128];
        int[] ret=new int[1];
        byte[] tmpPos = new byte[1024];//byte数组用于存位置信息
        ret= face.GetFeature(image,grayImage,feature,tmpPos,res);
        if(ret== SUCCESS){
        //to do 成功获取到活体人脸特征值
        }
    
<h4 id="7">2.7 双目校准 </h2>
    
    Int Calibration(Image image,Image grayImage,float[] scale,int[] Rect,int[] res);
    
    功能    对红外和普通光组成的双摄像头识别进行校准,要求1个人在最佳位置(0.8-1米)站定,大约需要校验20次,得到人脸框修正参数用于人脸画框,返回红外摄像头相对普通光的显示区域坐标,该区域是有效识别和活体检测区域
    参数    image :输入红外图像
            grayImage :输入彩色图像
            scale:输出 人脸框修正参数
            rect:输出红外图像与彩色图像重叠区域(红外图像在彩色图像的对应区域/推荐的检测区域)
            res: 执行结果,未发现人脸返回 ERROR_INVALID_PARAM
    返回值
        SUCCESS :正确校准
        ERROR_FAILURE :校准失败
        
    操作代码
        int[] ret=new int[1];
        float[] scale=new float[1];
        int[] rect=new int[4];
        ret= face.Calibration(image,grayimage,scale,rect,res);
        if(ret== SUCCESS){
        //to do 校准成功
        }

<h4 id="8">2.8 设备鉴权 </h2>
    
    int AuthorizedDevice(String uidStr,String password,Context activity)
    
    功能    设备鉴权
    参数    uidStr : OEMID号+合同号
            password: 用户密码
    返回值
        0:鉴权成功
        
    操作代码
        String oem_id ="1000000000000001";//OEMID
        String contract_id ="0001";//合同号
        String password = "0123456789abcdef0123456789abcdef";//初始授权密码
        String uidStr = oem_id+contract_id;
        int res =face.AuthorizedDevice(uidStr,password, LoginActivity.this); 
    
<h4 id="9">2.9 获取鉴权激活状态值 </h2>
    
    int getAuthStatus()
    
    功能    获取鉴权激活状态值
    参数    无
    返回值
        0:鉴权成功
    
    操作代码
        int res =face.getAuthStatus(); 
    
<h4 id="10">2.10 提取人脸特征 </h2>
    
    int GetFeature(Image image, float[] feature,List<FaceInfo> faceinfos , int[] res)
    
    功能    获取image中的人脸特征值数组,特征值数组是存储人脸特征信息的数组,由128个float型数字组成,只获取图片中一个人的特征。
    参数    image :人脸图片,用于检测的图片
            feature:存储image中检测到的人脸特征信息,无人脸返回空数组
            faceinfos : FaceInfo 清单。<用于保存返回人脸在图片中的坐标信息>
            res: 执行结果,未发现人脸返回 ERROR_INVALID_PARAM
    返回值
        SUCCESS :正确获取人脸信息
        ERROR_FAILURE :未获取到人脸信息
        
    操作代码
        float[] feature=new float[128];
        int[] ret=new int[1];
        byte[] tmpPos = new byte[1024];//byte数组用于存位置信息
        ret= face.GetFeature(image,feature,tmpPos,res);
        if(ret== SUCCESS){
        //to do 成功获取到人脸特征值
        }
        
<h4 id="11">2.11 提取人脸特征(根据人脸坐标信息) </h2>
    
    int GetFeature(Image image, FaceInfo detectInfo,float[] feature, int[] res)
    
    功能    根据传入的人脸坐标和关键点坐标信息,获取image中的人脸特征值数组,特征值数组是存储人脸特征信息的数组,由128个float型数字组成,只获取图片中一个人的特征。
    参数    image :人脸图片,用于检测的图片
            detectInfo:检测到的人脸信息,用于人脸特征提取
            feature:存储image中检测到的人脸特征信息,无人脸返回空数组
            res: 执行结果,未发现人脸返回 ERROR_INVALID_PARAM
    返回值
        SUCCESS :正确获取人脸信息
        ERROR_FAILURE :未获取到人脸信息
        
    操作代码
        float[] feature=new float[128];
        int[] ret=new int[1];
        float[] detectinfo=new
        float[]{x0,y0,x1,y1,landmarkx0,landmarky0,landmarkx1,landmarky1,
        landmarkx2,landmarky2,landmarkx3,landmarky3, landmarkx4,landmarky4}
        byte[] tmpPos = new byte[1024];//byte数组用于存位置信息
        ret= face.GetFeature(image,detectinfo,feature,res);
        if(ret== SUCCESS){
        //to do 成功获取到人脸特征值
        }

<h4 id="12">2.12 提取人脸关键点 </h2>
    
    int GetLandmark(Image image, float[] landmark,int[] res)
    
    功能    获取人脸关键点坐标信息
    参数    Image:人脸图片,用于提取人脸关键点信息的图片,只提取一个人的关键点信息
            Landmark:用于存储5个关键点坐标值,依次是左眼,右眼,鼻子,左侧嘴唇,右侧嘴唇
            res: 执行结果,未发现人脸返回 ERROR_INVALID_PARAM
    返回值
        SUCCESS :正确获取人脸关键点坐标信息
        ERROR_FAILURE :未获取到人脸关键点信息
        
    操作代码
        float[] landmark=new float[10];
        int[] ret=new int[1];
        ret=face.GetLandmark(image,landmark,res);
        if(ret== SUCCESS){
        }
    
<h4 id="13">2.13 参数设置 </h2>
   
    bool SetParameter(const String[] name, float value[] )
    
    功能    设置输入的参数名和对应数值
    参数    char[] name :参数的名字
            a :a参数值 内部参数,按示例设置,请不要随意修改
            b :b参数值 内部参数,按示例设置,请不要随意修改
            c :c参数值 内部参数,按示例设置,请不要随意修改
            d : d参数值 内部参数,按示例设置,请不要随意修改
            factor :检测人脸放大比例 内部参数,按示例设置,请不要随意修改
            min_size :最小人脸框大小 范围 32-80
            faceclarity :照片清晰度阈值 范围 建议200-400
            perfoptimize: 是否优化效果 范围 0或1
            livenessdetect:是否活体检测 范围0-1
            gray2colorscale:双目活体检测比值 范围 0.1-0.5
            frame_num:优化的帧数,范围20-40
            quality_thresh:图片质量阀值,建议范围0.7-0.8
            mode:工作模式 0 闸机 1 门禁
            facenum:检测最大人脸数,最多支持检测3张人脸识别1张脸,范围1-3
            value[] :参数的数值(可能多个)
    返回值  参数设置是否成功
    
    操作代码
        String[] name={"a","b","c","d","factor","min_size","clarity","perfoptimize","livenessdetect","gray2colorscale","frame_num","qualit_thresh","mode","facenum"};
        double[] value={0.9,0.9,0.9,0.715,0.6,64,400,1,0,0.5,20,0.8,1,1};
        face.SetParameter(name,value);

<h4 id="14">2.14 参数设置(参数可变长度) </h2>
    
    bool SetParameters(const String[] name, float value[] )
    
    功能    可变长度的设置输入的参数名和对应数值
    参数    char[] name :参数的名字(>=1),详情见1.13,
            value[] :参数的数值(>=1个)
    返回值 参数设置是否成功
    
    操作代码
        String[]name={"perfoptimize","livenessdetect","frame_num","quality_thresh","mode","facenum"};
        double[] value={1,0,20,0.8,1,1};
        face.SetParameter(name,value);

<h4 id="15">2.15 获取版本信息 </h2>
    
    public String GetVersion()
    
    功能    获取当前SDK版本的信息
    参数    无
    返回值  返回当前版本信息的字符串
    
    操作代码
        Face.GetVersion();
    
<h4 id="16">2.16 获取底层库信息 </h2>
    
    public String GetFacelibVersion()

    功能    获取当前SDK底层库的信息
    参数    无
    返回值  返回当前版本底层库的字符串
    
    操作代码
        Face.GetFacelibVersion();
    
<h4 id="17">2.17 打开人脸数据库 </h2>
    
    int OpenDB()

    功能    使用人脸数据库,内部人脸数据库可实现1:N高效快速查询
    参数    无
    返回值 
        执行成功 SUCCESS
        执行失败 ERROR_FAILURE
        
    操作代码
        if(face.OpenDB () == SUCCESS){
        // TODO
        }
    
<h4 id="18">2.18 注册人脸 </h2>
    
    int AddDB(float[] feature, string name)
    
    功能    注册人脸
    参数    feature:人脸特征
            name: 登记名字
    返回值
        执行成功 SUCCESS
        执行失败 ERROR_FAILURE
    
    操作代码
        String name= "test";
        int[]   res=new int[];
        if(Face.GetFeature(image,feature,tmpPos,res)==SUCCESS){
            Face.AddDB (feature,name);
        }
    
<h4 id="19">2.19 查询人脸特征对应名字 </h2>
    
    String QueryDB(float[] feature,float [] score)
    
    功能    给定人脸特征最接近的数据库所登记的人脸,并给出相似度
    参数    feature: 人脸特征
            score: 相似度分值
    返回值
        执行成功 返回登记名字
        执行失败 返回为unknown
    
    操作代码
        float[] score=new float[1];
        String name=Face.QueryDB(feature,score);
        if(score>thresh_hold){
        // TODO
        }

<h4 id="20">2.20 关闭人脸数据库 </h2>
    
    int CloseDB()
    
    功能    关闭人脸数据库
    参数    无
    返回值
        执行成功 SUCCESS
        执行失败 ERROR_FAILURE
        
    操作代码
        if(Face.CloseDB () == SUCCESS){
        // TODO
        }

<h4 id="21">2.21 获取人脸属性 </h2>
    
    int GetFaceAttr(Image image,FaceInfo data,FaceAttribute face_attr,int *res)
    
    功能    Detect后执行,通过检测获取的人脸位置和关键点信息,获取人脸属性包括年龄、性别和表情 。
    参数    image :人脸图片,用于检测的图片
            data:检测得到的人脸位置和人脸关键点信息
            face_attr : 存储计算得到的人脸属性
            res: 执行结果,未发现人脸返回 ERROR_INVALID_PARAM
    返回值
        SUCCESS :正确获取人脸属性
        ERROR_FAILURE :未获取到人脸属性
        
    操作代码
        if(faceapp::FetchFaceAttr(Image,data,attr,&res) == SUCCESS){
        // TODO
        }

<h4 id="22">2.22 释放人脸识别资源 </h2>
   
    public void Destroy()

    功能    释放初始化和设置参数时分配的资源
    参数    无
    返回值  无
    
    操作代码
        Face.Destroy(); 
    
<h4 id="23">2.23 示例代码 </h2>

    public class MainActivity extends Activity implements CvCameraViewListener2 {
    
        private FaceAPP face= FaceAPP.GetInstance(); //face 作为成员变量
        
        @Override
        protected void onCreate(Bundle savedInstanceState) {
        
            String[] name={"a","b","c","d","factor","min_size","clarity","perf-optimize","liveness-detect","gray2color-scale"};
            double[] value={0.9,0.9,0.9,0.715,0.6,64,400,1,0,0.5};
            face.SetParameter(name,value);
            mainLoop = new Thread() { //人脸检测不要放在 Android 主线程
                public void run() {
                    float[] feature=new float[128];
                    byte[] tmpPos = new byte[1024];// byte 数组用于存位置信息
                    switch (mixController.curState){
                        case mixController. STATE_IDLE :
                                FaceAPP.Image image= FaceAPP. GetInstance ().new Image();
                                 image.matAddrframe=mRgbaFrame.getNativeObjAddr();
                                int[] res=new int[1];
                                int ret;
                                ret= face.GetFeature(image,feature,tmpPos,res);
                                if(ret== SUCCESS){
                                //to do 成功获取到人脸特征值
                                }
                            break;
                    }        
                } 
            }
        }    
    }
