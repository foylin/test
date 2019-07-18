# Android API 描述

本SDK开发指南指导您如何安装和配置开发环境,如何通过调用 SDK 提供的接口函数(API)进行二次开发与系统集成。
用户按照要求调用SDK提供的API即可实现使用 人脸检测/跟踪、活体识别、人脸识别等服务的目的。

### 1. 主要返回参数
    public static final int SUCCESS = 0; //执行接口返回成功
    public static final int ERROR_INVALID_PARAM = -1; //非法参数
    public static final int ERROR_TOO_MANY_REQUESTS = -2; //太多请求
    public static final int ERROR_NOT_EXIST = -3; //不存在
    public static final int ERROR_FAILURE = -4; // 执行接口返回失败

### 2. 构造函数
```java
static FaceAPP GetInstance()

功能	获取单例的对象,人脸识别类采用单例模式,一个类Class只有一个实例存在

参数	无

返回值	FaceAPP类型的对象
```
实例代码 ：
```java
private FaceAPP face = FaceAPP.GetInstance();

```
### 3. 识别人脸特征
```java
int Recognize( Image image, float featureArray [][128], int size, 
	       List<FaceInfo> faceinfos, int[] res ) 

功能 	识别提交的Image中的人脸特征,然后和featureArray里这些特征数组进行比较,找出其中相似度最高的返回
	特征数组的二维数组的索引值

参数	image : 人脸图片
	featureArray : 特征数组的二维数组,特征值数组是存储人脸特征信息的数组,由128个float组成。

返回值	执行成功 SUCCESS
	执行失败 ERROR_FAILURE
```
实例代码 ：
```java
float[][] featurelist = new float[][]; //存储特征值的数组
int size = featurelist.lenth;
int[] ret = new int[1];
byte[] tmpPos = new byte[1024];
FaceAPP.Image image = FaceAPP.GetInstance().new Image();
image.matAddrframe = mRgbaFrame.getNativeObjAddr();
face.Recognize( image, featurelist, size, tmpPos, res );
```

### 4. 识别人脸特征(根据特征值)
```java
int Recognize( float[] feature, float featureArray [][128], int size, 
	       float[] high, int[] res )
    
功能	根据已经存在的人脸特征,然后和featureArray里这些特征数组进行比较,找出其中相似度最高的返回特征
	数组的二维数组的索引值,返回相似度得分值。

参数	eature : 特征数组
	featureArray : 特征数组的二维数组,特征值数组是存储人脸特征信息的数组,由128个float组成。
	size : 特征数组的二维数组大小
	high : float[]型,返回最大得分值
	res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM,
		 image中人脸不在feature数组中返回 ERROR_NOT_EXIST ,
		 >=0特征数组的索引号

返回值
	执行成功 SUCCESS
	执行失败 ERROR_FAILURE
```
实例代码 ：
```java
float[][] featurelist = new float[][]; //存储特征值的数组
int size = featurelist.lenth;
int[] ret = new int[1];
byte[] tmpPos = new byte[1024];
float[] feature;
float[] high = float[1];
FaceAPP.Image image = FaceAPP.GetInstance().new Image();
image.matAddrframe = mRgbaFrame.getNativeObjAddr();
face.Recognize( feature, featurelist, size, high, tmpPos, res );
```

### 5. 检测人脸
```java
int Detect( Image image, List<FaceInfo> faceinfos, int[] res )
    
功能	检测提交的图片中的是否有人脸

参数	image : 人脸图片,用于检测的图片
	faceinfos : FaceInfo 清单。<用于保存返回人脸在图片中的坐标信息>
	res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM

返回值
	执行成功 SUCCESS
	执行失败 ERROR_FAILURE
```
实例代码 ：
```java
int[] ret = new int[1];
byte[] tmpPos = new byte[1024];//byte数组用于存位置信息
FaceAPP.Image image = FaceAPP.GetInstance().new Image(); //初始化
image.matAddrframe = mRgbaFrame.getNativeObjAddr(); //image赋值
if( success = face.Detect( image, tmpPos, res ) ){
//to do
};
```
### 6. 比较特征数据
```java
int Compare( float[] origin, float[] chose, float score )
    
功能	用于比较两个feature值相似度

参数	origin : 待比较feature数组
	chose : 用于比较的feature数组
	score : origin和chose比较的相似度

返回值
	执行成功 SUCCESS
	执行失败 ERROR_FAILURE
```
实例代码 ：
```java
float score;
float[] origin = new float[128];
Float[] chose = new float[128];
face.Compare( origin, chose, score );
```
### 7. 双目活体
```java
int GetFeature( Image image, Image grayImage, float[] feature, 
		List<FaceInfo> faceinfos, int[] res )
    
功能	获取image中的人脸特征值数组,特征值数组是存储人脸特征信息的数组,由128个float型数字组成,
	只获取图片中一个人的特征,多于一人会返回错误信息。

参数	image : 人脸图片,用于检测的图片
	grayImage : 红外摄像头获取的图片
	feature : 存储image中检测到的人脸特征信息,无人脸返回空数组
	faceinfos : FaceInfo 清单。<用于保存返回人脸在图片中的坐标信息>
	res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM

返回值
	SUCCESS : 正确获取人脸信息
	ERROR_FAILURE : 未获取到人脸信息
```
实例代码 ：
```java
float[] feature = new float[128];
int[] ret = new int[1];
byte[] tmpPos = new byte[1024]; //byte数组用于存位置信息
ret = face.GetFeature( image, grayImage, feature, tmpPos, res);
if( ret == SUCCESS ){
//to do 成功获取到活体人脸特征值
}
```
### 8. 双目校准
```java
Int Calibration( Image image, Image grayImage, float[] scale, int[] Rect, int[] res );
    
功能	对红外和普通光组成的双摄像头识别进行校准,要求1个人在最佳位置(0.8-1米)站定,大约需要校验20次,
	得到人脸框修正参数用于人脸画框,返回红外摄像头相对普通光的显示区域坐标,该区域是有效识别和活体检测区域

参数	image : 输入红外图像
	grayImage : 输入彩色图像
	scale : 输出 人脸框修正参数
	rect : 输出红外图像与彩色图像重叠区域(红外图像在彩色图像的对应区域/推荐的检测区域)
	res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM

返回值
	SUCCESS : 正确校准
	ERROR_FAILURE : 校准失败
```
实例代码 ：
```java
int[] ret = new int[1];
float[] scale = new float[1];
int[] rect = new int[4];
ret = face.Calibration( image, grayimage, scale, rect, res );
if( ret == SUCCESS ){
//to do 校准成功
}
```
### 9. 设备鉴权
```java
int AuthorizedDevice( String uidStr, String password, Context activity )
    
功能	设备鉴权

参数	uidStr : OEMID号+合同号
	password : 用户密码
返回值
	0 : 鉴权成功
```
实例代码 ：
```java
String oem_id = "1000000000000001";//OEMID
String contract_id = "0001";//合同号
String password = "0123456789abcdef0123456789abcdef"; //初始授权密码
String uidStr = oem_id + contract_id;
int res = face.AuthorizedDevice( uidStr, password, LoginActivity.this ); 
```
### 10. 获取鉴权激活状态值
```java
int getAuthStatus()
    
功能	获取鉴权激活状态值

参数	无

返回值
	0 : 鉴权成功
```
实例代码 ：
```java
int res = face.getAuthStatus(); 
```
### 11. 提取人脸特征
```java
int GetFeature( Image image, float[] feature, List<FaceInfo> faceinfos, int[] res )
    
功能	获取image中的人脸特征值数组,特征值数组是存储人脸特征信息的数组,由128个float型数字组成,
	只获取图片中一个人的特征。

参数	image : 人脸图片,用于检测的图片
	feature : 存储image中检测到的人脸特征信息,无人脸返回空数组
	faceinfos : FaceInfo 清单。<用于保存返回人脸在图片中的坐标信息>
	res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM

返回值
	SUCCESS : 正确获取人脸信息
	ERROR_FAILURE : 未获取到人脸信息
```
实例代码 ：
```java
float[] feature = new float[128];
int[] ret = new int[1];
byte[] tmpPos = new byte[1024]; //byte数组用于存位置信息
ret = face.GetFeature( image, feature, tmpPos, res );
if( ret == SUCCESS ){
//to do 成功获取到人脸特征值
}
```
### 12. 提取人脸特征(根据人脸坐标信息)
```java
int GetFeature( Image image, FaceInfo detectInfo,float[] feature, int[] res )
    
功能	根据传入的人脸坐标和关键点坐标信息,获取image中的人脸特征值数组,特征值数组是存储人脸特征信息的数组,
	由128个float型数字组成,只获取图片中一个人的特征。

参数	image : 人脸图片,用于检测的图片
	detectInfo : 检测到的人脸信息,用于人脸特征提取
	feature : 存储image中检测到的人脸特征信息,无人脸返回空数组
	res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM

返回值
	SUCCESS : 正确获取人脸信息
	ERROR_FAILURE : 未获取到人脸信息
```
实例代码 ：
```java
float[] feature = new float[128];
int[] ret = new int[1];
float[] detectinfo = new float[]{ x0, y0, x1, y1, landmarkx0, landmarky0,
				 landmarkx1, landmarky1, landmarkx2, landmarky2,
				 landmarkx3, landmarky3, landmarkx4, landmarky4 }
byte[] tmpPos = new byte[1024]; //byte数组用于存位置信息
ret = face.GetFeature( image, detectinfo, feature, res );
if( ret == SUCCESS ){
//to do 成功获取到人脸特征值
}
```
### 13. 提取人脸关键点
```java
int GetLandmark ( Image image, float[] landmark, int[] res )
    
功能	获取人脸关键点坐标信息

参数	Image : 人脸图片,用于提取人脸关键点信息的图片,只提取一个人的关键点信息
	Landmark : 用于存储5个关键点坐标值,依次是左眼,右眼,鼻子,左侧嘴唇,右侧嘴唇
	res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM

返回值
	SUCCESS : 正确获取人脸关键点坐标信息
	ERROR_FAILURE : 未获取到人脸关键点信息
```
实例代码 ：
```java
float[] landmark = new float[10];
int[] ret = new int[1];
ret = face.GetLandmark( image, landmark, res);
if( ret == SUCCESS ){

}
```
### 14. 参数设置
```java
bool SetParameter( const String[] name, float value[] )
    
功能	设置输入的参数名和对应数值

参数	char[] name : 参数的名字
	a : a参数值 内部参数,按示例设置,请不要随意修改
	b : b参数值 内部参数,按示例设置,请不要随意修改
	c : c参数值 内部参数,按示例设置,请不要随意修改
	d : d参数值 内部参数,按示例设置,请不要随意修改
	factor : 检测人脸放大比例 内部参数,按示例设置,请不要随意修改
	min_size : 最小人脸框大小 范围 32-80
	faceclarity : 照片清晰度阈值 范围 建议200-400
	perfoptimize : 是否优化效果 范围 0或1
	livenessdetect : 是否活体检测 范围0-1
	gray2colorscale : 双目活体检测比值 范围 0.1-0.5
	frame_num : 优化的帧数,范围20-40
	quality_thresh : 图片质量阀值,建议范围0.7-0.8
	mode : 工作模式 0 闸机 1 门禁
	facenum : 检测最大人脸数,最多支持检测3张人脸识别1张脸,范围1-3
	value[] : 参数的数值(可能多个)

返回值	参数设置是否成功
```
实例代码 ：
```java
String[] name = { "a", "b","c", "d", "factor", "min_size", "clarity", "perfoptimize", 
		  "livenessdetect", "gray2colorscale", "frame_num", "qualit_thresh",
		  "mode", "facenum" };
double[] value = {0.9, 0.9, 0.9, 0.715, 0.6, 64, 400, 1, 0, 0.5, 20, 0.8, 1, 1 };
face.SetParameter( name, value );
```
### 15. 参数设置(参数可变长度)
```java
bool SetParameters( String[] name, float value[] )
    
功能	可变长度的设置输入的参数名和对应数值

参数	char[] name : 参数的名字(>=1),详情见14,
	value[] : 参数的数值(>=1个)

返回值	参数设置是否成功
```
实例代码 ：
```java
String[] name = { "perfoptimize", "livenessdetect", "frame_num", "quality_thresh", 
		  "mode", "facenum" };
double[] value = { 1, 0, 20, 0.8, 1, 1 };
face.SetParameter( name, value );
```
### 16. 获取版本信息
```java
public String GetVersion()
    
功能	获取当前SDK版本的信息

参数	无

返回值	返回当前版本信息的字符串
```
实例代码 ：
```java
Face.GetVersion();
```
### 17. 获取底层库信息
```java
public String GetFacelibVersion()
    
功能	获取当前SDK底层库的信息

参数	无

返回值	返回当前版本底层库的字符串
```
实例代码 ：
```java
Face.GetFacelibVersion();
```
### 18. 打开人脸数据库
```java
int OpenDB()
    
功能	使用人脸数据库,内部人脸数据库可实现1:N高效快速查询

参数	无

返回值 
	执行成功 SUCCESS
	执行失败 ERROR_FAILURE
```
实例代码 ：
```java
if(face.OpenDB() == SUCCESS){
// TODO
}
```
### 19. 注册人脸
```java
int AddDB( float[] feature, string name )
    
功能	注册人脸

参数	feature : 人脸特征
	name : 登记名字

返回值
	执行成功 SUCCESS
	执行失败 ERROR_FAILURE
```
实例代码 ：
```java
String name= "test";
int[] res=new int[];
if( Face.GetFeature( image, feature, tmpPos, res ) == SUCCESS ){
	Face.AddDB ( feature, name );
}
```
### 20. 查询人脸特征对应名字
```java
String QueryDB( float[] feature, float [] score )
    
功能	给定人脸特征最接近的数据库所登记的人脸,并给出相似度

参数	feature : 人脸特征
	score : 相似度分值
返回值
	执行成功 返回登记名字
	执行失败 返回为unknown
```
实例代码 ：
```java
float[] score = new float[1];
String name = Face.QueryDB( feature, score );
if( score > thresh_hold ){
// TODO
}
```
### 21. 关闭人脸数据库
```java
int CloseDB()
    
功能	关闭人脸数据库

参数	无

返回值
	执行成功 SUCCESS
	执行失败 ERROR_FAILURE
```
实例代码 ：
```java
if( Face.CloseDB() == SUCCESS ){
// TODO
}
```
### 22. 获取人脸属性
```java
int GetFaceAttr( Image image, FaceInfo data, FaceAttribute face_attr, int *res )
    
功能	Detect后执行,通过检测获取的人脸位置和关键点信息,获取人脸属性包括年龄、性别和表情 。

参数	image : 人脸图片,用于检测的图片
	data : 检测得到的人脸位置和人脸关键点信息
	face_attr : 存储计算得到的人脸属性
	res : 执行结果,未发现人脸返回 ERROR_INVALID_PARAM

返回值
	SUCCESS : 正确获取人脸属性
	ERROR_FAILURE : 未获取到人脸属性
```
实例代码 ：
```java
if( face.GetFaceAttr( Image, data, attr, res ) == SUCCESS ){
// TODO
}
```
### 23. 释放人脸识别资源
```java
public void Destroy()
    
功能	释放初始化和设置参数时分配的资源

参数	无

返回值	无
```
实例代码 ：
```java
Face.Destroy(); 
```
### 24. 示例代码
初始化双目摄像头人脸识别Demo。
```java
public class MainActivity extends Activity implements CvCameraViewListener2 {
    
      private FaceAPP face = FaceAPP.GetInstance(); //初始化FaceAPP
        
      @Override
      protected void onCreate(Bundle savedInstanceState) {
        
	String[] name = { "a", "b", "c", "d", "factor", "min_size", "clarity",
				 "perf-optimize", "liveness-detect", "gray2color-scale" };
	double[] value = { 0.9, 0.9, 0.9, 0.715, 0.6, 64, 400, 1, 0, 0.5 };

	//设置人脸识别参数，value[]中值要与name[]中参数一一对应，比如a->0.9, factor->0.6
	face.SetParameter( name, value );

	mainLoop = new Thread() { //人脸检测不要放在 Android 主线程
		public void run() {
		     float[] feature = new float[128];
	 	     byte[] tmpPos = new byte[1024]; // byte 数组用于存位置信息
		     switch ( mixController.curState ){

			  case mixController.STATE_IDLE :

			 	FaceAPP.Image image = FaceAPP.GetInstance ().new Image();

				//mRgbaFrame是人脸图像数据
				image.matAddrframe = mRgbaFrame.getNativeObjAddr(); 
			        int[] res = new int[1];

				//活体判断 + 提取人脸特征值
                                int ret = face.GetFeature( image, feature, tmpPos, res ); 

                                if( ret == SUCCESS ){ // 活体通过，并成功获取到人脸特征值
                                	//to do 成功获取到人脸特征值
                        	}
                           	break;
                    }        
                } 
            }
        }    
    }
```
