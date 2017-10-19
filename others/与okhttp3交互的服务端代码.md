## 此服务端代码的实现环境：myEclipse+Tomcat9.0 配合strus2

```java
package com.nuc;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.PrintWriter;

import javax.enterprise.inject.New;
import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.io.FileUtils;
import org.apache.struts2.ServletActionContext;

import com.opensymphony.xwork2.ActionSupport;

public class UserAction extends ActionSupport{

	private String username;
	
	private String password;
	
	public File mPhoto;
	
	public String mPhotoFileName;
	
	public String mPhotoContentType;//设置类型
	
	
	/**
	 * 上传图片以及用户信息
	 * @return
	 * @throws IOException
	 */
	public String uploadInfo() throws IOException{
		
		System.out.println(username+" , "+password);
		
		if (mPhoto == null) {
			System.out.println(mPhotoFileName + "is null .");
		}
		
		HttpServletRequest request = ServletActionContext.getRequest();
		
		String dir = ServletActionContext.getServletContext().getRealPath("files");
		
		File file = new File(dir,mPhotoFileName);
		
		FileUtils.copyFile(mPhoto,file);
		
		return null;
	}
	
	/**
	 * 上传文件
	 * @return
	 * @throws IOException
	 */
	public String doPostFile() throws IOException{
		
		HttpServletRequest request = ServletActionContext.getRequest();
		
		ServletInputStream si = request.getInputStream();
		
		String dir = ServletActionContext.getServletContext().getRealPath("files");
		
		File file = new File(dir,"test.jpg");
		
		FileOutputStream fos = new FileOutputStream(file);
		
		byte[] buff = new byte[1024];
		
		int len = 0;
		while ((len = si.read(buff))!=-1) {
			
			fos.write(buff,0,len);
			
		}
		
		fos.flush();
		fos.close();
		
		System.out.println("success!");
		return null;
	}
	
	/**
	 * 提交一个字符串
	 * @return
	 * @throws IOException
	 */
	public String doPostString() throws IOException{
		
		HttpServletRequest request = ServletActionContext.getRequest();
		
		ServletInputStream si = request.getInputStream();
		
		StringBuilder sb = new StringBuilder();
		
		byte[] buff = new byte[1024];
		
		int len = 0;
		while ((len = si.read(buff))!=-1) {
			
			sb.append(new String(buff,0,len));
			
		}
		
		si.close();
		System.out.println(sb.toString());
		
		return null;
	}
	
	/**
	 * 请求登录
	 * @return
	 * @throws IOException
	 */
	public String login() throws IOException{
		System.out.println(username+","+password);
		
		HttpServletResponse response = ServletActionContext.getResponse();
		
		PrintWriter  writer = response.getWriter();
		
		writer.write("login success!");
		
		writer.close();
		
		return null;
	}

	public String getUsername() {
		return username;
	}

	public void setUsername(String username) {
		this.username = username;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}
	
	
	
}
```
