# How to make 
![](https://raw.githubusercontent.com/oxygen-berry/imageuploadservice/main/image/202211241057411.png) 
<center>总体框架</center>  

+ ## 摄像头识别模块  

<table><tr><td bgcolor=DarkSeaGreen ><font size=5 color=white>数字识别方案一(仅手写数字识别) : opencv + CNN + PyTorch <font></td></tr></table> 

>#### 1.使用摄像头采集图像 
>``` 
>VideoCapture(“../test.avi”)
>cap = cv2.VideoCapture(0)
>while cap.isOpened():
>    # 读取视频
>    # 第一个ret 为True 或者False,代表有没有读取到图片
>    # 第二个frame表示截取到一帧的图片
>    ret, frame = cap.read()
>    # 判读是否按下q键，按下q键关闭摄像头；
>    if cv2.waitKey(30) & 0xff == ord('q'):
>        break
>    # 显示画面
>    cv2.imwrite('number5.jpg',frame)
>    cv2.imshow('number5.jpg', frame)
># 释放摄像头
>cap.release()
># 销毁窗口
>cv2.destroyAllWindows()
>```  
>#### 2.图像预处理，使其格式与数据集相同 
>灰度图——————二值化——————每个像素取反——————重新修改大小——————转换成tensor型并升维
>```
>img = cv2.imread('number5.jpg')
># 将图片转为灰度图
>gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
># 最大类间方差法(大津算法)，thresh会被忽略，自动计算一个阈值（二值化）
>retval, gray = cv2.threshold(gray, 127, 255, cv2.THRESH_BINARY)
># 创建一个size和img相同的黑色图像
>img2 = np.zeros_like(img)
># 遍历每个通道，反转颜色。采集的图片为黑字白底，而数据集为白字黑底。
>img2[:,:,0] = 255-gray
>img2[:,:,1] = 255-gray
>img2[:,:,2] = 255-gray
>img = cv2.resize(img2, (28, 28))
># 进行神经网络训练时使用的数据类型是tensor型，因此进行转化
>img_tensor = transforms.ToTensor()(img)
>img_tensor = img_tensor.unsqueeze(0) # 升维
>```  
>#### 3.加载模型并转化为onnx类型  
>```
>use_cuda = False
>model = Net(10)
># 注意：此处应把pth文件改为你训练出来的params_x.pth，x为epoch编号
># 一般来讲，编号越大，且训练集（train）和验证集（val）上准确率差别越小的（避免过拟合），效果越好。
># model = Net()新生成的一个新模型，model要将训练出来的params_x.pth中的参数加载加载进来
>model.load_state_dict(torch.load('output/params_yl.pth'))
># model = torch.load('output/model.pth')
># 评估模式。而非训练模式
># 在评估模式下，batchNorm层，dropout层等用于优化训练而添加的网络层会被关闭，从而使得评估时不会发生偏移
>model.eval()
>if use_cuda and torch.cuda.is_available():
>    model.cuda()
># ONNX 的目的在于提供一个跨框架的模型中间表达框架，用于模型转换和部署。ONNX 提供的计算图是通用的，格式也是开源的。
>to_onnx(model, 3, 28, 28, 'output/params.onnx')  
>```   
>#### 4.识别 
>``` 
>if use_cuda and torch.cuda.is_available():
>    prediction = model(Variable(img_tensor.cuda()))
>else:
>    prediction = model(Variable(img_tensor))
># torch.max(x , 1)返回两个结果，
># 第一个是最大值，第二个是对应的索引值；
># 第二个参数 0 代表按列取最大值并返回对应的行索引值，1 代表按行取最大值并返回对应的列索引值。
>pred = torch.max(prediction, 1)[1]
>print(pred) 
>```   

<table><tr><td bgcolor=DarkSeaGreen ><font size=5 color=white>数字识别方案二一(包含手写数字识别、英文字母识别) : opencv + Intel模型推理框架OpenVINO <font></td></tr></table> 

>+ ### 关于openvino  
>#### 1.为什么使用openvino  
>openvino是intel开发的深度学习模型推理加速引擎，总体使用感觉就是方便，压缩后的模型再cpu上跑的速度可以媲美gpu（据称精度损失都小于5%）。使用openvino还有一个优点，就是openvino内置优化过的opencv，处理视频图像更方便。  
>#### 2.如何使用openvino  
>首先要把训练好的各种模型转换为openvino的标准IR模型（包含一个xml文件和bin文件，分别是模型结构和模型参数），然后调用推理引擎去对输入文件进行预测。   
>github源代码链接[点击获取](https://github.com/openvinotoolkit/open_model_zoo)    
>后续部署到树莓派到教程[链接](https://www.intel.cn/content/www/cn/zh/support/articles/000055510/boards-and-kits/neural-compute-sticks.html) 
>
>+ ### 手写数字&单词代码解析 
>#### 1.使用摄像头采集图像 
>同方案一step1   
>#### 2.设备选择&读取模型 
>```
># 设备选择
>    if 'GPU' in args.device:
>        core.set_property("GPU", {"GPU_ENABLE_LOOP_UNROLLING": "NO", "CACHE_DIR": "./"})
># 读取模型
>    log.info('Reading model {}'.format(args.model))
>    model = core.read_model(args.model)
>    if len(model.inputs) != 1:
>        raise RuntimeError("Demo supports only single input topologies")
>    input_tensor_name = model.inputs[0].get_any_name()
>    if args.output_blob is not None:
>        output_tensor_name = args.output_blob
>    else:
>        if len(model.outputs) != 1:
>            raise RuntimeError("Demo supports only single output topologies")
>        output_tensor_name = model.outputs[0].get_any_name()
>```  
>#### 3.获取下载的字符列表
>``` 
># 获取下载的字符列表
>    characters = get_characters(args)
># 在文本标签和文本索引之间转换
>    codec = CTCCodec(characters, args.designated_characters, args.top_k)
>    if len(codec.characters) != model.output(output_tensor_name).shape[2]:
>        raise RuntimeError("The text recognition model does not correspond to decoding characterlist")
>``` 
>其中获取下载字符列表函数如下(args为参数解析器对象）：
>```
>def get_characters(args):
>    with open(args.charlist, 'r', encoding='utf-8') as f:
>        return ''.join(line.strip('\n') for line in f)
>```  
>#### 4.预处理图片
>``` 
>input_batch_size, input_channel, input_height, input_width = model.inputs[0].shape
>input_image = preprocess_input('character.jpg', height=input_height, width=input_width)[None, :, :, :]
>if input_batch_size != input_image.shape[0]:
>    raise RuntimeError("The model's input batch size should equal the input image's batch size")
>if input_channel != input_image.shape[1]:
>    raise RuntimeError("The model's input channel should equal the input image's channel")
>```
>图像预处理函数（转灰度图并resize）
>``` 
>def preprocess_input(image_name, height, width):
>    # 读取图像并转化为灰度值
>    src = cv2.imread(image_name, cv2.IMREAD_GRAYSCALE)
>    # 计算输入图像的长高比
>    ratio = float(src.shape[1]) / float(src.shape[0])
>    # 按这个比例乘模型输入图片的高
>    tw = int(height * ratio)
>    # interpolation: 插值方法;INTER_AREA - 基于局部像素的重采样。
>    # astype修改数据类型
>    rsz = cv2.resize(src, (tw, height), interpolation=cv2.INTER_AREA).astype(np.float32)
>    # [h,w] -> [c,h,w] 将图片扩展成三维数组
>    img = rsz[None, :, :]
>    _, h, w = img.shape
>    # right eqdge padding 进行填充，变成目标大小
>    pad_img = np.pad(img, ((0, 0), (0, height - h), (0, width - w)), mode='edge')
>    return pad_img
>``` 
>#### 5.加载模型并预测
>```
># 加载模型
>    compiled_model = core.compile_model(model, args.device)
>    infer_request = compiled_model.create_infer_request()
># 预测
>    for _ in range(args.number_iter):
>        infer_request.infer(inputs={input_tensor_name: input_image})
>        preds = infer_request.get_tensor(output_tensor_name).data[:]
>        result = codec.decode(preds)
>        print(result)
>``` 

<table><tr><td bgcolor=DarkSeaGreen ><font size=5 color=white>颜色识别 : opencv识别红橙黄绿蓝紫 <font></td></tr></table>  

>摄像头获取图像并设置抓取图像大小——————颜色空间转换（BGR转HSV）——————获取中心点像素位置及hsv——————根据h空间进行颜色识别  
>#### 1.摄像头获取图像并设置抓取图像大小  
>``` 
>cap = cv2.VideoCapture(0)
>if cap.open:
>    print("相机打开成功!\n")
>else:
>    print("未能与相机建立连接...")
># 设置相机属性
>cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
>cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)
>```  
>#### 2.颜色空间转换（BGR转HSV） 
>```
>_, frame = cap.read()
>hsv_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV) 
>``` 
>#### 3.获取中心点像素位置及hsv  
>```
>height, width, _ = frame.shape
>cx = int(width / 2)
>cy = int(height / 2)
># 获取中心位置像素点的hsv值
>pixel_center = hsv_frame[cy, cx]
>``` 
>#### 4.根据h空间进行颜色识别 
>```
> # 获取 H 空间的数值
>hue_value = pixel_center[0] 
>color = "Undefined"
>if hue_value < 5:
>    color = "RED"
>elif hue_value < 22:
>    color = "ORANGE"
>elif hue_value < 33:
>    color = "YELLOW"
>elif hue_value < 78:
>    color = "GREEN"
>elif hue_value < 131:
>    color = "BLUE"
>elif hue_value < 170:
>    color = "VIOLET"
>else:
>    color = "RED"
>```  

<table><tr><td bgcolor=DarkSeaGreen ><font size=5 color=white>物体移动检测：opencv对比视频前后帧 <font></td></tr></table>  

>摄像头获取图像——————预处理——————前后帧的对比并进行处理——————根据面积判断是否有移动  
>#### 1.打开摄像头、设置帧率，获取图像 
>``` 
>camera = cv2.VideoCapture(0)
>if camera.open:
>    print("相机打开成功!\n")
>else:
>    print("未能与相机建立连接...")
>fps = 30  # 帧率
>pre_frame = None  # 总是取前一帧做为背景（不用考虑环境影响）
>
>while True:
>    start = time.time()
>    # 读取视频流
>    res, cur_frame = camera.read()
>    if res != True:
>        break
>    end = time.time()
>    seconds = end - start
>    if seconds < 1.0 / fps: # 每1/30s读取一帧
>        time.sleep(1.0 / fps - seconds) # 挂起等待直到时间满足1/30s
>    cv2.imshow('img', cur_frame)
>    # 检测如何按下Q键，则退出程序
>    key = cv2.waitKey(30) & 0xff
>    if key == 27:
>        break
>``` 
>#### 2.预处理
>``` 
>gray_img = cv2.cvtColor(cur_frame, cv2.COLOR_BGR2GRAY)# 转灰度图
>gray_img = cv2.resize(gray_img, (500, 500))# 将图片缩放
>gray_img = cv2.GaussianBlur(gray_img, (21, 21), 0)# 用高斯滤波进行模糊处理 
>```  
>#### 3.前后帧的对比并进行处理
>``` 
>if pre_frame is None:# 如果没有背景图像就将当前帧当作背景图片
>    pre_frame = gray_img
>else:
>    img_delta = cv2.absdiff(pre_frame, gray_img)# absdiff把两幅图的差的绝对值输出到另一幅图上面来
>    thresh = cv2.threshold(img_delta, 25, 255, cv2.THRESH_BINARY)[1]# threshold阈值函数
>    thresh = cv2.dilate(thresh, None, iterations=2)# 膨胀图像 
>``` 
>#### 4.根据面积判断是否有移动
>``` 
>contours, hierarchy = cv2.findContours(thresh.copy(), cv2.RETR_LIST, cv2.CHAIN_APPROX_SIMPLE)
>for c in contours:
>    # 灵敏度
>    if cv2.contourArea(c) < 1000:
>        continue
>    else:
>        print("something is moving!!!")
>        break
>pre_frame = gray_img
>```













