---
layout: post
title: "使用ImageMagick处理图片"
date: 2014-03-11 18:06:07 +0800
comments: true
categories: java
---


在处理图片的过程中，一般都会使用ImageMagick，下面我来介绍下如何在CentOS系统中安装ImageMagick。


首先必须查看下系统是否已安装了ImageMagick


	identify -version


如果已安装，那么先卸载掉旧版本


	yum remove ImageMagick


然后下载安装源文件


	cd /usr/local/src/; wget ftp://mirror.aarnet.edu.au/pub/imagemagick/	ImageMagick-6.8.7-9.tar.gz


解压


	tar zxvf ImageMagick-6.8.7-9.tar.gz


编译


	cd ImageMagick-6.8.7-9
	./configure
	make 
	make install
	ldconfig /usr/local/lib


最后使用命令测试


	convert logo: logo.gif


######在Java中的使用

在java中要使用ImageMagick的话，必须要引入一个jar包，它就是im4java-1.4.0.jar，使用maven的同学，可以直接使用


	<dependency>
		<groupId>org.im4java</groupId>
		<artifactId>im4java</artifactId>
		<version>1.4.0</version>
	</dependency>


直接上代码说明比较方便


	public class ImageUtils {
		private ConvertCmd cmd = new ConvertCmd();
	    	private ImageUtils() {
        	// Windows下需要以下配置
        	// ImageMagick在windows中的安装目录
        	String im = "C:\\Program Files\\ImageMagick-6.8.7-Q16"
        	ProcessStarter.setGlobalSearchPath(im);
    	}
    
    	//缩放
    	public void resize(Integer width, Integer height, String src, String dst) throws Exception {
        	IMOperation imop = new IMOperation();
        	imop.addImage(src);
        	imop.resize(width, height);
        	imop.addImage(dst);
        	cmd.run(imop);
        
    	}
    
    	//剪裁
    	public void crop(String dst,Integer srcWidth, Integer srcHeight, Integer newWidth, Integer newHeight) throws Exception {
        	IMOperation imop = new IMOperation();
        	imop.addImage(dst);
        	imop.crop(srcWidth, srcHeight, newWidth, newHeight);
        	imop.addImage(dst);
        	cmd.run(imop);
        
    	}

更多功能还请参考官方文档.
