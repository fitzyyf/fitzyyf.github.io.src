title: "JAVA使用winrar解压缩和解带有密码的压缩包的一个类"
date: 2008-04-17 22:11
categories: 
- 编程语言
---

最近在开发一个国税相关的系统的时候，需要解压缩带有密码的压缩包，翻了翻JDK，才知道JDK不支持解压带有密码的压缩包。后来在项目经理的建议下，觉得通过使用winrar这个工具很不错。唯一不大好的地方就是需要客户安装winrar，或者在打包成web时候需要将winrar这个软件加载过去。
{% codeblock ZipUtil.java lang:java %}
package com.hfjh.common;

/**
 * 这个类是用来做为解压缩时，对文件的一些操作
 * yfyang 080411
 */
import java.io.File;

public class ZipUtil {

	public static final String password = "123456";
	public static final String winrarPath = "C:\\Program Files\\WinRAR\\WinRAR.exe";

	/**
	 * 将指定的压缩文件解压缩到指定的路径下 解压缩后在指定的路径下生成一个以压缩文件名的文件夹，文件下即为压缩文件的文件
	 *
	 * @param zipFile
	 *            压缩文件路径
	 * @param folder
	 *            要解压到何处的路径
	 * @return
	 */
	public static boolean zip(String zipFile, String folder) {
		boolean bool = false;
		folder = folder + stringUtil(zipFile);
		String cmd = winrarPath + " x -iext -ow -ver -- " + zipFile + " "
				+ folder;
		int source = zipFile.lastIndexOf("\\") + 1;
		String newPath = zipFile.substring(source);
		if (FileUtil.isFileExist(folder, newPath)) {
			bool = false;
			String msg = "在" + folder + "下文件" + newPath + "已经存在";
		} else {
			try {
				Process proc = Runtime.getRuntime().exec(cmd);
				if (proc.waitFor() != 0) {
					if (proc.exitValue() == 0) {
						bool = false;
					}
				} else {
					bool = true;
				}
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
		return bool;
	}

	/**
	 * 解压带有密码的压缩文件 默认密码为123456，如果需要更改则只要修改password的值
	 *
	 * @param zipFile
	 * @param folder
	 * @return
	 */
	public static boolean zipForPassword(String zipFile, String folder) {
		boolean bool = false;
		String _folder = "\"" + folder + stringUtil(zipFile) + "\\" +  "\"";
                zipFile = "\"" + zipFile + "\"";
		String cmd = winrarPath + " x -p" + password + " " + zipFile + " "
				+ _folder;
		int source = zipFile.lastIndexOf("\\") + 1;
		String newPath = zipFile.substring(source);
		String folderName = stringUtil(newPath);

		if (FileUtil.isFileExist(folder, folderName)) {
			bool = false;
			String msg = "在" + folder + "下文件" + newPath + "已经存在";
		} else {
			try {
				Process proc = Runtime.getRuntime().exec(cmd);
				if (proc.waitFor() != 0) {
					if (proc.exitValue() == 0) {
						bool = false;
					}
				} else {
					bool = true;
				}
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
		return bool;
	}



	/**
	 * String的方法工具，主要是针对路径中取得压缩文件的名称，不包括后缀名
	 *
	 * @param str
	 * @return 压缩文件的名称
	 */
	public static String stringUtil(String filePath) {
		String fileName = new File(filePath).getName();
		String fileRealName = null;
		int indexStr = fileName.lastIndexOf(".");
		fileRealName = fileName.substring(0, indexStr);
		return fileRealName;
	}

	/**
	 * 测试类
	 *
	 * @param args
	 */
	public static void main(String[] args) {
		String zipFile = "D:\\企业征信\\2340720080229.rar";
		String folder = "D:\\企业征信\\";
		boolean b = ZipUtil.zipForPassword(zipFile, folder);
		// String path = folder + ZipUtil.stringUtil(zipFile);
		// boolean d = ZipUtil.deleteFolder(path);
		System.out.println(b);
		// System.out.println(d);
	}
}

{% endcodeblock %}
    
