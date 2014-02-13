UploadFile
==========

android手机客户端上传文件，java servlet服务器端接收并保存到服务器

客户端：
```java
public class MainActivity extends Activity
{
	private TextView uploadInfo;

	@Override
	protected void onCreate(Bundle savedInstanceState)
	{
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		
		uploadInfo = (TextView) findViewById(R.id.upload_info);

		uploadFile();
	}

	public void uploadFile()
	{
		//服务器端地址
		String url = "http://192.168.0.108:8080/UploadFileServer/upload";
		//手机端要上传的文件，首先要保存你手机上存在该文件
		String filePath = Environment.getExternalStorageDirectory()
				+ "/1/power.apk";

		AsyncHttpClient httpClient = new AsyncHttpClient();

		RequestParams param = new RequestParams();
		try
		{
			param.put("file", new File(filePath));
			param.put("content", "liucanwen");
			
			httpClient.post(url, param, new AsyncHttpResponseHandler()
			{
				@Override
				public void onStart()
				{
					super.onStart();
					
					uploadInfo.setText("正在上传...");
				}
				
				@Override
				public void onSuccess(String arg0)
				{
					super.onSuccess(arg0);

					Log.i("ck", "success>" + arg0);
					
					if(arg0.equals("success"))
					{
						Toast.makeText(MainActivity.this, "上传成功！", 1000).show();
					}
					
					uploadInfo.setText(arg0);
				}
				
				@Override
				public void onFailure(Throwable arg0, String arg1)
				{
					super.onFailure(arg0, arg1);
					
					uploadInfo.setText("上传失败！");
				}
			});
			
		} catch (FileNotFoundException e)
		{
			e.printStackTrace();
			Toast.makeText(MainActivity.this, "上传文件不存在！", 1000).show();
		}
	}
}

服务器端：
```java
public class UploadFileServlet extends HttpServlet
{

	public void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException
	{
		response.setContentType("text/html");
		PrintWriter out = response.getWriter();

		// 创建文件项目工厂对象
		DiskFileItemFactory factory = new DiskFileItemFactory();

		// 设置文件上传路径
		String upload = this.getServletContext().getRealPath("/upload/");
		// 获取系统默认的临时文件保存路径，该路径为Tomcat根目录下的temp文件夹
		String temp = System.getProperty("java.io.tmpdir");
		// 设置缓冲区大小为 5M
		factory.setSizeThreshold(1024 * 1024 * 5);
		// 设置临时文件夹为temp
		factory.setRepository(new File(temp));
		// 用工厂实例化上传组件,ServletFileUpload 用来解析文件上传请求
		ServletFileUpload servletFileUpload = new ServletFileUpload(factory);

		// 解析结果放在List中
		try
		{
			List<FileItem> list = servletFileUpload.parseRequest(request);

			for (FileItem item : list)
			{
				String name = item.getFieldName();
				InputStream is = item.getInputStream();

				if (name.contains("content"))
				{
					System.out.println(inputStream2String(is));
				} else if(name.contains("file"))
				{
					try
					{
						inputStream2File(is, upload + "\\" + item.getName());
					} catch (Exception e)
					{
						e.printStackTrace();
					}
				}
			}
			
			out.write("success");
		} catch (FileUploadException e)
		{
			e.printStackTrace();
			out.write("failure");
		}

		out.flush();
		out.close();
	}

	// 流转化成字符串
	public static String inputStream2String(InputStream is) throws IOException
	{
		ByteArrayOutputStream baos = new ByteArrayOutputStream();
		int i = -1;
		while ((i = is.read()) != -1)
		{
			baos.write(i);
		}
		return baos.toString();
	}

	// 流转化成文件
	public static void inputStream2File(InputStream is, String savePath)
			throws Exception
	{
		System.out.println("文件保存路径为:" + savePath);
		File file = new File(savePath);
		InputStream inputSteam = is;
		BufferedInputStream fis = new BufferedInputStream(inputSteam);
		FileOutputStream fos = new FileOutputStream(file);
		int f;
		while ((f = fis.read()) != -1)
		{
			fos.write(f);
		}
		fos.flush();
		fos.close();
		fis.close();
		inputSteam.close();
		
	}
}

联系邮箱：liucanwen517@gmail.com
