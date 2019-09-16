# note2-uploadVideo
简单的记录了Android拍完视频后，上传视频到服务器上的一些方法
# Android上传视频的方法
网络请求工具用的是：OKHTTP
# 代码如下：
```java
public static void uploadVideo(final String url, final String filePath, final Context context, final Activity activity) {
progressDialog = showRequestDialog(context);
progressDialog.show();
//设置上传文件的MultipartBody.Builder对象
MultipartBody.Builder builder = new MultipartBody.Builder().setType(MultipartBody.FORM);
//获得文件路径为：filePath的文件
File file = new File(filePath);
if (file.exists()) {
//设置请求体 header
RequestBody body = RequestBody.create(MediaType.parse("video/*"), file);
builder.addFormDataPart("video", getFileName(), body);
MultipartBody postBody = builder.build();
//设置请求体 footer
//开始请求
Request request = new Request.Builder().url(url).post(postBody).build();
OkHttpClient client = new OkHttpClient();
Call call = client.newCall(request);

				call.enqueue(new Callback() {  

						@Override
						public void onResponse(Call arg0, Response resp) throws IOException {
				// TODO Auto-generated method stub
						String resp_str = resp.body().string();
						String msg = "";
						try {
							JSONObject object = new JSONObject(resp_str);
							msg = object.getString("result");
								} catch (JSONException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
        //文件上传成功的逻辑处理
				if ("0".equals(msg)) {//"0"表示上传成
					activity.runOnUiThread(new Runnable() {

						@Override
						public void run() {
							// TODO Auto-generated method stub
							progressDialog.dismiss();
							Toast.makeText(context, "上传成功", Toast.LENGTH_SHORT).show();
							Intent intent=new Intent();
							intent.setClass(activity, MainActivity.class);
							activity.startActivity(intent);
							activity.finish();
						}
					});
				} else {//文件上传失败的逻辑处理
					activity.runOnUiThread(new Runnable() {

						@Override
						public void run() {
							// TODO Auto-generated method stub
							progressDialog.dismiss();
							Toast.makeText(context, "上传失败", Toast.LENGTH_SHORT).show();	
						}
					});
				}

			}

			@Override
			public void onFailure(Call arg0, IOException arg1) {
				// TODO Auto-generated method stub

			}
		});
	}

}
//获得上传文件名称的方法
private static String getFileName() {
	SimpleDateFormat format = new SimpleDateFormat("YYYYMMDD_HHmmss");
	String fileName = format.format(new Date());
	fileName = fileName + ".mp4";
	return fileName;
}
//上传数据需要显示的进度对话框
private static ProgressDialog showRequestDialog(Context context) {
	ProgressDialog dialog = new ProgressDialog(context);
	dialog.setTitle("正在上传数据");
	dialog.setMessage("请稍后....");
	dialog.setCancelable(false);
	return dialog;
}
```
# 服务器端保存上传文件的方法
后端采用的是spring+springMvc+Hibernate的架构模式<br>
```java
@RequestMapping(value="saveVideo.html")
public void uploadVideo(MultipartHttpServletRequest request,HttpServletResponse response){
System.out.println("saveVideo.html--------我收到了通知");
String msg="-1";
Iterator iter=request.getFileNames();
MultipartFile file=request.getFile("video");
String fileName=file.getOriginalFilename();
String path = "/upload/video/";
//文件保存的路径
String rootName = request.getSession().getServletContext().getRealPath(path);
String fileSavePath = rootName;
//上传文件目录
File uploadPath = new File(fileSavePath);
if(!uploadPath.exists()){
uploadPath.mkdirs();
}
File uploadFile=new File(rootName,fileName);
try {
//这里使用Apache的FileUtils方法来进行保存
FileUtils.copyInputStreamToFile(file.getInputStream(), uploadFile);
msg="0";
} catch (IOException e) {
msg="-1";
System.out.println("文件上传失败"+e.getMessage());
}
//将结果返回给客户端
JSONObject object=new JSONObject();
object.put("result",msg);
PrintWriterUtil.print(object.toString(),response);
}
```
