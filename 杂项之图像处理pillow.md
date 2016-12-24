# 杂项之图像处理pillow
## 本节内容
1. 参考文献
2. 生成验证码源码
3. 一些小例子

## 1. 参考文献
http://pillow-cn.readthedocs.io/zh_CN/latest/  pillow中文文档
http://pillow.readthedocs.io/en/3.4.x/  pillow官方文档
http://blog.csdn.net/orangleliu/article/details/43529319  一些小例子
http://python.jobbole.com/83685/  pillow使用方法集合
## 2. 生成验证码源码
```python
#!/usr/bin/env python
# encoding:utf-8
# __author__: check_code
# date: 2016/12/22 9:43
# blog: http://huxianglin.cnblogs.com/ http://xianglinhu.blog.51cto.com/

import random
from PIL import Image, ImageDraw, ImageFont, ImageFilter

_letter_cases = "abcdefghjkmnpqrstuvwxy" # 小写字母，去除可能干扰的i，l，o，z
_upper_cases = _letter_cases.upper() # 大写字母
_numbers = ''.join(map(str, range(3, 10))) # 数字，将数字转换成字符串，并把0,1,2去除掉，防止干扰
init_chars = ''.join((_letter_cases, _upper_cases, _numbers))  # 将上面生成的内容拼接到一起

def create_validate_code(size=(120, 30),
                         chars=init_chars,
                         img_type="GIF",
                         mode="RGB",
                         bg_color=(255, 255, 255),
                         fg_color=(0, 0, 255),
                         font_size=18,
                         font_type="simkai.ttf",
                         length=4,
                         draw_lines=True,
                         n_line=(1, 2),
                         draw_points=True,
                         point_chance = 2):
    '''
    @todo: 生成验证码图片
    @param size: 图片的大小，格式（宽，高），默认为(120, 30)
    @param chars: 允许的字符集合，格式字符串
    @param img_type: 图片保存的格式，默认为GIF，可选的为GIF，JPEG，TIFF，PNG
    @param mode: 图片模式，默认为RGB
    @param bg_color: 背景颜色，默认为白色
    @param fg_color: 前景色，验证码字符颜色，默认为蓝色#0000FF
    @param font_size: 验证码字体大小
    @param font_type: 验证码字体，默认为 ae_AlArabiya.ttf
    @param length: 验证码字符个数
    @param draw_lines: 是否划干扰线
    @param n_lines: 干扰线的条数范围，格式元组，默认为(1, 2)，只有draw_lines为True时有效
    @param draw_points: 是否画干扰点
    @param point_chance: 干扰点出现的概率，大小范围[0, 100]
    @return: [0]: PIL Image实例
    @return: [1]: 验证码图片中的字符串
    '''

    width, height = size # 宽， 高
    img = Image.new(mode, size, bg_color) # 创建图形
    draw = ImageDraw.Draw(img) # 创建画笔，参数中传递了img对象

    def get_chars():
        '''生成给定长度的字符串，返回列表格式'''
        return random.sample(chars, length)

    def create_lines():
        '''绘制干扰线'''
        line_num = random.randint(*n_line) # 干扰线条数

        for i in range(line_num):
            # 起始点
            begin = (random.randint(0, size[0]), random.randint(0, size[1]))
            #结束点
            end = (random.randint(0, size[0]), random.randint(0, size[1]))
            draw.line([begin, end], fill=(0, 0, 0))  # 在起始点和结束点之间画一条线，颜色是黑色

    def create_points():
        '''绘制干扰点'''
        chance = min(100, max(0, int(point_chance))) # 大小限制在[0, 100]  # 设置干扰点在所有点中所占比例

        for w in range(width):
            for h in range(height):
                tmp = random.randint(0, 100)
                if tmp > 100 - chance:
                    draw.point((w, h), fill=(0, 0, 0))  # 满足条件的点就给打成黑色

    def create_strs():
        '''绘制验证码字符'''
        c_chars = get_chars()  # 获取生成的验证码字符串
        strs = ' %s ' % ' '.join(c_chars) # 每个字符前后以空格隔开

        font = ImageFont.truetype(font_type, font_size)  # 生成字体对象，包含字体类型和字体大小
        font_width, font_height = font.getsize(strs)  # 计算生成的文字的宽度和高度

        draw.text(((width - font_width) / 3, (height - font_height) / 3),  # 设置验证码的起始点
                    strs, font=font, fill=fg_color)

        return ''.join(c_chars)  # 返回验证码字符串

    if draw_lines:  # 如果为true则设置干扰线
        create_lines()
    if draw_points:  # 如果为true则设置干扰点
        create_points()
    strs = create_strs()  # 设置字符串

    # 图形扭曲参数
    params = [1 - float(random.randint(1, 2)) / 100,
              0,
              0,
              0,
              1 - float(random.randint(1, 10)) / 100,
              float(random.randint(1, 2)) / 500,
              0.001,
              float(random.randint(1, 2)) / 500
              ]
    img = img.transform(size, Image.PERSPECTIVE, params) # 创建扭曲
    img = img.filter(ImageFilter.EDGE_ENHANCE_MORE) # 滤镜，边界加强（阈值更大）
    return img, strs


#-----------------------------views.py------------------------------------------------
from io import BytesIO
def test(req):
    mstream = BytesIO()  # 在python3里面引入的是BytesIO,python2中引入StringIO
    validate_code = check_code.create_validate_code()  # 拿到两个返回值，第一个是图片对象，第二个是生成的随机字符串
    img = validate_code[0]
    img.save(mstream, "GIF")  # 将拿到的图片对象以 GIF 格式保存到内存中
    return HttpResponse(mstream.getvalue())  # 将内存中保存的图片返回给客户端
'''
img = img.transform(size, Image.PERSPECTIVE, params) # 创建扭曲

Image.transform(size, method, data=None, resample=0, fill=1) 参数
Transforms this image. This method creates a new image with the given size, and the same mode as the original, and copies data to the new image using the given transform.

Parameters:
size – The output size. 输出图片大小
method – The transformation method. This is one of
PIL.Image.EXTENT (cut out a rectangular subregion),  范围
PIL.Image.AFFINE (affine transform),  轮廓
PIL.Image.PERSPECTIVE (perspective transform),  透视
PIL.Image.QUAD (map a quadrilateral to a rectangle),  四方
PIL.Image.MESH (map a number of source quadrilaterals in one operation).  网格
data – Extra data to the transformation method.
resample – Optional resampling filter. It can be one of PIL.Image.NEAREST (use nearest neighbour), PIL.Image.BILINEAR (linear interpolation in a 2x2 environment), or PIL.Image.BICUBIC (cubic spline interpolation in a 4x4 environment). If omitted, or if the image has mode “1” or “P”, it is set to PIL.Image.NEAREST.
Returns:  返回一个新的image对象
An Image object.
'''

'''
img = img.filter(ImageFilter.EDGE_ENHANCE_MORE) # 滤镜，边界加强（阈值更大）

图像滤波在ImageFilter 模块中，在该模块中，预先定义了很多增强滤波器，可以通过filter( )函数使用，预定义滤波器包括：
BLUR、均值滤波
CONTOUR、找轮廓
DETAIL、详细
EDGE_ENHANCE、边缘加强
EDGE_ENHANCE_MORE、
EMBOSS、突出
FIND_EDGES、边缘检测
SMOOTH、平滑
SMOOTH_MORE、
SHARPEN、锐化滤镜
使用该模块时，需先导入
'''
```
**注意**：画笔能提供的功能封装路径:C:\Python35\Lib\site-packages\PIL\ImageDraw.py中的类ImageDraw中

## 一些小例子
```python
# -*- encoding=utf-8 -*-
'''
pil处理图片，验证，处理
大小，格式 过滤
压缩，截图，转换

图片库最好用Pillow
还有一个测试图片test.jpg, 一个log图片，一个字体文件
'''

#图片的基本参数获取
try:
    from PIL import Image, ImageDraw, ImageFont, ImageEnhance
except ImportError:
    import Image, ImageDraw, ImageFont, ImageEnhance

def compress_image(img, w=128, h=128):
    '''''
    缩略图
    '''
    img.thumbnail((w,h))
    im.save('test1.png', 'PNG')
    print u'成功保存为png格式, 压缩为128*128格式图片'

def cut_image(img):
    '''''
    截图, 旋转，再粘贴
    '''
    #eft, upper, right, lower
    #x y z w  x,y 是起点， z,w是偏移值
    width, height = img.size
    box = (width-200, height-100, width, height)
    region = img.crop(box)
    #旋转角度
    region = region.transpose(Image.ROTATE_180)
    img.paste(region, box)
    img.save('test2.jpg', 'JPEG')
    print u'重新拼图成功'

def logo_watermark(img, logo_path):
    '''''
    添加一个图片水印,原理就是合并图层，用png比较好
    '''
    baseim = img
    logoim = Image.open(logo_path)
    bw, bh = baseim.size
    lw, lh = logoim.size
    baseim.paste(logoim, (bw-lw, bh-lh))
    baseim.save('test3.jpg', 'JPEG')
    print u'logo水印组合成功'

def text_watermark(img, text, out_file="test4.jpg", angle=23, opacity=0.50):
    '''''
    添加一个文字水印，做成透明水印的模样，应该是png图层合并
    http://www.pythoncentral.io/watermark-images-python-2x/
    这里会产生著名的 ImportError("The _imagingft C module is not installed") 错误
    Pillow通过安装来解决 pip install Pillow
    '''
    watermark = Image.new('RGBA', img.size, (255,255,255)) #我这里有一层白色的膜，去掉(255,255,255) 这个参数就好了

    FONT = "msyh.ttf"
    size = 2

    n_font = ImageFont.truetype(FONT, size)                                       #得到字体
    n_width, n_height = n_font.getsize(text)
    text_box = min(watermark.size[0], watermark.size[1])
    while (n_width+n_height <  text_box):
        size += 2
        n_font = ImageFont.truetype(FONT, size=size)
        n_width, n_height = n_font.getsize(text)                                   #文字逐渐放大，但是要小于图片的宽高最小值

    text_width = (watermark.size[0] - n_width) / 2
    text_height = (watermark.size[1] - n_height) / 2
    #watermark = watermark.resize((text_width,text_height), Image.ANTIALIAS)
    draw = ImageDraw.Draw(watermark, 'RGBA')                                       #在水印层加画笔
    draw.text((text_width,text_height),
              text, font=n_font, fill="#21ACDA")
    watermark = watermark.rotate(angle, Image.BICUBIC)
    alpha = watermark.split()[3]
    alpha = ImageEnhance.Brightness(alpha).enhance(opacity)
    watermark.putalpha(alpha)
    Image.composite(watermark, img, watermark).save(out_file, 'JPEG')
    print u"文字水印成功"


#等比例压缩图片
def resizeImg(img, dst_w=0, dst_h=0, qua=85):
    '''''
    只给了宽或者高，或者两个都给了，然后取比例合适的
    如果图片比给要压缩的尺寸都要小，就不压缩了
    '''
    ori_w, ori_h = im.size
    widthRatio = heightRatio = None
    ratio = 1

    if (ori_w and ori_w > dst_w) or (ori_h and ori_h  > dst_h):
        if dst_w and ori_w > dst_w:
            widthRatio = float(dst_w) / ori_w                                      #正确获取小数的方式
        if dst_h and ori_h > dst_h:
            heightRatio = float(dst_h) / ori_h

        if widthRatio and heightRatio:
            if widthRatio < heightRatio:
                ratio = widthRatio
            else:
                ratio = heightRatio

        if widthRatio and not heightRatio:
            ratio = widthRatio

        if heightRatio and not widthRatio:
            ratio = heightRatio

        newWidth = int(ori_w * ratio)
        newHeight = int(ori_h * ratio)
    else:
        newWidth = ori_w
        newHeight = ori_h

    im.resize((newWidth,newHeight),Image.ANTIALIAS).save("test5.jpg", "JPEG", quality=qua)
    print u'等比压缩完成'

    '''''
    Image.ANTIALIAS还有如下值：
    NEAREST: use nearest neighbour
    BILINEAR: linear interpolation in a 2x2 environment
    BICUBIC:cubic spline interpolation in a 4x4 environment
    ANTIALIAS:best down-sizing filter
    '''

#裁剪压缩图片
def clipResizeImg(im, dst_w, dst_h, qua=95):
    '''''
        先按照一个比例对图片剪裁，然后在压缩到指定尺寸
        一个图片 16:5 ，压缩为 2:1 并且宽为200，就要先把图片裁剪成 10:5,然后在等比压缩
    '''
    ori_w,ori_h = im.size

    dst_scale = float(dst_w) / dst_h  #目标高宽比
    ori_scale = float(ori_w) / ori_h #原高宽比

    if ori_scale <= dst_scale:
        #过高
        width = ori_w
        height = int(width/dst_scale)

        x = 0
        y = (ori_h - height) / 2

    else:
        #过宽
        height = ori_h
        width = int(height*dst_scale)

        x = (ori_w - width) / 2
        y = 0

    #裁剪
    box = (x,y,width+x,height+y)
    #这里的参数可以这么认为：从某图的(x,y)坐标开始截，截到(width+x,height+y)坐标
    #所包围的图像，crop方法与php中的imagecopy方法大为不一样
    newIm = im.crop(box)
    im = None

    #压缩
    ratio = float(dst_w) / width
    newWidth = int(width * ratio)
    newHeight = int(height * ratio)
    newIm.resize((newWidth,newHeight),Image.ANTIALIAS).save("test6.jpg", "JPEG",quality=95)
    print  "old size  %s  %s"%(ori_w, ori_h)
    print  "new size %s %s"%(newWidth, newHeight)
    print u"剪裁后等比压缩完成"

if __name__ == "__main__":
    ''''' 
    主要是实现功能， 代码没怎么整理
    '''
    im = Image.open('test.jpg')  #image 对象
    compress_image(im)

    im = Image.open('test.jpg')  #image 对象
    cut_image(im)

    im = Image.open('test.jpg')  #image 对象
    logo_watermark(im, 'logo.png')

    im = Image.open('test.jpg')  #image 对象
    text_watermark(im, 'Orangleliu')

    im = Image.open('test.jpg')  #image 对象
    resizeImg(im, dst_w=100, qua=85)

    im = Image.open('test.jpg')  #image 对象  # 这个先切割再生成缩略图的功能之后会在生成头像时用到
    clipResizeImg(im, 100, 200)
```