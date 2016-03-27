---
layout: post
title: 感知哈希算法原理与实现
date: 2016-3-27
categories: blog
tags: [图像处理]
description: 数字图像处理算法
---

### 今天忽然想做一个图像识别的APP，但是在两张图片相似度的问题上产生了问题，感知哈希算法并不能解决这个问题，只是我在试着解决问题的过程中学到的一点知识。

### 这里的关键技术叫做"感知哈希算法"（Perceptual hash algorithm），它的作用是对每张图片生成一个"指纹"（fingerprint）字符串，然后比较不同图片的指纹。结果越接近，就说明图片越相似。   

### 下面是一个最简单的实现：

1. 第一步，缩小尺寸。
将图片缩小到8x8的尺寸，总共64个像素。这一步的作用是去除图片的细节，只保留结构、明暗等基本信息，摒弃不同尺寸、比例带来的图片差异。
 
2. 第二步，简化色彩。
将缩小后的图片，转为64级灰度。也就是说，所有像素点总共只有64种颜色。   

3. 第三步，计算平均值。
计算所有64个像素的灰度平均值。   

4. 第四步，比较像素的灰度。
将每个像素的灰度，与平均值进行比较。大于或等于平均值，记为1；小于平均值，记为0。  

5. 第五步，计算哈希值。
将上一步的比较结果，组合在一起，就构成了一个64位的整数，这就是这张图片的指纹。组合的次序并不重要，只要保证所有图片都采用同样次序就行了。

得到指纹以后，就可以对比不同的图片，看看64位中有多少位是不一样的。在理论上，这等同于计算"汉明距离"（Hamming distance）。如果不相同的数据位不超过5，就说明两张图片很相似；如果大于10，就说明这是两张不同的图片。

这种算法的优点是简单快速，不受图片大小缩放的影响，缺点是图片的内容不能变更。如果在图片上加几个文字，它就认不出来了。所以，它的最佳用途是根据缩略图，找出原图。         

实际应用中，往往采用更强大的pHash算法和SIFT算法，它们能够识别图片的变形。只要变形程度不超过25%，它们就能匹配原图。这些算法虽然更复杂，但是原理与上面的简便算法是一样的，就是先将图片转化成Hash字符串，然后再进行比较。

### 具体实现    

### 工具类   

```
/**
 * 图片工具类，主要针对图片水印处理
 * 
 * @author WANGHONG
 * 
 */
public class ImageHelper {

  // 项目根目录路径
  public static final String path = System.getProperty("user.dir");

  /**
   * 生成缩略图 <br/>
   * 保存:ImageIO.write(BufferedImage, imgType[jpg/png/...], File);
   * 
   * @param source
   *            原图片
   * @param width
   *            缩略图宽
   * @param height
   *            缩略图高
   * @param b
   *            是否等比缩放
   * */
  public static BufferedImage thumb(BufferedImage source, int width, int height, boolean b) {
    // targetW，targetH分别表示目标长和宽
    int type = source.getType();
    BufferedImage target = null;
    double sx = (double) width / source.getWidth();
    double sy = (double) height / source.getHeight();

    if (b) {
      if (sx > sy) {
        sx = sy;
        width = (int) (sx * source.getWidth());
      } else {
        sy = sx;
        height = (int) (sy * source.getHeight());
      }
    }

    if (type == BufferedImage.TYPE_CUSTOM) { // handmade
      ColorModel cm = source.getColorModel();
      WritableRaster raster = cm.createCompatibleWritableRaster(width, height);
      boolean alphaPremultiplied = cm.isAlphaPremultiplied();
      target = new BufferedImage(cm, raster, alphaPremultiplied, null);
    } else
      target = new BufferedImage(width, height, type);
    Graphics2D g = target.createGraphics();
    // smoother than exlax:
    g.setRenderingHint(RenderingHints.KEY_RENDERING, RenderingHints.VALUE_RENDER_QUALITY);
    g.drawRenderedImage(source, AffineTransform.getScaleInstance(sx, sy));
    g.dispose();
    return target;
  }

  /**
   * 图片水印
   * 
   * @param imgPath
   *            待处理图片
   * @param markPath
   *            水印图片
   * @param x
   *            水印位于图片左上角的 x 坐标值
   * @param y
   *            水印位于图片左上角的 y 坐标值
   * @param alpha
   *            水印透明度 0.1f ~ 1.0f
   * */
  public static void waterMark(String imgPath, String markPath, int x, int y, float alpha) {
    try {
      // 加载待处理图片文件
      Image img = ImageIO.read(new File(imgPath));

      BufferedImage image = new BufferedImage(img.getWidth(null), img.getHeight(null), BufferedImage.TYPE_INT_RGB);
      Graphics2D g = image.createGraphics();
      g.drawImage(img, 0, 0, null);

      // 加载水印图片文件
      Image src_biao = ImageIO.read(new File(markPath));
      g.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_ATOP, alpha));
      g.drawImage(src_biao, x, y, null);
      g.dispose();

      // 保存处理后的文件
      FileOutputStream out = new FileOutputStream(imgPath);
      JPEGImageEncoder encoder = JPEGCodec.createJPEGEncoder(out);
      encoder.encode(image);
      out.close();
    } catch (Exception e) {
      e.printStackTrace();
    }
  }

  /**
   * 文字水印
   * 
   * @param imgPath
   *            待处理图片
   * @param text
   *            水印文字
   * @param font
   *            水印字体信息
   * @param color
   *            水印字体颜色
   * @param x
   *            水印位于图片左上角的 x 坐标值
   * @param y
   *            水印位于图片左上角的 y 坐标值
   * @param alpha
   *            水印透明度 0.1f ~ 1.0f
   */

  public static void textMark(String imgPath, String text, Font font, Color color, int x, int y, float alpha) {
    try {
      Font Dfont = (font == null) ? new Font("宋体", 20, 13) : font;

      Image img = ImageIO.read(new File(imgPath));

      BufferedImage image = new BufferedImage(img.getWidth(null), img.getHeight(null), BufferedImage.TYPE_INT_RGB);
      Graphics2D g = image.createGraphics();

      g.drawImage(img, 0, 0, null);
      g.setColor(color);
      g.setFont(Dfont);
      g.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_ATOP, alpha));
      g.drawString(text, x, y);
      g.dispose();
      FileOutputStream out = new FileOutputStream(imgPath);
      JPEGImageEncoder encoder = JPEGCodec.createJPEGEncoder(out);
      encoder.encode(image);
      out.close();
    } catch (Exception e) {
      System.out.println(e);
    }
  }

  /**
   * 读取JPEG图片
   * 
   * @param filename
   *            文件名
   * @return BufferedImage 图片对象
   */
  public static BufferedImage readJPEGImage(String filename) {
    try {
      InputStream imageIn = new FileInputStream(new File(filename));
      // 得到输入的编码器，将文件流进行jpg格式编码
      JPEGImageDecoder decoder = JPEGCodec.createJPEGDecoder(imageIn);
      // 得到编码后的图片对象
      BufferedImage sourceImage = decoder.decodeAsBufferedImage();

      return sourceImage;
    } catch (FileNotFoundException e) {
      e.printStackTrace();
    } catch (ImageFormatException e) {
      e.printStackTrace();
    } catch (IOException e) {
      e.printStackTrace();
    }

    return null;
  }

  /**
   * 读取JPEG图片
   * 
   * @param filename
   *            文件名
   * @return BufferedImage 图片对象
   */
  public static BufferedImage readPNGImage(String filename) {
    try {
      File inputFile = new File(filename);
      BufferedImage sourceImage = ImageIO.read(inputFile);
      return sourceImage;
    } catch (FileNotFoundException e) {
      e.printStackTrace();
    } catch (ImageFormatException e) {
      e.printStackTrace();
    } catch (IOException e) {
      e.printStackTrace();
    }

    return null;
  }

  /**
   * 灰度值计算
   * 
   * @param pixels
   *            像素
   * @return int 灰度值
   */
  public static int rgbToGray(int pixels) {
    // int _alpha = (pixels >> 24) & 0xFF;
    int _red = (pixels >> 16) & 0xFF;
    int _green = (pixels >> 8) & 0xFF;
    int _blue = (pixels) & 0xFF;
    return (int) (0.3 * _red + 0.59 * _green + 0.11 * _blue);
  }

  /**
   * 计算数组的平均值
   * 
   * @param pixels
   *            数组
   * @return int 平均值
   */
  public static int average(int[] pixels) {
    float m = 0;
    for (int i = 0; i < pixels.length; ++i) {
      m += pixels[i];
    }
    m = m / pixels.length;
    return (int) m;
  }
}
```     

### 程序入口      

```
package com.test.image;

import java.awt.image.BufferedImage;
import java.util.ArrayList;
import java.util.List;

public class ImageSearch {

  /**
   * @param args
   */
  public static void main(String[] args) {
    List<String> hashCodes = new ArrayList<String>();

    String filename = ImageHelper.path + "\\images\\";
    String hashCode = null;

    for (int i = 0; i < 6; i++) {
      hashCode = produceFingerPrint(filename + "example" + (i + 1) + ".jpg");
      hashCodes.add(hashCode);
    }
    System.out.println("Resources: ");
    System.out.println(hashCodes);
    System.out.println();

    String sourceHashCode = produceFingerPrint(filename + "source.jpg");
    System.out.println("Source: ");
    System.out.println(sourceHashCode);
    System.out.println();

    for (int i = 0; i < hashCodes.size(); i++) {
      int difference = hammingDistance(sourceHashCode, hashCodes.get(i));
      if (difference == 0) {
        System.out.print("source.jpg图片跟example" + (i + 1) + ".jpg一样");
      } else if (difference <= 5) {
        System.out.print("source.jpg图片跟example" + (i + 1) + ".jpg非常相似");
      } else if (difference <= 10) {
        System.out.print("source.jpg图片跟example" + (i + 1) + ".jpg有点相似");
      } else if (difference > 10) {
        System.out.print("source.jpg图片跟example" + (i + 1) + ".jpg完全不一样");
      }
      System.out.println("\t汉明距离\t" + difference);
    }

  }

  /**
   * 计算"汉明距离"（Hamming distance）。 如果不相同的数据位不超过5，就说明两张图片很相似；如果大于10，就说明这是两张不同的图片。
   * 
   * @param sourceHashCode
   *            源hashCode
   * @param hashCode
   *            与之比较的hashCode
   */
  public static int hammingDistance(String sourceHashCode, String hashCode) {
    int difference = 0;
    int len = sourceHashCode.length();

    for (int i = 0; i < len; i++) {
      if (sourceHashCode.charAt(i) != hashCode.charAt(i)) {
        difference++;
      }
    }
    return difference;
  }

  /**
   * 生成图片指纹
   * 
   * @param filename
   *            文件名
   * @return 图片指纹
   */
  public static String produceFingerPrint(String filename) {
    BufferedImage source = ImageHelper.readPNGImage(filename);// 读取文件

    int width = 8;
    int height = 8;

    // 第一步，缩小尺寸。
    // 将图片缩小到8x8的尺寸，总共64个像素。这一步的作用是去除图片的细节，只保留结构、明暗等基本信息，摒弃不同尺寸、比例带来的图片差异。
    BufferedImage thumb = ImageHelper.thumb(source, width, height, false);

    // 第二步，简化色彩。
    // 将缩小后的图片，转为64级灰度。也就是说，所有像素点总共只有64种颜色。
    int[] pixels = new int[width * height];
    for (int i = 0; i < width; i++) {
      for (int j = 0; j < height; j++) {
        pixels[i * height + j] = ImageHelper.rgbToGray(thumb.getRGB(i, j));
      }
    }

    // 第三步，计算平均值。
    // 计算所有64个像素的灰度平均值。
    int avgPixel = ImageHelper.average(pixels);

    // 第四步，比较像素的灰度。
    // 将每个像素的灰度，与平均值进行比较。大于或等于平均值，记为1；小于平均值，记为0。
    int[] comps = new int[width * height];
    for (int i = 0; i < comps.length; i++) {
      if (pixels[i] >= avgPixel) {
        comps[i] = 1;
      } else {
        comps[i] = 0;
      }
    }

    // 第五步，计算哈希值。
    // 将上一步的比较结果，组合在一起，就构成了一个64位的整数，这就是这张图片的指纹。组合的次序并不重要，只要保证所有图片都采用同样次序就行了。
    StringBuffer hashCode = new StringBuffer();
    for (int i = 0; i < comps.length; i += 4) {
      int result = comps[i] * (int) Math.pow(2, 3) + comps[i + 1] * (int) Math.pow(2, 2) + comps[i + 2] * (int) Math.pow(2, 1) + comps[i + 2];
      hashCode.append(binaryToHex(result));
    }

    // 得到指纹以后，就可以对比不同的图片，看看64位中有多少位是不一样的。
    return hashCode.toString();
  }

  /**
   * 二进制转为十六进制
   * 
   * @param int binary
   * @return char hex
   */
  private static char binaryToHex(int binary) {
    char ch = ' ';
    switch (binary) {
    case 0:
      ch = '0';
      break;
    case 1:
      ch = '1';
      break;
    case 2:
      ch = '2';
      break;
    case 3:
      ch = '3';
      break;
    case 4:
      ch = '4';
      break;
    case 5:
      ch = '5';
      break;
    case 6:
      ch = '6';
      break;
    case 7:
      ch = '7';
      break;
    case 8:
      ch = '8';
      break;
    case 9:
      ch = '9';
      break;
    case 10:
      ch = 'a';
      break;
    case 11:
      ch = 'b';
      break;
    case 12:
      ch = 'c';
      break;
    case 13:
      ch = 'd';
      break;
    case 14:
      ch = 'e';
      break;
    case 15:
      ch = 'f';
      break;
    default:
      ch = ' ';
    }
    return ch;
  }
}
```   
### 完成，但要实现同一个物体两张图片的内容识别出来并判断相似度的道路还是很远啊，哪种算法可以实现这种功能，我还不知道，有知道的同学可以在评论区告诉我一声，多谢。












